
ubuntu 18. 04 apt 直接安装的mysql
```
# MySQL systemd service file

[Unit]
Description=MySQL Community Server
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=forking
User=mysql
Group=mysql
PIDFile=/run/mysqld/mysqld.pid
PermissionsStartOnly=true
ExecStartPre=/usr/share/mysql/mysql-systemd-start pre
ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid
TimeoutSec=infinity
Restart=on-failure
RuntimeDirectory=mysqld
RuntimeDirectoryMode=755
LimitNOFILE=5000

```