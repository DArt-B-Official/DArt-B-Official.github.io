---
title: "2주차 정규과제(2) - MySQL 공식문서: Aggregate Function"
date: 2025-09-08 21:00:00 +0900
categories: [SQL_ADVANCED, Week2]
tags: [MySQL, Subqueries, 공식문서, 번역]
---

# 그룹 함수

## 14.19.1 Aggregate Function Description

\> **원문 출처:** [MySQL 8.0 Reference Manual - 14.19.1 Aggregate Function Description](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html)  

\> **번역:** DArt-B 학회

이 절에서는 값의 집합에 대해 동작하는 집계 함수들을 설명합니다. 일반적으로 GROUP BY 절과 함께 사용하여 값을 부분집합으로 나눠 집계합니다.

------

## **집계 함수 표**

| **이름**         | **설명**                          |
| ---------------- | --------------------------------- |
| AVG()            | 인자의 평균값 반환                |
| BIT_AND()        | 비트 AND 반환                     |
| BIT_OR()         | 비트 OR 반환                      |
| BIT_XOR()        | 비트 XOR 반환                     |
| COUNT()          | 반환된 행 수 반환                 |
| COUNT(DISTINCT)  | 서로 다른 값의 개수 반환          |
| GROUP_CONCAT()   | 문자열 연결 결과 반환             |
| JSON_ARRAYAGG()  | 결과 집합을 단일 JSON 배열로 집계 |
| JSON_OBJECTAGG() | 결과 집합을 단일 JSON 객체로 집계 |
| MAX()            | 최댓값 반환                       |
| MIN()            | 최솟값 반환                       |
| STD()            | 모표준편차(= STDDEV_POP())        |
| STDDEV()         | 모표준편차(= STDDEV_POP())        |
| STDDEV_POP()     | 모표준편차                        |
| STDDEV_SAMP()    | 표본표준편차                      |
| SUM()            | 합계 반환                         |
| VAR_POP()        | 모분산                            |
| VAR_SAMP()       | 표본분산                          |
| VARIANCE()       | 모분산(= VAR_POP())               |

> 별도 언급이 없으면, 집계 함수는 NULL 값을 무시합니다.



- GROUP BY가 없는 문장에서 집계 함수를 사용하면 “모든 행을 하나의 그룹으로 집계”한 것과 같습니다. (자세한 내용은 **14.19.3 MySQL의 GROUP BY 처리** 참조)
- 대부분의 집계 함수는 **윈도 함수**로도 사용할 수 있습니다. 구문 설명에 [over_clause]가 있으면 OVER (...) 절을 선택적으로 붙일 수 있음을 의미합니다(자세한 내용은 **14.20.2 윈도 함수 개념과 구문**).



### **반환 타입 요약**

- 분산·표준편차 함수: 숫자 인자에 대해 DOUBLE 반환

- SUM(), AVG():

  - 정밀-값 인자(INTEGER, DECIMAL): DECIMAL 반환
  - 근사-값 인자(FLOAT, DOUBLE): DOUBLE 반환

  

### **시간형(temporal) 주의**

SUM(), AVG()는 시간형에 직접 동작하지 않습니다(숫자로 변환되어 비의도적 손실). 우회 예:

~~~sql
SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(time_col))) FROM tbl_name;
SELECT FROM_DAYS(SUM(TO_DAYS(date_col))) FROM tbl_name;
~~~

### **숫자 기대 함수의 캐스팅**

SUM(), AVG() 등 숫자 인자를 기대하는 함수는 필요 시 인자를 수치로 캐스팅합니다. SET, ENUM은 내부 숫자 값이 사용됩니다.



------

## **비트 집계 함수의 인자/결과 타입 (MySQL 8.0)**

- 8.0부터 비트 함수/연산자는 **이진 문자열** 타입(BINARY, VARBINARY, BLOB) 인자를 허용하고, 동일 계열 타입으로 반환할 수 있어 64비트를 넘는 연산 가능.
- 인자가 이진 문자열이면 **이진 문자열로 평가**되고, 그렇지 않으면 **부호 없는 64비트 정수**로 평가됩니다.
- 이진 문자열 평가 시, 인자 길이가 다르면 ER_INVALID_BITWISE_OPERANDS_SIZE, 511바이트 초과면 ER_INVALID_BITWISE_AGGREGATE_OPERANDS_SIZE.
- mysql 클라이언트에서 --binary-as-hex 옵션에 따라 이진 문자열 결과가 16진수 표기로 표시될 수 있습니다.



------

## **함수별 상세**

### **AVG([DISTINCT] expr) [over_clause]**

- expr의 평균값 반환. DISTINCT는 서로 다른 값만 평균.
- 일치 행이 없거나 expr이 NULL만이면 NULL 반환.
- OVER 사용 시 윈도 함수로 실행(**DISTINCT와는 함께 사용 불가**).



예:

~~~sql
SELECT student_name, AVG(test_score)
FROM student
GROUP BY student_name;
~~~



### **BIT_AND(expr) [over_clause]**

- 모든 비트에 대한 AND.
- **평가 방식**
  - 이진 문자열 평가: 인자들이 이진 문자열 타입이면 동일 길이의 이진 문자열 반환(길이 불일치/초과 시 오류).
  - 숫자 평가: 부호 없는 64비트 정수로 변환 후 정수 반환.
- 일치 행이 없으면 “모든 비트가 1인 값”을 길이에 맞춰 반환.
- NULL은 무시(전부 NULL이면 중립값 반환).
- OVER 가능(8.0.12+).



### **BIT_OR(expr) [over_clause]**

- 모든 비트에 대한 OR.
- 평가 방식·제약은 BIT_AND와 동일.
- 일치 행이 없으면 “모든 비트가 0인 값” 반환.
- OVER 가능(8.0.12+).



### **BIT_XOR(expr) [over_clause]**

- 모든 비트에 대한 XOR.
- 평가 방식·제약은 BIT_AND와 동일.
- 일치 행이 없으면 “모든 비트가 0인 값” 반환.
- OVER 가능(8.0.12+).



------

### **COUNT(expr) [over_clause]**

- expr가 **NULL이 아닌** 값의 개수(BIGINT) 반환.
- 일치 행이 없거나 expr이 모두 NULL이면 0.
- COUNT(NULL)은 0.
- OVER 가능.

예:

~~~sql
SELECT s.student_name, COUNT(*)
FROM student s JOIN course c ON s.student_id = c.student_id
GROUP BY student_name;
~~~



#### **COUNT(\*)**

#### **에 대한 엔진별 동작**

- InnoDB:

  - 정확한 총행수를 저장하지 않으므로 트랜잭션 격리에 따라 가시 행만 집계.
  - 8.0.13+: 추가 절이 없으면 단일 스레드 워크로드 최적화.
  - 보통 가장 작은 보조 인덱스를 순회(힌트로 변경 가능). 보조 인덱스가 없으면 클러스터드 인덱스 스캔.
  - 버퍼 풀에 없으면 느릴 수 있음 → 정확성보다 속도가 중요하면 카운터 테이블(주의: 동시성 확장성 한계) 또는 대략값은 SHOW TABLE STATUS.
  - COUNT(*) vs COUNT(1) 성능 차이 없음.

  

- MyISAM:

  - 단일 테이블, 다른 컬럼 미조회, WHERE 없음이면 매우 빠름(정확 행수 저장).
  - COUNT(1)은 첫 번째 컬럼이 NOT NULL일 때만 동일 최적화.

  

### **COUNT(DISTINCT expr[, expr ...])**

- **서로 다른** NULL이 아닌 표현식(들)의 조합 수.
- 일치 행이 없으면 0.
- MySQL은 다중 표현식을 직접 지정 가능(표준 SQL은 보통 연결 필요).



------

### **GROUP_CONCAT(expr)**

- 그룹 내 NULL이 아닌 값들을 연결한 문자열. 모두 NULL이면 NULL.
- 구문:

~~~sql
GROUP_CONCAT([DISTINCT] expr [, expr ...]
  [ORDER BY {unsigned_integer | col_name | expr} [ASC | DESC] [, col_name ...]]
  [SEPARATOR str_val])
~~~

- 기본 구분자: ,(콤마). SEPARATOR ''로 구분자 제거 가능.
- 결과는 group_concat_max_len(기본 1024)까지 잘림. 런타임 변경 예:

~~~sql
SET [GLOBAL | SESSION] group_concat_max_len = <val>;
~~~

- 결과 타입: 인자가 비이진/이진에 따라 TEXT/BLOB (단, group_concat_max_len ≤ 512면 VARCHAR/VARBINARY).

예:

~~~sql
SELECT student_name, GROUP_CONCAT(test_score)
FROM student
GROUP BY student_name;

SELECT student_name,
       GROUP_CONCAT(DISTINCT test_score ORDER BY test_score DESC SEPARATOR ' ')
FROM student
GROUP BY student_name;
~~~



### **JSON_ARRAYAGG(col_or_expr) [over_clause]**

- 각 행 값을 요소로 하는 **단일 JSON 배열** 반환. 요소 순서는 정의되지 않음.
- 행이 없거나 오류 시 NULL. 인자가 NULL이면 [null] 요소 포함.
- OVER 가능(8.0.14+).



예:

~~~sql
SELECT o_id, JSON_ARRAYAGG(attribute) AS attributes
FROM t3
GROUP BY o_id;
~~~



### **JSON_OBJECTAGG(key, value) [over_clause]**

- (키, 값) 쌍을 모아 **JSON 객체** 반환. 행이 없거나 오류 시 NULL. 키가 NULL이거나 인자 수가 2가 아니면 오류.
- **중복 키**는 표준 MySQL JSON 사양에 따라 **마지막 키가 이김**(nondeterministic 가능).
- OVER 가능(8.0.14+). 프레임 내 중복 키가 있으면 정렬 보장 없이는 비결정적.
- 특정 키 값을 보장하려면 OVER (ORDER BY ...) 혹은 LIMIT 1 등으로 순서를 강제.



예(순서에 따른 결과 차이):

~~~sql
SELECT JSON_OBJECTAGG(c, i) FROM t;                -- {"key": ?} (비결정)
SELECT JSON_OBJECTAGG(c, i) OVER (ORDER BY i)      -- 순차적으로 누적
SELECT JSON_OBJECTAGG(c, i) OVER (ORDER BY i DESC) -- 역순 누적
SELECT JSON_OBJECTAGG(c, i) OVER (ORDER BY i) LIMIT 1; -- 최솟값만
~~~



### **MAX([DISTINCT] expr) [over_clause]**

- 최댓값 반환. 문자열 인자 가능(문자열의 최댓값). DISTINCT는 생략과 동일 결과.
- 일치 행이 없거나 expr이 NULL만이면 NULL.
- OVER 가능(**DISTINCT와 함께는 불가**).
- 참고: ENUM, SET 비교는 집합 내 위치가 아니라 **문자열 값**으로 비교(정렬과 동작 차이 가능).



### **MIN([DISTINCT] expr) [over_clause]**

- 최솟값 반환. 나머지 사항은 MAX와 동일.



------

### 표준편차·분산

- STD(expr) [over_clause]

  - 모표준편차. STDDEV_POP()의 별칭(MySQL 확장). 일치 행 없거나 NULL만이면 NULL. OVER 가능.

- STDDEV(expr) [over_clause]

  - 모표준편차. STDDEV_POP()의 별칭(Oracle 호환). 나머지 동일.

- STDDEV_POP(expr) [over_clause]

  - 모표준편차(= VAR_POP()의 제곱근). OVER 가능.

- STDDEV_SAMP(expr) [over_clause]

  - 표본표준편차(= VAR_SAMP()의 제곱근). OVER 가능.

- VAR_POP(expr) [over_clause]

  - 모분산(분모 = N). VARIANCE()와 동일하나 VAR_POP()이 표준. OVER 가능.

- VAR_SAMP(expr) [over_clause]

  - 표본분산(분모 = N-1). OVER 가능.

- VARIANCE(expr) [over_clause]

  - 모분산. VAR_POP()의 별칭(MySQL 확장). OVER 가능.

  

------

### **SUM([DISTINCT] expr) [over_clause]**

- 합계를 반환. 일치 행이 없거나 expr이 NULL만이면 NULL.
- DISTINCT로 서로 다른 값만 합산.
- OVER 가능(**DISTINCT와 함께는 불가**).