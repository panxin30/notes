 # 时区
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