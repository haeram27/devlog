# aws cli (s3/rgw client)

- MicroCeph 자체에는 파일을 직접 업로드하거나 삭제하는 전용 CLI 명령어가 따로 없습니다.
- 대신 MicroCeph는 S3 호환 인터페이스(RGW)를 제공하므로, s3cmd나 aws-cli와 같은 표준 S3 도구를 사용하여 파일을 관리합니다.

## 1. aws-cli 설치 및 설정

AWS 서비스와 동일한 명령 체계를 사용할 수 있어 범용성이 높습니다. [8] 

- 설치 (공식 권장 v2 버전): [Installing AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## 2. AWS CLI 초기 설정

AWS CLI는 기본적으로 AWS 클라우드를 대상으로 하므로, 명령을 내릴 때마다 --endpoint-url 옵션을 주거나 별도의 프로필을 설정해야 합니다.

```bash
$ aws configure --profile microceph
AWS Access Key ID: (조회한 Access Key 입력)
AWS Secret Access Key: (조회한 Secret Key 입력)
Default region name: default (또는 비워둠)
Default output format: json
```

- Region과 Output format은 기본값(default, json)으로 두어도 무방합니다.

- 실행 시 주의사항:
MicroCeph는 로컬 서버이므로 명령마다 --endpoint-url을 붙여야 합니다.

```bash
aws s3 ls --endpoint-url=http://[서버IP]:80
```

## 3. AWS CLI (aws s3 명령어)

AWS CLI는 기본적으로 AWS 클라우드를 대상으로 하므로, 명령을 내릴 때마다 --endpoint-url 옵션을 주거나 별도의 설정해야 합니다.

가장 표준적으로 사용되는 명령어입니다.

- 버킷 생성 (Make Bucket)

```bash
aws s3 mb s3://버킷이름 --endpoint-url=http://[rgw-ip]:80
```

- 파일 업로드
- 
```bash
aws s3 cp 로컬파일명 s3://버킷이름/ --endpoint-url=http://[rgw-ip]:80
```

- 파일 삭제
- 
```bash
aws s3 rm s3://버킷이름/파일명 --endpoint-url=http://[rgw-ip]:80
```

- 버킷 삭제 (비어있어야 함)

```bash
aws s3 rb s3://버킷이름 --endpoint-url=http://[rgw-ip]:80
```

(버킷 안의 파일까지 한꺼번에 강제 삭제하려면 끝에 --force를 붙입니다.)

## 3. 관리자용 버킷 강제 삭제 (radosgw-admin) [8] 

파일 단위의 일반적인 조작보다는 버킷 전체를 비우거나 강제로 삭제할 때 사용합니다. [9, 10] 

- 버킷 내 모든 객체 포함 삭제:

sudo microceph.radosgw-admin bucket rm --bucket=[버킷이름] --purge-objects

## endpoint-url 기입 우회

안타깝게도 표준 aws configure 프로필 설정에는 '엔드포인트 URL'을 저장하는 공식 항목이 없습니다.

### 1. AWS CLI 플러그인 사용 (가장 추천)

awscli-plugin-endpoint라는 플러그인을 설치하면, 프로필 설정 파일(~/.aws/config)에 엔드포인트를 고정할 수 있습니다.

  1. 플러그인 설치:

```bash
pip install awscli-plugin-endpoint
```

  2. 설정 파일(~/.aws/config) 수정:

```text
[plugins]
endpoint = awscli_plugin_endpoint

[profile microceph]
region = default
s3 =
    endpoint_url = http://RGW_IP:포트
s3api =
    endpoint_url = http://RGW_IP:포트
```

  3. 사용: 이제 --endpoint-url 없이 프로필만 지정하면 됩니다.

```bash
aws s3 ls --profile microceph
```

### 2. 별칭(Alias) 활용 (가장 간편)

플러그인 설치가 번거롭다면, 쉘 설정 파일(.bashrc 또는 .zshrc)에 별칭을 등록하는 것이 가장 대중적인 방법입니다.

  1. .bashrc에 추가:

```bash
alias maws='aws --endpoint-url http://RGW_IP:포트 --profile microceph'
```

  2. 사용:

```bash
maws s3 ls
maws s3 mb s3://my-new-bucket
```

### 3. 환경 변수 활용

특정 터미널 세션 동안만 엔드포인트를 고정하고 싶을 때 유용합니다. (단, 이 방법은 일부 AWS CLI 버전이나 커스텀 스크립트에서 AWS_ENDPOINT_URL 변수를 인식하도록 설정해야 할 수 있습니다.)

```bash
export AWS_ENDPOINT_URL_S3="http://RGW_IP:포트"# 이후 명령 수행 (버전 2.x 일부에서 지원)
```
