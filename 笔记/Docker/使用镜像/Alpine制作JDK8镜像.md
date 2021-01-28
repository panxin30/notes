centos 7
用Alpine跑了JDK8的镜像结果发现,JDK还是无法执行.后来翻阅文档才发现
Java是基于GUN Standard C library(glibc)
Alpine是基于MUSL libc(mini libc)
安装参考:
~~~html
https://github.com/sgerrand/alpine-pkg-glibc
~~~
Dockerfile
```
FROM alpine:latest
ADD jdk-8u211-linux-x64.tar.gz /usr/local/
RUN echo http://mirrors.ustc.edu.cn/alpine/v3.9/main > /etc/apk/repositories && \
    echo http://mirrors.ustc.edu.cn/alpine/v3.9/community >> /etc/apk/repositories && \
    apk update && apk upgrade
RUN apk --no-cache add ca-certificates wget && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-2.29-r0.apk && \
    apk add glibc-2.29-r0.apk
ENV JAVA_HOME=/usr/local/jdk1.8.0_211
ENV CLASSPATH=$JAVA_HOME/bin
ENV PATH=.:$JAVA_HOME/bin:$PATH
CMD ["java","-version"]
```