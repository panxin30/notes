
user  nginx;
worker_processes  2;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


worker_rlimit_nofile 65535;

load_module modules/ngx_http_headers_more_filter_module.so;

events {
    use epoll;
    multi_accept on;
    worker_connections  65535;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    geoip2 /etc/nginx/GeoLite2-Country.mmdb {
        auto_reload 5m;
        $geoip2_metadata_country_build metadata build_epoch;
        $geoip2_data_country_code default=US source=$clientRealIp country iso_code;
        $geoip2_data_country_name country names en;
    }
    map $proxy_add_x_forwarded_for  $clientRealIp {
             ""    $remote_addr;
             ~^(?P<firstAddr>[0-9\.]+),?.*$     $firstAddr;
       }

    log_format access_json

            '$geoip2_data_country_code $clientRealIp $remote_addr $proxy_protocol_addr [$time_local] '
                    '$http_head '
                    '$http_host '
                    '$remote_addr '
                    '$request '
                    '$status '
                    '$body_bytes_sent '
                    '$http_referer '
                    '$request_body '
                    '$request_time '
                    '$upstream_addr '
                    '$upstream_response_time'
                    '[$http_user_agent] "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  access_json;

    sendfile        on;
    #tcp_nopush     on;
    server_tokens  off;
    #resolver 10.143.22.116;
    resolver 10.143.22.116 10.143.22.118 223.5.5.5 223.6.6.6;

    keepalive_timeout  65;

    #gzip  on;
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 3;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/json image/jpeg image/gif image/png;
    gzip_vary on;
    client_max_body_size 50m;
    server_names_hash_max_size 512;
    server_names_hash_bucket_size 1280;

    include /etc/nginx/support.d/*.conf;
    include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/preconf.d/*.conf;
    include /etc/nginx/preprod.d/*.conf;
    #include /etc/nginx/prod.d/*.conf;
    #include /etc/nginx/update.d/*.conf;


    upstream  staticlist{                                                                                
                server   172.19.91.179:8080;
                server   172.19.91.180:8080;
        }

    upstream  bw_ui_upload_oss{
                server 192.168.0.112:10391 weight=5;
                server 192.168.0.113:10391 weight=5;
                server 47.110.252.40:10391 weight=1;
                server 47.99.241.111:10391 weight=1;
        }

    upstream gw_facade{
		server 10.144.100.117:8014;
		server 10.30.74.217:8014;
	}
    upstream  bw_facade{                                                                                
                server   172.19.91.179:10200;
                server   172.19.91.180:10200;
                server   172.25.0.35:10200;
        }
    upstream  prebw_facade{
                server   172.16.179.126:10200;
                server   172.26.0.221:10200;
                server   172.26.0.222:10200;
        }

    upstream  sc_facade{
                server   172.19.91.179:10601;
                server   172.19.91.180:10601;
                server   172.25.0.35:10601;
        }
    upstream  presc_facade{
                server   172.16.179.126:10601;
                server   172.26.0.221:10601;
                server   172.26.0.222:10601;
        }

     upstream  ops_facade{
                server   172.19.91.179:10600;
                server   172.19.91.180:10600;
                server   172.25.0.35:10600;
        }  

     upstream  preops_facade{
                server   172.16.179.126:10600;
                server   172.26.0.221:10600;
                server   172.26.0.222:10600;
        }  

     upstream  bw_openapi{
                server   172.19.91.179:10500;
                server   172.19.91.180:10500;
        }

}
