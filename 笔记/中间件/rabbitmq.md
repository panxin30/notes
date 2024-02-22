# RabbitMQ和kafka的区别
1.应用场景方面
RabbitMQ：用于实时的，对可靠性要求较高的消息传递上。
kafka：用于处于活跃的流式数据，大数据量的数据处理上。
2.架构模型方面
producer，broker，consumer
RabbitMQ：以broker为中心，有消息的确认机制
kafka：以consumer为中心，无消息的确认机制
3.吞吐量方面
RabbitMQ：支持消息的可靠的传递，支持事务，不支持批量操作，基于存储的可靠性的要求存储可以采用内存或硬盘，吞吐量小。
kafka：内部采用消息的批量处理，数据的存储和获取是本地磁盘顺序批量操作，消息处理的效率高，吞吐量高。
4.集群负载均衡方面
RabbitMQ：本身不支持负载均衡，需要loadbalancer的支持
kafka：采用zookeeper对集群中的broker，consumer进行管理，可以注册topic到zookeeper上，通过zookeeper的协调机制，producer保存对应的topic的broker信息，可以随机或者轮询发送到broker上，producer可以基于语义指定分片，消息发送到broker的某个分片上。
# 实践
官方文档：http://www.rabbitmq.com/install-debian.html#apt
### **ubuntu 16.04安装**
1.Signing Key
In order to use the repository, add a key used to sign RabbitMQ releases to apt-key:
`wget -O - 'https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc' | sudo apt-key add -`
2.`echo "deb https://dl.bintray.com/rabbitmq/debian xenial main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list`
3.provides Erlang/OTP packages.  Add Repository Signing Key
```
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -
# 或者
sudo apt-key adv --keyserver "hkps.pool.sks-keyservers.net" --recv-keys "0x6B73A36E6026DFCA"
```
4. It is possible to list multiple repositories, for example, one that provides RabbitMQ and one that provides Erlang/OTP packages.
```
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian xenial erlang
deb https://dl.bintray.com/rabbitmq/debian xenial main
EOF
```
### **ubuntu 18.04安装**

1.Signing Key
In order to use the repository, add a key used to sign RabbitMQ releases to apt-key:
`wget -O - "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | sudo apt-key add -`
2.`echo "deb https://dl.bintray.com/rabbitmq/debian bionic main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list`
3.provides Erlang/OTP packages.  Add Repository Signing Key
```
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -
# 或者
sudo apt-key adv --keyserver "hkps.pool.sks-keyservers.net" --recv-keys "0x6B73A36E6026DFCA"
```
4. It is possible to list multiple repositories, for example, one that provides RabbitMQ and one that provides Erlang/OTP packages. 
```
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF
```
5. **安装**
```
apt-get update
apt-get install rabbitmq-server
rabbitmq-plugins enable rabbitmq_management#启用管理插件
rabbitmqctl add_user twdev LeanWork2015
rabbitmqctl set_user_tags twdev administrator?????

rabbitmqctl add_user msc abc123
rabbitmqctl add_vhost msc-mq
rabbitmqctl set_permissions -p msc-mq msc '.*' '.*' '.*'
rabbitmqctl list_users;
rabbitmqctl list_vhosts
```
## **guest 用户如何禁用**
```
rabbitmq-plugins enable rabbitmq_management
rabbitmqctl add_user bwpre admin
rabbitmqctl set_permissions  bwpre  ".*"  ".*"  ".*"
rabbitmqctl set_user_tags bwpre administrator
chmod 700 /var/lib/rabbitmq/.erlang.cookie
echo -n "leanwork" > /var/lib/rabbitmq/.erlang.cookie
chmod 400 /var/lib/rabbitmq/.erlang.cookie
service rabbitmq-server restart
rabbitmq-plugins enable rabbitmq_management
rabbitmqctl join_cluster rabbit@ali-hk-bwpre-mq1 --ram
```
### **常用命令**
```
rabbitmqctl list_queues --vhost brokerwork
rabbitmqctl delete_queue --vhost brokerwork BW_CUSTOMER_RCV_QUEUE
```
### **centos 7.4安装**
1 `https://www.rabbitmq.com/install-rpm.html#package-cloud`
`Using Bintray Yum Repository`

2 
`curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash` 
`curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash`