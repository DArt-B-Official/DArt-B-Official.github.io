---
title: '[Wargame] Webhacking.kr old-61 (SQL Injection)'
date: 2023-12-03 00:00:00
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e84dc58c-f43b-4fdf-86f1-2a356206e0b0)

문제 페이지에 접속하면 *view-source* 이외에는 다른 요소를 확인할 수 없다. 링크를 클릭하여 소스코드를 확인해보자.

<br>

```php
// id 파라미터를 받아 addslashes 함수를 통해 문자열 이스케이프
$_GET['id'] = addslashes($_GET['id']);

// 문자열 정규식 검사
if(preg_match("/\(|\)|select|from|,|by|\./i",$_GET['id'])) exit("Access Denied");

// 문자열 길이가 15를 초과하면 종료
if(strlen($_GET['id'])>15) exit("Access Denied");

// MySQL 쿼리문 실행
$result = mysqli_fetch_array(mysqli_query($db,"select {$_GET['id']} from chall61 order by id desc limit 1"));
echo "<b>{$result['id']}</b><br>";

// 쿼리 실행 결과 값이 admin 문자열과 같다면 문제 해결
if($result['id'] == "admin") solve(61);
```
코드에서 주요한 부분은 위와 같다.


<br><br>


## 🚩 문제 풀이

---

우선, 입력 값 id를 통해 구성하는 쿼리문을 분석해보자.

<br>

```sql
select {id} from chall61 order by id desc limit 1
```

: `chall61` 테이블을 id 컬럼을 기준으로 내림차순으로 정렬하였을 때 최상위 1개 데이터의 id 값을 추출

<br>

즉, id 컬럼을 기준으로 내림차순하였을 때 최상위 1개 데이터의 id 값을 정확하게 입력해야 해당 값을 추출할 수 있다. 우리는 이 값이 `admin`이어야 문제를 해결할 수 있으나, `?id=admin`으로 요청하여도 아무런 결과를 확인할 수 없기에 SQL Injection 공격을 통해 우리가 원하는 값을 추출해야 한다.

<br>

이를 위해서는 입력 값에 대한 정규식 검사 구문을 분석해야 한다. 분석 결과 다음과 같은 문자 및 문자열 사용 불가 조건이 적용되었다.

```
/\(|\)|select|from|,|by|\./i
```

- `(` 문자
- `)` 문자
- `select` 문자열
- `from` 문자열
- `,` 문자
- `by` 문자열
- `.` 문자
- Case sensitive

<br>

<u>결국 여러 필터링을 우회하고 `'admin'`이라는 문자열을 select 할 수 있도록 해야하는 것이 이 문제의 궁극적인 목표이다.</u> 이를 위해서는 MySQL에서 사용하는 **alias** 라는 키워드이다.

<br>

alias 키워드는 테이블이나 특정 컬럼에 새로운 이름, 즉 별칭을 지정해줄 때 사용한다.

<br>

```sql
select 'abc' as id;
```

위와 같이 실행한다면 `abc`라는 문자열 값을 가진 컬럼을 id로 부를 수 있다는 것이다.

<br>

```sql
select 'admin' as id from chall61 order by id desc limit 1
```

만약 `id` 파라미터 값으로 `'admin' as id`라는 값을 넘겨주면 위와 같은 쿼리문을 구성하게 되고 해당 쿼리문에서 `id` 컬럼은 `'admin'` 문자열을 가리키게 된다. 따라서 해당 쿼리문의 실행 결과는 `'admin'`이 된다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/20dedff0-7181-4428-a4d8-16d64ff11915)

여기서 한 가지 문제점이 있는데, 앞서 `id` 파라미터 값을 `addslashes` 함수를 통해 필터링하기에 싱글쿼터 `'`를 사용할 수 없다는 것이다. 그러나 MySQL에서는 hex 값을 ASCII 값으로 인식하기 때문에 위의 이미지와 같은 결과가 나오게 된다. 그러므로 우리는 `'admin'` 문자열 대신에 hex 값 `0x61646d696e`을 사용할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/dc9aeea1-9a23-4e22-b3fb-c3a04193c9d2)

```
?id=0x61646d696e as id
```

이제 파라미터로 넘길 값을 구성해보자. 이렇게 파라미터를 설정하고 url 요청을 하면 위의 이미지처럼 access denied 문구를 확인할 수 있다. 이유는 조건 중에 `id` 문자열의 길이가 15를 초과했기 때문이다. 이에 대한 해결 방법은 간단한데, <u>alias 키워드 `as`는 생략이 가능하다</u>는 점을 이용하면 된다.

<br>

```
?id=0x61646d696e id
```

따라서 위와 같이 구성하면 정확히 15글자로 값을 만들 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/81739c50-2f00-4068-8b3e-908f5bfa8096)

문제 해결!
