引用：https://www.cnblogs.com/operationhome/p/10907591.html
```
Docker-CE
Server Version: 18.09.6
Storage Driver: overlay2
Kernel Version: 5.2.0-050200-generic
Operating System: Ubuntu 18.04.2 LTS
```
**Docker 日志分为两类：**

Docker 引擎日志(也就是 dockerd 运行时的日志)，
容器的日志，容器内的服务产生的日志。
**使用 Docker-CE 版本**
docker logs命令 仅仅适用于以下驱动程序
```
local
json-file
journald
```
**Docker 日志驱动全局配置更改**
修改日志驱动，在配置文件 /etc/docker/daemon.json（注意该文件内容是 JSON 格式的）进行配置即可。
示例：
```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "2048m",
    "max-file": "5"
  }
}
```
**json-file 日志的路径位于** /var/lib/docker/containers/container_id/container_id-json.log。

json-file 的 日志驱动支持以下选项：

选项	描述	示例值
max-size	切割之前日志的最大大小。可取值单位为(k,m,g)， 默认为-1（表示无限制）。	--log-opt max-size=10m

max-file	可以存在的最大日志文件数。如果切割日志会创建超过阈值的文件数，则会删除最旧的文件。仅在max-size设置时有效。正整数。默认为1。	--log-opt max-file=3

labels	适用于启动Docker守护程序时。此守护程序接受的以逗号分隔的与日志记录相关的标签列表。	--log-opt labels=production_status,geo

env	适用于启动Docker守护程序时。此守护程序接受的以逗号分隔的与日志记录相关的环境变量列表。	--log-opt env=os,customer

env-regex	类似于并兼容env。用于匹配与日志记录相关的环境变量的正则表达式。	--log-opt env-regex=^(os|customer).

compress	切割的日志是否进行压缩。默认是disabled。	--log-opt compress=true

**清理Docker容器日志（治标）**
容器存储路径/data/docker
日志大小2048M
存5份
```
#!/bin/bash
logs=$(find /data/docker/containers/ -name *-json.log.*)

for log in $logs
        do
                cat /dev/null > $log
        done

exit 0
```