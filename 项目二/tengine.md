# Tengine完全兼容Nginx，因此可以参照Nginx的方式来配置Tengine。
# 简介
Tengine是由淘宝网发起的Web服务器项目。它在[Nginx](http://nginx.org/)的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。
# 部署
`wget http://tengine.taobao.org/download/tengine-2.3.2.tar.gz`
`tar zxf tengine-2.3.2.tar.gz`
安装依赖包
```
yum install gcc pcre-devel openssl-devel zlib-devel -y
```
创建nginx用户

`useradd -r -s /sbin/nologin nginx`
编译选项是在ubuntu22.04 apt 直接安装的nginx1.18上查看nginx -V 在此基础上追加了stream模块，会在 prefix=/app/tengine/ 下自动生成modules/ngx_stream_module.so
nginx.conf需要追加`load_module /app/tengine/modules/ngx_stream_module.so;`才能使stream模块生效
根据版本的不同，stream配置可能跟http同级或者包含在http内
```
./configure --user=nginx \
--group=nginx \
--prefix=/app/tengine \
--http-log-path=/var/log/nginx/access.log \
--error-log-path=/var/log/nginx/error.log \
--lock-path=/var/lock/nginx.lock \
--pid-path=/run/nginx.pid \
--with-compat \
--with-debug \
--with-pcre-jit \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_realip_module \
--with-http_auth_request_module \
--with-http_v2_module \
--with-http_dav_module \
--with-http_slice_module \
--with-threads \
--with-http_addition_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_sub_module \
--with-stream \
--with-stream=dynamic \
--with-stream_ssl_module \
--with-stream_realip_module
```
`make && make install`

`ln -s /app/tengine/sbin/nginx /usr/sbin/`

启动tengine服务
```
[root@localhost sbin]# cat /etc/systemd/system/nginx.service 
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStart=/app/tengine/sbin/nginx -c /app/tengine/conf/nginx.conf
ExecReload=/usr/bin/kill -s HUP $MAINPID
ExecStop=/usr/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```
