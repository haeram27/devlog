# s3cmd (MicroCeph 공식 가이드 추천)

- microceph client로 s3cmd 사용하는 방법

## 1. s3cmd 설치 및 설정

MicroCeph 공식 문서에서 주로 추천하는 도구입니다. [s3cmd 공식 가이드](https://techdocs.akamai.com/cloud-computing/docs/using-s3cmd-with-object-storage).

- 설치:

```bash
sudo apt update
sudo apt install s3cmd -y
```

## 2. s3cmd 초기 설정

s3cmd는 설정 파일(.s3cfg)에 엔드포인트 주소를 고정할 수 있어 MicroCeph(RGW) 환경에서 더 직관적입니다.

### 1단계: 대화형 설정 시작

```bash
s3cmd --configure
```

다음 항목들을 순서대로 입력합니다

   1. Access Key / Secret Key: 위에서 확인한 키를 입력합니다.
   2. S3 Endpoint: RGW 서버의 IP와 포트를 입력합니다. (예: 192.168.1.10:80)
   3. DNS-style bucket+hostname template: %(bucket)s.서버IP:포트 대신 단순히 서버IP:포트만 입력하거나 기본값을 유지합니다. (MicroCeph 환경에선 주로 host_base와 동일하게 설정합니다.)
   4. Use HTTPS protocol: No (SSL 설정을 따로 하지 않은 경우)
   5. Test access: y를 눌러 접속이 성공하는지 확인합니다. [9, 14] 

### 2단계: 수동 수정 (필요 시)

설정 완료 후 ~/.s3cfg 파일을 열어 다음 항목이 올바른지 확인하면 더 확실합니다.

```text
host_base = 서버IP:포트
host_bucket = 서버IP:포트/%(bucket)
use_https = False
```

## 1. s3cmd를 사용하는 방법 (MicroCeph 공식 가이드 추천)

- 먼저 sudo microceph enable rgw로 서비스를 켜고 사용자를 생성한 뒤, 설정을 완료해야 합니다.
- S3 전용 도구로 명령어가 조금 더 직관적입니다.

- 버킷 생성

```bash
s3cmd mb s3://버킷이름
```

- 파일 업로드

```bash
s3cmd put 로컬파일명 s3://버킷이름/
```

- 파일 삭제

```bash
s3cmd del s3://버킷이름/파일명
```

- 버킷 삭제

```bash
s3cmd rb s3://버킷이름
```

## 3. MicroCeph 사용자 키 확인법

설정에 필요한 키를 모른다면 다음 명령어로 확인하세요. [14] 

```bash
sudo microceph.radosgw-admin user info --uid=[사용자ID]
```

## 3. 관리자용 강제 삭제 (radosgw-admin) [8] 

파일 단위의 일반적인 조작보다는 버킷 전체를 비우거나 강제로 삭제할 때 사용합니다. [9, 10] 

- 버킷 내 모든 객체 포함 삭제:

```bash
sudo microceph.radosgw-admin bucket rm --bucket=[버킷이름] --purge-objects
```

## 참고

- [1] [https://canonical-microceph.readthedocs-hosted.com](https://canonical-microceph.readthedocs-hosted.com/v19.2.0-squid/reference/commands/)
- [2] [https://documentation.ubuntu.com](https://documentation.ubuntu.com/microcloud/default/microceph/tutorial/get-started/)
- [3] [https://documentation.ubuntu.com](https://documentation.ubuntu.com/microcloud/default/microceph/tutorial/get-started/)
- [4] [https://medium.com](https://medium.com/@liufirst/how-to-set-up-a-single-node-ceph-cluster-with-microceph-cli-guide-d7ebc06c1b2e)
- [5] [https://canonical-microceph.readthedocs-hosted.com](https://canonical-microceph.readthedocs-hosted.com/stable/tutorial/get-started/)
- [6] [https://medium.com](https://medium.com/@hojat_gazestani/basic-ceph-rgw-setup-and-s3-access-with-aws-cli-and-python-boto3-c3db142d3b55)
- [7] [https://oneuptime.com](https://oneuptime.com/blog/post/2026-03-31-rook-use-aws-cli-ceph-rgw-s3/view)
- [8] [https://docs.ceph.com](https://docs.ceph.com/en/reef/radosgw/admin/)
- [9] [https://daegonk.medium.com](https://daegonk.medium.com/ceph-lets-delete-everything-part-1-e523b59b1d6f)
- [10] [https://syn-flood.com](https://syn-flood.com/posts/2025/05/10/zero-byte-files.html)
