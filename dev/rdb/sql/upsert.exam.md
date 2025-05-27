# UPSERT

```sql
INSERT INTO B (id, name, score)
SELECT id, name, score
FROM A
WHERE score >= 80
ON CONFLICT (id) DO UPDATE
SET
    name = EXCLUDED.name,
    score = EXCLUDED.score;
	
```

- ON CONFLICT는 반드시 B 테이블에 유니크 제약(primary key or unique index) 이 있어야 작동
- 충돌 조건이 여러 컬럼일 경우 예: ON CONFLICT (col1, col2)
- 전체 레코드를 그대로 유지하고 싶으면 DO NOTHING도 가능:

```sql
ON CONFLICT (id) DO NOTHING;
```
