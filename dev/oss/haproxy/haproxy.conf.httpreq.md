#

## [http-request](https://www.haproxy.com/documentation/haproxy-configuration-manual/latest/#4.2-http-request)

Access control for Layer 7 requests

- syntax

```text
http-request <action> [options...] [ { if | unless } <condition> ]
```

`<action>`: [Action keyword matrix](https://www.haproxy.com/documentation/haproxy-configuration-manual/latest/#4.3)