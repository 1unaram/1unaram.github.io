---
title: '[Write-Up] 2025 Fiesta - Web Treasure Hunt'
date: 2025-09-02 00:00:00
categories: [Write-Up, Web]
tags: []
published: True
---

# #Intro

2025 Fiesta 금융보안 위협분석 대회에서 출제된 Web Treasure Hunt 문제를 풀다가 Git에 대해 새로운 경험을 하게 되어 기록하고자 Write Up으로 정리하게 되었다.

<br>

<br>

# #Challenge

![image](/assets/posts/250926-1.png)

문제는 주어진 URL에 접속하여 취약점을 발견하고 플래그를 획득하는 것이었다.

<br>

## 웹 서비스 탐색

![image](/assets/posts/250926-2.png)

웹 서비스는 URL을 통해 php로 작성된 간단한 형태의 사이트였고, 공격 벡터로 볼 수 있는 별다른 기능은 존재하지 않았다.

<br>


문제 설명에서 확인할 수 있듯이 숨겨진 서버의 구성 파일을 찾아야 했기에, `/robots.txt` 파일을 확인해보았다.

<br>

```
User-agent: *

Allow: /
Allow: /index.php
Allow: /notice.php
Allow: /events.php
Allow: /products.php
Allow: /exchange.php
Allow: /css/

Disallow: /attachments/flag.txt
Disallow: /.git/

Crawl-delay: 1
```

해당 파일에서 flag 파일의 위치를 확인할 수 있었고, `/.git` 디렉터리가 존재하는 것을 확인할 수 있었다. `/attachments/flag.txt` 파일은 Access Denied 메시지와 함께 접근이 불가능하였으나, `/.git` 경로는 접근 차단이 되어있지 않아 디렉터리 내용을 확인할 수 있었다.

<br>

## .git 디렉터리 분석

![image](/assets/posts/250926-3.png)

`.git` 디렉터리에는 Git 저장소의 모든 버전 관리 정보가 담겨있다. 사실 평상시에 `.git` 디렉터리를 열어본 적이 없었기에, 이번 기회에 구조를 확인할 수 있었다.

<br>

![image](/assets/posts/250926-4.png)

모든 파일을 열어보면서 어떤 내용이 들어있는지 확인하였고, 그 중 `.git/objects/pack` 디렉터리에 `.idx`와 `.pack`, `.rev` 파일에 많은 내용이 존재하는 것을 확인할 수 있었다.

<br>

Git의 `.pack` 파일은 객체를 압축하여 저장한 파일로, 단독으로 읽을 수 없기에 Git 도구를 이용해서 확인해야 한다. `.idx` 파일은 `.pack` 파일의 인덱스 역할을 하며, `.rev` 파일은 리비전 정보를 담고 있다.

<br>

핵심은 `.pack` 파일을 분석하는 것인데, `idx` 파일이 인덱스 역할을 하여 `.pack` 파일을 빠르게 읽을 수 있도록 도와준다.

<br>

이를 위해 우선 `wget` 도구를 이용하여 3개의 파일을 모두 다운로드하였다.

<br>

![image](/assets/posts/250926-5.png)

가장 먼저, Git 명령어 중 `git verify-pack -v <.idx 파일>` 명령어를 이용하여 `.idx` 파일의 내용을 확인할 수 있었다. 파일의 내용에는 Git 객체의 해시와 크기 정보가 포함되어 있었다.

<br>

![image](/assets/posts/250926-6.png)

객체의 해시 값을 `git cat-file -p <hash>` 명령어를 이용하여 각 객체의 내용을 확인할 수 있었는데 tree, parent, author, committer, message 등 다양한 정보가 담겨있었다. 출력된 정보의 해시 값은 다시 `cat-file` 명령어로 내용을 확인할 수 있었다. `.idx` 파일에 담긴 모든 해시 값을 하나씩 확인해보면서 어떤 내용이 담겨있는지 확인하였고, 그 중에는 `/attachments/flag.txt` 파일에 대한 내용은 확인할 수 없었다. BLOB 객체에 대해서는 `cat-file` 명령어로 내용을 확인할 수 있었는데, `/attachments/flag.txt` 파일에 대한 객체 기록이 존재했다면 읽을 수 있었을 것이라고 생각하였다.

<br>

```bash
mkdir restore-repo
cd restore-repo
git init

cp ../*.pack .git/objects/pack/
cp ../*.idx .git/objects/pack/
```

그러던 중 다른 소스코드에 대해서는 BLOB 객체가 존재하였고, 해당 소스코드를 확인하기 위해 git repository를 복원해보기로 하였다. 새로운 디렉터리를 생성하고 `git init` 명령어로 빈 저장소를 생성한 후, `.pack`과 `.idx` 파일을 복사하였다. 그 다음, `.idx` 파일에서 확인하였던 첫 번째 커밋의 해시 값을 이용하여 `git checkout <hash>` 명령어로 해당 커밋의 상태로 디렉터리 전체를 복원할 수 있었다.

<br>

![image](/assets/posts/250926-7.png)

복원된 파일의 내용을 하나씩 확인해보았고, 그 중에 `notice.php` 파일에서 파일을 다운로드 하는 기능이 존재하는 것을 확인할 수 있었다.

<br>

![image](/assets/posts/250926-8.png)

해당 기능에 따라 제공된 웹 사이트에서 `/notice.php?download=flag.txt` URL로 접속을 시도하였고, flag 파일을 얻을 수 있었다.
