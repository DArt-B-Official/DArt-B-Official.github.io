---
title: "1주차 정규과제 - MySQL 공식문서: Subqueries"
date: 2025-08-25 21:00:00 +0900
categories: [SQL_ADVANCED, Week1]
tags: [MySQL, Subqueries, 공식문서, 번역]
---

# Subqueries (서브쿼리)



## 15.2.15 Subqueries

\> **원문 출처:** [MySQL 8.0 Reference Manual - 15.2.15 Subqueries](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)  

\> **번역:** DArt-B 학회



**서브쿼리(Subquery)**는 하나의 SQL 문 안에 포함된 `SELECT` 문을 말합니다.  

SQL 표준에서 요구하는 모든 서브쿼리 형태와 연산을 MySQL은 지원하며, MySQL 고유의 기능도 일부 제공합니다.

**예시**

```sql
SELECT * 
FROM t1 
WHERE column1 = (
  SELECT column1 FROM t2
);
```

위 예시에서

- SELECT * FROM t1 ... → 외부 쿼리(outer query)
- (SELECT column1 FROM t2) → 서브쿼리(subquery)



즉, 서브쿼리는 외부 쿼리 안에 중첩되며, 또 다른 서브쿼리 안에 서브쿼리를 넣을 수도 있습니다(깊은 중첩 가능).

서브쿼리는 반드시 괄호로 감싸져야 합니다.



### 서브쿼리의 주요 장점

1. 쿼리를 구조적으로 분리하여 각 부분을 쉽게 파악할 수 있음.
2. 복잡한 JOIN이나 UNION을 대체할 수 있는 방법 제공.
3. 많은 사람들이 복잡한 조인보다 서브쿼리를 더 읽기 쉽다고 느낌.

>  사실 서브쿼리의 등장은 초기 SQL을 “Structured Query Language”라 부르게 된 계기 중 하나였습니다.



### SQL 표준에서 정의된 서브쿼리 구문 예시

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



#### 서브쿼리 결과 형태

- **Scalar**: 단일 값 반환
- **Column**: 단일 컬럼(여러 행 가능)
- **Row**: 단일 행(여러 컬럼 가능)
- **Table**: 다수의 행과 컬럼

각 결과 형태는 특정한 문맥에서만 사용 가능합니다.



## **서브쿼리에서 허용되는 요소**

서브쿼리는 일반 SELECT 문에서 허용되는 대부분의 요소를 포함할 수 있습니다:

- DISTINCT, GROUP BY, ORDER BY, LIMIT

- JOIN, 인덱스 힌트, UNION, 함수, 주석 등

  

MySQL 8.0.19부터는 TABLE 및 VALUES 구문도 서브쿼리에서 사용 가능합니다.

예를 들어, 다음 세 쿼리는 모두 동일한 결과를 반환합니다:

~~~sql
SELECT * FROM tt
    WHERE b > ANY (VALUES ROW(2), ROW(4), ROW(6));

SELECT * FROM tt
    WHERE b > ANY (SELECT * FROM ts);

SELECT * FROM tt
    WHERE b > ANY (TABLE ts);
~~~



### 서브쿼리를 사용할 수 있는 외부 구문

- SELECT
- INSERT
- UPDATE
- DELETE
- SET
- DO

---

## 15.2.15.1 The Subquery as Scalar Operand

가장 단순한 형태의 서브쿼리는 **스칼라 서브쿼리(scalar subquery)**로, 단일 값을 반환합니다.  
스칼라 서브쿼리는 단순한 피연산자로, 단일 컬럼 값이나 리터럴이 허용되는 거의 모든 곳에서 사용할 수 있습니다.  
그리고 다른 피연산자와 마찬가지로 다음과 같은 특성을 가집니다:  

- 데이터 타입  
- 길이  
- `NULL` 가능 여부 등  



**예시1**

```sql
CREATE TABLE t1 (s1 INT, s2 CHAR(5) NOT NULL);
INSERT INTO t1 VALUES(100, 'abcde');
SELECT (SELECT s2 FROM t1);
```

위 SELECT 문에서 서브쿼리는 단일 값 'abcde'를 반환합니다. 이 값은 CHAR 타입, 길이 5, 테이블 생성 시점의 기본 문자셋과 콜레이션을 따릅니다. 또한 결과 값은 NULL이 될 수 있습니다.

단, 스칼라 서브쿼리 결과의 **NULL 가능성**은 원래 컬럼 정의(NOT NULL)와 무관합니다. 예를 들어 t1이 비어있으면, 결과는 NULL입니다. (s2가 NOT NULL이어도 결과는 NULL)



### **스칼라 서브쿼리 사용 제한**

스칼라 서브쿼리를 사용할 수 없는 경우도 있습니다.

- 어떤 문법은 **리터럴 값만 허용**합니다.

  - 예: LIMIT → 정수 리터럴만 허용
  - 예: LOAD DATA → 문자열 리터럴 파일 이름만 허용

  따라서 이런 경우에는 서브쿼리를 사용할 수 없습니다.



**예시2**

~~~sql
CREATE TABLE t1 (s1 INT);
INSERT INTO t1 VALUES (1);

CREATE TABLE t2 (s1 INT);
INSERT INTO t2 VALUES (2);

SELECT (SELECT s1 FROM t2) FROM t1;
~~~

결과: 2

→ 이는 t2에 s1 = 2인 행이 존재하기 때문입니다.

MySQL 8.0.19 이후에는 TABLE 구문으로 같은 쿼리를 작성할 수 있습니다:

~~~sql
SELECT (TABLE t2) FROM t1;
~~~



### **표현식 안에서 사용**

스칼라 서브쿼리는 **표현식(expression)**의 일부로도 사용할 수 있습니다.

이 경우에도 반드시 괄호를 써야 합니다.

~~~sql
SELECT UPPER((SELECT s1 FROM t1)) FROM t2;

# MySQL 8.0.19 이후 버전
SELECT UPPER((TABLE t1)) FROM t2;
~~~



---

## 15.2.15.2 Comparisons Using Subqueries

서브쿼리의 가장 일반적인 사용 방식은 다음과 같습니다:

> non_subquery_operand comparison_operator (subquery)

여기서 `comparison_operator`는 다음 연산자 중 하나일 수 있습니다:  
`=`, `>`, `<`, `>=`, `<=`, `<>`, `!=`, `<=>`

예시:

```sql
... WHERE 'a' = (SELECT column1 FROM t1)
```

MySQL은 다음과 같은 구문도 허용합니다:

> non_subquery_operand LIKE (subquery)

과거에는 서브쿼리를 비교 연산자의 **오른쪽**에서만 사용할 수 있었고, 지금도 일부 오래된 DBMS는 이 규칙을 강제하기도 합니다.



**예시1 - JOIN으로는 불가능한 비교**

아래 쿼리는 t2 테이블의 column2의 **최대값**과 같은 값을 가지는 t1의 모든 행을 찾습니다:

~~~sql
SELECT * FROM t1
  WHERE column1 = (SELECT MAX(column2) FROM t2);
~~~



**예시2 - 특정 값이 2번 등장하는 경우 찾기**

이 쿼리는 t1 테이블에서 특정 값이 **2번 등장하는 경우**를 찾습니다.

이는 집계를 포함하기 때문에 단순 JOIN으로는 표현할 수 없습니다:

~~~sql
SELECT * FROM t1 AS t
  WHERE 2 = (SELECT COUNT(*) FROM t1 WHERE t1.id = t.id);
~~~

### **주의사항**

- 서브쿼리를 **스칼라 값**과 비교하려면, 해당 서브쿼리는 반드시 **단일 값**을 반환해야 합니다.

- 서브쿼리를 **Row Constructor**와 비교하려면, 해당 서브쿼리는 Row Subquery여야 하며, 반환되는 값의 개수가 Row Constructor와 동일해야 합니다.



---

## 15.2.15.3 Subqueries with ANY, IN, or SOME

### 구문 (Syntax)

```sql
operand comparison_operator ANY (subquery)
operand IN (subquery)
operand comparison_operator SOME (subquery)
```



여기서 comparison_operator는 다음 연산자 중 하나일 수 있습니다:

`=, >, <, >=, <=, <>, !=`



### **ANY**

ANY 키워드는 비교 연산자 뒤에 와야 하며,

**서브쿼리에서 반환된 값들 중 하나라도 조건을 만족하면 TRUE**를 반환합니다.

**예시**

~~~sql
SELECT s1 FROM t1 WHERE s1 > ANY (SELECT s1 FROM t2);
~~~

만약 t1에 (10)이라는 행이 있고, t2에 (21,14,7)이 있으면:

- 7 < 10이므로 결과는 TRUE
- 만약 t2가 (20,10)이면 결과는 FALSE
- 만약 t2가 비어 있으면 결과는 FALSE
- 만약 t2가 (NULL, NULL, NULL)이면 결과는 NULL



### **IN**

IN은 = ANY의 **별칭(alias)** 입니다.

따라서 다음 두 쿼리는 동일합니다:

~~~sql
SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
~~~

⚠️ 주의: IN과 = ANY는 항상 동의어는 아닙니다.

- IN은 **표현식 리스트**도 받을 수 있음.

- = ANY는 **서브쿼리**만 사용할 수 있음.

  (자세한 내용은 Section 14.4.2, “Comparison Functions and Operators” 참고)

또한, NOT IN은 <> ANY가 아니라 <> ALL의 별칭임. (→ 15.2.15.4에서 다룸)



### **SOME**

SOME은 ANY의 또 다른 **별칭**입니다.

즉, 아래 두 쿼리는 동일합니다:

~~~sql
SELECT s1 FROM t1 WHERE s1 <> ANY  (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 <> SOME (SELECT s1 FROM t2);
~~~

⚠️ SOME은 잘 사용되지 않지만, 의미를 더 명확하게 전달할 때 유용합니다.

예:

- 일반적으로 “a is not equal to any b” → “모든 b가 a와 같지 않다”라고 이해하기 쉽지만,
- SQL에서 <> ANY는 “a와 같지 않은 b가 하나라도 존재한다”라는 뜻임.
- 따라서 <> SOME을 쓰면 혼동을 줄일 수 있음.



### **MySQL 8.0.19 이후 TABLE 사용**

MySQL 8.0.19부터는 **TABLE 문법**을 사용하여 IN, ANY, SOME을 쓸 수 있습니다. 단, **테이블이 하나의 컬럼만 포함해야 함**.

**예시**

~~~sql
SELECT s1 FROM t1 WHERE s1 > ANY (TABLE t2);
SELECT s1 FROM t1 WHERE s1 = ANY (TABLE t2);
SELECT s1 FROM t1 WHERE s1 IN (TABLE t2);
SELECT s1 FROM t1 WHERE s1 <> ANY  (TABLE t2);
SELECT s1 FROM t1 WHERE s1 <> SOME (TABLE t2);
~~~



---

## 15.2.15.4 Subqueries with ALL

### 구문 (Syntax)

```sql
operand comparison_operator ALL (subquery)
```

### **ALL의 의미**

ALL 키워드는 비교 연산자 뒤에 와야 하며,

**서브쿼리에서 반환된 모든 값에 대해 비교 결과가 TRUE일 때 TRUE**를 반환합니다.

예시:

~~~sql
SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
~~~

만약 t1에 (10)이라는 행이 있고, t2에 (-5, 0, +5)가 있으면:

- 10 > -5, 10 > 0, 10 > +5 → 전부 만족 → 결과는 TRUE

t2가 (12, 6, NULL, -100)이면:

- 12 > 10 조건이 FALSE → 전체 결과는 FALSE

t2가 (0, NULL, 1)이면:

- NULL 포함 → 결과는 NULL



### **빈 테이블과 NULL 값 처리**

t2가 **비어 있는 경우**:

~~~sql
SELECT * FROM t1 WHERE 1 > ALL (SELECT s1 FROM t2);
~~~

→ 결과는 TRUE

그러나 다음은 NULL이 됨:

~~~sql
SELECT * FROM t1 WHERE 1 > (SELECT s1 FROM t2);
# 또한
SELECT * FROM t1 WHERE 1 > ALL (SELECT MAX(s1) FROM t2);
~~~

→ t2가 비어 있으면 NULL

즉, **NULL 값과 빈 테이블**은 서브쿼리 작성 시 반드시 고려해야 하는 **엣지 케이스**임.



### **NOT IN과의 관계**

NOT IN은 <> ALL의 별칭(alias)입니다.

즉, 아래 두 쿼리는 동일합니다:

~~~sql
SELECT s1 FROM t1 WHERE s1 <> ALL (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (SELECT s1 FROM t2);
~~~



### **MySQL 8.0.19 이후: TABLE 구문 지원**

MySQL 8.0.19부터는 ALL 및 NOT IN 구문에서도 TABLE을 사용할 수 있습니다.

단, 조건은 다음과 같습니다:

1. 서브쿼리에서 참조하는 테이블은 **단일 컬럼만 포함**해야 함
2. 서브쿼리가 **컬럼 표현식(column expression)에 의존하지 않아야 함**

예시 (t2가 단일 컬럼 테이블일 경우):

~~~sql
SELECT s1 FROM t1 WHERE s1 <> ALL (TABLE t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (TABLE t2);
~~~

하지만 다음과 같은 쿼리는 불가능합니다:

~~~sql
SELECT * FROM t1 WHERE 1 > ALL (SELECT MAX(s1) FROM t2);
~~~

이유: MAX(s1)과 같은 **컬럼 표현식에 의존**하기 때문입니다.

👉 요약:

- ALL은 **모든 값과 비교** → 전부 만족해야 TRUE
- NOT IN은 <> ALL과 동일
- NULL 및 빈 테이블은 반드시 고려해야 함
- MySQL 8.0.19부터 TABLE 구문 지원 (단, 조건 충족 필요)



---

## 15.2.15.5 Row Subqueries

### 개요

스칼라(Scalar) 서브쿼리나 컬럼(Column) 서브쿼리는 각각 **단일 값** 또는 **단일 컬럼**을 반환합니다.  반면, **Row 서브쿼리(Row Subquery)**는 **하나의 행(Row)**을 반환하며, 이 행은 여러 개의 컬럼 값을 포함할 수 있습니다. Row 서브쿼리 비교에 사용할 수 있는 연산자는 다음과 같습니다:

`=  >  <  >=  <=  <>  !=  <=>`

**예제 1**

~~~sql
SELECT * FROM t1
  WHERE (col1, col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);

SELECT * FROM t1
  WHERE ROW(col1, col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
~~~

- t2에 id = 10인 **단일 행**이 존재할 경우:

  - 그 행의 (col3, col4) 값이 t1의 (col1, col2)와 동일하면 TRUE → 해당 행 반환
  - 다르면 FALSE → 결과 없음

- 서브쿼리가 **행을 반환하지 않으면** → 결과는 NULL

- 서브쿼리가 **두 개 이상의 행을 반환하면** → 오류 발생

  (Row 서브쿼리는 최대 1개의 행만 반환 가능)



### **Row Constructor**

다음 두 표현은 동일합니다:

~~~sql
(1, 2)
ROW(1, 2)
~~~

Row Constructor와 Row 서브쿼리가 반환하는 행은 **동일한 개수의 값**을 가져야 합니다.

- 2개 이상의 컬럼을 반환하는 서브쿼리와 비교할 때 Row Constructor 사용 가능
- 단일 컬럼만 반환하는 서브쿼리는 **스칼라 값**으로 취급되므로 Row Constructor 사용 불가

예시 (실패하는 경우):

~~~sql
SELECT * FROM t1 WHERE ROW(1) = (SELECT column1 FROM t2);
~~~

→ 단일 컬럼 반환이므로 문법 오류 발생



### **Row Constructor의 활용**

Row Constructor는 다른 맥락에서도 사용 가능합니다.

예를 들어, 다음 두 쿼리는 **동일한 의미**를 가집니다:

~~~sql
SELECT * FROM t1 WHERE (column1, column2) = (1, 1);
SELECT * FROM t1 WHERE column1 = 1 AND column2 = 1;
~~~



**예제 2 : 두 테이블에서 동일한 행 찾기**

~~~sql
SELECT column1, column2, column3
  FROM t1
  WHERE (column1, column2, column3) IN
        (SELECT column1, column2, column3 FROM t2);
~~~

→ t1에 있는 행 중 t2에도 동일하게 존재하는 행을 반환



### **참고**

- Row 비교 연산자 동작 방식: **Section 14.4.2, “Comparison Functions and Operators”**
- Row Constructor 최적화: **Section 10.2.1.22, “Row Constructor Expression Optimization”**



---

## 15.2.15.6 Subqueries with EXISTS or NOT EXISTS

### 개요

- **EXISTS** 서브쿼리: 서브쿼리가 한 행이라도 반환하면 `TRUE`  
- **NOT EXISTS** 서브쿼리: 서브쿼리가 아무 행도 반환하지 않으면 `TRUE`

---

### 기본 예시

```sql
SELECT column1 FROM t1 WHERE EXISTS (SELECT * FROM t2);
```

- t2가 비어있지 않다면(행이 존재한다면) EXISTS → TRUE
- t2가 비어있다면 EXISTS → FALSE

> 참고: EXISTS 서브쿼리에서는 SELECT *, SELECT 5, SELECT column1 등 어떤 SELECT 리스트를 사용하더라도 결과에 영향 없음.

> (MySQL은 EXISTS 서브쿼리의 SELECT 절을 무시함)

### **실제 활용 예시**

#### **1) 한 개 이상의 도시에서 존재하는 상점 유형 찾기**

~~~sql
SELECT DISTINCT store_type FROM stores
  WHERE EXISTS (
    SELECT * FROM cities_stores
      WHERE cities_stores.store_type = stores.store_type
  );
~~~

#### **2) 어느 도시에도 존재하지 않는 상점 유형 찾기**

~~~sql
SELECT DISTINCT store_type FROM stores
  WHERE NOT EXISTS (
    SELECT * FROM cities_stores
      WHERE cities_stores.store_type = stores.store_type
  );
~~~

#### **3) 모든 도시에 존재하는 상점 유형 찾기**

~~~sql
SELECT DISTINCT store_type FROM stores
  WHERE NOT EXISTS (
    SELECT * FROM cities WHERE NOT EXISTS (
      SELECT * FROM cities_stores
       WHERE cities_stores.city = cities.city
         AND cities_stores.store_type = stores.store_type
    )
  );
~~~



- 위 쿼리는 **이중 NOT EXISTS** 구조

- 의미:

  - 공식적으로는 “특정 도시에서 존재하지 않는 상점이 있나?”
  - 쉽게 말하면: **모든 도시에 존재하는 상점 유형을 찾는다**

  

**MySQL 8.0.19 이후 확장 문법**

MySQL 8.0.19부터는 TABLE 키워드를 EXISTS/NOT EXISTS 서브쿼리에서 사용할 수 있음.

~~~sql
SELECT column1 FROM t1 WHERE EXISTS (TABLE t2);
~~~

- SELECT * FROM t2와 동일한 의미
- t2가 비어있지 않으면 TRUE, 비어있으면 FALSE



---

## 15.2.15.7 Correlated Subqueries (상관 서브쿼리)

### 개요

**상관 서브쿼리(Correlated Subquery)**란 서브쿼리 안에서 외부 쿼리(outer query)에 있는 테이블을 참조하는 경우를 의미합니다.

---

### 예시

```sql
SELECT * FROM t1
  WHERE column1 = ANY (
    SELECT column1 FROM t2
    WHERE t2.column2 = t1.column2
  );
```

- 여기서 서브쿼리는 t1의 column2를 참조하고 있음.
- 비록 t1이 서브쿼리의 FROM 절에는 없지만, MySQL은 외부 쿼리에서 찾아 참조합니다.

### **동작 예시**

- t1: (column1=5, column2=6)
- t2: (column1=5, column2=7)

단순히 WHERE column1 = ANY (SELECT column1 FROM t2)라면 TRUE. 그러나 t2.column2 = t1.column2 조건이 FALSE → 전체 결과는 FALSE.



### **스코프 규칙**

MySQL은 **안쪽에서 바깥쪽 순서로 평가**합니다.

~~~sql
SELECT column1 FROM t1 AS x
  WHERE x.column1 = (
    SELECT column1 FROM t2 AS x
      WHERE x.column1 = (
        SELECT column1 FROM t3
          WHERE x.column2 = t3.column1
      )
  );
~~~

- 위 쿼리에서 x.column2는 t2의 컬럼입니다.

  (이유: SELECT column1 FROM t2 AS x ...가 t2를 x로 재정의했기 때문)

- 즉, 외부에 있는 t1의 x가 아니라, 더 안쪽의 t2를 가리킴.



### **옵티마이저 최적화 (MySQL 8.0.24+)**

- subquery_to_derived 플래그가 켜져 있으면, **상관 스칼라 서브쿼리 → 파생 테이블(derived table)** 변환 가능.

#### **예시**

~~~sql
SELECT * FROM t1 
  WHERE (SELECT a FROM t2 WHERE t2.a=t1.a) > 0;
~~~

이를 변환하면

~~~sql
SELECT t1.* FROM t1 
  LEFT OUTER JOIN
    (SELECT a, COUNT(*) AS ct FROM t2 GROUP BY a) AS derived
  ON t1.a = derived.a 
     AND REJECT_IF(
         (ct > 1),
         "ERROR 1242 (21000): Subquery returns more than 1 row"
     )
  WHERE derived.a > 0;
~~~

- REJECT_IF() → 내부적으로 **하나 이상의 행이 반환되면 에러 처리** 역할
- 즉, “서브쿼리가 단일 행만 반환해야 한다”는 조건을 옵티마이저가 검증함.



### **변환 조건 (가능한 경우)**

- 서브쿼리는 SELECT 리스트, WHERE, HAVING 절에는 가능

(단, JOIN 조건, LIMIT, OFFSET, UNION 등은 불가)

- WHERE 절은 AND로만 연결 가능 (OR 있으면 불가)

- **변환 가능 조건**:

  - = 연산자만 가능 (<=>는 불가)
  - 좌/우 항 중 하나는 내부 참조, 하나는 외부 참조여야 함

  - 집계 함수(aggregate) 내부에는 상관 컬럼 불가

- SELECT, JOIN, ORDER BY, GROUP BY, HAVING 절에는 상관 컬럼 포함 불가 (단, WHERE만 가능)

- 윈도우 함수 사용 불가



### **정리**

- 상관 서브쿼리는 외부 쿼리의 행을 기준으로 매번 실행 → 성능에 영향 가능
- MySQL 8.0.24 이상에서는 일부 경우 옵티마이저가 자동 변환하여 성능 최적화
- 조건 충족 시 **파생 테이블 + JOIN**으로 변환됨