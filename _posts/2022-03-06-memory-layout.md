---
title: '[Study] Memory Layout'
date: 2022-03-06 00:00:00
categories: [Study]
tags: [os, memory, linux]    
---

# Memory Layout?
---
: **메모리 레이아웃(Memory Layout)**이란 프로그램이 실행될 때 운영체제는 프로세스가 사용 가능한 **가상 메모리(Virtual Memory)** 공간을 할당 받는데, 이 가상 메모리의 구성을 메모리 레이아웃이라고 합니다. 프로세스가 사용할 데이터를 적절한 곳에 적재하고 구획하여, 개발자의 직관적인 이해를 돕고 구획별 적절한 권한을 부여할 수 있어 메모리 오염(Memory Corruption) 취약점을 방지할 수 있는 특징이 있습니다.  메모리 레이아웃은 운영체제마다 다른데, 여기서는 Windows와 Linux 두 OS에 대한 레이아웃을 알아보겠습니다.

<br>

# 💻 Windows Memory Layout
---
## 섹션
### .text
: 실행 가능한 기계 코드가 위치하는 영역. 코드 영역으로도 불리는 이 영역은 제어문, 함수, 상수 들을 저장함.

- 읽기 권한**(r)**과 실행 권한**(x)**이 부여됨
- 쓰기 권한을 부여하면 공격자가 악의적인 코드를 삽입할 수 있기에 대부분의 운영체제는 쓰기 권한을 제거함.

<br />

### .data
: 컴파일 시점에 값이 정해진 *전역변수*, *정적변수* 들이 위치하는 영역.

- 메인(main) 함수 전에 선언 되어 프로그램이 끝날 때까지 메모리에 남아있는 변수.
- 읽기 권한**(r)**과 쓰기 권한**(w)**이 부여됨.

<br />

### .rdata
: 컴파일 시점에 값이 정해진 *전역상수*와 참조할 *DLL* 및 *외부 함수*들의 정보과 같은 임포트 데이터가 저장되는 영역.

- 읽기 권한**(r)**이 부여됨.
- 과거에는 참조할 DLL과 외부 함수들의 정보를 `.idata` 영역에 저장하였으나, 최근에는 `.rdata`에 저장하는 추세.
<br />

```c
const char data_rostr[] = "readonly_data"; // - (1)
char *str_ptr = "readonly"; // - (2)

int main() {...}
```

**(1)** - `"readonly_data"` 문자열이 const에 의해 *상수 문자열*로 저장되었기 때문에 `.rdata` 영역에 저장됨.

**(2)** - `str_ptr`은 *전역 변수*이기에 `.data`에 위치하나, `"readonly"`는 *상수 문자열*로 취급되기에 `.rdata` 영역에 저장됨.

<br />

## 섹션이 아닌 메모리
### Stack
: 프로세스의 각 Tread는 자신만의 스택 공간을 갖고 있는데 이 영역에는 보통 *지역변수*, *함수의 return address*, *매개변수(함수의 인자)*들이 저장됨.

- 읽기 권한**(r)**과 쓰기 권한**(w)**이 부여됨.
- 함수가 종료되면 해당 함수에 할당된 변수들을 해제함.
- **높은 주소**에서 **낮은 주소**로 메모리에 할당되기에 '아래로 자란다'라는 표현을 사용함.

<br />

### Heap
: 사용자에 의해 관리되는 영역으로, *모든 종류의 데이터*가 저장될 수 있어 여러 용도로 사용되는 영역. 비교적 스택보다 큰 데이터를 저장할 수 있으며, 전역적으로 접근이 가능함. 메모리 동적 할당으로 저장되는 데이터는 모두 힙에 저장됨.

- 읽기 권한**(r)**과 쓰기 권한**(w)**을 가지나, 상황에 따라서 실행 권한**(x)**을 가지는 경우도 존재.
- `malloc()`, `calloc()` 등으로 메모리 할당 받음.
- **낮은 주소**에서 **높은 주소**로 메모리에 할당되기에 '위로 자란다'라는 표현을 사용함.

> 💡 힙과 스택이 반대 방향으로 자라는 이유?
-> 두 영역이 동일한 방향으로 자라면, 한 영역을 모두 사용했을 때 이를 확장하는 과정에서 다른 영역과 충돌하게 됩니다. 이를 해결하기 위해 스택을 메모리의 **끝**에 위치 시키고, 힙과 스택을 반대로 자라게 하여 메모리를 최대한 자유롭게 이용하고 충돌 문제로부터 자유롭게 됩니다.

<br>

# 💻 Linux Memory Layout
---
> 리눅스에서는 프로세스의 메모리를 5가지의 **세그먼트(segment, 영역)**로 구분합니다.
{: .prompt-tip}

### Code Segment
: **코드 세그먼트(Code Segment)**는 **텍스트 세그먼트(Text Segment)**라고도 불리는데, 프로그램에서 실행 가능한 코드 데이터가 위치함. 윈도우의 **.text** 영역과 동일.

- 읽기 권한**(r)**과 실행 권한**(x)**이 부여됨
- 쓰기 권한을 부여하면 공격자가 악의적인 코드를 삽입할 수 있기에 대부분의 운영체제는 쓰기 권한을 제거함.

<br />

### Data Segment
: **데이터 세그먼트(Data Segment)**는 <span style="text-decoration: underline">컴파일 시점에 값이 정해지는</span> *전역변수* 및 *전역상수*들이 위치함.

- 기본적으로 읽기 권한**(r)**이 부여됨.
- 데이터 세그먼트는 읽기 권한에 추가로 쓰기 권한**(w)**을 부여하는지에 따라 세그먼트를 나눔.
    - **data 세그먼트** : 쓰기 권한이 <span style="color: red">부여된</span> 세그먼트로, *전역변수*와 같이 프로그램이 실행되면서 값이 변할 수 있는 데이터가 위치함.
    - **rodata(read-only data) 세그먼트** : 쓰기 권한이 <span style="color: red">부여되지 않은</span> 세그먼트로, *전역상수*와 같이 프로그램이 실행되면서 값이 변하면 안되는 데이터가 위치함.
<br />

```c
char data_rwstr[] = "writable_data"; // - (1)
const char data_rostr[] = "readonly_data"; // - (2)
char *str_ptr = "readonly"; // - (3)

int main() {...}
```

**(1)** - `data_rwstr`은 `"writable_data"`을 가리키는 *전역 변수*이기 때문에 **data 세그먼트**에 저장됨. 

**(2)** - `"readonly_data"` 문자열이 const에 의해 *상수 문자열*로 저장되었기 때문에 **rodata 세그먼트**에 저장됨.

**(3)** - `str_ptr`은 `"readonly"`를 가리키는 *전역 변수*이기에 **data 세그먼트**에 위치하나, `"readonly"`는 *상수 문자열*로 취급되기에 **rodata 세그먼트**에 저장됨.

<br />

### BSS Segment
: **BSS 세그먼트(BSS Segment, Block Started By Symbol Segment)**는 <span style="text-decoration: underline">컴파일 시점에 값이 정해지지 않는</span> *전역 변수*가 위치함.

- 읽기 권한**(r)**과 쓰기 권한**(w)**이 부여됨.
- 초기화하지 않은 전역변수처럼 프로그램 시작 시에 모두 **0**으로 값이 초기화 됨.

<br />

### Stack Segment
: 프로세스에서 사용하는 스택이 위치하는 메모리 영역. *함수의 인자*, *지역변수*, *매개변수*와 같은 임시 변수들이 프로그램 실행 중에 동적으로 할당 받아 메모리에 저장됨.

- 읽기 권한**(r)**과 쓰기 권한**(w)**이 부여됨.
- **높은 주소**에서 **낮은 주소**로 메모리에 할당.

<br />

### Heap Segment
: C언어에서 `malloc()`, `calloc()`와 같은 함수를 통해 프로그램 실행 중 동적으로 할당받는 메모리가 위치하는 영역. 

- 읽기 권한**(r)**과 쓰기 권한**(w)**이 부여됨.
- **낮은 주소**에서 **높은 주소**로 메모리에 할당.

<br>

# Summary
---
## Windows Memory Layout

|섹션|역할|일반적인 권한|사용 예|
|:---|:---|:------------|:------|
|**.text**|실행 가능한 코드가 위치하는 영역|읽기/실행|main() 등의 함수 코드|
|**.data**|초기화된 *전역변수*가 위치하는 영역|읽기/쓰기|초기화된 *전역변수*, *전역상수*|
|**.rdata**|초기화된 *전역상수*나 *임포트 데이터*가 위치하는 영역|읽기|*전역상수*, *외부 함수 정보 등의 임포트 데이터*|
|**Stack Segment**|임시 변수가 위치하는 영역|읽기/쓰기|*지역변수*, *함수의 인자* 등|
|**Heap Segment**|사용자에 의해 실행 중에 동적으로 사용되는 영역|읽기/쓰기|malloc(), calloc() 등으로 할당받은 메모리| 

## Linux Memory Layout

|세그먼트|역할|일반적인 권한|사용 예|
|:------|:--|:----------|:-----|
|**Code Segment**|실행 가능한 코드가 위치하는 영역|읽기/실행|main() 등의 함수 코드|
|**Data Segment**|초기화된 데이터가 위치하는 영역|읽기 or 읽기/쓰기|초기화된 *전역변수*, *전역상수*|
|**BSS Segment**|초기화되지 않은 데이터가 위치하는 영역|읽기/쓰기|초기화되지 않은 *전역변수*|
|**Stack Segment**|임시 변수가 위치하는 영역|읽기/쓰기|*지역변수*, *함수의 인자* 등|
|**Heap Segment**|사용자에 의해 실행 중에 동적으로 사용되는 영역|읽기/쓰기|malloc(), calloc() 등으로 할당받은 메모리| 
