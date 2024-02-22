# 穿透服务器，内部通信
网站前端静态资源放在国内阿里云OSS存储桶，这是他的

外网地址`test-fs.oss-cn-hangzhou.aliyuncs.com`
内网地址`test-fs.oss-cn-hangzhou-internal.aliyuncs.com`
用tinc打通阿里杭州和阿里香港，访问OSS走内网，国外访问加速.之前是开通了一条杭州到香港的高速通道
# 上传图片逻辑整理
微服务-->申请连接阿里云OSS，test-fs.oss-cn-hangzhou.aliyuncs.com-->生产集群内部dnsmasq写死了这个域名地址172.18.164.164-->负载均衡到cn-hk-crm-k8s-proxy01，2的80/443端口
-->cn-hk-crm-k8s-proxy01，2的80/443端口是sniproxy在使用-->sniproxy访问test-fs.oss-cn-hangzhou.aliyuncs.com

微服务调用-->未知域名:8000-->负载均衡到cn-hk-crm-k8s-proxy01，2的8000端口-->cn-hk-crm-k8s-proxy01，2的nginx转发到-->ali-hz-crm-proxy-api01,2的80端口-->访问leanwork-fs.oss-cn-hangzhou-internal.aliyuncs.com:80
# 服务器端
```
root@ali-hz-brokerwork-proxy-api01:/etc/tinc/opn# cat tinc-up
#!/bin/bash
ifconfig $INTERFACE 10.9.0.1 netmask 255.255.255.0
route add -host 172.17.128.202 gw 10.9.0.13
root@ali-hz-brokerwork-proxy-api01:/etc/tinc/opn# cat tinc-down
#!/bin/bash
ifconfig $INTERFACE down
root@ali-hz-brokerwork-proxy-api01:/etc/tinc/opn# cat tinc.conf
Name=core
Interface=opn
Mode=switch
Port=16384
TCPOnly=yes
PrivateKeyFile=/etc/tinc/opn/rsa_key.priv
root@ali-hz-brokerwork-proxy-api01:/etc/tinc/opn# cat hosts/core
Compression=9
Subnet=10.9.0.1/32
Address=外网ip
Port=16384

-----BEGIN RSA PUBLIC KEY-----
省略......
-----END RSA PUBLIC KEY-----
root@ali-hz-brokerwork-proxy-api01:/etc/tinc/opn# cat hosts/crmproxy01
Compression=9
Subnet=10.9.0.14/32

-----BEGIN RSA PUBLIC KEY-----
省略......
-----END RSA PUBLIC KEY-----
```
# 服务器端nginx的应用
```
root@ali-hz-brokerwork-proxy-api01:~# cat /etc/nginx/tcp.d/api.conf
stream {
    limit_conn_zone $binary_remote_addr zone=addr:50m;

    log_format proxy '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/stream-access.log proxy;
    server {
        listen 9101;
        proxy_pass api01;
    }
    upstream api01 {
        server 10.9.0.2:9100 weight=2 max_fails=1 fail_timeout=60s;
    }
    server {
        listen 9102;
        proxy_pass api02;
    }
    upstream api02 {
        server 10.9.0.3:9100 weight=2 max_fails=1 fail_timeout=60s;
    }
   map $server_port $tcp_cname {
        80 "test-fs.oss-cn-hangzhou-internal.aliyuncs.com:80";
        443 "test-fs.oss-cn-hangzhou-internal.aliyuncs.com:443";
   }
   server {
        resolver 100.100.2.136 100.100.2.138 valid=60s ipv6=off;
        listen 0.0.0.0:80;
        listen 0.0.0.0:443;

        proxy_pass $tcp_cname;
# 监听80/443端口，如果是访问80端口，根据上面的map $server_port取得值'80'，则proxy_pass $tcp_name的值就选第一项
# "test-fs.oss-cn-hangzhou-internal.aliyuncs.com:80"
# 访问443端口则选第二项 "test-fs.oss-cn-hangzhou-internal.aliyuncs.com:443"
   }
}
```
# 客户端
```
root@cn-hk-crm-k8s-proxy01:/etc/tinc/opn# cat tinc-up
#!/bin/bash
ifconfig $INTERFACE 10.9.0.14 netmask 255.255.255.0
root@cn-hk-crm-k8s-proxy01:/etc/tinc/opn# cat tinc-down
#!/bin/bash
ifconfig $INTERFACE down
root@cn-hk-crm-k8s-proxy01:/etc/tinc/opn# cat tinc.conf
Name=crmproxy01
ConnectTo=core
Interfce=opn
Mode=switch
Port=11900
PrivateKeyFile=/etc/tinc/opn/rsa_key.priv
root@cn-hk-crm-k8s-proxy01:/etc/tinc/opn# cat hosts/core
Compression=9
Subnet=10.9.0.1/32
Address=外网ip
Port=16384

-----BEGIN RSA PUBLIC KEY-----
省略......
-----END RSA PUBLIC KEY-----
root@cn-hk-crm-k8s-proxy01:/etc/tinc/opn# cat hosts/crmproxy01
Compression=9
Subnet=10.9.0.14/32

-----BEGIN RSA PUBLIC KEY-----
省略......
-----END RSA PUBLIC KEY-----
#如果有什么路径不通的，可以查下路由表
```
# 客户端nginx应用
```
root@cn-hk-crm-k8s-proxy01:~# cat /etc/nginx/tcp.d/stream.conf
stream {
    resolver 8.8.8.8 8.8.4.4 1.1.1.1 valid=300s;
    resolver_timeout 30s;
    log_format proxy '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';
    access_log /var/log/nginx/access-stream.log proxy;

    server {
        listen 8000;
        proxy_pass http-lfs;
    }
    upstream http-lfs {
        server 10.9.0.1:80 max_fails=2 weight=100 fail_timeout=60s;
        server 10.7.0.1:80 max_fails=2 weight=100 fail_timeout=60s;
        server 192.168.1.240:80 max_fails=2 weight=3 fail_timeout=60s; #ali-hz-brokerwork-proxy-api01内网IP
        server 192.168.1.242:80 max_fails=2 weight=3 fail_timeout=60s;
    }

    server {
        listen 4443;
        proxy_pass https-lfs;
    }
    upstream https-lfs {
        server 10.9.0.1:443 max_fails=2 weight=100 fail_timeout=60s;
        server 10.7.0.1:443 max_fails=2 weight=100 fail_timeout=60s;
        server 192.168.1.240:443 max_fails=2 weight=3 fail_timeout=60s;#ali-hz-brokerwork-proxy-api01内网IP
        server 192.168.1.242:443 max_fails=2 weight=3 fail_timeout=60s;
    }
}
```