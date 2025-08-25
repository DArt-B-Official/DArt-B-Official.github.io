---
title: '[PortSwigger] Academy: WebSockets vulnerabilities '
date: 2024-11-12 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy WebSockets vulnerabilities ](https://portswigger.net/web-security/learning-paths/websockets-security-vulnerabilities)] 을 수강하고 정리하였습니다.
{: .prompt-info }

<br><br>

# #WebSockets

---

웹 소켓은 현대 웹 애플리케이션에서 널리 사용된다. 이는 HTTP를 통해 시작되며 양방향 비동기 통신을 통해 오래동안 지속되는 연결을 제공한다. 웹 소켓은 사용자 동작을 수행하고 민감한 정보를 전달하는 것을 포함하여 많은 목적으로 사용된다. 일반 HTTP에서 발생하는 거의 모든 웹 보안 취약성은 웹 소켓 통신과 관련하여 발생할 수도 있다.

![image](https://github.com/user-attachments/assets/70174e66-f02e-4931-ba06-f10257b298cc)

<br>

- 사용자에게 받은 입력을 서버로 전달하는 과정이 안전하지 않다면 이는 SQL 인젝션이나 XML 외부 엔티티 삽입과 같은 취약점으로 이어질 수 있다.
- 웹 소켓을 통해 도달하는 일부 블라인드 취약점은 out-of-band(OAST) 기술을 통해서만 감지할 수 있다.
- 공격자가 제어할 수 있는 데이터가 웹 소켓을 통해 다른 애플리케이션 사용자에게 전달 된다면 이는 XSS나 클라이언트 사이드 취약점으로 이어질 수 있다.

<br>

<br>

# #Manipulating WebSocket messages to exploit vulnerabilities

---

웹 소켓에 영향을 주는 입력 기반의 주요 취약점은 웹 소켓 메시지의 내용을 변조함으로써 익스플로잇 될 수 있다.

<br>

`{"message":"Hello Carlos"}`

에를 들어, 웹 소켓을 사용하여 브라우저와 서버 사이에 채팅 메시지를 전달하는 채팅 애플리케이션이 있다고 가정하자. 사용자가 채팅 메시지를 입력할 때, 위와 같은 웹 소켓 메시지가 서버로 전송된다.

<br>

`<td>Hello Carlos</td>`

다시 한 번 웹 소켓을 통해 다른 채팅 사용자에게 메시지의 내용이 전달 되고, 사용자의 브라우저에 위와 같이 렌더링 될 것이다.

<br>

`{"message":"<img src=1 onerror='alert(1)'>"}`

이 상황에서, 입력 값에 대한 다른 프로세싱이나 방어 기술이 없다면 공격자는 위와 같은 웹 소켓 메시지를 전송함으로써 XSS 공격에 proof-of-concept(poc)를 수행할 수 있다.

<br>

## 🚩Lab: Manipulating WebSocket messages to exploit vulnerabilities

해당 랩은 Live chat 기능이 구현되어 있다. 메시지를 전송하면 서포트 에이전트가 답장을 해주는데, 이를 프록시를 통해 전달 되는 패킷을 조작하여 `alert()` 함수를 실행시켜야 한다.

<br>

![image](https://github.com/user-attachments/assets/3c9d527b-68db-41e2-ae64-bd42f872c72b)

메시지를 전송할 때 패킷을 보면 위와 같이 `message` 라는 필드에 데이터가 포함되어 전송된다.

<br>

![image](https://github.com/user-attachments/assets/194b01e1-ded2-4c6a-9530-034cba9ac0f3)

그러면 위와 같이 전송하는 `user` 명과 내용을 포함하여 응답을 받을 수 있고 브라우저에 그대로 그려진다.

<br>

```
{"message":"hello<img src=1 onerror='alert(1)'>"}
```

이제 전송하는 메시지 패킷 값에 `alert()` 함수를 실행하는 스크립트를 포함하여 전송하면 응답으로 같은 내용을 받게 되고 이것이 브라우저에 그려지면서 함수가 실행된다.

<br><br>

# #Manipulating the WebSocket handshake to exploit vulnerabilities

일부 웹 소켓 취약점은 웹 소켓 handshake를 조작할 때 발견되고 공격 가능하기도 한다. 이러한 취약점은 다음과 같은 설계 결함이 포함되는 경향이 있다.

- `X-Forwarded-For` 헤더와 같이 보안을 수행하기 위해 HTTP 헤더에 잘못된 신뢰를 하고 있을 때
- 웹 소켓 메시지가 처리되는 세션 컨텍스트는 일반적으로 handshake 메시지의 세션 컨텍스트에 의해 결정되기 때문에, 세션 처리 메커니즘의 결함이 존재할 때
- 애플리케이션에서 사용되는 사용자가 조작할 수 있는 HTTP 헤더가 존재할 때

<br><br>

## 🚩Lab: Manipulating the WebSocket handshake to exploit vulnerabilities

![image](https://github.com/user-attachments/assets/429b79e0-2567-4034-94c2-3f63e1fa81af)

이전 랩과 같이 채팅 메시지의 패킷 속 `contents` 필드에 `img` 태그를 포함시켜 전달하면 공격을 감지하고 연결이 끊긴다. 그 뒤에 재접속 해보아도 주소가 블랙리스트 처리되었다는 문구와 함께 채팅 서비스를 이용할 수 없는 것으로 보인다.

<br>

IP를 블랙리스트에 등록했으므로 이를 우회하기 위해 라이브 챗 요청 패킷 헤더에 `X-Forwarded-For: 0.0.0.0`을 추가하면 다시 라이브 챗으로 접속할 수 있다. 그러나 채팅 연결이 끊겨 더 이상 채팅을 할 수 없다. 그렇기에 Burp suite의 Proxy 탭에서 WebSockets History 탭에서 아까 연결 했던 웹 소켓 패킷을 찾아 Repeater로 전송하면 웹 소켓 연결을 다시 시도할 수 있어 채팅하는 동안 오고가는 웹 소켓 패킷을 확인할 수 있다.

<br>

![image](https://github.com/user-attachments/assets/2dcbb5ac-df9a-4c7c-823b-ced9154aec63)

이제 다시 태그와 스크립트를 전송하는데, 필터링에 걸리지 않도록 여러 방법을 시도해본다. 그러면 필터링을 우회할 수 있는 문자열을 찾을 수 있고, 성공적으로 `alert()` 함수를 실행할 수 있다.

<br><br>

# #Using cross-site WebSockets to exploit vulnerabilities

---

몇몇의 웹 소켓 보안 취약점은 공격자가 제어할 수 있는 웹 사이트로부터 크로스 도메인 웹 소켓 연결을 만들 때 발생한다. 이는 잘 알려져 있는 크로스 사이트 웹 소켓 하이재킹 공격인데, 이는 웹 소켓 handshake 시에 크로스 사이트 요청 위조(CSRF) 취약점을 이용하여 공격될 수 있다. 이 공격은 중요한 영향을 미칠 수 있는데, 공격자가 공격 대상 사용자를 대신하여 권한이 필요한 행동을 수행하거나 공격 대상 사용자가 접근할 수 있는 민감한 데이터를 획득할 수 있다.

<br>

## What is cross-site WebSocker hijacking?

크로스 사이트 웹 소켓 하이재킹, 소위 크로스 오리진 웹 소켓 은 웹 소켓 handshake 시에 크로스 사이트 요청 위조(CSRF) 취약점을 발생시킨다. 이는 웹 소켓 handshake 요청이 세션 처리를 위해 오로지 HTTP 쿠키에만 의존하고 CSRF 토큰이나 기타 예측할 수 없는 값을 포함하는 경우에 발생한다.

공격자는 취약한 애플리케이션으로의 크로스 사이트 웹 소켓 연결한 그들의 도메인에 악의적인 웹 페이지를 만들 수 있다. 이 애플리케이션은 공격 대상 사용자의 애플리케이션 세션 컨텍스트에서 연결을 처리한다.

그러면 공격자의 페이지는 임의의 메시지를 해당 연결을 통해 서버로 보낼 수 있고 서버로부터 받는 메시지의 내용을 읽을 수 있다. 이말인즉슨, 보통의 CSRF와는 다르게 공격자는 손상된 애플리케이션과 양방향 상호 작용을 얻는다.

<br>

## What is the impact of cross-site WebSocket hijacking?

성공적인 크로스 사이트 웹 소켓 하이재킹 공격은 다음과 같은 동작을 공격자가 수행할 수 있도록 한다.

- 피해자 사용자를 사칭하여 무단 행위를 수행. 일반적인 CSRF로 공격자는 임의의 메시지를 서버 사이드 애플리케이션에 전송할 수 있다. 만약 애플리케이션이 클라이언트에서 생성되는 웹 소켓 메시지를 어떠한 민감한 동작이든지 간에 사용 한다면, 공격자는 도메인 간 적절한 메시지를 생성하고 해당 동작을 트리거 할 수 있다.

- 사용자가 접근할 수 있는 민감한 데이터 조회. 일반적인 CSRF와는 다르게 크로스 사이트 웹 소켓 하이재킹은 공격자가 하이재킹된 웹 소켓을 통해 취약한 애플리케이션과 양방향 상호 작용을 제공한다. 만약 애플리케이션이 서버에서 생성된 웹 소켓 메시지를 어떠한 민감한 데이터든 사용자에게 반환하기 위해 사용한다면, 공격자는 그 메시지들을 인터셉트하고 피해자 사용자의 데이터를 획득할 수 있다.

<br>

## Performing a cross-site WebSocket hijacking attack

크로스 사이트 웹 소켓 하이재킹 공격은 근본적으로 웹 소켓 handshake 시 발생하는 CSRF 취약점이기 때문에, 공격의 첫 번째 스텝은 애플리케이션을 수행하는 웹 소켓 handshake를 검토하고 CSRF로부터 보호되고 있는지 여부를 확인하는 것이다.

CSRF 공격을 위한 일반적인 조건 측면에서, 공격자는 전형적으로 세션 관리를 위해 HTTP 쿠키만에만 의존하고 어떠한 토큰이나 예측 불가능한 값을 요청 파라미터에 포함하지 않는 handshake 메시지를 찾아야 한다.

<br>

```
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

예를 들어, 위와 같은 웹 소켓 handshake 요청은 세션 토큰이 쿠키로 전송되기 때문에 CSRF에 취약할 수 있다.

> `Sec-WebSocket-Key` 헤더는 프록시 캐싱 오류를 방지하기 위한 랜덤한 값이 포함되어 있으며 인증이나 세션 관리를 목적으로 사용되지 않는다.
{: .prompt-info}

<br>

만약 웹 소켓 handshake 요청이 CSRF에 취약하다면, 공격자의 웹 페이지는 취약한 사이트에서 웹 소켓을 오픈할 수 있도록 크로스 사이트 요청을 수행할 수 있다. 그 다음으로는 전적으로 애플리케이션의 로직과 웹 소켓을 사용하는 방식에 따라 달라진다. 해당 공격은 다음과 같은 동작을 포함할 수 있다.

- 피해자 사용자를 대신하여 허가 받지 않는 동작을 수행하도록 웹 소켓 메시지를 보낼 수 있다.
- 민감한 데이터를 조회하기 위해 웹 소켓 메시지를 보낼 수 있다.
- 때로는 민감한 데이터를 포함한 메시지를 받기 위해 그저 기다릴 수도 있다.

<br>

## 🚩Lab: Cross-site WebSocket hijacking

(Burp collaborator를 사용해야 하는 문제)

<br><br>

# #How to secure a WebSocket connection

웹 소켓으로 발생하는 보안 취약점의 위험을 줄이기 위해서는 다음과 같은 가이드라인을 따라야 한다.

- `wss://` 프로토콜을 사용한다. (TLS을 이용한 웹 소켓)
- 웹 소켓 엔드 포인트의 URL을 하드 코딩하고 확실히 이 URL이 사용자가 제어할 수 없도록 한다.
- 크로스 사이트 웹 소켓 하이재킹을 피하기 위해 CSRF로부터 웹 소켓 handshake 메시지를 보호한다.
- 웹 소켓을 통해 받는 데이터를 어느 방향이든지 신뢰하지 않는 것으로 간주한다. SQL 인젝션과 크로스 사이트 스크립트와 같이 입력 기반의 취약점으로 부터 보호하기 위해 데이터를 서버와 클라이언트 단에서 모두 안전하게 제어한다.

<br>

![image](https://github.com/user-attachments/assets/cd5285f6-d166-4264-b2e6-efe70d1c9f8e)
