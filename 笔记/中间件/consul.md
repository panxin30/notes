
```
# Consul Join
The`join`command tells a Consul agent to join an existing cluster. A new Consul agent must join with at least one existing member of a cluster in order to join an existing cluster. After joining that one member, the gossip layer takes over, propagating the updated membership state across the cluster.
```
# **consul集群搭建**
首先去官网现在合适的consul包：https://www.consul.io/downloads.html
```
unzip consul_1.2.3_linux_amd64.zip
cp -a consul  /usr/bin
consul
```
# **启动**
consul必须启动agent才能使用，有两种启动模式server和client，还有一个官方自带的ui。server用与持久化服务信息，集群官方建议3或5个节点。client只用与于server交互。ui可以查看集群情况的。

#### **server**：
```
cn1：

#consul agent  -bootstrap-expect 2  -server   -data-dir /data/consul0 -node=cn1 -bind=192.168.1.202 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1 

cn2:

#consul agent    -server  -data-dir /data/consul0 -node=cn2 -bind=192.168.1.201 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -join 192.168.1.202

cn3:

#consul agent  -server  -data-dir /data/consul0 -node=cn3 -bind=192.168.1.200 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -join 192.168.1.202
```
参数解释：

-bootstrap-expect:集群期望的节点数，只有节点数量达到这个值才会选举leader。

-server： 运行在server模式

-data-dir：指定数据目录，其他的节点对于这个目录必须有读的权限

-node：指定节点的名称

-bind：为该节点绑定一个地址

-config-dir：指定配置文件，定义服务的，默认所有一.json结尾的文件都会读

-enable-script-checks=true：设置检查服务为可用

-datacenter: 数据中心没名称，

-join：加入到已有的集群中

#### **client**：

#consul agent   -data-dir /data/consul0 -node=cn4 -bind=192.168.1.199 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -join 192.168.1.202

client节点可以有多个，自己根据服务指定即可。
```
ui:

#consul agent  -ui  -data-dir /data/consul0 -node=cn4 -bind=192.168.1.198  -client 192.168.1.198   -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -join 192.168.1.202
```
 -ui:使用自带的ui，

-ui-dir：指定ui的目录，使用自己定义的ui

-client：指定web  ui、的监听地址，默认127.0.0.1只能本机访问。

集群创建完成后：

使用一些常用的命令检查集群的状态：

#### **consul  info** 
可以在raft：stat看到此节点的状态是Fllower或者leader
**consul members**
Node Address Status Type Build Protocol DC Segment
cn1 192.168.1.202:8301 alive server 1.0.2 2 dc1 <all>
cn2 192.168.1.201:8301 alive server 1.0.2 2 dc1 <all>
cn3 192.168.1.200:8301 alive client 1.0.2 2 dc1 <default>

新加入一个节点有几种方式；

1、这种方式，重启后不会自动加入集群
#consul  join  192.168.1.202
2、#在启动的时候使用-join指定一个集群
#consul agent  -ui  -data-dir /data/consul0 -node=cn4 -bind=192.168.1.198 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -join 192.168.1.202
3、使用-startjoin或-rejoin
#consul agent  -ui  -data-dir /data/consul0 -node=cn4 -bind=192.168.1.198 -config-dir /etc/consul.d -enable-script-checks=true  -datacenter=dc1  -rejoin
客户端开机自启
使用supervisor
```
[program:consul-agent]
directory=/data/consul
command=/usr/bin/consul agent -config-dir /data/consul/
stdout_logfile=/data/consul/consul-stdout.log
stderr_logfile=/data/consul/consul-error.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
autostart=true
autorestart=true
```
cat /data/consul/config.json
```
{
"bind_addr": "172.16.179.126",
"client_addr": "0.0.0.0",
"datacenter": "feedwork",
"data_dir": "/data/consul",
"node_name": "ali-hk-bwpre-msc-1",
"rejoin_after_leave": true,
"retry_interval": "30s",
"retry_join": [
  "pre-cs-01.tools.lwork.com",
  "pre-cs-02.tools.lwork.com",
  "pre-cs-03.tools.lwork.com"
  ]
 }
```
访问ui：
http://192.168.1.198:8500/ui
端口：
8300：consul agent服务relplaction、rpc（client-server）
8301：lan gossip
8302：wan gossip
8500：http api端口
8600：DNS服务端口
# windows版consul客户端加入集群
```
consul client模式：
1）client.bat
consul agent -data-dir ./ -node=ali-hk-bw-report -bind=10.28.184.194 -dc=feedwork
pause
2）client-join.bat
consul.exe join 10.26.6.170
pause

consul server模式：
1)server.bat
```

## **手动注册服务**
可以在客户端或server端注册
```
curl -X PUT -d '{"ID": "bw-realtime-commission-1","Name": "bw-realtime-commission","Address": "192.168.60.90","Port": 30008,"EnableTagOverride": false,"Check": {"HTTP": "http://192.168.60.90:30008/health","Interval": "10s"}}' http://127.0.0.1:8500/v1/agent/service/register
curl -X PUT -d '{"ID": "bw-account-api-1","Name": "bw-account-api","Address": "192.168.60.90","Port": 30707,"EnableTagOverride": false,"Check": {"HTTP": "http://192.168.60.90:30707/ping","Interval": "10s"}}' http://127.0.0.1:8500/v1/agent/service/register
curl -X PUT -d '{"ID": "bw-bridge-deploy-1","Name": "bw-bridge-deploy","Address": "192.168.60.90","Port": 30660,"EnableTagOverride": false,"Check": {"HTTP": "http://192.168.60.90:30660/ping","Interval": "10s"}}' http://127.0.0.1:8500/v1/agent/service/register
```
## **删除consul集群中无用的微服务**
（service-id需在consul集群中查找，例如open-platform-prod-10188）
`curl -s http://localhost:8500/v1/agent/service/deregister/service-id` 
`curl -s http://localhost:8500/v1/agent/service/deregister/bw-custom-10220-1967695389`
## **删除consul集群中无用的node节点**
（ali-hk-fd-otss为node name）  
`curl http://consul.tools.lwork.com/v1/agent/force-leave/ali-hk-bw-qa-mgdb1`
