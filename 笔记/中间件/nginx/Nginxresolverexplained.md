内部arm海外无法访问，国内是正常的
ping crm.lwork.com指向了香港nginx
该nginx没有错误日志，access日志连接一个ip+端口超时，发现是反代到阿里云oss存储桶的，oss配置的是域名，域名就是使用的ELB，ELB后面的ip变动，内部crm的nginx没有配置resolver，不会自动更新
# Nginx resolver explained
Nginx resolver is playing very important part in creating fault tolerant setups, especially when it comes to the free open source version. In this article I’ll explain why we need Nginx resolver and how it works.

I remember the moment about a year or so ago when I came to the office and found people screaming that one of our sites was hacked. When I opened it in my browser I saw completely different site, definitely not ours..

After spending hours trying to figure out the problem, I was finally able to identify the root cause. It wasn’t a hack attack after all, it was something much more simple.. Turned out AWS load balancer changed IP and Nginx cached the old one (we used static reverse proxy configuration).

Let me remind how we usually configure Nginx as reverse proxy:
```
location / {
    proxy_pass http://service-999999.eu-west-2.elb.amazonaws.com;
} 
```
This super simple configuration will work just fine until one day / week / month later it will stop. Yes it will stop when AWS changes IP behind ELB DNS record.

But what exactly happens? Turns out when you specify hostname for proxy_pass like we did in our example, Nginx will resolve it once on startup or reload and then cache resulting IP address. For us Nginx was sending requests to the IP that was re-assigned from ELB to someone else EC2 instance. Fun right?

# What can we do to avoid the problem?

## Well, there are multiple ways:

1. Initially we wrote a small script that would monitor DNS pointer for the ELB and if it changed, we would reload Nginx. That worked, but we had to run additional piece in our infrastructure which we didn’t like.

2. Then there is Nginx Plus which provides special resolve flag that you can set on your upstream servers. It will honor DNS TTL and update them automatically if the pointer changes (as per official documentation). I’ve yet to try it. The problem with Nginx Plus that it costs 2.5K / per instance / per year..

3. There is a module on GitHub called nginx-upstream-dynamic-servers, but it doesn’t have recent updates at the time of writing.. (but do have opened Issues). Please let me know in the comments below if you use it.

4. Some time ago I stumbled upon an interesting free Nginx resolver alternative which I’ll describe below.

# Free Nginx resolver alternative
```
server {
  listen        80;
  server_name   example.com;

  location / {

    resolver 172.16.0.23;

    set $upstream_endpoint http://service-999999.eu-west-2.elb.amazonaws.com;

    proxy_pass $upstream_endpoint$request_uri;
  }
}
```
So here we use our famous Nginx resolver directive (172.16.0.23 is AWS default resolver, you can use Google 8.8.8.8, or your own).
When proxy_pass command is getting $variable instead of URI, it uses DNS resolver in case cache entry for the IP has expired. This little handy config secret is exactly what we need!

# URI Caveat when using $variable in proxy_pass with trailing slash
Normally in Nginx if we put trailing slash / at the end of the static proxy_pass directive, it will remove the part matched by the location and only proxy remaining part of the URI:
```
location /test/ {
    proxy_pass http://127.0.0.1:80/;
}
```
If we get /test/path, only /path will be sent down.

When we use $variable for the proxy_pass with trailing slash, **only / will be proxied!**
```
resolver 172.16.0.23;
set $upstream_endpoint http://service-999999.eu-west-2.elb.amazonaws.com/; # Trailing slash
location /test/ {
  proxy_pass $upstream_endpoint;
}
```
Now request to /test/path will proxy only / which is not what we would expect.

If we really need to preserve trailing slash functionality, we need to remove trailing slash from the $upstrem_endpoint and use rewrite inside of the location:
```
resolver 172.16.0.23;
set $upstream_endpoint http://service-999999.eu-west-2.elb.amazonaws.com; # No trailing slash
location /test/ {
  rewrite ^/test/(.*) /$1 break;
  proxy_pass $upstream_endpoint;
}
```
Finally our /test/path request will be converted to desired /path.

This post turned out a bit more technical than I expected, but it’s well worth spending extra bit of time to understand all the examples above. If you plan to use Nginx in the environment with changing ips behind DNS pointers (which is almost everywhere), then you need to either get Nginx Plus, or learn Nginx resolver secrets.



