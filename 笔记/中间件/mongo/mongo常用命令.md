## **单机mongo启用认证**
```
security:
		authorization: "enabled"
```
启动
mongod -f /etc/mongod.conf

##  **创建超级管理员**
```
use admin
db.createUser({"user":"admin","pwd":"WAer8R59G6","roles":["root"]});
```
## **首先要use admin**
```
db.createUser({user:"jason",pwd:"leanwork2018",roles:[{ role: "readAnyDatabase", db: "admin" }]});
db.createUser({user:"bwrakeback",pwd:"aLHGzwJP0",roles:["readWriteAnyDatabase"]});
退出开启认证模式，重启，再建立普通用户
use admin
db.auth('admin',' RhFUUWI6vQ0k');
```
##  **创建普通用户**
```
use mq-beta
db.createUser({user:"mq-beta",pwd:"abc123",roles:[{role:"readWrite",db:"mq-beta"}]});
建表
db.createCollection("test1")
删表
db.getCollection('test1').remove({})
删除用户
db.system.users.find()
```
# mongo 4.0+ 创建用户
```
use brokerwork
db.createUser({user:"brokerwork",pwd:"o6RJzAF1LELzutEv",roles:[{role:"readWrite",db:"brokerwork"}],mechanisms : ["SCRAM-SHA-1"]})
```
## **修改密码**
`db.changeUserPassword('tank2','test');`
### **删除一条记录**
```
db.t_pub_user.find({ "_id" : "dd2ba7f0-e7f9-453e-b960-ebdc83b251d3"})
{ "_id" : "dd2ba7f0-e7f9-453e-b960-ebdc83b251d3", "_class" : "com.lwork.bw.transfer.pubauth.domain.to.User", "tenantId" : "T000004", "productId" : "BW", "username" : "Tom", "phone" : "", "isPhoneVerified" : false, "email" : "tom@lwork.com", "isEmailVerified" : true, "createTime" : ISODate("1970-01-17T08:01:47.058Z"), "lastLoginTime" : ISODate("1970-01-17T20:33:03.058Z"), "isAlive" : false, "isSCUser" : false, "isPendingMigrateUser" : true }
_id这个是key，后面的""是值。
db.t_customer_profiles.deleteOne({"_id" : "3fd38e17-6a78-42d8-983a-545620295c07"})

db.getCollection('t_template').remove({'tenantId':{$in:[]}});

db.users.find({name:/hurry/});
db.t_product_deploy.find({url:/3308/});
# 删除collection
use db;
db.collection.drop();
# 删除db
use db;
db.dropDatabase()

```
#### **创建一个监控账户，库admin**
```
db.createUser({user:"zabbixmonitor",pwd:"leanwork2018",roles:["clusterMonitor"]});
```
### **因为mongodb默认是从主节点读写数据的，副本节点上不允许读，需要设置副本节点可以读**
`db.getMongo().setSlaveOk();`
### **查询最大连接数**
`db.serverStatus().connections;`
Current+available
mongod或mongos支持的最大并发连接数受操作系统ulimit(可通过/etc/security/limits.conf文件来配置)和服务端maxConn参数限制，取其中较小值，这两个参数均可调整。
**副本集状态**
-----
rs.status();

**冻结Secondary节点**
-------------

如果需要对Primary做一下维护，但是不希望在维护的这段时间内将其它Secondary节点选举为Primary节点，可以在每次Secondary节点上执行freeze命令，强制使它们始终处于Secondary节点状态。

myapp:SECONDARY> rs.freeze(100)
注：只能在Secondary节点上执行

复制代码
myapp:PRIMARY> rs.freeze(100)
{
	"ok" : 0,
	"errmsg" : "cannot freeze node when primary or running for election. state: Primary",
	"code" : 95,
	"codeName" : "NotSecondary"
}
复制代码
如果要解冻Secondary节点，只需执行

myapp:SECONDARY> rs.freeze()

添加和删除节点
-------
				   rs.add("ip+端口号")添加节点
这时使用命令rs.remove("IP+端口")即可移除该节点
### **查询mongo某表索引**
db.t_tasks_job_T900003.getIndexes()
### **为普通字段添加索引，并且为索引命名**
db.*集合名*.createIndex( {"*字段名*": 1 },{"**name**":'idx_*字段名*'})
`db.t_new_tasks_job_T001630.createIndex({"createTime":1.0, "jobType":1.0, "jobState":1.0},{"name":"t_new_tasks_job_c_j_j_1"});`
`db.finish_tickets.createIndex( { "ticket.closetime": -1,"ticket.trade_server":1,"deposits.user_id":1}, { "background": true, "name": "rcr_report_query_index" } );`
### **删除索引**
//删除t3 表中的所有索引
db.t3.dropIndexes()
//删除t4 表中的firstname 索引
db.t4.dropIndex({firstname: 1})

`db.t_new_tasks_job_T001152.dropIndex({"jobType" : 1,"createTime" : 1,"jobState" : 1})`
```
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "brokerwork.t_tasks_job_T200128"
	},
	{
		"v" : 1,
		"key" : {
			"tenantId" : 1
		},
		"name" : "t_job_t_c",
		"ns" : "brokerwork.t_tasks_job_T200128"
	}
]
```
```
db.t_tasks_job_T200128.dropIndex({tenantId:1})
```
## **mongo必需副本集或分片才能支持事务**
# mongo清理了collections空间不会立即释放
参考：https://zhuanlan.zhihu.com/p/38979077
# 配置单节点副本集
引用：https://driverzhang.github.io/post/mongo%E5%BC%80%E5%90%AF%E4%BA%8B%E5%8A%A1%E4%B9%8B%E5%89%AF%E6%9C%AC%E9%9B%86%E9%87%87%E5%9D%91/
