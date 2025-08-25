---
title: "[Dreamhack] Background - Computer Science"
date: 2022-07-11 00:00:00
categories: [CS]
tags: [assembly, dreamhack, quiz]
published: True
---

# 🔸 Linux Memory Layout

---

[[Study]Memory Layout](https://1unaram.github.io/posts/memory-layout/)

<br>

## Quiz: Linux Memory Layout

```c
#include <stdlib.h>
int a = 0xa;
const char b[] = "d_str";
int c;
int foo(int arg) {
  int d = 0xd;
  return 0;
}
int main()
{
  int *e = malloc(sizeof(*e));
  return 0;
}
```

<br>

**Q1. a가 위치하는 세그먼트는 어디인가?**

A. 데이터 세그먼트

→ 데이터 세그먼트에는 컴파일 시점에 정해지는 전역변수 및 전역 상수가 위치. a는 전역변수.

<br>

**Q2. b가 위치하는 세그먼트는 어디인가?**

A. 데이터 세그먼트

→ b는 전역 상수.

<br>

**Q3. foo가 위치하는 세그먼트는 어디인가?**

A. 코드 세그먼트

→ 실행 가능한 함수의 코드 데이터는 코드 세그먼트에 위치.

<br>

**Q4. "d_str"이 위치하는 세그먼트는 어디인가?**

A. 읽기 전용 데이터 (rodata) 세그먼트

→ 전역 상수와 같은 데이터는 데이터 세그먼트 중에서도 쓰기 권한이 부여되지 않은 세그먼트에 저장됨.

<br>

**Q5. d가 위치하는 세그먼트는 어디인가?**

A. 스택 세그먼트

→ 함수의 인자, 지역변수, 매개변수와 같은 임시 변수들은 프로그램 중에 동적으로 할당 받아 스택 세그먼트에 저장됨.

<br>

**Q6. c가 위치하는 세그먼트는 어디인가?**

A. BSS 세그먼트

→ 초기화하지 않은 전역 변수처럼 컴파일 시점에 값이 정해지지 않는 데이터는 BSS 세그먼트 저장됨. 프로그램 시작 시 0으로 초기화.

<br>

**Q7. e는 어느 세그먼트의 데이터를 가리키는가?**

A. 힙 세그먼트

→ `malloc()`, `calloc()`과 같은 함수를 통해 프로그램 중 동적으로 힙 세그먼트에 할당 받음.

<br>

# 🔸 Computer Architecture

---

- **명령어 집합구조(Instruction Set Architecture, ISA)** : 컴퓨터 구조 중에서 CPU가 사용하는 명령어와 관련된 설계. ex) 인텔의 x86-64 아키텍처

<br>

- 컴퓨터 구조의 세부 분야
    - 기능 구조의 설계
        - 폰 노이만 구조
        - 하버드 구조
        - 수정된 하버드 구조

    - 명령어 집합구조
        - x86, x86-64
        - ARM
        - MIPS
        - AVR

    - 마이크로 아키텍처
        - 캐시 설계
        - 파이프라이닝
        - 슈퍼 스칼라
        - 분기 예측
        - 비순차적 명령어 처리

    - 하드웨어 및 컴퓨팅 방법론
        - 직접 메모리 접근

<br>

- 폰 노이만 구조

    ![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/4d06633b-3b2c-42cd-b9aa-cd6e0d6066e7){: w="500" }

    연산과 제어는 중앙처리장치(CPU), 저장은 기억장치(memory), 장치간에 데이터나 제어 신호 교환은 버스(bus)를 사용.

    <br>

    - 중앙처리장치
        - 산술논리장치(ALU) - 산술/논리 연산 처리
        - 제어장치(Control Unit) - CPU 제어
        - 레지스터(Register) - CPU에 필요한 데이터를 저장

    - 기억장치
        - 주기억장치 : 프로그램 실행 과정에서 필요한 데이터 임시 저장에 사용
            1. 램(Random-Access Memory, RAM)
        - 보조기억장치 : 운영체제, 프로그램과 같은 데이터 장기간 보관에 사용
            1. 하드 드라이브
            2. SSD

    - 버스
        - 데이터 버스(Data Bus)
        - 주소 버스(Address Bus)
        - 제어 버스(Control Bus)
        - 랜선
        - 프로토콜

    <br>

    > **레지스터가 존재함에도 CPU 안에 레지스터가 필요한 이유**<br>
    : CPU의 연산속도가 기억장치와의 데이터 교환속도보다 압도적으로 빠르기 때문에, 기억장치만을 사용하면 병목현상이 발생. 따라서 교환속도를 단축하기 위해 *레지스터*와 *캐시*라는 저장장치를 내부에 갖고 있음.
    {: .prompt-tip}

<br>

- x86-64 아키텍처

    [[Study] Register](https://1unaram.github.io/posts/Register/#-x64-%ED%99%98%EA%B2%BD64bit)

<br>

# 🔶 x86 Assembly

---

[[Study] Assembly](https://1unaram.github.io/posts/Assembly/)

<br>

## Quiz: x86 Assembly 1

```
[Register]
rbx = 0x401A40

=================================

[Memory]
0x401a40 | 0x0000000012345678
0x401a48 | 0x0000000000C0FFEE
0x401a50 | 0x00000000DEADBEEF
0x401a58 | 0x00000000CAFEBABE
0x401a60 | 0x0000000087654321

=================================

[Code]
1: mov rax, [rbx+8]
2: lea rax, [rbx+8]
```

**Q1. 레지스터, 메모리 및 코드가 다음과 같다. Code를 1까지 실행했을 때, rax에 저장된 값은?**

→ 0xC0FFEE

<br>

**Q2. Code를 2까지 실행했을 때, rax에 들어있는 값은?**

→ 0x401a48

<br>

```
[Register]
rax = 0x31337
rbx = 0x555555554000
rcx = 0x2

=================================

[Memory]
0x555555554000| 0x0000000000000000
0x555555554008| 0x0000000000000001
0x555555554010| 0x0000000000000003
0x555555554018| 0x0000000000000005
0x555555554020| 0x000000000003133A

==================================

[Code]
1: add rax, [rbx+rcx*8]
2: add rcx, 2
3: sub rax, [rbx+rcx*8]
4: inc rax
```

**Q3. 레지스터, 메모리 및 코드가 다음과 같다. Code를 1까지 실행했을 때, rax에 저장된 값은?**

→ 0x3133A

<br>

**Q4. Code를 3까지 실행했을 때, rax에 저장된 값은?**

→ 0

<br>

**Q5. Code를 4까지 실행했을 때, rax에 저장된 값은?**

→ 1

<br>

```
[Register]
rax = 0xffffffff00000000
rbx = 0x00000000ffffffff
rcx = 0x123456789abcdef0

==================================

[Code]
1: and rax, rcx
2: and rbx, rcx
3: or rax, rbx
```

**Q6. 레지스터, 메모리 및 코드가 다음과 같다. Code를 1까지 실행했을 때, rax에 저장된 값은?**

→ 0x1234567800000000

<br>

**Q7. Code를 2까지 실행했을 때, rbx에 저장된 값은?**

→ 0x000000009ABCDEF0

<br>

**Q8. Code를 3까지 실행했을 때, rax에 저장된 값은?**

→ 0x123456789ABCDEF0

<br>

```
[Register]
rax = 0x35014541
rbx = 0xdeadbeef

==================================

[Code]
1: xor rax, rbx
2: xor rax, rbx
3: not eax
```

**Q9. 레지스터, 메모리 및 코드가 다음과 같다. Code를 1까지 실행했을 때, rax에 저장된 값은?**

→ 0xEBACFBAE

<br>

**Q10. Code를 2까지 실행했을 때, rax에 저장된 값은?**

→ 0x35014541

<br>

**Q11. Code를 3까지 실행했을 때, rax에 저장된 값은?**

→ 0xCAFEBABE

<br>

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

**Q12. end로 점프하면 프로그램이 종료된다고 가정하자. 프로그램이 종료됐을 때, 0x400000 부터 0x400019까지의 데이터를 대응되는 아스키 문자로 변환하면?**


```python
# solve.py

mem = [0x67,0x55,0x5c,0x53,0x5f,0x5d,0x55,0x10,
       0x44,0x5f,0x10,0x51,0x43,0x43,0x55,0x5d,
       0x52,0x5c,0x49,0x10,0x47,0x5f,0x42,0x5c,
       0x54,0x11,0x00,0x00,0x00,0x00,0x00,0x00]

for i in range(0x20):
    print(chr(mem[i]^0x30), end="")
```

→ Welcome to assembly world!

<br>

## Quiz: x86 Assembly 2

[[Dreamhack] Quiz: x86 Assembly 2](https://1unaram.github.io/posts/quiz-assembly2/)

<br>

## Quiz: x86 Assembly 3

[[Dreamhack] Quiz: x86 Assembly 3](https://1unaram.github.io/posts/quiz-assembly3)
