# S3 PutObject - Checksum 무결성 검증

## checksumSHA256

클라이언트가 업로드 데이터의 SHA-256 해시를 미리 계산해서 요청에 포함하면,
S3가 수신 후 직접 검증한다.

### 동작 흐름

```
클라이언트                           S3
  │  SHA-256 계산                     │
  │  → base64 인코딩                  │
  │                                   │
  ├─── PUT + x-amz-checksum-sha256 ──►│
  │                                   │ SHA-256 재계산
  │                                   │ 값 비교
  │◄─── 400 BadChecksum (불일치) ───-─┤
  │  or                               │
  │◄─── 200 OK (일치) ──────────────-─┤
```

### 요청 헤더 (SDK 내부 처리)

```
x-amz-checksum-sha256: <base64-encoded SHA-256>
x-amz-sdk-checksum-algorithm: SHA256
```

### base64Sha256 계산

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(fileBytes);
String base64Sha256 = Base64.getEncoder().encodeToString(hash);
```

### PutObject 요청

```java
PutObjectRequest request = PutObjectRequest.builder()
    .bucket(bucket)
    .key(key)
    .checksumSHA256(base64Sha256)
    .build();
```

### 효과

| 항목 | 설명 |
|------|------|
| **무결성 보장** | 전송 중 데이터 손상/변조 감지 |
| **S3 저장 메타데이터** | 체크섬을 객체 메타데이터로 저장 |
| **GetObject 검증** | 다운로드 시 `ChecksumMode.ENABLED`로 재검증 가능 |
| **불일치 시** | `400 BadDigest` 또는 `InvalidRequest` 에러 반환 |

### GetObject에서 재검증

```java
GetObjectRequest getRequest = GetObjectRequest.builder()
    .bucket(bucket)
    .key(key)
    .checksumMode(ChecksumMode.ENABLED)
    .build();
```

SDK가 수신 데이터와 저장된 체크섬을 자동 비교하며, 불일치 시 예외를 던진다.

### checksumSHA256 vs Content-MD5

| | `checksumSHA256` | `Content-MD5` |
|--|--|--|
| 알고리즘 | SHA-256 | MD5 |
| S3 메타데이터 저장 | O | X |
| GetObject 재검증 | O | X |
| AWS 권장 여부 | 권장 | 레거시 |

### 주의사항

- Multipart Upload에서는 파트별로 체크섬을 별도 설정해야 한다.
- `contentMD5()`와 동시에 설정하면 충돌 가능하므로 하나만 사용 권장.

---

## checksumSHA256 + Presigned URL

Presigned URL 생성 시 `checksumAlgorithm`을 지정하면,
해당 URL로 업로드하는 외부 클라이언트(브라우저, curl 등)도 체크섬 헤더를 반드시 포함해야 한다.

### 동작 흐름

```
서버 (Presigner)                   외부 클라이언트                S3
  │                                   │                            │
  │ checksumAlgorithm(SHA256) 지정    │                            │
  │ → Presigned URL 생성              │                            │
  │──────── URL 전달 ────────────────►│                            │
  │                                   │                            │
  │                         [외부 클라이언트 처리]                 │
  │                                   │ SHA-256 계산               │
  │                                   │ base64 인코딩              │
  │                                   │                            │
  │                                   │ PutObjectRequest 구성      │
  │                                   │  - URL: presigned URL      │
  │                                   │  - Header:                 │
  │                                   │    x-amz-checksum-sha256   │
  │                                   │  - Body: 파일 데이터       │
  │                                   │                            │
  │                                   ├──── PUT /bucket/key ──────►│
  │                                   │     ?X-Amz-Signature=...   │
  │                                   │     x-amz-checksum-sha256  │
  │                                   │                            │ 서명 검증
  │                                   │                            │ SHA-256 재계산
  │                                   │                            │ 값 비교
  │                                   │◄───────── 200 OK ──────────┤
  │                                   │  or                        │
  │                                   │◄──── 400 BadChecksum ──────┤
```

### Presigned URL 생성 (서버)

```java
PutObjectRequest putObjectRequest = PutObjectRequest.builder()
    .bucket(bucket)
    .key(key)
    .checksumAlgorithm(ChecksumAlgorithm.SHA256)  // 알고리즘만 지정
    .build();

PutObjectPresignRequest presignRequest = PutObjectPresignRequest.builder()
    .signatureDuration(Duration.ofMinutes(10))
    .putObjectRequest(putObjectRequest)
    .build();

PresignedPutObjectRequest presignedRequest = presigner.presignPutObject(presignRequest);
String presignedUrl = presignedRequest.url().toString();
```

### 업로드 (외부 클라이언트)

클라이언트가 직접 SHA-256을 계산해서 헤더에 포함해야 한다.

**curl 예시:**
```bash
CHECKSUM=$(sha256sum file.bin | awk '{print $1}' | xxd -r -p | base64)
curl -X PUT "$PRESIGNED_URL" \
  -H "x-amz-checksum-sha256: $CHECKSUM" \
  --data-binary @file.bin
```

**Java 예시:**
```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(fileBytes);
String base64Sha256 = Base64.getEncoder().encodeToString(hash);

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(presignedUrl))
    .header("x-amz-checksum-sha256", base64Sha256)
    .PUT(HttpRequest.BodyPublishers.ofByteArray(fileBytes))
    .build();
```

### 직접 PutObject vs Presigned URL 비교

| | 직접 PutObject | Presigned URL |
|--|--|--|
| 체크섬 계산 주체 | SDK가 자동 처리 | 외부 클라이언트가 직접 계산 |
| 헤더 추가 주체 | SDK 내부 처리 | 클라이언트가 직접 추가 |
| 서버 코드 노출 | 서버가 직접 업로드 | URL만 전달하면 됨 |
| 사용 시나리오 | 서버 → S3 | 브라우저/모바일 → S3 직접 업로드 |

### 주의 사항

- Presigned URL 생성 시 `checksumAlgorithm`을 지정했으면 클라이언트가 반드시 체크섬 헤더를 포함해야 한다. 누락 시 `400` 에러.
- `checksumSHA256(base64값)`은 직접 PutObject 전용이며, Presigner에서는 `checksumAlgorithm(ChecksumAlgorithm.SHA256)`만 지정한다 (값은 클라이언트가 계산).

---

## checksumSHA256 + Presigned Multipart Upload

Multipart Upload에서 Presigned Part URL을 발행할 때도 `checksumAlgorithm`을 지정할 수 있다.
단, `CreateMultipartUpload`에서 알고리즘을 지정하면 모든 파트에 일관되게 적용해야 하며,
`CompleteMultipartUpload` 시에도 각 파트의 체크섬을 포함해야 한다.

### 동작 흐름

```
서버                              외부 클라이언트                  S3
  │                                   │                            │
  │ CreateMultipartUpload             │                            │
  │  checksumAlgorithm(SHA256) ───────────────────────────────────►│
  │◄──────────────────────────────────────────── uploadId ─────────┤
  │                                   │                            │
  │ Part URL 생성 (파트별 반복)       │                            │
  │  UploadPartRequest                │                            │
  │   partNumber, uploadId            │                            │
  │   checksumAlgorithm(SHA256)       │                            │
  │  → Presigned Part URL             │                            │
  │──────── Part URL 전달 ───────────►│                            │
  │                                   │                            │
  │                        [외부 클라이언트 처리 - 파트별 반복]    │
  │                                   │ 파트 데이터 SHA-256 계산   │
  │                                   │ base64 인코딩              │
  │                                   │                            │
  │                                   ├──── PUT /bucket/key ──────►│
  │                                   │     ?partNumber=N          │
  │                                   │     &uploadId=...          │
  │                                   │     &X-Amz-Signature=...   │
  │                                   │     x-amz-checksum-sha256  │
  │                                   │                            │ 파트 체크섬 검증
  │                                   │◄── ETag + 파트 체크섬 ─────┤
  │                                   │                            │
  │◄──────── ETag + checksum 전달 ────┤                            │
  │                                   │                            │
  │ CompleteMultipartUpload           │                            │
  │  parts: [{partNumber, eTag,       │                            │
  │           checksumSHA256}, ...]   │                            │
  │───────────────────────────────────────────────────────────────►│
  │                                   │                            │ 전체 객체 조립
  │◄─────────────────────────────────────────────── 200 OK ────────┤
```

### 1단계: CreateMultipartUpload (서버)

```java
CreateMultipartUploadRequest createRequest = CreateMultipartUploadRequest.builder()
    .bucket(bucket)
    .key(key)
    .checksumAlgorithm(ChecksumAlgorithm.SHA256)  // 전체 업로드에 적용
    .build();

CreateMultipartUploadResponse createResponse = s3Client.createMultipartUpload(createRequest);
String uploadId = createResponse.uploadId();
```

### 2단계: Presigned Part URL 생성 (서버, 파트별 반복)

```java
UploadPartRequest uploadPartRequest = UploadPartRequest.builder()
    .bucket(bucket)
    .key(key)
    .uploadId(uploadId)
    .partNumber(partNumber)
    .checksumAlgorithm(ChecksumAlgorithm.SHA256)  // CreateMultipartUpload와 동일 알고리즘
    .build();

UploadPartPresignRequest presignRequest = UploadPartPresignRequest.builder()
    .signatureDuration(Duration.ofMinutes(30))
    .uploadPartRequest(uploadPartRequest)
    .build();

PresignedUploadPartRequest presignedPart = presigner.presignUploadPart(presignRequest);
String partUrl = presignedPart.url().toString();
```

### 3단계: 파트 업로드 (외부 클라이언트, 파트별 반복)

**curl 예시:**
```bash
CHECKSUM=$(sha256sum part1.bin | awk '{print $1}' | xxd -r -p | base64)
curl -X PUT "$PART_URL" \
  -H "x-amz-checksum-sha256: $CHECKSUM" \
  --data-binary @part1.bin
# 응답 헤더에서 ETag와 x-amz-checksum-sha256 값을 저장해야 함
```

### 4단계: CompleteMultipartUpload (서버)

클라이언트로부터 각 파트의 ETag와 체크섬을 수신한 후 완료 요청:

```java
List<CompletedPart> completedParts = List.of(
    CompletedPart.builder()
        .partNumber(1)
        .eTag(part1ETag)
        .checksumSHA256(part1Checksum)  // 클라이언트가 전달한 파트 체크섬
        .build(),
    CompletedPart.builder()
        .partNumber(2)
        .eTag(part2ETag)
        .checksumSHA256(part2Checksum)
        .build()
);

CompleteMultipartUploadRequest completeRequest = CompleteMultipartUploadRequest.builder()
    .bucket(bucket)
    .key(key)
    .uploadId(uploadId)
    .multipartUpload(CompletedMultipartUpload.builder().parts(completedParts).build())
    .build();

s3Client.completeMultipartUpload(completeRequest);
```

### 최종 객체의 체크섬 방식

Multipart Upload 완료 후 S3에 저장되는 체크섬은 **COMPOSITE** 방식이다.
각 파트의 SHA-256을 이어붙인 후 다시 SHA-256을 계산한 값이 저장된다.

```
최종 체크섬 = BASE64( SHA256( SHA256(part1) + SHA256(part2) + ... ) )
```

GetObject 시 `ChecksumMode.ENABLED`로 재검증하면 이 COMPOSITE 체크섬으로 비교한다.

### 주의사항

- `CreateMultipartUpload`에서 지정한 `checksumAlgorithm`과 각 `UploadPart`의 알고리즘이 반드시 일치해야 한다.
- `CompleteMultipartUpload`에 각 파트의 체크섬을 누락하면 `400` 에러가 발생한다.
- 파트 최소 크기는 5MB (마지막 파트 제외)이므로 소용량 파일은 단순 PutObject가 적합하다.
