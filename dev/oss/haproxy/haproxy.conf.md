# haproxy configuration

[haproxy configuration manual](https://www.haproxy.com/documentation/haproxy-configuration-manual/latest/)

## HTTP file dist server configuration

- haproxy-http-fileserv.cfg

```text
global
        log 127.0.0.1 local6
        maxconn 4096
        daemon

defaults
        log global
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
        option dontlognull
        option http-server-close
        option tcplog
        option httplog
        option httpclose
        option forwardfor
        option accept-invalid-http-request

frontend http
        bind *:<frontend-port>
        mode http
        acl files path -i /web/html/list_files.htm
        http-request set-var(req.hostname) req.hdr(Host), field(1,:),lower
#        http-request redirect code 301 location https://%[var(req.hostname)]:/ if files
        default_backend dist-http

backend dist-http
        balance roundrobin
        mode http
        server <host-ip> <host-ip>:<backend-port> check
```

## HTTPS file dist server configuration

- haproxy-https-fileserv.cfg

```text
global
        log 127.0.0.1 local5
        maxconn 4096
#        daemon

        ssl-server-verify none


defaults
        log global
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
        option dontlognull
        option http-server-close
        option tcplog
        option httplog
        option httpclose
        option forwardfor
        option accept-invalid-http-request

frontend http-in
        bind *:<frontend-port> ssl verify none crt /etc/cert/haproxy.pem
        mode http
        default_backend dist-https

backend dist-https
        mode http
        balance roundrobin
        server <host-ip> <host-ip>:<backend-port> check ssl verify none crt /etc/cert/haproxy.pem
```

## WAS configuration

- haproxy-https-lbwas.cfg

```text
global
        log 127.0.0.1 local2
        maxconn 4096
#        daemon

        ssl-default-bind-options force-tlsv12
        ssl-default-bind-ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:!ADH-AES128-GCM-SHA256:!ADH-AES256-GCM-SHA384
        ssl-server-verify none


defaults
        log global
        mode http
        retries 30
        timeout connect 5000ms
        timeout client 5m
        timeout server 5m
        option dontlognull
        option http-server-close
        option tcplog
        option httplog
        option httpclose
        option forwardfor
        option accept-invalid-http-request

frontend http-in
        bind *:<frontend-port> ssl verify none crt /etc/cert/haproxy.pem
        mode http
        http-request deny if METH_TRACE
        http-request redirect scheme https unless { ssl_fc }
        http-response set-header Strict-Transport-Security "max-age=16000000; includeSubDomains; preload;"
        http-response set-header Set-Cookie "HttpOnly; Secure; SameSite=Strict"

        default_backend tomcat-was

backend tomcat-was
        mode http
        balance source
        server <host-ip> <host-ip>:<backend-port> check ssl verify none crt /etc/cert/haproxy.pem
```