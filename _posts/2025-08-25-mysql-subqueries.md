---
title: "1주차 정규과제 - MySQL 공식문서: Subqueries"
date: 2025-08-25 21:00:00 +0900
categories: [SQL_ADVANCED, Week1]
tags: [MySQL, Subqueries, 공식문서, 번역]
---

# Subqueries (서브쿼리)

> **원문 출처:** [MySQL 8.0 Reference Manual - 15.2.15 Subqueries](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)  
> **번역:** DArt-B 학회

---

## 목차

- 15.2.15 Subqueries  
- 15.2.15.1 The Subquery as Scalar Operand  
- 15.2.15.2 Comparisons Using Subqueries  
- 15.2.15.3 Subqueries with ANY, IN, or SOME  
- 15.2.15.4 Subqueries with ALL  
- 15.2.15.5 Row Subqueries  
- 15.2.15.6 Subqueries with EXISTS or NOT EXISTS  
- 15.2.15.7 Correlated Subqueries  
- 15.2.15.8 Derived Tables  
- 15.2.15.9 Lateral Derived Tables  
- 15.2.15.10 Subquery Errors  
- 15.2.15.11 Optimizing Subqueries  
- 15.2.15.12 Restrictions on Subqueries  

---

## 15.2.15 Subqueries

**서브쿼리(Subquery)**란 하나의 SQL 문 안에 포함된 `SELECT` 문을 말합니다.  
MySQL은 SQL 표준에서 요구하는 모든 서브쿼리 형태와 연산을 지원하며, MySQL 고유의 기능도 일부 제공합니다.

예시:

~~~sql
SELECT * 
FROM t1 
WHERE column1 = (SELECT column1 FROM t2);
~~~

여기서  
- `SELECT * FROM t1 ...` → 외부 쿼리(outer query)  
- `(SELECT column1 FROM t2)` → 서브쿼리(subquery)  

즉, 서브쿼리는 외부 쿼리 안에 중첩되어 있고, 다른 서브쿼리 안에 또다시 서브쿼리를 넣을 수도 있습니다. (깊은 중첩 가능)  
서브쿼리는 항상 괄호 안에 포함되어야 합니다.

---

### 서브쿼리의 장점
1. 쿼리를 구조적으로 나눠 각 부분을 쉽게 구분할 수 있음.  
2. 복잡한 JOIN이나 UNION 없이도 원하는 결과를 얻는 대체 방법 제공.  
3. 많은 사람들이 복잡한 조인보다 서브쿼리가 더 읽기 쉽다고 느낌.  

서브쿼리의 등장은 SQL을 “Structured Query Language”라 부르게 된 계기 중 하나였습니다.

---

### 서브쿼리 구문 예시

~~~sql
DELETE FROM t1
WHERE s11 > ANY
 (SELECT COUNT(*) /* no hint */ FROM t2
  WHERE NOT EXISTS
   (SELECT * FROM t3
    WHERE ROW(5*t2.s1,77)=
     (SELECT 50,11*s1 FROM t4 UNION SELECT 50,77 FROM
      (SELECT * FROM t5) AS t5)));
~~~

---

### 서브쿼리 결과 형태
- Scalar: 단일 값  
- Column: 단일 컬럼(여러 행 가능)  
- Row: 단일 행(여러 컬럼 가능)  
- Table: 다수의 행과 컬럼  

각 결과 형태는 특정 맥락에서만 사용 가능.

---

### 서브쿼리에서 허용되는 요소
- DISTINCT, GROUP BY, ORDER BY, LIMIT  
- JOIN, 인덱스 힌트, UNION, 함수, 주석 등  

MySQL 8.0.19부터는 `TABLE`, `VALUES` 구문도 서브쿼리 안에서 사용 가능.

예시 (동등한 3가지 표현):

~~~sql
SELECT * FROM tt
 WHERE b > ANY (VALUES ROW(2), ROW(4), ROW(6));

SELECT * FROM tt
 WHERE b > ANY (SELECT * FROM ts);

SELECT * FROM tt
 WHERE b > ANY (TABLE ts);
~~~

---

### 서브쿼리를 사용할 수 있는 외부 구문
- SELECT  
- INSERT  
- UPDATE  
- DELETE  
- SET  
- DO  

---

### 참고
- 최적화 관련: Section 10.2.2 (Optimizing Subqueries, Derived Tables, View References, and Common Table Expressions)  
- 제약 사항: Section 15.2.15.12 (Restrictions on Subqueries)  

---

여기까지가 MySQL 공식문서 Subqueries (15.2.15) 번역 및 정리입니다.
