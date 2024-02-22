 # 一、jenkins新建项目
## 选择"构建一个自由风格的软件项目"-->名称inner-bw-pic
## '丢弃旧的构建'
## '参数化构建过程'
1.添加'Git参数'-->名称BRANCH-->参数类型分支或标签-->默认值origin/master
2.添加'布尔值参数'-->名称DEPLOY
3.添加'布尔值参数'-->名称UPLOAD
4.添加'布尔值参数'-->名称DEPLOY_QA
## '源码管理'
选git，git@gitlab.51sw.cc:inner/inner-bw-pic.git
Branches to build指定分支-->填写$BRANCH
## '构建环境'
选择Delete workspace before build starts
## '构建'
增加构建步骤-->执行shell-->命令
`bash /var/lib/jenkins/workspace/jenkins-deploy/crm-deploy.sh 5.0.$BUILD_NUMBER inner-bw-pic  ${DEPLOY} ${UPLOAD} ${DEPLOY_QA}`

完善"执行 shell"中的脚本来实现全部的部署过程。
```
cat /var/lib/jenkins/workspace/jenkins-deploy/crm-deploy.sh
#!/bin/bash
TIME=`date "+%Y-%m-%d %H:%M"`
VERSION=$1
ARTIFACT=$2
DEPLOY_DEV=$3
UPLOAD_NO_MASTER=$4
DEPLOY_QA=$5

echo "$ARTIFACT is tony"
DEPLOY_DIR=/var/lib/jenkins/workspace/${ARTIFACT}
SCRIPT_DIR=/var/lib/jenkins/workspace/jenkins-deploy

PEXIT (){
	echo $1
	exit 9
}

cd ${DEPLOY_DIR}

git pull

echo "{'name': '$ARTIFACT', 'version': '${BUILD_NUMBER}', 'time': '$TIME'}" >test
mvn clean install -Dmaven.test.skip=true ||PEXIT "mvn build failed"

sed "s/dog/${ARTIFACT}/g" ../Dockerfile >Dockerfile


if [ ${UPLOAD_NO_MASTER} = true ]
then
	echo '------------'
	echo ${BUILD_NUMBER}
	echo '------------'
	docker build . -t 192.168.60.231:5000/${ARTIFACT}:v${BUILD_NUMBER}
	docker login 192.168.60.231:5000 -u admin -p test
	docker push 192.168.60.231:5000/${ARTIFACT}:v${BUILD_NUMBER}
	docker logout 192.168.60.231:5000
	docker rmi 192.168.60.231:5000/${ARTIFACT}:v${BUILD_NUMBER}
fi

#if [ ${DEPLOY_DEV} = true ]
#then
#	echo "开始发布到k8s"
#	ssh root@192.168.60.168 "kubectl set image deployment/${ARTIFACT} ${ARTIFACT}=192.168.60.231:5000/${ARTIFACT}:v$BUILD_NUMBER} -n crm"
#fi
-----------------------------------------------------------------------------------------------------------------
cat /var/lib/jenkins/workspace/Dockerfile
FROM docker-public.test.com:5000/base-jdk:v1.2 #这就是一个安装了JDK的最简linux系统，已经提前制作并上传到仓库
MAINTAINER blue
ENV user=crm
ENV PORT=10200
RUN useradd $user -m -d /home/crm && mkdir -p /home/${user}/log
COPY target/*.jar /home/${user}/dog.jar
RUN chown -R ${user}.${user} /home/${user}
VOLUME /home/${user}/log
EXPOSE  ${PORT}
WORKDIR /home/${user}
USER ${user}
ENTRYPOINT exec java ${JAVA_OPT} -jar dog.jar --spring.cloud.config.profile=${RUN_ENV} --spring.profiles.active=${RUN_ENV}
```
# 二、镜像上传仓库及发布

## 1.新建仓库，仓库地址192.168.60.231:5000
```
Name:docker-registry
Format:docker
Type:hosted
Online:If checked, the repository accepts incoming requests
HTTP:#开放端口5000
Create an HTTP connector at specified port. Normally used if the server is behind a secure proxy.
5000

Storage
default
Validate that all content uploaded to this repository is of a MIME type appropriate for the repository format
Hosted
Allow redeploy
```
## 2.jenkins新项目inner-bw-pic点击构建
**错误一**：
`Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock`
处理：
`sudo gpasswd -a jenkins docker`
重新构建还是一样的错，发现下面错误提示有一句
`dial unix /var/run/docker.sock: connect: permission denied`
处理：
`chmod 777 /var/run/docker.sock`
-------------------------------------------------------
## 3.先配置连接私有仓库
`docker login 192.168.60.231:5000 -u admin -p test`
登录时，需要提供用户名和密码。认证的信息会被保存在~/.docker/config.json文件，在后续与私有镜像仓库交互时就可以被重用，而不需要每次都进行登录认证。
**错误二**
`Error response from daemon: Get "https://192.168.60.231:5000/v2/": http: server gave HTTP response to HTTPS client`
由于使用的是http协议，连接仓库前需要进行配置 vim /etc/docker/daemon.json
#在文件中添加如下的内容，告诉docker这个私有镜像仓库是一个安全的仓库：
`"insecure-registries": ["192.168.60.231:5000"]`
```
root@cn-office-tonytest-jenkins:~# cat /etc/docker/daemon.json 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": ["docker-public.test.com:5000","192.168.60.231:5000"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1000m"
  },
  "storage-driver": "overlay2"
}
    systemctl daemon-reload
    systemctl restart docker
```
**验证：**
jenkins@cn-office-tonytest-jenkins:~$ docker login 192.168.60.231:5000 -u admin -p test
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/.docker/config.json.

Login Succeeded
jenkins@cn-office-tonytest-jenkins:~$ cat .docker/config.json 
```
{
	"auths": {
		"192.168.60.231:5000": {
			"auth": "YWRtaW46Tmdpbng4MDE="
		},
		"docker-public.test.com:5000": {
			"auth": "YWRtaW46bGVhbndvcmsyMDE4"
		}
	}
}
```
**重新构建，编译，打包，制作镜像并上传到仓库，成功**
**错误三**
如果使用的是jenkins这个普通用户进行发布，可能需要在sudoer中增加jenkins用户免密使用docker命令
```
root@lwdevsrv01:~# cat /etc/sudoers
增加
jenkins ALL=(ALL)NOPASSWD:/usr/bin/docker
```
# 三、发布到k8s集群
jenkins脚本里面有直接更新镜像到开发环境k8s
# 四、k8s集群拉取上传到仓库的镜像
参考:https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/
通过secret yaml文件创建pull image所用的secret
If you already ran docker login
不一定非要在跑k8s上执行docker login。jenkins为了上传镜像到私有仓库，执行过docker login，他的.docker/config.json一样的效果
`base64 -w 0 ~/.docker/config.json`
cat pull-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: pull-secret
  namespace: crm
data:
    .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxOTIuMTY4LjAuOTY6ODA4MiI6IHsKCQkJImF1dGgiOiAiWVdSdGFXNDZUSGR2Y21zdVkyOXRNVEl6IgoJCX0KCX0sCgkiSHR0cEhlYWRlcnMiOiB7CgkJIlVzZXItQWdlbnQiOiAiRG9ja2VyLUNsaWVudC8xOC4wOS42IChsaW51eCkiCgl9Cn0=
type: kubernetes.io/dockerconfigjson
```

## 错误一 :部署secret后拉取镜像仍然报错
Failed to pull image "192.168.60.231:5000/inner-bw-pic:v8": rpc error: code = Unknown desc = Error response from daemon: Get "": http: server gave HTTP response to HTTPS client
将"192.168.60.231:5000"加入/etc/docker/daemon.json 应该是能解决问题
"insecure-registries": ["192.168.60.231:5000"]
## 处理办法
之前在阿里云部署vpn.51sw.cc申请了个证书，这里用nginx将vpn.51sw.cc指向192.168.60.231:5000
并在k8s集群服务器写死/etc/hosts(写hosts可能不行，需要在k8s的deployment中使用别名)
`192.168.60.179 vpn.51sw.cc`
```
      hostAliases:
      - hostnames:
        - gitlab.51sw.cc
        ip: 192.168.60.236
      - hostnames:
        - vpn.51sw.cc
        ip: 192.168.60.179
```
docker login vpn.51sw.cc -u admin -p test成功，并不需要把这个域名加入/etc/docker/daemon.json应该是用了https的原因
重新`base64 -w 0 ~/.docker/config.json` 替换上面pull-secret.yaml中的内容
再重新应用一次
`kubectl apply -f pull-secret.yaml`
### **k8s集群成功拉取到私有仓库存放的代码**

