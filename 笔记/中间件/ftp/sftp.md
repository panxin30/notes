官网使用nginx+php+wordpress ,使用的用户`apache`目录/home/test,目前需要使用sftp在更新代码
`drwxrwxrwx 13 apache apache  4096 10月 14 14:34 test`
ssh版本`OpenSSH_5.3p1, OpenSSL 1.0.1e-fips 11 Feb 2013`
1. 安装sftp
`yum install openssh-server openssl`
2. 配置sshd_config
```
#      Subsystem      sftp    /usr/libexec/openssh/sftp-server #注销使用sftp-server
        Subsystem       sftp    internal-sftp
        Match Group apache #告诉sshd 进程，apache 用户组中的用户适用下面的限制
        ChrootDirectory /home #限制sftp的活动目录在
        ForceCommand    internal-sftp
        AllowTcpForwarding no # 禁止tcp转发
        X11Forwarding no # 禁止X11转发
```
`service sshd restart`重启
