 通过`kubectl api-resources`找到我们需要的API资源，编写正确的YAML资源清单文件，但是kubernetes中资源对象太多了，不可能都记住，这时候可以借助`kubectl explain`命令来找到完整的结构
```
root@cn-office-crm-qa-all-k8s01:~# kubectl explain deployments
KIND:     Deployment
VERSION:  apps/v1

DESCRIPTION:
     Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object metadata.

   spec	<Object>
     Specification of the desired behavior of the Deployment.

   status	<Object>
     Most recently observed status of the Deployment.
```
#### **输出顶层的属性，`<string>` 表示字符串，`<Object>`表示对象，[] 表示数组，对象在YAML文件中需要缩进，数组就需要通过添加一个破折号来表示一个item**

对于对象和数组我们不知道里面有什么属性，我们可以继续查看`kubectl explain deployments.metadata`
或者
`kubectl explain deployments --recursive`
**这个命令可以将资源对象的完整属性列出来，而且缩进格式和YAML文件基本一致**
