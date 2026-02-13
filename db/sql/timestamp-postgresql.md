# timestamp handling - postgresql

## `문자열 DATE/TIME 표현`의 timestamptz 자동 캐스팅

### `문자열 DATE/TIME 표현`은 AT TIME ZONE 적용시 현재 세션 timezone의 timestamptz로 간주된다

```sql
SELECT '2025-06-01' AT TIME ZONE 'Asia/Seoul';                   -- 2025-06-01 09:00:00.000
SELECT '2025-06-01T00:00:00' AT TIME ZONE 'Asia/Seoul';          -- 2025-06-01 09:00:00.000
SELECT '2025-06-01T00:00:00.000+0000' AT TIME ZONE 'Asia/Seoul'; -- 2025-06-01 09:00:00.000
```

위 `AT TIME ZONE` SELECT 문에서 실행 결과는 모두 `2025-06-01 09:00:00.000` 이 된다.

***`AT TIME ZONE`의 적용대상이 되는 문자열 시간 표현식이 `현재 세션 timezone`의 `timestamptz`(timestamp with timezone) 형식으로 자동 캐스팅***되기 때문이다.
현재 세션의 timezone은 별도의 설정을 하지 않은 경우 default는 `UTC` 이다.

- '2025-06-01T00:00:00'  =>  2025-06-01 00:00:00.000+0000
- '2025-06-01'           =>  2025-06-01 00:00:00.000+0000

## genrate_series() 함수에서 `문자열 DATE/TIME 표현`은 현재 세션 timezone의 timestamptz로 간주된다

generate_series() 함수는 지정된 폐구간안에서 series 값을 갖는 테이블을 반환하는 함수이다.

syntax:

```text
generate_series(start, end, interval)
```

interval 생략시 default 1

예:

```sql
SELECT generate_series(1,3);   -- 1, 3, 5, 7
SELECT generate_series(1,7,2); -- 1, 3, 5, 7
```

generate_series를 이용해 `DATE/TIME` series를 생성할때, start와 end로 지정하는 `문자열 DATE/TIME 표현`은 자동으로 timestamptz로 캐스팅 된 후 연산된다.

예:

```sql
SELECT generate_series(
    '2025-06-01',
    '2025-06-05',
    INTERVAL '1 day'
) AS timeline;


SELECT generate_series(
    '2025-06-01T00:00:00',
    '2025-06-05T00:00:00',
    INTERVAL '1 day'
) AS timeline;
```

위 두 예제는 동일하게 아래의 결과를 출력한다.

```text
timeline
-----------------------------
2025-06-01 00:00:00.000 +0000
2025-06-02 00:00:00.000 +0000
2025-06-03 00:00:00.000 +0000
2025-06-04 00:00:00.000 +0000
2025-06-05 00:00:00.000 +0000
```
