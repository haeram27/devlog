# UPSERT

```sql
INSERT INTO B (id, name, score)
---
SELECT id, name, score
FROM A
WHERE score >= 80
---
ON CONFLICT (id) DO UPDATE
SET
    name = EXCLUDED.name,
    score = EXCLUDED.score;
```

- ON CONFLICT는 반드시 B 테이블에 유니크 제약(primary key or unique index) 이 있어야 작동, 예제의 경우 B의 id는 primary 이거나 uniq 속성
- `EXCLUDED`는 충돌이 발생되어 제외된 `신규` 값(기존 값 ❌)을 의미
- 충돌 조건이 여러 컬럼일 경우 예: ON CONFLICT (col1, col2)
- 충돌 발생시 기존 값을 유지하려면 `DO UPDATE` 대신 `DO NOTHING` 사용

```sql
ON CONFLICT (id) DO NOTHING;
```
