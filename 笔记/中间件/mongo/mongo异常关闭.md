mongodb 异常关闭，
/usr/bin/mongod --config /etc/mongod.conf 可以先用root用户手动启动下

再用mongod用户启动前，先删除mongodb数据存储目录里面的WiredTiger.lock，mongod.lock
检查mongodb的数据目录和日志是不是属于mongod

## yum安装mongodb-org,会同时安装依赖，卸载的时候一些依赖需要rpm -e卸载--nodeps（不检查依赖关系） 
## mongo集群，添加新节点，数据太大，新节点直接加入，会失败，可以先拷贝数据到新节点再开启
