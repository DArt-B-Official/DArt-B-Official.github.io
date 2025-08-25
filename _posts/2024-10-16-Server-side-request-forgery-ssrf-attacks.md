---
title: '[PortSwigger] Academy: Server-side request forgery (SSRF) attacks'
date: 2024-10-16 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy Server-side request forgery (SSRF) attacks](https://portswigger.net/web-security/learning-paths/ssrf-attacks)] 을 수강하고 정리하였습니다.
{: .prompt-info }

<br><br>

# #What is SSRF?

---

서버 사이드 요청 위조(SSRF)는 공격자가 서버 측 애플리케이션이 의도되지 않은 위치로 요청을 보내도록 만드는 웹 보안 취약점이다.

일반적인 SSRF 공격에서, 공격자는 서버가 조직의 인프라 내에서 내부 전용 서비스에 연결하도록 할 수 있다. 다른 경우에는, 서버가 임의의 외부 시스템에 연결을 하도록 만드는 것이 가능할 수도 있다. 이는 인증 크레덴셜과 같은 민감한 데이터를 유출시킬 수 있다.

<br><br>

# #What is the impact of SSRF attacks?

---

성공적인 SSRF 공격은 자주 조직 내에서 인가받지 않은 동작이나 데이터에 대한 접근을 발생시킨다. 이는 취약한 애플리케이션이나 애플리케이션이 소통할 수 있는 다른 백엔드 시스템에서가 될 수 있다. 몇몇의 경우에는 SSRF 취약점이 공격자가 임의의 명령어 실행을 수행할 수 있게끔 할 수도 있다. 외부의 서드파티 시스템에 연결을 유발할 수 있는 SSRF 익스플로잇으로 인해 악의적인 공격이 발생할 수 있다. 이는 취약한 애플리케이션을 호스팅하는 조직에서 발생한 것으로 보일 수 있다.

<br><br>

# #Common SSRF attacks

---

SSRF 공격은 신뢰 관계를 악용하여 취약한 애플리케이션에서 공격의 권한을 상승시키거나 인가되지 않은 동작을 수행한다. 이러한 신뢰 관계는 서버와 관련하여 존재하거나 같은 조직 내의 다른 백엔드 시스템과 관련할 수 있다.

<br>

## SSRF attacks against the server

**서버를 대상으로 하는** SSRF 공격에서 공격자는 루프백 네트워크 인터페이스를 통해서 애플리케이션이 애플리케이션을 호스팅하고 있는 서버로 다시 HTTP 요청을 보내도록 할 수 있다. 여기에는 일반적으로 호스트 `127.0.0.1`이나 `localhost`와 같은 호스트명이 포함된 URL을 제공하는 작업이 포함된다.

<br>

예를 들어, 사용자가 한 아이템의 재고가 특정한 상점에 있는지를 확인할 수 있도록 하는 쇼핑 애플리케이션이 있다고 해보자.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

제고 정보를 제공하기 위해서는 애플리케이션은 다양한 백앤드 REST API 쿼리를 해야한다. 이것은 프론트엔드 HTTP 요청을 통해 관련 백엔드 API 엔드포인트에 URL을 전달함으로써 진행된다. 사용자가 한 아이템의 재고 상태를 볼 때, 그들의 브라우저는 위와 같은 요청을 수행할 것이다.

<br>

서버는 `/admin` URL의 내용을 가져오고 이를 사용자에게 전달한다. 공격자는 `/admin` URL에 방문할 수 있으나, 정상적으로는 관리자 기능은 인가된 사용자에게만 접근할 수 있어야 한다. 이 말은 공격자가 관심있는 그 어느 것도 볼 수 없다는 뜻이다. 그러나, `/admin` URL에 대한 요청이 로컬 시스템에서 온다면, 정상적인 접근 제어는 우회된다. 이 애플리케이션은 관리자 기능에 대한 전체 액세스 권한을 부여하는데, 이는 요청이 신뢰하는 곳에서부터 오는 것처럼 보이기 때문이다.

<br>

왜 애플리케이션은 이러한 방식으로 동작하며, 암묵적으로 로컬 시스템에서부터 오는 요청을 신뢰하는가? 이는 다양한 다음의 이유들이 있다.

<br>

- 접근 통제 확인은 애플리케이션 서버 앞에 놓여있는 다른 구성요소에서 구현될 수 있다. 서버에 다시 연결되면, 검사는 우회된다.
- 재해 복구 목적으로, 애플리케이션은 로컬 시스템으로부터 오는 어떠한 사용자에게든 로그인 없이 관리자 접근을 허용할 수 있다. 이는 관리자가 자격 증명을 잃어버렸을 때 시스템을 복구할 수 있는 방법을 제공한다. 이것은 오직 완전히 믿을 수 있는 사용자가 서버에서 직접 왔다고 가정한다.
- 관리자 인터페이스는 애플리케이션과 다른 포트 번호에서 서비스 되고, 사용자에게 직접적으로 노출되지 않는다.

<br>

로컬 시스템에서 오는 요청이 평범한 요청과 다르게 다루어지는 이러한 종류의 신뢰 관계는 SSRF를 치명적인 취약점으로 만들곤 한다.

<br>

## 🚩Lab: Basic SSRF against the local server

문제 설명과 같이 이 랩은 내부 시스템으로부터 데이터를 가져오는 재고 확인 기능을 가지고 있다. 임의의 상품 페이지의 하단에 도시별로 재고를 조회할 수 있는 기능이 존재하는 것을 확인할 수 있다. 재고를 조회할 때 요청하는 패킷에는 `stockApi` 라는 필드가 존재하고 이곳에 요청하고자 하는 API의 URL을 전달하는 것을 알 수 있다.

<br>

![image](https://github.com/user-attachments/assets/3f17cb83-fc42-4bfd-a263-10eb90e95e49)

이 필드 값에 `http://localhost/admin/delete?user=carlos`와 같이 `carlos` 사용자를 지우기 위한 url과 파라미터명을 유추하여 요청 url을 구성한 후에 패킷을 전송하면, 응답으로 **"Missing parameter 'username'"**를 받을 수 있다. 이를 통해 파라미터명이 `username`이라는 사실을 알아낼 수 있다.

<br>

이제 요청 url을 `http://localhost/admin/delete?username=carlos`로 구성하여 패킷을 전송하면 랩을 해결할 수 있다.

<br>

## SSRF attacks other back-end systems

어떤 경우에서는 사용자가 직접적으로 접근할 수 없는 백엔드 시스템과 애플리케이션이 상호작용할 수 있는데 이러한 시스템은 라우팅할 수 없는 개인 IP 주소를 가지고 있다. 백엔드 시스템은 일반적으로 네트워크 토폴로지에 의해 보호되기 때문에 더 약한 보안 형태를 갖고 있곤 한다. 많은 경우에서 내부의 백엔드 시스템은 이 시스템에 상호작용할 수 있는 누군가의 의한 인증이 없이 접근할 수 있는 민감한 기능을 포함하고 있다.

<br>

## 🚩Lab: Basic SSRF against another back-end system

[풀이](https://1unaram.github.io/posts/PortSwigger-Server-side-vulnerabilities/#lab-basic-ssrf-against-another-back-end-system)

<br>

# #Circumventing common SSRF defenses

---

SSRF 동작을 포함하여 악의적인 익스플로잇을 방지하기 위한 방어를 갖고 있는 애플리케이션을 흔히 볼 수 있는데 이러한 방어는 우회될 수 있다.

<br>

## SSRF with blacklist-based input filters

몇몇의 애플리케이션은 `127.0.0.1`이나 `localhost`와 같이 호스트명을 가진 입력을 차단하거나 `/admin`과 같은 민감한 url 주소를 차단한다. 이러한 경우에 다음과 같은 방법으로 필터링을 우회할 수 있다.

<br>

- `2130706433`, `017700000001`, `127.1`과 같은 `127.0.0.1`의 대체 IP 표현을 사용한다.
- `127.0.0.1`로 해석되는 자신만의 도메인 이름을 가입한다. PortSwigger에서 제공하는 `spoofed.burpcollaborator.net`을 사용해도 된다.
- 차단된 문자열을 URL 인코딩이나 다른 방법을 사용하여 난독화시킨다.
- 타겟 URL로 리다이렉트하는 URL을 이용한다. 리다이렉트 하는 동안 `http:`를 `https:`로 바꾸어 일부 SSRF 방지 필터를 우회하는 것처럼 타겟 URL에 대해 다른 리다이렉트 코드나 다른 프로토콜을 사용한다.

<br>

## 🚩Lab: SSRF with blacklist-based input filter

![image](https://github.com/user-attachments/assets/f6d7c049-6679-4f89-abf5-492edc0f2881)

이전 랩과 같이 각 지점 별 재고를 조회할 때의 패킷에서 `stockApi` 필드로 url 요청을 보내는 것을 확인할 수 있다. 그러나 이전 랩과는 다르게 localhost로 요청을 보내도록 하면 보안 문제로 차단되었다는 응답 패킷을 받을 수 있다. localhost 문자열 필터링을 우회하여 localhost로 요청을 보내야 한다.

<br>

`127.0.0.1`의 대체 표현 IP인 `127.1`을 포함하여 `stopApi=http://127.1` 값으로 패킷을 전송하니 응답 패킷으로 정상 값을 받을 수 있는 것을 확인할 수 있다. `localhost` 문자열에 대한 필터링을 우회할 수 있음을 알 수 있다.

<br>

이제 `stockApi=http://127.1/admin` 값으로 패킷을 전송해보면 다시 필터링 되는 것을 확인할 수 있는데, 이는 `/admin` url에 접속하는 것에 대해 필터링이 존재하는 것을 유추할 수 있다. 이를 우회하기 위해 문자열을 URL 인코딩하여 `/%61dmin`과 같이 문자열을 구성하여 패킷을 전송하면 또다시 필터링에 걸리는 것을 볼 수 있다.

이는 더블 url 인코딩에 따라 `%` 기호를 url 인코딩하여 `/%61dmin`을 `/%2561dmin`과 같이 구성할 수 있다. 이를 포함하면 필터링에 우회할 수 있다.

<br>

`stockApi=http://127.1/%2561dmin/delete?username=carlos` 으로 필드 값을 포함하여 패킷을 전송하면 해당 계정을 지워 랩을 성공할 수 있다.

<br>

## SSRF with whitelist-based input filters

블랙리스트 기반으로 입력 값을 필터링하는 것과는 달리 허용하는 값들의 화이트리스트와 입력 값을 매치하여 필터링 하기도 한다. 이는 URL 구문 분석의 불일치를 이용하여 필터를 우회할 수 있다.

<br>

URL specification에는 URL이 다음과 같은 방법을 사용하여 임시 구문 분석이나 유효성 검사를 구현할 때 간과될 수 있는 여러 기능이 포함되어 있다.

<br>

- `@` 문자를 이용하여 호스트명 이전에 자격증명 값을 포함시킬 수 있다.

    `https://expected-host:fakepassword@evil-host`

- `#` 문자를 사용하여 URL fragment를 지시할 수 있다.

    `https://evil-host#expected-host`

- DNS 이름 지정 계층 구조를 활용하여 필요한 입력을 자격이 충분한 DNS 이름에 배치할 수 있다.

    `https://expected-host.evil-host`

- URL 파싱 코드를 피해 URL 인코딩 된 문자를 삽입할 수도 있다. 이는 특수하게 필터가 구현되어 있는 코드가 백엔드 HTTP 요청을 수행하는 코드와는 다르게 url 인코딩 문자들을 다룰 때 유용하다. 이때, 더블 인코딩 문자를 이용할 수 있다.

<br>

## Bypassing SSRF filters via open redirection

때때로는 오픈 리다이렉션 취약점을 이용하여 필터 기반 방어를 우회할 수도 있다. 백엔드 HTTP 요청을 만드는 데 사용되는 API가 리다이렉션을 지원하는 경우에, 필터를 만족시키고 원하는 백엔드를 대상으로 리다이렉션 되는 요청을 생성하는 URL을 구성할 수 있다.

<br>

```
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

예를 들어 위의 URL에서 오픈 리다이렉션 취약점이 존재한다고 해보자. 이는 `http://evil-user.net`로 리다이렉션 된다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

이를 참고하여 오픈 리다이렉션 취약점을 이용하여 URL 필터를 우회함으로써 위와 같이 SSRF 공격을 할 수 있다. 애플리케이션이 `stockAPI` url이 허용된 도메인인지 먼저 확인하기 때문에 SSRF 공격이 동작한다. 애플리케이션은 그 다음으로 오픈 리다이렉션을 트리거하는 제공된 URL로 요청을 보낸다. 이는 리다이렉션을 수행하고 공격자의 선택에 따른 내부 URL로 요청을 보낸다.

<br>

## 🚩Lab: SSRF with filter bypass via open redirection vulnerability

![image](https://github.com/user-attachments/assets/44ec2bba-f4e1-4290-ace4-731b152cb619)

이전 랩들과 같이 지역 별 재고를 확인하는 기능이 존재하는 시스템이다. 여기서 한 상품의 설명 페이지에서 다음 상품 페이지로 리다이렉션 할 수 있는 *Next Product* 버튼이 존재한다. 해당 버튼을 클릭하면 `GET /product/nextProduct?currentProductId=2&path=/product?productId=3 HTTP/2`과 같이 GET 요청을 보내는 것을 확인할 수 있다. 이때, 다음 상품의 url을 `path` 파라미터에 포함시키는 것을 확인할 수 있다. 이 파라미터 값을 조작하여 원하는 url로 요청을 보내도록 해보자.

<br>

문제 설명과 같이 `http://192.168.0.12:8080/admin` url에서 관리자 인터페이스를 접근할 수 있는 것을 이용하여 `path` 파라미터에 포함하여 요청을 보내자.

<br>

`GET /product/nextProduct?currentProductId=2&path=http://192.168.0.12:8080/admin HTTP/2`

해당 요청에 대한 응답 패킷의 상태코드가 302인 것으로 보아 원하는 url로 성공적으로 리다이렉션이 되었고, 해당 url에서 오픈 리다이렉션 취약점이 존재하는 것을 확인하였다.

<br>

이제 문제에서 요구하는 것처럼 `carlos` 계정을 삭제하기 위해 리다이렉션 url을 변경하여 요청을 보내자. `/product/nextProduct?currentProductId=2&path=http://192.168.0.12:8080/admin/delete?username=carlos`와 같이 url을 구성하고 요청을 보내었다. 그러나, 302 상태코드를 받을 뿐 랩이 해결되지는 않았다.

원인을 찾던 중 해당 요청을 패킷으로가 아닌 브라우저에서 직접 하였는데, 리다이렉션 후에 `http://192.168.0.12:8080/admin/delete?username=carlos` 해당 주소로 내 브라우저가 요청을 보내는 것을 확인하였다. 가장 중요한 것을 빼먹은 것인데 해당 주소를 내 브라우저가 아니라 서버에서 요청하게끔 해야만 했다. 이것이 SSRF 공격에서 가장 중요한 점임을 간과했다.

<br>

서버 측에서 요청을 보내기 위해 재고 조회 시에 패킷에서 사용하는 `stockApi` 필드에 `/product/nextProduct?currentProductId=2&path=http://192.168.0.12:8080/admin/delete?username=carlos` 값을 포함하여 패킷을 전송하였고 랩을 해결 수 있었다.

<br><br>

# #Bline SSRF vulnerabilities

---

블라인드 SSRF 취약점은 애플리케이션이 제공된 URL로 백엔드 HTTP 요청을 시도하나, 백엔드 요청에 대한 응답이 애플리케이션의 프론트엔드에 전달되지 않을 때 발생한다. 블라인드 SSRF는 공격하기 더 어려우나, 때때로는 서버나 다른 백엔드 구성요소에서 완전한 원격 코드 실행을 이끌기도 한다.

<br>

블라인드 SSRF 취약점의 영향은 단방향 본성 때문에 일반 SSRF 취약점보다 미미하다. 몇몇의 경우에는 완전한 원격 코드 실행을 수행할 수 있음에도 불구하고, 백엔드 시스템으로부터 민감한 데이터를 조회하는 사소한 공격까지 불가능하기도 하다.

<br>

## How to find and exploit blind SSRF vulnerabilities

블라인드 SSRF 취약점을 탐지하기 위해 가장 신뢰성 있는 방법은 **out-of-band (OAST)** 방법을 이용하는 것이다. 이것은 통제할 수 있는 외부의 시스템으로 HTTP 요청을 보내도록 시도하고, 해당 시스템과 네트워크 상호작용이 있는지 모니터링 하는 것이 포함된다.

<br>

out-of-band 방법을 이용하는 가장 쉽고 효과적인 방법은 *Burp Collaborator*를 이용하는 것이다. Burp Collaborator를 이용하여 특정한 도메인 이름을 생성하고, 이 이름을 애플리케이션에 페이로드로 전송한 다음, 해당 도메인과의 상호작용을 모니터링 할 수 있다. 만약, HTTP 요청이 애플리케이션으로부터 들어오는 것이 확인되면 SSRF에 취약하다는 것을 알 수 있다.

<br>

> SSRF 취약점을 테스트할 때 제공된 Collaborator 도메인에 대한 DNS look-up을 관찰하는 것이 일반적이지만 후속 HTTP 요청은 없다. 이는 일반적으로 애플리케이션이 도메인에 HTTP 요청을 시도하여 초기 DNS 조회를 발생시키나, 실제 HTTP 요청은 네트워크 수준 필터링에 의해 차단되기 때문에 이러한 결과가 일어난다. 아웃바운드 DNS 트래픽을 허용하는 것은 비교적 일반적인데, 이는 다양한 목적으로 필요하지만 예상치 못한 목적지로의 HTTP 연결은 차단하기 때문이다.
{: .prompt-info }

<br>

단순히 out-of-band HTTP 요청을 유발할 수 있는 블라인드 SSRF 취약점을 식별하는 것만으로는 공격 가능성에 대한 경로를 제공하지 않는다. 백엔드 요청으로부터의 응답을 볼 수 없기 때문에, 애플리케이션 서버가 도달할 수 있는 시스템의 콘텐츠를 탐색하는 데에 이 동작을 사용할 수 없다. 잘 알려진 취약점을 탐지하기 위한 페이로드를 보냄으로써 내부 IP 주소 공간을 맹목적으로 탐색할 수 있다. 만약 이러한 페이로드가 블라인드 out-of-band 방법도 사용하는 경우, 패치되지 않은 내부 서버의 치명적인 취약점을 발견할지도 모른다.

<br>

블라인드 SSRF 취약점을 공격하는 또 다른 방법으로는 애플리케이션이 공격자의 제어 하에 있는 시스템에 연결하도록 유도하고, 연결을 수행하는 HTTP 클라이언트에 악의적인 응답을 반환하는 것이다. 만약 서버의 HTTP 구현에서 심각한 클라이언트 측 취약점을 공격할 수 있다면, 애플리케이션 인프라 내에서 원격 코드 실행을 수행하게 될 지도 모른다.

<br>

## 🚩Lab: Blind SSRF with out-of-band detection

(Burp Collaborator를 사용해야 하는 문제)

<br><br>

# #Finding hidden attack surface for SSRF vulnerabilities

---

많은 SSRF 취약점은 찾아내기 쉬운데, 이는 애플리케이션의 정상적인 트래픽이 URL 전체를 포함하는 파라미터 요청을 포함하기 때문이다. SSRF의 다른 예시의 경우에는 더 찾아내기 어렵다.

<br>

## Partial URLs in requests

때로는 한 애플리케이션이 호스트명이나 URL 경로의 일부부을 요청 파라미터에 포함시킨다. 제출된 값은 서버 측에서, 요청된 전체 URL에 통합된다. 만일 이 값이 손쉽게 호스트명이나 URL 경로로 인식된다면, 잠재적인 공격 표면이 명백할 수 있다. 그러나, 전체 SSRF로서의 공격 가능성은 공격자가 요청되는 URL 전체를 제어하지 않기 때문에 제한된다.

<br>

## URLs within data formats

몇몇의 애플리케이션이 특정 포맷의 데이터를 처리할 때, 해당 포맷의 specification 내에 URL이 포함될 수 있으며, 이 URL이 데이터 파서에 의해 자동으로 요청될 수 있다. 이에 대한 좋은 예시는 클라이언트에서 서버로 구조화된 데이터를 전송하기 위해 웹 애플리케이션에서 널리 사용되는 **XML** 데이터 형식이다. XML 데이터에서 외부 엔티티를 정의하고 이를 호출하면 서버가 해당 URL에 요청을 보낼 수 있다.

<br>

예를 들어,

```xml
<!DOCTYPE data [
  <!ENTITY file SYSTEM "http://attacker.com/evil">
]>
<data>&file;</data>
```
이 경우에, 애플리케이션이 XML을 파싱하면서 주어진 url에 서버가 요청을 보낼 수 있다. 이런 요청을 서버 내부 네트워크의 리소스에 접근하거나 공격자에게 정보를 유출할 수 있게 만든다. 애플리케이션이 XML 포맷 데이터를 입력으로 받고 이를 파싱한다면, XXE injection 공격에 취약하게 된다. XXE를 통해 SSRF 공격을 하는 것 역시 취약하다는 의미이다.

<br>

## SSRF via the Referer header

어떤 애플리케이션에서는 방문자를 추적하기 위해 서버 측에서 분석할 수 있는 소프트웨어를 사용한다. 이 소프트웨어는 요청 내의 Refer header를 로깅하여 링크의 유입을 추적한다. 이러한 소프트웨어는 Refer header에 존재하는 어떠한 서드 파티 URL이든지 방문을 할 때가 있다. 이는 일반적으로 참조 사이트의 콘첸츠를 분석하기 위해 수행되는데, 유입되는 링크에서 사용되는 앵커 텍스트 역시 포함된다. 그 결과, Refer header는 SSRF 취약점에 대한 공격 표현으로 유용하곤 하다.

<br><br>

![image](https://github.com/user-attachments/assets/62dd714c-5ca3-4777-a874-2f53f22b78cb)
