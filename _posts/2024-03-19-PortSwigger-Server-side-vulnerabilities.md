---
title: '[PortSwigger] Server-side vulnerabilities'
date: 2024-03-19 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: True
---

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

## Lab: File path traversal, simple case

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

## Lab: Unprotected admin functionality

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

## Lab: Unprotected admin functionality with unpredictable URL

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

## Lab: User role controlled by request parameter

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

## Lab: User ID controlled by request parameter, with unpredictable user IDs

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

## Lab: User ID controlled by request parameter with password disclosure

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/b90a4829-348b-4bf9-8d87-d54731966cf6)

우선 `wiener:peter` 계정으로 로그인을 해보자. 그러면 `/my-account?id=wiener` url 경로에서 계정 페이지를 확인할 수 있는데, 이 때 Password 필드에 사용자의 계정 패스워드 값이 `input` 태그의 `value` 속성으로 미리 채워져 있고 이 값을 확인할 수 있다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/3c4ab7b2-dd75-43c8-aec0-ac89be6db304)

이제 `/my-account?id=administrator` url 경로로 요청을 해보면, 관리자 계정 페이지에 접속할 수 있고 Password 필드에 미리 채워진 패스워드 `7us9o2lpndjye6nnf5e6` 값을 획득할 수 있다. 해당 패스워드를 이용해 관리자 계정으로 로그인을 하면 위와 같이 *Admin panel*을 확인할 수 있다. 해당 페이지에서 *carlos*의 계정을 삭제하면 문제를 해결할 수 있다.

<br>

<br>

# #Authentication
