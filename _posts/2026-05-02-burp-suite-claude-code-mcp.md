---
title: "Burp Suite + Claude Code + MCP를 활용한 webhacking.kr 문제 풀이 프로젝트"
date: 2026-05-02 00:00:00 +0900
last_modified_at: 2026-05-02
categories: [AI, AI for Security]
tags: []
published: True
---

# #Intro

크롬 북마크를 정리하며 오랜만에 webhacking.kr에 접속해보았고, 오늘따라 기존에는 눈에 띄지 않던 Discord 채널 입장 링크가 보여 호기심에 입장해보았다. 그런데 ...

<br>

![image](/assets/posts/260502-1.png)

OMG. 웹 해킹 공부를 할 때 애용했던 이 사이트가 5월 15일 부로 서비스를 종료한다고 한다. 웹 해킹 공부를 할 때 많은 도움을 받았던 사이트인데, 이렇게 갑자기 서비스를 종료한다니 너무 아쉬웠다.

<br>

![image](/assets/posts/260502-2.png)

[https://1unaram.github.io/categories/webhacking-kr/](https://1unaram.github.io/categories/webhacking-kr/)

꾸준히 풀이하며 공부했던 내용을 정리하기도 했었는데, 모든 문제를 풀겠다는 다짐과는 다르게 문제 풀이를 소홀히하여 많은 문제를 풀지 못하였던 것이 마음에 걸렸다. 그래서 이번 기회에 서비스 종료까지 얼마 남지 않은 기간동안 Burp Suite + Claude Code + MCP를 활용해서 webhacking.kr의 모든 문제를 풀어보는 시간을 가져보고자 하였다.

이 조합을 활용하고 싶다는 생각은 항상 했지만 막상 이를 꾸준히 활용하기가 쉽지 않았는데, 이번 기회에 꾸준히 활용해보면서 익숙해지는 시간을 가져보고자 한다.

<br>

> Reference:
>
> - [https://github.com/portswigger/mcp-server](https://github.com/portswigger/mcp-server)
> - [https://hagsig.tistory.com/355](https://hagsig.tistory.com/355)
>   {: .prompt-info }

<br>

# 환경설정

## 1. Burp Suite MCP 설치

![image](/assets/posts/260502-3.png)

가장 먼저 Burp Suite MCP를 설치하였다. [Extensions] > [BApp Store]에서 [MCP Server]를 찾아 설치하였고, MCP 탭에서 [Enabled]를 체크하여 활성화하였다.

Burp Suite MCP를 사용하기 위해서는 Java가 설치되어 있고, 환경변수에 등록되어 jar command가 사용 가능한 상태여야 한다. 나는 두 조건을 모두 만족하고 있어 별도의 설정 없이 MCP Server를 활성화하자마자 바로 사용할 수 있었다.

<br>

## 2. Claude Code 설정

![image](/assets/posts/260502-4.png)

![image](/assets/posts/260502-6.png)

Burp Suite MCP 탭 하단에 [Installation] > [Install to Claude Desktop] 버튼이 있다. 이 버튼을 클릭하면 `claude_desktop_config.json` 파일에 MCP Server의 IP 주소와 Port 번호가 자동으로 입력된다.

하지만, 나는 Claude Desktop이 아닌 Claude Code를 활용하여 문제 풀이를 진행할 예정이었기에, 입력된 값을 프로젝트 폴더의 `.mcp.json` 파일에 입력하여 연결해주었다.

<br>

## 3. CLAUDE.md 작성

Claude Code가 문제 풀이를 진행할 때의 가이드라인을 담은 `CLAUDE.md` 파일 작성이 필요했다. 내가 원하는 방식은 다음과 같았고, 이 내용을 CLAUDE.md 파일에 정리하였다.

1. 일반적인 Real-world 취약점과는 달리 webhacking.kr 문제만의 유형에 대한 이해가 필요하기에, 내가 기존에 풀이했던 문제의 writeup을 참고하도록 한다.
2. 문제를 풀이한 이후에는 직접 writeup을 작성 후 저장하여 자가 학습이 가능하도록 한다.
3. writeup에 없는 유형 혹은 추가 다른 아이디어가 필요할 때, 웹 검색 기능을 활용하여 일반적인 취약점 기법을 탐색하여 페이로드 등을 구성하도록 한다.

이 밖에도 폴더 구조, 문제 풀이 순서 등을 명시해주었다. CLAUDE.md 파일의 내용은 문제 풀이를 진행하며 계속해서 업데이트 하였다.

처음에는 Burp Suite의 Open Browser에서 webhacking.kr 로그인과 풀고자 하는 문제 페이지 이동만 수행하고, 이후에는 온전히 Claude Code가 문제를 풀이하도록 하였다.

<br>

# 문제 풀이

우선 기본적인 프로젝트 구조와 `CLAUDE.md` 파일 내용을 정리한 이후에는 본격적으로 문제 풀이를 진행하였다. 아래와 같은 프롬프트로 문제 풀이를 진행하였다.

```
old-23 문제를 풀어줘
URL: https://webhacking.kr/challenge/bonus-3/
```

<br>

![image](/assets/posts/260502-7.png)

CLAUDE.md에 명시한대로 문제 풀이가 잘 진행되는 것을 확인할 수 있었고, Burp Suite MCP 요청도 잘 이뤄지는 것을 확인할 수 있었다.

<br>

그러나, Claude Code의 한 세션의 컨텍스트는 200k로 제한되어 있는데 이를 다 쓰고도 문제를 풀이하지 못하였다. 원인은 Claude Code가 문제 풀이 과정에서 너무 많은 컨텍스트를 낭비하였기 때문이었다.

나는 Burp Suite의 [Open Browser] 기능을 활용하여 로그인과 문제 페이지 이동만을 진행하고 나머지 과정은 모두 Claude Code가 진행하도록 하였는데, 이로 인해 Claude Code가 직관적으로 문제 풀이를 진행하지 못하다 보니 초반 방향 이탈이 발생하였고, 이로 인해 컨텍스트 낭비가 더 심해지는 악순환이 발생하였다.

이를 위해 `CLAUDE.md` 파일을 다음과 같이 개선하였다.

- ❌ 기존: URL 입력 → Claude가 직접 탐색 → 컨텍스트 낭비 → 방향 이탈
- ✅ 개선: 사람이 탐색 → HTTP History 축적 → Claude는 이를 활용하여 초기 방향 설정

즉, 완전 자율 탐색이 아닌, 사람이 먼저 HTTP History를 축적하여 Claude가 이를 활용하여 초기 방향을 설정할 수 있도록 개선한 것이다. 이를 통해 컨텍스트 낭비를 줄이고, 문제 풀이의 방향성을 개선할 수 있었다.

<br>

![image](/assets/posts/260502-10.png)

![image](/assets/posts/260502-11.png)

개선된 이후에 62k의 컨텍스트로 성공적으로 문제 풀이를 진행하고, writeup 까지 작성한 것을 확인할 수 있었다.

<br>

## old-21

![image](/assets/posts/260502-12.png)

![image](/assets/posts/260502-13.png)

<br>


