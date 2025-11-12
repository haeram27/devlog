# pg_hba.conf File

## reference

* <https://www.postgresql.org/docs/current/auth-pg-hba-conf.html>

## note

pg_hba.conf is used to configure access contol to postgres db
It supports detailed conditions per host type and user.

## formats

syntax:

```plain
local               database  user  auth-method [auth-options]
host                database  user  address     auth-method  [auth-options]
hostssl             database  user  address     auth-method  [auth-options]
hostnossl           database  user  address     auth-method  [auth-options]
hostgssenc          database  user  address     auth-method  [auth-options]
hostnogssenc        database  user  address     auth-method  [auth-options]
host                database  user  IP-address  IP-mask      auth-method  [auth-options]
hostssl             database  user  IP-address  IP-mask      auth-method  [auth-options]
hostnossl           database  user  IP-address  IP-mask      auth-method  [auth-options]
hostgssenc          database  user  IP-address  IP-mask      auth-method  [auth-options]
hostnogssenc        database  user  IP-address  IP-mask      auth-method  [auth-options]
include             file
include_if_exists   file
include_dir         directory
```

simple examples:

```plain
# Allow any user on the local system to connect to any database with
# any database user name using Unix-domain sockets (the default for local
# connections).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust

# The same using local loopback TCP/IP connections.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             127.0.0.1/32            trust

# The same as the previous line, but using a separate netmask column
#
# TYPE  DATABASE        USER            IP-ADDRESS      IP-MASK             METHOD
host    all             all             127.0.0.1       255.255.255.255     trust

# The same over IPv6.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             ::1/128                 trust

# The same using a host name (would typically cover both IPv4 and IPv6).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             localhost               trust
```

## meaning of the fields is as follows

### host type

|field|meaning|
|:---|:---|
|local|This record matches connection attempts using Unix-domain sockets. Without a record of this type, Unix-domain socket connections are disallowed.|
|host|This record matches connection attempts made using TCP/IP. host records match SSL or non-SSL connection attempts as well as GSSAPI encrypted or non-GSSAPI encrypted connection attempts.|

### address

|field|meaning|
|:---|:---|
|IP-address/IP-mask|    These two fields can be used as an alternative to the IP-address/mask-length notation. Instead of specifying the mask length, the actual mask is specified in a separate column. For example, 255.0.0.0 represents an IPv4 CIDR mask length of 8, and 255.255.255.255 represents a CIDR mask length of 32. <br> These fields do not apply to local records.|

### auth-method

|field|meaning|
|:---|:---|
|**trust**|Allow the connection unconditionally. This method allows anyone that can connect to the PostgreSQL database server to login as any PostgreSQL user they wish, without the need for a password or any other authentication. See Section 20.4 for details.|
|reject|Reject the connection unconditionally. This is useful for “filtering out” certain hosts from a group, for example a reject line could block a specific host from connecting, while a later line allows the remaining hosts in a specific network to connect.|
|**peer**|Obtain the client's operating system user name from the operating system and check if it matches the requested database user name. ***This is only available for local connections***.|
password|Require the client to supply an unencrypted password for authentication. Since the password is sent in clear text over the network, this should not be used on untrusted networks.|
|scram-sha-256|Perform SCRAM-SHA-256 authentication to verify the user's password.|
|md5|Perform SCRAM-SHA-256 or MD5 authentication to verify the user's password.|

---

## pg_hba.conf 개요

* 클라이언트가 데이터베이스에 접속하는 주소와 역할을 지정하고 인증하고자 하는 설정 파일입니다
* HBA는 호스트 기반 인증(host-based authentication)의 약어입니다
* 각 연결 시도에 대해 순차적으로 검사되므로 레코드의 순서는 중요합니다

### 설정 형식(설정 필드)

```text
# TYPE    DATABASE    USER    ADDRESS    METHOD
```

### TYPE

* 접속하고자 하는 대상(TYPE)을 크게 로컬(local), 원격지 IPv4 주소, 원격지 IPv6 주소, 복제 장소를 대상으로 설정합니다
* local - 운영체제 local을 의미합니다.
* host - TCP/IP를 사용한 연결 시도하는 주소 형태 입니다.

### DATABASE

* 접속하고자 하는 데이터베이스 이름을 지정합니다
* all 값은 모든 데이터베이스와 일치하는것을 의미합니다
* replication 값은 복제 연결이 요청되는 경우 지정함
  * 복제 연결은 특정 데이터베이스를 지정하지는 않습니다

### USER

* 데이터베이스 사용자 이름을 지정합니다.
* all 값은 모든 사용자와 일치하도록 지정합니다.
* 쉼표로 구분함해서 사용자 이름을 여러 개 쓸 수 있습니다

### ADDRESS

* 접속이 허용되는 client(접속 요청자)의 IP address
* 명시된 주소 범위 이외의 src ip로 부터의 접속 요청은 모두 거부 됩니다
* 단일 호스트의 경우 172.20.143.89/32, 소규모 네트워크 172.20.143.0/24, 대규모 네트워크의 경우 10.6.0.0/16 등입니다
* 0.0.0.0/0은 모든 IPv4 주소를 나타내며 ::/0은 모든 IPv6 주소를 나타냅니다

### METHOD

* 연결 사용하는 인증 방법을 지정합니다 여러 방법이 있으나 중요 내용만 설명합니다
* trust - 무조건 연결을 허용합니다
  * 패스워드나 다른 인증 없이 임의의 데이터베이스 사용자로 로그인하여 누구나 데이터베이스 서버에 연결할 수 있습니다
* reject - 무조건 연결을 거부하는 것으로 특정 호스트 또는 사용자 등을 "필터링" 하여 차단 할 때 유용합니다.
* peer - 로컬 연결에서만 사용 할 수 있는 것으로, 운영 체제 사용자 이름을 데이타베이스 사용자 이름과 일치 시켜 인증 합니다.
  * 예) 'postgres'  계정으로 local에서 superuser 권한 접속
* md5 - 사용자 패스워드 방식 인증입니다. 구버전 방식으로 암호화 알고리즘 취약성이 있어 사용을 지양합니다.
* scram-sha-256 - 사용자 패스워드 방식 인증입니다. sha256 패스워드 방식을 지원해야 합니다

### example

```text
# Database administrative login by Unix domain socket
local    all    postgres    peer
# => 로컬 superuser 접속 설정입니다. DISABLE하지 않습니다.

# "local" is for Unix domain socket connections only
#local  all    all    peer
local   all    all    scram-sha-256
# => 보안상 로컬접속 시 peer접속을 주석처리하고 패스워드로 접속하도록 설정합니다

# IPv4 local connections:
host    all    all    127.0.0.1/32    scram-sha-256
# => 루프백(로컬접속) IP를 허용합니다

host    all    all    0.0.0.0/0    scram-sha-256
# => 모든 IP 접속을 허용합니다.
# => 보안상 데이터베이스는 모든 IP 접속허용을 하지 않는게 일반적입니다

host    all   all    192.168.48.0/24    scram-sha-256
# => 특정 IP 접속을 허용합니다. 1개 IP 또는 범위 형태로 지정합니다

# IPv6 local connections:
host    all    all    ::1/128    scram-sha-256

# replication privilege
local    replication     all    peer
host     replication     all    192.168.48.0/24    scram-sha-256
# => 이중화 구성시 복제를 위한 허용을 지정합니다

```

### 원격 데이터베이스 접속

192.168.48.129 서버에 postgres 계정으로 postgres 데이터베이스에 접속하기

```bash
psql -h 192.168.48.129 -U postgres -d postgres
```
