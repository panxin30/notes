# 一、部署说明
## 将git clone代码到本地，用maven编译，打包成jar包，利用Dockerfile构建镜像的这一过程在jenkins上参数化，脚本化

# 二、安装maven环境
管理jenkins-->全局工具配置-->Maven选项-->新增maven-->选择版本，勾选自动安装-->点击保存
那这些在什么时候才真正的安装了呢？
新建项目是maven的时候，全局工具选择自动安装，运行这个maven项目的时候会自动下载maven

ubuntu20.04手动安装maven
apt update
apt install -y maven 会自动设置全局变量，安装jenkins的时候安装过JDK了
```
root@cn-office-tonytest-jenkins:~# mvn -v
Apache Maven 3.6.3
Maven home: /usr/share/maven
```

### **本地Maven使用私服**
安装和配置好之后，在开发中如何使用呢。可在maven的默认配置settings.xml中修改。
### **发布自己的jar到私服**
如果要发布自己的jar到私服，就需要修改工程的pom.xml，添加如下内容：
```
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>http://192.168.60.231:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshot</name>
        <url>http://192.168.60.231:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
**同时**
拷贝/usr/share/maven/conf/settings.xml到/var/lib/jenkins/.m2/这个目录下，并添加nexus的用户名和密码
```
  <servers>
    <server>
      <id>maven-proxy</id>
      <username>nexus</username>
      <password>test</password>
    </server>
    <server>
      <id>leanwork-thirdparty</id>
      <username>nexus</username>
      <password>test</password>
    </server>
    <server>
      <id>nexus-release</id>
      <username>nexus</username>
      <password>test</password>
    </server>
    <server>
      <id>nexus-snapshot</id>
      <username>nexus</username>
      <password>test</password>
    </server>
  </servers>
```
注意上面的repository的id值一定要跟settings.xml文件中配置的server一致。
# 三、jenkins配置第一个JAVA项目
安装插件Git Parameter
## 第一个构建的项目是java后台项目,选择"构建一个自由风格的软件项目"，比较灵活.

## 1. 常规-->勾选"丢弃旧的项目"。自定义"保持构建的天数"和"保持构建的最大个数"
## 2. 常规-->勾选"参数化构建过程"。

### **添加一个**'字符参数',名称'BRANCH'默认值'origin/master'。**手动填入分支或标签，值传给$BRANCH**
或者
### **添加一个**'git参数'，名称'BRANCH'默认值'origin/master'。**显示分支或标签，默认origin/master，值传给$BRANCH**
一个手动填写，一个直接鼠标选择
**crm-all.git是一个聚合git**，内部包含十几个微服务，除了选择好分支外，构建对应服务还需要填写具体名字，因此需要用到'Extended Choice Parameter'，在插件中心安装'Extended Choice Parameter Plug-In'
### **添加一个**'Extended Choice Parameter'-->名称MODULE_NAME-->Basic Parameter Types-->Parameter Type-->check boxes-->Number of Visible Items-->Delimiter-->Choose Source for Value(这里面的值是给check boxes用的，也就是列出了crm-all里面所有的微服务，发布的时候点击勾选具体微服务名字就行了)

### **添加一个** '布尔值参数'，名称DEPLOY_DEV。效果:勾选就给出true这个值，不勾选就给出false这个值
### **添加一个** '布尔值参数'，名称BUILD_IMAGE。效果:勾选就给出true这个值，不勾选就给出false这个值
## 3. 源码管理-->勾选'Git'. 第一次配置的时候需要授权，参考前面的jenkins安装和配置
填写Repository URL-->git@gitlab.51sw.cc:bw/crm-all.git
填写Branches to build-->$BRANCH(来自git参数或字符参数)
## 4. 构建环境-->勾选Delete workspace before build starts
## 5. 构建
增加构建步骤-->执行shell-->填写

'bash /var/lib/jenkins/workspace/jenkins-deploy/crm-all.sh 1.0.$BUILD_NUMBER crm-all ${DEPLOY_DEV} ${BUILD_IMAGE}'
$BUILD_NUMBER取值是构建历史中第几次构建
${DEPLOY_DEV}前面配置的布尔值的值
${BUILD_IMAGE}前面配置的布尔值的值
crm-all.sh 将git clone代码到本地，用maven编译，打包成jar包，利用Dockerfile构建镜像的这一过程编写shell脚本
