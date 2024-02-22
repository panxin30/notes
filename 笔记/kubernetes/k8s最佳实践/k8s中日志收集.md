## **Docker JSON 日志驱动将日志的每一行当作一条独立的消息。 该日志驱动不直接支持多行消息。你需要在日志代理级别或更高级别处理多行消息。**

参考：https://help.aliyun.com/document_detail/50441.html

# kubernetes log solustion:Log-pilot + Kafka + Logstash
log-pilot 具有如下特性：

*   一个单独的 log 进程收集机器上所有容器的日志。不需要为每个容器启动一个 log 进程。
*   支持文件日志和 stdout。docker log dirver 亦或 logspout 只能处理 stdout，log-pilot 不仅支持收集 stdout 日志，还可以收集文件日志。
*   声明式配置。当您的容器有日志要收集，只要通过 label 声明要收集的日志文件的路径，无需改动其他任何配置，log-pilot 就会自动收集新容器的日志。
*   支持多种日志存储方式。无论是强大的阿里云日志服务，还是比较流行的 elasticsearch 组合，甚至是 graylog，log-pilot 都能把日志投递到正确的地点。
*   开源。log-pilot 完全开源，您可以从[Git项目地址](https://github.com/AliyunContainerService/log-pilot)下载代码。如果现有的功能不能满足您的需要，欢迎提 issue。

# 系统ubuntu18.04,k8sv1.17.0
### **安装log-pilot**
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    k8s-app: log-pilot
  name: log-pilot
  namespace: crm-common
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: log-pilot
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: log-pilot
    spec:
      containers:
      - env:
        - name: PILOT_TYPE
          value: filebeat
        - name: LOGGING_OUTPUT
          value: kafka
        - name: KAFKA_BROKERS
          value: 192.168.60.90:9092
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
        image: registry.cn-hangzhou.aliyuncs.com/acs/log-pilot:0.9.7-filebeat
        imagePullPolicy: IfNotPresent
        name: log-pilot
        resources:
          limits:
            cpu: "1"
            memory: 1Gi
          requests:
            cpu: 100m
            memory: 1Gi
        securityContext:
          capabilities:
            add:
            - SYS_ADMIN
          procMount: Default
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: sock
        - mountPath: /var/log/filebeat
          name: logs
        - mountPath: /var/lib/filebeat
          name: state
        - mountPath: /host
          name: root
          readOnly: true
        - mountPath: /etc/localtime
          name: localtime
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: pull-secret
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
      volumes:
      - hostPath:
          path: /var/run/docker.sock
          type: ""
        name: sock
      - hostPath:
          path: /var/log/filebeat
          type: ""
        name: logs
      - hostPath:
          path: /var/lib/filebeat
          type: ""
        name: state
      - hostPath:
          path: /
          type: ""
        name: root
      - hostPath:
          path: /etc/localtime
          type: ""
        name: localtime
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
```
### **安装二进制kafka**
http://kafka.apache.org/quickstart
[**https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.4.1/kafka\_2.11-2.4.1.tgz**](https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.4.1/kafka_2.11-2.4.1.tgz)
1.下载，解压
sudo tar zxf kafka_2.11-2.4.0.tgz -C /opt/
2.安装supervisor
apt update && apt install supervisor
修改配置文件(kafka中把worker设置为一，解决pod日志乱序，但是性能可能会有问题。)
cat /etc/supervisor/supervisord.conf
```
[program:zookeeper]
command=/opt/kafka_2.11-2.4.0/bin/zookeeper-server-start.sh /opt/kafka_2.11-2.4.0/config/zookeeper.properties
stdout_logfile=/tmp/zookeeper-stdout.log
stderr_logfile=/tmp/zookeeper-error.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
autostart=true
autorestart=true
autorestart=true
[program:kafka]
command=/opt/kafka_2.11-2.4.0/bin/kafka-server-start.sh /opt/kafka_2.11-2.4.0/config/server.properties
stdout_logfile=/tmp/kafka-stdout.log
stderr_logfile=/tmp/kafka-error.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
autostart=true
autorestart=true
autorestart=true
```
启动zookeeper，启动kafka
### kafka需要提供外部访问，所以修改server.properties,listeners需要添加需要绑定的ip
### **安装logstash**
https://www.elastic.co/cn/downloads/logstash
### Logstash, as every single tool of the ELK stack, needs**Java**to run properly.
`sudo apt-get install default-jre`
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update && sudo apt install logstash
```
### **Enable your new service** on boot up and start it.

~~~
$ sudo systemctl enable logstash
$ sudo systemctl start logstash
~~~
### 修改配置文件
`deploy.spec.template.spec.containers.env`下面加一条
```
        - name: aliyun_logs_crm # crm这个名称空间要正确填写
          value: stdout
```
```
root@cn-office-bw-dev-msc1:/etc/logstash# cat conf.d/logstash.conf 
input {   
    kafka {
        bootstrap_servers => ["192.168.60.90:9092"] # 注意这里配置的kafka的broker地址不是zk的地址
        group_id => "logstash" # 自定义groupid 
        topics => ["crm-private"]  # kafka topic 名称 ,待收集日志服务的namespace，可以多个，以逗号分开。
        consumer_threads => 1 
        decorate_events => flase
        codec => "json"
        }
}

filter { 
            ruby {
                # 新增索引日期,解决地区时差问题
                code => "event.set('index_date', event.get('@timestamp').time.localtime + 8*60*60)"
            }
            mutate {
                 convert => ["index_date", "string"]
                 gsub => ["index_date", "T([\S\s]*?)Z", ""]
                 gsub => ["index_date", "-", "."]
            }
}

output {
    file {
      #path => "/data/logs/%{k8s_pod_namespace}/%{+YYYY-MM-dd}/%{k8s_pod}"
      #path => "/data/logs/%{k8s_pod_namespace}/%{index_date}/%{k8s_pod}"
      path => "/data/logs/%{k8s_pod_namespace}/%{index_date}/%{k8s_container_name}"
      codec => line { format => "%{message}"}
    }
}
```
**logstash的默认配置在config文件夹下**
默认是input125条，处理完125条再input下一批的125，优化可以考虑加大这个数值，每次处理完125条后，不会再有一个略微等待的过程。
#   # How many events to retrieve from inputs before sending to filters+workers
#   pipeline.batch.size: 125
