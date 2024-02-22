参考:https://docs.gitlab.com/omnibus/settings/ssl.html 直接使用 Let’s Encrypt自动颁发证书
参考:https://blog.csdn.net/asen_zh/article/details/117020531
# 环境：OS:Ubuntu 18.04.3 LTS Gitlab 14.6.1
# 一 、安装和配置必要依赖
```
sudo apt update
# sudo apt install -y curl openssh-server ca-certificates  可选择
```
# 二、添加GitLab软件包存储库并安装软件包
添加GitLab软件包存储库，可以切换为国内源，例如：gitlab-ce清华源。

`curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash`
确保你要使用的gitlab域名已正确设置DNS
`sudo EXTERNAL_URL="http://gitlab.test.cn" apt install gitlab-ce`
安装将自动配置并在该URL启动GitLab。默认安装最新版本

对于用https://URL，GitLab将通过Let’s Encrypt自动请求证书，这需要入站HTTP访问和有效的主机名。








# 附录
主配置文件/etc/gitlab/gitlab.rb
包含的nginx，redis，数据库等，要改配置生效的话，都从gitlab.rb里面改
gitlab-ctl tail #查看日志是否有报错
gitlab-ctl status 查看状态

gitlab-ctl reconfigure 加载配置文件，修改后必须执行
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION 查看gitlab版本

gitlab  为安装一体化，内置nginx/redis/postgresql等

			 gitlab    主程序目录：    /var/opt/gitlab/
						   主配置文件：  /var/opt/gitlab/gitlab-shell/config.yml
			 gitlab    认证文件：    /var/opt/gitlab/.ssh/authorized_keys
						  日志文件：   /var/log/gitlab/gitlab-shell/gitlab-shell.log  

			 nginx 配置文件 ：  /var/opt/gitlab/nginx/conf/nginx.conf
										  /var/opt/gitlab/nginx/conf/gitlab-http.conf
                                         

3、所有与gitlab相关服务统一通过gitlab来启动     
			gitlab:  启动、关闭、重启方式：    gitlab-ctl  start | stop | restart


# 给gitlab添加SSHkey

如果某个人想要克隆你的代码，就需要把他的公钥加入到gitlab中
```
# 在某个开发人员的机器上：
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
# 将id_rsa.pub文件内容全部复制
# 浏览器登录gitlab，进入gitlab
# Profile Settings-->SSH Keys--->Add SSH Key
#粘贴你复制的公钥，点击add按钮。
```

# 错误锦集
一、测试`sudo EXTERNAL_URL="https://gitlab.test.cn" apt install gitlab-ce` Let’s Encrypt自动颁发证书，报错
```
I ran this command: plesk bin extension --exec letsencrypt cli.php -d[mircoblitz.de5](http://mircoblitz.de/)\-d[www.mircoblitz.de](http://www.mircoblitz.de/)\--email[webmaster@lindworm.de](mailto:webmaster@lindworm.de)(but thats a wrapper for certbot)

It produced this output:  
\[2020-03-24 16:44:55.643\] ERR \[extension/letsencrypt\] Domain validation failed for[www.mircoblitz.de](http://www.mircoblitz.de/): Invalid response from[https://acme-v02.api.letsencrypt.org/acme/authz-v3/35420755145](https://acme-v02.api.letsencrypt.org/acme/authz-v3/3542075514).  
Details:  
Type: urn:ietf:params:acme:error:connection  
Status: 400  
Detail: Fetching[http://mircoblitz.de/.well-known/acme-challenge/sehiMs9DjRbD46dLs\_zYO5oKLa1ITkfrK\_SaHwt-VJE:19](http://mircoblitz.de/.well-known/acme-challenge/sehiMs9DjRbD46dLs_zYO5oKLa1ITkfrK_SaHwt-VJE:)Error getting validation data
```
错误原因：Your IPv6 address rejects connections:
解决办法：实验机器是腾讯云ubuntu18.04，在安全组添加了ipv6开通端口80和443。不知道禁用ipv6可行不？

# 修改密码
```
root@tony-test-gitlab:~# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

```