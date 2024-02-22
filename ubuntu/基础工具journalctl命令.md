```
Systemd 统一管理所有 Unit 的启动日志。带来的好处就是 ，可以只用journalctl一个命令，查看所有日志（内核日志和 应用日志）。

日志的配置文件/etc/systemd/journald.conf 
```

## journalctl用法

### 查看所有日志（默认情况下 ，只保存本次启动的日志）
```
        journalctl 
```

## 查看内核日志（不显示应用日志）
```
        journalctl -k 
```

## 查看系统本次启动的日志
```
        journalctl -b
```

## 按服务
```
journalctl -u httpd.service
```

## 按进程、用户或群组ID
```
journalctl _PID=8088
```

## 输出格式
```
root@cn-office-crm-qa-all-k8s02:~# journalctl -u mongod -o  json-pretty
{
        "__CURSOR" : "s=bca3fda678194adf93b185e09a97c34b;i=2bb;b=a6efcf69186444a7ac50e6d46541ff28;m=3f1d8c8;t=5c50e9f6baf2d;x=9afba0039965fae1",
        "__REALTIME_TIMESTAMP" : "1624041478401837",
        "__MONOTONIC_TIMESTAMP" : "66181320",
        "_BOOT_ID" : "a6efcf69186444a7ac50e6d46541ff28",
        "_MACHINE_ID" : "38d5d3255ee14f5c9e68eaf9270c4804",
        "_HOSTNAME" : "cn-office-crm-qa-all-k8s02",
        "PRIORITY" : "6",
        "SYSLOG_FACILITY" : "3",
        "SYSLOG_IDENTIFIER" : "systemd",
        "_UID" : "0",
        "_GID" : "0",
        "_SELINUX_CONTEXT" : "unconfined\n",
        "_TRANSPORT" : "journal",
        "_PID" : "1",
        "_COMM" : "systemd",
        "_EXE" : "/lib/systemd/systemd",
        "_CMDLINE" : "/sbin/init",
        "_CAP_EFFECTIVE" : "3fffffffff",
        "_SYSTEMD_CGROUP" : "/init.scope",
        "_SYSTEMD_UNIT" : "init.scope",
        "_SYSTEMD_SLICE" : "-.slice",
        "CODE_FILE" : "../src/core/job.c",
        "CODE_LINE" : "842",
        "CODE_FUNC" : "job_log_status_message",
        "JOB_TYPE" : "start",
        "JOB_RESULT" : "done",
        "MESSAGE_ID" : "39f53479d3a045ac8e11786248231fbf",
        "MESSAGE" : "Started MongoDB Database Server.",
        "UNIT" : "mongod.service",
        "INVOCATION_ID" : "e60677f45d794c3797f7686cda26a86e",
        "_SOURCE_REALTIME_TIMESTAMP" : "1624041478401812"
}
```