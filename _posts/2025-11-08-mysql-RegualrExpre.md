---
title: "7주차 정규과제 - MySQL 공식문서: Regular Expressions"
date: 2025-11-08 22:00:00 +0900
categories: [SQL_ADVANCED, Week7]
tags: [MySQL, RegularExpression, 공식문서, 번역]

---

# Regular Expressions (정규 표현식)

## 14.8.2 Regular Expression Functions and Operators

> **원문 출처:** [MySQL 8.0 Reference Manual - 14.8.2 Regular Expressions]
> (https://dev.mysql.com/doc/refman/8.0/en/regexp.html)  
> **번역:** DArt-B 학회



## 개요

정규 표현식(Regular Expression)은 복잡한 문자열 검색 조건을 표현하는 강력한 수단이다.  

MySQL에서는 **ICU(International Components for Unicode)**를 기반으로 정규 표현식 기능을 제공하며,  이는 **유니코드 지원**과 **멀티바이트 안전성(multibyte-safe)**을 보장한다.MySQL 8.0.4 이전에는 Henry Spencer의 정규식 엔진을 사용했으나, 이는 바이트 단위로 처리되어 멀티바이트 문자에 안전하지 않다.

---

## 정규 표현식 관련 함수 요약

| 이름 | 설명 |
|------|------|
| `REGEXP`, `RLIKE` | 문자열이 정규식과 일치하는지 확인 |
| `NOT REGEXP` | 정규식 불일치 검사 |
| `REGEXP_INSTR()` | 일치하는 부분 문자열의 시작 위치 반환 |
| `REGEXP_LIKE()` | 정규식 일치 여부 검사 |
| `REGEXP_REPLACE()` | 정규식과 일치하는 부분 문자열 치환 |
| `REGEXP_SUBSTR()` | 일치하는 부분 문자열 반환 |

---

## REGEXP / RLIKE

```sql
SELECT 'Michael!' REGEXP '.*';   -- 1 (모든 문자열과 일치)
SELECT 'a' REGEXP '^[a-d]';    -- 1 (a~d로 시작)
```

- `REGEXP`, `RLIKE`는 동의어

- NULL 포함 시 결과도 NULL

- 대소문자 구분 없음 (기본)

---

## REGEXP_INSTR()

일치하는 부분 문자열의 **위치(인덱스)** 반환

~~~sql
SELECT REGEXP_INSTR('dog cat dog', 'dog');     -- 1
SELECT REGEXP_INSTR('dog cat dog', 'dog', 2);   -- 9
SELECT REGEXP_INSTR('aa aaa aaaa', 'a{4}');    -- 8
~~~

선택 인자:

- `pos`: 검색 시작 위치 (기본: 1)

- `occurrence`: 몇 번째 일치를 찾을지 (기본: 1)

- `return_option`: 0이면 시작 위치, 1이면 끝 다음 위치 반환

- `match_type`: REGEXP_LIKE()와 동일한 매칭 옵션

  

---

## REGEXP_LIKE()

문자열이 정규식과 **일치하는지 검사** (1 or 0 반환)

~~~sql
SELECT REGEXP_LIKE('abc', 'ABC');       -- 1 (기본: 대소문자 무시)
SELECT REGEXP_LIKE('abc', 'ABC', 'c');    -- 0 (대소문자 구분)
~~~



매칭 옵션 (`match_type`):

- `c`: 대소문자 구분

- `i`: 대소문자 무시

- `m`: 여러 줄 모드 (`^`, `$` 라인 단위로 해석)

- `n`: `.`이 줄바꿈 문자도 포함

- `u`: UNIX 줄바꿈(`\\n`)만 허용



---

## REGEXP_REPLACE()

정규식과 일치하는 문자열을 **새 문자열로 치환**

~~~sql
SELECT REGEXP_REPLACE('a b c', 'b', 'X');        -- a X c
SELECT REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3); -- abc def X
~~~



## REGEXP_SUBSTR()

정규식과 일치하는 **부분 문자열 반환**

~~~sql
SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+');     -- abc

SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3);  -- ghi
~~~

---

## 정규 표현식 문법 요약

| 패턴 | 의미 | 예시 |
|------|------|------|
| `^` | 문자열 시작 | `^fo`는 'fo'로 시작 |
| `$` | 문자열 끝 | `end$`는 'end'로 끝남 |
| `.` | 임의의 문자 하나 | `f.o` → 'foo', 'fao' 등 |
| `*` | 0회 이상 반복 | `a*` → "", "a", "aa" |
| `+` | 1회 이상 반복 | `a+` → "a", "aa" |
| `?` | 0~1회 반복 | `a?` → "", "a" |
| `|` | OR (선택) | `abc|xyz` |
| `()` | 그룹핑 | `(abc)+` |
| `{m,n}` | 반복 횟수 지정 | `a{2,4}` |
| `[]` | 문자 집합 | `[a-z0-9]` |
| `[^]` | 부정 문자 집합 | `[^a-d]` |
| `[[:digit:]]` | 숫자 클래스 | `[[:digit:]]` = `[0-9]` |
| `[[:alpha:]]` | 알파벳 클래스 | `[[:alpha:]]` = `[a-zA-Z]` |

---

## 정규식 처리 시 주의사항

- `\`는 MySQL 문자열 해석과 정규식 해석을 모두 거쳐야 하므로 **두 번 이스케이프(`\\`)** 해야 함

~~~sql
SELECT REGEXP_LIKE('1+2', '1\\+2'); -- 1
~~~

- `(`, `[`, `{` 같은 특수문자는 반드시 이스케이프 필요



## ICU 기반 MySQL 8.0 정규 표현식 특징



| 항목 | 내용 |
|------|------|
| 엔진 | ICU (유니코드 완전 지원) |
| 인덱스 기준 | UTF-16 단위 (4바이트 이모지는 주의) |
| 중단 제어 | `regexp_stack_limit`, `regexp_time_limit` 시스템 변수 사용 |
| 라인매칭 | 여러 줄 해석 위해 `m` 또는 `(?m)` 필요 |



~~~sql
SELECT REGEXP_LIKE('fo\\nfo', '^f.*$', 'm'); -- 여러 줄 매칭 허용
~~~



- 이모지나 4바이트 문자 → 인덱싱 오류 주의 (유니코드 코드포인트와 차이 있음)



---

