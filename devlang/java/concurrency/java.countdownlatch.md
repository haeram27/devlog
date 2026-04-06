# CoundDownLatch

waiting all thread completion

```java
    int totalCount = 10;
    CountDownLatch latch = new CountDownLatch(totalCount);
    for (int count = 1; count <= totalCount; count++) {
            Thread.startVirtualThread(()->{
                log.info("thread {} start", count);
                try {
                    Thread.sleep(3000);
                } catch (Exception ex) {
                    log.error(ex.getMessage(), ex);
                } finally {
                    latch.countDown();
                }
                log.info("thread {} end", count);
            });
        }

    try {
        // Wait for all Thread finished or timeout after specified seconds
        boolean completed = latch.await(timeoutSeconds, TimeUnit.SECONDS);
        if (!completed) {
            log.warn("Timeout: {} thread still pending", latch.getCount());
        }
    } catch (java.lang.InterruptedException e) {
        log.warn("Page collection interrupted: " + e.getMessage());
        Thread.currentThread().interrupt(); // Restore the interrupt status after catching InterruptedException
    } catch (Exception e) {
        log.error("Error while waiting for all thread completion: {}", e.getMessage(), e);
    }
```
