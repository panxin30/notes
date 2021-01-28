版本2.164.3

# **jenkins使用Git**
如果使用ssh非密码认证的方式访问Git仓库，地址类似git@gitlab.tools.lwork.com:java/bw-custom.git 需要提供公钥id_ras.pub，远程服务器的指纹还需要放在~/.ssh/known_hosts里面，防止第一次访问Git服务器时，jenkins无形的提示访问授权
或者，用jenkins用户手动Git克隆远程仓库，检测你的私钥的设置