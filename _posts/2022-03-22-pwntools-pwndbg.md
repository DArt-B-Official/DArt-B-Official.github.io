---
title: '[pwnable] pwntools & pwndbg'
date: 2022-03-22 00:00:00
categories: [Study, Pwnable]
tags: [pwnable, pwntools, pwndbg]
---

> **pwntools**와 **pwndbg**를 사용하며 공부하는 내용을 계속 업데이트 할 예정입니다<br>
> **Reference** : Dreamhack - [Tool: pwntools]
{: .prompt-info}

<br>

# <span style="color: blue" >\# pwntools</span>
---

## 🔎 pwntools란?
: 시스템 해킹을 수행하며 자주 사용하는 함수들을 구현 해놓은 파이썬 모듈

<br />

## 🔧 pwntools 설치 & import
```bash
# pwntools 설치
pip3 install pwntools # python 3.X
pip install pwntools  # python 2.X
```

```python
# pwntools import
>>> from pwn import *
```

<br />

## ⚡ 사용법
### 1. process & remote

- `process` : 로컬 바이너리를 대상으로 익스플로잇 할 때 사용하는 함수(**local**)
- `remote` : 원격 서버를 대상으로 익스플로잇 할 때 사용하는 함수(**nc**)

```python
# 'test' 로컬 바이너리를 대상으로 익스플로잇 수행
p = process('./test')

# 'test.com'의 123포트에서 실행 중인 프로세스를 대상으로 익스플로잇 수행
p = remote('test.com', 123)
```

<br />

### 2. send

- `send` : 데이터를 프로세스에 전송하기 위해 사용

```python
# 프로세스 p에 'A'를 입력
p.send('A')

# 프로세스 p에 'A'+'\n'을 입력
p.sendline('A')

# 프로세스 p가 'hello'를 출력하면, 'A'를 입력
p.sendafter('hello', 'A')

# 프로세스 p가 'hello'를 출력하면, 'A'+'\n'을 입력
p.sendlineafter('A')
```

<br />

### 3. recv

- `recv` : 프로세스에서 데이터를 받기 위해 사용

```python
# 프로세스 p가 출력하는 데이터를 최대 8바이트까지 받아서 data에 저장
data = p.recv(8)

# 프로세스 p가 출력하는 데이터를 개행문자를 만날 때까지 받아서 data에 저장
data = p.recvline()

# 프로세스 p가 출력하는 데이터를 8바이트만 받아서 data에 저장. 해당 바이트만큼의 데이터를 받지 못하면 계속 기다림
data = p.recvn(8)

# 프로세스 p가 출력하는 데이터가 'hello'를 출력할 때까지 받아서 data에 저장
data = p.recvuntill('hello')

# 프로세스 p가 출력하는 데이터를 프로세스가 종료될 때까지 받아서 data에 저장
data = p.recvall()
```

<br />

### 4. packing & unpacking
- `packing` : little-endian의 바이트 배열로 변경하는 함수
- `p32` : 32bit 패킹
- `p64` : 64bit 패킹

- `unpacking` : little-endian의 바이트 배열을 unpack하는 함수
- `u32` : 32bit 언패킹
- `u64` : 64bit 언패킹

```python
# packing
>>> str32 = 0x41424344
>>> str64 = 0x4142434445464748
>>>
>>> print(p32(str32))
b'DCBA'
>>> print(p64(str64))
b'HGFEDCBA'
>>>
>>> # unpacking
>>> str32 = "ABCD"
>>> str64 = "ABCDEFGH"
>>>
>>> print(hex(u32(str32)))
0x44434241
>>> print(hex(u64(str64)))
0x4847464544434241
```

<br />

### 5. interactive
- `interactive()` : remote 또는 process로 연결 후 셸을 획득하거나, 익스플로잇의 특정 상황에 직접 입력을 주면서 출력을 확인하고 싶을 때 사용하는 함수

```python
p.interactive()
```

<br />

### 6. ELF
: ELF 헤더에 기록되어 있는 각종 정보를 참조하는 함수

```python
e = ELF('./test')

# ./test에서 puts()의 PLT 주소를 찾아 저장
puts_plt = e.plt['puts']

# ./test에서 read()의 GOT 주소를 찾아 저장
read_got = e.got['read']
```

<br />

### 7. context.log
: 익스플로잇을 디버깅할 때 디버그의 편의를 돕는 로깅 기능의 함수.

```python
# 에러만 출력
context.log_level = 'error'

# 프로세스와 익스플로잇 간에 오가는 모든 데이터를 화면에 출력
context.log_level = 'debug'

# 비교적 중요한 정보들만 출력
context.log_level = 'info'
```

<br />

### 8. context.arch
: pwntools는 셸코드 생성, 어셈블, 디스어셈블 등의 기능이 대상의 아키텍처에 영향을 받는데, 이 아키텍처 정보를 프로그래머가 지정할 수 있게 하는 함수

```python
# x86-64 아키텍처
context.arch = "amd64"

# x86 아키텍처
context.arch = "i386"

# arm 아키텍처
context.arch = "arm"
```

<br />

### 9. shellcraft
: 공격에 필요한 셸코드를 쉽게 꺼내쓸 수 있는 기능

[pwnlib.shellcraft 공식 문서](https://docs.pwntools.com/en/stable/shellcraft.html#submodules)에서 아키텍처별 함수를 확인할 수 있음

<br />

### 10. asm
- `asm()` : 어셈블 기능으로, 미리 아키텍처 지정 필요

<br />

### 11. slog
: 주소 값과 같은 hex 값을 출력할 때 함수를 정의하여 사용.
```python
def slog(n, m): return success(": ".join([n, hex(m)]))
```
- `n` : 문자열
- `m` : int형 hex 값

<br>


# <span style="color: blue" >\# pwndbg</span>
---

## 사용법

- `gdb file` : 디버깅 시작

- `readelf file` : ELF 확인
    - Entry point address(진입점, EP) : ELF 실행 시 진입점의 값부터 프로그램 실행

- `start` : 진입점부터 프로그램 분석

- gdb 명령어 축약
    - `b` : break
    - `c` : continue
    - `r` : run
    - `si` : step into (서브루틴의 내부로 들어감)
    - `ni` : next instruction (서브루틴의 내부로 들어가지 않음)
    - `i` : info
    - `k` : kill
    - `pd` : pdisas

- `disassemble func` : 해당 함수가 반환될 때까지 디스어셈블하여 보여줌
    - `u` /`nearpc` : 가독성 있게 보여줌

- `finish` : step into로 들어간 함수 내부에서 함수의 끝까지 한 번에 실행

- `x` : 메모리 주소 검사(특정 주소에서 원하는 길이만큼의 데이터를 원하는 형식으로 인코딩하여 볼 수 있음)
    - Format letters
        - `o` : 8진법
        - `x` : 16진법
        - `z` : 16진법, 왼쪽에 0으로 패딩
        - `d` : 10진법
        - `u` : unsigned 10진법
        - `t` : 2진법
        - `f` : float
        - `a` : 주소
        - `i` : 명령어
        - `c` : ASCII 바이트
        - `s` : 문자열
    - Size letters
        - `b` : 1byte
        - `h` : 2byte(halfword)
        - `w` : 4byte(word)
        - `g` : 8byte(giant)

- `tele` : 메모리 덤프 기능

- `vmap` : 가상 메모리의 레이아웃을 보여줌

- `catch` : 특정 이벤트가 발생했을 때, 프로세스를 중지시킴

    e.g.) `pwndbg> catch syscall arch_prctl`

- `watch` : 특정 주소에 저장된 값이 변경되면 프로세스를 중단시킴

    e.g.) `pwndbg> watch *(0x7ffff7d7f740+0x28)`

- `b` : breakpoint 설정

    e.g.) `pwndbg> b*main` : `main` 함수에 중단점 설정

- `bt` : 스택 프레임의 백트레이스 출력
