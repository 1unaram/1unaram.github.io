---
title: '[Wargame] Webhacking.kr old-39 (SQL Injection)'
date: 2023-12-03 00:00:00 +0900
last_modified_at: 2023-12-03
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

---

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/f2a85491-15a6-4315-8cd0-9b09ac93edd0)

문제 페이지에 접속하면 회원가입과 로그인할 수 있는 폼을 확인할 수 있다.
코드를 확인해보면 **JOIN** 시 ID는 `id`, PHONE은 `phone`의 name을 가지고, **LOGIN**시 ID는 `lid`, PHONE은 `lphone`의 name을 가지는 것을 확인할 수 있다. *view-source*를 통해 프로그램 소스코드를 확인해보자.

<br>

- JOIN 처리 코드

```php
if($_POST['id'] && isset($_POST['phone'])){
    $_POST['id'] = addslashes($_POST['id']);
    $_POST['phone'] = addslashes($_POST['phone']);
    if(strlen($_POST['phone'])>=20) exit("Access Denied");
    if(preg_match("/admin/i",$_POST['id'])) exit("Access Denied");
    if(preg_match("/admin|0x|#|hex|char|ascii|ord|select/i",$_POST['phone'])) exit("Access Denied");
    mysqli_query($db,"insert into chall59 values('{$_POST['id']}',{$_POST['phone']},'guest')");
}
```

1. `addslashes` 함수로 입력 값을 처리하기에 싱글쿼터 `'`를 사용할 수 없다.
2. `phone` 값의 길이는 20 이상일 수 없다.
3. `id` 값은 `/admin/i` 정규식을 만족할 수 없다.
4. `phone` 값은 `/admin|0x|#|hex|char|ascii|ord|select/i` 정규식을 만족할 수 없다.
5. 모든 조건을 만족했을 시에 `insert into chall59 values('{$_POST['id']}',{$_POST['phone']},'guest')` 구문을 실행한다.

<br>

- LOGIN 처리 코드

```php
if($_POST['lid'] && isset($_POST['lphone'])){
    $_POST['lid'] = addslashes($_POST['lid']);
    $_POST['lphone'] = addslashes($_POST['lphone']);
    $result = mysqli_fetch_array(mysqli_query($db,"select id,lv from chall59 where id='{$_POST['lid']}' and phone='{$_POST['lphone']}'"));
    if($result['id']){
        echo "id : {$result['id']}<br>lv : {$result['lv']}<br><br>";
        if($result['lv'] == "admin"){
            mysqli_query($db,"delete from chall59");
            solve(59);
        }
        echo "<br><a href=./?view_source=1>view-source</a>";
        exit();
    }
}
```

1. `addslashes` 함수로 입력 값을 처리하기에 싱글쿼터 `'`를 사용할 수 없다.
2. 데이터베이스에서 사용자가 입력한 `lid`와 `lphone`에 해당하는 데이터의 `lv`이 `admin`일 경우 문제를 해결할 수 있다.

<br>

SQL Injection을 통해 `lv`이 `admin`인 계정으로 회원가입하고 로그인하면 문제를 풀 수 있다.

<br><br>


## 🚩 문제 풀이

---

우선 `lv`이 `admin` 값을 가지는 계정을 생성해보자. join 시와 login 시에 모두 **싱글쿼터**를 사용할 수 없기에 이를 사용하지 않고 계정을 생성하는 방법을 찾아야한다.

<br>

우리가 입력한 값은 `insert` 구문을 구성하여 MySQL 쿼리를 실행하기 때문에 MySQL에서 지원하는 문자열 관련 함수를 이용하여 admin 계정을 생성해보자.

<br>

`id=admi&phone=1,concat(id,chr(110))--+` (주석 처리 뒤에 공백 문자 필요)

대표적인 문자열 관련 함수는 `concat`, `substr`, `replace`, `reverse`, `upper`, `lower` 등이 있는데, 처음에는 이 중 concat 함수를 이용하여 POST 요청 시 전송하는 바디 값을 구성하였다. 그러나, `phone` 값의 길이가 20이상일 수 없다는 조건에 충족하지 못하였다.

<br>

`id=nimda&phone=1,reverse(id))--+`

그래서 `reverse` 함수를 이용하여 구성하였고, 이는 모든 조건을 충족하였다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/404a0a4a-8776-4184-9520-17a361644788)

페이지에 직접 입력하여 join하면 이런 모습이 될 것이다.

<br>

`lid=nimda&lphone=1`

요청을 보내고 위의 바디 값으로 LOGIN을 시도하였더니 문제를 풀 수 있었다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/2e9cf037-7dd3-4dad-ad8e-8dd1075c2457)

페이지에 직접 입력하여 login하면 위의 그림과 같다.
