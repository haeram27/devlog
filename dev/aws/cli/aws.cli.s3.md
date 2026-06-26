# aws cli - s3 command

## list object

s3 high level command 호출

```bash
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3 ls s3://bucket-name --recursive --debug
```

s3 low api 직접 호출

```bash
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3api list-objects-v2 --bucket bucket-name
```

aws cli command는 prooxy 설정을 참조하므로 정상 환경에서 403 응답 등의 오류가 발생한다면 proxy 설정을 제외하고 명령을 실행해 본다.

```bash
env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy \
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3 ls s3://bucket-name

unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3 ls s3://bucket-name
```

### remove object

특정 object 삭제

```bash
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3 rm s3://bucket-name/invalid-tenant-id/file.zip
```

### 디렉토리 삭제

특정 path 이하 모든 object 삭제

```bash
aws --profile console --endpoint-url=http://1.2.3.4:1234 s3 rm s3://bucket-name/invalid-tenant-id/wallpaper/ --recursive
```