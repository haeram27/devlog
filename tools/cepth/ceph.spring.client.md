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

