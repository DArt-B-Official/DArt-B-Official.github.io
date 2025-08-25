---
title: '[pwnable] System Hacking 노트'
date: 2025-02-06 00:00:00
categories: [Study, Pwnable]
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


<br>

# gdb.attach()

---

WSL에서는 python 코드 내에서 `gdb.attach(p)`를 실행해도 터미널을 선택하라는 문구와 함께 실행되지 않는다. 이는 python 익스플로잇 파일을 실행할 때의 pid를 인자로 하여 직접 gdb를 실행해야 한다.

1. 디버깅하고 싶은 부분에서 `pause()` 구문 추가

    주로, 페이로드 전송 전에서 사용한다.

2. `python file.py`를 통해 익스플로잇 파일 실행

    실행 시 `[+] Starting local process './basic_rop_x86': pid 317`와 같이 pid 번호가 출력된다

3. 앞에서 얻은 pid를 인자로 하여 새로운 터미널에서 `gdb -p 317` 실행

4. 내가 걸고자 하는 곳에 브레이크포인트 설정

    `b * main+40`

5. gdb에서 `c`를 입력하여 continue

6. 파이썬을 실행한 터미널에서 아무 키나 눌러서 pause 해제

7. gdb를 확인하면 브레이크를 설정한 주소에서 멈춘 것을 확인할 수 있음

