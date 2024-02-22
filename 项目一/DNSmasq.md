参考：https://cloud.tencent.com/developer/article/1174717

# DNSmasq配置文件说明
```
port=53
 # 指定DNSmasq的监听端口，默认为53端口，也可设置为5353端口从而防止53端口DNS污染（但某些设备如win并不支持非53端口）

resolv-file=/xxx/xx.conf
 # 指定DNSmasq获取上游DNS服务器地址的文件，不配置此项则默认从 /etc/resolv.conf(linux默认DNS配置文件) 获取
 
strict-order
 # 严格按照 resolv-file 参数指定的文件中按从上到下的顺序发送DNS解析请求，直至获取解析应答成功为止
 
listen-address=
 # 指定DNSmasq监听的地址，若仅提供为本机使用可设置为 127.0.0.1 ，留空或设置为 0.0.0.0 即任何人都可访问
 
address=/xxx.xx/x.x.x.x
 # 自定义某些地址的解析服务器，可以 过滤或者指定 某些网站（支持ipv6，直接写ipv6的地址即可）
 # 过滤广告或者某域名，例如配置 address=/www.nanqinlang.com/127.0.0.1
 # 把广告域名解析请求发送到错误的解析服务器IP 127.0.0.1 即可屏蔽该域名的访问
 # 也可以把 www.google.com 等指向一个国外的SNI代理IP，即可实现DNS科学上网
 
server=208.67.222.222#5353
 # 指定上游DNS解析服务器,此处推荐设置为可使用5353端口的opendns
```
# 生产服务器上的配置
root@cn-hk-crm-k8s-proxy01:~# cat /etc/dnsmasq.conf
```
log-queries
log-facility=/var/log/dnsmasq.log
no-resolv
# 不读取 resolv-file 来确定上游服务器
no-hosts
# 不加载本地的 /etc/hosts 文件
interface=eth0
server=/mysql.rds.aliyuncs.com/100.100.2.136
server=/redis.rds.aliyuncs.com/100.100.2.136
server=/mongodb.rds.aliyuncs.com/100.100.2.136
server=/registry-hk-tools.test.com/100.100.2.136
server=/pay.test.com/100.100.2.136
server=/kafka-bw-hk-prod.public.test.com/100.100.2.136
#指定域名或泛域名使用指定DNS
#server=1.1.1.1
strict-order
# 严格按照resolv.conf中的顺序进行查找
address=/ntp.ubuntu.com/172.25.0.36
address=/cn-hk-crm-k8s-slave01.test.com/172.17.128.208
address=/cn-hk-crm-k8s-slave02.test.com/172.18.164.156
address=/cn-hk-crm-k8s-slave03.test.com/172.18.164.158
address=/cn-hk-crm-k8s-slave04.test.com/172.17.128.210
address=/test-fs.oss-cn-hangzhou.aliyuncs.com/172.18.164.164
address=/#/172.25.0.36
# 指定域名解析到特定的IP上
```
# 管理控制内网DNS
将局域网中的所有的设备的本地DNS设置为已经安装Dnsmasq的服务器IP地址。
局域网中所有用户访问*.mysql.rds.aliyuncs.com将会使用DNS100.100.2.136来解析
局域网中所有用户访问test-fs.oss-cn-hangzhou.aliyuncs.com域名解析指向172.18.164.164
address=/#/172.25.0.36，其他域名解析将指向172.25.0.36

或者使用sniproxy
--------------------------------------------------------------------------------------------

