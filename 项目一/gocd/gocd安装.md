# GoCD - 简介

GoCD 是开源的持续集成与持续交付系统

GoCD 由 GoCD Server 和 GoCD Agents 组成，进行构建需要至少一个 GoCD Agents.

GoCD Server 提供界面给用户用于控制整个 GoCD 系统，并且分发工作给 agents.

GoCD Agents 完成 server 下发的工作（运行命令，部署工程，等等..）
# ubuntu20.04  helm 安装gocd
参考：https://artifacthub.io/packages/helm/gocd/gocd
```
helm repo add gocd https://gocd.github.io/helm-chart
"gocd" has been added to your repositories
helm search repo gocd
NAME       	CHART VERSION	APP VERSION	DESCRIPTION                                       
gocd/gocd  	1.40.8       	21.4.0     	GoCD is an open-source continuous delivery serv...
stable/gocd	1.32.0       	20.8.0     	GoCD is an open-source continuous delivery serv...
```
最新版本以及稳定版
**用了10块云盘多的raid0    raid0,读写速度快，无冗余，容量无损失，数据安全性最低**
# 错误一、
root@k8s2-01:/home/ubuntu# helm install gocd stable/gocd
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest
## stable/gocd的repo连接不上。选择了gocd/gocd这支repo
# 安装
```
root@k8s2-01:/home/ubuntu# kubectl create ns gocd
namespace/gocd created
root@k8s2-01:/home/ubuntu# helm install --namespace gocd  gocd gocd/gocd
NAME: gocd
LAST DEPLOYED: Sun Mar  6 06:02:26 2022
NAMESPACE: gocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
# 在gocd-server的deployment增加静态卷   gocd-agent也需要修改为静态卷
```
      - hostPath:
          path: /data1/ops
          type: Directory
        name: goserver-vol
        
      dnsPolicy: ClusterFirst
      nodeSelector:
        gocd: client
        
kubectl label nodes k8s2-01 gocd=client
kubectl taint nodes --all node-role.kubernetes.io/master-
```
# gocd连接gitlab
```
kubectl exec -it -n gocd gocd-server-78979ff8c8-6fqqm /bin/bash

   83  ssh-keygen 
   85  cd .ssh/
   87  cat id_rsa.pub 
登录gitlab-->点击profile settings-->SSH Keys-->打开id_rsa.pub（公钥） 复制全部内容到Key这个选项
   89  git clone git@gitlab.51sw.cc:inner/inner-bw-pic.git
```

出于安全原因，所有新安装的 GoCD 代理都需要在 GoCD 服务器上启用，然后才能将工作分配给它们。这可以防止未经授权的人访问您的源代码。
要启用新安装的 GoCD 代理，请执行以下操作：
1.打开 GoCD 服务器仪表板
2.agents 找到新添加的agent并启动。GoCD 服务器现在将为此代理安排工作。

# 错误二、
```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: 
```
/var/run/docker.sock是挂载到了gocd-agent上,但还需要chmod 777 /var/run/docker.sock

# 错误三、
```
Step 1/12 : FROM docker-public.lwork.com:5000/base-jdk:v1.2
Get "https://docker-public.lwork.com:5000/v2/": http: server gave HTTP response to HTTPS client
[script-executor] Script completed with exit code: 1.
```
gocd所在的服务器将docker-public.lwork.com:5000加入
```
root@ali-hk-public-ops-k8s-master01:~# cat /etc/docker/daemon.json
{
  "insecure-registry": "docker-public.lwork.com:5000"
}
```