---
title: '[HackCTF] Login'
date: 2022-03-06 00:00:00 +0900
last_modified_at: 2022-03-06
categories: [Wargame, HackCTF]
tags: [hackctf, wargame, webhacking]
---

> \#web \#100pts


# 🚩 문제
---

![image](https://user-images.githubusercontent.com/37824335/222786190-dbce8ec8-c10b-411d-9626-c2873a000fde.png)

<br/>


# 🚩 문제 풀이
---
## 👁‍🗨 문제 파악

![image](https://user-images.githubusercontent.com/37824335/222786256-4f4a855b-22aa-4fb7-96d6-23451433f87e.png)

문제에서 제공한 링크로 들어가니 간단한 로그인 폼이 보였다.
**View Source**를 누르니

![image](https://user-images.githubusercontent.com/37824335/222786722-341e05b4-e559-4e30-8839-c20a945e7170.png){: w="700"}

위와 같은 php코드를 주었다. 코드를 해석해보면, 사용자에게 id와 pw를 입력받아 sql 쿼리문에 삽입한 후 데이터베이스에서 해당 정보를 꺼내와 배열에 할당하고 id에 해당하는 값이 존재하면 flag를 내뱉는 형태였다.

<br />

## 👁‍🗨 Exploit

단순한 **SQL Injection**으로 파악되었으나 DB에 어떤 id가 존재하는지를 모르는 상태였다. 그래서 admin 계정은 존재할 것 같아 Usename으로 `admin'#` 문자열을 삽입하였다.

```sql
# origin
select * from jhyenuser where binary id='$id' and pw ='$pw'

# SQL Injection
select * from jhyenuser where binary id='admin'#' and pw =''
```

<br>

위와 같이 쿼리문이 작성되었고 로그인을 시도하니

![image](https://user-images.githubusercontent.com/37824335/222787168-ad8bab38-a671-4095-9446-a471568d4790.png)

flag를 뱉었다!
