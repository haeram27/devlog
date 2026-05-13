# spring jar 실행시 properties 지정하기

우선순위(높은순)는 다음과 같습니다.
1. jar 명령행 인자: `--spring.profiles.active=prod`
2. JVM 시스템 프로퍼티: `-Dspring.profiles.active=prod`
3. 환경변수: `SPRING_PROFILES_ACTIVE=prod`
4. application.yml의 `spring.profiles.active: dev`

즉 같은 키를 여러 곳에 넣으면 명령행 인자가 최종값으로 적용됩니다.
application.yml이 application.properties 보다 우선 순위가 높지만 두 파일을 혼용하지 마십시오.

## java -jar 인자 방식
```bash
java -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar \
  --ceph.aws.s3.endpoint=http://127.0.0.1:8555 \
  --ceph.aws.s3.region=us-east-1 \
  --ceph.aws.s3.access-key=YOUR_ACCESS_KEY \
  --ceph.aws.s3.secret-key=YOUR_SECRET_KEY \
  --ceph.aws.s3.path-style-access-enabled=true
```

## JVM 시스템 프로퍼티 방식
```bash
java \
  -Dceph.aws.s3.endpoint=http://127.0.0.1:8555 \
  -Dceph.aws.s3.region=us-east-1 \
  -Dceph.aws.s3.access-key=YOUR_ACCESS_KEY \
  -Dceph.aws.s3.secret-key=YOUR_SECRET_KEY \
  -Dceph.aws.s3.path-style-access-enabled=true \
  -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar
```

## 환경변수 방식
```bash
export CEPH_AWS_S3_ENDPOINT=http://127.0.0.1:8555
export CEPH_AWS_S3_REGION=us-east-1
export CEPH_AWS_S3_ACCESS_KEY=YOUR_ACCESS_KEY
export CEPH_AWS_S3_SECRET_KEY=YOUR_SECRET_KEY
export CEPH_AWS_S3_PATH_STYLE_ACCESS_ENABLED=true

java -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar
```

## application.yml 방식
```yml
ceph:
  aws:
    s3:
      endpoint: http://127.0.0.1:8555
      region: us-east-1
      access-key: YOUR_ACCESS_KEY
      secret-key: YOUR_SECRET_KEY
      path-style-access-enabled: true
```