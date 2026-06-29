# 어플리케이션 개발 중 메모리 문제 케이스

## Heap Memory Leak

```bash
free
```

## File I/O에 의한 Clean Cache 누적
Clean cache가 "언젠가" 회수되긴 하지만, 충분히 빠르지 않으면 OOM이 먼저 발생