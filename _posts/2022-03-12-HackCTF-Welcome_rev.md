---
title: "[HackCTF] Welcome_rev"
date: 2022-03-12 00:00:00
categories: [Wargame, HackCTF]
tags: [hackctf, wargame, reversing]
---


# 🚩 문제 정보

---

![image](https://user-images.githubusercontent.com/37824335/223177017-22db356c-3b32-4202-ad99-0c243f877294.png)

**\#HackCTF \#Reversing \#Welcome_REV \#50pts**

<br />


# 🚩 문제 풀이

---

## 👁‍🗨 문제 파악

![image](https://user-images.githubusercontent.com/37824335/223177254-1468817d-2ea8-45d9-b0ae-5ea79ad367e6.png){: w="300"}

주어진 파일은 파일 형식을 알 수 없는 파일이었다. 이 파일을 분석하여 flag를 찾아내는 것으로 파악하였다.

<br />

## 👁‍🗨 풀이 시도

우선 주어진 파일을 IDA에서 열어보았다

<br>

![image](https://user-images.githubusercontent.com/37824335/223177392-2f4d7feb-b4bd-4331-b748-2831546f2eee.png){: w="700"}

좌측의 *function name*탭에서 **main**함수가 보였고 그래프 형태로 함수를 보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223177434-4fc5c962-eb89-430f-8b05-2f6d571daf62.png){: w="700"}

위와 같은 형태였는데 분석해보니 사용자에게 패스워드를 입력을 받은 후 **check_password** 함수를 실행하여, 패스워드가 맞다면 **"Congrats, now where's my flag?"** 패스워드가 틀리면 **"Incorrect Password!"**의 문자열을 출력(**_puts**)하는 형식이었다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223178006-28e0a92b-0676-49b9-aa8a-0f7ca2ff9733.png)

그런데 패스워드가 맞을 경우에 flag를 출력하는 동작이 보이질 않았다. 그래서 패스워드를 확인하는 함수로 추정되는 **check_password** 함수에서 사용자의 입력과 비교하는 구문을 찾아보기로 하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223178084-34af7e84-939e-4630-9577-cc2e3c5b0bdd.png)

**check_password** 함수의 정의 부분으로 이동하여 동작 흐름을 따라가니

<br>

![image](https://user-images.githubusercontent.com/37824335/223178132-7cd3be9d-a7bd-47cd-b6ff-37008b440455.png)

긴 문자열을 **_strncmp** 함수에서 사용하고 있음을 발견하였다. 해당 문자열을 따라가 디스어셈블리어로 확인하니

<br>

![image](https://user-images.githubusercontent.com/37824335/223178195-13ac0e3d-2c2a-44ea-8f6f-a23dfc142c53.png)

`.rodata` 영역에 문자열들이 그대로 나열되어 있음을 확인할 수 있었고 따라온 긴 문자는 '**=**' 문자가 뒤에 삽입된 형태로 존재함을 확인할 수 있었다. '='로 문자열이 끝남을 미루어보아 해당 문자열은 **해시(Hash)**임을 짐작할 수 있었다.

온라인 해시 변환 사이트에서 해당 문자열을 변환 시도하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223178348-fef1e5f8-d19f-41f6-bd30-5b10560dc5db.png)

flag 형태로 복호화 되었다!

<br>

![image](https://user-images.githubusercontent.com/37824335/223178418-4426b046-cded-4647-8bba-7fc84f4b13ef.png){: w="400" }
