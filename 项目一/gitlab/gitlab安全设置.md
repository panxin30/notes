# 一、gitlab 安全
1今天收到了阿里云异地登录的短信报警，登录阿里云后台发现，有人从深圳登录了我的服务器（本人在北京），查看详细信息一共登录了5次，前两次是使用的git用户进行登录，后两次已经变成了root用户，怀疑是有人通过使用git用户登录到服务器并破解了我的root账户，首先我的阿里云服务器是只支持秘钥登录的，相应的在后续做了如下调整，没有再次发现相关问题。

修改/etc/passwd中git用户的相关权限。
```
gitlab-www:x:997:994::/var/opt/gitlab/nginx:/bin/false
git:x:996:993::/var/opt/gitlab:/bin/git-shell
gitlab-redis:x:995:992::/var/opt/gitlab/redis:/bin/false
gitlab-psql:x:994:991::/var/opt/gitlab/postgresql:/bin/git-shell
gitlab-prometheus:x:993:990::/var/opt/gitlab/prometheus:/bin/git-shell
```
将/bin/bash修改为/bin/git-shell。
使用 Git 自带的 git-shell 工具限制 Git 库所属用户的活动范围。将此设置为git-shell，那么该用户就无法使用普通的 bash 或者 csh 什么的 shell 程序。
如果此时使用git用户想要登录服务器，将会出现如下报错。

二、
参考：https://www.yisu.com/zixun/15648.html

