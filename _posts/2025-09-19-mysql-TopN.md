---
title: "4주차 정규과제 - MySQL 공식문서: SELECT Statement"
date: 2025-09-22 22:00:00 +0900
categories: [SQL_ADVANCED, Week4]
tags: [MySQL, WindowFunctions, 공식문서, 번역]
---

# Top N 쿼리 설명

## 15.2.13 SELECT Statement

> **원문 출처:** [MySQL 8.0 Reference Manual - 15.2.13 SELECT Statement]
(https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html)  
> **번역:** DArt-B 학회

## **개요**

`SELECT` 문은 한 개 이상의 테이블에서 행(row)을 조회하는 데 사용됩니다.  
또한 `UNION` 연산자, 서브쿼리, 공통 테이블 표현식(CTE, WITH 절)과 함께 사용할 수 있습니다.  

MySQL 8.0.31부터는 `INTERSECT`, `EXCEPT` 연산자도 지원합니다.  
자세한 내용은 관련 절을 참조하세요.

---

## **구문**

```sql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position} [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [into_option]
    [FOR {UPDATE | SHARE}
        [OF tbl_name [, tbl_name] ...]
        [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]
    [into_option]
```



---

## **SELECT 절 주요 옵션**

| **옵션**               | **설명**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| ALL (기본값)           | 모든 행을 반환, 중복 포함                                    |
| DISTINCT / DISTINCTROW | 중복 제거 후 행 반환                                         |
| HIGH_PRIORITY          | SELECT 실행을 UPDATE보다 우선 처리 (MyISAM 등 테이블 수준 잠금 엔진만 해당) |
| STRAIGHT_JOIN          | 옵티마이저가 아닌 작성 순서대로 조인 실행                    |
| SQL_SMALL_RESULT       | 결과가 작음을 힌트 → 메모리 임시 테이블 사용                 |
| SQL_BIG_RESULT         | 결과가 큼을 힌트 → 디스크 기반 임시 테이블/정렬 사용         |
| SQL_BUFFER_RESULT      | 결과를 임시 테이블에 버퍼링 → 테이블 잠금 빨리 해제          |
| SQL_CALC_FOUND_ROWS    | LIMIT 무시한 전체 행 수 계산 (→ FOUND_ROWS()로 조회 가능, 8.0.17부터 **deprecated**) |
| SQL_NO_CACHE           | 쿼리 캐시 비활성화 (8.0 이후 효과 없음, deprecated)          |

------

## 주요 절 설명

- **select_expr**

  조회할 컬럼 또는 표현식 지정. 최소 1개 필요.

  \* → 모든 컬럼 선택, tbl_name.* → 특정 테이블의 모든 컬럼 선택.

  보이지 않는 컬럼(invisible column)은 반드시 명시해야 조회됨.

- **FROM table_references**

  조회할 테이블 지정. 여러 테이블을 지정하면 조인.

  테이블/컬럼 별칭 가능. 인덱스 힌트(USE INDEX, FORCE INDEX) 가능.

- **PARTITION**

  파티션 테이블에서 특정 파티션만 조회 가능.

- **WHERE**

  행을 필터링하는 조건. 집계 함수 사용 불가.

- **GROUP BY**

  행을 그룹화. WITH ROLLUP 옵션 사용 가능.

  → HAVING은 그룹화된 결과에 조건을 적용.

- **HAVING**

  WHERE와 달리 집계 함수 조건 가능.

  단, 원래는 GROUP BY 컬럼/집계 함수만 허용되나 MySQL 확장으로 SELECT alias도 허용.

- **ORDER BY**

  정렬. ASC (오름차순, 기본), DESC (내림차순).

  숫자 인덱스(1,2,3…)도 가능하나 비표준 → 권장되지 않음.

- **LIMIT**

  반환할 행 수 제한.

  - LIMIT 5 → 처음 5행
  - LIMIT 5, 10 → 6번째부터 10개
  - PostgreSQL 호환: LIMIT 10 OFFSET 5

- **INTO**

  결과를 파일/변수에 저장.

  - INTO OUTFILE
  - INTO DUMPFILE
  - INTO var_name ...

- **FOR UPDATE / FOR SHARE**

  조회된 행을 잠금. 트랜잭션 종료 시까지 유지.

  - NOWAIT: 잠금 불가 시 에러 반환
  - SKIP LOCKED: 잠긴 행은 건너뛰고 반환

------

## 사용 예시

~~~sql
-- 단순 계산
SELECT 1 + 1;
-- 2

-- DUAL 테이블 (더미 테이블) 사용
SELECT 1 + 1 FROM DUAL;
-- 2

-- 컬럼 alias
SELECT CONCAT(last_name, ', ', first_name) AS full_name
FROM mytable ORDER BY full_name;

-- GROUP BY + HAVING
SELECT user, MAX(salary) 
FROM users
GROUP BY user
HAVING MAX(salary) > 10;

-- LIMIT 사용
SELECT * FROM tbl LIMIT 5;        -- 처음 5행
SELECT * FROM tbl LIMIT 5, 10;    -- 6행부터 10개
~~~



