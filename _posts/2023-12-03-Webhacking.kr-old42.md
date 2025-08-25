---
title: '[Wargame] Webhacking.kr old-42 (Base64)'
date: 2023-12-03 00:00:00
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/8d91e02b-c0d2-4d7f-a697-722c81b96d9f)

문제 페이지에 접속하면 위와 같은 표와 함께 해당 파일들을 다운로드 받을 수 있는 듯한 링크를 확인할 수 있다. 하나씩 링크를 클릭해보면, `test.txt` 파일은 정상적으로 다운로드 받아지는 반면에, `flag.docx` 파일은 Access Denied 창을 확인할 수 있다.

<br><br>


## 🚩 문제 풀이

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/effc87c4-9415-40c0-8f57-4fcac6cfbc92)

문제 페이지에 대한 요청 후 응답 패킷을 확인해본 결과, 위의 이미지처럼 각 링크를 다운받을 수 있는 url 값을 확인할 수 있었다.

<br>

```
?down=dGVzdC50eHQ=
```

위의 문자열이 `test.txt` 파일을 다운 받을 수 있는 경로인데, `down` 파라미터 값이 `=`로 끝나는 것으로 보아 **base64**로 인코딩된 값임을 유추할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/f0729e75-f125-4458-9bfd-20fcc3dea172)

문자열을 디코딩하면 위의 이미지와 같이 `test.txt` 문자열을 확인할 수 있다. 그 말인즉슨, 다운받고자 하는 파일명을 base64로 인코딩한 값을 다운받는 파라미터의 값으로 사용하고 있음을 유추할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ab0d39df-8b76-498c-9a2b-7e10dd4aee87)

이제 `flag.docx` 값을 base64로 인코딩하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/04d8f554-bc44-4604-8d13-894a7e830ef1)

그 다음으로 인코딩한 값을 이용하여 url을 구성하고 접속하면, docx 파일을 다운로드 받을 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b6c71e5f-4852-4147-98de-7d7a3c6f38b5)

해당 파일을 열어보면 Flag 값을 확인할 수 있다.
