## `GROUP BY`와 `PARTITION BY` 차이이


* `GROUP BY`: 집계 함수 사용용도, 그룹별 ROW 1개만 남음

```sql
SELECT SUM(col2) as 
FROM tb_
GROUP BY group
```

* `PARTITION BY`: 윈도우 함수와 함께 사용해서, **원본 행을 유지한 채** 그룹별 계산 → 결과 **그대로 유지**




| 항목         | `GROUP BY`                 | `PARTITION BY` (윈도우 함수 용)                        |
| ---------- | -------------------------- | ------------------------------------------------ |
| **용도**     | 집계 (SUM, AVG, COUNT 등)     | 행(row)별 계산을 위해 그룹 내에서 연산 (누적합, 순위 등)             |
| **출력 행 수** | 그룹당 1행 (그룹된 결과만 남음)        | 원본 행 수 그대로 유지됨                                   |
| **어디에 사용** | 일반 SQL의 `SELECT` 절에서 사용    | `OVER (...)` 구문을 사용하는 윈도우 함수에서 사용                |
| **함수 예시**  | `COUNT(*)`, `SUM(sales)` 등 | `ROW_NUMBER()`, `RANK()`, `SUM(...) OVER(...)` 등 |

---

## 예제

예제 테이블: `sales`

| id | department | amount |
| -- | ---------- | ------ |
| 1  | HR         | 100    |
| 2  | HR         | 150    |
| 3  | IT         | 200    |
| 4  | IT         | 300    |

---

### 1. `GROUP BY` 예제 — 부서별 총 매출

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

→ **부서별로 그룹을 묶고, 한 줄씩만 결과 출력됨.**

---

### 2. `PARTITION BY` 예제 — 각 행에서 부서별 누적 매출

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

→ **데이터는 그대로 유지하면서, 각 부서 내에서 누적 합계를 계산.**
