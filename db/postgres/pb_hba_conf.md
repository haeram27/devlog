# pg_hba.conf File

## reference
* https://www.postgresql.org/docs/current/auth-pg-hba-conf.html

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

## meaning of the fields is as follows:
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
