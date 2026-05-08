# ceph resumable upload with spring boot

Spring Boot 환경에서는 AWS SDK v2를 빈(Bean)으로 등록하여 관리하고, 말씀하신 FileUpload 중단 시 상태를 직렬화하여 저장하는 로직을 서비스 레이어에 구현하면 됩니다.

구체적인 구현 방법과 예시 코드를 정리해 드립니다.

## 1. 의존성 설정 (build.gradle)
Spring Boot 버전과 호환되는 SDK를 추가합니다.

```groovy
dependencies {
    implementation 'software.amazon.awssdk:s3-transfer-manager:2.20.0'
    implementation 'software.amazon.awssdk.crt:aws-crt:0.21.1'
    implementation 'software.amazon.awssdk:s3:2.20.0'
}
```

## 2. Configuration 설정
Ceph RGW 연결을 위한 S3TransferManager를 빈으로 등록합니다.

```java
@Configurationpublic class CephConfig {
    @Value("${ceph.s3.endpoint}")
    private String endpoint;
    @Value("${ceph.s3.access-key}")
    private String accessKey;
    @Value("${ceph.s3.secret-key}")
    private String secretKey;

    @Bean
    public S3TransferManager s3TransferManager() {
        S3AsyncClient s3AsyncClient = S3AsyncClient.builder()
            .endpointOverride(URI.create(endpoint))
            .region(Region.US_EAST_1) // Ceph 설정에 맞춤
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create(accessKey, secretKey)))
            .forcePathStyle(true) // Ceph 사용 시 필수 설정인 경우가 많음
            .build();

        return S3TransferManager.builder()
            .s3Client(s3AsyncClient)
            .build();
    }
}
```

## 3. 업로드 및 재개 로직 (Service)
FileUpload를 중단(pause)하고 상태를 저장한 뒤 재개하는 핵심 로직입니다.

```java
@Service
@RequiredArgsConstructorpublic class CephService {
    private final S3TransferManager transferManager;
    private ResumableFileUpload resumableUpload; // 실제로는 DB나 파일에 저장 필요

    public void uploadWithPause(Path sourceFile, String bucket, String key) {
        FileUpload fileUpload = transferManager.uploadFile(u -> u
            .putObjectRequest(p -> p.bucket(bucket).key(key))
            .source(sourceFile)
        );

        // 예시: 중간에 네트워크 오류나 특정 조건에 의해 중단 시
        // resumableUpload = fileUpload.pause(); 
    }

    public void resumeUpload() {
        if (resumableUpload != null) {
            FileUpload resumedUpload = transferManager.resumeUploadFile(resumableUpload);
            resumedUpload.completionFuture().join(); // 완료 대기
        }
    }
}
```

## 4. 추가 팁

* 파일 크기 관리: `ResumableFileUpload`는 메모리에만 존재하면 안 되므로, 서버가 재시작될 경우를 대비해 해당 객체의 상태 정보(파일 경로, 업로드 ID 등)를 JSON 형태로 파일 시스템이나 Redis에 직렬화하여 보관하는 로직을 추가하는 것이 좋습니다.
* `S3TransferManager` 성능: `aws-crt-client`를 사용하면 Java 기본 클라이언트보다 멀티파트 업로드 성능이 훨씬 뛰어납니다.
* 중단된 파일 정리: Ceph 대시보드나 CLI(radosgw-admin multipart list)를 통해 완료되지 않은 업로드가 쌓여 있는지 주기적으로 확인하세요.

중단된 업로드 상태를 어디에 저장(DB vs Local File)하여 관리하실 계획인가요? 상황에 맞는 보관 방식을 추천해 드릴 수 있습니다.



---

네, 두 라이브러리를 모두 사용하는 방식이 실무에서 가장 권장되는 표준 조합입니다.
그 이유는 두 라이브러리가 담당하는 역할(Layer)이 완전히 다르기 때문입니다.
## 1. 왜 두 개를 같이 써야 하나요?

| 구분 | AWS SDK for Java (v2) | radosgw-admin4j |
|---|---|---|
| 역할 | 데이터 작업 (Data Plane) | 관리 작업 (Control Plane) |
| 주요 기능 | 파일 업로드/다운로드, 버킷 내 객체 리스트 조회, 삭제 | 사용자 생성/삭제, Access Key 발급, 할당량(Quota) 설정 |
| 통신 프로토콜 | 표준 S3 API | Ceph Admin Ops API (전용 API) |
| 비유 | 은행 창구에서 입출금 업무 | 은행 관리자가 계좌 개설/한도 변경 업무 |

## 2. 프로젝트 구성 예시 (Spring Boot)
Spring Boot에서 이 두 라이브러리를 함께 설정하면 다음과 같은 구조가 됩니다.
[pom.xml 또는 build.gradle]

* software.amazon.awssdk:s3: 일반적인 파일 처리를 위해 추가
* com.twonote:radosgw-admin4j: 관리자 기능을 위해 추가

[Service 계층 예시]

@Servicepublic class CephService {

    private final S3Client s3Client;       // AWS SDK (데이터용)
    private final RadosgwAdmin rgwAdmin;   // radosgw-admin4j (관리용)

    public void createUserAndBucket(String userId) {
        // 1. 관리자 API로 사용자 생성 (radosgw-admin4j 사용)
        User user = rgwAdmin.createUser(userId);
        
        // 2. 생성된 사용자의 키로 버킷 생성 (AWS SDK 사용)
        // (또는 관리자 권한으로 직접 생성 가능)
        s3Client.createBucket(b -> b.bucket(userId + "-storage"));
    }

    public void uploadFile(String bucket, String key, byte[] content) {
        // 3. 실제 데이터 업로드 (AWS SDK 사용)
        s3Client.putObject(..., RequestBody.fromBytes(content));
    }
}

## 3. 하나만 쓸 수는 없나요?

* AWS SDK만 사용할 경우: 사용자 계정을 동적으로 생성하거나 Ceph 서버의 상세 설정을 변경하는 기능을 구현할 수 없습니다. (단순 저장소로만 쓴다면 이것만으로 충분합니다.)
* radosgw-admin4j만 사용할 경우: 파일 업로드/다운로드 기능이 라이브러리에 포함되어 있지 않아 저장소 기능을 아예 쓸 수 없습니다.

결론적으로, 사용자 관리 기능을 포함한 스토리지 서비스를 직접 구축하신다면 두 라이브러리를 병행하는 것이 가장 깔끔하고 안정적인 선택입니다.
혹시 Spring Boot 버전(2.x vs 3.x)을 알려주시면, 두 라이브러리를 한꺼번에 등록하는 Configuration 설정 코드를 정리해 드릴까요?

