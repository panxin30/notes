# 说明
```
root@ali-hn-lw-jenkins:~# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
8.14.4-ee
```
**备份来自8.14.4-ee,现在准备恢复一份在新服务器上，同时新服务器上gitlab使用社区版
EE版本的备份不能直接恢复到CE版本，恢复的时候需要版本相同
因此首先安装8.14.4-ee，恢复数据后，再降级8.14.4-ee到8.14.4-ce**
# ubuntu16.04 安装指定版本gitlab
18.04根据核心名字来选择如xenial该成bionic
```
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ee/ubuntu/pool/xenial/main/g/gitlab-ee/gitlab-ee_8.14.4-ee.0_amd64.deb
dpkg -i gitlab-ee_8.14.4-ee.0_amd64.deb
```
# 头一次安装，配置文件不完善，需要修改
```
vim /etc/gitlab/gitlab.rb
external_url 'http://gitlab.51sw.cc'
 gitlab_rails['lfs_enabled'] = true #大文件上传下载
 gitlab_rails['lfs_storage_path'] = "/data/lfs/lfs-objects"
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_archive_permissions'] = 0644 # See: https://docs.gitlab.com/ce/raketasks/backup_restore.html#backup-archive-permissions
gitlab_rails['backup_pg_schema'] = 'public'
gitlab_rails['backup_keep_time'] = 604800
# 根据需要开启白名单
#gitlab_rails['rack_attack_git_basic_auth'] = {
 #'enabled' => true,
  #'ip_whitelist' => ["127.0.0.1","172.16.108.21","172.16.108.22"],
  #'maxretry' => 10,
  #'findtime' => 60,
  #'bantime' => 3600
#}
unicorn['port'] = 9090
#在gitlab服务器上集成jenkins
#nginx['custom_nginx_config'] = "include /var/opt/gitlab/nginx/conf/jenkins.conf;"
```
# 重新读取配置，同时会启动服务
`gitlab-ctl reconfigure`
# 拷贝备份文件到/var/opt/gitlab/backups目录下
# 导入备份
`gitlab-rake gitlab:backup:restore BACKUP=1393513186`
# 导入备份报错
```
ACCES: Permission denied @rb_file_s_rename
Restoring lfs objects ... 
rake aborted!
Errno::EACCES: Permission denied @ rb_file_s_rename - (/data/lfs/lfs-objects, /data/lfs/lfs.1532511121)
# Solution:
chown -R git.root /data/lfs/
chmod 700 /data/lfs/
```
# 下载gitlab 8.14.4-ce
`wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/xenial/main/g/gitlab-ce/gitlab-ce_8.14.4-ce.0_amd64.deb`
# 停止、卸载8.14.4-ee并安装8.14.4-ce版
# dpkg -r 并不会删除数据和配置文件
```
dpkg -l | grep gitlab
gitlab-ctl stop
gitlab-ctl uninstall
dpkg -r gitlab-ee
dpkg -i gitlab-ce_8.14.4-ce.0_amd64.deb
```
# 重置root密码
```
gitlab-rails console -e production
# 低版本可以尝试使用下面一句命令：
gitlab-rails console production

user=User.where(id:1).first
user.password='test'
user.password_confirmation='test'
user.save!
```

