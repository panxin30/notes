在RHEL7之前，服务管理是分布式的被SysV init或UpStart通过/etc/rc.d/init.d下的脚本管理。这些脚本是经典的Bash脚本，允许管理员控制服务的状态。在RHEL7中，这些脚本被服务单元文件替换。
在systemd中，服务、挂载等资源统一被称为单元，所以systemd中有许多单元类型，服务单元文件的扩展名是.service，同脚本的功能相似。例如有查看、启动、停止、重启、启用或者禁止服务的参数。
systemd单元文件放置位置：
/usr/lib/systemd/system/默认单元文件安装目录
/run/systemd/system单元运行时创建，这个目录优先于安装目录
/etc/systemd/system系统管理员创建和管理的单元目录，优先级最高。
`systemctl list-unit-files | grep enable` 查看开机启动项
```
启动服务：systemctl start vsftpd.service
关闭服务：systemctl stop vsftpd.service
重启服务：systemctl restart vsftpd.service
显示服务的状态：systemctl status vsftpd.service
在开机时启用服务：systemctl enable vsftpd.service
在开机时禁用服务：systemctl disable vsftpd.service
查看服务是否开机启动：systemctl is-enabled vsftpd.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed
```
2.systemd 服务管理
```
systemctl常用命令：
启动服务 systemctl start name.service
关闭服务 systemctl stop name.service
重启服务 systemctl restar tname.service
仅当服务运行的时候，重启服务 systemctl try-restart name.service
重新加载服务配置文件 systemctl relaod name.service
检查服务运作状态 systemctl status name.service 或者 systemctl is-active name.service
展示所有服务状态详细信息 systemctl list-units--type service --all
允许服务开机启动 systemctl enable name.service
禁止服务开机启动 systemclt disable name.service
检查服务开机启动状态 systemctl status name.service 或者systemctl is-enabled name.service
列出所有服务并且检查是否开机启动 systemctl list-unit-files --type service3.服务详细信息查看
使用如下命令列出服务：
systemctl list-units --type service
默认只列出处于激活状态的服务，如果希望看到所有的服务，使用--all或-a参数：
systemctl list-units--type service --all
有时候希望看到所以可以设置开机启动的服务，使用如下命令：
systemctl list-unit-files --type service
查看服务详细信息，使用如下命令：
systemctl status name.service
服务信息关键词解释
Loaded服务已经被加载，显示单元文件绝对路径，标志单元文件可用。
Active服务已经被运行，并且有启动时间信息。
Main PID与进程名字一致的PID，主进程PID。
Status服务的附件信息。
Process相关进程的附件信息。
CGroup进程的CGroup信息。
```
CentOS 7设置开机启动服务
```
vim /etc/systemd/system/bw-custom.service
[Unit]
Description=bw-custom
After=supervisord.service

[Service]
WorkingDirectory=/home/brokerwork/bw-custom
ExecStart=/usr/bin/java -server -Xms1024M -Xmx1024M -XX:+UseNUMA -XX:+UseParallelGC -XX:NewRatio=1 -XX:MaxDirectMemorySize=512M -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=dump/ -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -verbosegc -Xloggc:dump/gc.log -Dlogging.config=classpath:logback-deploy.xml -jar bw-custom.jar --spring.profiles.active=preprod

SuccessExitStatus=143

User=brokerwork
Group=brokerwork

[Install]
WantedBy=multi-user.target
```
##  systemctl enable name.service
