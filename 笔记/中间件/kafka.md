官网 http://kafka.apache.org/documentation/#operations
~~~
    env:
      - name: "FILEBEAT_OUTPUT"
        value: "kafka"
      - name: "KAFKA_BROKERS"
        value: "kafka1:9092,kafka2:9092" 
~~~
### Kafka 从 2.2 版本开始将 kafka-topic.sh 脚本中的−−zookeeper参数标注为 “~过时~”，推荐使用−−bootstrap-server参数

## **kafka连接自己服务器的时候没有白名单，导致错误**
## **listeners，用于server真正bind 。advertisedListeners， 用于开发给用户，如果没有设定，直接使用listeners**
**启动kafka**
kafka启动时先启动zookeeper，再启动kafka；关闭时相反，先关闭kafka，再关闭zookeeper
启动zookeeper：
        bin/zookeeper-server-start.sh config/zookeeper.properties &
启动kafka：
        bin/kafka-server-start.sh config/server.properties &
————————————————

##create topic
bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic test
##list topic
bin/kafka-topics.sh --list --zookeeper localhost:2181
bin/kafka-topics.sh --list --zookeeper zk-qa:2181
bin/kafka-topics.sh --list --zookeeper zookeeper:2181
###producer 生产者
bin/kafka-console-producer.sh --broker-list 192.168.60.168:9092 --topic test
# config/server.properties 中的advertisedListeners 绑定的是192.168.60.168:9092
###cumster 消费者
bin/kafka-console-consumer.sh --bootstrap-server kafka-bw-hk.public.lwork.com:9092 --topic test --from-beginning
# 域名解析到了192.168.60.168内网
bin/kafka-console-consumer.sh --bootstrap-server 192.168.60.168:9092 --topic test --from-beginning

在dockerup上面根据wurstmeister/kafka:2.12-2.1.0，查看该镜像可以配置哪些环境变量
### k8s的kafka通过这条命令**强制使用这个地址**，即使在外部用了nodeportIP最终转发进kafka会跳转到这个地址
advertised.listeners=PLAINTEXT://52.25.141.94:9092

bin/zookeeper-shell.sh zk:2181 <<< "get /brokers/ids/0"

### **k8s安装kafka**

### **Kafka 专用术语**
Broker：Kafka 集群包含一个或多个服务器，这种服务器被称为 broker。

Topic：每条发布到 Kafka 集群的消息都有一个类别，这个类别被称为 Topic。（物理上不同 Topic 的消息分开存储，逻辑上一个 Topic 的消息虽然保存于一个或多个 broker 上，但用户只需指定消息的 Topic 即可生产或消费数据而不必关心数据存于何处）。

Partition：Partition 是物理上的概念，每个 Topic 包含一个或多个 Partition。

Producer：负责发布消息到 Kafka broker。

Consumer：消息消费者，向 Kafka broker 读取消息的客户端。

Consumer Group：每个 Consumer 属于一个特定的 Consumer Group（可为每个 Consumer 指定 group name，若不指定 group name 则属于默认的 group）。

