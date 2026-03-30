# postgres 추천 설정

## postgresql.conf

| 시스템 메모리 | m8 (≤8GB) | m64 (>8GB~<128GB) | m128 (≥128GB) | 설명 |
|---------------------------------|-----------|-------------------|---------------|------------------|
| max_connections                 | 200       | 500               | 800           | 최대 연결 수 |
| shared_buffers                  | 1024MB    | 2048MB            | 8192MB        | 공유 버퍼 (RAM의 25%) |
| work_mem                        | 4MB       | 8MB               | 32MB          | 정렬/해시 연산당 메모리 |
| maintenance_work_mem            | 128MB     | 256MB             | 1GB           | VACUUM/인덱스 빌드용 |
| effective_cache_size            | 2GB       | 6GB               | 22GB          | 플래너 캐시 추정값 |
| max_wal_size                    | 8GB       | 16GB              | 16GB          | 체크포인트 간 최대 WAL |
| min_wal_size                    | 1GB       | 2GB               | 2GB           | 최소 WAL 파일 크기 |
| checkpoint_completion_target    | 0.9       | 0.9               | 0.9           | 체크포인트 분산률 |
| wal_buffers                     | 16MB      | 16MB              | 16MB          | WAL 쓰기 버퍼 |
| max_worker_processes            | 4         | 8                 | 16            | 백그라운드 워커 수 |
| max_parallel_workers            | 2         | 4                 | 8             | 병렬 쿼리 워커 수 |
| max_parallel_workers_per_gather | 1         | 2                 | 4             | 쿼리당 병렬 워커 수 |
| random_page_cost                | 1.1       | 1.1               | 1.1           | SSD 기준 랜덤 I/O 비용 |