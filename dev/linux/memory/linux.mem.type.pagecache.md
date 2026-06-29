# PageCache

## Anonymous Pages vs File-backed Pages

## Anonymous Page (익명 페이지)

- malloc(), new로 할당한 메모리
- 스택, 힙
- non-file 대상 mmap()으로 매핑한 파일
- 디스크 백업 없음
- Swap 가능 대상

## File-backed Pages (파일 백업 페이지)

- file 대상 mmap()으로 매핑한 파일
- 실행 파일 코드 영역
- 공유 라이브러리
- Page Cache
- 디스크 백업 있음
- Swap 대상 아님, 메모리 회수 필요시 그냥 버림 (evict)

## 메모리 Page 분류와 회수 방식

### 메모리 페이지 분류와 회수 방식

```text
                     프로세스 메모리
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
   Anonymous Pages                   File-backed Pages
   (디스크 백업 없음)                 (디스크 백업 있음)
        │                                   │
        │                                   │
    ┌-──┴─-──┐                        ┌─────┴──-───┐
    │ malloc │                        │ mmap(file) │
    │ stack  │                        │ execve     │
    │ heap   │                        │ shared lib │
    │ anon   │                        │ Page Cache │
    │ mmap   │                        └─────┬────-─┘
    └───┬─-──┘                              │
        │                                   │
  메모리 압박 발생                     메모리 압박 발생
        │                                   │
        ↓                                   ↓
   Swap 영역에 쓰기                    디스크에서 버림
   (Swap I/O 발생)                     (I/O 없음 if clean)
        │                                   │
        ↓                                   ↓
   나중에 Swap에서 읽음              나중에 원본 파일에서 읽음
   (Swap in)                           (Page fault)
```

#### File-backed mmap vs Anonymous mmap

```txt
// File-backed mmap (Swap 안 됨)
int fd = open("data.dat", O_RDWR);
void *ptr = mmap(NULL, size, PROT_READ|PROT_WRITE, 
                 MAP_SHARED, fd, 0);
// → File Page Cache와 동일
// → 회수 시 그냥 버림 (원본 파일에서 다시 읽음)

// Anonymous mmap (Swap 됨!)
void *ptr = mmap(NULL, size, PROT_READ|PROT_WRITE,
                 MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);
// → Anonymous 페이지와 동일
// → 회수 시 Swap으로 내보냄
```


## 10. 핵심 정리

### 질문에 대한 답

> "File Page Cache도 Swap의 대상이 되나?"

**절대 아닙니다!**

**이유:**

1. **백업이 이미 있음**
   - Page Cache는 디스크 파일의 복사본
   - Swap에 또 쓸 필요 없음
   - 그냥 버리고 나중에 원본 파일 읽으면 됨

2. **Swap vs Drop**
```
   Anonymous Pages → Swap (디스크 쓰기 필요)
   File Page Cache → Drop (그냥 버림)
```

3. **성능 차이**
```
   Page Cache 회수: 매우 빠름 (버리기만)
   Anonymous Swap: 느림 (Swap 쓰기 I/O)
```

### swappiness의 진짜 의미

```
vm.swappiness = 회수 비율 조절

swappiness=10:
  → Page Cache 90% 회수 (drop)
  → Anonymous 10% 회수 (swap)

swappiness=60:
  → Page Cache 70% 회수 (drop)
  → Anonymous 30% 회수 (swap)
```

### 메모리 타입별 요약표

| 타입 | 예시 | 회수 방식 | Swap 대상 | 비고 |
|------|------|----------|----------|------|
| Anonymous | malloc, stack | Swap out | ✅ Yes | 디스크 백업 없음 |
| File Cache | cat, mmap(file) | Drop/Evict | ❌ No | 원본 파일이 백업 |
| Dirty Cache | write(), 수정된 mmap | Writeback → Drop | ❌ No | 먼저 파일에 씀 |
| Clean Cache | read(), 읽기 전용 | Drop | ❌ No | 즉시 버림 |
