Pod:
+ 自主Pod
+ **控制器管理的Pod**
    + RelicationController 最早的。可以滚动更新和回滚。
    + ReplicaSet 新一代的。1用户期待的pod副本有几个，2标签选择器，3现存pod不够，它会新建。不建议直接使用，而是deployment。deployment
    + Deployment  声明式 管理无状态 用的最多，还支持滚动更新和回滚.建构在RS之上的ReplicaSet，还能控制更新节奏 
deployment---replicaset---pod
kubectl explain deploy
pod数量可以大于节点数量，pod和节点没有对应关系
    + statefulSet  有状态副本集。每一个pod都是被单独管理，独有标识和独有数据集，在被加时需要数据在。比如redis cluster,mysql主从，mongo cluster。非常麻烦，没法抽取个共同策略，只能单独对待。
CDR：Custom Defined Resources,1.8+
Operator:
Helm:外壳，helm install（相当于yum install）
    + DaemonSet  需要在每一个node运行一个，而不是随意运行（实现系统级的），一般是无状态的，必须是守护状态的
    + Job,Ctonjob 作业，周期性作业。任务完成就释放
## Pod控制器
  ReplicationController:
  ReplicaSet:
  Deployment:
  DaemonSet:
  Job:
  Cronjob:
  StatefulSet