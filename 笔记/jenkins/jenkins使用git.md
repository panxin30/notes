版本2.164.3

# **jenkins使用Git**
如果使用ssh非密码认证的方式访问Git仓库，地址类似git@gitlab.tools.lwork.com:java/bw-custom.git 需要提供公钥id_ras.pub给gitlab，远程服务器的指纹还需要放在~/.ssh/known_hosts里面，防止第一次访问Git服务器时，jenkins无形的提示访问授权
或者，用jenkins用户手动Git克隆远程仓库，检测你的私钥的设置

**1. 在需要拉取代码的服务器生成新的密钥,或者用已有的**：
`ssh-keygen`
生成了二个文件在.ssh/目录下:id_rsa 和 id_rsa.pub
```
jenkins@cn-office-tonytest-jenkins:~$ ssh-keygen
Generating public/private rsa key pair.
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
jenkins@cn-office-tonytest-jenkins:~$ cat .ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZaYbFBKxscGCfU729RrchRG5jIpNHICKG3oyq3vG5EZGheiUJ4NAJrxQeUmWUgScrYmOQiRY13KPNfZWxP+ptTwuslhosHhVvNNKEyo8eLbWYsBMm1ijUMGYiYfJIqCUcnI6PnN345jgBfN75hHG+JxBtB2zWgZRBWX/jZW2C/i7WmN5v3Jj6kCXqxaiPlcUZXTodBfSEH7eUL6I1vPdFOlh2miqZ5dxd/JGf4zCAPGD6I2UKYI/gpH/dbN148RBu8YgDhba/C9o3xPOG2yamKbW5O4kBP3dFYYI774HZqfM1QbsmBbUKakKXPcv8johselm1xrGw9H53S8XsCKmIi1viT88ttmQjE9+Sc5siAMkIcF8DGQ1MUdba9MF9+MgMxmmoTyMDdrG7V2ARzeUc4w7inUtxNFnMv4WvYdoUd0dJRVCLBdSdbctQCh2G0iYDphr8szlEavr8Oo0bRGZ7ZVnch5TtQXNTrn1vUi3XXxjKrpp0wrsPYU2AMflJS60= jenkins@cn-office-tonytest-jenkins
```
**2.拷贝公钥到你的gitlab**
登录gitlab-->点击profile settings-->SSH Keys-->打开id_rsa.pub（公钥） 复制全部内容到Key这个选项
**3. 验证**
保证git@gitlab.51sw.cc22端口对密钥生成的服务器开放
```
jenkins@cn-office-tonytest-jenkins:~/test$ git clone git@gitlab.51sw.cc:bw/crm-all.git
Cloning into 'crm-all'...
Updating files: 100% (5742/5742), done.
```
**4. 在jenkins上管理页面配置**
现在是jenkins需要拉取git@gitlab.51sw.cc:bw/crm-all.git，公钥已经放在gitlab了，jenkins上管理页面配置还需要配置下刚才生成的私钥
manage jenkins-->manage credentials-->system-->global credentials-->添加credentials-->拷贝刚才生成的私钥到对应配置项
