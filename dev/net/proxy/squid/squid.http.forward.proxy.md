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

ACL(Access Control List)은 Access Control의 대상을 표현하는 rule이다.
ip, port를 베이스로 대상을 지정할 수 있으며, acl의 묶음 또한 하나의 acl이 될 수 있다.

acl 지시자를 사용한 ACL 선언은 단지 조건을 정의하는 것이며, 어떤 것도 차단하거나 허용하지 않는다.
http_access 등의 명령에서 ACL을 사용해야 실제 효과가 있다.


ex: ACL 선언 및 http_access로 등록

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

### request와 acl의 매칭 순서

http_access 명령으로 등록된 ACL의 순서가 중요하다.
squid는 접속 요청이 왔을 때, ACL이 등록된 순서에 따라 매칭을 시도하기 때문이다.
http_access 규칙은 등록된 순서대로 매칭 시도되며, 처음 매칭되는 규칙이 실행되고 이후 규칙은 무시된다.

Squid는 `위에서 아래로 차례대로` http_access 규칙을 검사한다.
`처음으로 조건에 부합하는 룰`이 발견되면, 그에 따라 허용(allow) 또는 차단(deny)되고, `이후 규칙은 무시`됩니다.

### access control configuration order

일반적으로 테스트 상황이 아닌 실제 서비스 환경에서는 http_access 사용시 deny를 먼저, allow를 나중에, 마지막은 deny all로 설정하는 방식이 보편적입니다.

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
