# CoundDownLatch

waiting all thread completion

```java
    @Test
    public void CountDownLatchTest() {
        int totalCount = 10;
        CountDownLatch latch = new CountDownLatch(totalCount);
        for (int i = 1; i <= totalCount; i++) {
                final int id = i;
                Thread.startVirtualThread(()->{
                    log.info("thread {} start", id);
                    try {
                        Thread.sleep(3000L);
                    } catch (Exception ex) {
                        log.error(ex.getMessage(), ex);
                    } finally {
                        latch.countDown();
                    }
                    log.info("thread {} end", id);
                });
            }

        try {
            // Wait for all Thread finished or timeout after specified seconds
            boolean completed = latch.await(60L, TimeUnit.SECONDS);
            if (!completed) {
                log.warn("Timeout: {} thread still pending", latch.getCount());
            }
        } catch (java.lang.InterruptedException e) {
            log.warn("latch interrupted: " + e.getMessage());
            Thread.currentThread().interrupt(); // Restore the interrupt status after catching InterruptedException
        } catch (Exception e) {
            log.error("Error while waiting for all thread completion: {}", e.getMessage(), e);
        }
    }
```
