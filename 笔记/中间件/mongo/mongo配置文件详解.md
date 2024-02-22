 mongo version 3.6.3
```
# mongodb.conf

# 数据库文件位置
dbpath=/var/lib/mongodb

#日志文件的路径
logpath=/var/log/mongodb/mongodb.log

# 是否追加方式写入日志，默认True
logappend=true

# 设置绑定ip
bind_ip = 127.0.0.1
# 设置端口
port = 27017

# 是否以守护进程方式运行，默认false
fork = true

# 启用日志文件，默认启用
journal=true

# 启用定期记录CPU利用率和 I/O 等待,默认false
#cpu = true

# 是否以安全认证方式运行，默认是不认证的非安全方式
#noauth = true
#auth = true

# 详细记录输出，默认false
#verbose = true

#用于开发驱动程序时验证客户端请求
#objcheck = true

# # 启用数据库配额管理,默认false
#quota = true

# 设置oplog日志记录等级，默认0
#   0=off (default)
#   1=W
#   2=R
#   3=both
#   7=W+some reads
#oplog = 0

# 是否打开动态调试项，默认false
#nocursors = true

# 忽略查询提示，默认false
#nohints = true

# 禁用http界面，默认为localhost：28017
#nohttpinterface = true

# 关闭服务器端脚本，这将极大的限制功能，默认false
#noscripting = true

# 关闭扫描表，任何查询将会是扫描失败
#notablescan = true

# 关闭数据文件预分配
#noprealloc = true

# 为新数据库指定.ns文件的大小，单位:MB
# nssize = <size>

# 用于Mongo监控服务器的Accout token。
#mms-token = <token>

# Mongo监控服务器的服务器名称。
#mms-name = <server-name>

# Mongo监控服务器的Ping间隔时间，即心跳
#mms-interval = <seconds>

# Replication Options

# 设置主从复制参数
#slave = true # 设置从节点
#source = master.example.com # 指定从节点的主节点
# Slave only: 指定要复制的单个数据库
#only = master.example.com
# or
#master = true # 设置主节点
#source = slave.example.com 

# 设置副本集的名字，所有的实例指定相同的名字属于一个副本集
replSet = name

#pairwith = <server:port>

# 仲裁服务器地址
#arbiter = <server:port>

# 默认为false，用于从实例设置。是否自动重新同步
#autoresync = true

# 指定的复制操作日志（OPLOG）的最大大小
#oplogSize = <MB>

# 限制复制操作的内存使用
#opIdMem = <bytes>

# 设置ssl认证
# Enable SSL on normal ports
#sslOnNormalPorts = true

# SSL Key file and password
#sslPEMKeyFile = /etc/ssl/mongodb.pem
#sslPEMKeyPassword = pass
```
```
# 普通配置文件
# mongodb.conf

dbpath=/var/lib/mongodb  
logpath=/var/log/mongodb/mongodb.log 
pidfilepath=/var/log/mongodb/master.pid  
directoryperdb=true  #只能在数据库初始化的时候配置
logappend=true  
bind_ip=127.0.0.1 
port=27017  
fork=true  

# 集群配置文件
dbpath=/var/lib/mongodb  
logpath=/var/log/mongodb/mongodb.log 
pidfilepath=/var/log/mongodb/master.pid  
directoryperdb=true  
logappend=true  
replSet=name  
bind_ip=127.0.0.1 
port=27017  
fork=true  
noprealloc=true
```
那么想改用directoryperdb模式，必定需要数据迁移，如何实施，官方手册给出了方案：

```
For standalone instances:

1. Use mongodump on the existing mongod instance to generate a backup.

2. Stop the mongod instance.

3. Add the storage.directoryPerDB value and configure a new data directory

4. Restart the mongod instance.

5. Use mongorestore to populate the new data directory.

For replica sets:

1. Stop a secondary member.

2. Add the storage.directoryPerDB value and configure a new data directory to that secondary member.

3. Restart that secondary.

4. Use initial sync to populate the new data directory.

5. Update remaining secondaries in the same fashion.

6. Step down the primary, and update the stepped-down member in the same fashion. 
```