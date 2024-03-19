---
title: "[Dreamhack] Quiz: x86 Assembly 2"
date: 2022-02-28 00:00:00
categories: [Write-Up, Dreamhack]
tags: [assembly, dreamhack, quiz]
---

> **Dreamhack x86 Assembly** 강의를 듣고 푼 Quiz를 정리하였습니다.
{: .prompt-info}

<br>

## # Quiz

---

![image](https://user-images.githubusercontent.com/37824335/223080923-30b09d5b-4b01-4699-8869-6317a15b3408.png){: w="700" }

<br>

## # 풀이

---

```
[Register]
rcx = 0
rdx = 0
rsi = 0x400000
=======================
[Memory]
0x400000 | 0x67 0x55 0x5c 0x53 0x5f 0x5d 0x55 0x10
0x400008 | 0x44 0x5f 0x10 0x51 0x43 0x43 0x55 0x5d
0x400010 | 0x52 0x5c 0x49 0x10 0x47 0x5f 0x42 0x5c
0x400018 | 0x54 0x11 0x00 0x00 0x00 0x00 0x00 0x00
=======================
[code]
1: mov dl, BYTE PTR[rsi+rcx]
2: xor dl, 0x30
3: mov BYTE PTR[rsi+rcx], dl
4: inc rcx
5: cmp rcx, 0x19
6: jg end
7: jmp 1
```

- 📌**code 1** : `rsi + rcx`의 주소를 참조한 값을 `dl` 레지스터에 복사

    > **dl** : RDX(Extended Data Register), DX(Data Register)의 하위 8bit 레지스터

    **Result** : `dl = 0x67 0x55 0x5c 0x53 0x5f 0x5d 0x55 0x10`

<br />

- 📌**code 2** : `dl` 레지스터 값과 `0x30` 값을 xor 연산하여 `dl`에 저장

    ![image](https://user-images.githubusercontent.com/37824335/223082535-c5b7ce4f-f934-43fb-8e5e-9c4dbada4233.png)

    **Result** : `dl = 0x57 0x65 0x6C 0x63 0x6F 0x6D 0x65 0x20`

<br />

- 📌**code 3** : `dl` 레지스터 값을 `rsi + rcx` 주소 참조 값에 복사

    **Result** : `0x400000 <= 0x57 0x65 0x6C 0x63 0x6F 0x6D 0x65 0x20`

<br />

- 📌**code 4** : `rcx` 값 1 증가

    **Result** : `rcx = 0x1`

<br />

- 📌**code 5** : `rcx` 값과 `0x19` 크기 비교

<br />

- 📌**code 6** : `rcx` 값이 `0x19`보다 크면 end로 jump

    **Result** : `0x1 > 0x19` (False)

<br />

- 📌**code 7** : 1로 jump

<br />

미루어 보아, `rcx`는 1씩 증가하고 `rsi + rcx`는`0x400000`부터 `0x400019`까지 접근하며 xor 연산을 할 것이다. 따라서 네 메모리 공간에 모두 xor 연산을 한 결과를 ASCII로 변환하면 원하는 문자열을 얻어낼 수 있을 것이다!

<br />

**Result**

`0x400000` ~ `0x400019` => Convert Hex to ASCII using [Online tool](https://www.rapidtables.com/convert/number/hex-to-ascii.html)

```
[Before Memory]
0x400000 | 0x67 0x55 0x5c 0x53 0x5f 0x5d 0x55 0x10 : gU\S_]U
0x400008 | 0x44 0x5f 0x10 0x51 0x43 0x43 0x55 0x5d : D_QCCU]
0x400010 | 0x52 0x5c 0x49 0x10 0x47 0x5f 0x42 0x5c : R\IG_B\
0x400018 | 0x54 0x11 0x00 0x00 0x00 0x00 0x00 0x00 : T

[After Memory]
0x400000 | 0x57 0x65 0x6C 0x63 0x6F 0x6D 0x65 0x20 : Welcome
0x400008 | 0x74 0x6F 0x20 0x61 0x73 0x73 0x65 0x6D : to assem
0x400010 | 0x62 0x6C 0x79 0x20 0x77 0x6F 0x72 0x6C : bly worl
0x400018 | 0x64 0x21 0x30 0x30 0x30 0x30 0x30 0x30 : d!000000
```

![image](https://user-images.githubusercontent.com/37824335/223082601-6d1369f6-06c5-49f3-a9b1-753ffdaa3e8a.png)