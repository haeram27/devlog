# AWS SDK for Java v2 - S3Presigner 메서드 간략 용도

- 기준 문서: https://docs.aws.amazon.com/java/api/latest/software/amazon/awssdk/services/s3/presigner/S3Presigner.html
- 정리 기준: 같은 이름의 오버로드(예: Request/Consumer)는 하나로 묶어 설명

| 메서드 | 간략 용도 |
|---|---|
| builder | S3Presigner 빌더 생성 |
| create | 기본 Region/Credential 체인으로 S3Presigner 즉시 생성 |
| presignAbortMultipartUpload | 멀티파트 업로드 중단(Abort)용 사전서명 URL/요청 생성 |
| presignCompleteMultipartUpload | 멀티파트 업로드 완료(Complete)용 사전서명 URL/요청 생성 |
| presignCreateMultipartUpload | 멀티파트 업로드 시작(CreateMultipartUpload)용 사전서명 URL/요청 생성 |
| presignDeleteObject | 객체 삭제(DeleteObject)용 사전서명 URL/요청 생성 |
| presignGetObject | 객체 다운로드(GetObject)용 사전서명 URL/요청 생성 |
| presignHeadBucket | 버킷 메타 확인(HeadBucket)용 사전서명 URL/요청 생성 |
| presignHeadObject | 객체 메타 확인(HeadObject)용 사전서명 URL/요청 생성 |
| presignPutObject | 객체 업로드(PutObject)용 사전서명 URL/요청 생성 |
| presignUploadPart | 멀티파트 파트 업로드(UploadPart)용 사전서명 URL/요청 생성 |

## 메서드 분류 정리

### 1) 클라이언트 생성

- create, builder

### 2) 단일 객체 작업 사전서명

- presignPutObject
- presignGetObject
- presignDeleteObject
- presignHeadObject

### 3) 버킷 메타 확인 사전서명

- presignHeadBucket

### 4) 멀티파트 업로드 사전서명

- presignCreateMultipartUpload
- presignUploadPart
- presignCompleteMultipartUpload
- presignAbortMultipartUpload

## 대표 요청 객체와 주의점

| 메서드(대표) | 대표 Request 타입 | 주의점 |
|---|---|---|
| presignPutObject | PutObjectPresignRequest | 업로드 헤더(Content-Type, x-amz-*)를 실제 요청과 동일하게 맞춰야 서명 검증이 통과 |
| presignGetObject | GetObjectPresignRequest | 만료 시간(서명 TTL)과 캐시 정책을 함께 설계 |
| presignDeleteObject | DeleteObjectPresignRequest | 삭제 권한 위임 범위를 최소화하고 짧은 TTL 권장 |
| presignHeadObject | HeadObjectPresignRequest | 본문 없이 존재/메타 확인 용도로 사용 |
| presignHeadBucket | HeadBucketPresignRequest | 버킷 존재/접근 확인 용도로 사용 |
| presignCreateMultipartUpload | CreateMultipartUploadPresignRequest | 업로드 시작 단계에서 SSE, 메타데이터 정책을 확정 |
| presignUploadPart | UploadPartPresignRequest | 파트 번호별로 별도 사전서명 필요, ETag 수집 필수 |
| presignCompleteMultipartUpload | CompleteMultipartUploadPresignRequest | 완료 시 파트 번호/ETag 정합성 불일치에 주의 |
| presignAbortMultipartUpload | AbortMultipartUploadPresignRequest | 실패/중단 업로드 정리를 자동화해 비용 누수 방지 |

## 사용 시 공통 체크포인트

- 사전서명 URL은 누구나 사용할 수 있으므로 노출 경로(로그, 브라우저 히스토리, 리퍼러)를 관리해야 합니다.
- 가능한 짧은 만료 시간을 사용하고, 필요 시 서버에서 재발급합니다.
- presign 요청 시 포함한 헤더/쿼리 파라미터는 실제 호출에서도 정확히 동일해야 합니다.
- 브라우저 업로드(CORS) 시 버킷 CORS 정책에 메서드/헤더를 미리 허용해야 합니다.
- presigner와 대상 버킷의 리전이 다르면 서명 오류(AuthorizationHeaderMalformed 등)가 발생할 수 있습니다.
