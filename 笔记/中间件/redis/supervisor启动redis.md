redis需要关闭守护进程
[program:redis6379]
directory=/data/redis6379
command=/usr/bin/redis-server /data/redis6379/redis.conf
autorestart=true
[program:redis6380]
directory=/data/redis6380
command=/usr/bin/redis-server /data/redis6380/redis.conf
autorestart=true