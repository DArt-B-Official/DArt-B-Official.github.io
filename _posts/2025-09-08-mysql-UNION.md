---
title: "2주차 정규과제 - MySQL 공식문서: Union, INTERSECT"
date: 2025-09-08 21:00:00 +0900
categories: [SQL_ADVANCED, Week2]
tags: [MySQL, Subqueries, 공식문서, 번역]
---

# 집합 연산자

## 15.2.18. Union Clause

\> **원문 출처:** [MySQL 8.0 Reference Manual - 15.2.14.SEt Operations](https://dev.mysql.com/doc/refman/8.0/en/set-operations.html)  

\> **번역:** DArt-B 학회

~~~sql
query_expression_body UNION [ALL | DISTINCT] query_block
    [UNION [ALL | DISTINCT] query_expression_body]
    [...]
~~~



### 설명

UNION은 여러 쿼리 블록의 결과를 하나의 결과 집합으로 결합합니다.

예시는 SELECT 문을 사용합니다:

~~~sql
mysql> SELECT 1, 2;
+---+---+
| 1 | 2 |
+---+---+
| 1 | 2 |
+---+---+

mysql> SELECT 'a', 'b';
+---+---+
| a | b |
+---+---+
| a | b |
+---+---+

mysql> SELECT 1, 2 UNION SELECT 'a', 'b';
+---+---+
| 1 | 2 |
+---+---+
| 1 | 2 |
| a | b |
+---+---+
~~~



## **MySQL 8.0에서의 UNION 처리 (MySQL 5.7과 비교)**





MySQL 8.0에서는 SELECT와 UNION에 대한 파서 규칙이 더 일관성 있게 리팩터링되었으며, 동일한 SELECT 구문이 모든 문맥에서 동일하게 적용되도록 변경되었습니다. 이로 인해 몇 가지 사용자 가시적인 차이가 발생했으며, 일부 문장은 다시 작성해야 할 수도 있습니다:



- NATURAL JOIN은 표준 SQL에 맞게 선택적으로 INNER 키워드를 허용합니다.

  예: NATURAL INNER JOIN

- 괄호 없이도 **right-deep join**이 허용됩니다.

  예: ... JOIN ... JOIN ... ON ... ON

- STRAIGHT_JOIN도 이제 다른 INNER JOIN과 유사하게 USING 절을 허용합니다.

- 파서는 쿼리 표현식에 괄호를 허용합니다.

   예: (SELECT ... UNION SELECT ...)

  참고: [15.2.11, “괄호 쿼리 표현식”]

- 파서는 SQL_CACHE 및 SQL_NO_CACHE 쿼리 수정자의 허용 위치를 문서화된 규칙에 더 잘 맞추었습니다.

- **UNION의 좌측 중첩(left-hand nesting)** 은 이전에는 서브쿼리에서만 허용되었으나, 이제 최상위 문에서도 허용됩니다.

   예:

~~~sql
(SELECT 1 UNION SELECT 1) UNION SELECT 1;
~~~



- **잠금 절(locking clause)** (FOR UPDATE, LOCK IN SHARE MODE)는 UNION이 아닌 쿼리에서만 허용됩니다.

  따라서, 잠금 절을 포함하는 SELECT 문은 반드시 괄호로 묶어야 합니다.

  더 이상 허용되지 않는 예:

  ~~~sql
  SELECT 1 FOR UPDATE UNION SELECT 1 FOR UPDATE;
  ~~~

  올바른 예:

  ~~~sql
  (SELECT 1 FOR UPDATE) UNION (SELECT 1 FOR UPDATE);
  ~~~

  
  ## 15.2.14 Set Operations with UNION, INTERSECT, and EXCEPT

## **SQL 집합 연산 개요**

SQL 집합 연산은 여러 쿼리 블록의 결과를 하나의 결과로 결합합니다.

- **쿼리 블록(query block)**: SELECT 같은 결과 집합을 반환하는 SQL 문 (단순 테이블이라고도 함)
- **MySQL 8.0.19 이상**: TABLE과 VALUES 문도 지원



SQL 표준에서 정의된 집합 연산은 다음과 같습니다:

- **UNION**: 두 쿼리 블록의 모든 결과를 결합하고, 중복 제거
- **INTERSECT**: 두 쿼리 블록 모두에 공통으로 존재하는 행만 반환, 중복 제거
- **EXCEPT**: 두 쿼리 블록 A, B에서 A에만 존재하고 B에는 없는 행 반환, 중복 제거
  - 일부 DBMS(Oracle 등)는 MINUS라는 이름 사용 (MySQL에서는 지원하지 않음)
- MySQL은 오래전부터 **UNION** 지원
- **INTERSECT, EXCEPT**는 **MySQL 8.0.31 이상**에서 지원



모든 집합 연산자는 ALL과 DISTINCT를 지원합니다:

- ALL: 중복을 제거하지 않음
- DISTINCT: 중복을 제거 (기본 동작, 생략 가능)



------

## **문법 개요**

~~~sql
query_block [set_op query_block] [set_op query_block] ...

query_block:
    SELECT | TABLE | VALUES

set_op:
    UNION | INTERSECT | EXCEPT
~~~

보다 정확히 표현하면:

~~~sql
query_expression:
  [with_clause]
  query_expression_body
  [order_by_clause] [limit_clause] [into_clause]

query_expression_body:
    query_term
 |  query_expression_body UNION [ALL | DISTINCT] query_term
 |  query_expression_body EXCEPT [ALL | DISTINCT] query_term

query_term:
    query_primary
 |  query_term INTERSECT [ALL | DISTINCT] query_primary

query_primary:
    query_block
 |  '(' query_expression_body [order_by_clause] [limit_clause] [into_clause] ')'

query_block:
    query_specification   -- SELECT
 |  table_value_constructor -- VALUES
 |  explicit_table          -- TABLE
~~~

### **연산자 우선순위**

- INTERSECT는 UNION, EXCEPT보다 먼저 평가됨

  - 예: TABLE x UNION TABLE y INTERSECT TABLE z

    → TABLE x UNION (TABLE y INTERSECT TABLE z)

- UNION, INTERSECT는 **교환법칙 성립**

- EXCEPT는 **교환법칙 불가**



## **결과 컬럼 이름과 데이터 타입**

- 결과 컬럼 이름은 **첫 번째 쿼리 블록**에서 가져옴
- 같은 위치에 있는 컬럼은 **데이터 타입이 일치**해야 함
- 타입이 다를 경우, 모든 블록의 값을 고려해 자동 변환



예:

~~~sql
SELECT REPEAT('a',1) UNION SELECT REPEAT('b',20);
~~~

결과:

~~~sql
a
bbbbbbbbbbbbbbbbbbbb
~~~

---

## **TABLE / VALUES 문과 집합 연산**

- MySQL 8.0.19 이상: TABLE, VALUES를 SELECT처럼 사용 가능

예:

~~~sql
TABLE t1 UNION TABLE t2;
VALUES ROW(4,-2), ROW(5,9) UNION TABLE t2;
~~~

컬럼 이름을 맞추려면 SELECT와 별칭 사용:

~~~sql
SELECT * FROM (TABLE t2) AS t(x,y) UNION TABLE t1;
~~~



---

## **DISTINCT와 ALL**

- **기본값**: DISTINCT (중복 제거)

- ALL: 중복 제거하지 않음

- 혼합 사용 시 DISTINCT가 우선 적용

  

------

## **ORDER BY와 LIMIT**

- 개별 쿼리 블록에 적용하려면 괄호로 묶기:

~~~sql
(SELECT a FROM t1 WHERE a=10 ORDER BY a LIMIT 10)
UNION
(SELECT a FROM t2 WHERE a=11 ORDER BY a LIMIT 10);
~~~

- 전체 결과에 적용하려면 마지막에 배치:

~~~sql
SELECT a FROM t1
EXCEPT
SELECT a FROM t2
ORDER BY a LIMIT 10;
~~~



- ORDER BY만 단독으로 쓰이면 최적화로 제거됨
- VALUES도 ORDER BY, LIMIT 가능하지만 WHERE는 불가

주의:

- 별칭(alias)을 줬다면 ORDER BY에서는 반드시 별칭 사용
- 정렬 시 블록 구분용 컬럼 추가 가능 (sort_col)

------

## **집합 연산의 제약 사항**

1. **제약**

   - HIGH_PRIORITY: 첫 번째 SELECT에서는 무효, 이후 SELECT에서는 오류
   - INTO: 마지막 SELECT에서만 사용 가능
   - UNION ... INTO 구문은 8.0.20부터 폐지 예정

2. **집합 연산과 집계 함수**

   - ORDER BY 절에서 집계 함수 사용 불가

   ~~~sql
   TABLE t1 INTERSECT TABLE t2 ORDER BY MAX(x);
   -- 오류 발생
   ~~~

3. **잠금 절 (FOR UPDATE, LOCK IN SHARE MODE)**

- UNION 등과 함께 쓰려면 반드시 괄호로 묶어야 함


