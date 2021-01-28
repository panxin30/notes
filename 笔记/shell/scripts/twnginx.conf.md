```
user nginx;
worker_processes 4;
worker_rlimit_nofile 65535;

events {
        use epoll;
        worker_connections 65535;
}

http {

    limit_req_zone $cookie_token zone=session_limit:30m rate=1r/s;
    limit_req_zone $clientRealIp$uri zone=auth_limit:30m rate=4r/m;
    resolver 10.143.22.116 10.143.22.118 8.8.8.8 8.8.4.4 valid=60s;
    geoip2 /etc/GeoLite2-Country.mmdb {
        auto_reload 5m;
        $geoip2_metadata_country_build metadata build_epoch;
        $geoip2_data_country_code default=US source=$clientRealIp country iso_code;
        $geoip2_data_country_name country names en;
    }

    geoip_country /usr/local/nginx/conf/GeoIP.dat;
    vhost_traffic_status_zone shared:vhost_traffic_status:512m;
#	vhost_traffic_status_filter_by_set_key $geoip_country_code Country::*;
    #    vhost_traffic_status_filter_by_set_key $clientRealIp|$host|$status|$geoip2_data_country_code Host::$host;
	vhost_traffic_status_filter_by_host on;
    map $http_x_forwarded_for  $clientRealIp {
            ""      $remote_addr;
            ~^(?P<firstAddr>[0-9\.]+),?.*$  $firstAddr;
    }
    log_format  main  
        '$clientRealIp $remote_addr $proxy_protocol_addr [$time_local] '
                    '$http_head '
                    '$http_host '
                    '$remote_addr '
                    '$request '
                    '$status '
                    '$body_bytes_sent '
                    '$http_referer '
                    '$uri '
                    '$request_time '
                    '$upstream_addr '
                    '$upstream_response_time'
                    '"X-Api-tenantId:$http_x_api_tenantId" '
                     '"X-Api-Token:$http_x_api_token" '
		    # '"$uri" '
		     '"X-tenant-ID:$http_x_tenant_id" '
                    '[$http_user_agent] "$http_x_forwarded_for" "$geoip2_data_country_code"';
        #add_header X-Frame-Options SAMEORIGIN;
        add_header X-Frame-Options "Allow-From https://www.growingio.com";
	sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        server_tokens off;
	server_names_hash_bucket_size 364;
        proxy_connect_timeout      600;
        proxy_send_timeout         600;
        proxy_read_timeout         600;
        proxy_buffer_size          4k;
        proxy_buffers              8 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        client_body_buffer_size    128k;
        client_header_buffer_size 4k;
        client_max_body_size 30m;
        large_client_header_buffers 2 2k;
        include mime.types;
        default_type application/octet-stream;
	fastcgi_intercept_errors on; 


        access_log /var/log/nginx/access.log main;
        error_log /var/log/nginx/error.log;


        gzip on;
        gzip_disable "msie6";




    	include /usr/local/nginx/conf/sites-enabled/*.conf;
    	include /usr/local/nginx/conf/conf.d/*.conf;
}
```
