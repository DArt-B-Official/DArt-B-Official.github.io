---
title: '[Wargame] Webhacking.kr old-31 (nc)'
date: 2024-03-03 00:00:00
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

```
$port = rand(10000,10100);
$socket = fsockopen($_GET['server'],$port,$errno,$errstr,3) or die("error : {$errstr}");

Warning: fsockopen(): unable to connect to {공인 ip 주소}:10029 (Connection timed out) in /var/www/html/challenge/web-16/index.php on line 23
error : Connection timed out
```

문제 페이지에 들어가면 위의 문구를 확인할 수 있다.

<br>

위의 두 줄은 php로 작성된 코드로, 10000부터 10100까지의 랜덤한 수를 port로 하고 server 인자 값의 ip 주소와 소켓 통신을 하는 구문이다. 그러나 해당 ip 주소는 내가 접속한 컴퓨터의 공인 ip 주소이기에 10000번부터 10100번까지의 포트가 열려있지 않을 것이기에 통신할 수 없을 것이다. 따라서 해당 ip를 사용하는 컴퓨터의 10000번부터 10100까지의 포트를 열어두면 문제를 해결할 수 있을 것이다.

<br><br>


## 🚩 문제 풀이

---

이 문제는 두 가지 방법으로 풀이할 수 있다. 첫 번째는 **포트포워딩**을 이용하는 것이고, 두 번째는 **개인 서버를 이용**하는 것이다.

<br>

### 1. 포트포워딩

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/1f6f3451-878b-4a9b-ba2b-561191c46752)

공유기 설정 페이지로 이동하여 위와 같이 포트포워딩을 설정하는 곳을 찾는다. 서버에서는 랜덤하게 10000번에서 10100번의 포트를 무작위하게 접속 시도할 예정이기에 해당하는 모든 포트로 들어오는 접속을 내부포트 10000번으로 포워딩할 수 있도록 설정한다. 여기서 IP 주소는 <u>cmd > ipconfig > 이더넷 어댑터 이더넷 > IPv4 주소</u>에서 확인할 수 있다.

<br>

이제 내부에서 10000번 포트를 서비스해야 한다. 이를 위해서 [윈도우용 netcat(nc)](https://eternallybored.org/misc/netcat/) 프로그램을 설치하자.

<br>

```bash
nc64.exe -lvp 10000
```

터미널 창에서 netcat이 설치된 경로에서 위의 명령어를 실행하여 10000번 포트를 열어보자. 여기서 -l 옵션은 리스닝, -v 옵션은 자세한 정보 출력, -p 옵션은 포트 지정이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/926cf0ed-d59e-4773-b1bb-0dd724ae714b)

마지막으로 문제 페이지를 재접속하면 해당 ip 주소의 랜덤한 포트로 소켓 통신을 시도할 것이고, nc로 열어둔 10000번 포트가 통신을 진행하게 되어 Flag 값을 얻게 될 것이다.

<br>
<br>

### 2. 개인 서버 이용

aws와 같은 개인 클라우드 서버가 있다면 공유기를 포트 포워딩하는 방식보다 훨씬 쉽게 문제를 해결할 수 있다. 나는 aws의 ec2를 이용하고 있어 손쉽게 문제를 해결할 수 있었다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7dc01376-4152-42eb-a8b7-059757f7264b)

우선, 해당 클라우드의 보안 규칙을 수정하여 10000번 포트부터 10100번 포트까지 사용할 수 있도록 하자.

<br>

```bash
for i in {10000..10100}; do (nc -kl $i &); done
```

그 다음으로 위와 같이 nc와 쉘 코드를 작성하여 100개의 포트를 모두 listen 하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a12583c7-09bb-43e8-b8d6-85b0eb8b7a88)

이제 문제 페이지를 재접속하여 소켓 통신을 시도하면 위와 같이 Flag를 획득할 수 있다.
