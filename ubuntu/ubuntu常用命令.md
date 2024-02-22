# 查看已经安装了的软件 
`sudo apt list --installed | grep mongo`
# 修改ubuntu时区
```
timedatectl
cat /etc/timezone 
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai
cat /etc/timezone 
timedatectl
```
本地时间                      Local time: Tue 2020-01-07 10:48:05 CST
世界时间                  Universal time: Tue 2020-01-07 02:48:05 UTC
 硬件时钟                       RTC time: Tue 2020-01-07 02:48:06
# 修改ubuntu时间
```
需要取消自动从互联网同步时间才可以的

timedatectl set-ntp 0

上面的命令可以关闭自动同步，然后你再设置就好了

如果又要打开可以运行

timedatectl set-ntp 1
```
# ubuntu18.04修改家目录
新建用户worker，家目录/home/worker
直接修改/etc/passwd 的worker改家目录为/data/logstash/logs

或者
useradd -d /data/test -s /bin/bash worker
# ubuntu随机强密码
```
1.  `#假设你想要生成 5 个 14 字符长的密码，方法如下：`
2.  `$ pwgen -s 14  5`
3.  `7YxUwDyfxGVTYD em2NT6FceXjPfT u8jlrljbrclcTi IruIX3Xu0TFXRr X8M9cB6wKNot1e`
4.  `#如果你真的想要生成 20 个超强随机密码，方法如下：`

6.  `$ pwgen -cnys 14  20`
```
# 修改服务器名称
hostnamectl set-hostname 名字
# 禁用IPV6
```
编辑/etc/default/grub，修改GRUB_CMDLINE_LINUX_DEFAULT和GRUB_CMDLINE_LINUX以在启动时禁用IPv6：

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash ipv6.disable=1"

GRUB_CMDLINE_LINUX="ipv6.disable=1"

保存文件并运行：

sudo update-grub
```
# ssh长连接
在服务端

编辑服务器 /etc/ssh/sshd_config，最后增加
```
ClientAliveInterval 60
ClientAliveCountMax 1
```
这 样，SSH Server 每 60 秒就会自动发送一个信号给 Client，而等待 Client 回应
在客户端
编辑/etc/ssh/ssh_confi
```
    TCPKeepAlive yes
    ServerAliveinterval 30
```