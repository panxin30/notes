docker build
# 用于go语言
```
FROM docker-public.test.com:5000/base-jdk:v1.2
ENV artifact bw-copy-trade
ENV cfile app.toml                                                                   
ENV workingDir /home/brokerwork
ENV user brokerwork
RUN useradd $user -m -d $workingDir && \
    mkdir $workingDir/log && \
        ln -s /dev/stdout $workingDir/log/process.log && \
    ln -s /dev/stdout $workingDir/log/access.log && \
    chown $user:$user $workingDir/log && \
    chmod 755 $workingDir
ADD $artifact /home/$user/$artifact
ADD start-s.sh /home/$user/start.sh
COPY $cfile /home/$user/
WORKDIR $workingDir
RUN chmod +x $artifact start.sh
USER $user
CMD ["sh", "-c", "./start.sh ${artifact}"]
```
# 用于java
```
FROM docker-public.test.com:5000/base-jdk:v1.2
ENV artifact bw-account.jar
ENV workingDir /home/brokerwork
ENV user brokerwork
RUN useradd $user -m -d $workingDir && \
	mkdir $workingDir/log && \
        ln -s /dev/stdout $workingDir/log/process.log && \
	ln -s /dev/null $workingDir/log/access.log && \
	chown $user:$user $workingDir/log && \
	chmod 755 $workingDir 
ADD $artifact /home/$user/$artifact
WORKDIR $workingDir 
USER $user
CMD ["sh", "-c", "java ${JAVA_OPT} -jar ${artifact}"]
```