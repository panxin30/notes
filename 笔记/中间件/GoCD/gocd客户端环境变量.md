![title](https://leanote.com/api/file/getImage?fileId=5dce0e6dab64410a98000416)
**k8s容器中的gocd客户端默认是没有环境变量的**，将相关依赖下载到服务器中相关目录，该目录是挂载到容器中的。
`$PATH:/usr/bin:/usr/sbin:/sbin:/bin:/usr/local/bin:/usr/local/sbin:/home/go/go/bin:/home/go/bin:/home/go/java/bin:/home/go/java10/bin:/home/go/maven/bin:/home/go/node-v11.14.0-linux-x64/bin`
在进入**gocd客户端容器做调试**的时候，需要手动申明下环境变量
 `export PATH=$PATH:/usr/bin:/usr/sbin:/sbin:/bin:/usr/local/bin:/usr/local/sbin:/home/go/go/bin:/home/go/bin:/home/go/java/bin:/home/go/java10/bin:/home/go/maven/bin:/home/go/node-v11.14.0-linux-x64/bin`