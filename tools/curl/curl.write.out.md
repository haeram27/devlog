# curl write-out option

## refs

- manpage --write-out
- [curl offical doc: writeout](https://everything.curl.dev/usingcurl/verbose/writeout.html)
- [사용 가능한 변수 목록](https://everything.curl.dev/usingcurl/verbose/writeout.html#available---write-out-variables)

## write-out (w) option

- curl은 통신 과정에 사용된 정보를 --write-out(-w) 옵션을 이용해서 stdout으로 출력할 수 있다.

- 주로 HTTP 응답코드나 redicrected url, 연결 시간 등의 정보를 획득하는데 사용한다. 

- 출력 형식은 메세지 문자열을 정의하면 되며, 문자열에 사전 정의된 변수를 참조할 수 있다.

- 변수 참조 형식은 `%{varnmae}` 이다.

- 출력 메세지를 파일에 작성하고 -w 옵션에 파일이름을 지정할 수 있다. 파일 지정 방식은 @을 사용한다. 예: `-w "@filename"`

- 현재 curl에서 출력할 수 있는 변수에 대해서는 `%{json}` 변수를 사용해서 출력해 보면 좋다. 별도로 문서를 찾지 않아도 유용한 변수 값을 최대한 출력해 준다.

### 출력 형식

문자열 메세지에 변수를 참조형식을 사용할 수 있다.

```bash
curl -s -o /dev/null -w "response_code: %{response_code}"  https://www.example.com
```

### 예제 1

```bash
curl -fksSL --connect-timeout 2 -o /dev/null -w "response_code: %{response_code}"  https://www.example.com
```

```text
200
```

- response_code 는 HTTP 응답 코드 이다.

### 예제 2

```bash
curl -fksSL --connect-timeout 2 -o /dev/null -w "%{json}"  https://www.example.com | jq .response_code
```

```text
200
```

## 예제 3

```bash
curl -fksSL --connect-timeout 2 -o /dev/null -w "%{json}"  https://www.example.com | jq .
```

```json
{
  "content_type": "text/html",
  "errormsg": null,
  "exitcode": 0,
  "filename_effective": "/dev/null",
  "ftp_entry_path": null,
  "http_code": 200,
  "http_connect": 0,
  "http_version": "2",
  "local_ip": "192.168.10.1",
  "local_port": 33618,
  "method": "GET",
  "num_connects": 1,
  "num_headers": 8,
  "num_redirects": 0,
  "proxy_ssl_verify_result": 0,
  "redirect_url": null,
  "referer": null,
  "remote_ip": "203.251.233.170",
  "remote_port": 443,
  "response_code": 200,
  "scheme": "HTTPS",
  "size_download": 1256,
  "size_header": 309,
  "size_request": 77,
  "size_upload": 0,
  "speed_download": 63723,
  "speed_upload": 0,
  "ssl_verify_result": 20,
  "time_appconnect": 0.01592,
  "time_connect": 0.005963,
  "time_namelookup": 0.00335,
  "time_pretransfer": 0.016109,
  "time_redirect": 0,
  "time_starttransfer": 0.019624,
  "time_total": 0.01971,
  "url": "https://www.example.com",
  "url_effective": "https://www.example.com/",
  "urlnum": 0,
  "curl_version": "libcurl/7.81.0 OpenSSL/3.0.2 zlib/1.2.11 brotli/1.0.9 zstd/1.4.8 libidn2/2.3.2 libpsl/0.21.0 (+libidn2/2.3.2) libssh/0.9.6/openssl/zlib nghttp2/1.43.0 librtmp/2.3 OpenLDAP/2.5.19"
}
```

### 예제 4

```bash
curl -fksSL --connect-timeout 2 -o /dev/null -w "@messgefile"  https://www.example.com
```

```text
content_type: text/html; charset=utf-8
errormsg:
exitcode: 0
filename_effective: /dev/null
ftp_entry_path:
http_code: 200
http_connect: 000
http_version: 2
local_ip: 192.168.10.1
local_port: 40170
method: GET
num_connects: 1
num_headers: 18
num_redirects: 0
proxy_ssl_verify_result: 0
redirect_url:
referer:
remote_ip: 20.200.245.247
remote_port: 443
response_code: 200
scheme: HTTPS
size_download: 293533
size_header: 4949
size_request: 72
size_upload: 0
speed_download: 2419075
speed_upload: 0
ssl_verify_result: 20
time_appconnect: 0.099858
time_connect: 0.082347
time_namelookup: 0.078217
time_pretransfer: 0.100126
time_redirect: 0.000000
time_starttransfer: 0.105768
time_total: 0.121341
url: https://github.com
url_effective: https://github.com/
urlnum: 0
```

messagefile 내용:

```txt
content_type: %{content_type} \n
errormsg: %{errormsg} \n
exitcode: %{exitcode} \n
filename_effective: %{filename_effective} \n
ftp_entry_path: %{ftp_entry_path} \n
http_code: %{http_code} \n
http_connect: %{http_connect} \n
http_version: %{http_version} \n
local_ip: %{local_ip} \n
local_port: %{local_port} \n
method: %{method} \n
num_connects: %{num_connects} \n
num_headers: %{num_headers} \n
num_redirects: %{num_redirects} \n
proxy_ssl_verify_result: %{proxy_ssl_verify_result} \n
redirect_url: %{redirect_url} \n
referer: %{referer} \n
remote_ip: %{remote_ip} \n
remote_port: %{remote_port} \n
response_code: %{response_code} \n
scheme: %{scheme} \n
size_download: %{size_download} \n
size_header: %{size_header} \n
size_request: %{size_request} \n
size_upload: %{size_upload} \n
speed_download: %{speed_download} \n
speed_upload: %{speed_upload} \n
ssl_verify_result: %{ssl_verify_result} \n
time_appconnect: %{time_appconnect} \n
time_connect: %{time_connect} \n
time_namelookup: %{time_namelookup} \n
time_pretransfer: %{time_pretransfer} \n
time_redirect: %{time_redirect} \n
time_starttransfer: %{time_starttransfer} \n
time_total: %{time_total} \n
url: %{url} \n
url_effective: %{url_effective} \n
urlnum: %{urlnum} \n
```
