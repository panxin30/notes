### **location配置用于匹配请求的URL，即ngnix中的$request_uri变量**
**1.location配置格式**：
`location [ 空格 | = | ~ | ~* |^~|!~ | !~* ] /uri/ {}`
**2.loacation匹配顺序**
location = /uri 　　　精确匹配，只有完全匹配上才能生效。匹配后不再向下匹配
location ^~ /uri 　　  对URL路径进行前缀匹配，并且在正则之前。普通字符匹配,如果匹配则不再向下匹配
`http://www.test.com/uri`
location ~ pattern 　 区分大小写的正则匹配。
location ~* pattern 　不区分大小写的正则匹配。
location /uri 　　　　不带任何修饰符，也表示前缀匹配，但是在正则匹配之后，如果没有正则命中，命中最长的规则。
location / 　　　　　通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中的default。
匹配的搜索顺序优先级为：

~~~
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)
~~~
**3.location是否以"/"结尾**
1.  没有"/"结尾时，location/abc/def可以匹配/abc/defghi请求，也可以匹配/abc/def/ghi等
2.  而有"/"结尾时，location/abc/def/不能匹配/abc/defghi请求，只能匹配/abc/def/anything这样的请求

**4.proxy_pass是否以"/"结尾**
**在nginx中配置proxy_pass时，当在后面的url加上了/，相当于是绝对路径，则nginx不会把location中匹配的路径部分加入代理uri；如果没有/，则会把匹配的路径部分加入代理uri。**
-------------------------------------------------------------------------------------------------------------------------
**网页是个单页面，访问网页index以外的路径或者写错路径，都rewrite给index.html**
```
     location ^~ / {
        rewrite /(.*)$  /$ui_dir/dist2/$ui_version/index/index.html break;
        proxy_pass $ui_url/dist2/$ui_version/index/index.html;
        more_clear_headers 'Last-Modified';
        proxy_redirect off;
    }
```
# 案例
实验案例
测试"^~" 和 “~”。浏览器输入http://localhost/helloworld/test，返回601。如将#1注释，#2打开，浏览器输入http://localhost/helloworld/test，返回603。注：#1和#2不能同时打开
```
location ^~ /helloworld { #1 
	return 601;
} 
#location /helloworld { #2 
# 	return 602;
#}
 location ~ /helloworld { 
 		return 603;
  } 
```
*   测试正则表达式的顺序（正则匹配与顺序相关）。浏览器输入http://localhost/helloworld/test/a.html，返回602；将#2和#3调换顺序，浏览器输入http://localhost/helloworld/test/a.html，返回603
```
location /helloworld/test/ { #1
return 601;
}
location ~ /helloworld { #2
return 602;
}
location ~ /helloworld/test { #3
return 603;
}
```
测试普通字符串的长短（普通字符串的匹配与顺序无关，与匹配度有关)