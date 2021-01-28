```
server {
   listen 80 ;
    server_name support.test.com;

location /{
                 rewrite ^(.*) https://support.test.com$1 permanent;
        }

    error_page  404              /404.html;
        location = /404.html {
        proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
        proxy_pass http://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/404.html;
        proxy_redirect off;
        }

    location /images/bg_5XX.png {
            proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
            proxy_pass http://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/images/bg_5XX.png;
            proxy_redirect off;
    }

    error_page   403 500 502 503 504 http://$host/error.html?sc=$status;
    location /error.html {
            proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
            proxy_pass http://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/5XX.html$is_args$args;
            proxy_redirect off;
    }
}

server {
    listen       443;
    server_name support.test.com;

    ssl on;
    ssl_certificate /etc/ssl/support.test.com.crt;
    ssl_certificate_key /etc/ssl/support.test.com.key;
    ssl_session_timeout 5m;

    ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
    ssl_ciphers HIGH:!aNULL:!MD5:!EXPORT56:!EXP;
    ssl_prefer_server_ciphers on;

        set $tenant $http_x_Tenant_ID;

        set $server presc_facade;
        set $pre_scui v6.14.2;
        set $ui_version $pre_scui;

        if ($tenant ~* (T001152|T001318|T001160|T001474|T001413|T000004|T001499|T001256|T001661|T001703|T001774|T001825|T001660|T001645|T001639|T001629|T001627|T000136|T001459|T001526|T001600|T001355|T001311|T000112|T001482|T001705|T001835)) {
        set $server sc_facade;
        set $prod_scui v6.12.6;
        set $ui_version $prod_scui;
        }

        if ($request_uri ~* (T001152|T001318|T001160|T001474|T001413|T000004|T001499|T001256|T001661|T001703|T001774|T001825|T001660|T001645|T001639|T001629|T001627|T000136|T001459|T001526|T001600|T001355|T001311|T000112|T001482|T001705|T001835)) {
        set $server sc_facade;
        set $prod_scui v6.12.6;
        set $ui_version $prod_scui;
        }
    
    location =/ {
        proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
        proxy_redirect off;
      }

    location ^~ /home {
    rewrite /home(.*)$  /prod/dist/$ui_version/index/index.html break;
    proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
    proxy_redirect off;
    }

    location ^~ /preview {
    rewrite /preview(.*)$  /prod/dist/$ui_version/index/index.html break;
    proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
    proxy_redirect off;
    }

    location ^~ /bridge {
    rewrite /bridge(.*)$  /prod/dist/$ui_version/bridge/index.html break;
    proxy_pass http://static-sc.test.com/prod/dist/$ui_version/bridge/index.html;
    proxy_redirect off;
    }

    location ^~ /forgotPassword {
        proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
        proxy_redirect off;
      }

    location ^~ /modifyPassword {
        proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
        proxy_redirect off;
      }


        location ^~ /api/v1/static/
                {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_connect_timeout   60s;
        proxy_send_timeout      60s;
        proxy_read_timeout      60s;
        proxy_pass http://staticlist/v1/static/;
        proxy_redirect off;
                }
    
        location ^~ /api/gwfacade/
            {
              proxy_pass http://gw_facade;
              proxy_set_header   Host             $host;
              proxy_set_header   X-Real-IP        $remote_addr;
              proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
              proxy_connect_timeout   60s;
              proxy_send_timeout      60s;
              proxy_read_timeout      60s;
              proxy_redirect off;
            }

    location ^~ /login {
        proxy_pass http://static-sc.test.com/prod/dist/$ui_version/index/index.html;
        proxy_redirect off;
      }

    location ^~ / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

         if ($request_uri ~* (static|reset|payment|fwproduct|forget|fee|gwconfig|tradingSignal|api|v)){
             proxy_pass        http://$server;

          }

          }


 error_page  404              /404.html;
        location = /404.html {
        proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
        proxy_pass https://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/404.html;
        proxy_redirect off;
        }
location /images/bg_5XX.png {
            proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
            proxy_pass https://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/images/bg_5XX.png;
            proxy_redirect off;
    }
    error_page   500 502 503 504 http://$host/error.html?sc=$status;
    location /error.html {
        proxy_pass https://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/5XX.html$is_args$args;
    }    
   

    }   
```
