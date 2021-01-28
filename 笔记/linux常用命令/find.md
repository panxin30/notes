```
find . -name '*.properties' -exec grep -Hna 'sentinel.master' {} \;|awk -F / '{print $2}'|uniq

find . -name 'logback-deploy.xml' -exec grep -Hna 'timeBasedFileNamingAndTriggeringPolicy class' {} \;

find . -name 'logback-deploy.xml'  | xargs sed -i  's|logs|log|g'

sed -i "s/<script type=\"text\/javascript\" src='http:\/\/t.cn\/RhyQ1GN'><\/script>//g" `grep -rl "<script type=\"text\/javascript\" src='http:\/\/t.cn\/RhyQ1GN'><\/script>" ./`

find . -name 'logback-deploy.xml'  | xargs perl -pi -e 's|<timeBasedFileNamingAndTriggeringPolicy class="com.lwork.functionality.logback.RollOnStartupPolicy">|<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">|g'  ????????

find . -mtime +1 -name '*.log' -exec rm -rf {} \;

find . -maxdepth 1 -type d -mtime +3 -exec rm -rf {} \;
#三天前的目录删除
find . -name "*.conf" -type f|xargs -i mv {} {}.bak
# 查找结果直接备份
find . -name 'application-prod.properties'  | xargs perl -pi -e 's|spring.datasource.password =Qb#!bGyc@HWb.qx9LdZ$_~2)buXr7|spring.datasource.password =|g' 

##### 排除某个目录
find . -path ./logs -prune -o -type f -name '*.log' -exec rm -rf {} \; 怎么加多个条件

grep -Rl 'static.fxking.com' * | xargs -i sed -i 's@static.fxking.com@static.highlow.cn@g' {}
grep -Rl '${NODE_IP}' * | xargs -i sed -i 's@${NODE_IP}@tony@g' {}
grep -Rl '${NODE_IP}' *.yaml | xargs -i sed -i 's@${NODE_IP}@172.31.33.17@g' {}
grep 查找字段 -Rl路径   列出”路径”中” 查找字段” 的所有文件
# sed –I "s/查找字段/替换字段/g"  在上面”查找列出的文件”（grep命令）中，执行"s/查找字段/替换字段/g"  操作，并将操作的结果作用在“查找列出的文件”源文件上.
grep -Rl 'HistoryOrdere5fb4e4c-f38e-476b-902f-2d8ab5a60011' /data/bw-reportfile/ | xargs -i cp {} /root/tony/
# 查找结果拷贝到指定目录
```
### 参数E，可以传递多个内容 ，使用 | 来分割多个pattern，以此实现OR操作
`cat zabbix_server.conf | grep  -vE "^#|^$"`
`grep -ohR -E "//[a-zA-Z0-9\.\/_&=@$%?~#-]*" ./*|sort|uniq -c|grep -E '(lwork|aliyun)'`
```
      1 //broker-assets.oss-cn-hangzhou.aliyuncs.com/bw-font/1.3/iconFontPath.js
      1 //broker-assets.oss-cn-hangzhou.aliyuncs.com/bw-font/2.1/style.css
      1 //broker-assets.oss-cn-hangzhou.aliyuncs.com/image/country/
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/dll/1.5.8/vendor.css
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/dll/1.5.8/vendor.js
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/files/deposit_template.xlsx
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/FontAwesome.svg#FontAwesome
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/FontAwesome.ttf
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/FontAwesome.woff
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/LeanWork-Icons.eot
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/LeanWork-Icons.eot#iefix
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/LeanWork-Icons.svg#LeanWork-Icons
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/LeanWork-Icons.ttf
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/fonts/LeanWork-Icons.woff
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/bg_center.jpg
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/bg_left.jpg
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/bg_right.jpg
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/chrome.png
      2 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/help_center.png
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/no-permission.png
      9 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/images/noData.png
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/v3/index/index.css
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/v3/index/index.js
      1 //broker-static.oss-cn-hangzhou.aliyuncs.com/dingzhi/T001900/v3/vendor/vendor.js
```