---
title: '[PortSwigger] Academy: SQL Injection'
date: 2024-04-01 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy](https://portswigger.net/web-security/learning-paths)] SQL Injection을 수강하고 정리하였습니다.
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

