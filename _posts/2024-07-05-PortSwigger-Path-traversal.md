---
title: '[PortSwigger] Academy: Path traversal'
date: 2024-07-05 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy Path traversal](https://portswigger.net/web-security/learning-paths/path-traversal)] 을 수강하고 정리하였습니다.
{: .prompt-info }

<br><br>

# #What is path traversal?

---

Path traversal은 directory traversal로 알려져 있다. 이 취약점은 공격자가 애플리케이션이 구동 중인 서버에서 임의의 파일을 읽을 수 있도록 한다. 다음과 같은 파일을 포함한다.

- 애플리케이션 코드와 데이터
- 백엔드 시스템을 위한 크레덴셜
- 민감한 운영 시스템 파일

몇몇의 경우에는 공격자가 서버에 임의의 파일을 작성할 수도 있는데, 이는 그 파일들이 애플리케이션 데이터나 동작을 수정하고 결과적으로 서버의 모든 제어를 취할 수 있다.

<br><br>

# #Reading arbitrary files via path traversal

---

`<img src="/loadImage?filename=218.png">`

판매 중인 물건의 이미지를 보여주는 쇼핑 애플리케이션이 있다고 가정하자. 이는 위와 같은 HTML을 통해 이미지를 로드할 것이다.

<br>

`/var/www/images/218.png`

`loadImage` url은 `filename` 파라미터를 가지고 특정한 파일의 내용을 반환한다. 이 이미지 파일은 디스크 내 `/var/www/images/` 경로에 저장된다. 하나의 이미지를 반환하기 위해서는 애플리케이션이 요청된 파일 이름을 베이스 디렉터리에 붙이고 파일 시스템 API를 이용하여 파일의 내용을 읽는다. 다시 말해, 애플리케이션은 위와 같은 파일 경로를 읽는다.

<br>

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`

이 애플리케이션은 path traversal 공격에 대한 디펜스가 구현되어 있지 않다. 결과적으로 공격자는 위와 같은 URL을 요청하여 서버의 파일 시스템으로부터 `/etc/passwd` 파일을 조회할 수 있다.

<br>

`/var/www/images/../../../etc/passwd`

이는 애플리케이션이 위와 같은 파일 경로를 읽도록 하는데, `../` 시퀀스는 파일 경로 내에서 유효하고 디렉터리 구조의 상위 레벨로 올라감을 의미한다. `/var/www/images/`으로 부터 3개의 연속된 `../` 시퀀스는 파일 시스템의 루트로 올라갈 수 있고, 따라서 실제로 읽는 파일은 `/etc/passwd` 파일이다.

<br>

Unix 계열 os에서 이 파일은 서버에 등록된 사용자의 구체적인 기본 정보를 포함하고 있으나, 공격자는 같은 기법을 이용해 다른 임의의 파일을 조회할 수 있다. Windows에서 `../`와 `..\` 모두 유효한 디렉터리 탐색 시퀀스이다. 다음은 윈도우 기반 서버에 대한 동일한 공격의 예시이다.

`https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`

<br>

## 🚩Lab: File path traversal, simple case

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/42935d4c-d4fc-4819-98bb-f2342863ac28)

각 상품의 세부 사항을 보여주는 페이지에 접속할 시에 위와 같은 경로로 이미지를 요청하는 것을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/86653905-e905-47b7-903a-5a0aac6a22a3)

해당 url의 파라미터 `filename` 값으로 `../../../../etc/passwd`를 설정하고 요청하면 응답 패킷으로 `/etc/passwd` 파일을 받을 수 있다.

<br><br>

# #Common obastacles to exploiting path traversal vulnerabilities

사용자의 입력 값을 파일 경로에 포함시키는 많은 애플리케이션들은 path traversal 공격을 방어 기법을 구현한다. 만약 애플리케이션이 사용자가 제공한 파일명으로부터 디렉터리 탐색 시퀀스를 제거하거나 차단하는 경우에, 다양한 기법을 이용하여 우회할 수 있다.

<br>

`filename=/etc/passwd`와 같은 파일 시스템 루트의 절대 경로를 사용하여 어떠한 탐색 시퀀스를 사용하지 않고도 직접적으로 참조할 수 있다.

<br>

## 🚩Lab: File path traversal, traversal sequences blocked with absolute path bypass

이전 lab과 마찬가지로 `filename` 파라미터에 `../../../../etc/passwd`를 포함하여 요청하면 파일이 존재하지 않는다는 응답을 받게 된다. 그러나 절대 경로를 사용하여 파라미터 값을 `/etc/passwd`로 하여 요청하면 해당 파일을 응답으로 받을 수 있다.

---

<br>

또한  `....//` 혹은 `....\/`와 같은 nested traversal sequences를 사용하면, 내부 시퀀스가 제거될 때 기본 traversal sequence로 사용할 수 있다.

<br>

## 🚩Lab: File path traversal, traversal sequences stripped non-recursively

이전 lab들에서 사용한 파라미터로는 `/etc/passwd` 파일을 불러올 수 없다. 필터링을 우회하기 위해 `../` 시퀀스를 제거한다는 가정 하에 `....//` 시퀀스를 포함시키면 내부의 `../`가 제거되며 `../`가 남게 된다. 이를 이용하여 `....//....//....//etc/passwd`를 파라미터로 포함하여 요청을 보내면 해당 파일을 응답으로 받을 수 있다.

---

<br>

어떤 경우에서는 URL 경로 혹은 `multipart/form-data` 요청의 `filename` 파라미터 내에서와 같이 웹 서버가 사용자의 입력 값을 애플리케이션에 전달하기 전에 디렉터리 탐색 시퀀스를 제거할 수도 있다. `../` 문자열을 URL 인코딩 혹은 심지어 <u>dobule URL 인코딩</u>함으로써 이러한 종류의 sanitization을 우회할 수 있다. 이는 `%2e%2e%2f`와 `%252e%252e%252f`로 각각 인코딩된다. `..%c0%af` 혹은 `..%ef%bc%8f`와 같이 다양한 비표준 인코딩도 동작할 수 있다.

> **Double URL Encoding**<br>
> 1. `../` (Plain)<br>
> 2. `%2e%2e%2f` (URL Encoding)<br>
> 3. `%252e%252e%252f` (URL Encode %)

<br>

## 🚩Lab: File path traversal, traversal sequences stripped with superfluous URL-decode

이전 lab에서 사용한 페이로드는 모두 필터링 된다. 따라서 URL 인코딩이나 더블 URL 인코딩으로 필터링을 우회해보자.

<br>

우선 URL 인코딩을 사용해보자. `../` 시퀀스를 URL 인코딩하면 `%2e%2e%2f`이다. 이를 이용하여 `filename` 파라미터 값을 정하면 `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd`가 된다. 하지만 이 파라미터 값으로 요청을 보내면 응답을 받을 수 없음을 알 수 있다.

<br>

이제 더블 URL 인코딩 방법으로 인코딩을 해보자. `../`를 URL 인코딩하면 `%2e%2e%2f`이고, `%` 기호를 다시 한 번 URL 인코딩하면 `%252e%252e%252f`이 된다. 이를 이용하여 파라미터 값을 정하면 `%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd`이 되고 이를 포함하여 요청하면 `/etc/passwd` 파일을 읽을 수 있다.

---

<br>

애플리케이션은 사용자가 제공한 파일명을 `/var/www/images`와 같은 예상된 기본 폴더로 시작하도록 요청할 수 있다. 이러한 경우에는 적합한 탐색 시퀀스에 이어 요청된 기본 폴더를 포함할 수 있다. 예를 들어, `filename=/var/www/images/../../../etc/passwd`와 같다.

<br>

## 🚩Lab: File path traversal, validation of start of path

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ecd04aa2-13ba-4f79-a115-57c90517434b)

위와 같이 이미지를 요청하는 URL에 기본 폴더가 포함된 것을 확인할 수 있다. 이러한 경우에 파라미터 값에 기본 폴더를 포함하여 `/var/www/images/../../../etc/passwd`와 같이 설정하여 요청을 보내면 해당 파일을 응답으로 받을 수 있다.

---

<br>

이번엔 애플리케이션이 사용자가 제공한 파일명을 `.png`와 같이 예상된 파일 확장자로 끝내도록 요청할 수 있다. 이러한 경우에는 null byte를 요청된 확장자 이전에 추가하여 파일 경로를 효과적으로 끝낼 수 있다. 예를 들면, `filename=../../../etc/passwd%00.png`와 같다.'

<br>

## 🚩Lab: File path traversal, validation of file extension with null byte bypass

이 문제는 파일 확장자를 검사하는 필터링을 갖고 있다. 이를 위해 null byte를 확장자 앞에 넣어 우회할 수 있다. `filename` 파라미터 값을 `../../../etc/passwd%00.png`로 하여 요청하면 응답으로 파일의 내용을 받을 수 있다.

<br><br>

# #How to prevent a path traversal attack

---

가장 효과적으로 path traversal vulnerabilities를 방지하는 방법은 사용자로부터 제공받은 입력 값을 파일 시스템 API에 전달하는 것을 피하는 것이다. 이를 수행하는 많은 애플리케이션 기능을 다시 작성하여 보다 안전한 방식으로 동작할 수 있다.

<br>

만약 사용자로부터 제공받은 입력 값을 파일 시스템 API에 전달하는 것이 불가피하다면, 공격을 막기 위해 두 가지 레이어를 추천한다.

1. 사용자 입력을 사용하기 전에 검증한다. 이상적으로는 사용자의 입력 값과 허용되는 값의 화이트리스트를 비교하는 것이다. 만약 이것이 불가능하다면, 입력 값이 알파벳과 숫자와 같이 허용된 값만을 포함하고 있는지 검증하는 것이다.

2. 제공받은 입력 값을 검증한 후에 기본 디렉터리에 입력 값을 붙이고 플랫폼 파일 시스템 API를 사용하여 경로를 표준화해라. 표준화된 경로가 예상되는 기본 디렉터리로 시작하는지 검증하는 것이다.

<br>

```java
File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // process file
}
```

위의 예시는 사용자 입력 값에 기반으로 표준 경로를 검증하는 간단한 Java 코드의 예시이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/8d0fed6d-54d1-47b5-9b64-39d4549003d0)
