---
title: '[Wargame] Webhacking.kr Write-Ups'
date: 2023-02-06 00:00:00
categories: [Wargame-Up, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: true
---



## 🚩 old-01

![image](https://user-images.githubusercontent.com/37824335/227117786-19627989-fd23-4052-bfee-f9f956ccdb3e.png)

문제 페이지로 이동하면 위와 같은 화면을 볼 수 있다. view-source를 눌러 코드를 확인해보자.

<br>

```php
<?php
  include "../../config.php";
  if($_GET['view-source'] == 1){ view_source(); }
  if(!$_COOKIE['user_lv']){
    SetCookie("user_lv","1",time()+86400*30,"/challenge/web-01/");
    echo("<meta http-equiv=refresh content=0>");
  }
?>
```

상단 코드에서 중요한 부분은 `user_lv` 이름의 쿠키 값을 설정하는 부분인 것 같다.

<br>

```php
<?php
  if(!is_numeric($_COOKIE['user_lv'])) $_COOKIE['user_lv']=1;
  if($_COOKIE['user_lv']>=4) $_COOKIE['user_lv']=1;
  if($_COOKIE['user_lv']>3) solve(1);
  echo "<br>level : {$_COOKIE['user_lv']}";
?>
```

하단 코드에서 문제를 풀 수 있는 힌트를 얻었다. `user_lv` 명의 쿠키 값이 3 초과 4 미만이어야 문제를 풀 수 있음을 확인하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227118033-515bd877-0e13-47de-9607-97e977549fe9.png){: w="300"}


메인 페이지로 이동하여 쿠키 값을 해당 범위의 수에 속하는 3.5로 수정하면

<br>

![image](https://user-images.githubusercontent.com/37824335/227118145-300cef3b-a8e6-4e1a-8ee8-966fe320e737.png)

풀린다!

<br>

---

## 🚩 old-06

![image](https://user-images.githubusercontent.com/37824335/227118210-dbd8c09c-39ab-450d-b61c-235cf38681e5.png)

문제 페이지에 들어가면 위와 같은 페이지가 보인다. view-source를 클릭해보니 php코드를 볼 수 있었다. 핵심 코드만을 보자.

<br>

![image](https://user-images.githubusercontent.com/37824335/227118267-e76ed68f-3b11-4959-9a6d-2a4478488b1e.png){: w="600"}

`user`명의 쿠키 값이 존재하는 경우에 위의 코드를 실행하는 것으로 보인다. `guest`와 `123qwe` 문자열을 각각 id와 pw값으로 설정한 후 해당 문자열을 20번 반복하여 **base64** 인코딩한 다음에 인코딩한 각 문자열에서 1, 2, 3, 4, 5, 6, 7, 8 문자를 각각의 문자로 치환한다. 그렇게 나온 결과 id, pw 문자열을 쿠키로 설정하는 동작이다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227118340-fe19aba8-7e24-46c0-9c86-387d26435186.png){: w="500"}

해당 웹페이지의 쿠키를 살펴보니 base64로 인코딩 된 것으로 보이는 문자열이 value 값으로 설정되어 있었다. HTML 코드 하단의 php 코드를 살펴보자.

<br>

![image](https://user-images.githubusercontent.com/37824335/227118428-b3e2381b-c5dd-45bf-b9ce-84f79918a880.png){: w="600"}

코드에서 필요없는 부분은 지웠다. 해당 코드는 `user`, `password` name의 쿠키 값을 가져와 위에서 진행한 문자열 치환의 역을 수행하고, base64 문자열을 20번 반복 decode하여 결과 문자열이 id와 passwword가 각각 `admin`과 `nimda`라는 문자열과 일치하면 문제가 풀리는 듯하다. 그럼 우리는 `admin`과 `nimda`를 역으로 치환하고, base64 decode를 20번 수행하여 나온 문자열을 쿠키로 각각 설정하면 문제가 풀리는 것을 알 수 있다. 단순 반복 과정이니 Python으로 코드를 짜보자.

<br />

```python
import base64
```
base64 인코딩과 디코딩 과정이 필요하므로 python 내장 모듈 **base64**를 사용하자.

<br>

![image](https://user-images.githubusercontent.com/37824335/227119124-a3a9a5de-c9e1-4372-bb8b-2bf09d5e73fe.png)

역으로 치환하는 과정은 함수로 구현하여 문자열 id와 pw를 역으로 치환한 문자열을 쉽게 구해낼 수 있었다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227119141-00fc28ca-1ed1-4237-a9cf-7e9baaf7030f.png)

그리고 각각 위에서 설명하였던 과정대로 코드를 진행하고 출력한 값을 쿠키로 설정하니

<br>

![image](https://user-images.githubusercontent.com/37824335/227119180-0b80c45b-ce97-4437-b94a-e75939cecf34.png)

풀렸다!

<br>

---

## 🚩 old-10

![image](https://user-images.githubusercontent.com/37824335/227119274-751be999-20f4-4cf3-9b97-20f837490425.png)

문제 페이지에 들어가니 트랙을 연상하게 하는 그림이 있었다. 소스코드를 먼저 살펴보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227119312-fb3603a0-28bf-45b6-bc13-303ff1436ce5.png)

다른 부분은 화면을 그리는 태그들이 존재하였고, 트랙 위의 **O**를 그리는 태그는 `onclick`, `onmouseover`, `onmouseout` 속성을 가지고 있었다.
`onmouseover`과 `onmouseout` 속성은 **O** 글자와 **yOu** 글자를 변경하는 역할을 하였고, `onclick` 속성은 클릭 시에 1px씩 글자를 오른쪽으로 옮기는 역할을 하고 있었다. 클릭해보면

<br>

![image](https://user-images.githubusercontent.com/37824335/227119343-5bb739ee-6ac7-4bf9-b39b-71b601b5cbdd.png)

조금씩 이동함을 알 수 있었는데, `onclick` 속성에서 해당 글자 스타일 속성의 left 값이 1600px이 되면 특정 페이지로 이동함을 알 수 있었다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227119651-3d667c9f-8cdc-4f1e-8606-6d0bb03605de.png)

그래서 해당 태그의 left 값을 1599px로 설정하고 글자를 한 번 클릭하니 1600px이 되면서

<br>

![image](https://user-images.githubusercontent.com/37824335/227119697-86c54244-dbdd-4ead-8301-4df11ef7c609.png)

문제가 풀렸다!

<br>

---

## 🚩 old-11

![image](https://user-images.githubusercontent.com/37824335/227119817-0b60a882-dae4-4c70-8375-77e7cb5b43b1.png)

문제 페이지에 들어가면 위와 같은 화면을 볼 수 있다. view-source를 눌러 코드를 확인해보자.

<br>

```php
<?php
  $pat="/[1-3][a-f]{5}_.*$_SERVER[REMOTE_ADDR].*\tp\ta\ts\ts/";
  if(preg_match($pat,$_GET['val'])){
    solve(11);
  }
  else echo("<h2>Wrong</h2>");
  echo("<br><br>");
?>
```

해당 부분이 중요한 로직인데, `$pat` 변수에 정규식 문자열을 할당하고 해당 정규식과 `val` 명의 인자 값을 비교하여 정규식과 매칭하는지 검사한다. 정규식을 분석해보면

- **[1-3]** : 1-3 범위의 숫자
- **[a-f]{5}** : a-f 범위의 문자 - 최소 5개
- **_** : _ 문자 하나
- **.*** : 어떤 문자열이든 zero or more
- **$_SERVER[REMOTE_ADDR]** : 접속한 클라이언트의 IP 주소를 나타내는 PHP 환경변수
- **\t** : tab 문자
- **p** : p 문자
- **a** : a 문자
- **s** : s 문자

이를 만족시키는 문자열을 `val` 인자로 전달해야 한다. 정규식을 충족시키는 문자열을 작성해보면,
`1aaaaa_[ip주소]%09p%09a%09s%09s`
가 될 수 있다. 여기서 %09는 tab 문자를 url encode 방식으로 나타낸  것이다.

`https://webhacking.kr/challenge/code-2/?val=1aaaaa_[ip주소]%09p%09a%09s%09s`
이를 주소창에 인자로 입력하여 GET 방식으로 넘겨주면,

<br>

![image](https://user-images.githubusercontent.com/37824335/227119895-001bc291-9682-4cb0-9693-d62ffaaa2f17.png)

풀린다!

<br>

---

## 🚩 old-14

![image](https://user-images.githubusercontent.com/37824335/227119952-af1e09d2-1cee-44da-9b4c-5c71a3d5164f.png)

문제 페이지에 들어가보니 input 창과 check 버튼만이 놓여있었다. 소스코드 먼저 확인해보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227119993-0945b50e-4099-4e80-9a63-eefbeb77d766.png)

form 태그 내에 input 태그 두 개가 있었고 그 중 버튼에 `ck()` 함수가 onClilck 속성으로 지정되어 있었다. 스크립트 구문에 `ck()` 함수의 내용이 있었고, 셋 째줄까지는 대략 브라우저의 URL의 ".kr" 문자열의 인덱스 값에 30을 곱한 값을 `ul` 변수에 저장하고 있었다. 이를 입력해야 하는 것으로 보였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120026-5002bcf2-2525-4cfd-bc0c-9666510c45b5.png){: w="300"}

`ul` 값을 알아내기 위해 콘솔 창에서 구문을 실행하여 나온 숫자 값을 입력하니

![](https://velog.velcdn.com/images/1unaram/post/f9161049-bf77-4bb0-86f8-61ba08cfad33/image.png)

풀렸다!

<br>

---

## 🚩 old-15

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/1c9cea64-b076-4c21-9197-045a05803772)

문제 페이지에 들어가면 위와 같은 문구와 함께 사이트에 접근할 수 없고 확인 버튼을 누르면 webhacking.kr 메인 페이지로 이동한다.

burp suite를 이용해 접속할 때의 패킷을 잡아보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/f9ffbc0d-1709-4f83-806a-767276aa7549)

요청에 따라 응답 패킷을 확인하면 alert 함수가 실행되고 이후에 `location.href='/';` 구문에 따라 메인 페이지로 이동하는 것이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ea8e153a-105f-4aea-b8b6-bd9d6c2acc01)

그러나 플래그를 얻을 수 있는 페이지로 이동하는 a 태그가 이후에 삽입되므로, 위처럼 메인 페이지로 이동하는 구문을 주석처리하여 응답 패킷을 받으면

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5a1808f8-5ba8-4843-9d05-989cb773d3db)

a 태그를 확인할 수 있고, 이를 누르면 플래그를 획득하여 문제를 풀 수 있다.

<br>

---

## 🚩 old-16

![image](https://user-images.githubusercontent.com/37824335/227120196-634574ca-75bb-46ef-8dc6-19c08a75130e.png)

문제 페이지에 들어가니 색을 가진 아스타리스크(*)가 보였다. 소스코드부터 보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120237-91609ee6-8280-4d19-a255-0d383f3212d0.png)

body 태그에는 font 태그 4개 사이에 script문이 들어있었다. body 태그에는 키보드 이벤트 발생 시에 keyCode 값을 `mv()` 함수의 인자로 전달하고 있다. `mv()` 함수 내에서는 `kk()` 함수를 호출하고 있는데 이 함수는 생성된 난수로 6자리 자연수를 만들어 해당 숫자에 해당하는 색상 코드의 색을 가진 *를 생성하고 있다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120279-2bfa72df-2102-44e4-80e5-603f26650680.png)

키보드를 막 누르면 이런 화면을 볼 수 있다. 이 동작은 중요하지 않고, `mv()` 함수 내의 마지막 `if`문은 keyCode 값이 124와 같으면 php 동작을 하는 방식이다. ASCII code 124번은 `|` 문자를 의미하기에 키보드로 입력하면

<br>

![image](https://user-images.githubusercontent.com/37824335/227120330-510ef5ff-d517-4429-b78a-d3c258fd75aa.png)

풀렸다 !

<br>

---

## 🚩 old-17

![image](https://user-images.githubusercontent.com/37824335/227120443-578ea08a-9a20-4303-810c-783d0fb6d0c7.png)

문제 페이지에 들어가니 14번과 마찬가지로 입력 창과 check 버튼이 있었다. 소스코드를 먼저 보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120473-1b92567a-8a8b-43cc-acf2-02b6546615cd.png)

14번과 유사한 문제임을 알 수 있었고 마찬가지로 콘솔 창에서 `unlock` 변수를 계산 해보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120500-bb21092f-c5ba-451a-8146-b66a6aa093fb.png)

연산하여 나온 결과를 input에 입력하고 버튼을 클릭하면

<br>

![image](https://user-images.githubusercontent.com/37824335/227120520-cc52d694-79d5-4b9c-8e22-3d81e9617bf7.png)

풀렸다!

<br>

---


## 🚩 old-18

![image](https://user-images.githubusercontent.com/37824335/227120579-af1ad117-8511-49d7-b205-62b32059722b.png)

문제 페이지에 들어가면 위와 같은 화면이 나온다. 제목에 크게 쓰여있듯이 SQL Injection을 사용하는 문제인 것 같다. view-source를 눌러보자.

<br>

![image](https://user-images.githubusercontent.com/37824335/227120688-d99f3c09-9319-4e94-a175-9db87216d1fd.png)

홈페이지를 구성하는 PHP 파일이 있다. 그 중 중요한 php 코드만을 보자.
GET 방식으로 `no` 파라미터를 받아온다. 그 다음 `preg_match()` 함수를 통해 받아온 파라미터 값에서 정규표현식으로 매칭되는 문자열이 있는지 검사하여 있다면, 종료하는 형식이다. 이를 우회하여 **SQL Injection** 공격 기법으로 데이터베이스서 값을 꺼내어 `result` 변수에 "admin" 문자열을 집어 넣으면 풀리는 문제이다. 문자열 우회를 위해 정규표현식을 분석해보자.

<br>

> 정규표현식 : `/ |\/|\(|\)|\||&|select|from|0x/i`

- **/ /i** : case insensitive 방식의 정규표현식이다. select와 from은 사용하지 못할 것 같다.
- **\|** : or을 기준으로 문자를 끊을 수 있다. 하나씩 끊어서 보자.
- **공백** : `공백` 불가
- **/** : `/` 불가
- **\(** : `(` 불가
- **\)** : `)` 불가
- **\|** : `|` 불가
- **&** : `&` 불가
- **select** : `select` 문자열 불가(대문자도 불가)
- **from** : `from` 문자열 불가(대문자도 불가)
- **0x** : `0x` 문자열 불가

<br />

이들을 사용하지 않고 아래와 같이 삽입문을 작성할 수 있다.
`0%09or%09id='admin'%09and%09no=2`

- **%09** : `Tab` 문자이다. 공백 대신 사용할 수 있다.
- **or** : or 문으로 id 값은 `admin`, no 값은 `2`인 데이터를 분별할 수 있다.

<br>

🔽 삽입된 SQL문
```sql
 select id from chall18 where id='guest' and no=0 or id='admin' and no=2
```

삽입문을 URL을 통해 요청하면,

<br>

![image](https://user-images.githubusercontent.com/37824335/227121058-235c44f3-282b-406b-81f3-49bab7f4fa73.png){: w="500"}

풀렸다!

<br>

---

## 🚩 old-19

문제 페이지에 접속하면 id 입력 값에 'admin'이라는 문자열이 채워져있고 제출할 수 있는 폼을 확인할 수 있다. admin으로 로그인 시도하면 admin이 아니라는 문구를 확인할 수 있고, 다른 id로 로그인을 시도하면 해당 아이디로 로그인할 수 있고, cookie 값이 설정되는 것을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/377f60b4-bed3-4a99-a3f6-b0f06f8cff23)

데이터베이스에서 id를 조회하는 시스템일 수도 있다는 생각에 SQL injection을 시도하였다. 입력 값 길이 제한이 있어 요청 패킷을 수정하여 전송하였다. 문자열은 `admin'+or+1=1--`로 설정하여 전송하였는데 위와 같이 로그인이 된 것을 확인할 수 있었다. 이때, `--` 문자가 사라진 것으로 보아 주석으로 인식하는 듯 하였다.

<br>

그래서 입력 값을 `admin'--`으로 바꾸어 로그인을 시도하자, 문제가 해결되었다.

<br>

---

## 🚩 old-20

![image](https://user-images.githubusercontent.com/37824335/227121197-944b0f99-3dbd-4e44-95ee-18f904641bc0.png)

문제 페이지에서는 위와 같은 구성이었고 우선 소스코드를 보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227121255-bf2f7a4d-4f1c-453d-90eb-5d1118bd6130.png)

input 창들은 form 태그 내에 위치했고 submit 버튼을 누르면 `ck()`함수가 실행되는 형태였다. 함수의 내용을 보아 nickname, comment, captcha에 값을 입력해야 하며 랜덤으로 생성되는 captcha 문자를 알맞게 입력해야 form의 submit이 작동하는듯 했다. 2초 내에 제출하기 위해서는  python의 **selenium** 모듈을 이용하였다.

<br>

🔽 **import**
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager
from time import sleep
```

![image](https://user-images.githubusercontent.com/37824335/227121319-c8cc4031-e19a-4b1d-ac00-1fff121909fc.png)

`driver.Chrome()` 함수는 무슨 이유인지 자꾸 오류가 나서 ChromeDriverManager를 이용하여 크롬 드라이버를 셀레니움 드라이버로 설정하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227121347-67d6349d-242c-4590-9f47-b0d8f26420c7.png)

webhacking.kr 사이트는 로그인 상태에서 문제 사이트에 접근할 수 있기 때문에 사용자에게 id, password를 입력 받고

<br>

![image](https://user-images.githubusercontent.com/37824335/227121379-9899ebb0-88c4-4f44-9ea5-02cf20e17385.png)

로그인 창의 각 태그의 XPATH를 복사하여 id, password를 입력하고 로그인을 하도록 하였다. 처음에는 `find_element_by_xpath()` 함수를 이용하였으나 DeprecationWarning 문구가 출력되어 `find_element()` 함수를 이용하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227121673-73765c79-4fac-4b8a-845a-a8f97bdc694d.png)

로그인 세션이 유지된 상태에서 문제 페이지로 이동

<br>

![image](https://user-images.githubusercontent.com/37824335/227121723-3e4a93ca-9ae4-4275-827d-cb4c256e2999.png)

문제 페이지로 이동 후 nickname과 comment에는 아무 문자를 채워넣고, captcha의 value 속성 값을 가져와 captcha로 입력하고, submit 버튼을 클릭하도록 하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227121802-63f64b35-ed74-4444-bb4b-cd2bc827b524.png){: w="400"}

실행 시키니 풀렸다 !

<br>

---

## 🚩 old-24

문제 페이지에 접속하면 클라이언트의 ip와 agent 정보와 소스보기 링크를 확인할 수 있다. "Wrong IP!" 문구를 보아 IP를 조작해야함을 알 수 있다. 소스코드를 확인하자.

<br>

```php
// $_SERVER, $_COOKIE 배열 속의 키 값을 변수화
extract($_SERVER);
extract($_COOKIE);

// 변수 값 저장
$ip = $REMOTE_ADDR;
$agent = $HTTP_USER_AGENT;

if($REMOTE_ADDR){
  // $REMOTE_ADDR 문자열 필터링
  $ip = htmlspecialchars($REMOTE_ADDR);
  $ip = str_replace("..",".",$ip);
  $ip = str_replace("12","",$ip);
  $ip = str_replace("7.","",$ip);
  $ip = str_replace("0.","",$ip);
}

...(생략)...

// 필터링된 $ip 변수 문자열이 127.0.0.1과 같을 경우 문제 해결
if($ip=="127.0.0.1"){
  solve(24);
  exit();
}
else{
  echo "<hr><center>Wrong IP!</center>";
}
```

코드에서 주요한 부분은 위와 같다. 서버에 접속한 클라이언트의 정보를 담은 `$_SERVER` 배열과 `$_COOKIE` 배열의 키 값을 변수화하고 그 중 클라이언트의 ip 값을 담은 `$REMOTE_ADDR` 변수 값을 필터링한 값이 문자열 '127.0.0.1'이면 문제를 해결할 수 있다.

<br>

여기서 주목해야하는 점은 클라이언트의 ip 값을 `$_SERVER[$REMOTE_ADDR]`와 같이 배열에 직접 접근하여 값을 가져오는 것이 아니라 배열의 키 값을 변수화하였다는 점이다. 이때 `$_SERVER` 배열뿐만 아니라 `$_COOKIE` 배열도 변수화하고 있는데, `extract` 함수로 `$_COOKIE` 배열을 나중에 변수화하고 있기 때문에 `$_SERVER`의 키 값과 중복된다면 `$_COOKIE` 배열의 키 값으로 변수 값이 정해진다. php 코드를 통해 이를 확인해보았다.

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7cb08698-aafb-4cbb-aa4a-79d14e808533)

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/79003277-c96d-4041-a992-ccc60fb06acd)

<br>

그러므로 클라이언트가 요청 시에 조작할 수 없는 `$_SERVER` 배열 대신에 `$_COOKIE` 값을 이용하여 `$REMOTE_ADDR`을 전송하면 그 값을 지정할 수 있다. 따라서 필터링을 우회하여 문자열 '127.0.0.1'을 만들수 있는 문자열 `112277...00...00...1`을 요청 cookie 값에 포함하여 요청을 보내보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/d3ea8f8f-030e-479c-844a-172f31fd3072)

위와 같이 쿠키 값을 포함하여 요청을 전송하면

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ec260ae4-04b0-4797-baa6-33b78a054da4)

클라이언트 ip를 조작할 수 있다.

<br>

---

## 🚩 old-26

문제 페이지에 접속하면 페이지의 소스를 볼 수 있는 view-source 링크를 확인할 수 있다. 해당 링크로 이동해보자.

```php
<?php
  // id 파라미터로 받은 값에서 'admin'이라는 문자열과 정규식 검사를 하여 같다면 "no!"를 출력하고 프로그램을 종료한다.
  if(preg_match("/admin/",$_GET['id'])) { echo"no!"; exit(); }

  // id 파라미터로 받은 값을 urldecode 함수를 통해 디코딩하고 해당 값을 id 파라미터 값에 다시 담는다.
  $_GET['id'] = urldecode($_GET['id']);

  // id 파라미터 값이 문자열 'admin'과 같다면 문제를 해결한다.
  if($_GET['id'] == "admin"){
    solve(26);
  }
?>
```

소스코드에서 중요한 부분은 위와 같다. 우리는 정규식 검사에서 'admin' 문자열을 우회하여 id 파라미터로 넘기면 문제를 풀 것만 같다. 하지만 percent-encoding으로 url 인코딩도 해보았지만 문제는 풀리지 않았다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7f3b0c55-20de-4d38-bff2-ec50c4c2be76)

이때, `urldecode` 함수를 주목하였다. 해당 함수에 대한 정보를 php 공식 홈페이지에서 확인하였다. 여기서 하나의 정보를 얻을 수 있었는데, `$_GET`과 `$_REQUEST` 변수는 이미 디코딩 되어 있다는 것이다. 우리가 `id` 파라미터를 넘길 때 `$_GET` 변수에는 이 값이 이미 한 번 디코딩 되어 있고,  `urldecode($_GET['id'])` 구문은 디코딩된 문자열을 다시 한 번 디코딩하는 셈이 되는 것이다. 따라서 디코딩이 2번 되는 것을 확인할 수 있다.

<br>

```
admin -> %61%64%6d%69%6e
%61%64%6d%69%6e -> %25%36%31%25%36%34%25%36%64%25%36%39%25%36%65
```

따라서 우리는 `id` 파라미터 값으로 'admin' 문자열을 두 번 인코딩한 값을 넘겨주면 문제를 풀 수 있다.

---

## 🚩 old-27

![image](https://user-images.githubusercontent.com/37824335/227121891-0e54b8b6-b567-4c1c-b69f-e2a950929b72.png)

18번 문제와 같은 SQL Injection 문제이다. 마찬가지로 소스코드 내에서 중요한 부분만을 보자.

<br>

![image](https://user-images.githubusercontent.com/37824335/227121916-6dca857f-541f-468f-b392-5d9fa99d1999.png)

같은 방식으로 정규표현식을 이용하여 문자열을 검사하고 db의 데이터를 접근한다. 정규표현식을 분석해보자.

<br>

> 정규표현식 : `/#|select|\(| |limit|=|0x/i`

- **/ /i** : case insensitive 방식의 정규표현식이다. select와 from은 사용하지 못할 것 같다.
- **\|** : or을 기준으로 문자를 끊을 수 있다. 하나씩 끊어서 보자.
- **#** : `#` 불가
- **select** : `select` 문자열 불가(대문자도 불가)
- **\(** : `(` 불가
- **공백** : `공백` 불가
- **limit** : `limit` 문자열 불가(대문자도 불가)
- **=** : `=` 불가
- **0x** : `0x` 문자열 불가

<br>

18번 문제와는 다르게 괄호()가 SQL문에 들어있다. 이를 유의하여 아래와 같이 삽입문을 작성할 수 있다.

> `0)%09or%09id%09like%09'admin'%09and%09no>1%09and%09no<3%09;%00`

- **)** : 괄호를 닫아주어 구문을 분리하자.
- **%09** : `Tab` 문자이다. 공백 대신 사용할 수 있다.
- **like** : `=` 문자 대신 문자열을 비교할 수 있는 연산자이다. `id='admin'`과 같은 기능.
- **no>1 and no<3** : `=` 문자 대신 숫자 비교를 위해 부등호로 숫자 2로 제한하자.
- **;%00** : 뒤에 있는 `)` 문자를 상쇄시키기 위해 주석 역할을 하는 문자를 넣어주자.

<br>

🔽 삽입된 SQL문
```sql
select id from chall27 where id='guest' and no=(0) or id like 'admin' and no>1 and no<3 ;%00)
```

<br>

URL로 요청하면,

![image](https://user-images.githubusercontent.com/37824335/227122102-795ebd56-7932-4a2f-b420-993a4855eb5a.png)

풀린다!

<br>

---

## 🚩 old-32

![image](https://user-images.githubusercontent.com/37824335/227122221-077240f1-6718-41c2-91ed-81a145b0f263.png){: w="400"}


문제 페이지에 들어가니 1500여개의 Webhacking.kr 유저의 목록이 있었고 유저별로 hit 점수가 있었다. 유저 한 명을 클릭하니 그의 hit이 늘어난 것으로 보아 100번 클릭해야 하는 문제로 보였다. 그러나 한 번 hit한 후로는

<br>

![image](https://user-images.githubusercontent.com/37824335/227122299-0ae1ba5b-d78e-4cdf-85a9-2cde3f3bfac3.png)

다음과 같은 문구와 함께 hit를 할 수 없었다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227122345-fde90a16-5ca8-41c2-a26b-3ccc347f7e92.png)

cookie를 확인해본 결과 vote_check명의 값으로 ok가 세팅된 것을 볼 수 있었고 이를 삭제하니 hit 할 수 있었다. 100번 반복을 위해서는 자동화가 필요할 것 같아 파이썬의 **selenium** 모듈을 사용하였다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227122380-78447fb2-b013-4d0d-b9f7-d713d74f79de.png)

Webhacking.kr은 로그인을 해야 문제 페이지에 접근할 수 있기 때문에 selenium에서 로그인 하는 과정이 필요하다. 해당 과정은 *Challenge(old) - 32* 에서 다루었다. 문제 페이지로 넘어갔다.

<br>

![image](https://user-images.githubusercontent.com/37824335/227122499-77e6502a-8969-4a05-b60b-148453e53c72.png)

반복문을 통해 100번 hit를 하도록 구현하였고,

<br>

![image](https://user-images.githubusercontent.com/37824335/227122526-ac1271ca-ac22-49a3-85d5-a9624878bcc8.png){: w="400"}

hit는 테이블 태그의 `onclick` 속성으로 위와 같이 URL을 통해 GET 호출을 하고 있었고, 나의 User Name을 매개변수로 하여 호출하도록 하였다. 반복을 하며 'vote_check'명의 cookie도 중간 중간 삭제 해주었고 코드를 실행하니 풀렸다!

<br>

---

## 🚩 old-33

---

이 문제는 단계별로 문제를 풀며 그 다음 단계의 문제 페이지 경로를 알아내어 최종 문제까지 풀어내는 문제이다. 페이지의 소스를 보고 요구하는 조건을 만족시키면 다음 단계로 넘어가는 링크를 제공하도록 되어있다.

첫 번째 문제부터 확인해보자.

<br>

```php
<?php
if($_GET['get']=="hehe") echo "<a href=???>Next</a>";
else echo("Wrong");
?>
```

GET 요청 시에 파라미터 `get` 값으로 `hehe` 값을 포함시켜 요청하면 된다.

<br>

```
https://webhacking.kr/challenge/bonus-6/?get=hehe
```

위와 같이 요청하면 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php
if($_POST['post']=="hehe" && $_POST['post2']=="hehe2") echo "<a href=???>Next</a>";
else echo "Wrong";
?>

```

두 번째 문제는 바디 값으로 `post` 데이터에 `hehe`를, `post2` 데이터에 `hehe2`를 포함하여 **POST** 요청을 보내면 된다.

콘솔 도구를 통해서도 요청할 수 있지만 응답 패킷에 `form`을 삽입하여 요청 해보자.

<br>

```html
<form action="lv2.php" method="post">
  <input name="post">
  <input name="post2">
  <input type=submit>
</form>
```

위의 구문을 응답 패킷에 추가하고, 각각의 input 값으로 `hehe`와 `hehe2`를 넣어주고 제출하면 다음 문제로 넘어갈 수 있다.

<br>

```php
<?php
if($_GET['myip'] == $_SERVER['REMOTE_ADDR']) echo "<a href=???>Next</a>";
else echo "Wrong";
?>
```

세 번째 문제는 문제를 접속한 컴퓨터의 공인 IP 값을 `myip` 파라미터에 포함시켜 GET 요청을 보내면 된다.

```
https://webhacking.kr/challenge/bonus-6/33.php?myip={공인ip}
```

<br>

```php
<?php
if($_GET['password'] == md5(time())) echo "<a href=???>Next</a>";
else echo "hint : ".time();
?>
```

네 번째 문제는 php에서 `time` 함수 값을 `md5` 함수를 통해 암호화한 값을 `password` 파라미터에 포함시켜 GET 요청을 보내면 되는 문제이다.

새로고침할 때마다 hint로 제공되는 time 값의 포맷에 따라 몇 초 뒤의 시간 값을 온라인 md5 암호화 툴을 이용하여 생성된 문자열을 이용해  해당 시간 때까지 새로고침하면 문제를 풀 수 있다.

<br>

|-----|----|
|time()|md5(time())|
|1701696200|c4f184e9285c85f721c80625538d0096|

```
https://webhacking.kr/challenge/bonus-6/l4.php?password=c4f184e9285c85f721c80625538d0096
```

<br>

```php
<?php
if($_GET['imget'] && $_POST['impost'] && $_COOKIE['imcookie']) echo "<a href=???>Next</a>";
else echo "Wrong";
?>

```

다섯 번째 문제는 다음 조건을 만족시켜 요청을 보내야한다.

1. `imget` 파라미터를 포함시켜 **GET** 요청
2. `impost` 데이터를 포함하여 **POST** 요청
3. `imcookie` 쿠키 값을 포함시켜 요청

<br>

```html
<form action="md555.php" method="post">
  <input name="impost">
  <input type="submit">
</form>
```

우선 2번 조건을 위해 위와 같은 응답 패킷에 `form`을 삽입하였다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/c0eb25f2-7612-4d44-b196-162c30616032)

그 다음으로 input 값에 아무 데이터를 삽입하고, 요청을 보내는 패킷에서 URL 부분에 `imget` 파라미터 값을 포함하였고, `imcookie` 값을 쿠키 값으로 지정하면 위와 같은 패킷이 구성된다. 이대로 요청을 보내면 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php
if($_COOKIE['test'] == md5($_SERVER['REMOTE_ADDR']) && $_POST['kk'] == md5($_SERVER['HTTP_USER_AGENT'])) echo "<a href=???>Next</a>";
else echo "hint : {$_SERVER['HTTP_USER_AGENT']}";
?>
```

여섯 번째 문제는 다음 조건을 만족시키면 문제를 해결할 수 있다.

1. `test` 쿠기 값으로 공인 ip 값을 md5 암호화한 값을 포함시킨다.
2. 바디 값에 `kk` 값으로 클라이언트의 `HTTP_USER_AGENT` 값을 md5 암호화한 값을 포함시켜 **POST** 요청을 한다.

<br>

```html
<form action="gpcc.php" method="post">
  <input name="kk">
  <input type="submit">
</form>
```

2번 조건을 위해 마찬가지로 응답 패킷에 `form`을 삽입하고, input 데이터로 hint를 통해 얻은 값을 암호화하여 전달하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/0e4e7eca-de7a-4d62-918c-0bf7afdcae8e)

이때 요청 패킷에 cookie 값을 포함시켜 전송하면 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php
$_SERVER['REMOTE_ADDR'] = str_replace(".","",$_SERVER['REMOTE_ADDR']);
if($_GET[$_SERVER['REMOTE_ADDR']] == $_SERVER['REMOTE_ADDR']) echo "<a href=???>Next</a>";
else echo "Wrong<br>".$_GET[$_SERVER['REMOTE_ADDR']];
?>
```

일곱 번째 문제는 코드 해석에 주의해야 하는 문제이다. 공인 ip 값에서 `.`을 제거한 값을 `$_SERVER['REMOTE_ADDR']` 변수에 할당하고, 해당 값을 파라미터 이름과 값으로 하여 **GET** 요청하면 된다.

<br>

```
https://webhacking.kr/challenge/bonus-6/wtff.php?118235412=118235412
```

이런식으로 요청하면 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php

// GET 요청 시에 받은 파라미터의 키와 값을 변수화한다.
extract($_GET);

// GET 파라미터에 addr 값이 없다면 addr 변수에 공인 ip를 할당한다.
if(!$_GET['addr']) $addr = $_SERVER['REMOTE_ADDR'];

// addr 변수 값이 127.0.0.1 과 같다면 다음 단계 링크를 제공한다.
if($addr == "127.0.0.1") echo "<a href=???>Next</a>";
else echo "Wrong";
?>
```

여덟 번째 문제에서는 주어진 소스코드를 위와 같이 분석하였다. 파라미터로 `addr` 키 값을 `127.0.0.1`로 넘겨주면 `extract` 함수에 의해 `addr` 키 값이 변수로 지정되고, 이에 첫 번째 if문이 실행되지 않는다. 두 번째 if문에서 변수 `addr` 값을 비교하는데 우리는 `127.0.0.1`을 넘겨주었기에 조건이 참이 되고 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php
for($i=97;$i<=122;$i=$i+2){
  $answer.=chr($i);
}
if($_GET['ans'] == $answer) echo "<a href=???.php>Next</a>";
else echo "Wrong";
```

아홉 번째 문제는 ASCII 값 97부터 2씩 증가하며 122까지 문자로 변환하며 문자를 붙이고, 해당 문자열을 `ans` 파라미터 값으로 포함하여 **GET** 요청을 보내면 된다.

<br>

```
https://webhacking.kr/challenge/bonus-6/nextt.php?ans=acegikmoqsuwy
```

위의 주소와 같이 요청하면 다음 단계로 넘어갈 수 있다.

<br>

```php
<?php
$ip = $_SERVER['REMOTE_ADDR'];
for($i=0;$i<=strlen($ip);$i++) $ip=str_replace($i,ord($i),$ip);
$ip=str_replace(".","",$ip);
$ip=substr($ip,0,10);
$answer = $ip*2;
$answer = $ip/2;
$answer = str_replace(".","",$answer);
$f=fopen("answerip/{$answer}_{$ip}.php","w");
fwrite($f,"<?php include \"../../../config.php\"; solve(33); unlink(__FILE__); ?>");
fclose($f);
?>

```

마지막 문제인 열 번째 문제는 공인 ip 문자열을 가공하고 해당 문자열을 포함한 경로에 있는 php 파일에 접근하면 문제를 풀 수 있다. 문자열 가공하는 과정을 따라가는 방법도 있겠지만, php 파일 하나를 임시로 만들어 변수 값 `answer`과 `ip`를 획득하였다.

php 파일은 `{최상위 경로}/answerip/{$answer}_{$ip}.php`에 있었고, url에 입력하여 접근하니 문제를 풀 수 있었다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b96c32ff-9ba6-4ed5-b033-67f702598fa4)

<br>


---


## 🚩 old-36

```
While editing index.php file using vi editor in the current directory, a power outage caused the source code to disappear.
Please help me recover.
```

문제 페이지에 접속하면 위와 같은 문구를 확인할 수 있다. vi 에디터로 index.php 파일을 수정하다가 비정상 종료되었으니 복구해달라는 것이다. 가장 먼저 생각난 것은 vi와 같은 에디터로 편집 시에 임시 파일을 생성한다는 사실이었다.검색을 통해 vi 에디터 사용 중에 생성되는 임시 파일은 스왑파일(swp)이라는 사실을 알아내었다.

<br>

```
.{원본파일이름}.swp
```

위와 같은 형식으로 스왑파일이 생성됨을 알게 되었다. index.php를 수정하다가 종료되었으니 해당 파일 이름은 `.index.php.swp`이 될 것이다. 해당 파일은 현재 디렉터리에 있을 것이므로 주소창에 입력하여 접근해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/344e7a24-1912-415f-900a-edc04e96aa5d)

위와 같은 파일을 다운받을 수 있었다. 바이너리 파일이라 메모장이나 다른 편집 프로그램으로는 제대로된 내용을 확인할 수 없었다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b659226f-ef58-4276-a81a-e4ebcbf7ec6a)

리눅스의 cat 명령어를 통해 해당 파일을 열어보면 위와 같이 flag 변수에 담겨 있는 flag 값을 확인할 수 있었다. auth 탭에서 인증하면 문제를 풀 수 있다.

<br>

---

## 🚩 old-38

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/556010bf-f17c-458d-8134-b2aefdce11af)

문제 페이지에 접속하면 'Log Injection'이라는 문구와 로그인 하는 입력 창 하나를 발견할 수 있다. 무작위 문자열을 입력하고 로그인 시도하자 아무 일도 일어나지 않았다. 요청과 응답 패킷을 확인하다가 해당 페이지의 주석에서 admin page로 이동할 수 있는 경로를 알게 되었다. 해당 페이지로 이동하자 나의 ip 주소와 내가 입력한 값들을 볼 수 있었고, 다른 사용자의 정보까지 로그로 확인할 수 있었다. 'admin'으로 로그인 해야한다는 문구를 확인할 수 있다.

<br>

Log Injection을 통해 내 ip가 admin으로 로그인 되었음을 나타내야 하는 문제이다. 입력 값이 그대로 admin 페이지에 추가되는 것으로 보아, 내 ip가 admin으로 로그인 하였다는 로그를 남기도록 해보자.

<br>

`guest\n{내 ip 주소}:admin`

처음에는 이와 같이 입력하여 넘겨보았다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/acfb8f58-5b79-487c-98d8-2042da91a508)

그러나 `\n` 문자가 그대로 출력되었는데, 요청 시에 바디 값이 `id=guest%5Cn118.235.15.246%3Aadmin`로 전달되었었다. URL 인코딩되어 넘어갔음을 알 수 있다. 그래서 입력 값을 다양하게 하면서 요청을 해보았다.

|--|--|--|
|입력|요청|결과|
|--|--|--|
|guest\n{ip}:admin|id=guest%5Cn{ip}%3Aadmin|{ip}:guest\n{ip}:admin|
|guest\r\n{ip}:admin|id=guest%5Cr%5Cn{ip}%3Aadmin|{ip}:guest\r\n{ip}:admin|
|guest%0a{ip}:admin|id=guest%250aip%3Aadmin|{ip}:guest%0a{ip}:admin|

<br>

여러 시도 끝에 두 가지 해결 방법이 있었다.

<br>

1. `<input>` 태그가 아닌 `<textarea>`로 입력 창을 바꾸어 줄바꿈 키 입력 후 요청

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/bbf46d9d-f3c3-40cb-925a-a7a7dc5efeff)

위와 같이 요청할 때 바디 값이 `id=guest%0D%0A118.235.15.246%3Aadmin`로 설정되며 줄바꿈 처리 됨.

<br>

2. 요청 시에 바디 값을 `id=guest%0D%0A118.235.15.246%3Aadmin`로 바꾸어 요청

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/baa868fa-09e7-4755-aa2c-2fb65d236c3a)

마찬가지로 줄바꿈 처리되어 요청 전송 됨.

<br>

이제 admin 페이지에 접속해보면 문제를 풀었다는 alert를 확인할 수 있다.

<br>

---

## 🚩 old-39

문제 페이지에서는 입력 값을 제출할 수 있는 양식과 소스를 볼 수 있는 링크가 있었다. 임의의 문자열을 전송해도 아무 일도 일어나지 않으므로 소스를 확인하자.

<br>

```php
<?php
  // db 연결
  $db = dbconnect();

  if($_POST['id']){

    // id 값에서 문자열 \\ 값을 제거
    $_POST['id'] = str_replace("\\","",$_POST['id']);

    // id 값에서 문자열 ' 값을 '' 으로 변경
    $_POST['id'] = str_replace("'","''",$_POST['id']);

    // id 값에서 문자열을 0번째 인덱스부터 15개의 문자까지 추출
    $_POST['id'] = substr($_POST['id'],0,15);

    // id 값을 이용하여 sql 구문을 구성하여 쿼리 실행
    $result = mysqli_fetch_array(mysqli_query($db,"select 1 from member where length(id)<14 and id='{$_POST['id']}"));

    // result[0] 값이 1인 경우 문제 해결
    if($result[0] == 1){
      solve(39);
    }
  }
?>
```

소스코드에서 중요한 부분은 위와 같다. 여기서 입력한 id 값을 문자열 가공한 이후에 sql 구문을 실행하여 원하는 값을 조회해야함을 알 수 있다. id 값을 포함하여 실행하는 SQL 구문은 아래와 같다.

<br>

```
member 테이블에서 id 문자열 길이가 14보다 작고 id 값이 사용자가 입력한 id 값과 같은 데이터가 있다면 1 값을 반환한다.
```

<br>

`select 1 from member where length(id)<14 and id='~`

우선 우리가 입력하는 id 값은 쿼리 중 ~ 위치에 입력된다. 이때 입력된 값이 싱글 쿼터 `'` 로 열었지만 싱글 쿼터로 닫지 않아 온전한 문자열로 인식되지 않음을 알 수 있다. 그렇다고 `'`를 사용하면 `str_replace` 함수에 의해 `''`로 변경되어 문법 오류가 발생한다. 게다가 문자열 `\\`을 제거 시키기 때문에 `\\'`와 같이 사용하여 `'`를 문자로 인식시킬 수 없다. 우리는 테이블에 존재하는 id 중 길이가 14보다 작으며 문자열로 인식 시키기 위해 싱글 쿼터 하나를 뒤에 붙여야 한다.

<br>

```php
$_POST['id'] = substr($_POST['id'],0,15);
```

우선, 싱글 쿼터 하나를 입력할 방법은 위의 구문에 의해 찾을 수 있다. 총 15개의 입력 값 이후의 값은 잘리기 때문에 15번째 문자가 싱글 쿼터라면, `str_replace` 함수에 의해 16번째에 싱글쿼터가 하나 추가될 것이고, `substr` 함수에 의해 추가된 싱글쿼터가 잘릴 것이다. 이러면 우선 `id='12345678901234'`와 같이 문자열로 인식시킬 수 있다.

<br>

그러나 두 번째 문제가 있다. 14자리의 id 값을 가진 데이터가 테이블에 존재한다고 가정해도 쿼리문에서 `length(id)<14` 조건이 존재하기에 애초에 조건은 항상 False가 된다.

<br>

[https://techblog.woowahan.com/2559/](https://techblog.woowahan.com/2559/)

그렇다면 방법이 없는가? 위의 블로그에서 해답을 찾을 수 있었다. 간단히 요약하자면, MySQL에서는 문자열을 비교할 때 하나의 문자열 오른쪽 끝에 공백이 있을 때 다른 문자열에 공백을 이용해 문자열 길이를 맞추고 비교한다. 따라서 문자열 오른쪽 끝에 공백이 있더라도 이를 제외한 문자열끼리만 비교한다는 뜻이다.

그렇기에 `임의의 id 값 + 14번째 자리까지 공백 채우기  + '`를 추가하면 결과적으로 `id 값 + '`로 이루어진 문자열이 쿼리문에 들어가는 것이다. 여기서 id 값은 대부분의 테이블에 있는 admin으로 구성하면,

<br>

```
admin         '
```

이와 같이 페이로드를 구성할 수 있고, 제출하면 문제를 풀 수 있다.
