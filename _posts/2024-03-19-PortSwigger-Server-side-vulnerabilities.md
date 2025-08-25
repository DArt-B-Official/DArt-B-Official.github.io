---
title: '[PortSwigger] Academy: Server-side vulnerabilities'
date: 2024-03-19 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy Server-side vulnerabilities](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice)] 를 수강하고 정리하였습니다.
{: .prompt-info }

<br>

<br>

# #Path traversal

---

Directory traversal로 알려져 있다. 이 취약점은 공격자에게 어플리케이션이 실행 중인 서버에서 임의의 파일을 읽을 수 있도록 한다. 이때 파일은 다음을 포함할 수 있다.

- 어플리케이션의 코드와 데이터

- 백엔드 시스템의 인증

- 민감한 시스템 운영 파일

몇몇의 경우에는 공격자가 서버에 임의의 파일을 작성할 수 있도록 하고, 어플리케이션 데이터나 동작을 수정할 수 있도록 하며, 궁극적으로 서버의 모든 지휘권을 취할 수도 있다.

<br>

`<img src="/loadImage?filename=218.png">`

판매 중인 상품을 보여주는 쇼핑 어플리케이션을 생각해보자. 이는 위와 같은 HTML을 사용하여 이미지를 로딩할 것이다.

<br>

`/var/www/images/218.png`

`loadImage` url은 `filename` 파라미터를 받고 특정 파일의 내용을 보여준다. 만약 이미지 파일이 `/var/www/images/` 위치에 저장되어 있다면 위와 같은 파일 경로를 읽을 것이다.


<br>

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`

이 어플리케이션은 path traversal 공격에 대한 처리가 없다면, 공격자는 위의 URL을 요청함으로써 서버의 파일 시스템으로부터 `/etc/passwd` 파일을 조회할 수 있을 것이다.

<br>

문자열 `../`는 파일 시스템 내에서 <u>이전 파일 경로로 이동</u>하는 것을 의미하는 유효한 파일 경로이다. `../../../` 문자열을 통해 파일 시스템의 root `/`으로 이동할 수 있고, `/etc/passwd` 파일을 읽을 수 있게 된다.

<br>

해당 파일은 Unix 기반 시스템에서는 서버에 등록된 사용자의 세부정보를 포함한 기본 파일이지만, 공격자가 같은 방법을 이용하면 다른 임의의 파일을 조회할 수도 있을 것이다.

<br>

`https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`

Windows에서는 `../`와 `..\` 모두 유효한 directory traversal 문자열이다. 위는 Windows 기반 서버에서 유효한 공격 예시이다.

<br>

## 🚩Lab: File path traversal, simple case

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/328156d7-4194-47a2-9bd4-3955c2cd9b94)

*View details* 버튼을 클릭하여 상품을 조회할 때 패킷을 잡다보면 위와 같이 상품의 이미지를 GET 요청하는 패킷을 확인할 수 있다. 이때, `filename` 파라미터를 전달하는 것을 확인할 수 있는데 이 값을 `../../../../../../etc/passwd`로 설정하여 요청을 전송한다. `../` 문자열을 여러 번 사용하는 이유는 상품 이미지 파일이 저장되는 경로가 어디에 위치한지 정확히 모르기 때문에 여러 번 사용함으로써 root `/` 위치로 확실하게 이동하게 하기 위함이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5077e6e7-3561-45c0-9a17-b435d5275000)

이제 응답 패킷을 확인하면 해당 파일을 조회할 수 있음을 확인할 수 있다.

<br>

<br>

# #Access Control

Access Control은 어떤 사람이나 어떤 것이 동작을 수행하거나 리소스에 접근에 인가되었는지에 대한 어플리케이션의 제한이다. 웹 어플리케이션에서 이는 Authentication(권한 부여)와 Session management(세션 관리)에 달려 있다.

<br>

- **Authentication**은 사용자가 누구인지를 확인하는 과정

- **Session management**은 연속된 HTTP 요청이 같은 사용자로부터 이루어졌는지를 확인하는 과정

- **Access control**은 사용자가 동작을 수행하려는 시도가 허용되는지 판단하는 과정

<br>

Broken access controls은 흔하게 발생하고 치명적인 보안 취약점으로 종종 존재한다. Access controls의 설계와 관리는 비즈니스, 조직, 그리고 기술 구현에 대한 법적 제한에 적용되는 복잡하고 동적인 문제이다. Access control 설계 결정은 사람에 의해 이루어지기에, 오류에 대한 가능성이 높다.

<br>

## Vertical privilege escalation

사용자가 접근이 허용되지 않은 기능에 대한 접근을 획득했다면, 이것은 vertical privilege escalation이다. 예를 들어, 관리자가 아닌 사용자가 다른 사용자 계정을 지울 수 있는 관리자 페이지에 대한 접근을 획득했다면 이것이 vertical privilege escalation이다.

<br>

## Unprotected functionality

vertical privilege escalation은 어플리케이션이 민감한 기능에 대한 어느 보호도 되어있지 않을 때 발생한다. 예를 들어, 관리자 기능은 일반 사용자의 기본 페이지로부터가 아닌 관리자의 기본 페이지로부터 연결되어 있어야 한다. 그러나, 관리자와 관련된 url을 탐색함으로써 사용자가 관리자 기능에 접근할 수도 있을 것이다.

<br>

`https://insecure-website.com/admin`

예를 들어, 한 웹사이트가 위와 같은 url로 민감한 기능을 호스팅하고 있다고 하자.

이는 사용자 인터페이스 기능에 접근하는 링크를 갖고 있는 관리자 뿐만 아니라 접근이 허락되지 않은 어떠한 사용자에 의해서도 접근 가능할지도 모른다. 몇몇의 경우에는, 관리자 url이 `robots.txt` 파일과 같은 위치에 노출될 수도 있다.

<br>

url이 어느 곳에도 노출되어 있지 않더라도, 공격자는 민감한 기능의 경로에 대해 wordlist를 사용하여 brute-force을 할 수도 있다.

<br>

## 🚩Lab: Unprotected admin functionality

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/34222cd0-35d7-4772-a6b2-ff45674d5432)

`/robots.txt` 경로로 이동하면 위와 같은 페이지를 확인할 수 있고, `/administrator-panel` 페이지가 관리자와 관련된 페이지임을 유추할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e6d5ab8e-cb26-475f-8cc5-51112a18facc)

해당 경로로 이동하면 위와 같이 사용자를 삭제할 수 있는 페이지를 확인할 수 있고, 문제에 맞게 *carlos* 사용자를 삭제하면 문제를 해결할 수 있다.

<br>

어떤 경우에서는 예측할 수 없는 url을 사용함으로서 민감한 기능을 숨기기도 한다. 소위 **sercurity by obscurity**의 예시이다. 그러나, 민감한 기능을 숨기는 것은 효과적인 access control을 제공하지는 않는데, 이는 사용자가 다양한 방법으로 난독화된 url을 찾을 수 있기 떄문이다.

<br>

`https://insecure-website.com/administrator-panel-yb556`

위의 url을 통해 관리자 기능을 호스팅하는 어플리케이션이 있다고 하자.

이는 공격자가 곧바로 예측할 수는 없지만, 어플리케이션이 사용자에게 여전히 url을 노출시킬 수 있다. url은 사용자의 역할에 따라 사용자 인터페이스를 구축하는 Javascript에서 노출될 수 있다.

<br>

```javascript
<script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
</script>
```

이 스크립트는 사용자가 관리자라면 사용자의 UI에 링크를 추가하는 기능을 수행한다. 그러나, 이는 사용자의 역할에 관계없이 모든 사용자에게 보여질 수 있다.

<br>

## 🚩Lab: Unprotected admin functionality with unpredictable URL

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/3d041339-c414-479e-b9e1-c10315a807e5)

위와 같이 페이지 소스 코드에서 `script` 키워드로 검색하면 위와 같은 스크립트가 포함되어 있음을 확인할 수 있다. 이는 관리자에 대한 UI를 제공하는 스크립트로서 관리자 페이지의 경로인 `/admin-3vwkiw`를 확인할 수 있다.

해당 경로로 이동하여 문제에 맞춰 *carlos*를 삭제하면 문제를 해결할 수 있다.

<br>

## Parameter-based access control methods

어떤 어플리케이션은 사용자의 접근 권한이나 역할을 로그인에서 결정하고 이 정보를 사용자가 조작할 수 있는 위치에 저장하기도 한다. 여기서 사용자가 조작할 수 있는 위치는 **숨겨진 필드**, **쿠키**, **미리 정해진 쿼리 스트링 파라미터**가 될 수 있다.

<br>

`https://insecure-website.com/login/home.jsp?admin=true`

`https://insecure-website.com/login/home.jsp?role=1`

위와 같은 예시처럼 어플리케이션이 사용자가 제출한 값에 따라 access control 결정을 하기도 한다. 이 방법은 사용자가 값을 수정하고 관리자 기능과 같이 사용자가 접근해서는 안되는 기능에 접근할 수 있기 때문에 안전하지 않다.

<br>

## 🚩Lab: User role controlled by request parameter

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/6c7af10c-b820-4b25-ba9d-5bbd7334f91a)

문제에서 주어진 것처럼 `wiener:peter` 값으로 계정 로그인을 해보자. 그 다음 쿠키 값을 확인하면 위와 같이 `session`과 `Admin` 이름의 쿠키가 존재하는 것을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a6d957d4-97a2-4ab6-94ea-e5e4b091f656)

이제 `/admin` 페이지로 이동하면 admin 계정으로 로그인하지 않았기 때문에 해당 페이지에 접속할 수 없다는 문구를 확인할 수 있다. 이제 `Admin` 쿠키 값을 `false`에서 `true`로 바꾸고 다시 접속해보자.

성공적으로 관리자 페이지에 접근할 수 있고, *carlos* 계정을 삭제함으로써 문제를 해결하자.

<br>

## Horizontal privilege escalation

Horizontal privilege escalation은 사용자가 자기 자신의 리소스 대신에 다른 사용자의 리소스에 접근할 수 있을 때 발생한다. 예를 들어, 한 직원이 본인의 기록 뿐만 아니라 다른 직원의 기록까지 접근할 수 있을 때를 말한다.

Horizontal privilege escalation 공격은 vertical privilege escalation 공격 방법의 비슷한 유형이 사용된다. 예를 들어, 한 사용자가 다음 url을 통해 본인의 계정 페이지에 접근할 수 있다고 가정해보자.

`https://insecure-website.com/myaccount?id=123`

<br>

만약 공격자가 `id` 파라미터 값을 다른 사용자의 값으로 수정할 수 있다면, 공격자는 다른 사용자의 계정 페이지에 접근할 수 있고 관련된 데이터와 기능을 수행할 수 있을 것이다.

> 이는 Insecure Direct Object Reference (IDOR) 취약점의 예시이다. 이러한 종류의 취약점은 사용자가 조작할 수 있는 파라미터 값이 리소스나 기능에 직접적으로 접근할 수 있을 때 발생한다.
{: .prompt-info }

<br>

몇몇의 어플리케이션에서는, 공격 가능한 파라미터가 예측 가능한 값을 사용하지 않는다. 예를 들어, 어플리케이션이 사용자를 식별하기 위해 증가하는 숫자를 사용하는 것 대신에 globally unique identifiers (GUIDs)를 사용할 수도 있다. 이것은 공격자가 다른 사용자의 식별자를 추측하고 예측하는 것을 막는다. 그러나, 다른 사용자의 GUIDs는 사용자의 댓글이나 리뷰와 같이 사용자가 참조될 수 있는 곳에서 노출될 수도 있다.

<br>

## 🚩Lab: User ID controlled by request parameter, with unpredictable user IDs

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/66ffdb79-8b5c-4710-91c8-d20005afc51d)

*Coping with Hangovers* 포스트를 열어보면 *carlos*가 포스트의 작성자인 것을 확인할 수 있다. 페이지 소스 코드를 보면 *carlos*의 블로그로 이동할 수 있는 태그를 확인할 수 있는데 여기서 `userId` 값을 확인할 수 있다.

<br>

이제 로그인 페이지에서 `wiener:peter` 계정으로 로그인하면 계정 페이지에 로그인할 수 있는데, 접속된 url을 확인해보면 `id`라는 파라미터로 로그인 계정을 식별하고 있는 것을 확인할 수 있다. 이 값을 아까 획득한 carlos의 `userId`값으로 바꾸어 url 요청을 하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7c47881a-012c-44eb-8065-32d0d23d88af)

이제 carlos의 계정 페이지에 접속할 수 있고 API Key로 획득하였다. 해당 값을 Submit solution 하면 문제를 해결할 수 있다.

<br>

## Horizontal to vertical privilege escalation

종종 horizontal privilege escalation 공격은 더 많은 권한을 가진 사용자를 침해함으로써 vertical privilege escalation으로 바뀔 수 있다. 예를 들어 horizontal escalation은 공격자가 다른 사용자의 패스워드를 재설정하거나 획득할 수 있도록 한다. 만약 공격자가 관리자를 타겟으로 하고 관리자의 계정을 침해한다면, 공격자는 관리 권한을 획득하고 따라서 vertical privilege escalation을 수행할 수 있다.

<br>

`https://insecure-website.com/myaccount?id=456`

위의 예시와 같이 한 공격자가 horizontal privilege escalation에서 언급했던 파라미터 변조 기술을 사용하여 다른 사용자 계정 페이지에 대한 권한을 획득할 수 있다.

만약 공격자의 타겟인 사용자가 어플리케이션의 관리자라면, 공격자는 관리자 계정 페이지에 대한 접근을 획득할 수 있다. 이 페이지는 관리자의 패스워드를 노출시키거나 이를 바꿀 수 있는 수단을 획득할 수도 있고, 특정 권한이 주어진 기능을 직접적으로 접근할 수 있게 된다.

<br>

## 🚩Lab: User ID controlled by request parameter with password disclosure

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b90a4829-348b-4bf9-8d87-d54731966cf6)

우선 `wiener:peter` 계정으로 로그인을 해보자. 그러면 `/my-account?id=wiener` url 경로에서 계정 페이지를 확인할 수 있는데, 이 때 Password 필드에 사용자의 계정 패스워드 값이 `input` 태그의 `value` 속성으로 미리 채워져 있고 이 값을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/3c4ab7b2-dd75-43c8-aec0-ac89be6db304)

이제 `/my-account?id=administrator` url 경로로 요청을 해보면, 관리자 계정 페이지에 접속할 수 있고 Password 필드에 미리 채워진 패스워드 `7us9o2lpndjye6nnf5e6` 값을 획득할 수 있다. 해당 패스워드를 이용해 관리자 계정으로 로그인을 하면 위와 같이 *Admin panel*을 확인할 수 있다. 해당 페이지에서 *carlos*의 계정을 삭제하면 문제를 해결할 수 있다.

<br>

<br>

# #Authentication

## Authentication vulnerabilitis

Authentication vulnerabilities는 개념적으로 이해하기 쉬우나, 이는 인증과 보안 사이의 명백한 관계 때문에 치명적이다. Authentication vulnerabilities는 공격자가 민감한 데이터와 기능에 접근할 수 있도록 해준다. 또한 추가적인 익스플로잇을 위한 공격 표면을 노출시기기도 한다. 이러한 이유로 authentication vulunerabilities를 식별하고 익스플로잇하는 방법과 흔한 보호 수단을 우회하는 방법을 배우는 것이 중요하다.

<br>

## What is the difference between authentication and authorization?

인증은 한 사용자가 그들이 누구인지 주장하는 것을 확인하는 절차이다. 권한은 한 사용자가 어떠한 행동을 해도 되는지 확인하는 과정을 포함하고 있다.

<br>

예를 들어, 인증이 한 사이트에 접근을 시도하는 사용자명 `Carlos123`을 가진 누군가가 계정을 가진 인물과 동일한 인물인지를 결정한다. 일단 `Carlos123`이 인증되면, 그 사용자의 권한이 어떤 행동들이 허락되는지를 결정한다. 예를 들어, 사용자가 다른 사용자에 대한 개인 정보를 접근할 수 있는 권한을 가지거나 다른 사용자의 계정을 삭제할 수 있는 등의 행동을 수행하는 권한을 가질 수 있다.

<br>

## Brute-force attacks

Brute-force 공격은 공격자가 유효한 자격 증명을 추측하기 위해 시행 착오 시스템을 사용하는 경우를 말한다. 이 공격은 전형적으로 사용자의 이름과 패스워드 wordlist를 사용하여 자동화되어 있다. 이 공격을 위한 툴을 사용하는 이 자동화 과정은 잠재적으로 공격자가 빠른 속도로 막대한 횟수의 로그인 시도를 할 수 있게 해준다. Brute-forcing은 사용자 이름과 비밀번호를 완전히 무작위하게 만드는 경우만 존재하지는 않는다. 기본적인 논리 혹은 공개적으로 이용 가능한 지식을 이용함으로써 공격자는 미세 조정하여 훨씬 더 정확한 추측을 할 수 있다. 이것은 이러한 공격의 효율성을 상당히 증가시킨다. 사용자를 이증하는 유일한 방법으로 비밀번호 기반 로그인을 사용하는 웹 사이트는 충분한 brute-force 보호 기법을 적용하지 않는다면 매우 높은 취약점을 가질 수 있다.

<br>


## Brute-forcing usernames

사용자명은 이메일 주소와 같이 인식 가능한 패턴에 부합하는 경우에 특히 쉽게 추측할 수 있다. 예를 들어, `firstname.lastname@somecompany.com` 형식을 사용하는 매우 흔한 비즈니스 로그인 방식을 볼 수 있다. 그러나, 분명한 패턴이 존재하지 않더라도 높은 권한을 갖고 있는 계정이 `admin`이나 `administrator`와 같은 예측 가능한 사용자명으로 만들어지는 경우도 있다. 감사 중에 웹 사이트가 잠재적인 사용자명을 공개적으로 노출하고 있는지 확인해라. 예를 들어, 로그인 없이 사용자의 프로필에 접근할 수 있는가? 그렇다면 실제적인 프로필의 내용이 숨겨져 있더라도, 프로필에서 사용된 이름은 로그인할 때 사용되는 사용자명이랑 같은 경우가 있다. 또한 이메일 주소가 노출되지 않는지 HTTP 응답을 체크해야만 한다. 때때로는 응답에 관리자와 같이 높은 권한을 가진 사용자의 이메일 주소가 포함되기도 한다.

<br>

## Brute-forcing passwords

패스워드도 마찬가지로 brute-force 공격을 받을 수 있으며, 패스워드의 강도에 따라 난이도가 달라질 수 있다. 많은 웹 사이트는 패스워드 정책 폼을 사용하는데, 이는 사용자가 brute-force 공격만으로 크랙하기 여러운 높은 엔트로피를 가진 패스워드를 생성하도록 강제한다. 다음은 일반적으로 패스워드를 강제할 때 포함되는 것이다.

- 최소 글자 수
- 대소문자 혼합
- 1개 이상의 특수문자

그러나, 높은 엔트로피를 가진 패스워드는 컴퓨터 혼자만으로는 크랙하기 어려운 반면에, 우리는 사용자가 무의식적으로 이 시스템에 도입한 취약점을 익스플로잇하기 위해 사람의 행동에 대한 기본적인 지식을 이용할 수 있다. 무작위의 글자 조합으로 강한 패스워드를 만드는 것보다 사용자가 기억할 수 있는 패스워드를 선택하여 비밀번호 정책에 맞추려고 시도하는 경우가 많다. 예를 들어, `mypassword`가 패스워드로 허용되지 않았다면 사용자는 `Mypassword!`나 `Myp4$$w0rd`와 같은 것을 대신에 시도해볼 수 있다.

<br>

정책에 따라 사용자가 정기적으로 패스워드를 변경하도록 요구하는 경우에 사용자들이 선호하는 패스워드로 바꾸기 위해 사소하고 예측 가능한 변화를 만드는 일은 흔히 일어난다. 예를 들어, `Mypassword1!`는 `Mypassword1?`나 `Mypassword2!`가 될 수 있다. 가능한 자격 증명이나 예측 가능한 패턴에 대한 지식은 brute-force 공격을 더 정교하게 만들어 단순히 가능한 모든 글자 조합을 반복하는 것보다 더욱 효과적으로 만들 수 있다.

<br>

## Username enumeration

사용자명 열거(Username enumeration)은 공격자가 주어진 사용자명이 유효한지 확인하기 위해 웹 사이트 동작의 변화를 관측할 수 있게 해준다. 사용자명 열거는 일반적으로 로그인 페이지(예를 들어, 유효한 사용자명을 입력했으나 패스워드가 틀린 경우) 혹은 회원가입 페이지에서 이미 사용 중인 사용자명을 입력했을 경우에 발생한다. 이는 공격자가 유효한 사용자명의 최종 후보 목록을 생성할 수 있기 때문에 로그인 brute-force 공격에 대한 시간과 노력을 막대하게 줄일 수 있다. 로그인 페이지를 brute-force 공격을 시도하는 동안 다음 사항의 차이점에 특히 주의해야 한다.

- **상태 코드** : brute-force 공격을 하는 동안에 응답받은 HTTP 상태 코드는 대부분의 추측이 틀릴 것이기 때문에 동일할 가능성이 높다. 만약 다른 상태 코드를 받았다면, 사용자명이 맞았다는 강력한 표시이다. 웹 사이트에서는 결과에 관계없이 항상 동일한 상태 코드를 반환하는 것이 가장 좋은 방법이지만 이 방법이 항상 지켜지는 것은 아니다.

- **에러 메시지** : 때때로는 사용자명과 패스워드 모두 틀린 경우거나 패스워드만 틀린 경우에 따라 반환하는 에러 메시지가 다를 수 있다. 웹 사이트에서는 두 경우에 동일하고 일반적인 메시지를 사용하는 것이 가장 좋은 방법이지만 작은 타이핑 오류가 가끔 발생한다. 렌더링된 페이지에 해당 문자가 보이지 않는 경우에도 문자 하나만 잘못되어도 두 메시지가 구분된다.

- **응답 시간** : 대부분의 요청이 비슷한 응답 시간으로 처리된 경우, 이로부터 벗어나는 것은 뒤에서 뭔가 다른 일이 일어나고 있음을 나타낸다. 이는 사용자명이 맞았다는 추측을 나타내는 또다른 지표이다. 예를 들어, 한 웹 사이트가 사용자명이 유효한 경우에만 패스워드가 맞는지를 체크할 수 있다. 이 추가적인 과정은 응답 시간을 조금 늘릴 수 있다. 이는 미묘할 수도 있는데, 공격자는 웹 사이트가 처리하는데 눈에 띄게 오래 걸리는 과도하게 긴 패스워드를 입력함으로써 지연 시간을 더 분명하게 만들 수 있다.

<br>

## 🚩Lab: Username enumeration via different responses

이 문제는 주어진 wordlists를 이용해 brute-force 공격으로 로그인을 하는 문제이다. 파이썬 requests 모듈을 이용하여 코드를 짜는 방법도 있겠지만 Burp Suite의 **Intruder** 기능을 이용하여 문제를 해결해보자. Intruder는 Burp Suite에서 제공하는 웹 어플리케이션을 대상으로 사용자 정의 공격을 할 수 있는 자동화 도구이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/24feb153-5c41-4a8b-9739-345fe92ca82f)

우선 **Proxy - HTTP history** 탭에서 로그인을 시도한 요청 패킷을 찾아 우클릭하여 **Send to Intruder**를 클릭한다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/0dae99bc-4be8-4deb-9d46-980aae2650e5)

**Intruder - Positions** 탭에서 **Attack type**을 <u>Sniper</u>로 체크한다. **Payload positions**에서 POST 요청에 포함되는 값을 변경할 수 있는데, 위의 이미지처럼 username은 `§`로 감싸 변수처럼 만들어주고 패스워드는 아무런 값을 대입해두자. 우선 username을 알아내기 위함이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/6ccaf7a3-abd6-493b-9cad-0f9c1712df8e)

**Intruder - Payloads** 탭에서 **Payload sets**을 위의 이미지처럼 설정하여 준다.

<br>

문제에서 주어진 username의 wordlist를 복사하여 **Payload settings**에서 Paste를 눌러 리스트를 생성해준다.

<br>

이제 우측 상단에 있는 **Start attack** 버튼을 눌러 공격을 시작한다. 잠시 기다리면 모든 단어에 대해서 자동으로 username 값을 변경하며 요청을 처리하고 각종 정보를 보여준다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5b676151-78a5-4c72-8c7c-4d4f2012756a)

모든 단어에 대해 시도하고 나면 Length 값이 다른 하나의 요청을 찾을 수 있다. 이 페이로드가 올바른 username인 것을 알아낼 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/20b550ab-3afd-4b64-b399-d326401db14e)

이제 username을 `antivirus`로 채우고 password 값을 변수화하고 wordlist를 수정하여 같은 방식으로 공격을 시도하면 위와 같이 상태 코드와 길이가 다른 요청을 찾을 수 있고, `qwerty`가 올바른 password임을 알아낼 수 있다.

<br>

## Bypassing two-factor authentication

때로는 이중 인증(two-factor authentication) 구현에 결함이 있어 완전히 우회하는 경우도 있다. 만약 사용자에게 처음으로 비밀번호를 입력하라는 메시지가 표시된 다음에 구별된 페이지에서 인증 코드를 입력하라는 메시지가 표시되면, 사용자는 인증 코드를 입력하기 전에 사실상 "로그인" 상태에 있다. 이러한 경우에 첫 번째 인증 단계를 완료한 후 "로그인 전용" 페이지로 직접 건너뛸 수 있는지 테스트해 볼 가치가 있다. 때때로 웹 사이트가 페이지를 로딩하기 전에 두 번째 단계를 완료했는지 여부를 실제로 확인하지 않을 수도 있다.

<br>

## 🚩Lab: 2FA simple bypass

간단한 이중 인증 우회 문제이다. 우선 주어진 계정 `carlos:montoya`로 로그인을 한다. 이제 이중 인증 코드를 입력하는 창을 볼 수 있는데 이때, *Back to lab description* 버튼을 눌러 메인 화면으로 이동한다. 이 상태는 이미 로그인된 상태이다. 이제 My account로 이동하면 쉽게 문제를 풀 수 있다.

<br>

<br>

# #Server-side request forgery (SSRF)

서버 측 요청 위조(Sever-side request forgery) 즉, SSRF는 공격자가 서버 측 어플리케이션을 의도되지 않은 경로로 요청을 보내도록 만들 수 있도록 하는 웹 보안 취약점이다. 일반적인 SSRF 공격에서 공격자는 서버가 조직 인프라 내의 내부 전용 서비스에 연결되도록 할 수 있다. 다른 경우로 공격자는 서버가 임의의 외부 시스템에 연결됟록 할 수 있다. 이는 인증 자격 증명과 같은 민감한 데이터가 유출될 수 있다.

<br>

## SSRF attacks against the server

서버에 대한 SSRF 공격에서, 공격자는 loopback 네트워크 인터페이스를 통해 어플리케이션이 어플리케이션을 호스팅하는 서버에 HTTP 요청을 다시 보내도록 한다. 여기에는 일반적으로 `127.0.0.1`(loopback 어댑터를 가리키는 예약된 IP 주소) 혹은 `localhost`(같은 어댑터를 위해 일반적으로 사용되는 이름)과 같은 호스트 이름이 포함된 URL을 제공하는 작업이 포함된다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

예를 들어, 사용자가 특정한 상점에 한 상품이 재고가 있는지 사용자가 확인할 수 있는 쇼핑 어플리케이션이 있다고 가정하자. 재고 정보를 제공하기 위해서는 어플리케이션은 다양한 백엔드 REST API 쿼리를 해야만 한다. 프론트엔드 HTTP 요청을 통해 관련 백엔드 API 엔드포인트에 URL을 전달하여 이를 수행한다. 사용자가 상품에 대한 재고 상태를 조회할 때, 브라우저는 위와 같은 요청을 보낸다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

이는 서버가 특정한 URL에 요청을 보내고, 재고 상태를 조회하고, 이를 사용자에게 전달한다. 위의 예시는 공격자가 서버에 로컬 URL을 지정하도록 요청을 수정할 수 있다.

서버는 `/admin` URL에 있는 내용을 가져오고 이를 사용자에게 전달한다.

<br>

공격자는 `/admin` URL에 방문할 수 있지만 관리 기능은 일반적으로 인증된 사용자만 액세스할 수 있다. 이는 공격자가 관심있는 내용을 전혀 볼 수 없는 것을 의미한다. 그러나 로컬 서버에서 `/admin` URL로 요청을 보낸다면, 일반적인 액세스 제어는 우회된다. 요청이 신뢰할 수 있는 경로에서 시작된 것으로 나타나므로 어플리케이션은 모든 액세스를 관리 기능에 제공한다.

<br>

## 🚩Lab: Basic SSRF against the local server

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ede5c922-a8db-4b93-85c4-011a41337f69)

한 상품의 세부 정보 페이지로 들어가면 각 나라의 재고를 확인할 수 있다. Check stock 버튼을 누를 때 요청하는 패킷을 확인하면 위와 같이 `stockApi` 값을 전달하고 있다. 이제 해당 API를 `http://localhost/admin`로 변경하여 요청을 해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/1f1e5adb-f6f9-4e6b-8651-ac14ab9bc068)

왼쪽과 같이 요청을 보내면 오른쪽과 같은 응답을 얻는데 이는 관리자 페이지임을 알 수 있다. carlos 계정을 삭제하는 부분은 `a` 태그로 경로가 있으므로 `/admin/delete?username=carlos` 경로로 요청을 보내면 된다. 이제 다시 `stockApi` 값을 `stockApi=http://localhost/admin/delete?username=carlos`와 같이하여 요청을 전송하면 문제가 해결됨을 알 수 있다.

<br>

## SSRF attacks against other back-end systems

어떤 경우에서는 어플리케이션 서버가 사용자가 직접적으로 접근할 수 없는 백엔드 시스템과 상호작용할 수 있다. 이러한 시스템은 라우팅할 수 없는 개인 IP 주소가 있는 경우가 많다. 백엔드 시스템은 일반적으로 네트워크 토폴로지에 의해 보호되므로 보안 상태가 약한 경우가 많다. 많은 경우에 내부 백엔드 시스템에는 시스템과 상호작용할 수 있는 사람은 누구나 인증 없이 액세스할 수 있는 중요한 기능이 포함되어 있다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

이전 예시에서 `https://192.168.0.68/admin` 백엔드 URL에서 관리자 인터페이스가 있다고 해보자. 공격자는 SSRF 취약점을 익스플로잇하기 위해 위와 같은 요청을 보내고 관리자 인터페이스에 액세스할 수 있다.

<br>

## 🚩Lab: Basic SSRF against another back-end system

이 문제는 이전 문제와 같이 재고를 체크하는 기능에서 요청할 때 포함하는 `stockApi` 값을 `192.168.0.X`로 하여 관리자 페이지에 접근하면 되는 문제이다. 마찬가지로 **Intruder** 기능을 이용하여 마지막 숫자 값을 0부터 255로 하여 응답이 다른 주소를 찾아내면 된다. 페이로드는 `stockApi=http://192.168.0.§1§:8080/admin`로 하여 기능을 사용하면 된다.

<br>

`192.168.0.142`에서 `200` 응답코드를 받을 수 있고 `stockApi=http://192.168.0.142:8080/admin/delete?username=carlos` 값을 포함하여 요청을 보내면 문제를 해결할 수 있다.

<br>

<br>

# #File upload vulnerabilites

파일 업로드 취약점은 웹 서버가 사용자에게 파일의 이름, 타입, 내용, 사이즈와 같은 것들을 충분히 검증하지 않은 채로 파일 시스템에 업로드할 수 있도록 허용 해주는 경우를 이야기한다. 이에 대한 제한을 적절하게 적용하지 못하면 기본 이미지 업로드 기능조차도 대신 임의적이고 잠재적으로 위험한 파일을 업로드하는 데 사용될 수 있다. 이는 원격 코드 실행을 가능하게 하는 서버 사이드 스크립트를 포함할 수도 있다. 어떤 경우에는 파일을 업로드하는 행위 자체만으로 피해를 입힐 수 있다. 또다른 공격으로는 파일에 대한 후속 HTTP 요청이 포함될 수 있으며 일반적으로 서버에서 파일 실행을 트리거한다.

<br>

## How do file upload vulnerabilities arise?

매우 명백한 위험을 감안할 때, 실제 웹 사이트에서 사용자가 업로드할 수 있는 파일에 대해 어떠한 제한도 두지 않는 경우는 거의 없다. 보다 일반적으로 개발자는 본질적으로 결함이 있거나 쉽게 우회할 수 있는 강력한 검증이라고 생각하는 것을 구현한다. 예를 들어, 위험한 파일 타입을 블랙리스트에 두는 시도를 할 수 있지만 파일 확장자를 확인할 때 구문 분석 불일치를 고려하지 못한다. 모든 블랙리스트와 마찬가지로 여전히 위험할 수 있는 더  모호한 파일 형식을 실수로 생략하기도 쉽다.

<br>

## Exploiting unrestricted file uploads to deploy a web shell

보안 관점에서, 최악의 시나리오는 웹 사이트가 PHP, Java, Python 파일과 같은 서버 사이드 스크립트를 업로드하도록 허용하는 것이고, 그 코드들을 실행하도록 구성되어 있는 것이다. 이는 서버에서 자신만의 웹 쉘을 만드는 것이 쉽도록 한다.

> **웹 쉘(Web shell)**은 공격자가 쉽게 HTTP 요청을 올바른 엔드포인트에 전송함으로써 원격 웹 서버에서 임의의 명령을 실행하도록 하는 악의적인 코드이다.
{: .prompt-tip }

<br>

만약 성공적으로 웹 쉘을 업로드하는 것을 할 수 있다면, 서버에 대한 모든 제어권을 갖게 된다. 이는 임의의 파일을 읽거나 작성하고, 민감한 데이터를 유출하고, 심지어 서버를 사용하여 내부 인프라와 네트워크 외부의 서버 모두에 대한 공격을 전환할 수도 있다.

<br>

`<?php echo file_get_contents('/path/to/target/file'); ?>`

예를 들어 위와 같은 한 줄의 PHP 코드가 파일 시스템으로부터 임의의 파일을 읽도록 사용될 수 있다.

<br>

`<?php echo system($_GET['command']); ?>`

보다 많은 기능을 수행할 수 있는 웹 쉘은 위와 같다. 이 스크립트는 다음의 요청을 통해 시스템 명령어를 전달할 수 있다.

<br>

`GET /example/exploit.php?command=id HTTP/1.1`

<br>

## 🚩Lab: Remote code execution via web shell upload

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5167ba17-e54d-4b9e-858f-a1bb1b37b079)

주어진 계정 `wiener:peter`로 로그인하여 계정 페이지로 이동하면 아바타 사진을 업로드할 수 있는 기능이 존재한다. 우선 위와 같이 임의의 사진 하나를 업로드 해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b6d3db2e-cc53-478b-a396-24510cf91bf6)

이제 이미지가 업로드된 위치를 확인하기 위해 페이지의 소스코드에서 이미지를 어떤 경로에서 불러오는지를 확인해보자. 그럼 `/files/avatars/snoopy.png` 경로에서 이미지를 가져오는 것을 확인할 수 있다.

<br>

```php
// webshell.php
<?php echo system($_GET['command']); ?>
```

이제 해당 웹 사이트는 업로드 파일에 대한 검사를 진행하지 않기 때문에 웹 쉘 파일을 하나 제작하여 똑같이 업로드한다. 성공적으로 파일이 업로드되는 것을 확인할 수 있다.

<br>

`GET /files/avatars/webshell.php?command=ls HTTP/2`

이제 파일이 저장되는 해당 경로에 `command` 파라미터를 포함하여 시스템 명령어 `ls`를 실행하도록 요청하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/fd887d0b-90f3-466c-931a-beca31713464)

응답으로 `ls` 명령어에 대한 결과 값을 받을 수 있다.

<br>

`GET /files/avatars/webshell.php?command=cat+/home/carlos/secret HTTP/2`

이제 문제에서 설명하는 것처럼 `/home/carlos/secret` 파일을 읽기 위해 `cat` 명령어를 이용하여 파라미터를 구성하자. 이때 명령어가 공백으로 구분되기 때문에 URL에서 공백을 의미하는 `+` 문자를 추가해주었다.

이제 응답으로 파일 내용을 받을 수 있고 해당 문자열을 제출하면 문제를 해결할 수 있다.

<br>

## Exploiting flawed validation of file uploads

실제 환경에서 우리가 이전 lab에서 본 것처럼 파일 업로드 공격에 대한 어떠한 보호 기법도 적용되지 않은 웹 사이트를 찾기는 어렵다. 그러나 보호 기법이 있다고 해서 그것이 강하다는 것을 의미하지는 않는다. 때로는 원격 코드 실행을 위한 웹 쉘을 얻기 위한 이러한 메커니즘의 결함을 악용할 수 있다.

<br>

## Flawed file type validation

브라우저가 HTML form을 제출할 때, 일반적으로 `application/x-www-form-url-encoded` content type과 함께 `POST` 요청하여 데이터를 전송한다. 이는 이름이나 주소와 같은 간단한 텍스트를 전달하기에 좋다. 그러나 이미지 파일이나 PDF 문서와 같은 많은 양의 바이너리 데이터를 보내기에는 적합하지 않는다. 이러한 경우에는 `multipart/form-data` content type을 선호한다.

<br>

```
POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    wiener
    ---------------------------012345678901234567890123456--
```

이미지 업로드, 설명 입력, 이름 입력하는 필드를 포함하는 form에 대해 생각해보자. 위와 같은 요청을 제출할 것이다. 여기서 볼 수 있듯이, 메시지의 body는 form의 입력에 대해 각각 부분이 나누어져 있다. 각 부분은 `Content-Disposition` 헤더를 포함하고 있는데, 이는 입력 필드와 관련된 기본 정보를 제공한다. 이러한 개별적인 부분은 각자의 `Content-Type` 헤더를 포함할 수도 있는데, 이는 서버에게 이 입력을 사용하여 제출된 데이터의 MIME 유형을 알려준다.

<br>

웹 사이트가 파일 업로드에 대해 검증하는 방식은 특정한 `Content-Type` 헤더를 예측할 수 있는 MIME 타입과 매칭되는지를 확인하는 것이다. 예를 들어 서버가 이미지 파일만을 예측하는 경우에는 `image/jpeg`와 `image/png`와 같은 타입만을 허용할 것이다. 문제는 헤더의 값이 서버에 의해 암묵적으로 신뢰하고 있을 때 발생한다. 파일 내용이 실제로 예상되는 MIME 타입과 일치하는지 확인하기 위해 추가적인 검증이 수행되지 않는 경우에 이러한 보호 기법은 *Burp Repeater*와 같은 툴을 사용하여 쉽게 우회될 수 있다.

<br>

## 🚩Lab: Web shell upload via Content-Type restriction bypass

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/03557868-1349-4537-a530-4fdb1487993a)

이전 lab과 같이 PHP 웹 쉘을 제작하여 아바타 이미지로 업로드를 시도하여 보면 위와 같이 `application/octet-stream` 파일 유형은 허용되지 않고 `image/jpeg`나 `image/png` 유형만 허락된다는 문구를 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/9e487844-8e4b-479a-a50e-5870e235043b)

*Burp Suite*에서 방금 요청한 패킷을 살펴보면 `Content-Type`이 `application/octet-stream`으로 설정되어 요청이 전송된 것을 확인할 수 있다. 해당 패킷을 **Repeater** 기능으로 보내어 파일 유형을 `image/jpeg`으로 바꾸어 요청을 전송하여 보자.

<br>

이제 `GET /files/avatars/webshell.php?command=ls HTTP/2` 요청에 대한 응답을 통해 정상적으로 웹 쉘이 작동하는지를 확인해보면 성공적으로 웹 쉘이 업로드 된 것을 확인할 수 있다.

<br>

`GET /files/avatars/webshell.php?command=cat+/home/carlos/secret HTTP/2`

이제 문제에서 주어진 파일의 내용을 `cat` 명령어를 통해 조회하면 문제를 해결할 수 있다.

<br>

<br>

# #OS command injection

OS command injection은 shell injection으로 알려져 있다. 이는 공격자가 OS 명령어를 어플리케이션이 작동 중인 서버에서 실행하고 어플리케이션과 데이터까지 손상시키도록 허용한다. 종종 공격자는 OS command injection을 활용하여 호스팅 중인 인프라의 다른 부분을 손상 시키고 신뢰 관계를 악용하여 공격을 조직 내의 다른 시스템으로 전환할 수 있다.

<br>

OS command injection 취약점이 존재하는 것을 확인한 후에, 시스템에 대한 정보를 얻을 수 있는 몇몇의 초기 명령어는 유용하다. 아래의 표는 Linux와 Windows 플랫폼에서 유용한 명령어들이다.

|**Purpose of command**|**Linux**|**Windows**|
|--|--|--|
|Name of current user|`whoami`|`whoami`|
|Operating system|`uname -a`|`ver`|
|Network configuration|`ifconfig`|`ipconfig /all`|
|Network connections|`netstat -an`|`netstat -an`|
|Running processes|`ps -ef`|`tasklist`|

<br>

## Injecting OS commands

`https://insecure-website.com/stockStatus?productID=381&storeID=29`

이번 예시는 사용자가 특정 매장에 상품에 대한 재고가 존재하는지를 조회할 수 있는 쇼핑 어플리케이션이다. 위의 URL을 통해 정보에 대해 액세스할 수 있다.

<br>

`stockreport.pl 381 29`

재고 정보를 제공하기 위해서는 어플리케이션은 다양한 레거시 시스템을 쿼리해야 한다. 보통 이 기능은 제품 및 매장 ID를 위와 같은 인수로 사용하여 쉘 명령을 호출하여 구현한다.

<br>

`& echo aiwefwlguh &`

이 어플리케이션은 OS command injection에 대한 보호 기법이 구현되어 있지 않기에 공격자는 위와 같은 임의의 명령을 실행하기 위한 입력 값을 제출할 수 있다.

<br>

`stockreport.pl & echo aiwefwlguh & 29`

이 입력이 `productId` 매개변수로 제출되면 어플리케이션은 위와 같은 명령어를 실행할 것이다. `echo` 명령어를 사용하면 제공된 문자열을 출력할 수 있다. 이는 OS command injection에서 테스트하기에 유용한 방법이다. `&` 문자는 쉘 명령 구분자이다.

<br>

```
Error - productID was not provided
aiwefwlguh
29: command not found
```

이 예시에서 3개의 개별 명령이 차례로 실행된다. 사용자에게 보여지는 결과 값은 위와 같다.

- `stockreport.pl` 명령은 인자 없이 실행되기에 에러 메시지를 출력한다.
- 삽입된 `echo` 명령어는 실행되고 전달된 문자열이 결과로 출력된다.
- 원래의 `29` 인자는 명령어로서 실행되고 에러를 발생한다.

삽입된 명령어 뒤에 추가 명령 구분 기호 `&`를 배치하는 것이 유용한데 그 이유는 삽입된 명령을 삽입 지점 뒤에 나오는 어떤 것이든과 구분하기 때문이다. 이렇게 하면 다음에 나오는 내용으로 인해 삽입된 명령어가 실행되지 않을 가능성이 줄어든다.

<br>

## 🚩Lab: OS command injection, simple case

상품의 재고를 조회하는 `POST /product/stock HTTP/2` 요청에서 `productId`와 `storeId`를 body 값으로 포함하는 것을 확인할 수 있다.

<br>

`productId=1&storeId=%26echo%20hello%26`

이때 값에 `&`를 이용하여 OS 명령어를 실행하도록 위와 같이 body 값을 구성할 수 있다. 여기서 `%26`은 `&` 문자를 URL 인코딩한 값이고 `%20`은 공백 문자를 URL 인코딩한 값이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b14df0d8-09a5-4ade-a297-902de2a88c09)

이 body 값을 포함하여 요청을 보내면 `hello` 문자열을 포함한 응답을 받을 수 있다. 따라서 OS 명령어가 정상적으로 실행되고 그 결과 값을 반환 받을 수 있음을 알 수 있다.

<br>

`productId=1&storeId=%26whoami%26`

이제 `whoami` 명령어를 포함하여 body 값을 구성하고 요청을 보내면 문제를 해결할 수 있다.

<br>

<br>

# #SQL Injection

SQL injection(SQLi)는 공격자가 어플리케이션이 데이터베이스에 수행하는 쿼리를 방해할 수 있게 하는 웹 보안 취약점이다. 이는 공격자가 정상적으로 조회할 수 없는 데이터를 조회하도록 해준다. 이 데이터는 다른 사용자나 어플리케이션에 접근할 수 있는 다른 데이터를 포함할 수도 있다. 많은 경우에 공격자는 데이터를 수정하고 지움으로써 어플리케이션의 내용이나 동작의 영구적인 변화를 초래할 수 있다. 몇몇의 경우에서는 공격자는 SQL 인젝션을 확대하여 기본 서버나 기타 백엔드 인프라를 손상시킬 수도 있다. 이는 denial-of-service 공격을 수행할 수 있게 만들기도 한다.

<br>

## How to detect SQL injection vulnerabilities

- 싱글 쿼터 `'`를 입력하고 에러나 다른 변화가 있는지를 살펴본다.
- 원래 진입전과 다른 값을 평가하고 어플리케이션 응답에서 차이점을 찾는 일부 SQL 구문을 주입한다.
- `OR 1=1`이나 `OR 1=2`와 같은 조건문을 입력하고 어플리케이션 응답의 차이점을 확인한다.
- SQL 쿼리에서 시간 지연을 발생시키는 페이로드를 제출하고 시간에 따른 응답 차리를 확인한다.
- SQL 쿼리 내에서 실행될 때 네트워크 상호 작용을 발생하는 OAST 페이로드를 제출하고 상호작용을 모니터링한다.

또는 *Burp Scanner*를 사용해서 대부분의 SQL 인젝션 취약점을 빠르고 안정적으로 찾을 수 있다.

<br>

## Retrieving hidden data

`https://insecure-website.com/products?category=Gifts`

카테고리에 따라 상품을 보여주는 쇼핑 어플리케이션이 있다고 가정하자. 사용자가 **Gift** 카테고리를 클릭할 때 브라우저는 위와 같은 URL을 요청할 것이다.

<br>

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

이는 어플리케이션이 데이터베이스에서 관련된 상품의 정보를 조회하는 SQL query를 만들도록 할 것이다.

<br>

`released=1` 제한은 공개되지 않은 상품을 숨기기 위해 사용된다. 그렇다면 공개되지 않은 상품은 `released=0`이라고 가정할 수 있다.

<br>

`https://insecure-website.com/products?category=Gifts'--`

어플리케이션이 어떠한 SQL 주입 공격에 대한 보호 기법을 적용하지 않는다면 공격자가 위와 같은 공격을 수행할 수 있음을 의미한다.

<br>

`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

공격에 따라 만들어지는 SQL 쿼리는 위와 같다. `--`는 SQL에서 주석을 의미한다. 이는 잇따르는 구문은 주석으로 해석되고 이를 삭제하는 효과를 의미한다. 이 예시에서 `AND released = 1`는 더 이상 쿼리에 포함되지 않는다. 결과적으로 공개되지 않은 상품을 포함하여 모든 상품이 공개될 것이다.

<br>

`https://insecure-website.com/products?category=Gifts'+OR+1=1--`

어플리케이션이 카테고리에 있는 모든 상품을 표시하도록 하는 위와 같은 비슷한 공격을 할 수도 있다.

<br>

## 🚩Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

`GET /filter?category=Gifts HTTP/2`

카테고리를 선택할 때 위의 요청을 보내는 것을 확인할 수 있다. 이때 `category` 파라미터 값으로 해당하는 카테고리명을 전달한다.

<br>

`GET /filter?category=Gifts'+or+1=1+-- HTTP/2`

현재 보여지는 상품들은 `released=1` 조건이 포함된 상품들이므로 공개되지 않은 상품을 포함해서 보기 위해서는 위와 같이 SQL문을 구성해야한다. `--` 문으로 인해 잇따르는 `released` 제한을 무력화시킬 수 있기에 모든 상품들이 조회되는 것이다.

<br>

## Subverting application logic

`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`

이번에는 사용자명과 패스워드를 로그인에 사용하는 어플리케이션을 가정해보자. 만약 사용자가 사용자명으로 `wiener`을 패스워드로 `bluecheese`를 입력했다면 어플리케이션은 위와 같은 SQL 쿼리를 수행하여 자격을 체크할 것이다.

<br>

만약 쿼리가 사용자의 세부사항을 반환한다면 그 로그인은 성공한 것이다. 이런 경우에서 공격자는 패스워드 없이 어떠한 유저로도 로그인 할 수 있다. SQL 주석을 의미하는 `--`를 사용하여 `WHERE`절에서 패스워드 체크 과정을 삭제할 수 있다.

<br>

`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`

예를 들어, 사용자명으로 `administrator'--`와 패스워드에 공백 문자를 넣고 제출하면 위와 같은 쿼리를 구성할 것이다. 이 쿼리는 `username`이 `administrator`인 사용자를 반환하고 성공적으로 공격자가 해당 사용자로 로그인 하도록 한다.

<br>

## 🚩Lab: SQL injection vulnerability allowing login bypass

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/d805842f-9584-47f0-b147-ec98eb59e70b)

로그인 페이지에서 `username` 값으로 `administrator'--`을, `password` 값으로 임의의 문자를 넣어 로그인하면 패스워드를 체크하는 구문이 주석 처리되어 패스워드 없이 `administrator` 계정으로 로그인 할 수 있다.

<br><br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/496362b9-a2d5-4bc1-8e5e-c2393a43f619)
