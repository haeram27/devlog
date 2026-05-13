# Ceph RGW 사용자 Capability 설정 가이드 (Ceph 19.2.3)

## 1. 기준 버전

- Ceph: 19.2.3 (Squid)

## 2. caps 기본 문법

```bash
radosgw-admin caps add --uid=<uid> --caps="<cap>=<perm>[;<cap>=<perm>...]"
```

예시:

```bash
radosgw-admin caps add --uid=rgw-admin --caps="users=read;buckets=read"
```

## 3. Ceph 에서 사용하는 cap 키

- users
- buckets
- metadata
- usage
- zone

설명:
- users: 사용자 관련 관리 API 권한
- buckets: 버킷 관련 관리 API 권한
- metadata: 메타데이터 조회/관리 API 권한
- usage: 사용량 통계 조회/관리 API 권한
- zone: zone/zonegroup 관련 조회/관리 API 권한

## 4. perm 값 (각 cap에 지정 가능)

- `read`
- `write`
- `*`
- `read,write`

참고:

- `*` 는 일반적으로 read + write 전체 권한 의미
- `read,write` 표기도 가능하지만, 운영 문서에서는 보통 `*` 표기를 더 많이 사용

## 5. 권장 권한 템플릿

1. 조회 전용 계정

```plain
--caps="users=read;buckets=read;metadata=read;usage=read;zone=read"
```

1. 사용자/버킷 운영 계정

```plain
--caps="users=write;buckets=write;metadata=read"
```

1. 관리자 계정(광범위 권한)

```plain
--caps="buckets=*;users=*;usage=read;metadata=read;zone=read"
```

## 6. 생성/수정/확인 예시

```bash
# 사용자 생성:
radosgw-admin user create --uid=myuser --display-name="My User"

# caps 부여:
radosgw-admin caps add --uid=myuser --caps="users=read;buckets=read;usage=read;metadata=read;zone=read"

# caps 추가/변경:
radosgw-admin caps add --uid=myuser --caps="buckets=write"

# caps 제거:
radosgw-admin caps rm --uid=myuser --caps="buckets=write"

# 최종 확인:
radosgw-admin user info --uid=myuser
```

## 7. 주의사항

- caps는 RGW Admin Ops API 권한입니다.
- 일반 S3 데이터 접근 권한(버킷 정책, ACL, IAM)과는 별개입니다.
- 최소 권한으로 시작하고 필요한 항목만 점진적으로 추가하는 방식이 안전합니다.
- 운영 환경에서는 사용자 유형(조회 전용, 운영자, 관리자)별로 계정을 분리하는 것이 좋습니다.