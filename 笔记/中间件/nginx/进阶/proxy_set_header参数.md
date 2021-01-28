#  **允许重新定义或者添加发往后端服务器的请求头。**

**value可以包含文本、变量或者它们的组合。 当且仅当当前配置级别中没有定义proxy_set_header指令时，会从上面的级别继承配置**
```
118.112.177.203 |                                                            $clientRealIp
172.17.128.185 |                                                               $remote_addr
[12/Mar/2020:15:38:41 +0800] |                                 [$time_local]                                                                               
sunny2.bw.brokerwork.net |                                      $http_host
GET /favicon.ico HTTP/1.1 |                                        $request
200 |                                                                                       $status
https://sunny2.bw.brokerwork.net/ |                     $http_referer
/ |                                                                                             $uri
0.006 |                                                                                    $request_time
172.18.164.160:31102 |                                                  $upstream_addr
0.006 |                                                                                    $upstream_response_time
"X-Remote-Port:36282" |                                              "X-Remote-Port:$http_x_remote_port"
"X-Proxy-Port:-" |                                                              "X-Proxy-Port:$http_x_proxy_port"
"X-tenant-ID:-" |                                                                "X-tenant-ID:$http_x_tenant_id"
"HEADER-API-TOKEN:T000000203c72426-0293-454e-833b-f1ee280f0550" |  "HEADER-API-TOKEN:$http_x_api_token" 自定义
[Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36] |  [$http_user_agent]
"118.112.177.203" |                                                         $http_x_forwarded_for
"CN"|                                                                                      $geoip2_data_country_code
```

proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Real-Port $remote_port;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
三个header分别表示：
X-Real-IP            客户端或上一级代理ip
X-Real-Port          客户端或上一级端口
X-Forwarded-For      包含了客户端和各级代理ip的完整ip链路
其中X-Real-IP是必需的，后两项选填。当只存在一级nginx代理的时候X-Real-IP和X-Forwarded-For是一致的，而当存在多级代理的时候，X-Forwarded-For 就变成了如下形式
![](images/screenshot_1584064360394.png)
X-Forwarded-For: 客户端ip， 一级代理ip， 二级代理ip...
在获取客户端ip的过程中虽然X-Forwarded-For是选填的，但是个人建议还是保留这，以便出现安全问题的时候，可以根据日志文件回溯来源。
### 举个例子，有一个web应用，在它之前通过了两个nginx转发，www.kevin.com 即用户访问该web通过两台nginx。
`在第一台nginx中,使用`

`proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;`

`现在的$proxy_add_x_forwarded_for变量的``"X-Forwarded-For"``部分是空的，所以只有$remote_addr，而$remote_addr的值是用户的ip，于是赋值以后，`

`X-Forwarded-For变量的值就是用户的真实的ip地址了。`

`到了第二台nginx，使用`

`proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;`

`现在的$proxy_add_x_forwarded_for变量，X-Forwarded-For部分包含的是用户的真实ip，$remote_addr部分的值是上一台nginx的ip地址，`

`于是通过这个赋值以后现在的X-Forwarded-For的值就变成了``"用户的真实ip，第一台nginx的ip"`
最前面的nginx的$remote_addr（客户端IP）获取的是真实的客户IP地址，而后面两层nginx后端服务器获取的都是自己上一层nginx的IP。然后$host(用户访问的主机或域名)也是如此。