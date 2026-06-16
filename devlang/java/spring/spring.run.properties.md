# spring property 사용하기

## 프로퍼티 플레이스홀더(Property Placeholder)

스프링에서 `${my.user.id}` 같은 형식의 참조는 보통 **프로퍼티 플레이스홀더(Property Placeholder)** 또는 **플레이스홀더 표현식**이라고 부릅니다.

- `${...}`: 외부 설정값(환경변수, application.yml, 시스템 프로퍼티 등)에서 키를 찾아 치환
- `#{...}`: SpEL(Spring Expression Language) 표현식, 런타임에 동적으로 Bean 객체 참조 또는 계산식 사용시

예를 들어 `${my.user.id:defaultValue}`처럼 `:`을 사용해 기본값도 줄 수 있습니다.

### 플레이스 홀더(`${...}`) 용도

스프링의 프로퍼티 플레이스홀더(`${...}`)는 “프로퍼티 값을 읽는 지점”에서 거의 대부분 사용가능합니다.

“외부 설정값을 읽는 위치”면 대부분 가능하고, 가장 흔한 사용처는 application.yml 값과 @Value입니다.

대표적으로는 아래입니다.

1. 설정 파일 내부 값 참조  
- application.yml / application.properties 안에서 다른 프로퍼티를 참조할 때  
- 예: app.name: ${spring.application.name:my-app}

2. 빈 필드/생성자 주입  
- `@Value("${my.user.id}")` 형태로 필드, 생성자 파라미터, 메서드 파라미터에 주입

3. 어노테이션 속성값  
- 문자열 속성을 받는 많은 스프링 어노테이션에서 사용 가능  
- 예: @Scheduled(cron = "${job.cron}")  
- 예: @KafkaListener(topics = "${kafka.topic.name}")

3. `@ConfigurationProperties` 바인딩 대상 값  
- 설정 파일 쪽 값에 플레이스홀더가 있으면, 해석된 뒤 바인딩됨  
- 즉 클래스 내부에 직접 ${...}를 쓰기보다 설정값에서 사용

4. XML 기반 설정(레거시)  
- XML 빈 정의의 property 값에도 `${...}` 사용 가능

추가로 자주 쓰는 문법:
- 기본값 지정: `${my.user.id:guest}`
- 중첩 참조: `${outer:${inner:default}}`

### SpEL 표현식 (`#{...}`) 용도

SpEL은 스프링 안에서 값을 “동적으로 계산”할 때 씁니다.  
즉 `${...}`가 단순 설정값 치환이라면, `#{...}`는 조건/연산/메서드 호출까지 가능한 표현식입니다.

대표 용도는 아래입니다.

1. 조건 계산
- 예: `@Value("#{systemProperties['user.timezone'] ?: 'UTC'}")`
- 시스템 값이 없으면 기본값 사용 같은 로직

2. 빈/프로퍼티 참조
- 다른 빈의 값 참조, 빈 메서드 호출
- 예: `@Value("#{myBean.timeout * 2}")`

3. 컬렉션 처리
- 리스트/맵 접근, 필터링, 선택
- 예: `#{myMap['key']}`, `#{myList[0]}`

4. 어노테이션 속성의 동적 값
- `@Scheduled`, `@Cacheable` 등에서 조건식이나 key 계산에 활용

5. 보안 표현식(Spring Security)
- `@PreAuthorize("hasRole('ADMIN') and #id == principal.id")`
- 접근 제어 규칙을 선언적으로 작성

한 줄 정리:
- `${...}`: 외부 설정값 치환
- `#{...}`: 스프링 컨텍스트 기반의 동적 계산/로직 표현

실무에서는 설정 키 참조는 `${...}`를 기본으로, 계산이나 조건이 필요할 때만 `#{...}`를 쓰는 것이 가장 깔끔합니다.

SpEL 표현식에서 참조 할 수 있는 컬렉션은 “SpEL 평가 시점에 컨텍스트에 올라와 있는 객체”의 컬렉션을 뜻합니다.

즉, 자바 코드 안의 아무 로컬 변수를 자동으로 보는 게 아니라, SpEL이 접근 가능하도록 노출된 대상이어야 합니다.

대표 예시는 아래입니다.

- 스프링 빈의 필드/프로퍼티에 있는 List, Map, 배열
- 메서드 인자(예: 보안 표현식에서 #id, #req 같은 파라미터)
- SpEL 내부에서 만든 리터럴 컬렉션(리스트/맵)

정리하면:

가능: SpEL 컨텍스트에 있는 컬렉션
불가: 컨텍스트에 없는 임의의 로컬 변수 컬렉션

## property 설정

우선순위(높은순-낮은번호가 최종적용)는 다음과 같습니다.
1. java 명령행 인자
2. JVM 시스템 프로퍼티
3. OS 환경변수
4. jar 외부 config 파일 (application.yml/application.properties)
5. jar 내부 config 파일 (application.yml/application.properties)

즉 같은 키를 여러 곳에 넣으면 `명령행 인자`가 최종값으로 적용됩니다.
application.yml이 application.properties 보다 우선 순위가 높지만 두 파일을 혼용하지 마십시오.

### java 명령행 인자(argument) 방식
```bash
java -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar \
  --ceph.aws.s3.endpoint=http://127.0.0.1:8555 \
  --ceph.aws.s3.region=us-east-1 \
  --ceph.aws.s3.access-key=YOUR_ACCESS_KEY \
  --ceph.aws.s3.secret-key=YOUR_SECRET_KEY \
  --ceph.aws.s3.path-style-access-enabled=true
```

### JVM 시스템 프로퍼티(JVM 환경변수) 방식
```bash
java \
  -Dceph.aws.s3.endpoint=http://127.0.0.1:8555 \
  -Dceph.aws.s3.region=us-east-1 \
  -Dceph.aws.s3.access-key=YOUR_ACCESS_KEY \
  -Dceph.aws.s3.secret-key=YOUR_SECRET_KEY \
  -Dceph.aws.s3.path-style-access-enabled=true \
  -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar
```

### OS 환경변수 방식
```bash
export CEPH_AWS_S3_ENDPOINT=http://127.0.0.1:8555
export CEPH_AWS_S3_REGION=us-east-1
export CEPH_AWS_S3_ACCESS_KEY=YOUR_ACCESS_KEY
export CEPH_AWS_S3_SECRET_KEY=YOUR_SECRET_KEY
export CEPH_AWS_S3_PATH_STYLE_ACCESS_ENABLED=true

java -jar build/libs/spring-ceph-client-0.0.1-SNAPSHOT.jar
```

OS 환경 변수의 경우 spring 내부 에서 참조(소스 또는 applicaton.yml 파일) 할 때 스프링(특히 Spring Boot)의 환경변수 바인딩에서 완화 규칙이 적용되어 매칭됩니다.

OS 환경 변수의 경우 대문자와 `_` 구분자로 이름을 지정하는데 spring 내부에서 참조할 때는 spring이 자동으로 소문자와 `.`으로 바인딩 해줍니다. 실제 OS 환경변수 이름이 바뀌는 것은 아니고 조회/바인딩 시점에만 이렇게 해석된다는 점입니다.

- `CEPH_AWS_S3_ENDPOINT`를 `${ceph.aws.s3.endpoint}`으로 참조
- `SPRING_PROFILES_ACTIVE`를 `${spring.profiles.active}`으로 참조


### application.yml 방식
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

## 외부 config 파일 가능 위치

Spring Boot 기본 동작 기준(2.4+), 외부 application.yml 탐색 위치는 아래와 같습니다.

### config 파일(application.yml) 기본 탐색 위치
1. 실행 현재 디렉터리
- `./`

2. 실행 현재 디렉터리의 config 디렉터리
- `./config/`

3. 실행 현재 디렉터리의 config 바로 아래 하위 디렉터리들
- `./config/*/`

예시로 실제 파일명은 보통 아래 형태를 둡니다.
1. ./application.yml
2. ./config/application.yml
3. ./config/dev/application.yml
4. ./config/prod/application.yml

프로필 파일도 같은 위치 규칙을 따릅니다.
1. application-prod.yml
2. application-dev.yml


`실행 현재 디렉토리`란 java 명령을 실행한 현재 작업 디렉터리(CWD)입니다.

즉 기준은 jar 파일 위치가 아니라, 터미널에서 명령을 친 위치입니다.

예시:
1. /opt/app 에서 `java -jar /srv/bin/myapp.jar` 실행
- 실행 디렉터리: /opt/app
- 기본 외부 config 탐색: /opt/app, /opt/app/config, /opt/app/config/*

2. /srv/bin 에서 같은 명령 실행
- 실행 디렉터리: /srv/bin
- 기본 외부 config 탐색: /srv/bin, /srv/bin/config, /srv/bin/config/*

jar와 같은 폴더를 기준으로 고정하고 싶으면 `spring.config.location` 또는 `spring.config.additional-location`으로 경로를 명시하는 방식이 가장 안전합니다.

### config 파일 추가 경로 설정

추가로, 기본 위치 외에 임의 경로를 쓰려면 다음 옵션으로 확장 가능합니다. 다음 설정을 사용하면 지정된 위치에서 
1. spring.config.location: 기본 위치를 대체 (기본 위치 탐색 무시)
2. spring.config.additional-location: 기본 위치에 추가 (기본 위치 탐색 + 추가 위치도 탐색)

참고로 classpath 내부(jar 내부)의 application.yml은 외부가 아니라 내부 설정 위치입니다.

여러 위치를 location 설정에 추가 하려면 콤마(,)로 구분해서 한 줄에 나열하면 됩니다.

1. 명령행 인자 예시
``` 
--spring.config.location=optional:file:./config/,optional:file:/etc/myapp/,optional:classpath:/custom/
```
2. 환경변수 예시
```
SPRING_CONFIG_LOCATION=optional:file:./config/,optional:file:/etc/myapp/,optional:classpath:/custom/
```

3.application.yml 예시  
```yaml
spring:
  config:
    location: optional:file:./config/,optional:file:/etc/myapp/,optional:classpath:/custom/
```

주의할 점

1. spring.config.location은 기본 탐색 위치를 대체합니다.  
2. 디렉터리를 지정할 때는 끝에 / 를 붙이는 것이 안전합니다.  
3. 없을 수도 있는 경로는 optional: 접두어를 붙이면 시작 실패를 방지할 수 있습니다.  
4. 기본 위치를 유지하면서 추가만 하려면 spring.config.additional-location을 쓰면 됩니다.

#### `optional:file:` 접두어
`optional:file:` 표현은 Spring Boot 설정 위치 문자열에서 쓰는 접두어 조합입니다.

의미를 나누면:

1. file:
- 파일 시스템 경로를 뜻합니다.
- 예: /etc/myapp/, ./config/ 같은 OS 경로를 읽겠다는 의미입니다.

2. optional:
- 해당 경로가 없어도 애플리케이션 시작을 실패시키지 말라는 의미입니다.
- optional 이 없으면, 지정한 위치가 없을 때 시작 오류가 날 수 있습니다.

즉 optional:file:/etc/myapp/ 는
“로컬 파일 시스템의 /etc/myapp/ 를 설정 위치로 보되, 없어도 그냥 넘어가라”는 뜻입니다.

비교:
- file:/etc/myapp/  → 경로 없으면 실패 가능
- optional:file:/etc/myapp/ → 경로 없어도 계속 실행

추가로 같은 규칙이 classpath 에도 적용되어 optional:classpath:/custom/ 처럼 쓸 수 있습니다.
