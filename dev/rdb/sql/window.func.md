# 윈도우의 정의

**윈도우(Window)**는 SELECT된 행들의 집합 중, 특정 행에 대해 누적/집계 연산을 수행할 때 참조할 수 있도록 정의한 **논리적 행의 집합(범위)**입니다.

관계형 데이터베이스(Relational Database)에서 **윈도우(Window)**는 윈도우 함수를 사용하기 위한 논리적 그룹의 개념입니다. 윈도우는 `OVER(PARTITION BY <column, ...)`로 생성되며, 가장 간단히는 PARTITION BY에 의해 지정된 column에서 `값이 같은 행(row)`들을 묶어서 묶음당 하나의 윈도우로 구성합니다.

**윈도우 함수(Window Function)** 또는 **OLAP 함수**는 집계 함수(Aggregate Function)의 확장 개념입니다. 일반적인 집계 함수는 여러 행을 하나의 결과로 "묶어" 버리지만, 윈도우 함수는 **윈도우라는 범위에 소속된 각 행에 대해 그 범위 안에서 집계나 순위 등을 계산**해서 그 결과를 각 행에 컬럼으로 `추가` 합니다. 일반적인 집계 함수로는 할 수 없는 윈도우내에서 순위(ranking) 또는 현재 행까지의 변화량 연산 등을 수행할 수 있습니다.


# 윈도우(Window) 함수

윈도우(그룹) 별 

윈도우 함수(Window Function)는 SQL에서 **행(Row)의 집합(Window, 즉 "창") 위에서 계산을 수행하여**, **결과를 각 행에 그대로 유지하면서** 집계나 순위 계산 등을 가능하게 하는 함수입니다.

일반적인 `GROUP BY` 집계와 달리, **윈도우 함수는 행을 제거하지 않고 원본 데이터를 유지**하면서 계산 결과를 옆에 추가해줍니다.

---

## 윈도우 함수 핵심 구조

```sql
<윈도우 함수>(<인자>) OVER (
    [PARTITION BY ...]
    [ORDER BY ...]
    [ROWS BETWEEN ...]
)
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

* `OVER (...)`은 윈도우를 정의하는 부분.
* `PARTITION BY <column>`: 윈도우를 구성할 \<column\>, 여기서는 department_id가 같은 값들끼리 하나의 윈도우 그룹을 구성하게 됨.
* `AVG(salary)`: 윈도우 함수. 윈도우 그룹별 평균 값

---

## 주요 특징

| 특징            | 설명                                              |
| ------------- | ----------------------------------------------- |
| 원본 행 유지       | 기존 행(row)을 제거하지 않음                              |
| 그룹 내 연산 가능    | PARTITION BY로 그룹을 나누고 그 안에서 계산 수행               |
| 정렬 기준으로 누적 가능 | ORDER BY와 함께 사용해 누적 계산, 순위 부여 등 가능              |
| 다양한 함수에 적용 가능 | SUM, AVG, COUNT, ROW\_NUMBER, RANK, LEAD, LAG 등 |

---

## 자주 쓰이는 윈도우 함수 예시

| 함수             | 설명                 | 예시 사용 목적           |
| -------------- | ------------------ | ------------------ |
| `ROW_NUMBER()` | 순차적인 고유 번호         | 정렬 순서에 따라 N번째 행 선택 |
| `RANK()`       | 공동 순위 부여 (띄어지는 순위) | 스포츠 순위처럼 동점 고려할 때  |
| `DENSE_RANK()` | 공동 순위 부여 (띄어지지 않음) | 연속적인 순위 부여         |
| `SUM()`        | 누적 합계              | 날짜별 누적 매출 계산       |
| `AVG()`        | 이동 평균 등            | 최근 7일 평균 매출        |
| `LAG()`        | 이전 행의 값 참조         | 전일 대비 변화량          |
| `LEAD()`       | 다음 행의 값 참조         | 다음날 예측값과의 차이       |

---

## 간단 예제

테이블: `sales`

| id | region | date       | sales |
| -- | ------ | ---------- | ----- |
| 1  | A      | 2025-01-01 | 100   |
| 2  | A      | 2025-01-02 | 150   |
| 3  | A      | 2025-01-03 | 200   |
| 4  | B      | 2025-01-01 | 80    |

### 예제: 지역별 누적 합계

```sql
SELECT
  id,
  region,
  date,
  sales,
  SUM(sales) OVER (PARTITION BY region ORDER BY date) AS cumulative_sales
FROM sales;
```

결과:

| id | region | date       | sales | cumulative\_sales |
| -- | ------ | ---------- | ----- | ----------------- |
| 1  | A      | 2025-01-01 | 100   | 100               |
| 2  | A      | 2025-01-02 | 150   | 250               |
| 3  | A      | 2025-01-03 | 200   | 450               |
| 4  | B      | 2025-01-01 | 80    | 80                |

---

## 요약

* 윈도우 함수 = **"행을 유지하면서 집계·순위·이전/다음 참조 등을 계산"**
* `OVER()` 절 안에서 `PARTITION BY`, `ORDER BY` 등을 조합해 윈도우 구성성
* **데이터 분석, 리포트, 시계열 데이터 처리 등에서 매우 유용**

---


## ROWS BETWEEN

`ROWS BETWEEN`은 SQL의 **윈도우 함수 (`OVER()` 절)** 안에서 \*\*"윈도우의 범위"\*\*를 명확하게 지정할 때 사용합니다.

즉, 윈도우 함수가 **어디부터 어디까지의 행을 대상으로 계산할지** 범위를 설정하는 옵션입니다.

---

### 기본 구조

```sql
<윈도우 함수> OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS BETWEEN <시작 범위> AND <끝 범위>
)
```

---

### 예시: 누적 합 구하기

```sql
SELECT 
    employee_id,
    salary,
    SUM(salary) OVER (
        ORDER BY employee_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM employees;
```

#### 해석:

* `SUM(salary)`를 구할 때,
* 현재 행 기준으로 **이전 모든 행부터 현재 행까지**의 합을 구합니다.

---

### 사용 가능한 범위 옵션

| 범위 문법                 | 의미          |
| --------------------- | ----------- |
| `UNBOUNDED PRECEDING` | 맨 처음 행부터    |
| `1 PRECEDING`         | 현재 행에서 1행 앞 |
| `CURRENT ROW`         | 현재 행만       |
| `1 FOLLOWING`         | 현재 행에서 1행 뒤 |
| `UNBOUNDED FOLLOWING` | 맨 마지막 행까지   |

* offset: 
* CURRENT ROW는 현재 연산 대상인 행을 의미한다.

---

### 다양한 예시

#### 1. 누적합 (running total)

```sql
SUM(sales) OVER (
  ORDER BY sale_date
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

→ 과거부터 현재까지 누적 매출

---

#### 2. 이동 평균 (3행)

```sql
AVG(score) OVER (
  ORDER BY test_date
  ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
)
```

→ 현재 행의 이전, 현재, 다음 행 총 3행의 평균

---

#### 3. 현재 행만 보기

```sql
SUM(salary) OVER (
  ORDER BY employee_id
  ROWS BETWEEN CURRENT ROW AND CURRENT ROW
)
```

→ `SUM`이지만 사실상 자기 자신만 포함 → 자기 자신의 값

---

### ✅ ROWS vs RANGE (추가 개념)

* `ROWS`: 물리적 행 기준 (진짜 몇 행 앞뒤인지)
* `RANGE`: 정렬된 값의 범위 기준

예:
`RANGE BETWEEN 100 PRECEDING AND CURRENT ROW`는 **정렬 컬럼이 숫자면 값 기준으로 범위**를 계산합니다.

---
