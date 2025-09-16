---
title: "3주차 정규과제(1) - MySQL 공식문서: Window Function Concepts and Syntax"
date: 2025-09-08 22:00:00 +0900
categories: [SQL_ADVANCED, Week2]
tags: [MySQL, WindowFunctions, 공식문서, 번역]
---

# 윈도 함수

## 14.20.2 Window Function Concepts and Syntax

> **원문 출처:** [MySQL 8.0 Reference Manual - 14.20.2 Window Function Concepts and Syntax](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html)  
> **번역:** DArt-B 학회

이 절에서는 윈도 함수를 사용하는 방법을 설명합니다. 예제에서는 **14.19.2 GROUPING()** 함수 설명에서 사용된 것과 동일한 *sales* 데이터셋을 사용합니다.

~~~sql
mysql> SELECT * FROM sales ORDER BY country, year, product;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2001 | Finland | Phone      |     10 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Computer   |   1200 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   1500 |
| 2001 | USA     | Computer   |   1200 |
| 2001 | USA     | TV         |    150 |
| 2001 | USA     | TV         |    100 |
+------+---------+------------+--------+
~~~

---

## 윈도 함수 기본 개념

- **집계 함수**: 여러 행을 하나의 결과 행으로 축소 (예: `SUM(profit)` → 전체 합계)
- **윈도 함수**: 각 행마다 결과를 반환 (집계와 유사하지만 “행 단위”)

- **현재 행(Current row)**: 함수 계산이 수행되는 대상 행  
- **윈도(Window)**: 현재 행을 기준으로 관련된 행들의 집합  

예시:

~~~sql
SELECT SUM(profit) AS total_profit FROM sales;
-- 전체 합계 = 7535

SELECT country, SUM(profit) AS country_profit
FROM sales
GROUP BY country
ORDER BY country;
-- 국가별 합계: Finland=1610, India=1350, USA=4575
~~~

윈도 함수 사용 시:

~~~sql
SELECT
  year, country, product, profit,
  SUM(profit) OVER() AS total_profit,
  SUM(profit) OVER(PARTITION BY country) AS country_profit
FROM sales
ORDER BY country, year, product, profit;
~~~

- `OVER()` → 전체 행을 하나의 파티션으로 취급, 각 행에 전체 합계 표시  
- `OVER(PARTITION BY country)` → 국가별 파티션 합계 표시  

---

## 사용 위치와 실행 순서

- 윈도 함수는 **SELECT 목록**과 **ORDER BY 절**에서만 사용 가능  
- 실행 순서:  
  `FROM → WHERE → GROUP BY → HAVING → (윈도 함수) → ORDER BY → LIMIT`

---

## 윈도 함수로 사용 가능한 집계 함수

`OVER` 절이 있으면 윈도 함수, 없으면 일반 집계 함수로 동작합니다:

- `AVG()`  
- `BIT_AND()`, `BIT_OR()`, `BIT_XOR()`  
- `COUNT()`  
- `JSON_ARRAYAGG()`, `JSON_OBJECTAGG()`  
- `MAX()`, `MIN()`  
- `STDDEV_POP()`, `STDDEV()`, `STD()`  
- `STDDEV_SAMP()`  
- `SUM()`  
- `VAR_POP()`, `VARIANCE()`  
- `VAR_SAMP()`

---

## 비집계 전용 윈도 함수

이 함수들은 윈도 함수로만 사용 가능하며 `OVER` 절이 필수:

- `CUME_DIST()`  
- `DENSE_RANK()`  
- `FIRST_VALUE()`  
- `LAG()`, `LEAD()`  
- `LAST_VALUE()`  
- `NTH_VALUE()`  
- `NTILE()`  
- `PERCENT_RANK()`  
- `RANK()`  
- `ROW_NUMBER()`

예: `ROW_NUMBER()`  

~~~sql
SELECT
  year, country, product, profit,
  ROW_NUMBER() OVER(PARTITION BY country) AS row_num1,
  ROW_NUMBER() OVER(PARTITION BY country ORDER BY year, product) AS row_num2
FROM sales;
~~~

- `row_num1`: 정렬 없는 파티션 → 비결정적 번호 부여  
- `row_num2`: 정렬 기준 추가 → 일관된 순번 부여  

---

## OVER 절 문법

```sql
over_clause:
    {OVER (window_spec) | OVER window_name}
```

- OVER(window_spec): 괄호 안에 직접 윈도 정의
- OVER window_name: WINDOW 절에서 정의한 이름 참조 (→ 14.20.4 Named Windows 참조)



### **window_spec 구성 요소**

~~~sql
window_spec:
    [window_name] [partition_clause] [order_clause] [frame_clause]
~~~

- window_name: WINDOW 절에서 정의된 윈도 이름
- partition_clause: PARTITION BY로 행을 그룹화 (표준 SQL은 컬럼만 허용, MySQL은 표현식도 허용)
- order_clause: 각 파티션 내 행의 순서 지정
- frame_clause: 파티션 내 “프레임(부분집합)” 정의 (→ 14.20.3 Frame Specification)