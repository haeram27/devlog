# POST 메소드

## RequestBody 없는 POST 메소드 사용 

REST API 설계에서 **request body 없는 POST**를 사용하는 주된 이유들:

### 1. 동작(Action)이 리소스 생성이 아닌 경우

POST는 본래 "서버에 작업을 요청"하는 의미를 가진다. 부수 효과(side effect)가 있는 동작은 GET이 아닌 POST를 써야 한다.

```
POST /orders/123/cancel       # 주문 취소 (body 불필요)
POST /accounts/456/activate   # 계정 활성화
POST /sessions/789/refresh    # 세션 갱신
```

### 2. 멱등성(Idempotency)이 보장되지 않는 경우

- `PUT`, `DELETE`는 멱등성이 보장되어야 함
- 같은 요청을 여러 번 보냈을 때 결과가 달라질 수 있는 동작이라면 POST가 적합

### 3. GET의 한계 회피 목적이 아닌 경우

GET은 캐싱, 북마크, 브라우저 히스토리에 남는 등 부수 효과가 없어야 하므로, **상태를 변경하는 동작**은 query parameter가 없더라도 POST를 사용한다.

### 4. RPC 스타일 API

```
POST /payments/process
POST /reports/generate
POST /cache/flush
```

리소스 중심이 아닌 **동작 중심** 설계에서는 body 없이 URL만으로 의도를 표현하기도 한다.

---

### 요약

| 이유 | 설명 |
|------|------|
| 상태 변경 | GET은 안전(safe)해야 하므로 변경 동작엔 POST |
| 비멱등 동작 | 중복 실행 시 결과가 다를 수 있는 경우 |
| RPC 스타일 | 동작(verb) 중심 URL 설계 |
| 의미론적 명확성 | "이 요청은 무언가를 트리거한다"는 의도 표현 |