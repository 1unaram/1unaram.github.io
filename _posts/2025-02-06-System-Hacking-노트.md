---
title: '[Study] System Hacking 노트'
date: 2025-02-06 00:00:00
categories: [Study, System Hacking]
tags: [pwnable]
published: True
---

> 시스템 해킹을 공부하며 도움이 될 만한 내용을 계속해서 정리합니다.
{: .prompt-info }

<br>

<br>

# ELF 바이너리의 심볼 테이블(Symbol Table)

---

ELF 바이너리에는 실행에 필요한 함수 및 변수의 주소를 저장하는 **심볼 테이블(Symbole Table)**이 포함되어 있다.

- pwntools

    - pwntools의 `ELF()` 클래스를 사용하면, 이 심볼 테이블을 분석하여 특정 함수의 주소를 가져올 수 있다.

    - `e.symbols`는 해당 ELF에서 심볼로 등록된 변수나 함수들의 주소를 반환하는 딕셔너리.


<br>

# ret2main

---

`main` 함수는 실행 파일의 `.text` 섹션에 위치하는데, **NX(Non-eXecutable)**가 적용되더라도 `.text` 섹션의 코드 실행은 허용되므로, `main` 함수의 주소를 직접 참조할 수 있음.

**ASLR(Address Space Layout Randomization)**이 적용되어도 **라이브러리, 스택, 힙**의 주소를 랜덤화하지만 **.text 섹션**의 주소는 랜덤화하지 않기 때문에 `main` 함수의 주소는 변하지 않는다.

그러나, **PIE(Position Independent Executable)**이 적용되면 실행할 때마다 .text 섹션이 랜덤한 주소에 매핑되기에 `main` 함수의 주소도 바뀌게 됨.

<br>

- main 함수 주소를 직접 확인하는 방법

    1. objdump

    ```sh
    objdump -d ./basic_rop_x64 | grep "<main>"
    ```

    2. readelf

    ```sh
    readelf -s ./basic_rop_x64 | grep "main"
    ```
