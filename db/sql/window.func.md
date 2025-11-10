- [윈도우의 정의](#윈도우의-정의)
  - [윈도우, 파티션, 프레임의 계층 구조](#윈도우-파티션-프레임의-계층-구조)
  - [윈도우(Window) 함수](#윈도우window-함수)
    - [윈도우 함수 핵심 구조](#윈도우-함수-핵심-구조)
    - [윈도우 함수 주요 특징](#윈도우-함수-주요-특징)
    - [자주 쓰이는 윈도우 함수 예시](#자주-쓰이는-윈도우-함수-예시)
    - [윈도우 함수 간단 예제](#윈도우-함수-간단-예제)
    - [결과 분석](#결과-분석)
    - [상세 결과](#상세-결과)
  - [윈도우 프레임](#윈도우-프레임)
    - [파티션 vs 윈도우 프레임 비교](#파티션-vs-윈도우-프레임-비교)
    - [윈도우 프레임 종류](#윈도우-프레임-종류)
    - [윈도우 프레임 정의 정의 방법 세가지](#윈도우-프레임-정의-정의-방법-세가지)
    - [기본 윈도우 프레임 (중요)](#기본-윈도우-프레임-중요)
    - [`BETWEEN`에 사용 가능한 범위 옵션](#between에-사용-가능한-범위-옵션)
      - [ROWS BETWEEN 예시](#rows-between-예시)
    - [RANGE vs ROWS vs GROUPS](#range-vs-rows-vs-groups)
      - [ROWS (물리적 행)](#rows-물리적-행)
      - [RANGE (값 기준) - 기본값](#range-값-기준---기본값)
      - [GROUPS (그룹 기준)](#groups-그룹-기준)
  - [`GROUP BY`와 `PARTITION BY` 차이](#group-by와-partition-by-차이)
    - [예제](#예제)
      - [1. `GROUP BY` 예제 — 부서별 총 매출](#1-group-by-예제--부서별-총-매출)
      - [2. `PARTITION BY` 예제 — 각 행에서 부서별 누적 매출](#2-partition-by-예제--각-행에서-부서별-누적-매출)
  - [예제 : 지역별 누적 합계](#예제--지역별-누적-합계)

# 윈도우의 정의

**윈도우(Window)**는 SELECT된 행들의 집합 중, 특정 행에 대해 누적/집계 연산을 수행할 때 참조할 수 있도록 정의한 **논리적 행의 집합(범위)**입니다.

관계형 데이터베이스(Relational Database)에서 **윈도우(Window)**는 윈도우 함수를 사용하기 위한 논리적 그룹의 개념입니다. 윈도우는 `OVER` 절에 의해서 정의 되며, "윈도우 함수"의 "윈도우"는 **"창문을 통해 주변을 보듯이, 현재 행에서 다른 행들을 볼 수 있는 관점/범위"**를 의미합니다.

| 관점 | 의미 |
|------|------|
| **어원** | 실제 창문(window)에서 유래 |
| **개념** | 현재 행에서 다른 행들을 "보는" 범위/시야 |
| **기술적** | 슬라이딩 윈도우 패턴 (이동하는 관찰 범위) |
| **SQL적** | 개별 행을 유지하면서 다른 행들을 참조하는 메커니즘 |

"윈도우 함수"의 "윈도우"는 **"창문을 통해 주변을 보듯이, 현재 행에서 다른 행들을 볼 수 있는 관점/범위"**를 의미합니다

**윈도우 함수(Window Function)** 또는 **OLAP 함수**는 집계 함수(Aggregate Function)의 확장 개념입니다. 일반적인 집계 함수는 여러 행을 하나의 결과로 "묶어" 버리지만, 윈도우 함수는 **윈도우라는 범위에 소속된 각 행에 대해 그 범위(윈도우 프레임) 안에서 집계나 순위 등을 계산**해서 그 결과를 각 행에 컬럼으로 `추가` 합니다. 일반적인 집계 함수로는 할 수 없는 윈도우내에서 순위(ranking) 또는 현재 행까지의 변화량 연산 등을 수행할 수 있습니다.

PostgreSQL 문서에서는 이렇게 구분합니다:

- Window definition (윈도우 정의): OVER 절 전체
- Partition (파티션): PARTITION BY로 나눈 그룹
- Window frame (윈도우 프레임): ROWS/RANGE/GROUPS로 지정, 현재행에서 함수 연산시 참조할 파티션내 데이터의 범위, 함수가 바라보는 연산의 영역, 파티션 경계를 넘지 못함

## 윈도우, 파티션, 프레임의 계층 구조

```text
┌─────────────────────────────────────────────────┐
│  Window = OVER 절 전체                          │
│  ┌───────────────────────────────────────────┐  │
│  │  Partition (파티션)                       │  │
│  │  - PARTITION BY로 나눈 그룹               │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  Window Frame (윈도우 프레임)       │  │  │
│  │  │  - 각 행의 실제 계산 범위           │  │  │
│  │  │  - ROWS BETWEEN으로 지정            │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

`PARTITIION BY`를 생략하면 `윈도우 = 파티션`이 됩니다.
프레임 정의(ROWS/RANGE/GROUPS)를 생략하면 `프레임 = 파티션`이 됩니다.

## 윈도우(Window) 함수

윈도우 함수(Window Function)는 SQL에서 **행(Row)의 집합(Window frame) 위에서 계산을 수행하여**, **결과를 각 행에 그대로 유지하면서** 집계나 순위 계산 등을 가능하게 하는 함수입니다.

윈도우는 OVER(...) 구문을 이용하여 생성 합니다.

일반적인 `GROUP BY` 집계와 달리, **윈도우 함수를 사용하기 위한 PARTITION BY는 묶음시 행을 제거하지 않고 원본 데이터를 유지**하면서 계산 결과를 옆에 추가해줍니다.

- 윈도우 함수 = **"행을 유지하면서 집계·순위·이전/다음 참조 등을 계산"**
- `OVER()` 절 안에서 `PARTITION BY`, `ORDER BY`, `ROWS BETWEEN ... AND ...` 등을 조합해 윈도우 구성
- **데이터 분석, 리포트, 시계열 데이터 처리 등에서 매우 유용**

---

### 윈도우 함수 핵심 구조

```sql
SELECT 
    column,
    window_function() OVER (
        PARTITION BY group_column    -- 파티션: 데이터를 그룹으로 나눔
        ORDER BY sort_column         -- 정렬: 파티션 내에서 순서 지정
        ROWS BETWEEN ... AND ...     -- 윈도우 프레임: 현재 행에서 함수를 수행할 때 연산의 대상이 되는 범위
    )
FROM table;
```

예:

```sql
SUM(sales) OVER (PARTITION BY region ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

윈도우 함수 예제 Query: 부서별 salary 평균

```sql
SELECT
  employee_id,
  department_id,
  salary,
  AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary
FROM employees;
```

- `OVER (...)`은 윈도우를 정의하는 부분.
- `PARTITION BY <column>`: 파티션 정의, 여기서는 department_id가 같은 값들끼리 하나의 파티션을 구성하게 됨.
- `AVG(salary)`: 윈도우 함수. 윈도우 그룹별 평균 값

---

### 윈도우 함수 주요 특징

| 특징 | 설명 |
| --- | --- |
| 원본 행 유지 | 기존 행(row)을 제거하지 않음 |
| 파티션 내 연산 가능 | PARTITION BY로 그룹을 나누고 그 안에서 계산 수행 |
| 정렬 기준으로 누적 가능 | ORDER BY와 함께 사용해 누적 계산, 순위 부여 등 가능 |
| 다양한 함수에 적용 가능 | SUM, AVG, COUNT, ROW\_NUMBER, RANK, LEAD, LAG 등 |

---

### 자주 쓰이는 윈도우 함수 예시

| 함수 | 설명 | 예시 사용 목적 |
| -------------- | ------------------ | ------------------ |
| `ROW_NUMBER()` | 순차적인 고유 번호 | 정렬 순서에 따라 N번째 행 선택 |
| `RANK()`       | 공동 순위 부여 (동순위 발생시 동순위 수 만큼 다음 순위 띄어지는 순위, 1-2-2-4) | 스포츠 순위처럼 동점 고려할 때 |
| `DENSE_RANK()` | 공동 순위 부여 (동순위 발생하더라도 다음 순위는 띄어지지 않음, 1-2.2-3) | 연속적인 순위 부여 |
| `SUM()`        | 누적 합계 | 날짜별 누적 매출 계산 |
| `AVG()`        | 이동 평균 등 | 최근 7일 평균 매출 |
| `LAG()`        | 이전 행의 값 참조 | 전일 대비 변화량 |
| `LEAD()`       | 다음 행의 값 참조 | 다음날 예측값과의 차이 |

---

### 윈도우 함수 간단 예제

```sql
-- 샘플 데이터
CREATE TABLE sales (
    product VARCHAR(10),
    day INT,
    amount INT
);

INSERT INTO sales VALUES
('A', 1, 100),
('A', 2, 150),
('A', 3, 120),
('A', 4, 180),
('B', 1, 200),
('B', 2, 220),
('B', 3, 210);

-- 예시 쿼리
SELECT 
    product,
    day,
    amount,
    SUM(amount) OVER (
        PARTITION BY product              -- 파티션
        ORDER BY day
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING  -- 윈도우 프레임
    ) AS moving_sum
FROM sales
ORDER BY product, day;
```

### 결과 분석

```text
제품 A의 파티션:
┌────────────────────────────────┐
│  day 1: 100                    │  ← 파티션 시작
│  day 2: 150                    │
│  day 3: 120                    │
│  day 4: 180                    │  ← 파티션 끝
└────────────────────────────────┘

day 2일 때의 윈도우 프레임:
     [1 PRECEDING]  [CURRENT]  [1 FOLLOWING]
         day 1        day 2       day 3
         100    +     150    +    120     = 370
         └──────────────────────────┘
              윈도우 프레임
```

### 상세 결과

```text
product | day | amount | moving_sum | 설명
--------|-----|--------|------------|---------------------------
A       | 1   | 100    | 250        | 100 + 150 (앞 행 없음)
A       | 2   | 150    | 370        | 100 + 150 + 120
A       | 3   | 120    | 450        | 150 + 120 + 180
A       | 4   | 180    | 300        | 120 + 180 (뒤 행 없음)
B       | 1   | 200    | 420        | 200 + 220 (새 파티션 시작)
B       | 2   | 220    | 630        | 200 + 220 + 210
B       | 3   | 210    | 430        | 220 + 210
```

---

## 윈도우 프레임

### 파티션 vs 윈도우 프레임 비교

| 개념 | 설명 | 예시 |
|------|------|------|
| **Partition** | 데이터를 논리적으로 나누는 그룹 | 제품별, 부서별 |
| **Window Frame** | 각 행마다 실제 계산에 사용되는 범위 (파티션을 벗어나지 못함) | 현재 행 ± N개 행 |

```sql
-- 파티션만 있는 경우 (윈도우 프레임 = 전체 파티션)
SELECT 
    product,
    SUM(amount) OVER (PARTITION BY product) AS total_per_product
FROM sales;

-- 결과: 각 파티션의 전체 합계
-- A: 550 (100+150+120+180)
-- B: 630 (200+220+210)
```

```sql
-- 윈도우 프레임을 명시한 경우
SELECT 
    product,
    day,
    amount,
    SUM(amount) OVER (
        PARTITION BY product
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sum
FROM sales;

-- 결과: 누적 합계 (윈도우가 점점 커짐)
-- A, day 1: 100
-- A, day 2: 250 (100+150)
-- A, day 3: 370 (100+150+120)
-- A, day 4: 550 (100+150+120+180)
```

### 윈도우 프레임 종류

윈도우 프레임 = 각 행의 계산 범위

```text
┌─────────────────────────────────────────┐
│         전체 테이블                     │
│  ┌───────────────────────────────────┐  │
│  │   파티션 1 (PARTITION BY)         │  │
│  │   ┌───────────────────────┐       │  │
│  │   │  윈도우 프레임 1      │       │  │  ← 현재 행의 계산 범위
│  │   └───────────────────────┘       │  │
│  │   ┌───────────────────────┐       │  │
│  │   │  윈도우 프레임 2      │       │  │  ← 다음 행의 계산 범위
│  │   └───────────────────────┘       │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │   파티션 2 (PARTITION BY)         │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 윈도우 프레임 정의 정의 방법 세가지

| 모드 | 기준 | 동작 |
| --- | --- | --- |
| **ROWS** | 물리적 행 개수 | 정확히 N개의 행 |
| **RANGE** | 값의 범위 | ORDER BY 값이 같은 행들을 하나로 취급 (기본값) |
| **GROUPS** | 그룹 개수 | ORDER BY 값이 같은 행 그룹을 하나로 취급 |

```sql
-- 1. ROWS: 물리적 행 수 기준
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING

-- 2. RANGE: 값 기준 (ORDER BY 값이 같은 행 포함)
RANGE BETWEEN INTERVAL '1 day' PRECEDING AND CURRENT ROW

-- 3. GROUPS: 그룹 기준 (ORDER BY 값이 같은 행을 하나의 그룹으로)
GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING
```

### 기본 윈도우 프레임 (중요)

윈도우 프레임 정의를 생략하면 기본 적으로 아래의 `RANGE BETWEEN`으로 프레임이 정해짐
`ORDER  BY`가 생략되면 ROWS와 RANGE의 차이가 없어 진다.

```sql
-- ORDER BY가 있으면
OVER (PARTITION BY x ORDER BY y)
-- 기본값: RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- ORDER BY가 없으면
OVER (PARTITION BY x)
-- 기본값: 전체 파티션 = RANGE|ROWS UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

### `BETWEEN`에 사용 가능한 범위 옵션

| 범위 문법             | 의미        |
| --------------------- | ----------- |
| `UNBOUNDED PRECEDING` | 맨 처음 행부터     |
| `<offset> PRECEDING`  | 현재 행에서 \<offset\>행 앞 |
| `CURRENT ROW`         | 현재 행만          |
| `<offset> FOLLOWING`  | 현재 행에서 \<offset\>행 뒤 |
| `UNBOUNDED FOLLOWING` | 맨 마지막 행까지   |

- offset: 정수 n (n > 0)
- CURRENT ROW는 현재 연산 대상인 행을 의미한다.

#### ROWS BETWEEN 예시

```sql
-- 이동 평균 (3일)
SELECT 
    product,
    day,
    amount,
    AVG(amount) OVER (
        PARTITION BY product           -- 제품별로 나눔
        ORDER BY day
        ROWS BETWEEN 2 PRECEDING       -- 윈도우: 이전 2개 + 현재
             AND CURRENT ROW
    ) AS moving_avg_3day
FROM sales;

-- 누적 합계
SELECT 
    product,
    day,
    amount,
    SUM(amount) OVER (
        PARTITION BY product           -- 제품별로 나눔
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING  -- 윈도우: 처음부터 현재까지
             AND CURRENT ROW
    ) AS cumulative_sum
FROM sales;

-- 전체 파티션 합계 (윈도우 프레임 지정 안 함)
SELECT 
    product,
    day,
    amount,
    SUM(amount) OVER (
        PARTITION BY product           -- 윈도우 = 전체 파티션
    ) AS total_per_product
FROM sales;
```

### RANGE vs ROWS vs GROUPS

```sql
-- 테스트 데이터 (day 값에 중복 있음)
CREATE TABLE sales (
    product VARCHAR(1),
    day INT,
    amount INT
);

INSERT INTO sales VALUES
('A', 1, 100),
('A', 1, 150),  -- day=1 중복
('A', 2, 120),
('A', 3, 180),
('A', 3, 200);  -- day=3 중복
```

#### ROWS (물리적 행)

```sql
SELECT 
    day,
    amount,
    SUM(amount) OVER (
        ORDER BY day
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS sum_rows
FROM sales
WHERE product = 'A'
ORDER BY day, amount;
```

**결과**:

```text
day | amount | sum_rows | 설명
----|--------|----------|------------------------
1   | 100    | 100      | [행1]
1   | 150    | 250      | [행1, 행2] ← 물리적 2개 행
2   | 120    | 270      | [행2, 행3]
3   | 180    | 300      | [행3, 행4]
3   | 200    | 380      | [행4, 행5]
```

#### RANGE (값 기준) - 기본값

```sql
SELECT 
    day,
    amount,
    SUM(amount) OVER (
        ORDER BY day
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS sum_range
FROM sales
WHERE product = 'A'
ORDER BY day, amount;
```

**결과**:

```text
day | amount | sum_range | 설명
----|--------|-----------|----------------------------------
1   | 100    | 250       | day<=1인 모든 행 [행1,행2] ✓
1   | 150    | 250       | day<=1인 모든 행 [행1,행2] ✓ (동일!)
2   | 120    | 370       | day<=2인 모든 행 [행1,행2,행3]
3   | 180    | 750       | day<=3인 모든 행 [행1~행5] ✓
3   | 200    | 750       | day<=3인 모든 행 [행1~행5] ✓ (동일!)
```

#### GROUPS (그룹 기준)

```sql
SELECT 
    day,
    amount,
    SUM(amount) OVER (
        ORDER BY day
        GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS sum_groups
FROM sales
WHERE product = 'A'
ORDER BY day, amount;
```

**결과**:

```text
day | amount | sum_groups | 설명
----|--------|------------|----------------------------------
1   | 100    | 250        | 그룹1[day=1] = [행1,행2]
1   | 150    | 250        | 그룹1[day=1] = [행1,행2] (동일!)
2   | 120    | 370        | 그룹1[day=1] + 그룹2[day=2]
3   | 180    | 500        | 그룹2[day=2] + 그룹3[day=3]
3   | 200    | 500        | 그룹2[day=2] + 그룹3[day=3] (동일!)
```

---

## `GROUP BY`와 `PARTITION BY` 차이

- `GROUP BY`: 그룹별 집계 함수 사용 용도, SELECT 결과로 그룹별 ROW 1개만 남음

그룹별 스코어의 평균 값

```sql
SELECT group_id, avg(score) FROM table GROUP BY group_id
```

- `PARTITION BY`:
  - 윈도우 함수와 함께 사용해서, SELECT 결과에서 그룹별 모든 행 **그대로 유지**
  - 윈도우란 table에서 삭제되지 않은

```sql
SELECT group_id, dense_rank(score) FROM table GROUP BY group_id
```

| 항목 | `GROUP BY` | `PARTITION BY` (윈도우 함수 용) |
| --- | --- | --- |
| **용도** | 집계 (SUM, AVG, COUNT 등)     | 행(row)별 계산을 위해 그룹 내에서 연산 (누적합, 순위 등) |
| **출력 행 수** | 그룹당 1행 (그룹된 결과만 남음) | 원본 행 수 그대로 유지됨 |
| **어디에 사용** | 일반 SQL의 `SELECT` 절에서 사용 | `OVER (...)` 구문을 사용하는 윈도우 함수에서 사용 |
| **함수 예시**  | `COUNT(*)`, `SUM(sales)` 등 | `ROW_NUMBER()`, `RANK()`, `SUM(...) OVER(...)` 등 |

---

### 예제

예제 테이블: `sales`

| id | department | amount |
| -- | ---------- | ------ |
| 1  | HR         | 100    |
| 2  | HR         | 150    |
| 3  | IT         | 200    |
| 4  | IT         | 300    |

---

#### 1. `GROUP BY` 예제 — 부서별 총 매출

```sql
SELECT
  department,
  SUM(amount) AS total_sales
FROM sales
GROUP BY department;
```

결과:

| department | total\_sales |
| ---------- | ------------ |
| HR         | 250          |
| IT         | 500          |

→ **부서별로 ROW 그룹을 묶고, 한 줄씩만 결과 출력됨.**

---

#### 2. `PARTITION BY` 예제 — 각 행에서 부서별 누적 매출

```sql
SELECT
  id,
  department,
  amount,
  SUM(amount) OVER (PARTITION BY department ORDER BY id) AS running_total
FROM sales;
```

결과:

| id | department | amount | running\_total |
| -- | ---------- | ------ | -------------- |
| 1  | HR         | 100    | 100            |
| 2  | HR         | 150    | 250            |
| 3  | IT         | 200    | 200            |
| 4  | IT         | 300    | 500            |

→ **ROW 데이터는 그대로 유지하면서, 각 부서 내에서 누적 합계를 계산.**

## 예제 : 지역별 누적 합계

테이블: `tb_test_sales`

| id | region | date       | sales |
| -- | ------ | ---------- | ----- |
| 1  | A      | 2025-01-01 | 100   |
| 2  | A      | 2025-01-02 | 150   |
| 3  | A      | 2025-01-03 | 200   |
| 4  | B      | 2025-01-01 | 80    |

`tb_test_sales` 테이블 생성

```sql
DROP table if exists tb_test_sales;
DO $$
BEGIN
    CREATE TABLE IF NOT EXISTS tb_test_sales();
    COMMENT ON TABLE tb_test_sales IS 'sales';

    ALTER TABLE IF EXISTS tb_test_sales ADD COLUMN IF NOT EXISTS id bigint NOT NULL;
    ALTER TABLE IF EXISTS tb_test_sales ADD COLUMN IF NOT EXISTS region text NOT NULL;
    ALTER TABLE IF EXISTS tb_test_sales ADD COLUMN IF NOT EXISTS "date" DATE NOT NULL;
    ALTER TABLE IF EXISTS tb_test_sales ADD COLUMN IF NOT EXISTS sales bigint NOT NULL;
    ALTER TABLE IF EXISTS tb_test_sales ADD COLUMN IF NOT EXISTS modified_time timestamptz NOT NULL;

    COMMENT ON COLUMN tb_test_sales.id IS 'id';
    COMMENT ON COLUMN tb_test_sales.region IS 'region';
    COMMENT ON COLUMN tb_test_sales."date" IS 'date';
    COMMENT ON COLUMN tb_test_sales.sales IS 'amount of sales';
    COMMENT ON COLUMN tb_test_sales.modified_time IS 'time to modified data';


    CREATE SEQUENCE IF NOT EXISTS tb_test_sales_id_seq
        START WITH 1
        INCREMENT BY 1
        NO MINVALUE
        NO MAXVALUE
        CACHE 1;

    ALTER SEQUENCE IF EXISTS tb_test_sales_id_seq OWNED BY tb_test_sales.id;

    ALTER TABLE IF EXISTS ONLY tb_test_sales ALTER COLUMN id SET DEFAULT nextval('tb_test_sales_id_seq'::regclass);

    IF NOT EXISTS (SELECT 1 FROM information_schema.table_constraints WHERE table_name = 'tb_test_sales' AND constraint_type = 'PRIMARY KEY') THEN
        ALTER TABLE IF EXISTS tb_test_sales ADD CONSTRAINT tb_test_sales_pk PRIMARY KEY (id);
    END IF;

END $$ LANGUAGE 'plpgsql';


INSERT INTO tb_test_sales (id, region, "date", sales, modified_time)
values
  (1, 'A', '2025-01-01'::DATE, 100, now())
  ,(2, 'A', '2025-01-02'::DATE, 150, now())
  ,(3, 'A', '2025-01-03'::DATE, 200, now())
  ,(4, 'B', '2025-01-01'::DATE, 80, now())
ON CONFLICT(id) DO NOTHING;

SELECT * FROM tb_test_sales;
```

`PARTITION BY`를 사용한 윈도우 누적 합계 연산

```bash
SELECT
  id,
  region,
  date,
  sales,
  SUM(sales) OVER (PARTITION BY region ORDER BY date) AS cumulative_sales
FROM tb_test_sales;
```

결과:

| id | region | date       | sales | cumulative\_sales |
| -- | ------ | ---------- | ----- | ----------------- |
| 1  | A      | 2025-01-01 | 100   | 100               |
| 2  | A      | 2025-01-02 | 150   | 250               |
| 3  | A      | 2025-01-03 | 200   | 450               |
| 4  | B      | 2025-01-01 | 80    | 80                |

---
