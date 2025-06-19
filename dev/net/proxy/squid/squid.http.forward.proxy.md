# squid : http forward proxy server

## official homepage

https://www.squid-cache.org/

## installation

```bash
# squid install: ubuntu
sudo apt-get install -y squid

# squid install: rhel
yum install -y squid

# edit squid configuration
vi /etc/squid/squid.conf

# check running status
systemctl status squid

# run squid
systemctl enable squid
systemctl start squid

# check logs
vi /var/log/squid/access.log

# check proxy works
curl -kvx http://localhost:3128 https://google.com
curl -kvx http://localhost:3128 https://github.com
```

## test configuration

* /etc/squid/squid.conf
  
```bash
# comment all access deny configuration
#http_access deny !Safe_ports
#http_access deny CONNECT !SSL_ports
#http_access allow localhost manager
#http_access deny manager
#http_access deny to_localhost
#http_access deny all
http_access allow all

# set listen port
# Squid normally listens to port 3128
# http_port 3128
http_port 0.0.0.0:3128
```

## access control using ACLs

[ACLs](https://www.squid-cache.org/Versions/v7/cfgman/acl.html)

```bash
# ACL: Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 0.0.0.1-0.255.255.255  # RFC 1122 "this" network (LAN)
acl localnet src 10.0.0.0/8             # RFC 1918 local private network (LAN)
acl localnet src 100.64.0.0/10          # RFC 6598 shared address space (CGN)
acl localnet src 169.254.0.0/16         # RFC 3927 link-local (directly plugged) machines
acl localnet src 172.16.0.0/12          # RFC 1918 local private network (LAN)
acl localnet src 192.168.0.0/16         # RFC 1918 local private network (LAN)
acl localnet src fc00::/7               # RFC 4193 local private network range
acl localnet src fe80::/10              # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT

http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports

http_access allow localhost manager
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all
```

### access control configuration order

1. deny specific acl
2. allow specific acl
3. deny all other acl (`http_access deny all`)

## ssh port-forward to proxy server

* linux:

```bash
[[ -z $(ss -tulnp | fgrep :3128) ]] && ssh -fCN -L 3128:<http.proxy.host.ip>:3128 <gateway.remote.sshd> -o ServerAliveCountMax=10 -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -o TCPKeepAlive=no
```

* windows:

```bash
start /B ssh -fCN -L 3128:<http.proxy.host.ip>:3128 <gateway.remote.sshd> -o ExitOnForwardFailure=yes -o ServerAliveInterval=60 -o ServerAliveCountMax=10
```

## logs

### configuration

```text
cache_access_log /usr/local/squid/logs/access.log
 - 접근로그를 기록하는 파일을 설정한다.
```

```text
cache_log /usr/local/squid/logs/cache.log
 - 캐쉬설정에 관한 로그를 기록하는 파일을 설정한다.
```

```text
cache_store_log /usr/local/squid/logs/store.log
 - 저장되는 로그(이미지, 아이콘 등)를 기록하는 파일을 설정한다.
```

```text
debug_options ALL,1
 - 스퀴드가 동작할 때 오류체크 기능을 사용하여 로그파일에 기록할 수 있게 하는 옵션이다.
 - 현재 설정은 모든 항목에 대해 기본적인 값만 로그에 남도록 설정한 것이다.
```

```text
buffered_logs on
 - 로그 기록시 사용되는 시스템 자원을 절약함으로써 약간의 속도 향상을 기대할 수 있는 옵션이다.
```