# jcmd를 이용한 memory-leak 분석

- **Java 실행 시** JVM 분석용 JVM 옵션을 추가하여 jcmd 런타임 진단 도구로 정보 추출

## 메모리 릭 분석용 JVM 옵션

```bash
java \
  # Native Memory Tracking (필수)
  -XX:NativeMemoryTracking=detail \
  
  # Heap Dump 자동 생성 (OOM 발생 시)
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/dumps/heap_dump.hprof \
  
  # GC 로그 (메모리 패턴 분석용)
  -Xlog:gc*:file=/logs/gc.log:time,tags:filecount=5,filesize=100M \
  
  # 기본 메모리 설정
  -Xmx4G \
  -Xms4G \
  
  -jar your-app.jar
```

## Docker 환경 설정 예시

### 1. **Dockerfile**
```dockerfile
FROM openjdk:17-slim

# 덤프 디렉토리 생성
RUN mkdir -p /dumps /logs

# 환경변수로 JVM 옵션 설정
ENV JAVA_OPTS="-XX:NativeMemoryTracking=detail \
               -XX:+HeapDumpOnOutOfMemoryError \
               -XX:HeapDumpPath=/dumps/heap_dump.hprof \
               -Xlog:gc*:file=/logs/gc.log:time,tags \
               -Xmx4G -Xms4G"

COPY target/app.jar /app.jar

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app.jar"]
```

### 2. **docker-compose.yml**
```yaml
services:
  app:
    image: your-app:latest
    volumes:
      - ./dumps:/dumps      # 힙 덤프 저장
      - ./logs:/logs        # 로그 저장
    environment:
      JAVA_OPTS: >
        -XX:NativeMemoryTracking=detail
        -XX:+HeapDumpOnOutOfMemoryError
        -XX:HeapDumpPath=/dumps/heap_dump.hprof
        -Xlog:gc*:file=/logs/gc.log:time,tags
        -Xmx4G
```

## jcmd 명령어 사용법

### 1. **PID 확인**
```bash
# 컨테이너 내부에서
docker exec <container-id> jcmd

# 또는
docker exec <container-id> ps aux | grep java
```

### 2. **Native Memory 분석**
```bash
# 기본 요약
docker exec <container-id> jcmd <pid> VM.native_memory summary

# 상세 정보
docker exec <container-id> jcmd <pid> VM.native_memory detail

# 베이스라인 설정 (비교 분석용)
docker exec <container-id> jcmd <pid> VM.native_memory baseline

# 베이스라인 대비 변화 확인
docker exec <container-id> jcmd <pid> VM.native_memory summary.diff
```

### 3. **Heap Dump 수동 생성**
```bash
# 힙 덤프 생성
docker exec <container-id> jcmd <pid> GC.heap_dump /dumps/manual_dump.hprof

# 컨테이너에서 로컬로 복사
docker cp <container-id>:/dumps/manual_dump.hprof ./
```

### 4. **기타 유용한 jcmd 명령어**
```bash
# GC 실행
docker exec <container-id> jcmd <pid> GC.run

# 스레드 덤프
docker exec <container-id> jcmd <pid> Thread.print > thread_dump.txt

# JVM 플래그 확인
docker exec <container-id> jcmd <pid> VM.flags

# 시스템 속성 확인
docker exec <container-id> jcmd <pid> VM.system_properties
```

## 메모리 릭 분석 워크플로우

```bash
# 1. 베이스라인 설정
docker exec app jcmd 1 VM.native_memory baseline

# 2. 애플리케이션 부하 발생

# 3. 일정 시간 후 변화 확인
docker exec app jcmd 1 VM.native_memory summary.diff

# 4. 힙 덤프 생성
docker exec app jcmd 1 GC.heap_dump /dumps/leak_analysis.hprof

# 5. 로컬로 복사 후 분석 (Eclipse MAT, VisualVM 등)
docker cp app:/dumps/leak_analysis.hprof ./
```

## 주의사항

1. **NativeMemoryTracking**은 성능 오버헤드가 있습니다 (약 5-10%)
   - 프로덕션: `summary` 사용
   - 디버깅: `detail` 사용

2. **힙 덤프 파일 크기**는 `-Xmx`와 비슷하므로 충분한 디스크 공간 필요

3. **볼륨 마운트** 필수: 컨테이너 재시작 시 덤프 파일 유실 방지

이 설정으로 메모리 릭을 효과적으로 분석할 수 있습니다!