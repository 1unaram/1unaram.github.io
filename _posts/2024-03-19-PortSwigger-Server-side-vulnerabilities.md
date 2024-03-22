---
title: '[PortSwigger] Server-side vulnerabilities'
date: 2024-03-19 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

> [[PortSwigger Academy](https://portswigger.net/web-security/learning-paths)] Server-side vulnerabilities를 수강하고 정리하였습니다.
{: .prompt-info }

<br>

<br>

# #Path traversal

---

Directory traversal로 알려져 있다. 이 취약점은 공격자에게 어플리케이션이 실행 중인 서버에서 임의의 파일을 읽을 수 있도록 한다. 이때 파일은 다음을 포함할 수 있다.

- 어플리케이션의 코드와 데이터

- 백엔드 시스템의 인증

- 민감한 시스템 운영 파일

몇몇의 경우에는 공격자가 서버에 임의의 파일을 작성할 수 있도록 하고, 어플리케이션 데이터나 동작을 수정할 수 있도록 하며, 궁극적으로 서버의 모든 지휘권을 취할 수도 있다.

<br>

`<img src="/loadImage?filename=218.png">`

판매 중인 상품을 보여주는 쇼핑 어플리케이션을 생각해보자. 이는 위와 같은 HTML을 사용하여 이미지를 로딩할 것이다.

<br>

`/var/www/images/218.png`

`loadImage` url은 `filename` 파라미터를 받고 특정 파일의 내용을 보여준다. 만약 이미지 파일이 `/var/www/images/` 위치에 저장되어 있다면 위와 같은 파일 경로를 읽을 것이다.


<br>

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`

이 어플리케이션은 path traversal 공격에 대한 처리가 없다면, 공격자는 위의 URL을 요청함으로써 서버의 파일 시스템으로부터 `/etc/passwd` 파일을 조회할 수 있을 것이다.

<br>

문자열 `../`는 파일 시스템 내에서 <u>이전 파일 경로로 이동</u>하는 것을 의미하는 유효한 파일 경로이다. `../../../` 문자열을 통해 파일 시스템의 root `/`으로 이동할 수 있고, `/etc/passwd` 파일을 읽을 수 있게 된다.

<br>

해당 파일은 Unix 기반 시스템에서는 서버에 등록된 사용자의 세부정보를 포함한 기본 파일이지만, 공격자가 같은 방법을 이용하면 다른 임의의 파일을 조회할 수도 있을 것이다.

<br>

`https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`

Windows에서는 `../`와 `..\` 모두 유효한 directory traversal 문자열이다. 위는 Windows 기반 서버에서 유효한 공격 예시이다.

<br>

## 🚩Lab: File path traversal, simple case

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/328156d7-4194-47a2-9bd4-3955c2cd9b94)

*View details* 버튼을 클릭하여 상품을 조회할 때 패킷을 잡다보면 위와 같이 상품의 이미지를 GET 요청하는 패킷을 확인할 수 있다. 이때, `filename` 파라미터를 전달하는 것을 확인할 수 있는데 이 값을 `../../../../../../etc/passwd`로 설정하여 요청을 전송한다. `../` 문자열을 여러 번 사용하는 이유는 상품 이미지 파일이 저장되는 경로가 어디에 위치한지 정확히 모르기 때문에 여러 번 사용함으로써 root `/` 위치로 확실하게 이동하게 하기 위함이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5077e6e7-3561-45c0-9a17-b435d5275000)

이제 응답 패킷을 확인하면 해당 파일을 조회할 수 있음을 확인할 수 있다.

<br>

<br>

# #Access Control

Access Control은 어떤 사람이나 어떤 것이 동작을 수행하거나 리소스에 접근에 인가되었는지에 대한 어플리케이션의 제한이다. 웹 어플리케이션에서 이는 Authentication(권한 부여)와 Session management(세션 관리)에 달려 있다.

<br>

- **Authentication**은 사용자가 누구인지를 확인하는 과정

- **Session management**은 연속된 HTTP 요청이 같은 사용자로부터 이루어졌는지를 확인하는 과정

- **Access control**은 사용자가 동작을 수행하려는 시도가 허용되는지 판단하는 과정

<br>

Broken access controls은 흔하게 발생하고 치명적인 보안 취약점으로 종종 존재한다. Access controls의 설계와 관리는 비즈니스, 조직, 그리고 기술 구현에 대한 법적 제한에 적용되는 복잡하고 동적인 문제이다. Access control 설계 결정은 사람에 의해 이루어지기에, 오류에 대한 가능성이 높다.

<br>

## Vertical privilege escalation

사용자가 접근이 허용되지 않은 기능에 대한 접근을 획득했다면, 이것은 vertical privilege escalation이다. 예를 들어, 관리자가 아닌 사용자가 다른 사용자 계정을 지울 수 있는 관리자 페이지에 대한 접근을 획득했다면 이것이 vertical privilege escalation이다.

<br>

## Unprotected functionality

vertical privilege escalation은 어플리케이션이 민감한 기능에 대한 어느 보호도 되어있지 않을 때 발생한다. 예를 들어, 관리자 기능은 일반 사용자의 기본 페이지로부터가 아닌 관리자의 기본 페이지로부터 연결되어 있어야 한다. 그러나, 관리자와 관련된 url을 탐색함으로써 사용자가 관리자 기능에 접근할 수도 있을 것이다.

<br>

`https://insecure-website.com/admin`

예를 들어, 한 웹사이트가 위와 같은 url로 민감한 기능을 호스팅하고 있다고 하자.

이는 사용자 인터페이스 기능에 접근하는 링크를 갖고 있는 관리자 뿐만 아니라 접근이 허락되지 않은 어떠한 사용자에 의해서도 접근 가능할지도 모른다. 몇몇의 경우에는, 관리자 url이 `robots.txt` 파일과 같은 위치에 노출될 수도 있다.

<br>

url이 어느 곳에도 노출되어 있지 않더라도, 공격자는 민감한 기능의 경로에 대해 wordlist를 사용하여 brute-force을 할 수도 있다.

<br>

## 🚩Lab: Unprotected admin functionality

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/34222cd0-35d7-4772-a6b2-ff45674d5432)

`/robots.txt` 경로로 이동하면 위와 같은 페이지를 확인할 수 있고, `/administrator-panel` 페이지가 관리자와 관련된 페이지임을 유추할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e6d5ab8e-cb26-475f-8cc5-51112a18facc)

해당 경로로 이동하면 위와 같이 사용자를 삭제할 수 있는 페이지를 확인할 수 있고, 문제에 맞게 *carlos* 사용자를 삭제하면 문제를 해결할 수 있다.

<br>

어떤 경우에서는 예측할 수 없는 url을 사용함으로서 민감한 기능을 숨기기도 한다. 소위 **sercurity by obscurity**의 예시이다. 그러나, 민감한 기능을 숨기는 것은 효과적인 access control을 제공하지는 않는데, 이는 사용자가 다양한 방법으로 난독화된 url을 찾을 수 있기 떄문이다.

<br>

`https://insecure-website.com/administrator-panel-yb556`

위의 url을 통해 관리자 기능을 호스팅하는 어플리케이션이 있다고 하자.

이는 공격자가 곧바로 예측할 수는 없지만, 어플리케이션이 사용자에게 여전히 url을 노출시킬 수 있다. url은 사용자의 역할에 따라 사용자 인터페이스를 구축하는 Javascript에서 노출될 수 있다.

<br>

```javascript
<script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
</script>
```

이 스크립트는 사용자가 관리자라면 사용자의 UI에 링크를 추가하는 기능을 수행한다. 그러나, 이는 사용자의 역할에 관계없이 모든 사용자에게 보여질 수 있다.

<br>

## 🚩Lab: Unprotected admin functionality with unpredictable URL

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/3d041339-c414-479e-b9e1-c10315a807e5)

위와 같이 페이지 소스 코드에서 `script` 키워드로 검색하면 위와 같은 스크립트가 포함되어 있음을 확인할 수 있다. 이는 관리자에 대한 UI를 제공하는 스크립트로서 관리자 페이지의 경로인 `/admin-3vwkiw`를 확인할 수 있다.

해당 경로로 이동하여 문제에 맞춰 *carlos*를 삭제하면 문제를 해결할 수 있다.

<br>

## Parameter-based access control methods

어떤 어플리케이션은 사용자의 접근 권한이나 역할을 로그인에서 결정하고 이 정보를 사용자가 조작할 수 있는 위치에 저장하기도 한다. 여기서 사용자가 조작할 수 있는 위치는 **숨겨진 필드**, **쿠키**, **미리 정해진 쿼리 스트링 파라미터**가 될 수 있다.

<br>

`https://insecure-website.com/login/home.jsp?admin=true`

`https://insecure-website.com/login/home.jsp?role=1`

위와 같은 예시처럼 어플리케이션이 사용자가 제출한 값에 따라 access control 결정을 하기도 한다. 이 방법은 사용자가 값을 수정하고 관리자 기능과 같이 사용자가 접근해서는 안되는 기능에 접근할 수 있기 때문에 안전하지 않다.

<br>

## 🚩Lab: User role controlled by request parameter

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/6c7af10c-b820-4b25-ba9d-5bbd7334f91a)

문제에서 주어진 것처럼 `wiener:peter` 값으로 계정 로그인을 해보자. 그 다음 쿠키 값을 확인하면 위와 같이 `session`과 `Admin` 이름의 쿠키가 존재하는 것을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/a6d957d4-97a2-4ab6-94ea-e5e4b091f656)

이제 `/admin` 페이지로 이동하면 admin 계정으로 로그인하지 않았기 때문에 해당 페이지에 접속할 수 없다는 문구를 확인할 수 있다. 이제 `Admin` 쿠키 값을 `false`에서 `true`로 바꾸고 다시 접속해보자.

성공적으로 관리자 페이지에 접근할 수 있고, *carlos* 계정을 삭제함으로써 문제를 해결하자.

<br>

## Horizontal privilege escalation

Horizontal privilege escalation은 사용자가 자기 자신의 리소스 대신에 다른 사용자의 리소스에 접근할 수 있을 때 발생한다. 예를 들어, 한 직원이 본인의 기록 뿐만 아니라 다른 직원의 기록까지 접근할 수 있을 때를 말한다.

Horizontal privilege escalation 공격은 vertical privilege escalation 공격 방법의 비슷한 유형이 사용된다. 예를 들어, 한 사용자가 다음 url을 통해 본인의 계정 페이지에 접근할 수 있다고 가정해보자.

`https://insecure-website.com/myaccount?id=123`

<br>

만약 공격자가 `id` 파라미터 값을 다른 사용자의 값으로 수정할 수 있다면, 공격자는 다른 사용자의 계정 페이지에 접근할 수 있고 관련된 데이터와 기능을 수행할 수 있을 것이다.

> 이는 Insecure Direct Object Reference (IDOR) 취약점의 예시이다. 이러한 종류의 취약점은 사용자가 조작할 수 있는 파라미터 값이 리소스나 기능에 직접적으로 접근할 수 있을 때 발생한다.
{: .prompt-info }

<br>

몇몇의 어플리케이션에서는, 공격 가능한 파라미터가 예측 가능한 값을 사용하지 않는다. 예를 들어, 어플리케이션이 사용자를 식별하기 위해 증가하는 숫자를 사용하는 것 대신에 globally unique identifiers (GUIDs)를 사용할 수도 있다. 이것은 공격자가 다른 사용자의 식별자를 추측하고 예측하는 것을 막는다. 그러나, 다른 사용자의 GUIDs는 사용자의 댓글이나 리뷰와 같이 사용자가 참조될 수 있는 곳에서 노출될 수도 있다.

<br>

## 🚩Lab: User ID controlled by request parameter, with unpredictable user IDs

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/66ffdb79-8b5c-4710-91c8-d20005afc51d)

*Coping with Hangovers* 포스트를 열어보면 *carlos*가 포스트의 작성자인 것을 확인할 수 있다. 페이지 소스 코드를 보면 *carlos*의 블로그로 이동할 수 있는 태그를 확인할 수 있는데 여기서 `userId` 값을 확인할 수 있다.

<br>

이제 로그인 페이지에서 `wiener:peter` 계정으로 로그인하면 계정 페이지에 로그인할 수 있는데, 접속된 url을 확인해보면 `id`라는 파라미터로 로그인 계정을 식별하고 있는 것을 확인할 수 있다. 이 값을 아까 획득한 carlos의 `userId`값으로 바꾸어 url 요청을 하자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/7c47881a-012c-44eb-8065-32d0d23d88af)

이제 carlos의 계정 페이지에 접속할 수 있고 API Key로 획득하였다. 해당 값을 Submit solution 하면 문제를 해결할 수 있다.

<br>

## Horizontal to vertical privilege escalation

종종 horizontal privilege escalation 공격은 더 많은 권한을 가진 사용자를 침해함으로써 vertical privilege escalation으로 바뀔 수 있다. 예를 들어 horizontal escalation은 공격자가 다른 사용자의 패스워드를 재설정하거나 획득할 수 있도록 한다. 만약 공격자가 관리자를 타겟으로 하고 관리자의 계정을 침해한다면, 공격자는 관리 권한을 획득하고 따라서 vertical privilege escalation을 수행할 수 있다.

<br>

`https://insecure-website.com/myaccount?id=456`

위의 예시와 같이 한 공격자가 horizontal privilege escalation에서 언급했던 파라미터 변조 기술을 사용하여 다른 사용자 계정 페이지에 대한 권한을 획득할 수 있다.

만약 공격자의 타겟인 사용자가 어플리케이션의 관리자라면, 공격자는 관리자 계정 페이지에 대한 접근을 획득할 수 있다. 이 페이지는 관리자의 패스워드를 노출시키거나 이를 바꿀 수 있는 수단을 획득할 수도 있고, 특정 권한이 주어진 기능을 직접적으로 접근할 수 있게 된다.

<br>

## 🚩Lab: User ID controlled by request parameter with password disclosure

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b90a4829-348b-4bf9-8d87-d54731966cf6)

우선 `wiener:peter` 계정으로 로그인을 해보자. 그러면 `/my-account?id=wiener` url 경로에서 계정 페이지를 확인할 수 있는데, 이 때 Password 필드에 사용자의 계정 패스워드 값이 `input` 태그의 `value` 속성으로 미리 채워져 있고 이 값을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/3c4ab7b2-dd75-43c8-aec0-ac89be6db304)

이제 `/my-account?id=administrator` url 경로로 요청을 해보면, 관리자 계정 페이지에 접속할 수 있고 Password 필드에 미리 채워진 패스워드 `7us9o2lpndjye6nnf5e6` 값을 획득할 수 있다. 해당 패스워드를 이용해 관리자 계정으로 로그인을 하면 위와 같이 *Admin panel*을 확인할 수 있다. 해당 페이지에서 *carlos*의 계정을 삭제하면 문제를 해결할 수 있다.

<br>

<br>

# #Authentication

## Authentication vulnerabilitis

Authentication vulnerabilities는 개념적으로 이해하기 쉬우나, 이는 인증과 보안 사이의 명백한 관계 때문에 치명적이다. Authentication vulnerabilities는 공격자가 민감한 데이터와 기능에 접근할 수 있도록 해준다. 또한 추가적인 익스플로잇을 위한 공격 표면을 노출시기기도 한다. 이러한 이유로 authentication vulunerabilities를 식별하고 익스플로잇하는 방법과 흔한 보호 수단을 우회하는 방법을 배우는 것이 중요하다.

<br>

## What is the difference between authentication and authorization?

인증은 한 사용자가 그들이 누구인지 주장하는 것을 확인하는 절차이다. 권한은 한 사용자가 어떠한 행동을 해도 되는지 확인하는 과정을 포함하고 있다.

<br>

예를 들어, 인증이 한 사이트에 접근을 시도하는 사용자명 `Carlos123`을 가진 누군가가 계정을 가진 인물과 동일한 인물인지를 결정한다. 일단 `Carlos123`이 인증되면, 그 사용자의 권한이 어떤 행동들이 허락되는지를 결정한다. 예를 들어, 사용자가 다른 사용자에 대한 개인 정보를 접근할 수 있는 권한을 가지거나 다른 사용자의 계정을 삭제할 수 있는 등의 행동을 수행하는 권한을 가질 수 있다.

<br>

## Brute-force attacks

Brute-force 공격은 공격자가 유효한 자격 증명을 추측하기 위해 시행 착오 시스템을 사용하는 경우를 말한다. 이 공격은 전형적으로 사용자의 이름과 패스워드 wordlist를 사용하여 자동화되어 있다. 이 공격을 위한 툴을 사용하는 이 자동화 과정은 잠재적으로 공격자가 빠른 속도로 막대한 횟수의 로그인 시도를 할 수 있게 해준다. Brute-forcing은 사용자 이름과 비밀번호를 완전히 무작위하게 만드는 경우만 존재하지는 않는다. 기본적인 논리 혹은 공개적으로 이용 가능한 지식을 이용함으로써 공격자는 미세 조정하여 훨씬 더 정확한 추측을 할 수 있다. 이것은 이러한 공격의 효율성을 상당히 증가시킨다. 사용자를 이증하는 유일한 방법으로 비밀번호 기반 로그인을 사용하는 웹 사이트는 충분한 brute-force 보호 기법을 적용하지 않는다면 매우 높은 취약점을 가질 수 있다.

<br>


## Brute-forcing usernames

사용자명은 이메일 주소와 같이 인식 가능한 패턴에 부합하는 경우에 특히 쉽게 추측할 수 있다. 예를 들어, `firstname.lastname@somecompany.com` 형식을 사용하는 매우 흔한 비즈니스 로그인 방식을 볼 수 있다. 그러나, 분명한 패턴이 존재하지 않더라도 높은 권한을 갖고 있는 계정이 `admin`이나 `administrator`와 같은 예측 가능한 사용자명으로 만들어지는 경우도 있다. 감사 중에 웹 사이트가 잠재적인 사용자명을 공개적으로 노출하고 있는지 확인해라. 예를 들어, 로그인 없이 사용자의 프로필에 접근할 수 있는가? 그렇다면 실제적인 프로필의 내용이 숨겨져 있더라도, 프로필에서 사용된 이름은 로그인할 때 사용되는 사용자명이랑 같은 경우가 있다. 또한 이메일 주소가 노출되지 않는지 HTTP 응답을 체크해야만 한다. 때때로는 응답에 관리자와 같이 높은 권한을 가진 사용자의 이메일 주소가 포함되기도 한다.

<br>

## Brute-forcing passwords

패스워드도 마찬가지로 brute-force 공격을 받을 수 있으며, 패스워드의 강도에 따라 난이도가 달라질 수 있다. 많은 웹 사이트는 패스워드 정책 폼을 사용하는데, 이는 사용자가 brute-force 공격만으로 크랙하기 여러운 높은 엔트로피를 가진 패스워드를 생성하도록 강제한다. 다음은 일반적으로 패스워드를 강제할 때 포함되는 것이다.

- 최소 글자 수
- 대소문자 혼합
- 1개 이상의 특수문자

그러나, 높은 엔트로피를 가진 패스워드는 컴퓨터 혼자만으로는 크랙하기 어려운 반면에, 우리는 사용자가 무의식적으로 이 시스템에 도입한 취약점을 익스플로잇하기 위해 사람의 행동에 대한 기본적인 지식을 이용할 수 있다. 무작위의 글자 조합으로 강한 패스워드를 만드는 것보다 사용자가 기억할 수 있는 패스워드를 선택하여 비밀번호 정책에 맞추려고 시도하는 경우가 많다. 예를 들어, `mypassword`가 패스워드로 허용되지 않았다면 사용자는 `Mypassword!`나 `Myp4$$w0rd`와 같은 것을 대신에 시도해볼 수 있다.

<br>

정책에 따라 사용자가 정기적으로 패스워드를 변경하도록 요구하는 경우에 사용자들이 선호하는 패스워드로 바꾸기 위해 사소하고 예측 가능한 변화를 만드는 일은 흔히 일어난다. 예를 들어, `Mypassword1!`는 `Mypassword1?`나 `Mypassword2!`가 될 수 있다. 가능한 자격 증명이나 예측 가능한 패턴에 대한 지식은 brute-force 공격을 더 정교하게 만들어 단순히 가능한 모든 글자 조합을 반복하는 것보다 더욱 효과적으로 만들 수 있다.

<br>

## Username enumeration

사용자명 열거(Username enumeration)은 공격자가 주어진 사용자명이 유효한지 확인하기 위해 웹 사이트 동작의 변화를 관측할 수 있게 해준다. 사용자명 열거는 일반적으로 로그인 페이지(예를 들어, 유효한 사용자명을 입력했으나 패스워드가 틀린 경우) 혹은 회원가입 페이지에서 이미 사용 중인 사용자명을 입력했을 경우에 발생한다. 이는 공격자가 유효한 사용자명의 최종 후보 목록을 생성할 수 있기 때문에 로그인 brute-force 공격에 대한 시간과 노력을 막대하게 줄일 수 있다. 로그인 페이지를 brute-force 공격을 시도하는 동안 다음 사항의 차이점에 특히 주의해야 한다.

- **상태 코드** : brute-force 공격을 하는 동안에 응답받은 HTTP 상태 코드는 대부분의 추측이 틀릴 것이기 때문에 동일할 가능성이 높다. 만약 다른 상태 코드를 받았다면, 사용자명이 맞았다는 강력한 표시이다. 웹 사이트에서는 결과에 관계없이 항상 동일한 상태 코드를 반환하는 것이 가장 좋은 방법이지만 이 방법이 항상 지켜지는 것은 아니다.

- **에러 메시지** : 때때로는 사용자명과 패스워드 모두 틀린 경우거나 패스워드만 틀린 경우에 따라 반환하는 에러 메시지가 다를 수 있다. 웹 사이트에서는 두 경우에 동일하고 일반적인 메시지를 사용하는 것이 가장 좋은 방법이지만 작은 타이핑 오류가 가끔 발생한다. 렌더링된 페이지에 해당 문자가 보이지 않는 경우에도 문자 하나만 잘못되어도 두 메시지가 구분된다.

- **응답 시간** : 대부분의 요청이 비슷한 응답 시간으로 처리된 경우, 이로부터 벗어나는 것은 뒤에서 뭔가 다른 일이 일어나고 있음을 나타낸다. 이는 사용자명이 맞았다는 추측을 나타내는 또다른 지표이다. 예를 들어, 한 웹 사이트가 사용자명이 유효한 경우에만 패스워드가 맞는지를 체크할 수 있다. 이 추가적인 과정은 응답 시간을 조금 늘릴 수 있다. 이는 미묘할 수도 있는데, 공격자는 웹 사이트가 처리하는데 눈에 띄게 오래 걸리는 과도하게 긴 패스워드를 입력함으로써 지연 시간을 더 분명하게 만들 수 있다.

<br>

## 🚩Lab: Username enumeration via different responses

이 문제는 주어진 wordlists를 이용해 brute-force 공격으로 로그인을 하는 문제이다. 파이썬 requests 모듈을 이용하여 코드를 짜는 방법도 있겠지만 Burp Suite의 **Intruder** 기능을 이용하여 문제를 해결해보자. Intruder는 Burp Suite에서 제공하는 웹 어플리케이션을 대상으로 사용자 정의 공격을 할 수 있는 자동화 도구이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/24feb153-5c41-4a8b-9739-345fe92ca82f)

우선 **Proxy - HTTP history** 탭에서 로그인을 시도한 요청 패킷을 찾아 우클릭하여 **Send to Intruder**를 클릭한다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/0dae99bc-4be8-4deb-9d46-980aae2650e5)

**Intruder - Positions** 탭에서 **Attack type**을 <u>Sniper</u>로 체크한다. **Payload positions**에서 POST 요청에 포함되는 값을 변경할 수 있는데, 위의 이미지처럼 username은 `§`로 감싸 변수처럼 만들어주고 패스워드는 아무런 값을 대입해두자. 우선 username을 알아내기 위함이다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/6ccaf7a3-abd6-493b-9cad-0f9c1712df8e)

**Intruder - Payloads** 탭에서 **Payload sets**을 위의 이미지처럼 설정하여 준다.

<br>

문제에서 주어진 username의 wordlist를 복사하여 **Payload settings**에서 Paste를 눌러 리스트를 생성해준다.

<br>

이제 우측 상단에 있는 **Start attack** 버튼을 눌러 공격을 시작한다. 잠시 기다리면 모든 단어에 대해서 자동으로 username 값을 변경하며 요청을 처리하고 각종 정보를 보여준다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/5b676151-78a5-4c72-8c7c-4d4f2012756a)

모든 단어에 대해 시도하고 나면 Length 값이 다른 하나의 요청을 찾을 수 있다. 이 페이로드가 올바른 username인 것을 알아낼 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/20b550ab-3afd-4b64-b399-d326401db14e)

이제 username을 `antivirus`로 채우고 password 값을 변수화하고 wordlist를 수정하여 같은 방식으로 공격을 시도하면 위와 같이 상태 코드와 길이가 다른 요청을 찾을 수 있고, `qwerty`가 올바른 password임을 알아낼 수 있다.

<br>

## Bypassing two-factor authentication

때로는 이중 인증(two-factor authentication) 구현에 결함이 있어 완전히 우회하는 경우도 있다. 만약 사용자에게 처음으로 비밀번호를 입력하라는 메시지가 표시된 다음에 구별된 페이지에서 인증 코드를 입력하라는 메시지가 표시되면, 사용자는 인증 코드를 입력하기 전에 사실상 "로그인" 상태에 있다. 이러한 경우에 첫 번째 인증 단계를 완료한 후 "로그인 전용" 페이지로 직접 건너뛸 수 있는지 테스트해 볼 가치가 있다. 때때로 웹 사이트가 페이지를 로딩하기 전에 두 번째 단계를 완료했는지 여부를 실제로 확인하지 않을 수도 있다.

<br>

## 🚩Lab: 2FA simple bypass

간단한 이중 인증 우회 문제이다. 우선 주어진 계정 `carlos:montoya`로 로그인을 한다. 이제 이중 인증 코드를 입력하는 창을 볼 수 있는데 이때, *Back to lab description* 버튼을 눌러 메인 화면으로 이동한다. 이 상태는 이미 로그인된 상태이다. 이제 My account로 이동하면 쉽게 문제를 풀 수 있다.

<br>

<br>

# #Server-side request forgery (SSRF)

서버 측 요청 위조(Sever-side request forgery) 즉, SSRF는 공격자가 서버 측 어플리케이션을 의도되지 않은 경로로 요청을 보내도록 만들 수 있도록 하는 웹 보안 취약점이다. 일반적인 SSRF 공격에서 공격자는 서버가 조직 인프라 내의 내부 전용 서비스에 연결되도록 할 수 있다. 다른 경우로 공격자는 서버가 임의의 외부 시스템에 연결됟록 할 수 있다. 이는 인증 자격 증명과 같은 민감한 데이터가 유출될 수 있다.

<br>

## SSRF attacks against the server

서버에 대한 SSRF 공격에서, 공격자는 loopback 네트워크 인터페이스를 통해 어플리케이션이 어플리케이션을 호스팅하는 서버에 HTTP 요청을 다시 보내도록 한다. 여기에는 일반적으로 `127.0.0.1`(loopback 어댑터를 가리키는 예약된 IP 주소) 혹은 `localhost`(같은 어댑터를 위해 일반적으로 사용되는 이름)과 같은 호스트 이름이 포함된 URL을 제공하는 작업이 포함된다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

예를 들어, 사용자가 특정한 상점에 한 상품이 재고가 있는지 사용자가 확인할 수 있는 쇼핑 어플리케이션이 있다고 가정하자. 재고 정보를 제공하기 위해서는 어플리케이션은 다양한 백엔드 REST API 쿼리를 해야만 한다. 프론트엔드 HTTP 요청을 통해 관련 백엔드 API 엔드포인트에 URL을 전달하여 이를 수행한다. 사용자가 상품에 대한 재고 상태를 조회할 때, 브라우저는 위와 같은 요청을 보낸다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

이는 서버가 특정한 URL에 요청을 보내고, 재고 상태를 조회하고, 이를 사용자에게 전달한다. 위의 예시는 공격자가 서버에 로컬 URL을 지정하도록 요청을 수정할 수 있다.

서버는 `/admin` URL에 있는 내용을 가져오고 이를 사용자에게 전달한다.

<br>

공격자는 `/admin` URL에 방문할 수 있지만 관리 기능은 일반적으로 인증된 사용자만 액세스할 수 있다. 이는 공격자가 관심있는 내용을 전혀 볼 수 없는 것을 의미한다. 그러나 로컬 서버에서 `/admin` URL로 요청을 보낸다면, 일반적인 액세스 제어는 우회된다. 요청이 신뢰할 수 있는 경로에서 시작된 것으로 나타나므로 어플리케이션은 모든 액세스를 관리 기능에 제공한다.

<br>

## 🚩Lab: Basic SSRF against the local server

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/ede5c922-a8db-4b93-85c4-011a41337f69)

한 상품의 세부 정보 페이지로 들어가면 각 나라의 재고를 확인할 수 있다. Check stock 버튼을 누를 때 요청하는 패킷을 확인하면 위와 같이 `stockApi` 값을 전달하고 있다. 이제 해당 API를 `http://localhost/admin`로 변경하여 요청을 해보자.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/1f1e5adb-f6f9-4e6b-8651-ac14ab9bc068)

왼쪽과 같이 요청을 보내면 오른쪽과 같은 응답을 얻는데 이는 관리자 페이지임을 알 수 있다. carlos 계정을 삭제하는 부분은 `a` 태그로 경로가 있으므로 `/admin/delete?username=carlos` 경로로 요청을 보내면 된다. 이제 다시 `stockApi` 값을 `stockApi=http://localhost/admin/delete?username=carlos`와 같이하여 요청을 전송하면 문제가 해결됨을 알 수 있다.

<br>

## SSRF attacks against other back-end systems

어떤 경우에서는 어플리케이션 서버가 사용자가 직접적으로 접근할 수 없는 백엔드 시스템과 상호작용할 수 있다. 이러한 시스템은 라우팅할 수 없는 개인 IP 주소가 있는 경우가 많다. 백엔드 시스템은 일반적으로 네트워크 토폴로지에 의해 보호되므로 보안 상태가 약한 경우가 많다. 많은 경우에 내부 백엔드 시스템에는 시스템과 상호작용할 수 있는 사람은 누구나 인증 없이 액세스할 수 있는 중요한 기능이 포함되어 있다.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

이전 예시에서 `https://192.168.0.68/admin` 백엔드 URL에서 관리자 인터페이스가 있다고 해보자. 공격자는 SSRF 취약점을 익스플로잇하기 위해 위와 같은 요청을 보내고 관리자 인터페이스에 액세스할 수 있다.

<br>

## 🚩Lab: Basic SSRF against another back-end system

이 문제는 이전 문제와 같이 재고를 체크하는 기능에서 요청할 때 포함하는 `stockApi` 값을 `192.168.0.X`로 하여 관리자 페이지에 접근하면 되는 문제이다. 마찬가지로 **Intruder** 기능을 이용하여 마지막 숫자 값을 0부터 255로 하여 응답이 다른 주소를 찾아내면 된다. 페이로드는 `stockApi=http://192.168.0.§1§:8080/admin`로 하여 기능을 사용하면 된다.

<br>

`192.168.0.142`에서 `200` 응답코드를 받을 수 있고 `stockApi=http://192.168.0.142:8080/admin/delete?username=carlos` 값을 포함하여 요청을 보내면 문제를 해결할 수 있다.

<br>

<br>

# #File upload vulnerabilites
