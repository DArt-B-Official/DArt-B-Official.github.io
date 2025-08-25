---
title: '[Wargame] Webhacking.kr old-47 (SMTP Injection)'
date: 2024-03-02 00:00:00
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a417e0db-f0c4-41cb-9a6d-d11437da339b)

문제 페이지에서 Mail Subject를 입력할 수 있는 폼이 주어지고, send를 통해 메일을 발송할 수 있다.

해당 기능 외에는 소스코드나 요청 응답 패킷에서 확인할 수 있는 다른 취약점은 보이지 않았다. 메일의 subject를 조작할 수 있는 것으로 보아 **SMTP Header Injection**을 이용하는 문제로 보인다.

<br>

> **SMTP Header Inejction**
<br>
SMTP Header Injection은 적절한 검사 없이 사용자 입력이 메일 헤더에 위치될 때 발생하며, 이로 공격자가 임의의 값을 포함한 추가적인 헤더를 삽입할 수 있다. 이는 메일의 사본을 제 3자에게 전송하고, 바이러스를 첨부하고, 피싱 공격을 전달하고, 메일의 내용을 변경하는 데에 악용될 수 있다. 이 문제는 메일에 비밀번호 재설정 토큰과 같이 공격자에게 제공되지 않는 민감한 정보가 포함된 경우에 특히 심각하다.
{: .prompt-tip}

<br>

이를 이용하여 Flag 값을 포함한 메일을 나의 메일로 보낼 수 있도록 헤더를 삽입하면 된다.

<br><br>

## 🚩 문제 풀이

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/2d085b7f-5b24-41cd-a686-cc09cc7982a6)

우선, 메일을 보낼 때의 Header를 확인해보자. 구글 메일에서 원본을 볼 수 있는데 위와 같은 헤더를 확인할 수 있다.
**To** 필드에 수신자 항목이 들어가고, 우리가 입력할 수 있는 메일의 제목은 **Subject** 항목에 들어가는 것을 확인할 수 있다. 여기서 <u>각 필드는 줄바꿈으로 구분하고 있음</u>을 알 수 있다.

<br>

그렇다면 추가적인 수신자를 지정할 수 있는 방법을 찾아야한다. 메일을 보낼 때 *참조*와 *숨은참조*를 포함할 수 있는데, 이는 메일의 수신자 이외에도 다른 사람에게 메일을 보낼 수 있는 또다른 방법이다.

메일에서 참조와 숨은참조를 추가하여 보낸 메일의 헤더를 확인해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/47a8cc01-7383-430a-a111-a1a18604c833)

위와 같이 참조는 **CC**라는 필드로, 숨은참조는 **BCC**라는 필드로 지정된 것을 확인할 수 있다. 이제 둘 중 하나의 필드를 삽입하여 나의 메일로 받을 수 있도록 하면 된다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5e5b268a-dc8c-4083-9cfb-05e18490a608)

위에서 Header의 필드는 줄바꿈을 통해 구분하는 것을 알게 되었으므로 Subject 입력 태그를 **input**에서 **textarea**로 변경하여 줄바꿈 문자를 인식하도록 한다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/d79f2f6b-e081-4e24-9583-8dac7ff2fb68)

그 다음으로 Subject와 Cc(혹은 Bcc) 필드를 추가하여 메일을 전송하면

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/214eaf20-020c-41a5-990c-34bfe76d5e68)

Flag를 확인할 수 있다.
