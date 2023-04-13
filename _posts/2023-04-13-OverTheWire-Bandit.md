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

```