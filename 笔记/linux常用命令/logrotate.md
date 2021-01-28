
参考：https://www.cnblogs.com/kevingrace/p/6307298.html
# **调试**
手动强制切割日志，需要加-f参数；不过正式执行前最好通过Debug选项来验证一下（-d参数），这对调试也很重要
### /usr/sbin/logrotate -f /etc/logrotate.d/nginx
### /usr/sbin/logrotate -d -f /etc/logrotate.d/nginx

logrotate命令格式：
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。

**根据日志切割设置进行操作，并显示详细信息**
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -v /etc/logrotate.conf
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -v /etc/logrotate.d/php

**根据日志切割设置进行执行，并显示详细信息,但是不进行具体操作，debug模式**
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -d /etc/logrotate.conf
[root@huanqiu_web1 ~]# /usr/sbin/logrotate -d /etc/logrotate.d/nginx

# **logrotate参数说明**

compress 通过gzip 压缩转储以后的日志  
nocompress 不做gzip压缩处理  
create mode owner group 轮转时指定创建新文件的属性，如create 0777 nobody nobody  
nocreate 不建立新的日志文件  
delaycompress 和compress 一起使用时，转储的日志文件到下一次转储时才压缩  
nodelaycompress 覆盖 delaycompress 选项，转储同时压缩。  
missingok 如果日志丢失，不报错继续滚动下一个日志  
ifempty 即使日志文件为空文件也做轮转，这个是logrotate的缺省选项。  
notifempty 当日志文件为空时，不进行轮转  
mail address 把转储的日志文件发送到指定的E-mail 地址  
olddir directory 转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统  
noolddir 转储后的日志文件和当前日志文件放在同一个目录下  
sharedscripts 运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本  
prerotate 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行  
postrotate 在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行  
daily 指定转储周期为每天  
weekly 指定转储周期为每周  
monthly 指定转储周期为每月  
rotate count 指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份  
dateext 使用当期日期作为命名格式  
dateformat .%s 配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数

# **logrotate配置文件位置**

Linux系统默认安装logrotate工具，它默认的配置文件在：  
/etc/logrotate.conf  
/etc/logrotate.d/

# **定时轮循机制**

**Logrotate是基于CRON来运行的，其脚本是/etc/cron.daily/logrotate，日志轮转是系统自动完成的。**

  
logrotate这个任务默认放在cron的每日定时任务cron.daily下面 /etc/cron.daily/logrotate  
/etc/目录下面还有cron.weekly/, cron.hourly/, cron.monthly/ 的目录都是可以放定时任务的

定义好了每日执行任务的脚本cron.daily/logrotate ，再查看crontab的内容，里面设置好了对应的cron.xxly执行时间