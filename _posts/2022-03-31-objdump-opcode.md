---
title: "[pwnable] Objdump Opcode 추출 명령어"
date: 2022-03-31 00:00:00
categories: [Study, Pwnable]
tags: [pwnable]
---

```bash
for i in $(objdump -d [file] | grep "^ " | cut -f 2); do echo -n \\x$i; done
```

- `for i in $( )` : **$( )** 내의 명령을 실행한 값을 반복하여 i로 접근
- `objdump -d [file_path]` : **[file]** 경로의 오브젝트 파일을 기계어로 역어셈블
- `grep "^ "` : 공백으로 시작하는 문자열 탐색
- `cut -f 2` : 문자열의 2번째 필드만 추출
- `do echo -n \\x$i` : **"\x" + 변수 i** 값을 줄바꿈 없이 출력
- `done` : 반복문 종료
