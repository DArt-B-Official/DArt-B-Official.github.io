---
title: "5주차 정규과제 - MySQL 공식문서: WITH(Common Table Expressions) - WITH RECURSIVE"
date: 2025-10-22 22:00:00 +0900
categories: [SQL_ADVANCED, Week5]
tags: [MySQL, WindowFunctions, 공식문서, 번역]
---

# WITH RECURSIVE 설명

## 15.2.20 WITH (Common Table Expreesions)

> **원문 출처:** [MySQL 8.0 Reference Manual - 15.2.20 WITH (Common Table Expreesions)]
(https://dev.mysql.com/doc/refman/8.0/en/with.html)  
> **번역:** DArt-B 학회

## 개요

**재귀 공통 테이블 표현식(Recursive CTE)** 은 **자신의 이름을 참조하는 서브쿼리**를 가진 CTE를 말한다.  

기본 형태:

~~~sql
WITH RECURSIVE cte (n) AS (
 SELECT 1        -- 비재귀(시드) 부분
 UNION ALL
 SELECT n + 1 FROM cte -- 재귀 부분
 WHERE n < 5
)
SELECT * FROM cte;
~~~

결과:

~~~sql
+---+
| n |
+---+
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
+---+
~~~

## **구조와 규칙**

- 어떤 CTE라도 자신을 참조한다면 WITH 절은 반드시 WITH RECURSIVE 로 시작해야 한다.

  (자기참조가 없으면 RECURSIVE는 **생략 가능**하지만 사용해도 무방.)

- RECURSIVE 를 빼먹으면 보통 다음 오류가 난다:

  ERROR 1146 (42S02): Table 'cte_name' doesn't exist

### **재귀 CTE의 두 부분**

CTE 서브쿼리는 **UNION ALL** (또는 **UNION DISTINCT**)로 구분된 두 부분으로 구성된다.

1. **비재귀(시드) SELECT**: 초기 행(들) 생성. **CTE 이름을 참조하지 않는다.**

2. **재귀 SELECT**: 이전 단계의 결과를 참조하여 추가 행 생성. **CTE 이름을 FROM 에서 1회 참조.**

   재귀 SELECT가 **더 이상 새 행을 못 만들면 종료**된다.

> 각 SELECT 부분은 **여러 SELECT의 UNION**으로 이뤄질 수도 있다.

### **타입 추론과 NULL 가능성**

- **열 타입은 비재귀 SELECT**의 열 타입만으로 추론되며, **모든 열은 NULL 허용**으로 간주된다.

  (재귀 SELECT는 타입 결정에서 무시됨)



### **UNION ALL vs UNION DISTINCT**

- UNION DISTINCT 를 사용하면 **중복 행을 제거**한다.

  전이 폐쇄(그래프 탐색) 같은 쿼리에서 **무한 루프 방지**에 유용하다.



### **반복(Iteration) 동작**

- 재귀 부분의 각 반복은 **직전 반복에서 생성된 행들만**을 입력으로 사용한다.
- 재귀 부분에 **여러 쿼리 블록**이 있으면 **실행 순서는 미정**이며, 각 블록은 자신 또는 다른 블록이 직전 반복 이후 만든 행을 사용할 수 있다.



## **컬럼 폭 확장 이슈와 CAST**

비재귀 SELECT가 결정한 **열 폭** 때문에, 재귀 SELECT가 더 긴 문자열을 만들어도 **잘릴 수** 있다.

~~~sql
WITH RECURSIVE cte AS (
  SELECT 1 AS n, 'abc' AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
~~~

- **nonstrict 모드**: 'abc'로 **잘려서** 출력
- **strict 모드**: ERROR 1406 (22001) Data too long for column 'str'

해결: **비재귀 부분에서 미리 폭을 넓혀** 준다.

~~~sql
WITH RECURSIVE cte AS (
  SELECT 1 AS n, CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
~~~



## **이름 기반 접근(열 위치가 달라도 됨)**

열은 **위치가 아니라 이름으로 참조**된다. 재귀 부분은 비재귀 부분과 **열 위치가 달라도** 이름으로 접근할 수 있다.

~~~sql
WITH RECURSIVE cte AS (
  SELECT 1 AS n, 1 AS p, -1 AS q
  UNION ALL
  SELECT n + 1, q * 2, p * 2
  FROM cte
  WHERE n < 5
)
SELECT * FROM cte;
~~~

출력에서 p와 q는 각 단계마다 **서로의 이전 값**을 토대로 바뀐다.



## **문법 제약(재귀 SELECT에 한함)**

재귀 SELECT **내부에서는** 다음을 사용할 수 **없다**:

- 집계 함수 (SUM() 등)
- 윈도 함수
- GROUP BY
- ORDER BY
- DISTINCT

> **MySQL 8.0.19 이전**: LIMIT **금지**

> **MySQL 8.0.19+**: 재귀 SELECT에서 LIMIT [OFFSET] **허용** (생성 즉시 제한되어 효율적)

기타 제약:

- 재귀 SELECT는 **CTE를 FROM에서 단 한 번만** 참조해야 하며, **서브쿼리 내부에서 참조 금지**.
- 다른 테이블과 **JOIN 가능**하나, **LEFT JOIN의 오른쪽**에 CTE를 둘 수 없다.

> 위 제약은 표준 SQL에서 기원하며, ORDER BY, LIMIT(≤8.0.18), DISTINCT 제약은 MySQL 특화.



## **EXPLAIN과 비용**

- 재귀 SELECT에 대한 EXPLAIN의 Extra 컬럼에 **Recursive** 가 표시된다.
- 표시되는 비용은 **반복 1회당 비용**이며, **전체 반복 횟수는 옵티마이저가 예측 불가**(WHERE 종료시점 예측 어려움).
- 결과가 매우 커지면 내부 임시 테이블이 **디스크 기반**으로 바뀌며 성능 저하 가능 → **in-memory 임시 테이블 한도**를 늘리면 개선될 수 있다.



## **재귀 제한과 안전장치**

종료 조건을 깜빡하면 **무한 재귀**에 빠질 수 있으므로, 다음 안전장치를 활용하자.

- cte_max_recursion_depth (세션/글로벌): **허용 재귀 레벨 수** 제한
- max_execution_time (세션): SELECT **실행 시간 제한(ms)**
- 옵티마이저 힌트 MAX_EXECUTION_TIME(ms), 또는 /*+ SET_VAR(cte_max_recursion_depth = ... ) */
- (8.0.19+) **재귀 SELECT 내부 LIMIT** 사용으로 **행 수 상한** 설정

예:

~~~sql
SET SESSION cte_max_recursion_depth = 10;       -- 얕은 재귀만 허용
SET SESSION max_execution_time = 1000;          -- 1초 제한
~~~

또는:

~~~sql
WITH RECURSIVE cte(n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte LIMIT 10000
)
SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM cte;
~~~

무한 루프에 빠진 경우, 다른 세션에서 KILL QUERY로 종료 가능.



## **예제 모음**

### **1) 단순 수열**

~~~sql
WITH RECURSIVE cte(n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5
)
SELECT * FROM cte;
~~~



### **2) 피보나치 수열**

~~~sql
WITH RECURSIVE fibonacci(n, fib_n, next_fib_n) AS (
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n
  FROM fibonacci
  WHERE n < 10
)
SELECT * FROM fibonacci;
~~~



### **3) 날짜 시리즈 생성 + 공백 채우기**

판매일자에 구멍이 있어도 **모든 날짜**를 만들고 LEFT JOIN으로 합계를 채운다.

~~~sql
WITH RECURSIVE dates(d) AS (
  SELECT MIN(date) FROM sales
  UNION ALL
  SELECT d + INTERVAL 1 DAY FROM dates
  WHERE d + INTERVAL 1 DAY <= (SELECT MAX(date) FROM sales)
)
SELECT dates.d, COALESCE(SUM(price), 0) AS sum_price
FROM dates
LEFT JOIN sales ON dates.d = sales.date
GROUP BY dates.d
ORDER BY dates.d;
~~~



### **4) 계층(조직도) 탐색**

~~~sql
WITH RECURSIVE employee_paths (id, name, path) AS (
  SELECT id, name, CAST(id AS CHAR(200))
  FROM employees
  WHERE manager_id IS NULL      -- 최상위(CEO)

  UNION ALL

  SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
  FROM employee_paths AS ep
  JOIN employees AS e ON ep.id = e.manager_id
)
SELECT * FROM employee_paths
ORDER BY path;
~~~



## **CTE vs 파생 테이블(서브쿼리)**

공통점

- 둘 다 **이름이 있고**, **문장 범위 내**에서만 존재.

차이점(CTE의 장점)

- 파생 테이블은 **한 번만 참조** 가능하지만, **CTE는 여러 번 참조** 가능.
- **자기 자신을 참조(재귀)** 가능.
- **CTE 간 상호 참조** 가능.
- 쿼리 **선두부 정의**로 가독성 향상.
- CREATE [TEMPORARY] TABLE 과 달리 **명시적 생성/삭제 권한 불필요**.