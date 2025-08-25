---
title: '[Study] Content Security Policy'
date: 2023-01-24 00:00:00
categories: [Study, Web Hacking]
tags: [webhacking]   
published: true
---

> [Dreamhack](https://dreamhack.io/) - Web Hacking Advanced (Client Side) 를 공부하며 정리하였습니다.
{: .prompt-info }

<br>

# # Content Security Policy

---

## Background

웹 브라우저는 웹 서버로부터 받는 컨텐츠가 의도된 컨텐츠인지 확인할 수 없기에, 페이지의 컨텐츠에서 사용하는 자원들이 모두 웹 서버에서 의도한 자원이 맞는지 확인하기 위해 **Content Security Policy(CSP)**가 탄생했다.

<br>

## Content Security Policy

**Content Security Policy(CSP, 컨텐츠 보안 정책)**는 XSS나 데이터를 삽입하는 류의 공격이 발생하였을 때 피해를 줄이고 웹 관리자가 공격 시도를 보고 받을 수 있도록 새롭게 추가된 보안 계층이다.

CSP 헤더는 1개 이상의 정책 지시문이 세미콜론으로 분리된 형태로 이루어져 있다. 정책 지시문은 지시문(e.g. `default-src`, `srcipt-src` 등)과 1개 이상의 출처(e.g. `'self'`, `https:`, `*.dreamhack.io` 등)가 공백으로 분리된 형태로 지정하여야 한다.

<br>

CSP 구문은 다음 방법으로 적용할 수 있다.

1. `Content-Security-Policy` HTTP 헤더에 추가하여 적용할 수 있다. 해당 구문에서 `policy-directive` 부분에 CSP를 정의하는 정책 디렉티브를 작성한다.
    
    ```
    Content-Security-Policy: <policy-directive>; <policy-directive>
    ```
    
    페이지 내부의 자원들이 같은 오리진 혹은 [https://example.com](https://example.com/) 에서만 로드되어야 함을 나타내는 예시
    
    ```
    Content-Security-Policy: default-src 'self' https://example.com
    ```
    

2. CSP 헤더는 `meta` 태그의 엘리먼트로도 정의할 수 있다.
    
    ```html
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://example.com">
    ```

<br>

## Content Security Policy 기본 정책

### Inline Code

CSP는 인라인 코드(Inline code)를 유해하다고 간주한다. 인라인 코드는 태그의 `src` 속성으로 코드를 로드하지 않고 태그 내에 직접 코드를 삽입하는 것을 의미한다.

`<script>alert(1);</script>` 와 같이 태그 내에 코드를 직접 삽입했기 때문에 인라인 코드이다. 이러한 형태를 지양하고, `<script src="alert.js"></script>`와 같이 `src` 속성에 코드 경로를 정의하는 방식을 권장한다.

CSP는 `<script>` 태그 내에 코드를 삽입하는 것, `on*` 이벤트 핸들러 속성, `javascript:` URL 스킴, `CSS` 스타일 시트를 모두 인라인 코드로 간주하고 허용하지 않는다.

<br>

### Eval

CSP는 기본적으로 문자열 텍스트를 실행 가능한 자바스크립트 코드 형태로 변환하는 매커니즘을 유해하다고 간주한다.

- `eval`
- `new Function()`
- `setTimeout([string], ...)`
- `setInterval([string], ...)`

위와 같이 문자열 형태로 입력을 받는 함수의 실행은 모두 차단된다. 다만 해당 함수에 문자열 입력이 아닌 이라인 함수 형태로 파라미터가 전달 될 때에는 차단되지 않는다.

<br>

## Policy Directive

`<Policy-directive`는 `<directive> <value>` 형태로 구성된다.

- `<directive>` : 지시문. 컨텐츠 내에서 로드하는 리소스를 세분화해 어떤 리소스에 대한 출처를 제어할지 결정한다.
- `<value>` : `<directive>`에서 정의한 리소스의 출처를 정의한다. `<value>`에는 여러 개의 출처가 정의될 수 있고 공백을 통해 구분된다.

<br>

| 지시문 | 설명 |
| --- | --- |
| defulat-src | -src로 끝나는 모든 리소스의 기본 동작을 제어. 만약 CSP 구문 내에서 지정하지 않은 지시문이 존재한다면 default-src의 정의를 따라간다. |
| img-src | 이미지를 로드할 수 있는 출처를 제어한다. |
| script-src | 스크립트 태그 관련 권한과 출처를 제어한다. |
| style-src | 스타일시트 관련 권한과 출처를 제어한다. |
| child-src | 페이지 내에 삽입된 프레인 컨텐츠에 대한 출처를 제어한다. |
| base-uri | 페이지의 <base> 태그에 나타날 수 있는 URL을 제어한다. |

<br>

| 출처 | 설명 |
| --- | --- |
| *://example.com | 출처의 scheme은 와일드카드(*)를 이용해 표현할수 있다. |
| https://*.example.com | 출처의 호스트 서브도메인은 와일드카드를 이용해 표현할 수 있다.(단, 와일드 카드는 호스트의 중간에 들어갈 수 없다.) 서브도메인을 와일드카드로 표현할 시, 서브도메인이 붙어있지 않는 도메인은 포함되지 않는다. i.e) https://*.example.com으로 출처를 표기할 경우, https:.//example.com은 포함 안됨 |
| https://example.com:* | 출처의 포트는 와일드카드를 이용해 표현할 수 있다. |
| none | 모든 출처를 허용하지 않는다. |
| self | 페이지의 현재 출처(Origin) 내에서 로드하는 리소스만 허용한다. |
| unsafe-inline | 예외적으로 인라인 코드의 사용을 허용한다. |
| unsafe-eval | 예외적으로 eval과 같은 텍스트-자바스크립트 변환 메커니즘의 사용을 허용한다. |
| nonce-<base64-value\> | nonce 속성을 설정하여 예외적으로 인라인 코드를 사용한다. <base64-value\> 는 반드시 요청마다 다른 난수 값으로 설정해야 한다. 해당 출처를 설정하면 unsafe-inline 은 무시된다. |
| <hash-algorithm\>-<base64-value\> | script 혹은 style 태그 내 코드의 해시를 표현한다. 해당 출처를 설정하면 unsafe-inline 은 무시된다. |

<br>

## CSP Examples

- `Content-Security-Policy: default-src 'self'`
    
    : 모든 리소스의 출처를 현재 페이지와 같은 출처로 제한한다.

<br>

- `Content-Security-Policy: default-src 'self' https://example.com`

    : 모든 리소스의 출처를 현재 페이지와 같은 출처와 https://example.com으로 제한한다.

<br>

- `Content-Security-Policy: default-src 'self'; img-src *; script-src static.example.com`

    : 모든 리소스의 출처를 현재 페이지와 같은 출처로 제한, 이미지의 출처는 모든 호스트를 허용, 스크립트 태그의 출처는 static.example.com으로 제한한다.

<br>

- `Content-Security-Policy: child-src 'self' frame.example.com`

    : 페이지 내에 삽입된 프레임 컨텐츠 URL은 frame.example.com 내의 컨텐츠만 로드할 수 있다.

<br>

- `Content-Security-Policy: base-uri 'none'`

    : base 태그의 URL은 어느 것도 허용하지 않는다.

<br>

- `Content-Security-Policy: script-src 'unsafe-eval`

    : 자바스크립트 코드 내에 `eval`과 같은 텍스트-자바스크립트 변환 메커니즘의 사용을 허용한다.

<br>

- `Content-Security-Policy: script-src 'unsafe-inline'`

    : 스크립트 태그 내 인라인 코드의 사용을 허용한다.

<br>

- `Content-Security-Policy: script-src 'nonce-YTQyYWZkODYtYWYyNy00ZGQzLTg2YjMtNzJhY2ZmOWY5OGNj'`

    : 스크립트 태그의 nonce 속성에 YTQyYWZkODYtYWYyNy00ZGQzLTg2YjMtNzJhY2ZmOWY5OGNj 값이 존재하지 않으면 스크립트 로드에 실패한다.

<br>

- `Content-Security-Policy: script-src 'sha256-5jFwrAK0UV47oFbVg/iCCBbxD8X1w+QvoOUepu4C2YA='`

    : 스크립트 태그 내의 코드 혹은 src 속성으로 지정된 파일의 sha256 해시를 base64로 인코딩한 결과가 5jFwrAK0UV47oFbVg/iCCBbxD8X1w+QvoOUepu4C2YA= 와 다르다면 스크립트 로드에 실패한다.

<br>

# # CSP Bypass

---

## 신뢰하는 도메인에 업로드

브라우저가 불러오는 자원의 출처가 파일 업로드 및 다운로드 기능을 제공한다면, 공격자는 출처에 스크립트와 같은 자원을 업로드한 뒤 다운로드 경로로 웹 페이지에 자원을 포함시킬 수 있다.

```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self'">

<h1>검색 결과: <script src="/download_file.php?id=177742"></script></h1>
```

<br>

## JSONP API

CSP에서 허용한 출처가 JSONP API를 지원한다면, `callback` 파라미터에 원하는 스크립트를 삽입하여 공격이 가능하다. 예를 들어 웹 페이지에서 `*.google.com`에서 온 출처만 허용할 경우, 구글에서 JSONP API를 지원하는 서버를 찾아 `callback`에 원하는 스크립트를 삽입할 수 있다.

```
https://accounts.google.com/o/oauth2/revoke?callback=alert(1);
```

<br>

## nonce 예측 가능

CSP의 `nonce`를 이용하면 따로 도메인이나 해시 등을 지정하지 않아도 공격자가 예측할 수 없는 `nonce` 값이 태그 속성에 존재할 것을 요구함으로써 XSS 공격을 방어할 수 있는데, 이를 위해서는 `nonce`가 공격자가 취득하거나 예측할 수 없는 값이어야 한다.

그러나 이 값을 생성하는 알고리즘이 취약하여 값을 예측할 수 있다면 공격자는 이를 유추해 자신의 스크립트를 웹 사이트에 삽입할 수 있다. ex) 현재 시각(`srand() / rand()`) 등 공격자가 알 수 있는 정보.

<br>

## base-uri 미지정

HTML 하이퍼링크에서 호스트 주소 없이 경로를 지정하면 브라우저는 현재 문서를 기준으로 문서를 해석한다. HTML `<base>` 태그는 경로가 해석되는 기준점을 변경할 수 있도록 하며, `<a>`, `<form>` 등의 `target` 속성의 기본 값을 지정하도록 한다.

이떄, `base-uri` CSP 구문을 지정하지 않은 경우 `base` 태그를 이용하여 임의 자원을 로드할 수 있다. 이러한 형태의 공격을 Nonce Retargeting이라고 부른다.

`base-uri` 지시문을 임의로 지정하지 않는 이상 `default` 초기 값이 존재하지 않는다. 따라서 웹 서비스를 개발할 때에는 반드시 `base-uri` 지시문을 정의해주어야 한다.