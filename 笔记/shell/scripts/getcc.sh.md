
```
#!/bin/bash
nyr=$(date "+%d/%b/%Y")
mnow=$(date +"%H:%M")
mfiveago=$(date +"%H:%M:%S" -d'-5minute')
moneago=$(date +"%H:%M:%S" -d'-1minute')
msecondago=$(date +"%H:%M:%S" -d'-2minute')
TIMED=$(date +"%Y%m%d%H%M%S")

cat /var/log/nginx/*tw.lwork.com.log |grep "$nyr" |sed -n "/$msecondago/,/$mnow/p" |awk '{print $1,$6,$14}' |sort |uniq -c |sort -k1nr |awk '$1 >=50' >/root/cc && \
echo `date` >>/root/cc.bak && \
cat /root/cc >>/root/cc.bak && \
cat /root/cc |grep -v '127.0.0.1' |awk '{print $2}' |sort -u >/dev/shm/cc && \
num=$(cat /dev/shm/cc|wc -l)
if [[ $num -ge 1 ]]
then
        cp /dev/shm/cc /tmp/ccnginx3 && \
        cat /tmp/ccnginx2 >/root/working/blackips.list && cd /root/working/ && ./add-blacklist
else
        echo "x"
fi
```