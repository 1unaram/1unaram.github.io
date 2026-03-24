---
title: "[Dreamhack] Quiz: x86 Assembly 3"
date: 2022-03-20 00:00:00 +0900
last_modified_at: 2022-03-20
categories: [Write-Up, Dreamhack]
tags: [assembly, dreamhack, quiz]
---

> **Dreamhack x86 Assembly** 강의를 듣고 푼 Quiz를 정리하였습니다.
{: .prompt-info}

<br>

## # Quiz

---

![image](https://user-images.githubusercontent.com/37824335/223169976-52b4889e-c012-4ea9-8f8f-c35a0c913bf2.png)

<br>

## # 풀이

---

```
[Code]
main:
    push rbp
    mov rbp, rsp
    mov esi, 0xf
    mov rdi, 0x400500
    call 0x400497 <write_n>
    mov eax, 0x0
    pop rbp
    ret

write_n:
    push rbp
    mov rbp, rsp
    mov QWORD PTR [rbp-0x8],rdi
    mov DWORD PTR [rbp-0xc],esi
    xor rdx, rdx
    mov edx, DWORD PTR [rbp-0xc]
    mov rsi,QWORD PTR [rbp-0x8]
    mov rdi, 0x1
    mov rax, 0x1
    syscall
    pop rbp
    ret

==================================
[Memory]
0x400500 | 0x3037207964343372
0x400508 | 0x003f367562336420
```

### 📌 main 함수

- `push rbp` : rbp를 스택에 push (for. 기존 스택 프레임 저장)

- `mov rbp, rsp` : rsp를 rbp로 옮김 (for. 새로운 스택 프레임 생성)

- `mov esi, 0xf` : `esi = 0xf`

- `mov rdi, 0x400500` : `rdi = 0x400500(=0x3037207964343372)`

- `call 0x400497 <write_n>` : wirte_n 이름의 함수 호출

- `mov eax, 0x0` : `eax = 0x0`

- `pop rbp` : rbp를 스택에서 pop (스택 해제)

- `ret` : 호출 주소로 돌아감 (함수 종료)


### 📌 write_n 함수

- `push rbp` : rbp를 스택에 push (for. main 함수의 주소를 스택에 push)

- `mov rbp, rsp` : rsp를 rbp로 옮김 (for. 새로운 스택 프레임 생성)

- `mov QWORD PTR [rbp-0x8],rdi` : rdi를 `rbp-0x8` 주소를 8byte만큼 참조하여 대입 => `0x3037207964343372`값 저장

- `mov DWORD PTR [rbp-0xc],esi` : esi를 `rbp-0xc` 주소를 4byte만큼 참조하여 대입 => `0xf`값 저장

- `xor rdx, rdx` : rdx와 rdx를 xor 연산하여 rdx에 저장 => rdx 값을 0으로 초기화

- `mov edx, DWORD PTR [rbp-0xc]` : `rbp-0xc` 주소를 4byte만큼 참조하여 edx에 대입

- `mov rsi, QWORD PTR [rbp-0x8]` : `rbp-0x8` 주소를 8byte만큼 참조하여 rsi에 대입 - **(1)**

- `mov rdi, 0x1` : `rdi = 0x1` - **(2)**

- `mov rax, 0x1` : `rax = 0x1` - **(3)**

- `syscall` - **(4)**

- `pop rbp` : rbp를 스택에서 pop (스택 해제)

- `ret` : 호출 주소로 돌아감 (함수 종료)

> **(1)** : ASCII 문자는 1byte(=8bit)로 한 문자를 표현하고, 16진수는 한 문자를 4bit로 표현하기에 16진수 두 문자로 ASCII 문자 한 문자를 표현할 수 있다. `rbp-0x8`을 8byte만큼 참조하면 0x400500부터 0x400508까지 총 32bit를 참조하게 된다. 이때, x86 시스템은 Little-endian이기 때문에 높은 주소에서 낮은 주소로 바이트를 배열한다.
{: .prompt-tip }

> **(1), (2), (3)**이 이해가 되지 않아 **syscall**의 동작 방법을 찾아보았다.
>
> `rax` 레지스터는 사칙연산 명령어에서 사용되기도 하지만 **syscall**의 번호를 가리키는 레지스터이기도 하다. 그래서 **(3)**에서 설정한 `rax = 0x1`은 **(4)**에서 시스템 콜이 **write** 함수를 호출하는 것이다.
>
> 또한, rdi 레지스터는 인덱스 레지스터로도 사용되기도 하지만 함수 호출 시 첫 번째 인자로 넘겨주는 값을 저장하기도 한다. 따라서 **(2)**에서 지정한 `0x1`이 **write** 함수를 호출 할 때 인자 값으로 넘어간 것이다. 이는 [syscall table](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md)에서 찾아볼 수 있듯이 `unsigned int fd`로 인자로 넘긴 것이다. `rdi -> rsi -> rdx -> rcx -> r8 -> r9` 순으로 인자를 넘긴다.
{: .prompt-tip }

위를 종합해보면 **syscall**이 **write**함수를 호출하면서 인자로 [`0x1`, `0x3037207964343372(reverse) 0x003f367562336420(reverse)`, `0`]을 넘긴 것이다.

<br>

write 되는 Hex 값을 ASCII로 변환하면,

![image](https://user-images.githubusercontent.com/37824335/223170069-7343cf60-7fba-4076-b69b-d48d199df00d.png)

`r3ady 70 d3bu6?` (`ready to debug?`) 답을 찾을 수 있다!

![image](https://user-images.githubusercontent.com/37824335/223170120-25aff6c2-1f57-46c0-9245-3c1991989937.png)

<br />


~~<span style="color: red">
레지스터의 쓰임과 용도를 먼저 공부하지 않아 이해가 되지 않는 것이었다.. 빠른 시일 내에 레지스터를 정리해야겠다,,,
</span>~~


