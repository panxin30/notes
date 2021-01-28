Nginx WEB服务器定期更新，如果需要将低版本升级或者将高版本降级，升级或者降级方法如下，分为四个步骤，包括软件下载、预编译、编译、配置(没有install)，具体方法如下
获取旧版本nginx的configure选项
下载新版本源码包
编译新版本NGINX && make
备份旧版本的nginx可执行文件，复制新版本的nginx执行文件到/usr/local/nginx/sbin目录下
**cp objs/nginx /usr/local/nginx/sbin/**
测试新版本nginx是否正常
[root@localhost nginx-1.13.5]# /usr/local/nginx/sbin/nginx –t
平滑重启升级Nginx
[root@localhost nginx-1.13.5]# /usr/local/nginx/sbin/nginx -s reload
验证nginx是否升级成功