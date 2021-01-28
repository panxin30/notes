ubuntu 18.04
```
[Unit]
Description=bw-account
After=supervisord.service

[Service]
WorkingDirectory=/data/deploy/bw-account
ExecStart=/usr/bin/java -server -Xms400M -Xmx400M -XX:+UseNUMA -XX:+UseParallelGC -XX:NewRatio=1 -XX:MaxDirectMemorySize=256M -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=dump/ -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -verbosegc -Xloggc:dump/gc.log -Dlogging.config=classpath:logback-deploy.xml -jar bw-account.jar --spring.profiles.active=test

SuccessExitStatus=143

#User=brokerwork
#Group=brokerwork

[Install]
WantedBy=multi-user.target
```

```
#!/bin/bash

JAR_HOME=/home/brokerwork/bw-message
jarname=bw-message.jar
pidfile=${JAR_HOME}/server.pid
USER=brokerwork

MEMOPS="-server -Xms512M -Xmx512M -XX:+UseNUMA -XX:+UseParallelGC -XX:NewRatio=1 -XX:MaxDirectMemorySize=512M"
GCLOGOPS="-XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=/home/brokerwork/bw-message/dump/ -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -verbosegc -Xloggc:$JAR_HOME/log/gc.$$.log"
APP_OPTS="-Dlogging.config=classpath:logback-deploy.xml"

#启动方法
start(){
        if [ -e $JAR_HOME/log/process.log ];then
                mv $JAR_HOME/log/process.log $JAR_HOME/log/process.`date +"%Y%m%d-%H%S"`.log
        fi
        cd $JAR_HOME
        nohup java  ${MEMOPS} ${GCLOGOPS} ${APP_OPTS} -jar $JAR_HOME/${jarname} --spring.profiles.active=prod>${JAR_HOME}/nohup.out 2>&1 &
        sleep 3
        PID=`ps -eaf|grep $jarname|grep -v grep|awk '{print $2}'`
        if [ ${#PID} -ne 0 ]
        then
                echo 'start ok'
        else
                echo 'start faild'
        fi

}
#停止方法
stop(){
        id=`ps -ef |grep -v grep|grep "$jarname"|awk '{print $2}'`
        if [ ${#id} -ne 0 ]
        then
                kill -9 $id
        else
                echo 'pid not found.'
        fi

}

case "$1" in
start)
start
;;
stop)
stop
;;
restart)
stop
sleep 5
start
;;
*)
printf 'Usage: %s {start|stop|restart}\n' "$prog"
exit 1
;;
esac
```