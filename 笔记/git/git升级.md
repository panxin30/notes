centos6升级git
```
1. 安装开发环境

　　yum install -y curl curl-devel zlib-devel openssl-devel perl cpio expat-devel gettext-devel

2. 卸载旧版本git

　　yum remove git -y

3. 编译安装新版本git

　　3.1 编译并安装
　　　　git clone https://github.com/git/git.git ~/git
　　　　cd ~/git
　　　　autoconf
　　　　./configure
　　　　make -j4
　　　　su root
　　　　make install
　　　　ln -sf /usr/local/bin/git /usr/bin/git
　　3.2 配置

　　　　echo "export GIT_SSL_NO_VERIFY=1" > ~/.bash_profile (如果不加入这句会出现fatal Peer certificate cannot be authenticated with known CA certificate)
```
### **二进制版本升级**
centos6x
```
#使用yum安装的版本，先进行卸载
yum remove git gettext-devel -y
#安装依赖
yum install curl curl-devel zlib-devel openssl-devel perl cpio expat-devel gettext-devel autoconf expat-devel perl-devel
#下载源码包，编译安装
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.24.0.tar.gz
tar -zxvf git-2.24.0.tar.gz
cd git-2.24.0
make configure
./configure --prefix=/usr/local/git
make && make install 
#添加软连接
ln -s /usr/local/git/bin/* /usr/bin/

git --version
```
### **yum 安装** You can use WANDisco's CentOS repository to install Git 2.x: for CentOS 6, for CentOS 7
```
Install WANDisco repo package:

yum install http://opensource.wandisco.com/centos/6/git/x86_64/wandisco-git-release-6-1.noarch.rpm
- or -
yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-1.noarch.rpm
- or -
yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm
Install the latest version of Git 2.x:

yum install git
Verify the version of Git that was installed:

git --version
```