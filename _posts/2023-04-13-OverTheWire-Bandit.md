---
title: '[Wargame] Over The Wire - Bandit'
date: 2023-04-13 00:00:00
categories: [Write-Up]
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

# Level0 -> Level1
---

```shell
bandit0@bandit:~$ ls
readme
bandit0@bandit:~$ cat readme
NH2SXQwcBdpmTEzi3bvBHMM9H66vVXjL
```

<br>

# Level1 -> Level2
---

```shell
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat ./-
rRGizSaX8Mk1RTb1CNQoXTcYZWU6lgzi
```

<br>

# Level2 -> Level3
---

```shell
bandit2@bandit:~$ ls
spaces in this filename
bandit2@bandit:~$ cat 'spaces in this filename'
aBZ0W5EmUfAf7kHTQeOwd8bauFJ2lAiG
```

<br>

# Level3 -> Level4
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

# Level4 -> Level5
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

# Level5 -> Level6
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

# Level6 -> Level7
---

```shell
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2> /dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
z7WtoNQU2XfjmMtWA8u5rN4vzqu4v99S
```

<br>

# Level7 -> Level8  
---

```shell
bandit7@bandit:~$ cat data.txt | grep 'millionth'
millionth       TESKZC0XvTetK0S9xNwm25STk5iWrBvP
```

<br>

# Level8 -> Level9
---

```shell
bandit8@bandit:~$ sort data.txt | uniq -u
EN632PlfYiZbn3PhVK3XOGSlNInNE00t
```

<br>

# Level9 -> Level10
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

# Level10 -> Level11
---

```shell
bandit10@bandit:~$ base64 -d data.txt
The password is 6zPeziLdR2RKNdNYFNb6nVCKzphlXHBM
```

<br>

# Level11 -> Level12
---

```shell
bandit11@bandit:~$ cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'
The password is JVNBBFSmZwKKOP0XbFXOoW8chDz5yVRv
```

<br>

# Level12 -> Level13
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

# Level13 -> Level14
---

```shell
bandit13@bandit:~$ ls
sshkey.private
bandit13@bandit:~$ ssh bandit14@bandit.labs.overthewire.org -p 2220 -i sshkey.private

bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq
```

<br>

# Level14 -> Level15
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

# Level15 -> Level16
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

# Level16 -> Level17
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

# Level17 -> Level18
---

```shell
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< f9wS9ZUDvZoo3PooHgYuuWdawDFvGld2
---
> hga5tuuCLF6fFzUpnagiMN8ssu9LFrdg
```

<br>

# Level17 -> Level18
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

# Level18 -> Level19
---

```shell
bandit19@bandit:~$ ls
bandit20-do
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
VxCazJaVykI6W36BkBU0mJTCM8rR95XT
```

<br>

# Level20 -> Level21
---