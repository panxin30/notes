# protected-mode no 保护模式关闭
------------------------

官方网站
https://redis.io/topics/sentinel

1.
Sentinel是一个管理多个redis实例的工具，它可以实现对redis的监控、通知、自动故障转移。sentinel不断的检测redis实例是否可以正常工作，通过API向其他程序报告redis的状态，如果redis master不能工作，则会自动启动故障转移进程，将其中的一个slave提升为master，其他的slave重新设置新的master实例。也就是说，它提供了：
监控（Monitoring）： Sentinel 会不断地检查你的主实例和从实例是否正常。
通知（Notification）： 当被监控的某个 Redis 实例出现问题时， Sentinel 进程可以通过 API 向管理员或者其他应用程序发送通知。
自动故障迁移（Automatic failover）： 当一个主redis实例失效时， Sentinel 会开始记性一次failover， 它会将失效主实例的其中一个从实例升级为新的主实例， 并让失效主实例的其他从实例改为复制新的主实例； 而当客户端试图连接失效的主实例时， 集群也会向客户端返回新主实例的地址， 使得集群可以使用新主实例代替失效实例。
Redis Sentinel自身也是一个分布式系统， 你可以在一个架构中运行多个 Sentinel 进程， 这些进程使用流言协议（gossip protocols)来接收关于主Redis实例是否失效的信息， 然后使用投票协议来决定是否执行自动failover，以及评选出从Redis实例作为新的主Redis实例。
1.启动sentinel的方法
当前Redis stable版已经自带了redis-sentinel这个工具。虽然 Redis Sentinel 已经提供了一个单独的可执行文件 redis-sentinel ， 但实际上它只是一个运行在特殊模式下的 Redis实例， 你可以在启动一个普通 Redis实例时通过给定 –sentinel 选项来启动 Redis Sentinel 实例。也就是说：
redis-sentinel /path/to/sentinel.conf
等同于
redis-server /path/to/sentinel.conf --sentinel
其中sentinel.conf是redis的配置文件，Redis sentinel会需要写入配置文件来保存sentinel的当前状态。当配置文件无法写入时，Sentinel启动失败。

#### 2.sentinel的配置

一个简单的sentinel配置文件实例如下：
port 26329
sentinel monitor myredis 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
sentinel notification-script myredis <script-path>
sentinel 选项的名字 主Redis实例的名字 选项的值
配置文件的格式为：
第一行配置指示 Sentinel 去监视一个名为 mymaster 的主redis实例， 这个主实例的 IP 地址为本机地址127.0.0.1 ， 端口号为 6379 ， 而将这个主实例判断为失效至少需要 2 个 Sentinel 进程的同意，只要同意 Sentinel 的数量不达标，自动failover就不会执行。同时，一个Sentinel都需要获得系统中大多数Sentinel进程的支持， 才能发起一次自动failover， 并预留一个新主实例配置的编号。而当超过半数Redis不能正常工作时，自动故障转移是无效的。
各个选项的功能如下：
down-after-milliseconds 选项指定了 Sentinel 认为Redis实例已经失效所需的毫秒数。
当实例超过该时间没有返回PING，或者直接返回错误， 那么 Sentinel 将这个实例标记为主观下线（subjectively down，简称 SDOWN ）。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移： 只有在足够数量的 Sentinel 都将一个实例标记为主观下线之后，实例才会被标记为客观下线（objectively down， 简称 ODOWN ）， 这时自动故障迁移才会执行。具体的行为如下：
（1）. 每个 Sentinel 每秒一次向它所监控的主实例、从实例以及其他 Sentinel 实例发送一个 PING 命令。当一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 那么这个实例会被 Sentinel 标记为主观下线。如果一个主实例被标记为主观下线， 并且有足够数量的 Sentinel （至少要达到配置文件指定的数量）在指定的时间范围内同意这一判断， 那么这个主实例被标记为客观下线。
（2）.在一般情况下， 每个 Sentinel 进程会以每 10 秒一次的频率向它已知的所有主实例和从实例发送 INFO 命令。 当一个主实例被 Sentinel实例标记为客观下线时， Sentinel 向下线主实例的所有从实例发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
（3）.当没有足够数量的 Sentinel 同意主实例已经下线， 主Redis服务实例的客观下线状态就会被移除。 当主服务器重新向 Sentinel 的PING 命令返回有效回复时， 主服务器的主观下线状态就会被移除。
parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从Redis实例在同步新的主实例， 在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长。
尽管复制过程的绝大部分步骤都不会阻塞从实例， 但从redis实例在载入主实例发来的 RDB 文件时， 仍然会造成从实例在一段时间内不能处理命令请求： 如果全部从实例一起对新的主实例进行同步， 那么就可能会造成所有从Redis实例在短时间内全部不可用的情况出现。
所以从实例被设置为允许使用过期数据集（参见对 redis.conf 文件中对 slave-serve-stale-data 选项），可以缓解所有从实例都在同一时间向新的主实例发送同步请求的负担。你可以通过将这个值设为 1 来保证每次只有一个从Redis实例处于不能处理命令请求的同步状态。
failover-timeout如果在该时间（ms）内未能完成failover操作，则认为该failover失败。
notification-script: 指定sentinel检测到该监控的redis实例指向的实例异常时，调用的报警脚本。该配置项可选，但是很常用。

#### 3. Sentinel集群的运行机制

一个 Sentinel进程可以与其他多个 Sentinel进程进行连接， 每个 Sentinel进程之间可以互相检查对方的可用性， 并进行信息交换。
和其他集群不同的是，你无须设置其他Sentinel的地址，Sentinel进程可以通过发布与订阅来自动发现正在监视相同主实例的其他Sentinel。同样，你也不必手动列出主实例属下的所有从实例，因为Sentinel实例可以通过询问主实例来获得所有从实例的信息。
每个 Sentinel 都订阅了被它监视的所有主服务器和从服务器的 sentinel:hello 频道， 查找之前未出现过的 sentinel进程。 当一个 Sentinel 发现一个新的 Sentinel 时，它会将新的 Sentinel 添加到一个列表中，这个列表保存了 Sentinel 已知的，监视同一个主服务器的所有其他Sentinel。

#### 4. 启动Redis和sentinel

下面的测试环境为centos el6.x86_64，Redis 3.2.3。下面首先配置一个小型的M/2S环境。
/etc/redis/redis-m.conf
daemonize yes
pidfile /var/run/redis/redis-server-m.pid
port 6379
loglevel notice
logfile /var/log/redis/redis-server-m.log
databases 16
##disable snapshot
save ""
dir /app/redis-m
appendonly yes
appendfilename "appendonly.aof"
##not to be a slave
#slaveof no one

/etc/redis/redis-s1.conf
daemonize yes
pidfile /var/run/redis/redis-server-s1.pid
port 6380
loglevel warning
logfile /var/log/redis/redis-server-s1.log
databases 16
##disable snapshot
save ""
dir /app/redis-s1
appendonly yes
appendfilename "appendonly.aof"
##to be a slave
slaveof 127.0.0.1 6379

/etc/redis/redis-s2.conf
daemonize yes
pidfile /var/run/redis/redis-server-s2.pid
port 6381
loglevel warning
logfile /var/log/redis/redis-server-s2.log
databases 16
##disable snapshot,enable AOF
save ""
dir /app/redis-s2
appendonly yes
appendfilename "appendonly.aof"
##to be a slave
slaveof 127.0.0.1 6379
启动后，查看进程和复制状态是否正常：
ps -ef | grep redis
root 17560 1 0 09:48 ? 00:00:00 redis-server *:6379 
root 17572 1 0 09:48 ? 00:00:00 redis-server *:6380 
root 17609 1 0 09:49 ? 00:00:00 redis-server *:6381

redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=547,lag=0
slave1:ip=127.0.0.1,port=6381,state=online,offset=547,lag=1
master_repl_offset:547
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:546
前面我们了解了，Sentinel可以通过master Redis实例来获得它的从实例的信息。所以每一个Sentinel只配置主实例的监控即可。Sentinel之间端口有所不同。
port 26379
sentinel monitor myredis 127.0.0.1 6379 2
sentinel down-after-milliseconds myredis 60000
sentinel failover-timeout myredis 180000
sentinel parallel-syncs myredis 1
sentinel notification-script myredis /etc/redis/log-issues.sh
其中，通知脚本简单的记录一下failover事件。
#!/bin/bash

echo "master failovered at `date`" > /root/redis_issues.log
另外两个端口分别为port 26380。然后通过redis-sentinel命令来启动两个sentinel实例。

#### 启动方式

#### /usr/local/redis/bin/redis-sentinel /usr/local/redis/etc/sentinel.conf --sentinel > /usr/local/redis/logs/sentinel.log 2>&1 &
redis-sentinel /path/to/sentinel.conf

redis-server /path/to/sentinel.conf --sentinel
通过日志输出可以看出已经成功监控master和两个slave。并和另一个sentinel进程通信。
[18207] 08 May 13:28:29.336 # Sentinel runid is d72be991001b994568d3de1b746f611884b0343a
[18207] 08 May 13:28:29.336 # +monitor master myredis 127.0.0.1 6379 quorum 2
[18207] 08 May 13:28:29.339 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
[18207] 08 May 13:28:29.339 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
[18207] 08 May 13:28:36.602 * +sentinel sentinel 127.0.0.1:26479 127.0.0.1 26479 @ myredis 127.0.0.1 6379

#### 访问

#### 使用vip进行轮询，所以在sentinel配置文件中绑定了三台服务器各自的内网地址
redis-cli -h 10.80.96.111 -p 26379 -a 'mima' info Sentinel
或者sentinel master mymaster  查看sentinel信息

#### 测试
Testing the failover
At this point our toy Sentinel deployment is ready to be tested. We can just kill our master and check if the configuration changes. To do so we can just do:
redis-cli -p 6379 DEBUG sleep 30
This command will make our master no longer reachable, sleeping for 30 seconds. It basically simulates a master hanging for some reason.
If you check the Sentinel logs, you should be able to see a lot of action:
Each Sentinel detects the master is down with an +sdown event.
This event is later escalated to +odown, which means that multiple Sentinels agree about the fact the master is not reachable.
Sentinels vote a Sentinel that will start the first failover attempt.
The failover happens.
If you ask again what is the current master address for mymaster, eventually we should get a different reply this time:
127.0.0.1:5000> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6380"
So far so good... At this point you may jump to create your Sentinel deployment or can read more to understand all the Sentinel commands and internals.

#### sentinel 命令
Sentinel commands
The following is a list of accepted commands, not covering commands used in order to modify the Sentinel configuration, which are covered later.
PING This command simply returns PONG.
SENTINEL masters Show a list of monitored masters and their state.
SENTINEL master <master name> Show the state and info of the specified master.
SENTINEL slaves <master name> Show a list of slaves for this master, and their state.
SENTINEL sentinels <master name> Show a list of sentinel instances for this master, and their state.
SENTINEL get-master-addr-by-name <master name> Return the ip and port number of the master with that name. If a failover is in progress or terminated successfully for this master it returns the address and port of the promoted slave.
SENTINEL reset <pattern> This command will reset all the masters with matching name. The pattern argument is a glob-style pattern. The reset process clears any previous state in a master (including a failover in progress), and removes every slave and sentinel already discovered and associated with the master.
SENTINEL failover <master name> Force a failover as if the master was not reachable, and without asking for agreement to other Sentinels (however a new version of the configuration will be published so that the other Sentinels will update their configurations).
SENTINEL ckquorum <master name> Check if the current Sentinel configuration is able to reach the quorum needed to failover a master, and the majority needed to authorize the failover. This command should be used in monitoring systems to check if a Sentinel deployment is ok.
SENTINEL flushconfig Force Sentinel to rewrite its configuration on disk, including the current Sentinel state. Normally Sentinel rewrites the configuration every time something changes in its state (in the context of the subset of the state which is persisted on disk across restart). However sometimes it is possible that the configuration file is lost because of operation errors, disk failures, package upgrade scripts or configuration managers. In those cases a way to to force Sentinel to rewrite the configuration file is handy. This command works even if the previous configuration file is completely missing.

注：注：如果redis master设置了验证密码，还需配置masterauth xxx，指redis master上认证密码即可。
	如果使用sentinel监控主从服务器及实现主备切换，那么需要设置以下参数：
	requirepass 123456	
	masterauth 123456
