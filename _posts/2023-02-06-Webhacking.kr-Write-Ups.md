---
title: Webhacking.kr Write-Ups
date: 2023-02-06 00:00:00
categories: [Write-Up]
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
