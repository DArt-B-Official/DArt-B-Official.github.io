---
title: "[Study] Python - requests"
date: 2022-03-17 00:00:00
categories: [Study, python]
tags: [webhacking, python]
---

# 패키지 설치

---

```shell
pip install requests
```
pip를 이용하여 requests 라이브러리를 설치

<br />

# 사용법

---

```python
import requests
from requests import *
```

<br />

# 요청(Request)

---

```python
requests.request(method, url, **kwargs)
```


<br />

## Request Header

requests는 요청 시 기본 값으로 Header 4개가 포함됨.

```
{
  'User-Agent': default_user_agent(),
  'Accept-Encoding': DEFAULT_ACCEPT_ENCODING,
  'Accept': '*/*',
  'Connection': 'keep-alive',
}
```

<br />

## Request Method

|메소드|함수|
|:----|:--|
|**GET**|requests.get()|
|**POST**|requests.post()|
|**PUT**|requests.put()|
|**DELETE**|requests.delete()|
|**PATCH**|requests.patch()|
|**OPTIONS**|requests.options()|
|**HEAD**|requests.head()|

요청은 모두 response 개체의 인스턴스 반환함

<br />

- **parameter** : 메소드별 매개변수

    - url
    	: URL을 넘겨주는 매개변수(url만 필수 요소이고 나머지는 선택 요소)
    - params
    	: 튜플, 딕셔너리 형식으로 매개변수에 넣으면 양식이 URL 인코딩 되어 URL에 추가
    - data
    	: 튜플, 딕셔너리 형식으로 매개변수에 넣으면 양식이 인코딩 되어 요청 본문에 추가
    - json
    	: JSON 매개변수를 이용하여 요청 본문에 JSON 형식으로 추가

<br />

# 응답(Response)

---

```python
res = requests.get(url)
```

- `res.status_code` : HTTP 응답 코드
- `res.text` : Text 또는 HTML 형태의 데이터
- `res.json` : JSON 형태의 데이터
- `res.headers` : headers 정보
- `res.cookies` : cookies 정보
- `res.encoding` : 데이터 인코딩
- `res.content` :  bytes 타입의 데이터
