---
title: '[Wargame] Webhacking.kr old-25 (PHP)'
date: 2024-08-03 00:00:00 +0900
last_modified_at: 2024-08-03
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

![image](https://github.com/user-attachments/assets/d7113151-a6eb-4154-8010-3fd1876dd0f2)

문제 페이지에 들어가면 `/?file=hello` 라는 url로 연결되고, `hello.php` 라는 파일의 내용이 출력되는 것으로 보이는 페이지를 확인할 수 있다. 이로 보아, url의 파라미터로 파일명을 전달하면 해당 파일을 읽을 수 있는 것을 알 수 있다.

<br>

`/?file=index`로 요청하면 아무런 결과도 받을 수 없으나, `/?file=flag`로 요청하면 위와 같이 **FLAG is in the code**라는 문자열을 확인할 수 있다. 이는 `flag.php`라는 파일을 **읽은 것**이 아니라 **실행한 것**으로 유추할 수 있고, 플래그 값은 해당 파일을 **읽어야** 알 수 있는 것으로 유추할 수 있다.


<br><br>


## 🚩 문제 풀이

---

이를 해결하기 위해서는 **PHP Wrappers**라는 개념이 필요하다. PHP Wrapper란 파일 시스템 함수와 함께 사용하기 위한 다양한 URL 스타일 프로토콜을 내장 wrapper가 제공하는 것을 말한다. [PHP 공식 문서](https://www.php.net/manual/en/wrappers.php)에서 자세한 내용을 확인할 수 있다.

<br>

![image](https://github.com/user-attachments/assets/bf0f6a96-d4a6-4cd0-84ca-dae65ac3ffdf)

공식 문서에서 wrapper의 종류를 확인할 수 있는데, 이 중에서 우리는 **파일을 읽어야** 하기 때문에 I/O streams와 관련된 `php://` wrapper를 사용할 것이다.

<br>

그 중 `php://filter`를 사용하면 파라미터를 이용하여 특정 파일 경로와 타입 등을 지정할 수 있다. php에서 제공하는 filter의 리스트는 [공식 문서](https://www.php.net/manual/en/filters.php)에서 찾아볼 수 있는데, 여러 필터를 이용하여 파일을 추출할 수 있었다.

<br>

사용 방법은 `php://filter/read={적용하려는 필터}/resource={파일 경로}`이다.

파일을 추출할 수 있었던 필터는 `string.rot13`, `convert.base64-encode`, `convert.quoted-printable-encode` 등이 있었다.

<br>

![image](https://github.com/user-attachments/assets/899504db-fc73-4648-a185-095004ae0531)

사용 가능한 필터 중 `convert.base64-encode`를 이용하여 url 페이로드를 만들면 **`php://filter/read=convert.base64-encode/resource=./flag`**와 같이 구성할 수 있고, 이를 포함하여 요청하면 위와 같이 base64로 인코딩된 값을 얻을 수 있다.

<br>

```
<?php
  echo "FLAG is in the code";
  $flag = "FLAG{this_is_your_first_flag}";
?>
```

이를 decode 한 값에서 플래그를 찾을 수 있다.


<br><br>


## 🚩 추가 분석

---

```php
<?php
  echo("<pre>");
  system("ls -al");
  echo("</pre>");
  if(!$_GET['file']) echo("<meta http-equiv=refresh content=0;url=?file=hello>");
  echo "<hr><textarea rows=10 cols=100>";
  $file = $_GET['file'].".php";
  if($file == "index.php") exit(); // anti infinite loop
  include $file;
  echo "</textarea>";
?>
```
같은 방식으로 `index.php` 파일을 추출할 수 있는데, `include $file;` 부분에서 LFI 취약점이 발생하는 것을 확인할 수 있었다.
