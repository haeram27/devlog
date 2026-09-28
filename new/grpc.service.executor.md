# gRPC Service Executor

GrpcServiceExecutor 예:

```
package com.example.grpc.server.util;

import com.google.protobuf.InvalidProtocolBufferException;
import com.google.protobuf.MessageOrBuilder;
import com.google.protobuf.util.JsonFormat;
import io.grpc.stub.StreamObserver;
import java.util.function.Supplier;
import org.slf4j.Logger;

public final class GrpcServiceExecutor {
  private static final JsonFormat.Printer JSON_PRINTER =
      JsonFormat.printer().omittingInsignificantWhitespace();

  private GrpcServiceExecutor() {
  }

  public static <T> void execute(StreamObserver<T> observer, Supplier<T> supplier) {
    observer.onNext(supplier.get());
    observer.onCompleted();
  }

  public static void logRequestJson(Logger logger, String context, MessageOrBuilder message) {
    String requestJson;
    try {
      requestJson = JSON_PRINTER.print(message);
    } catch (InvalidProtocolBufferException e) {
      logger.warn("{} - failed to render request as JSON: {}", context, e.getMessage());
      requestJson = message.toString();
    }
    logger.info("{} called with request={}", context, requestJson);
  }
}
```

GrpcService

```java
@RequiredArgsConstructor
public class ConsoleReportHelloGrpcService extends ConsoleReportHelloServiceGrpc.ConsoleReportHelloServiceImplBase {

    private final ConsoleReportHelloService consoleReportService;

/* Header key is always lowercase in gRPC Metadata, so use lowercase for the header key when testing with grpcurl or other gRPC clients.
grpcurl -plaintext \
-H 'x-grpc-header: myheader' \
-d '{"message":"hello"}' \
localhost:10001 \
com.example.GrpcHelloService/Hello
*/
    @Override
    public void hello(HelloRequest request, StreamObserver<HelloResponse> responseObserver) {
        GrpcServiceExecutor.logRequestJson(log, "ConsoleReportGrpcService#hello", request);
        log.info("tenantId: {}", getTenantIds());
        GrpcServiceExecutor.execute(responseObserver, () -> {
            validateNotBlank(request.getMessage(), "message");
            HelloResult result = consoleReportService.hello(request.getMessage());
            return HelloResponse.newBuilder()
                .setMessage(result.message())
                .setHeaders(result.headers())
                .build();
        });
    }

    private static void validateNotBlank(String value, String fieldName) {
        if (value == null || value.isBlank()) {
            log.error("Validation failed: {} is blank", fieldName);
            throw new IllegalArgumentException(fieldName + " must not be blank");
        }
    }

    private List<String> getTenantIds() {
      return TenantContext.getTenantIds();
    }
}

```