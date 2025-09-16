---
title: "3주차 정규과제(2) - MySQL 공식문서: Window Function Descriptions"
date: 2025-09-08 22:00:00 +0900
categories: [SQL_ADVANCED, Week2]
tags: [MySQL, WindowFunctions, 공식문서, 번역]
---

# 윈도 함수 설명

## 14.20.1 Window Function Descriptions

> **원문 출처:** [MySQL 8.0 Reference Manual - 14.20.1 Window Function Descriptions](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html)  
> **번역:** DArt-B 학회

이 절은 **비집계 윈도 함수**를 설명합니다. 각 행에 대해, 해당 행과 관련된 행 집합(파티션/프레임)을 이용해 값을 계산합니다.  
대부분의 **집계 함수**도 `OVER` 절이 있으면 윈도 함수로 사용할 수 있습니다(→ **14.19.1 Aggregate Function Descriptions**).  
`OVER` 절, 윈도/파티션/프레임/피어(peer) 개념과 사용 예시는 → **14.20.2 Window Function Concepts and Syntax** 참고.

---

## 윈도 함수 목록 (Table 14.30)

| 이름             | 설명                                     |
| ---------------- | ---------------------------------------- |
| `CUME_DIST()`    | 누적분포값(현재 값 이하 비율)            |
| `DENSE_RANK()`   | 파티션 내 현재 행의 순위(동점 간격 없음) |
| `FIRST_VALUE()`  | 프레임 첫 행의 값                        |
| `LAG()`          | 파티션에서 현재 행보다 N행 이전의 값     |
| `LAST_VALUE()`   | 프레임 마지막 행의 값                    |
| `LEAD()`         | 파티션에서 현재 행보다 N행 이후의 값     |
| `NTH_VALUE()`    | 프레임 N번째 행의 값                     |
| `NTILE()`        | 파티션을 N개 버킷으로 나눈 버킷 번호     |
| `PERCENT_RANK()` | 백분위 순위(0~1)                         |
| `RANK()`         | 파티션 내 현재 행의 순위(동점 간격 발생) |
| `ROW_NUMBER()`   | 파티션 내 현재 행의 일련번호             |

### `over_clause`와 `null_treatment`

- `over_clause`는 `OVER (...)` 구문을 의미(→ **14.20.2**).  
- 일부 함수는 `null_treatment`(표준 SQL) 옵션이 있으나, MySQL은 **`RESPECT NULLS`만 허용**(기본).  
  `IGNORE NULLS`는 파싱되지만 **오류**가 발생합니다.

---

## 함수별 설명

### `CUME_DIST() over_clause`

- 파티션 내 **현재 행 값 이하**의 비율(누적분포) 반환. 범위: **0 ~ 1**.  
- **ORDER BY 권장**: 없으면 모든 행이 피어로 간주되어 값은 `N/N = 1`(N=파티션 크기).

예(요약):

```sql
SELECT
  val,
  ROW_NUMBER()   OVER w AS row_number,
  CUME_DIST()    OVER w AS cume_dist,
  PERCENT_RANK() OVER w AS percent_rank
FROM numbers
WINDOW w AS (ORDER BY val);
```

### **DENSE_RANK() over_clause**

- 파티션 내 순위를 반환하되 **동점 간격이 생기지 않도록** 연속 순위 부여.
- **ORDER BY 권장**(없으면 모두 피어).

> RANK()와의 비교 예시는 아래 RANK() 설명 참조.



### **FIRST_VALUE(expr) [null_treatment] over_clause**

- **프레임 첫 행**의 expr 값을 반환.
- 프레임 정의에 따라 결과가 달라짐(예: ROWS UNBOUNDED PRECEDING 등).

예(요약 — LAST_VALUE, NTH_VALUE와 함께):

~~~sql
SELECT
  time, subject, val,
  FIRST_VALUE(val)  OVER w AS first,
  LAST_VALUE(val)   OVER w AS last,
  NTH_VALUE(val, 2) OVER w AS second,
  NTH_VALUE(val, 4) OVER w AS fourth
FROM observations
WINDOW w AS (
  PARTITION BY subject ORDER BY time
  ROWS UNBOUNDED PRECEDING
);
~~~

> 프레임에 N번째 행이 포함되지 않으면 NTH_VALUE()는 NULL 반환.



### **LAG(expr [, N[, default]]) [null_treatment] over_clause**

- 파티션에서 **현재 행 기준 N행 이전**의 expr 값을 반환.

- 해당 행이 없으면 default 반환. 기본값: N=1, default=NULL.

- **제약(8.0.22+)**:

  - N은 0 ~ 2^63 범위의 **비음수 정수 리터럴/파라미터/변수**여야 함.
  - N은 NULL일 수 없음. N=0이면 현재 행의 expr.

  

차분/차이 계산 예:

~~~sql
SELECT
  t, val,
  LAG(val)        OVER w AS lag,
  LEAD(val)       OVER w AS lead,
  val - LAG(val)  OVER w AS lag_diff,
  val - LEAD(val) OVER w AS lead_diff
FROM series
WINDOW w AS (ORDER BY t);
~~~



피보나치 누적 아이디어:

~~~sql
SELECT
  n,
  LAG(n, 1, 0)      OVER w AS lag,
  LEAD(n, 1, 0)     OVER w AS lead,
  n + LAG(n, 1, 0)  OVER w AS next_n,
  n + LEAD(n, 1, 0) OVER w AS next_next_n
FROM fib
WINDOW w AS (ORDER BY n);
~~~

---

### **LAST_VALUE(expr) [null_treatment] over_clause**

- **프레임 마지막 행**의 expr 값을 반환.
- 프레임/정렬 정의에 따라 값이 달라집니다(→ FIRST_VALUE() 예시 참고).

------

### **LEAD(expr [, N[, default]]) [null_treatment] over_clause**

- 파티션에서 **현재 행 기준 N행 이후**의 expr 값을 반환.
- 해당 행이 없으면 default 반환. 기본값: N=1, default=NULL.
- **제약(8.0.22+)**: LAG()와 동일(비음수 정수, NULL 불가, 0 ~ 2^63).

> 예시는 LAG() 설명 참고.

------

### **NTH_VALUE(expr, N) [from_first_last] [null_treatment] over_clause**

- 프레임 **N번째 행**의 expr 값을 반환. 없으면 NULL.

- N은 **양의 정수 리터럴**이어야 함.

- from_first_last(표준): MySQL은 **FROM FIRST만 허용**(기본).

  FROM LAST는 파싱되나 **오류**. 같은 효과를 내려면 **내림차순 ORDER BY**로 프레임을 뒤집어 구현.

- **제약(8.0.22+)**: N에 NULL 사용 불가.

------

### **NTILE(N) over_clause**

- 파티션을 **N개 버킷**으로 분할 후, 현재 행의 **버킷 번호(1~N)** 반환.
- N은 **양의 정수 리터럴**.
- **제약(8.0.22+)**: N은 NULL 불가, 0 ~ 2^63 범위의 정수(리터럴/파라미터/변수).
- **ORDER BY 권장**: 없으면 피어 처리로 분할이 비결정적.

예(요약):

~~~sql
SELECT
  val,
  ROW_NUMBER() OVER w AS row_number,
  NTILE(2)     OVER w AS ntile2,
  NTILE(4)     OVER w AS ntile4
FROM numbers
WINDOW w AS (ORDER BY val);
~~~

> 8.0.22+: NTILE(NULL) 금지.



---

### **PERCENT_RANK() over_clause**

- 파티션 내 **최대값을 제외한** 값 기준 **상대 백분위 순위** 반환.
- 공식: (rank - 1) / (rows - 1) (0~1).
- **ORDER BY 권장**.
- 예시는 CUME_DIST() 절 비교 예 참고.

------

### **RANK() over_clause**

- 파티션 내 순위를 반환하되, **동점 시 같은 순위**를 부여하며 **간격(gaps)** 발생.
- **ORDER BY 권장**.



RANK() vs DENSE_RANK() 비교 예(요약):

~~~sql
SELECT
  val,
  ROW_NUMBER() OVER w AS row_number,
  RANK()       OVER w AS rank,
  DENSE_RANK() OVER w AS dense_rank
FROM numbers
WINDOW w AS (ORDER BY val);
~~~

- RANK(): 동점 다음 값의 순위가 **동점 개수만큼 건너뜀**.
- DENSE_RANK(): 다음 값의 순위는 **+1**(간격 없음).



### **ROW_NUMBER() over_clause**

- 파티션 내 현재 행의 **일련번호(1 ~ 파티션 행수)** 반환.
- **ORDER BY**가 번호 부여 순서를 결정. 없으면 **비결정적**.
- 동점에 **서로 다른 번호**를 부여. 동점에 동일 값을 원하면 RANK()/DENSE_RANK() 사용.