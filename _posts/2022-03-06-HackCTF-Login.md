---
title: '[HackCTF] Login'
date: 2022-03-06 00:00:00
categories: [HackCTF, Wargame]
tags: [hackctf, wargame, webhacking]
---

> \#web \#100pts


# ๐ฉ ๋ฌธ์ 
---

![image](https://user-images.githubusercontent.com/37824335/222786190-dbce8ec8-c10b-411d-9626-c2873a000fde.png)

<br/>


# ๐ฉ ๋ฌธ์  ํ์ด
---
## ๐โ๐จ ๋ฌธ์  ํ์

![image](https://user-images.githubusercontent.com/37824335/222786256-4f4a855b-22aa-4fb7-96d6-23451433f87e.png)

๋ฌธ์ ์์ ์ ๊ณตํ ๋งํฌ๋ก ๋ค์ด๊ฐ๋ ๊ฐ๋จํ ๋ก๊ทธ์ธ ํผ์ด ๋ณด์๋ค.
**View Source**๋ฅผ ๋๋ฅด๋

![image](https://user-images.githubusercontent.com/37824335/222786722-341e05b4-e559-4e30-8839-c20a945e7170.png){: w="700"}

์์ ๊ฐ์ php์ฝ๋๋ฅผ ์ฃผ์๋ค. ์ฝ๋๋ฅผ ํด์ํด๋ณด๋ฉด, ์ฌ์ฉ์์๊ฒ id์ pw๋ฅผ ์๋ ฅ๋ฐ์ sql ์ฟผ๋ฆฌ๋ฌธ์ ์ฝ์ํ ํ ๋ฐ์ดํฐ๋ฒ ์ด์ค์์ ํด๋น ์ ๋ณด๋ฅผ ๊บผ๋ด์ ๋ฐฐ์ด์ ํ ๋นํ๊ณ  id์ ํด๋นํ๋ ๊ฐ์ด ์กด์ฌํ๋ฉด flag๋ฅผ ๋ด๋ฑ๋ ํํ์๋ค.

<br />

## ๐โ๐จ Exploit

๋จ์ํ **SQL Injection**์ผ๋ก ํ์๋์์ผ๋ DB์ ์ด๋ค id๊ฐ ์กด์ฌํ๋์ง๋ฅผ ๋ชจ๋ฅด๋ ์ํ์๋ค. ๊ทธ๋์ admin ๊ณ์ ์ ์กด์ฌํ  ๊ฒ ๊ฐ์ Usename์ผ๋ก `admin'#` ๋ฌธ์์ด์ ์ฝ์ํ์๋ค.

```sql
# origin
select * from jhyenuser where binary id='$id' and pw ='$pw'

# SQL Injection
select * from jhyenuser where binary id='admin'#' and pw =''
```

<br>

์์ ๊ฐ์ด ์ฟผ๋ฆฌ๋ฌธ์ด ์์ฑ๋์๊ณ  ๋ก๊ทธ์ธ์ ์๋ํ๋

![image](https://user-images.githubusercontent.com/37824335/222787168-ad8bab38-a671-4095-9446-a471568d4790.png)

flag๋ฅผ ๋ฑ์๋ค!
