---
title: '[Study] Stack All-in-One'
date: 2022-08-21 00:00:00
categories: [Study, CS]
tags: [pwnable, stack]
---

# \#0. Intro

---

시스템 해킹을 공부하면서 *BOF*, *Stack Buffer Overflow*, *ROP* 등의 개념을 공부하기 위해서는 **Stack**의 구조와 동작 과정을 정확하게 이해해야 함을 통감했습니다. 이번 포스트에서는 프로그램이 동작하며 Stack이 어떻게 사용되는지를 *Assebly, Register, Endian*등의 내용을 포함하여 자세하게 기술해보겠습니다.

> 모든 개념을 정확하게 이해하고 있는 것은 아니지만 계속 공부하며 내용을 수정·보완해 나아갈 예정입니다. <u>잘못된 내용이 있을 수 있으므로</u> 참고해주시면 감사하겠습니다 😊

<br />

# \#1. Stack?

---

우선 Stack이란,
프로그램이 실행될 때 운영체제는 프로세스가 사용 가능한 **가상 메모리(Virtual Memory)**공간을 할당해주는데 이 가상 메모리의 구성을 **메모리 레이아웃(Memory Layout)**이라고 하고, 그 중 <u>지역변수, 함수의 return address, 함수의 매개변수</u>와 같은 임시 변수들이 프로그램 실행 중에 동적으로 할당 받아 저장되는 메모리 공간을 **Stack**이라고 합니다.

<br />

Stack은 Windows와 Linux에서 **Stack Segment**라고 하는 공간을 뜻하며, **낮은 주소**에서 **높은 주소**로 메모리가 저장되는 다른 메모리 공간에 비해, **높은 주소**에서 **낮은 주소**로 메모리 공간이 할당됩니다. 이를 ***"위로 자란다"*** 라고 표현하기도 합니다.

<br />

또한 Stack은 `push`와 `pop` 연산을 통해 메모리에 값을 저장하는데, **후입선출(Last-In-First-Out)**의 방식을 따라 나중에 저장된 값이 먼저 제거되는 구조입니다.

<br />

🔽 Linux Memory Layout

![image](https://user-images.githubusercontent.com/37824335/224681910-38c16caa-b40a-453a-925f-dca69238759b.png)

<br />

# \#2. Stack Frame

---

첫 번째로, 스택 프레임에 대해서 알아보겠습니다.
스택 프레임(Stack Frame)이란, 프로그램이 실행되며 함수 호출 시 <u>함수의 매개변수, 호출이 끝난 뒤 돌아갈 반환 주소(return address) 값, 함수 내에서 선언된 지역 변수</u> 등이 저장되는데, 이렇게 스택 영역에 저장되는 함수의 호출 정보를 **스택 프레임(Stack Frame)**이라고 합니다.
함수가 호출되면 스택 프레임을 생성하고(메모리 블록이라고 이해하면 좋음) 함수가 반환되면 스택 프레임을 반납합니다(메모리 반환).스택 프레임을 이용하여 함수에 사용하는 변수들을 효율적으로 관리할 수 있고, 함수 호출 이전의 상태로 돌아갈 수 있습니다.

🔽 스택 프레임 구조

![image](https://user-images.githubusercontent.com/37824335/224682117-233b70c6-7b81-4a86-91fc-04f1ea92035a.png)

위의 이미지는 스택 프레임의 구조를 나타내고 있습니다. 자세한 설명은 이후에 진행하겠습니다.

<br />

## 📃 예제 분석


간단한 덧셈 프로그램으로 스택 프레임의 생성 과정을 알아봅시다. 여기서는 ***x64*** 환경에서 동작하는 과정을 다뤄볼 예정이고, 분석의 편의를 위해 `-fno-stack-protector -no-pie` 옵션을 주어 각종 보호 기법을 해제하고 컴파일 하겠습니다.

```c
// add.c
// gcc -fno-stack-protector -no-pie -o add add.c

#include <stdio.h>

int add(int c, int d) {
	int e = c;
    int f = d;

    return e + f;
}

int main() {
	int a, b;

    a = 2;
    b = 3;

    printf("%d", add(a, b));
}
```

1.  `main` 함수 실행
2. 지역 변수 int형(4byte) `a`, `b` 생성
3. `2`, `3`을 각각 할당
4. `add(a, b)`의 반환 값을 int형으로 출력

    a. `add` 함수 실행

    b. 지역 변수 int형(4byte) `c`, `d`에 `a`, `b` 값 할당

    c. `c + d` 값을 return

<br />

디버거를 통해 차근차근 확인해봅시다. 사용한 디버거는 [pwndbg](https://github.com/pwndbg/pwndbg)입니다.

```shell
$ gdb -q ./add
```

<br />

### 📌 Register
분석에 앞서 많이 사용되는 **레지스터(register)**를 간단하게 정리하겠습니다. 레지스터에 대한 자세한 정보는 [이곳](https://velog.io/@1unaram/OS-Register)에서 더 확인할 수 있습니다.

|이름(x86)|이름(x64)|주용도|
|:-------|:--------|:-----|
|🔽 **범용 레지스터**|||
|EAX|RAX|산술 연산 시 상수/변수 값 저장, 함수의 반환 값 지정|
|EBX|RBX|산술 연산 시 상수/변수 값 저장, 함수의 반환 값 지정|
|ECX|RCX|반복문의 반복 횟수와 각종 연산의 시행 횟수로 사용되는 값 저장, 범용적으로 사용|
|🔽 **포인터 레지스터**|||
|ESP|RSP|스택 프레임의 끝지점(top) 주소를 가리키는 포인터|
|EBP|RBP|스택 프레임의 바닥(시작) 주소를 가리키는 포인터|
|🔽 **명령어 포인터 레지스터**|||
|EIP|RIP|CPU가 어느 부분의 기계어 코드를 다음으로 실행할지 가리키는 역할을 함|

<br />

### 📌 함수 프롤로그, 에필로그

```shell
pwndbg> disassemble main
```

![image](https://user-images.githubusercontent.com/37824335/224682371-7cf58fe3-b834-4407-8684-6a66d7b76b3e.png)

![image](https://user-images.githubusercontent.com/37824335/224682521-bc94ec6d-2045-4985-af66-9f003a0b7afd.png)

`main` 함수의 어셈블리 코드를 살펴보면, 함수의 시작과 끝 부분에 위와 같은 명령을 확인할 수 있습니다. 이는 각각 **함수의 프롤로그(prolog)**, **함수의 에필로그(epilog)**라고 하고 <u>함수의 호출 시점과 종료 시점</u>에 실행됩니다.

<br/>

먼저 <span style="color: red">**함수의 프롤로그(Prolog)**</span>란, 함수 호출 후 스택 프레임의 유지를 위해 시행되는 명령어 과정입니다. 명령어는 다음과 같이 구성되어 있습니다.

**(1) `push rbp`**

**(2) `mov rbp, rsp`**

- (1) - rbp 레지스터의 값을 stack에 `push`합니다. rbp 레지스터는 스택 프레임의 바닥(시작) 주소 값을 갖고 있는데, 이 주소 값을 **기준**으로 함수 내에서 여러 변수에 접근합니다. 그렇기 때문에 함수 내에서 또 다른 함수를 호출할 경우, 하나 뿐인 rbp 레지스터를 활용하기 위해서 <u>또 다른 함수 호출 전 rbp가 갖고 있던 값을 스택을 이용하여 저장해두어야 합니다.</u>

- (2) - 저장한 이후에는 함수가 호출되며 새롭게 생성된 스택 프레임을 이용하기 위해 마찬가지로 **기준**이 되는 <u>rbp 레지스터 값을 현재 스택의 top을 가리키는 rsp로 설정해야 합니다.</u>

<br />

그 다음으로 <span style="color: red">**함수의 에필로그(Epliog)**</span>란, 함수의 모든 동작을 마치고 함수를 호출한 지점으로 돌아가기 위해 시행되는 명령어 과정입니다. 명령어는 다음과 같이 구성되어 있습니다.

**(1) `leave`**

**(2) `ret`**

- (1) - `leave`는

    ```
    mov rsp, rbp
    pop rbp
    ```

    의 명령어와 같은 동작을 하는 명령어로, 해당 함수를 호출하며 기준이 되었던 rbp 레지스터의 값을 <u>이전 함수의 rbp 값</u>으로 재설정 해주는 과정입니다. rsp를 rbp 값으로 재설정 해주는 동작은 함수에서 사용했던 지역 변수를 **정리**한다는 의미를 가집니다.
    `pop rbp`를 통해 스택 top(현재 rsp가 가리키고 있는 곳, rbp)에 있는 값을 꺼내어 rbp 레지스터에 저장합니다.

    > 여기서, <u>이전 함수의 rbp</u>를 저장하는 곳을 **Stack Frame Pointer(SFP)**라고 부릅니다. 현재 스택 프레임의 기준을 세우면서, 이전 함수의 rbp 공간을 저장하는 역할을 하고 있습니다.

- (2) - `ret`은

    ```
    pop rip
    jump rip
    ```

    의 명령어와 같은 동작을 하는 명령입니다. `leave` 명령어를 통해 스택의 top이 **RET**를 가리키고 있게 되고, 이 값을 `pop` 하여 rip 레지스터에 저장함으로써 다음 명령줄인 `jump rip`를 통해 **RET**에 저장되었던 <u>현재 함수 호출 위치의 다음 명령어 line</u>으로 Instruction Pointer(명령어 포인터, rip, eip)를 이동시켜 프로그램의 실행을 변경합니다.

    > 여기서, **RET**란 Return Address를 의미합니다. 프로그램은 함수 호출 시 **Instruction Pointer(EIP/RIP)**를 이동시켜 호출한 함수의 명령어가 위치한 주소로 흐름을 바꾸게 되는데, 이때 함수 동작을 모두 마치고 함수를 호출한 명령어가 위치한 주소로 복귀해야합니다. 이를 위해, 함수 호출 전에 Stack에 복귀해야할 명령어 주소를 `push` 합니다. 이 주소가 위치한 곳이 **RET**로, **SFP**보다 높은 주소에 위치합니다.

    ![image](https://user-images.githubusercontent.com/37824335/224683249-7d7e2457-27ee-4173-b47a-b3f189787a96.png)

    예를 들어,

    ```
    0x1230	~
    0x1238	call	0x5555 <func1>
    0x1240 	~
    ```

    위와 같이 `0x1238` 에서 다른 함수를 호출한다면 호출 전에 `push 0x1240` 명령을 수행하여, stack을 통해 함수의 동작을 모두 마친 뒤에 복귀할 명령어 주소를 접근할 수 있습니다.


<br />

### 📌 지역 변수

다음으로는 함수 내에서 생성된 지역 변수를 어떻게 저장하고 접근하는지 알아봅시다.

![image](https://user-images.githubusercontent.com/37824335/224683417-df967759-4899-4961-86fe-60b2be74874f.png)

그림을 통해 설명하겠습니다.

<br>

![image](https://user-images.githubusercontent.com/37824335/224683443-9b45e747-04b5-4313-939e-a3841152a361.png)

- `<main+8>` : `rsp = rsp - 0x10`의 의미를 담고 있는 명령을 수행합니다. 이는 이후에 사용할 지역 변수를 위해 <u>stack을 확장</u>하는 역할을 합니다. <span style="color:red">빨간색 박스</span>가 확장된 stack 공간을 의미합니다.
- `<main+12>` : c코드에서 `a = 2;` 코드를 수행하는 명령으로, `rbp - 0x4` 위치에 `0x2`를 저장하는 의미의 명령을 수행합니다. *int* 타입 변수는 4byte이기 때문에, 4byte에 해당하는 공간만큼 `a`와 `b`가 공간을 차지하고 있습니다.
- `<main+19>` : <main+12>와 같은 역할의 명령으로, c코드에서 `b = 3;` 코드를 수행하는 명령으로, `rbp - 0x8` 위치에 `0x3`을 저장하는 의미의 명령을 수행합니다.

<br>

> **Assembly** <br>
> `type PTR expression` - *expression*을 *type*으로 형변환
{: .prompt-tip}

<span style="color:red">여기서 알 수 있듯이, 지역 변수는 base pointer(`rbp/ebp`)를 기준으로 하여 저장하고 접근합니다.</span>

<br />

### 📌 함수 호출 방식(함수 호출 규약)

![image](https://user-images.githubusercontent.com/37824335/224683906-f307c9dd-afaa-4c2b-82e8-b09134b7ef2c.png)

`<main+26>` ~ `<main+34>` 까지의 명령은 `add` 함수 호출 이전에 시행하여 <u>`add`의 인자</u>를 지정합니다.

함수를 호출할 때 함수를 호출하는 방식에 따라 인자는 어떻게 전달할 것인지, 사용한 스택은 누가 정리할 것인지 등이 달라집니다. 이를 **함수 호출 규약(Calling Convention)**이라고 합니다.

| 함수호출규약 | 사용 컴파일러 | 인자 전달 방식 | 스택 정리 | 적용 |
| --- | --- | --- | --- | --- |
|🔽 **x86**|||||
| stdcall | MSVC | Stack | Callee | WINAPI |
| <u>cdecl</u> | GCC, MSVC | Stack | Caller | 일반 함수 |
| fastcall | MSVC | ECX, EDX | Callee | 최적화된 함수 |
| thiscall | MSVC | ECX(인스턴스),Stack(인자) | Callee | 클래스의 함수 |
|🔽 **x64**|||||
| MS ABI | MSVC | RCX, RDX, R8, R9 | Caller | 일반 함수,Windows Syscall |
| <u>System ABI(SYSV)</u> | GCC | RDI, RSI, RDX, RCX, R8, R9, XMM0–7 | Caller | 일반 함수 |

<br />

저희는 *GCC*를 이용하여 컴파일 했기 때문에 **System ABI(SYSV)** 호출 규약을 지켜야 합니다. 다른 호출 규약에 따른 자세한 인자 전달 방식은 여기서 설명하지 않겠습니다.

전달해야 하는 인자가 `a`와 `b` 두 개이기 때문에 `rdi`와 `rsi` 레지스터를 이용해야 합니다.

- `<main+26>` : `rbp - 0x8`(`b` 변수)를 `edx` 레지스터에 저장.
- `<main+29>` : `rbp - 0x4`(`a` 변수)를 `eax` 레지스터에 저장.
- `<main+32>` : `esi` 레지스터에 `edx` 값을 저장.
- `<main+34>` : `edi` 레지스터에 `eax` 값을 저장.
- `<main+36>` : `0x401136` 주소에 위치하는 `add` 함수 호출

❗ 결과적으로, `a = 2`는 `edi`에 `b = 3`은 `esi`에 저장되었습니다.

<br>

> **`x64` 환경인데 `edx`, `eax`, `edi`, `esi`와 같은 레지스터를 사용하는 이유?**
예를 들어, `edx` 레지스터는 `rdx` 레지스터의 <u>하위 32bit와 호환</u>됩니다.
자세한 내용은 [이곳](https://velog.io/@1unaram/OS-Register#%EB%A0%88%EC%A7%80%EC%8A%A4%ED%84%B0-%ED%98%B8%ED%99%98)을 참고해주세요.
{: .prompt-tip}

<br />

이제 `add` 함수 내부를 들여다봅시다.

```bash
pwndbg> disassemble add
```

![image](https://user-images.githubusercontent.com/37824335/224684107-232784ca-cf8a-4449-bad2-bb221a158c87.png){: w="500" }

`add` 함수의 동작을 그림으로 나타내겠습니다.

<br>

![image](https://user-images.githubusercontent.com/37824335/224684135-74ce38d2-ebfa-4928-b005-696204f191d2.png)

- `add` 호출 전 : `main` 함수에서 `add` 함수를 호출하기 전에 <u>`main` 함수에서 호출한 명령어(`add(a, b)`) 그 다음 명령어 주소를 **RET** 공간에 저장합니다.</u> 이때, `rsp`는 `RET`를 가리킬 것입니다.

- 함수의 프롤로그

    - `<add+4>` : main 함수의 `rbp` 값을 stack에 저장해둡니다. `add` 함수가 종료되면 이 값을 꺼내어 `main`으로 복귀 시 `rbp`로 사용할 예정입니다.
    - `<add+5>` : 현재 `rsp`가 가리키는 위치를 `rbp`로 설정합니다.


`<add+8>` 부터는 지역 변수의 위치를 고려하여 `add` 함수의 스택 프레임을 다시 그려보겠습니다.

<br>

![image](https://user-images.githubusercontent.com/37824335/224684491-60d5307a-57af-4d84-8be5-b872e18eacee.png)

- `<add+ 8>` : `edi`(=2) 값을 `rbp - 0x14`에 저장합니다. 이는 매개 변수로 받은 `a = 2` 값을 `c`에 저장하는 과정으로 해석할 수 있습니다.
- `<add+11>` : `esi`(=3) 값을 `rbp - 0x18`에 저장합니다. 이는 매개 변수로 받은 `b = 3` 값을 `d`에 저장하는 과정을 해석할 수 있습니다.

- `e = c` 명령어 과정
    - `<add+14>` : `rbp - 0x14`(=2) 값을 `eax`에 저장합니다.
    - `<add+17>` : `eax`(=2) 값을 `rbp - 0x4`에 저장합니다.

- `f = d` 명령어 과정
    - `<add+20>` : `rbp - 0x18`(=3) 값을 `eax`에 저장합니다.
    - `<add+23>` : `eax`(=3) 값을 `rbp - 0x8`에 저장합니다.

- `c + d` 명령어 과정
    - `<add+26>` : `rbp - 0x4`(=2) 값을 `edx`에 저장합니다.
    - `<add+29>` : `rbp - 0x8`(=3) 값을 `eax`에 저장합니다.
    - `<add+32>` : `eax = eax + edx` 과정을 수행합니다. `return e + f`에서 반환 값인 `e + f` 값은 `eax` 레지스터에 저장되어 있고, `main` 함수에서 `eax`에 접근하여 반환 값을 사용할 예정입니다.

<br />

그렇다면 `c`와 `f` 변수 사이의 공간에는 어떤 값이 있을까요? 디버거를 통해 알아봅시다.

```bash
pwndbg> x/10x 0x7fffffffe110-0x20
```

![image](https://user-images.githubusercontent.com/37824335/224684584-f2318d8d-ae2c-44fa-9370-c461d35e4ccf.png)

`rbp`는 `0x7fffffffe110` 주소를 가리키고 있습니다. 두 변수 사이의 공간에는 아무 의미 없는 값인 쓰레기 값이 들어가 있습니다. 이렇게 dummy 값이 들어가 있는 이유에 대해서는 공부하면서 점차 보완하여 작성하겠습니다. 현재 저희는 저 값을 사용하지 않기 때문에 무시하고 넘어가도록 하겠습니다.

<br />

- 함수의 에필로그
    - `<add+34>` : `leave` 역할을 하는 코드가 존재해야 하나, `mov rsp, rbp` 명령어가 존재하지 않습니다. 이는 `add` 함수 내에서 stack을 확장하지 않았기 때문에 `rsp` 값이 함수의 시작 시점의 값과 동일하기 때문에 스택을 정리할 필요가 없기 때문입니다. main 함수의 rbp를 저장하고 있는 **SFP**에서 값을 꺼내어 rbp를 설정해줍니다.
    - `<add+35>` : **RET**에 저장되어 있던 복귀해야할 명령어 주소를 꺼내어 `jump` 합니다. `main` 함수로 돌아가는 셈입니다.

<br />

> **Assembly**
`rbp - 0x14` 값을 바로 `rbp - 0x4`에 넣는 명령어를 사용하지 않는 이유는 Assembly의 `mov` 명령어는 적어도 하나의 피연산자는 memory이면 안되기 때문입니다.
{: .prompt-tip }

> 혹시 눈치를 채셨나요? 이미지 속 *매개 변수*는 어떻게 된 것일까요?
저희는 **System ABI(SYSV)** 규약을 준수하기 때문에 stack으로 인자를 전달하지 않고 레지스터로 인자를 전달하게 되고, *매개 변수* 공간은 필요 없는 것이지요. **cdecl**와 같은 규약은 stack으로 인자를 전달하기 때문에 <u>함수를 호출하기 전에 stack에 함수의 인자를 `push` 해야합니다.</u>
> <br />
> 다음 이미지는 gcc-multilib을 통해 x86 환경으로 컴파일하여 **cdecl** 호출 규약을 준수한 프로그램의 `add` 함수 호출 과정입니다. `push` 명령어를 통해 인자를 stack에 넣고, 함수를 호출하는 것을 확인할 수 있습니다.
>
> 🔽 `gcc -m32 -o add_32 add.c`
> ![image](https://user-images.githubusercontent.com/37824335/224684744-16efeffa-3a52-44f0-bba7-e65264d316ba.png)
{: .prompt-tip }

<br />

이제 `add` 함수를 빠져나와 `main` 함수로 복귀합시다.

![image](https://user-images.githubusercontent.com/37824335/224684779-ea7d5b5f-3b25-46e8-b87b-39153457ec7c.png){: w="600" }

- `<main+41>` : `esi`에 `eax` 값을 저장합니다. 이때, `eax`에는 `add` 함수의 반환 값인 `5`가 저장되어 있습니다. 이러한 방식으로 함수를 호출한 부분에서 함수의 반환 값에 접근합니다.
- `<main+43>` ~ `<main+55>` : `printf` 함수를 호출하기 위한 과정입니다. 여기서는 중요하지 않으니 넘어가도록 하겠습니다.

![image](https://user-images.githubusercontent.com/37824335/224684908-29128592-4f6a-4b7d-9cf9-82d51864e624.png)

- `<main+60>` : 사용한 `eax` 레지스터를 정리합니다.
- `<main+65>` ~ `<main+66>` : `main` 함수의 에필로그 부분입니다.

<br />

예제 분석을 통해 **Stack Frame**에 대해서 알아보았습니다. 다음으로는 **Stack Alignment**에 관해 내용을 추가하여 업데이트 할 예정입니다. 잘못된 부분이 있다면 댓글로 알려주시면 감사하겠습니다 😉

<br />

> **참고 자료** <br>
> [https://www.tcpschool.com/c/c_memory_stackframe](https://www.tcpschool.com/c/c_memory_stackframe) <br>
> [https://sy99.tistory.com/13](https://sy99.tistory.com/13) <br>
> [https://popcorntree.tistory.com/61](https://popcorntree.tistory.com/61) <br>
> [https://dreamhack.io/](https://dreamhack.io/)
