# aws cli installation

[aws user guide - aws cli 설치](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)

## linux

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## config

aws cli 명령을 사용할 환경 설정 및 credential 설정

```bash
aws [--profile PROFILE] [--debug] configure
```

**`~/.aws/config`**

```text
[default]
region = us-east-1
output = json
[profile console]
region = us-east-1
output = json
```

**`~/.aws/credentials`**
```text
[default]
aws_access_key_id = ABCDEF1234567890
aws_secret_access_key = wD1eAksaBD19QLIcoDfi90ANCo9MDuliiDm
[console]
aws_access_key_id = ABCDEF1234567890
aws_secret_access_key = wD1eAksaBD19QLIcoDfi90ANCo9MDuliiDm
```