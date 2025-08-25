---
title: '[pwnable] Over The Wire - Bandit'
date: 2023-04-13 00:00:00
categories: [Wargame, Pwnable]
tags: [wargame, overthewire, bandit]
published: true
---

# OverTheWire - Bandit
Bandit Site: [https://overthewire.org/wargames/bandit/](https://overthewire.org/wargames/bandit/)

<br>

> **Environment**: WSL2 Ubuntu 20.04.4 LTS<br>
> **Host**: bandit.labs.overthewire.org<br>
> **Port**: 2220
{: .prompt-info }

<br>

```shell
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

<br>

# # Level 1 ~ 5

## Level0 -> Level1
---

```shell
bandit0@bandit:~$ ls
readme
bandit0@bandit:~$ cat readme
NH2SXQwcBdpmTEzi3bvBHMM9H66vVXjL
```

<br>

## Level1 -> Level2
---

```shell
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat ./-
rRGizSaX8Mk1RTb1CNQoXTcYZWU6lgzi
```

<br>

## Level2 -> Level3
---

```shell
bandit2@bandit:~$ ls
spaces in this filename
bandit2@bandit:~$ cat 'spaces in this filename'
aBZ0W5EmUfAf7kHTQeOwd8bauFJ2lAiG
```

<br>

## Level3 -> Level4
---

```shell
bandit3@bandit:~$ ls
inhere
bandit3@bandit:~$ cd inhere/
bandit3@bandit:~/inhere$ ls -al
total 12
drwxr-xr-x 2 root    root    4096 Feb 21 22:03 .
drwxr-xr-x 3 root    root    4096 Feb 21 22:03 ..
-rw-r----- 1 bandit4 bandit3   33 Feb 21 22:03 .hidden
bandit3@bandit:~/inhere$ cat .hidden
2EW7BBsr6aMMoJ2HjW067dm8EgX26xNe
```

<br>

## Level4 -> Level5
---

```shell
bandit4@bandit:~$ ls
inhere
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls
-file00  -file02  -file04  -file06  -file08
-file01  -file03  -file05  -file07  -file09
bandit4@bandit:~/inhere$ cat ./-file00 ./-file01 ./-file02 ./-file03 ./-file04 ./-file05 ./-file06 ./-file07 ./-file08 ./-file09
... lrIWWI6bB37kxfiCQZqUdOIYfr6eEeqR ...
```

![image](https://user-images.githubusercontent.com/37824335/231701441-769becb3-f693-4a02-953c-1bcf800b8734.png)

<br>

# # Level 6 ~ 10

## Level5 -> Level6
---

```shell
bandit5@bandit:~$ ls
inhere
bandit5@bandit:~$ cd inhere/
bandit5@bandit:~/inhere$ find ./* -size 1033c
./maybehere07/.file2
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
P4L4vucdmLnm8I7Vl7jG1ApGSfjYKqJU
```

<br>

## Level6 -> Level7
---

```shell
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2> /dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
z7WtoNQU2XfjmMtWA8u5rN4vzqu4v99S
```

<br>

## Level7 -> Level8
---

```shell
bandit7@bandit:~$ cat data.txt | grep 'millionth'
millionth       TESKZC0XvTetK0S9xNwm25STk5iWrBvP
```

<br>

## Level8 -> Level9
---

```shell
bandit8@bandit:~$ sort data.txt | uniq -u
EN632PlfYiZbn3PhVK3XOGSlNInNE00t
```

<br>

## Level9 -> Level10
---

```shell
bandit9@bandit:~$ strings data.txt | grep -e '^='
=XeOh
=vb`
=I6a
========== password
========== is
========== G7w8LIi6J3kTb8A7j9LgrywtEUlyyp6s
```

<br>

# # Level 11 ~ 15

## Level10 -> Level11
---

```shell
bandit10@bandit:~$ base64 -d data.txt
The password is 6zPeziLdR2RKNdNYFNb6nVCKzphlXHBM
```

<br>

## Level11 -> Level12
---

```shell
bandit11@bandit:~$ cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'
The password is JVNBBFSmZwKKOP0XbFXOoW8chDz5yVRv
```

<br>

## Level12 -> Level13
---

```shell
bandit12@bandit:~$ mkdir /tmp/bandit
bandit12@bandit:~$ cp data.txt /tmp/bandit/data.txt
bandit12@bandit:~$ cd /tmp/bandit
bandit12@bandit:/tmp/bandit$ cat data.txt
00000000: 1f8b 0808 8c3f f563 0203 6461 7461 322e  .....?.c..data2.
00000010: 6269 6e00 0134 02cb fd42 5a68 3931 4159  bin..4...BZh91AY
00000020: 2653 5953 6696 8100 001b 7fff fbdb effb  &SYSf...........
00000030: b41f 6efa a7cb ebee fff3 b7ad 897d f77f  ..n..........}..
...
bandit12@bandit:/tmp/bandit$ cat data.txt | xxd -r > d.txt
bandit12@bandit:/tmp/bandit$ file d.txt
d.txt: gzip compressed data, was "data2.bin", last modified: Tue Feb 21 22:02:52 2023, max compression, from Unix, original size modulo 2^32 564
bandit12@bandit:/tmp/bandit$ mv d.txt data2.bin.gz
bandit12@bandit:/tmp/bandit$ gunzip data2.bin.gz
bandit12@bandit:/tmp/bandit$ file data2.bin
data2.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/bandit$ mv data2.bin data2.bin.bz2
bandit12@bandit:/tmp/bandit$ bunzip2 data2.bin.bz2
bandit12@bandit:/tmp/bandit$ file data2.bin
data2.bin: gzip compressed data, was "data4.bin", last modified: Tue Feb 21 22:02:52 2023, max compression, from Unix, original size modulo 2^32 20480
bandit12@bandit:/tmp/bandit$ mv data2.bin data4.bin.gz
bandit12@bandit:/tmp/bandit$ gunzip data4.bin.gz
bandit12@bandit:/tmp/bandit$ file data4.bin
data4.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/bandit$ tar -x -f data4.bin
bandit12@bandit:/tmp/bandit$ ls
data4.bin  data5.bin  data.txt
bandit12@bandit:/tmp/bandit$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/bandit$ tar -x -f data5.bin
bandit12@bandit:/tmp/bandit$ ls
data4.bin  data5.bin  data6.bin  data.txt
bandit12@bandit:/tmp/bandit$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/bandit$ mv data6.bin data6.bin.bz2
bandit12@bandit:/tmp/bandit$ bunzip2 data6.bin.bz2
bandit12@bandit:/tmp/bandit$ file data6.bin
data6.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/bandit$ tar -x -f data6.bin
bandit12@bandit:/tmp/bandit$ ls
data4.bin  data5.bin  data6.bin  data8.bin  data.txt
bandit12@bandit:/tmp/bandit$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Tue Feb 21 22:02:52 2023, max compression, from Unix, original size modulo 2^32 49
bandit12@bandit:/tmp/bandit$ mv data8.bin data9.bin.gz
bandit12@bandit:/tmp/bandit$ gunzip data9.bin.gz
bandit12@bandit:/tmp/bandit$ file data9.bin
data9.bin: ASCII text
bandit12@bandit:/tmp/bandit$ cat data9.bin
The password is wbWdlBxEir4CaE8LaPhauuOo6pwRmrDw
```

<br>

## Level13 -> Level14
---

```shell
bandit13@bandit:~$ ls
sshkey.private
bandit13@bandit:~$ ssh bandit14@bandit.labs.overthewire.org -p 2220 -i sshkey.private

bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq
```

<br>

## Level14 -> Level15
---

```shell
bandit14@bandit:~$ telnet localhost 30000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq
Correct!
jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt

Connection closed by foreign host.
```

<br>

# # Level 16 ~ 20

## Level15 -> Level16
---

```shell
bandit15@bandit:~$ openssl s_client localhost:30001
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN = localhost
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN = localhost
verify error:num=10:certificate has expired
notAfter=Apr 17 01:35:22 2023 GMT
verify return:1
depth=0 CN = localhost
notAfter=Apr 17 01:35:22 2023 GMT
verify return:1
---
...
---
read R BLOCK
jN2kgmIXJ6fShzhT2avhotn4Zcka6tnt
Correct!
JQttfApK4SeyHwDlI9SXGR50qclOAil1

closed
```

<br>

## Level16 -> Level17
---

```shell
bandit16@bandit:~$ nmap localhost -p31000-32000
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-17 11:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00012s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds
bandit16@bandit:~$ openssl s_client localhost:31790
bandit16@bandit:~$ openssl s_client localhost:31790
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN = localhost
...
read R BLOCK
JQttfApK4SeyHwDlI9SXGR50qclOAil1
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
...
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

closed
bandit16@bandit:~$ mkdir /tmp/rsa/
bandit16@bandit:~$ cd /tmp/rsa
bandit16@bandit:/tmp/rsa$ vim /tmp/rsa/sshkey.private # Write RSA Private key
bandit16@bandit:/tmp/rsa$ chmod 400 sshkey.private
bandit16@bandit:/tmp/rsa$ ssh bandit17@bandit.labs.overthewire.org -p 2220 -i sshkey.private

bandit17@bandit:~$ find /* -name 'bandit17' 2> /dev/null
/etc/bandit_pass/bandit17
/home/bandit17
bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
VwOSWtCA7lRKkTfbr2IDh6awj9RNZM5e
```

<br>

## Level17 -> Level18
---

```shell
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< f9wS9ZUDvZoo3PooHgYuuWdawDFvGld2
---
> hga5tuuCLF6fFzUpnagiMN8ssu9LFrdg
```

<br>

## Level18 -> Level19
---

이번 level은 특별하게도 `ssh bandit18@bandit.labs.overthewire.org -p 2220` 명령을 통해 ssh 연결 후 이전 레벨에서 얻은 password를 입력하면 ssh 연결이 바로 끊긴다. Level Goal에 적혀있듯이 `.bashrc` 파일이 수정되었기에 ssh를 통해 실행된 terminal이 영향을 받은 것이다. `ssh`는 원격 연결 이외에도 원격 코드 실행을 허용하는 점을 이용하면 된다.

```shell
ssh bandit18@bandit.labs.overthewire.org -p 2220 ls
...
bandit18@bandit.labs.overthewire.org's password:
readme
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
...
bandit18@bandit.labs.overthewire.org's password:
awhqfNnAbc1naukrpqDYcF95h7HoMTrC
```
<br>

## Level19 -> Level20
---

```shell
bandit19@bandit:~$ ls
bandit20-do
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
VxCazJaVykI6W36BkBU0mJTCM8rR95XT
```

<br>

# # Level 21 ~ 25

## Level20 -> Level21
---

```shell
bandit20@bandit:~$ ./suconnect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

`suconnect` 라는 프로그램은 특정 포트에 TCP 연결을 하고, 해당 포트로부터 correct password(bandit20 password)를 전송하면 다음 password(bandit21 password)를 주는 프로그램인듯 하다.

<br>

1. (1번 터미널)우선 nc listen으로 특정 포트를 열어두자
```shell
bandit20@bandit:~$ nc -l 12345
```

2. (2번 터미널)다른 터미널에서 bandit20에 접속한 후에 `suconnect`를 이용하여 12345 포트에 접속하자
```shell
bandit20@bandit:~$ ./suconnect 12345
```

3. (1번 터미널)listen 포트에서 bandit20 password를 입력하자
```shell
bandit20@bandit:~$ nc -l 12345
VxCazJaVykI6W36BkBU0mJTCM8rR95XT
```

4. (2번 터미널)password를 읽고, 다음 password를 전송한다는 문구를 확인한다
```shell
bandit20@bandit:~$ ./suconnect 12345
Read: VxCazJaVykI6W36BkBU0mJTCM8rR95XT
Password matches, sending next password
```

5. (1번 터미널)다음 password를 확인한다
```shell
bandit20@bandit:~$ nc -l 12345
VxCazJaVykI6W36BkBU0mJTCM8rR95XT
NvEJF7oVjkddltPSrdKEFOllh9V1IBcq
```

<br>

## Level21 -> Level22
---

```shell
bandit21@bandit:~$ ls /etc/cron.d
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit24       e2scrub_all  sysstat
cronjob_bandit17_root  cronjob_bandit23  cronjob_bandit25_root  otw-tmp-dir
bandit21@bandit:~$ cat /etc/cron.d/cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
bandit21@bandit:~$ /usr/bin/cronjob_bandit22.sh
chmod: changing permissions of '/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv': Operation not permitted
/usr/bin/cronjob_bandit22.sh: line 3: /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv: Permission denied
bandit21@bandit:~$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
WdDozAdTM2z9DiFEQ2mGlwngMfj4EZff
```
## Level22 -> Level23
---

```shell
bandit22@bandit:~$ ls /etc/cron.d
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit24       e2scrub_all  sysstat
cronjob_bandit17_root  cronjob_bandit23  cronjob_bandit25_root  otw-tmp-dir
bandit22@bandit:~$ cat /etc/cron.d/cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
bandit22@bandit:~$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
bandit22@bandit:/etc/cron.d$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
bandit22@bandit:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
QYw0Y2aiA672PsMmh9puTQuhoz8SyR2G
```

## Level23 -> Level24
---

```shell
bandit23@bandit:~$ ls /etc/cron.d
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit24       e2scrub_all  sysstat
cronjob_bandit17_root  cronjob_bandit23  cronjob_bandit25_root  otw-tmp-dir
bandit23@bandit:~$ cat /etc/cron.d/cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@bandit:~$ /usr/bin/cronjob_bandit24.sh
/usr/bin/cronjob_bandit24.sh: line 5: cd: /var/spool/bandit23/foo: No such file or directory
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname/foo || exit 1
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -rf ./$i
    fi
done
```

현재 `/usr/bin/cronjob_bandit24.sh` 스크립트가 cron에 의해 **매분**마다 bandit24 사용자 권한으로 실행된다.
해당 스크립트의 동작을 정리하면 다음과 같다.

1. `/var/spool/{현재 사용자 이름}/foo` 디렉터리로 이동한다.
    - `whoami`는 현재 사용자 이름을 출력하는 명령
2. 해당 디렉터리에서 `.`과 `..`을 제외한 모든 파일명에 대해 반복문을 수행한다.
    - `for i in * .*`에서 `*`는 모든 파일, `.*`은 숨긴 모든 파일을 의미
3. 반복문 내에서는 해당 파일명의 소유자가 "bandit23"과 같은지 확인한다
    - `stat --format "%U" ./$i` : `./$i` 파일의 소유자를 출력
4. bandit23과 같다면, 60초 이내에 해당 파일을 실행해야 한다.
    - `timeout -s 9 60 ./$i`에서 `-s 9`는 강제 종료 신호인 9번 신호(SIGKILL)를 이용한다는 의미

<br>

```shell
bandit23@bandit:~$ ls -al /var/spool/bandit24/
total 12
dr-xr-x--- 3 bandit24 bandit23 4096 Apr 23 18:04 .
drwxr-xr-x 5 root     root     4096 Apr 23 18:04 ..
drwxrwx-wx 3 root     bandit24 4096 Apr 24 15:22 foo
```

foo 디렉터리는 작성 권한이 있기 때문에 해당 디렉터리 내에 bandit24 password를 다른 파일로 출력할 수 있는 스크립트를 작성해야 한다.

<br>

```shell
bandit23@bandit:~$ vim /var/spool/bandit24/foo/shell.sh
```
 vim을 통해 아래의 내용을 포함한 스크립트 파일을 만들자.

```
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/bandit24_pass
```

<br>

파일 생성 후, bandit24 사용자가 해당 파일을 실행할 수 있도록 권한을 설정 해준다.

```shell
bandit23@bandit:~$ chmod 777 /var/spool/bandit24/foo/shell.sh
```

<br>

cron이 `cronjob_bandit24.sh`를 실행할 때까지 잠시 기다린 후에, password를 복사한 파일을 읽으면 password를 획득할 수 있다.

```shell
bandit23@bandit:~$ cat /tmp/bandit24_pass
VAfGXJ1PBSsPSnvsjI8p759leLZ9GGar
```

<br>

## Level24 -> Level25
---

```shell
bandit24@bandit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
VAfGXJ1PBSsPSnvsjI8p759leLZ9GGar 0000
Wrong! Please enter the correct pincode. Try again.
```

위와 같이 30002 포트에서 pincode를 체크하는 데몬이 실행 중이다. 문제 설명대로 0000부터 9999까지 브루트포싱을 수행하는 방법 밖에 없다. shell script를 작성해보자.

<br>

```shell
bandit24@bandit:~$ mkdir /tmp/my_bandit24
bandit24@bandit:~$ cd /tmp/my_bandit24
bandit24@bandit:/tmp/my_bandit24$ vim brute_force.sh
bandit24@bandit:/tmp/my_bandit24$ cat brute_force.sh
#!/bin/bash

for i in {0000..9999}
        do
                echo "VAfGXJ1PBSsPSnvsjI8p759leLZ9GGar $i"
        done

bandit24@bandit:/tmp/my_bandit24$ ./brute_force.sh | nc localhost 30002 | grep -v 'Wrong'
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
Correct!
The password of user bandit25 is p7TaowMYrmu23Ol8hiZh9UvD0O9hpx8d

Exiting.
```

<br>

# # Level 26 ~ 30

## Level25 -> Level26
---

```shell
bandit25@bandit:~$ ls
bandit26.sshkey
bandit25@bandit:~$ ssh bandit26@localhost -i bandit26.sshkey -p 2220
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
...
  Enjoy your stay!

  _                     _ _ _   ___   __
 | |                   | (_) | |__ \ / /
 | |__   __ _ _ __   __| |_| |_   ) / /_
 | '_ \ / _` | '_ \ / _` | | __| / / '_ \
 | |_) | (_| | | | | (_| | | |_ / /| (_) |
 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/
Connection to localhost closed
```

홈 디렉터리에 있는 sshkey 파일을 이용하여 bandit26 사용자로 접속을 시도하면, 접속이 되자마자 끊긴다. 문제에서 주어진 것처럼 bandit26의 shell을 살펴보자.

<br>

```shell
bandit25@bandit:~$ cat /etc/passwd | grep 'bandit25'
bandit25:x:11025:11025:bandit level 25:/home/bandit25:/bin/bash
bandit25@bandit:~$ cat /etc/passwd | grep 'bandit26'
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
bandit25@bandit:~$ ls -al /usr/bin/showtext
-rwxr-xr-x 1 root root 58 Apr 23 18:04 /usr/bin/showtext
bandit25@bandit:~$ cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0
```

bandit26 사용자는 bandit25 사용자와 달리 `/bin/bash`  쉘을 사용하지 않고 `/usr/bin/showtext`라는 파일을 사용하고 있음을 알 수 있다. 해당 파일은 기타 사용자도 읽고 실행할 수 있는 권한을 갖고 있었고, 파일을 출력해보았다.

`/usr/bin/showtext` 파일에서 `exec more ~/text.txt` 명령 부분을 살펴볼 수 있는데, 이는 bandit26의 홈 디렉터리에 존재하는 `text.txt`라는 파일을 `more` 명령어로 실행 후에 종료(`exit 0`)함을 알 수 있다.

`more` 명령어는 파일을 현재 터미널 크기에 맞게끔 출력하는 명령어인데, `text.txt` 파일을 해당 명령어로 읽고 있으므로, 터미널 창을 조절하여 프로그램을 바로 종료하지 않도록 할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/37824335/234309552-40d44bc3-b68e-4076-8460-af7a17d839a9.png)

위처럼 터미널 크기를 줄이면 `more` 명령어가 실행 중인 것을 확인할 수 있고, `more`의 옵션 중 `v`키를 누르면 vi 에디터를 실행할 수 있다.

이제 vi 에디터의 기능 중 명령 모드에서 `:r {파일명}`을 입력하면 외부 파일을 해당 파일에 삽입할 수 있으므로 `/etc/bandit_pass/bandit26` 파일 내용을 삽입하여 보자.

<br>

![image](https://user-images.githubusercontent.com/37824335/234311095-7b28ec4d-1169-4ba8-909c-badff4ec007b.png)

위와 같이 password `c7GvcKlw9mC7aUQaPx7nwFstuAIBw1o1` 를 읽을 수 있다.

<br>

## Level26 -> Level27
---

문제에 접근하기 위해서는 bandit26 사용자로 접속하여 shell을 실행해야 한다. 이전 레벨 단계에서 vim에 접속까지 한다음에 명령 모드에서 `:set shell=/bin/bash` 명령을 입력한다. 이제 `:sh` 명령을 입력하면 bash shell을 실행할 수 있다.

<br>

```shell
bandit26@bandit:~$ ls
bandit27-do  text.txt
bandit26@bandit:~$ ./bandit27-do cat /etc/bandit_pass/bandit27
YnQpBuifNMas1hcUFk70ZmqkhUU2EuaS
```

<br>

## Level27 -> Level28
---

```shell
bandit27@bandit:~$ mkdir /tmp/bandit27
bandit27@bandit:~$ cd /tmp/bandit27
bandit27@bandit:/tmp/bandit27$ git clone ssh://bandit27-git@localhost/home/bandit27-git/repo
Cloning into 'repo'...
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:C2ihUBV7ihnV1wUXRb4RrEcLfXC5CXlhmAAM/urerLY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? ^[[A^[[A^[[A^C
bandit27@bandit:/tmp/bandit27$ git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
Cloning into 'repo'...
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
ED25519 key fingerprint is SHA256:C2ihUBV7ihnV1wUXRb4RrEcLfXC5CXlhmAAM/urerLY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Could not create directory '/home/bandit27/.ssh' (Permission denied).
Failed to add the host to the list of known hosts (/home/bandit27/.ssh/known_hosts).
                         _                     _ _ _
                        | |__   __ _ _ __   __| (_) |_
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_
                        |_.__/ \__,_|_| |_|\__,_|_|\__|


                      This is an OverTheWire game server.
            More information on http://www.overthewire.org/wargames

bandit27-git@localhost's password:
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
bandit27@bandit:/tmp/bandit27$ ls
repo
bandit27@bandit:/tmp/bandit27$ ls repo
README
bandit27@bandit:/tmp/bandit27$ cat repo/README
The password to the next level is: AVanL161y9rsbcJIsFHuw35rjaOM19nR
```

<br>

## Level28 -> Level29
---

```shell
bandit28@bandit:~$ mkdir /tmp/bandit28
bandit28@bandit:~$ cd /tmp/bandit28
bandit28@bandit:/tmp/bandit28$ git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo
Cloning into 'repo'...
...
bandit28-git@localhost's password:
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 9 (delta 2), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (9/9), done.
Resolving deltas: 100% (2/2), done.
bandit28@bandit:/tmp/bandit28$ ls
repo
bandit28@bandit:/tmp/bandit28$ cd repo
bandit28@bandit:/tmp/bandit28/repo$ ls
README.md
bandit28@bandit:/tmp/bandit28/repo$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
```

현재 README.md는 password가 지워진 상태이다. git log를 확인해보면

<br>

```shell
bandit28@bandit:/tmp/bandit28/repo$ git log
commit 899ba88df296331cc01f30d022c006775d467f28 (HEAD -> master, origin/master, origin/HEAD)
Author: Morla Porla <morla@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    fix info leak

commit abcff758fa6343a0d002a1c0add1ad8c71b88534
Author: Morla Porla <morla@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    add missing data

commit c0a8c3cf093fba65f4ee0e1fe2a530b799508c78
Author: Ben Dover <noone@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    initial commit of README.md
```

세 개의 commit을 확인할 수 있고, 그 중 두 번째 커밋에 data가 있을 것 같다. checkout 명령어로 커밋 시점으로 이동하자.

<br>

```shell
bandit28@bandit:/tmp/bandit28/repo$ git checkout abcff758fa6343a0d002a1c0add1ad8c71b88534
Previous HEAD position was c0a8c3c initial commit of README.md
HEAD is now at abcff75 add missing data
bandit28@bandit:/tmp/bandit28/repo$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: tQKvmcwNYcFS6vmPHIUSI3ShmsrQZK8S
```

<br>

## Level29 -> Level30
---

```shell
bandit29@bandit:~$ mkdir /tmp/bandit29
bandit29@bandit:~$ cd /tmp/bandit29
bandit29@bandit:/tmp/bandit29$ git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo
Cloning into 'repo'...
...
Resolving deltas: 100% (2/2), done.
bandit29@bandit:/tmp/bandit29$ ls
repo
bandit29@bandit:/tmp/bandit29$ cd repo
bandit29@bandit:/tmp/bandit29/repo$ ls
README.md
bandit29@bandit:/tmp/bandit29/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
bandit29@bandit:/tmp/bandit29/repo$ git log
commit 4bd5389f9f2b9e96ba517aa751ee58d051905761 (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Sun Apr 23 18:04:40 2023 +0000

    fix username

commit 1a57cf10158f133c4f40ff82251f605a7618631d
Author: Ben Dover <noone@overthewire.org>
Date:   Sun Apr 23 18:04:40 2023 +0000

    initial commit of README.md
bandit29@bandit:/tmp/bandit29/repo$ git checkout 1a57cf10158f133c4f40ff82251f605a7618631d
Note: switching to '1a57cf10158f133c4f40ff82251f605a7618631d'.
...
HEAD is now at 1a57cf1 initial commit of README.md
bandit29@bandit:/tmp/bandit29/repo$ ls
README.md
bandit29@bandit:/tmp/bandit29/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit29
- password: <no passwords in production!>
```

이전 레벨과는 다르게 이전 commit으로 커밋 시점을 이동하여도 password를 확인할 수 없다. password 필드에서 확인할 수 있는 것처럼 production 브랜치는 password를 제외시킨 것 같다.

그렇다면 원격 저장소의 development용 브랜치를 찾아 password를 확인하자.

<br>

```shell
bandit29@bandit:/tmp/bandit29/repo$ git branch -r
  origin/HEAD -> origin/master
  origin/dev
  origin/master
  origin/sploits-dev
bandit29@bandit:/tmp/bandit29/repo$ git checkout -t origin/dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
Switched to a new branch 'dev'
bandit29@bandit:/tmp/bandit29/repo$ git branch
* dev
  master
bandit29@bandit:/tmp/bandit29/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: xbhV3HpNGlTIdnjUrdAlPzc2L6y9EOnS
```


<br>

# # Level 31 ~

## Level30 -> Level31
---

```shell
bandit30@bandit:~$ mkdir /tmp/bandit30
bandit30@bandit:~$ cd /tmpbandit30
-bash: cd: /tmpbandit30: No such file or directory
bandit30@bandit:~$ cd /tmp/bandit30
bandit30@bandit:/tmp/bandit30$ git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo
Cloning into 'repo'...
...
Receiving objects: 100% (4/4), done.
bandit30@bandit:/tmp/bandit30$ ls
repo
bandit30@bandit:/tmp/bandit30$ cd repo
bandit30@bandit:/tmp/bandit30/repo$ ls
README.md
bandit30@bandit:/tmp/bandit30/repo$ cat README.md
just an epmty file... muahaha
bandit30@bandit:/tmp/bandit30/repo$ git log
commit 59530d30d299ff2e3e9719c096ebf46a65cc1424 (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Sun Apr 23 18:04:42 2023 +0000

    initial commit of README.md
bandit30@bandit:/tmp/bandit30/repo$ git branch
* master
bandit30@bandit:/tmp/bandit30/repo$ git branch -r
  origin/HEAD -> origin/master
  origin/master
```

이전 레벨과는 다르게 변경 사항, 브랜치 등의 주요 내용이 보이지 않는다.

git에서 또 다른 내용을 기입할 수 있는 기능 중 태그(tag)에 대해 조회해보자.

<br>

```shell
bandit30@bandit:/tmp/bandit30/repo$ git tag
secret
bandit30@bandit:/tmp/bandit30/repo$ git show secret
OoffzGDlzhAlerFJ2cAiz1D41JW1Mhmt
```

<br>

## Level31 -> Level32
---

```shell
bandit31@bandit:~$ mkdir /tmp/bandit31
bandit31@bandit:~$ cd /tmp/bandit31
bandit31@bandit:/tmp/bandit31$ git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo
Cloning into 'repo'...
...
Receiving objects: 100% (4/4), done.
bandit31@bandit:/tmp/bandit31$ ls
repo
bandit31@bandit:/tmp/bandit31$ cd repo
bandit31@bandit:/tmp/bandit31/repo$ ls
README.md
bandit31@bandit:/tmp/bandit31/repo$ cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master

bandit31@bandit:/tmp/bandit31/repo$ git branch
* master
bandit31@bandit:/tmp/bandit31/repo$ echo 'May I come in?' > key.txt
bandit31@bandit:/tmp/bandit31/repo$ git add -f key.txt
bandit31@bandit:/tmp/bandit31/repo$ git commit -m 'Add file'
[master 0c159d9] Add file
 1 file changed, 1 insertion(+), 1 deletion(-)
bandit31@bandit:/tmp/bandit31/repo$ git push
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
...
Total 6 (delta 1), reused 0 (delta 0), pack-reused 0
remote: ### Attempting to validate files... ####
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote: Well done! Here is the password for the next level:
remote: rmCBvG56y58BXzv98yZGdO7ATVL5dW8y
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote: Wrong!
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
To ssh://localhost:2220/home/bandit31-git/repo
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'ssh://localhost:2220/home/bandit31-git/repo'
```

<br>

## Level32 -> Level33
---

```shell
lunaram@DESKTOP-6FMR5E6:/mnt/c/Users/Rami$ ssh bandit32@bandit.labs.overthewire.org -p2220
                         _                     _ _ _
                        | |__   __ _ _ __   __| (_) |_
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
...
WELCOME TO THE UPPERCASE SHELL
>> ls
sh: 1: LS: not found
```

위와 같이 shell이 실행되는데, uppercase shell이라고 한 것처럼 명령을 대문자로 바꾸어 인식이 되지 않는다.

<br>

```shell
>> !@#123
sh: 1: !@#123: not found
```

특수기호 및 숫자는 변환이 되지 않는데, 이를 이용하여 일반 shell을 의미하는 `$0` 변수를 입력하여 shell을 실행하여 password를 읽어내자.

<br>

```shell
>> $0
$ ls
uppershell
$ pwd
/home/bandit32
$ whoami
bandit33
$ cat /etc/bandit_pass/bandit33
odHo63fHiFqcWWJG9rLiLDtPm45KzUKy
```
