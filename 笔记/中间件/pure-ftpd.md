yum -y install pure-ftpd服务端
yum -y install lftp         客户端
pure-ftpd.conf  pureftpd-ldap.conf  pureftpd-mysql.conf  pureftpd.passwd  pureftpd.pdb  pureftpd-pgsql.conf配置文件

，/etc/pure-ftpd/下存在一个默认的pure-ftpd.conf，每一行都有详尽的注释，有时间可以阅读一下。

*****
权限与用户

首先我们需要新建一个用户，此用户作为pure-ftpd虚拟用户的依托用户，此用户的home directory也将作为FTP root,当然你新建目录的时候也可以不指定home directory，并在建立完毕以后将FTP root chown给此用户。

*****
groupadd ftpgroup        //新建系统组
useradd -g ftpgroup -M -s /sbin/nologin ftpadmin //新建一个FTP用户，不创建用户目录，假定已经存在一个FTP root作为FTP的根目录。
chown –R ftpuser:ftpgroup ftproot/ //把FTP目录的所属用户和组改为虚拟用户所依托的系统用户和组
pure-pw useradd puser –u ftpuser –d /ftproot –m //命令格式很好懂，pure-pw 命令使用useradd 需要添加的用户名， -u标明虚拟用户并且与系统用户权限关联，-d指定了系统用户Home目录中的子目录，且被限制在这个子目录里面（此处是否被限制与上文的conf文件设定相关）如果需要访问系统用户的HOME Directory，则直接使用参数 -D，-m则写入用户数据库，用户设置即时生效，无需重启进程。
pure-config.pl etc/pure-ftpd/pure-ftpd.conf &//后台启动进程。

*****

此处，我们新建了一个FTPadmin用户，此用户对FTP ROOT拥有所有权限，如果需要一个只读用户，需要新建一个在ftpgroup内的用户，然后再使用pure-pw创建一个依托此用户的虚拟用户即可。即pure-pw虚拟用户对FTP目录的操作权限与其依托的系统用户对FTP目录的权限保持一致。

*****
因此可以根据自己需要来自行选择软件；
　　配置用户少安全性高的服务器可以选用 VSFTPD
　　配置功能复杂的服务器就可以选用 Pure-FTPD
