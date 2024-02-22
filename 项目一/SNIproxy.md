# SNIproxy
参考：http://iii80.com/?action=show&id=855

# 原理简单说明

假如你有一台 海外的服务器 IP为：`233.233.233.233`，上面搭建了 SNI Proxy，并且配置正常并启动。

然后你本地Hosts文件在最后添加一条：

~~~
233.233.233.233 www.google.com
~~~

保存Hosts文件并打开浏览器访问` https://www.google.com `，然后你就会发现你可以进入` https://www.google.com `网站了。

# 原理解析：

```
Hosts设置 233.233.233.233 www.google.com 后，浏览器访问 https://www.google.com  
=>> 浏览器搜索Hosts文件发现设置的解析IP(233.233.233.233) 
=>> 浏览器访问 SNI Proxy(233.233.233.233) 
=>> SNI Proxy收到信息然后去访问 https://www.google.com 并获取网站数据，然后把网站数据原封不动的返回给你 =>> 浏览器收到 SNI Proxy返回的 网站数据并显示出来 
=>> 你看到了 https://www.google.com 网页
```

简单的来说，SNI Proxy 会把请求的网站比如`https://www.google.com`获取并原封不动的返回请求者，不需要对证书进行解密和加密，所以不需要配置证书。

SNI Proxy 可以简单的实现这样的 反向代理功能。
# 配置
```
root@cn-hk-crm-k8s-proxy01:~# cat /etc/sniproxy.conf
user nobody
group nogroup
pidfile /var/run/sniproxy.pid
resolver {
    nameserver 100.100.2.138
    mode ipv4_only
}
error_log {
    syslog daemon
}
listen 0.0.0.0 443 {
    proto tls
    table https_hosts
    access_log {
        filename /tmp/sniproxy-https-access.log
    }
}
table https_hosts {
    .*\.oss-cn-hangzhou\.aliyuncs\.com$ *:443
    mirrors\.cloud\.aliyuncs\.com$ *:443
#    .*   *:443
}
```
