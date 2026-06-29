# AWS SDK for Java v2 - S3Client 메서드 간략 용도

- 기준 문서: https://docs.aws.amazon.com/java/api/latest/software/amazon/awssdk/services/s3/S3Client.html
- 정리 기준: 같은 이름의 오버로드(예: Request/Consumer)는 하나로 묶어 설명

| 메서드 | 간략 용도 |
|---|---|
| abortMultipartUpload | 진행 중인 멀티파트 업로드 중단 |
| builder | S3Client 빌더 생성 |
| completeMultipartUpload | 멀티파트 업로드 완료(조립) |
| copyObject | 객체를 다른 버킷/키로 서버사이드 복사 |
| create | 기본 Region/Credential 체인으로 S3Client 즉시 생성 |
| createBucket | 버킷 또는 버킷 메타데이터 구성 생성 |
| createBucketMetadataConfiguration | 버킷 또는 버킷 메타데이터 구성 생성 |
| createBucketMetadataTableConfiguration | 버킷 또는 버킷 메타데이터 구성 생성 |
| createMultipartUpload | 멀티파트 업로드 세션 시작 |
| createSession | 세션 기반 접근(예: S3 Express)용 세션 생성 |
| deleteBucket | 버킷 설정/정책 삭제 |
| deleteBucketAnalyticsConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketCors | 버킷 설정/정책 삭제 |
| deleteBucketEncryption | 버킷 설정/정책 삭제 |
| deleteBucketIntelligentTieringConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketInventoryConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketLifecycle | 버킷 설정/정책 삭제 |
| deleteBucketMetadataConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketMetadataTableConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketMetricsConfiguration | 버킷 설정/정책 삭제 |
| deleteBucketOwnershipControls | 버킷 설정/정책 삭제 |
| deleteBucketPolicy | 버킷 설정/정책 삭제 |
| deleteBucketReplication | 버킷 설정/정책 삭제 |
| deleteBucketTagging | 버킷 설정/정책 삭제 |
| deleteBucketWebsite | 버킷 설정/정책 삭제 |
| deleteObject | 객체 삭제 |
| deleteObjectTagging | 객체 관련 부가 정보(예: 태그) 삭제 |
| deleteObjects | 객체 여러 개 일괄 삭제 |
| deletePublicAccessBlock | Public Access Block 설정 삭제 |
| getBucketAbac | 버킷 설정/정책/상태 조회 |
| getBucketAccelerateConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketAcl | 버킷 설정/정책/상태 조회 |
| getBucketAnalyticsConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketCors | 버킷 설정/정책/상태 조회 |
| getBucketEncryption | 버킷 설정/정책/상태 조회 |
| getBucketIntelligentTieringConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketInventoryConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketLifecycleConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketLocation | 버킷 설정/정책/상태 조회 |
| getBucketLogging | 버킷 설정/정책/상태 조회 |
| getBucketMetadataConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketMetadataTableConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketMetricsConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketNotificationConfiguration | 버킷 설정/정책/상태 조회 |
| getBucketOwnershipControls | 버킷 설정/정책/상태 조회 |
| getBucketPolicy | 버킷 설정/정책/상태 조회 |
| getBucketPolicyStatus | 버킷 설정/정책/상태 조회 |
| getBucketReplication | 버킷 설정/정책/상태 조회 |
| getBucketRequestPayment | 버킷 설정/정책/상태 조회 |
| getBucketTagging | 버킷 설정/정책/상태 조회 |
| getBucketVersioning | 버킷 설정/정책/상태 조회 |
| getBucketWebsite | 버킷 설정/정책/상태 조회 |
| getObject | 객체를 스트리밍/변환 방식으로 다운로드 |
| getObjectAcl | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectAsBytes | 객체를 바이트 배열(ResponseBytes)로 다운로드 |
| getObjectAttributes | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectLegalHold | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectLockConfiguration | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectRetention | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectTagging | 객체 메타/잠금/태그 등 정보 조회 |
| getObjectTorrent | 객체의 토렌트 메타데이터 조회/다운로드 |
| getObjectTorrentAsBytes | 객체 토렌트 메타데이터를 바이트로 다운로드 |
| getPublicAccessBlock | Public Access Block 설정 조회 |
| headBucket | 버킷 존재/접근 가능 여부 및 메타 확인 |
| headObject | 객체 본문 없이 메타데이터만 확인 |
| listBucketAnalyticsConfigurations | 버킷 하위 구성 리소스 목록 조회 |
| listBucketIntelligentTieringConfigurations | 버킷 하위 구성 리소스 목록 조회 |
| listBucketInventoryConfigurations | 버킷 하위 구성 리소스 목록 조회 |
| listBucketMetricsConfigurations | 버킷 하위 구성 리소스 목록 조회 |
| listBuckets | 계정의 버킷 목록 조회 |
| listBucketsPaginator | 버킷 목록을 페이지 단위 반복 조회 |
| listDirectoryBuckets | 디렉터리 버킷(S3 Express) 목록 조회 |
| listDirectoryBucketsPaginator | 디렉터리 버킷 목록을 페이지 단위 반복 조회 |
| listMultipartUploads | 진행 중인 멀티파트 업로드 목록 조회 |
| listMultipartUploadsPaginator | 진행 중인 멀티파트 업로드 목록을 페이지 단위 반복 조회 |
| listObjectVersions | 객체 버전 목록 조회 |
| listObjectVersionsPaginator | 객체 버전 목록을 페이지 단위 반복 조회 |
| listObjects | 객체 목록 조회(ListObjects v1) |
| listObjectsV2 | 객체 목록 조회(ListObjects v2) |
| listObjectsV2Paginator | 객체 목록(v2)을 페이지 단위 반복 조회 |
| listParts | 특정 멀티파트 업로드의 파트 목록 조회 |
| listPartsPaginator | 파트 목록을 페이지 단위 반복 조회 |
| putBucketAbac | 버킷 설정/정책 적용 또는 갱신 |
| putBucketAccelerateConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketAcl | 버킷 설정/정책 적용 또는 갱신 |
| putBucketAnalyticsConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketCors | 버킷 설정/정책 적용 또는 갱신 |
| putBucketEncryption | 버킷 설정/정책 적용 또는 갱신 |
| putBucketIntelligentTieringConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketInventoryConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketLifecycleConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketLogging | 버킷 설정/정책 적용 또는 갱신 |
| putBucketMetricsConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketNotificationConfiguration | 버킷 설정/정책 적용 또는 갱신 |
| putBucketOwnershipControls | 버킷 설정/정책 적용 또는 갱신 |
| putBucketPolicy | 버킷 설정/정책 적용 또는 갱신 |
| putBucketReplication | 버킷 설정/정책 적용 또는 갱신 |
| putBucketRequestPayment | 버킷 설정/정책 적용 또는 갱신 |
| putBucketTagging | 버킷 설정/정책 적용 또는 갱신 |
| putBucketVersioning | 버킷 설정/정책 적용 또는 갱신 |
| putBucketWebsite | 버킷 설정/정책 적용 또는 갱신 |
| putObject | 객체 업로드(단일 PUT) |
| putObjectAcl | 객체 ACL/잠금/보존/태그 등 설정 적용 |
| putObjectLegalHold | 객체 ACL/잠금/보존/태그 등 설정 적용 |
| putObjectLockConfiguration | 객체 ACL/잠금/보존/태그 등 설정 적용 |
| putObjectRetention | 객체 ACL/잠금/보존/태그 등 설정 적용 |
| putObjectTagging | 객체 ACL/잠금/보존/태그 등 설정 적용 |
| putPublicAccessBlock | Public Access Block 설정 적용 |
| renameObject | 객체 이름(키) 변경 요청(지원 스토리지/시나리오에서 사용) |
| restoreObject | 아카이브(Glacier 등) 객체 복원 요청 |
| serviceClientConfiguration | 클라이언트 서비스 설정 정보 조회 |
| serviceMetadata | 서비스 메타데이터(엔드포인트/리전 정보) 조회 |
| updateBucketMetadataInventoryTableConfiguration | 버킷 메타데이터 테이블 구성 갱신 |
| updateBucketMetadataJournalTableConfiguration | 버킷 메타데이터 테이블 구성 갱신 |
| updateObjectEncryption | 객체 암호화 상태/설정 갱신 |
| uploadPart | 멀티파트 업로드 파트 전송 |
| uploadPartCopy | 기존 객체를 소스로 멀티파트 파트 복사 |
| utilities | S3 유틸리티(S3Utilities) 접근 |
| waiter | 리소스 상태 대기(waiter) 유틸리티 접근 |
| writeGetObjectResponse | S3 Object Lambda용 GetObject 응답 쓰기 |

## 메서드 분류 정리

### 1) 클라이언트 생성/유틸

- create, builder
- serviceClientConfiguration, serviceMetadata
- utilities, waiter

### 2) 버킷 생성/삭제

- createBucket
- deleteBucket

### 3) 버킷 설정 조회(getBucket*)

- getBucketAcl, getBucketCors, getBucketEncryption, getBucketPolicy
- getBucketLifecycleConfiguration, getBucketVersioning, getBucketWebsite
- getBucketLocation, getBucketLogging, getBucketTagging, getBucketReplication
- getBucketOwnershipControls, getBucketRequestPayment, getBucketPolicyStatus
- getBucketAnalyticsConfiguration, getBucketMetricsConfiguration
- getBucketInventoryConfiguration, getBucketIntelligentTieringConfiguration
- getBucketMetadataConfiguration, getBucketMetadataTableConfiguration
- getBucketNotificationConfiguration, getBucketAbac
- getPublicAccessBlock

### 4) 버킷 설정 적용(putBucket*)

- putBucketAcl, putBucketCors, putBucketEncryption, putBucketPolicy
- putBucketLifecycleConfiguration, putBucketVersioning, putBucketWebsite
- putBucketLogging, putBucketTagging, putBucketReplication
- putBucketOwnershipControls, putBucketRequestPayment
- putBucketAnalyticsConfiguration, putBucketMetricsConfiguration
- putBucketInventoryConfiguration, putBucketIntelligentTieringConfiguration
- putBucketNotificationConfiguration, putBucketAbac
- putPublicAccessBlock

### 5) 버킷 설정 삭제(deleteBucket*)

- deleteBucketCors, deleteBucketEncryption, deleteBucketPolicy
- deleteBucketLifecycle, deleteBucketTagging, deleteBucketReplication
- deleteBucketWebsite, deleteBucketOwnershipControls
- deleteBucketAnalyticsConfiguration, deleteBucketMetricsConfiguration
- deleteBucketInventoryConfiguration, deleteBucketIntelligentTieringConfiguration
- deleteBucketMetadataConfiguration, deleteBucketMetadataTableConfiguration
- deletePublicAccessBlock

### 6) 객체 업로드/다운로드/메타

- putObject, getObject, getObjectAsBytes
- headObject, headBucket
- copyObject, renameObject, restoreObject
- updateObjectEncryption

### 7) 객체 보안/거버넌스/태그

- getObjectAcl, putObjectAcl
- getObjectTagging, putObjectTagging, deleteObjectTagging
- getObjectLegalHold, putObjectLegalHold
- getObjectRetention, putObjectRetention
- getObjectLockConfiguration, putObjectLockConfiguration

### 8) 객체/버전/멀티파트 목록 조회

- listObjects, listObjectsV2, listObjectsV2Paginator
- listObjectVersions, listObjectVersionsPaginator
- listMultipartUploads, listMultipartUploadsPaginator
- listParts, listPartsPaginator
- listBuckets, listBucketsPaginator
- listDirectoryBuckets, listDirectoryBucketsPaginator
- listBucketAnalyticsConfigurations
- listBucketMetricsConfigurations
- listBucketInventoryConfigurations
- listBucketIntelligentTieringConfigurations

### 9) 멀티파트 업로드

- createMultipartUpload
- uploadPart, uploadPartCopy
- completeMultipartUpload
- abortMultipartUpload

### 10) 특수 기능(S3 Express / Object Lambda)

- createSession
- writeGetObjectResponse
- getObjectTorrent, getObjectTorrentAsBytes

## 대표 요청 객체와 주의점

| 메서드(대표) | 대표 Request 타입 | 주의점 |
|---|---|---|
| createBucket | CreateBucketRequest | 리전 제약(LocationConstraint)과 네이밍 중복을 확인 |
| deleteBucket | DeleteBucketRequest | 버킷이 비어 있어야 삭제 가능 |
| listObjectsV2 | ListObjectsV2Request | 대량 조회 시 paginator 사용 권장 |
| getObject | GetObjectRequest | 큰 파일은 스트리밍 처리, 메모리 적재 지양 |
| putObject | PutObjectRequest | Content-Type/Length, SSE, 체크섬 설정 검토 |
| copyObject | CopyObjectRequest | 소스/대상 권한, 암호화 키(SSE-KMS) 호환 확인 |
| deleteObjects | DeleteObjectsRequest | 부분 실패 가능, 응답에서 실패 키 재처리 필요 |
| headObject | HeadObjectRequest | 본문 없이 메타 확인, 존재 여부 체크에 유용 |
| getObjectAttributes | GetObjectAttributesRequest | 필요한 속성만 요청해 비용/지연 최소화 |
| createMultipartUpload | CreateMultipartUploadRequest | 업로드 중단 시 abort 누락하면 스토리지 비용 발생 |
| uploadPart | UploadPartRequest | 파트 번호/ETag 정확히 관리(완료 조립에 필요) |
| completeMultipartUpload | CompleteMultipartUploadRequest | 파트 목록 순서/ETag 불일치 시 실패 가능 |
| abortMultipartUpload | AbortMultipartUploadRequest | 실패한 대용량 업로드 정리에 필수 |
| putBucketPolicy | PutBucketPolicyRequest | 잘못된 정책은 서비스 접근 장애를 유발할 수 있음 |
| getBucketPolicyStatus | GetBucketPolicyStatusRequest | 퍼블릭 노출 상태 점검용으로 활용 |
| putPublicAccessBlock | PutPublicAccessBlockRequest | 계정/버킷 정책과 함께 동작하므로 최종 접근 결과 검증 필요 |
| getPublicAccessBlock | GetPublicAccessBlockRequest | 차단 정책 적용 상태를 정기 점검 |
| putBucketEncryption | PutBucketEncryptionRequest | KMS 키 권한(IAM/KMS policy) 누락 여부 확인 |
| getBucketEncryption | GetBucketEncryptionRequest | 기본 암호화 강제 여부 확인 |
| putBucketVersioning | PutBucketVersioningRequest | 비용 증가(버전 누적)와 수명주기 정책 동시 설계 권장 |
| listObjectVersions | ListObjectVersionsRequest | 삭제 마커 포함 결과 해석 필요 |
| restoreObject | RestoreObjectRequest | Glacier 복원은 시간/비용이 발생 |
| putObjectTagging | PutObjectTaggingRequest | 태그 기반 정책(ABAC)과 과금 분류에 영향 |
| getObjectTagging | GetObjectTaggingRequest | 태그 기반 접근제어 디버깅에 유용 |
| createSession | CreateSessionRequest | S3 Express 등 세션 만료/재발급 처리 필요 |
| writeGetObjectResponse | WriteGetObjectResponseRequest | Object Lambda 전용 경로에서만 사용 |
| updateObjectEncryption | UpdateObjectEncryptionRequest | 지원 버킷/스토리지 유형에서만 동작 가능 |
| renameObject | RenameObjectRequest | 일반 S3 전체 공통 기능이 아니므로 지원 조건 사전 확인 |
