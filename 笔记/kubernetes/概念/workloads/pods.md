```
总结概括为以下10条：

1.  不要直接部署裸的Pod。
2.  为工作负载选择合适的Controller。
3.  使用Init容器确保应用程序被正确的初始化。
4.  在应用程序工作负载启动之前先启动service。
5.  使用Deployment history来回滚到历史版本。
6.  使用ConfigMap和Secret来存储配置。
7.  在Pod里增加Readiness和Liveness探针。
8.  给Pod设置CPU和内存资源限额。
9.  定义多个namespace来限制默认service范围的可视性。
10.  配置HPA来动态扩展无状态工作负载。
```
分别用于确保不同类型的pod资源，来符合用户期望的方式进行运行。
HPA-HorizontalPodAuto
rpm -ql 查看安装后有哪些目录
### **大部分资源的配置清单都有5个部分**：
## 1. apiVersion group/version 
$ kubectl api-version
## 2.kind:资源类别
## 3.metadata：元数据，嵌套的字段，二级，三级
+ name:
+ namespace:
+ labels:
+ annotations:(注释)与label不同的地方在于,它不能用于挑选资源对象,仅用于为对象提供“元数据”。
    每个的资源的引用PATH
        /api/group/version/namespace/type/name
## 4.spec: 我们接下来创建的资源对象，应该满足什么样的规范（然后靠控制器来）disired state
## 5.status:当前这个资源的当前状态（只读,由k8s集群维护）
    kubectl explain pods 查看pods怎么定义
    kubectl explain pods.metadate
    kubectl explain pods.spec
    kubectl explain 
## **创建资源的方法**
    apiserver 仅接受JSON格式的资源定义;
    yaml格式提供配置清单，apiserver可自动将其转为json格式，然后再提交;
pod资源
+ spec.containers <[]object>
        - name <string>
          image <string>
          imagePullPolicy <string>镜像pull策略。不能直接热更，创建好后，就只能删除，更改，再新建了
            Always,Never,IfNotPresent 总是下载，总是不下载，本地有就不下载
+ 修改镜像中的默认应用：
        command要运行的程序
        args传递给程序的参数
            https:kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
+ 标签：
            key=value
                key:字母，数字，_,-,.五种。只能字母数字开头
                value:可以为空，只能以字母或数字开头及结尾，中间可使用_ - .
    kubectl get pods -l app 过滤标签
+ 标签选择器：
        等值关系：=，==,!=,
        集合关系：
              KEY in (VALUE1,VALUE2,...)
              KEY not in (VALUE1,VALUE2,...)
+ 节点选择器nodeSelector
+ nodeName直接指定运行在哪个节点
+ annotations:
       与label不同的地方在于，它不能用于挑选资源对象，仅用于为对象提供“元数据”
+ **Pod生命周期中的重要行为**:
    初始化容器
    容器探测:
      liveness主容器是否处于运行状态
      readiness容器中的主进程是否已经准备就绪，并可以对外提供服务
+ **restartPolicy**: 
    Always,OnFailure,Never,Default to Always
+ **探针类型**有三种：livenessProbe健康检测，readiness就绪检测，liftcycle?
    ExecAction：exec，通常用来嵌套一个command
    TCPSocketAction：tcpSocket，向哪个地址的哪个端口发起请求
    HTTPGetAction：httpGet，向哪个地址的哪个端口的url发起请求
