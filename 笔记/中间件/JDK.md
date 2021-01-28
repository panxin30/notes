### **ubuntu 16.04 install jdk8**
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```
### **ubuntu 18.04 install jdk8**
`sudo apt install openjdk-8-jdk`

```
tar -xzf  jdk-8u71-linux-x64.tar.gz
mkdir -p /usr/local/java/
mv jdk1.8.0_71/ /usr/local/java/jdk1.8 下.       
#然后配置环境变量，这样可以任何地方引用jdk，如下配置：       
#vi /etc/profile 最后面加入以下语句：       
       export JAVA_HOME=/usr/local/java/jdk1.8   #jdk存放的位置
       export JRE_HOME=${JAVA_HOME}/jre 
       export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
       export PATH=${JAVA_HOME}/bin:$PATH      
#source /etc/profile   #使环境变量马上生效       
#java -version    #查看java版本，看到jdk1.8.0_25版本即代表java jdk安装成功。
```
## windows下安装JDK
系统变量--->新建 JAVA_HOME变量，变量值填写jdk安装路径，
系统变量--->找到 Path 变量，在变量值最后输入;%JAVA_HOME%\bin;（注意在最前面有个;号）
系统变量--->新建 CLASSPATH 变量，变量值输入 .;%JAVA_HOME%\jre\lib\rt.jar;.;(注意最前面有一点，最后面有;.;号)
测试环境变量是否配置成功。运行 cmd--->输入 java -version,出现如图所示，说明安装配置成功。也可以通过输入java ，javac等命令来测试是否成功

## 脚本
```
#!/bin/bash
yum -y install tar gcc gcc-c++
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u71-b15/jdk-8u71-linux-x64.tar.gz
tar -zxf jdk-8u71-linux-x64.tar.gz
sleep 3
mkdir -p /usr/local/java/
mv jdk1.8.0_71/ /usr/local/java/jdk1.8

echo 'export JAVA_HOME=/usr/local/java/jdk1.8' >>/etc/profile
echo 'export JRE_HOME=${JAVA_HOME}/jre' >>/etc/profile
echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib' >>/etc/profile
echo 'export PATH=${JAVA_HOME}/bin:$PATH' >>/etc/profile
sleep 2
. /etc/profile
#source ./jdkInstall.sh  在脚本外执行该命令，能实现刷新/etc/profile
```