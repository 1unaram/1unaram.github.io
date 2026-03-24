---
title: "[H4CKING GAME] REV - Keygen"
date: 2022-05-16 00:00:00 +0900
last_modified_at: 2022-05-16
categories: [Wargame, H4CKING GAME]
tags: [wargame, reversing]
---

![image](https://user-images.githubusercontent.com/37824335/223182873-28afba2f-616c-4ed5-9420-dec83c530f32.png)

<br>

# # 문제 파악

---

![image](https://user-images.githubusercontent.com/37824335/223182979-08b941ae-0add-44fa-b063-253b9c752757.png)

문제에서 주어진 **keygen.exe** 파일을 실행해보니 알맞은 flag를 입력해야 함을 알 수 있었다. IDA를 이용하여 실행 파일을 열어보았다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223183069-93743549-1f27-4f39-82c6-c15f21af24ee.png)

main 함수를 찾을 수 있었고, 그래프에서 *"Input flag : "*, *"No... This is not flag..."* 등의 문자열을 확인할 수 있었다. 다행히도 디컴파일 할 수 있었다.

<br>

![image](https://user-images.githubusercontent.com/37824335/223183140-4014c51e-a226-432c-9b9e-0486f9941b3d.png)

분석해보자면,

- Line 7 : **Buf2**에 **"?;FJDnv8dw8lRulyRmmt"** 문자열을 copy.
- Line 8 : **Buf1**의 메모리를 0x15 만큼 0으로 초기화.
- Line 9 : 함수의 내부로 들어간 결과 `printf()` 함수로 추정됨. **aInputFlag** 변수는 *"Input flag : "* 문자열을 가리킴.
- Line 10 : 함수의 내부로 들어간결과 `scanf()` 함수로 추정됨. 사용자에게 입력을 받아 **Buf1** 변수에 저장.
- Line 11-16 : i를 1부터 19까지 1씩 증가시키며 **Buf1**(사용자에게 입력 받은 문자열)의 글자 하나씩 xor 연산과 뺄셈 연산을 진행하여 저장.
- Line 17-21 : **Buf1**과 **Buf2**의 메모리를 비교하여 동일하면 **aGoodThisIsFlag** 변수('Good!! This is flag!!')를 출력하고, 동일하지 않으면 **aNoThisIsNotFla** 변수('No... This is not flag...')를 출력.

이 문제의 핵심은 **Buf2** 문자열을 반복문 내의 연산을 통해 flag 값을 알아내는 것 같다.

<br>

```
   Buf2[i] == ((Buf1[i] ^ 0x11) ^ 0x1B) - 3
-> Buf2[i] + 3 == (Buf1[i] ^ 0x11) ^ 0x1B
-> (Buf2[i] + 3) ^ 0x1B == Buf1[i] ^ 0x11
-> ((Buf2[i] + 3) ^ 0x1B) ^ 0x11 == Buf1[i]
```

우선 **Buf1**의 각 글자(=flag의 각 글자)는 위의 식을 통해 알 수 있다.이제 Python으로 코드를 짜보자.

<br />

🔽 **keygen.py**
```python
# Buf2[i] == ((Buf1[i] ^ 0x11) ^ 0x1B) - 3

Buf2 = '?;FJDnv8dw8lRulyRmmt'
for i in range(0, 20):
    char = ord(Buf2[i])
    char += 3
    char ^= 0x1B
    char ^= 0x11
    print(chr(char), end="")

```

<br>

🔽 **Result**

![image](https://user-images.githubusercontent.com/37824335/223183596-599f7db6-3153-461f-a85f-6981a1b40a40.png)

<br />

![image](https://user-images.githubusercontent.com/37824335/223183621-d7253bc5-1288-409b-9ba5-37baf28354cc.png)

Flag를 찾았다!
