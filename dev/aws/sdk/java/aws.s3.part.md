# sadf

```java
import software.amazon.awssdk.services.s3.model.UploadPartRequest;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.presigner.model.PresignedUploadPartRequest;
import software.amazon.awssdk.services.s3.presigner.model.PresignedGetObjectRequest;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;
import software.amazon.awssdk.services.s3.presigner.model.UploadPartPresignRequest;

import java.time.Duration;

public class S3PresignUtil {

    public String regeneratePartPresignedUrl(S3Presigner presigner, String bucketName, String objectKey, String uploadId, int partNumber) {
        
        // 1. 특정 Part에 대한 UploadPartRequest 생성
        UploadPartRequest uploadPartRequest = UploadPartRequest.builder()
                .bucket(bucketName)
                .key(objectKey)
                .uploadId(uploadId)
                .partNumber(partNumber)
                .build();

        // 2. Presigner 요청 구성 (필요한 유효 시간 설정)
        UploadPartPresignRequest presignRequest = UploadPartPresignRequest.builder()
                .signatureDuration(Duration.ofMinutes(15)) // URL 유효 시간 (예: 15분)
                .uploadPartRequest(uploadPartRequest)
                .build();

        // 3. Presigned URL 생성 및 반환
        return presigner.presignUploadPart(presignRequest).url().toString();
    }
}
```