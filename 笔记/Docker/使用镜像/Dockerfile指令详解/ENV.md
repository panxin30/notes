### **ENV**
### 容器中查看环境变量`printenv`
用于为镜像定义所需的环境变量，并可被Dockerfile文件中位于其后的其他指令（如ENV，ADD，COPY等）所调用
```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```
可以一行设置多个变量，如果value中有空格，可以使用反斜线转义，也可以通过对value加引号
```
FROM docker-public.lwork.com:5000/base-jdk:v1.2
ENV artifact=bw-account.jar workingDir=/home/brokerwork user=brokerwork

RUN useradd $user -m -d $workingDir && \
        mkdir $workingDir/log && \
        ln -s /dev/stdout $workingDir/log/process.log && \
        ln -s /dev/stdout $workingDir/log/access.log && \
        chown $user:$user $workingDir/log && \
        chmod 755 $workingDir
COPY $artifact /home/$user/$artifact
WORKDIR $workingDir
USER $user
CMD ["sh", "-c", "java -jar ${artifact}"]

```
通过环境变量，我们可以让一份`Dockerfile`制作更多的镜像，只需使用不同的环境变量即可。