# nginx configuration

## ref

<https://nginx.org/en/docs/http/ngx_http_core_module.html>

## example

```text
user  mynginx;
worker_processes  2;

error_log  /log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  128;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  65;

    set_real_ip_from 127.0.0.1;

    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    #include /etc/nginx/conf.d/http-server.conf;
    server {
        listen                        8080;
        listen                   [::]:8080;
        server_name              my_http_server;

        limit_rate 2m;

        #include /etc/nginx/conf.d/locations;
        location / {
            root    /http-file-server/public;
            index   index.htm index.html index.xhtml default.htm index.php;
            autoindex off;
        }
        #-- include

        #include /etc/nginx/conf.d/locations_deploy;
        location /download/update/ses {
            root    /http-file-server/deploy;
            #include /etc/nginx/conf.d/accessip;
            autoindex off;
        }
        #-- include

        #include /etc/nginx/conf.d/custom-http-error-page;
        error_page
            400 401 402 403 404 405 406 408 409 410 411 412 413 414 415 416 421 429 497
            500 501 502 503 504 505 507
            /error.html;
        location = /error.html {
                internal;
                alias /etc/nginx/html/error.html;
        }
        #-- include 
    }
    #-- include

    #include /etc/nginx/conf.d/https-server.conf;
    server {
        listen                       8081 ssl;
        listen                  [::]:8081 ssl;
        server_name             my_https_server;

        limit_rate 2m;

        ssl_certificate         /etc/cert/cacert.pem;
        ssl_certificate_key     /etc/cert/cacert.pem;
        ssl_protocols           TLSV1.2;
        ssl_ciphers             ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256;

        include /etc/nginx/conf.d/locations;
        include /etc/nginx/conf.d/locations_deploy;
        include /etc/nginx/conf.d/custom-http-error-page;
    }
}
```
