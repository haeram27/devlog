# microceph 사용자 계정 생성

- MicroCeph 설치 시 RGW(S3)용 기본 사용자는 미리 정의되어 있지 않습니다.
- 클러스터 관리용 client.admin 계정은 존재하지만, 이는 S3 API 접속용이 아닌 내부 관리용입니다
- S3(Object Storage) 서비스를 이용하려면 반드시 사용자를 직접 생성해야 합니다. [MicroCeph 공식 가이드](https://canonical-microceph.readthedocs-hosted.com/stable/tutorial/get-started/)에 따른 생성 및 키 확인 방법은 다음과 같습니다.

## rgw user command 목록

- commands:
  - user create - create a new user
  - user modify - modify user
  - user info - get user info
  - user rename - rename user
  - user rm - remove user
  - user suspend - suspend a user
  - user enable - re-enable user after suspension
  - user check - check user info
  - user stats - show user stats as accounted by quota subsystem
  - user list - list users

## 1. rgw 사용3자 목록 확인

```bash
sudo radosgw-admin user list
sudo radosgw-admin user info --uid="myuser"
```

## 2. rgw 사용자 생성 및 키 확인

radosgw-admin 명령어를 사용하여 사용자를 생성하면, 실행 결과로 Access Key와 Secret Key가 포함된 JSON 데이터가 바로 출력됩니다.

radosgw-admin 명령어는 `microceph.radosgw-admin` 명령의 alias 입니다.

```bash
$ ls -al /snap/bin/radosgw-admin
lrwxrwxrwx 1 root root ? 23 2026-05-11 11:58 radosgw-admin -> microceph.radosgw-admin*
```

- 사용자 생성:

```bash
# 일반 사용자: 본인이 생성한 버킷에만 접근 가능
sudo radosgw-admin user create --uid=myuser --display-name="My S3 User"

# 관리자 사용자: rgw의 Admin Ops API에 접근 가능(대시보드 관리, 시스템 운영)
sudo radosgw-admin user create --uid=rgw-admin-ops-user --display-name="RGW Admin Ops User" --caps="buckets=*;users=*;usage=read;metadata=read;zone=read" --rgw-zonegroup=default --rgw-zone=default
```

- caps 항목
  - `buckets=*`: 모든 버킷 관리 권한
  - `users=*`: 모든 사용자 관리 권한
  - `usage=read`,` metadata=read` 등: 사용량 및 메타데이터 조회 권한

- 출력 내용 예시:

```json
{
    "user_id": "myuser",
    "display_name": "My S3 User",
    "keys": [
        {
            "access_key": "ABCDEFGHIJKLMNOPQRST",
            "secret_key": "abcdefghijklmnopqrstuvwxyz0123456789"
        }
    ],
    ...
}
```

이 출력 내용 중 access_key와 secret_key를 복사해서 보관하거나 s3cmd/aws-cli 설정에 사용하세요.

### 일반 사용자와 관리자 계정 비교

| 구분 | myuser | rgw-admin-ops-user |
|---|---|---|
| 사용자 성격 | 일반 유저 (데이터 저장용) | 관리 유저 (운영/모니터링용) |
| 타인 버킷 접근 | 불가능 (본인 것만 가능) | 관리 권한(caps)을 통해 가능 |
| Admin API 사용 | 불가 | 가능 |

- S3의 보안 원칙: S3(및 RGW)는 기본적으로 "거부(Deny)" 정책을 따릅니다. myuser는 일반 사용자이므로, 본인이 소유하지 않은 다른 사용자의 버킷에는 접근 권한이 없습니다.
- 접근하려면?: 다른 사용자가 버킷 정책(Bucket Policy)이나 ACL을 통해 myuser에게 명시적으로 권한을 허용해 주어야만 접근이 가능합니다.
- 반면 rgw-admin-ops-user는?: 부여된 caps 덕분에 관리 API를 통해 시스템 내의 모든 버킷 정보를 조회하거나 관리할 수 있는 특수 권한을 가집니다.

## 2. 이미 생성된 사용자의 키를 다시 확인하려면?

```bash
sudo radosgw-admin  user info --uid=myuser
```

## 3. 참고

- [1] [https://docs.ceph.com](https://docs.ceph.com/en/latest/radosgw/admin/)
- [2] [https://gist.github.com](https://gist.github.com/abasu0713/f0e15a352946649ce89d15f865739404)
