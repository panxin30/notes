默认地址~/.kube/config
**其中certificate-authority-data、client-certificate-data、client-key-data分别对应CA证书、client证书和client私钥，不过config文件里的内容是才用base64编码的。**

**在开启了 TLS 的集群中，每当与集群交互的时候少不了的是身份认证，使用 kubeconfig（即证书） 和 token 两种认证方式是最简单也最通用的认证方式**