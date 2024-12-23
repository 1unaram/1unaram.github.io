---
title: "[PortSwigger] Academy: Web Cache Deception"
date: 2024-12-20 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: False
---

> [[PortSwigger Academy Web Cache Deception](https://portswigger.net/web-security/learning-paths/web-cache-deception)] 을 수강하고 정리하였습니다.
> {: .prompt-info }

<br><br>

# #Web Caches

---

웹 캐시는 서버와 사용자 사이에 존재하는 시스템이다. 클라이언트가 정적 리소스를 요청하면, 요청은 가장 먼저 캐시로 향해진다. 만약 캐시가 리소스에 대한 카피가 없다면(Cache Miss), 요청은 응답하는 서버로 보내진다. 그 다음 응답은 사용자에게 전달되기 이전에 캐시로 전달되고, 캐시는 응답을 저장할지 결정하는 미리 구성된 일련의 룰(Cache Rule)을 이용하여 응답을 저장하거나 저장하지 않은 다음에 사용자에게 전달 된다.

<br>

만약 캐시가 응답을 저장했다면, 동일한 정적인 리소스에 대한 요청이 나중에 만들어질 때 캐시는 저장된 응답의 카피를 사용자에게 곧바로 제공한다(Cache Hit).

<br>

![image](https://github.com/user-attachments/assets/a0cf9f01-a522-4992-a4c8-b9655679b752)

<br>

캐싱하는 것은 웹 콘텐츠를 전달하는 측면에서 점점 흔해지고 중요해지는데, 전 세계에서 제공되는 서버에 저장되어 있는 콘텐츠의 카피를 캐싱하는 데에 사용되는 Content Delivery Networks(CDNs)의 사용에서 특히 그렇다. CDNs는 데이터 이동 거리를 최소화함으로써 로드하는 시간을 줄여 사용자와 가장 가까운 서버로부터 콘텐츠를 제공하게 되어 전달 속도를 향상 시킨다.

<br>
<br>

## Cache keys

캐시는 HTTP 요청을 받을 때, 곧바로 전달할 수 있는 캐싱된 응답이 존재하는지 혹은 원래의 서버로 곧바로 요청을 보내야만 하는지를 결정해야 한다. 캐시는 HTTP 요청의 요소로부터 **cache key**라는 것을 생성함으로써 이 결정을 내린다. 일반적으로 이는 URL 경로와 쿼리 파라미터를 포함하고 있으나, 헤더와 콘텐츠 타입과 같은 다른 다양한 요소를 포함하기도 한다.

만일 들어온 요청의 캐시 키가 이전 요청의 캐시 키와 매치된다면, 캐시는 그들을 동등하다고 보고 캐싱된 응답의 카피를 제공한다.

<br>

## Cache rules

캐시 룰은 어떤 것이 캐싱되고 얼마나 오래 존재해야 하는지를 결정한다. 캐시 룰은 일반적으로 자주 바뀌지 않고 다수의 페이지에서 재사용되는 정적인 리소스들을 저장하기 위해 세팅된다. 동적인 콘텐츠는 사용자가 가장 최신의 데이터를 서버로부터 바로 얻을 수 있도록 하며 민감한 정보를 포함하곤 하기 때문에 캐싱되지 않는다. Web Cache Deception 공격은 캐시 룰이 적용되는 방법을 공격하기에 다양한 타입의 룰을 아는 것이 중요한데, 특히 요청의 URL 경로에서 정해져 있는 문자열을 기반으로 하는 경우를 포함한다.

<br>

예를 들어

- 정적인 파일 확장자 룰 : `.css` 혹은 `.js`와 같이 요청된 리소스의 파일 확장자를 매치한다.
- 정적인 디렉터리 룰 : 특정한 prefix로 시작하는 모든 URL 경로를 매치한다. 이는 주로 `/static`이나 `/assets`와 같이 오로지 정적인 리소스만을 포함하고 있는 특정한 타겟 디렉토리를 위해 사용된다.
- 파일 이름 룰 : `robots.txt`와 `favicon.ico`와 같이 웹 운영에서 글로벌하게 요청되고 거의 변하지 않는 특정한 파일 이름을 타겟 파일로 매치한다.

<br><br>

# #Constructing a web cache deception attack

---

일반적으로 말하기를, 기본 web cache deception 공격은 다음과 같은 스텝을 포함하고 있다.

1. 민감한 정보를 포함하는 동적인 응답을 리턴하는 타겟 엔드포인트를 찾아낸다. 렌더링된 페이지에서는 민감한 정보가 보이지 않을 수 있기 때문에 Brup Suite에서 응답을 조사한다. 원본 서버의 상태를 변경하는 요청은 일반적으로 캐싱 되지 않기 때문에 `GET`, `HEAD`, `OPTIONS` 메소드를 지원하는 엔드포인트에 포커싱한다.

2. 캐시와 원본 서버가 URL 경로를 파싱하는 방법 내에서 모순을 찾아낸다. 여기서 모순은 URL을 리소스에 매핑하기, 구분 기호 문자 처리, 경로 정규화와 같은 방법 내에서 존재할 수 있다.

3. 모순을 이용하여 캐시를 속여 동적 응답을 저장하도록 하는 악성 URL을 만든다. 피해자가 URL에 접근할 때, 응답이 캐시에 저장된다. Burp를 이용해서 동일한 URL이 피해자의 데이터를 포함하고 있는 캐싱된 응답을 가져오도록 요청을 보낼 수 있다. 취약점을 숨길 수 있는 세션이나 유효하지 않은 로컬 데이터 없이 사용자를 리다이렉트 할 수 있기 때문에 브라우저에서 직접적으로 이를 하면 안된다.

<br>

## Using a cache buster

취약점을 테스트하고 web cache deception 공격을 진행하는 동안, 우리가 보내는 각 요청이 다른 캐시 키를 가지고 있도록 해야한다. 그렇지 않으면, 테스트 결과에 영향을 끼치는 캐싱된 응답을 받을 수 있기 때문이다.

URL 경로와 어떠한 쿼리 파라미터든지 모두 일반적으로 캐시 키를 포함하고 있기 때문에, 쿼리 스트링을 경로에 추가하고 요청을 보낼 땨마다 이를 바꿈으로써 키를 바꿀 수 있다. **Param Miner** 확장자를 이용하여 이 과정을 자동화할 수 있다. 이를 하기 위해서는 먼저 해당 확장을 설치하고, 상단의 _Param Miner > Settings_ 메뉴를 클릭하고, *Add dynamic cachebuster*를 선택하면 된다. 이제 Burp는 특정한 쿼리 스트링을 매 요청에 추가할 수 있다. 추가된 쿼리 스트링은 _Logger_ 탭에서 확인할 수 있다.

<br>

## Detecting cached responses

테스트를 하는 동안, 캐싱된 응답을 식별할 줄 아는 것이 중요하다. 이를 위해서는 응답 헤더와 응답 시간을 보면 된다.

<br>

다양한 응답 헤더가 응답이 캐싱 되었다는 것을 가리킨다. 예를 들어,

- `X-Cache` 헤더는 응답이 캐시로부터 전달되었는지에 대한 정보를 제공한다.

  - `X-Cache: hit` : 응답이 캐시로부터 전달되었다.
  - `X-Cache: miss` : 캐시가 요청의 키에 대한 응답을 갖고 있지 않기에 원본 서버로부터 가져왔다. 대부분의 경우에는 그 다음에 캐시를 한다. 이를 확인하기 위해서는 요청을 다시 보내어 값이 hit로 업데이트 되었는지를 보면 된다.
  - `X-Cache: dynamic` : 원본 서버가 동적으로 콘텐츠를 생성했다. 일반적으로 이는 응답이 캐싱에 적합하지 않다는 것을 의미한다.
  - `X-Cache: refresh` : 캐싱된 콘텐츠의 기간이 만료되어 리프레시하거나 갱신이 필요하다.

- `Cache-Control` 헤더에는 최대 수명 `max-age`이 0보다 더 큰 `public`과 같은 캐싱을 나타내는 지시문이 포함될 수 있다. 이는 리소스가 캐싱 가능하다는 것을 암시할 뿐이다. 또한 항상 캐싱을 나타내는 것이 아니며, 캐시가 이 헤더를 무시하는 경우도 있다.

<br>

만일 같은 응답에 대해 응답 시간에 큰 차이가 있다는 것을 알아내었다면, 이는 마찬가지로 빠른 응답이 캐시로부터 전달되었음을 이야기 한다.

<br><br>

# #Explotiing static extension cache rules

---

캐시 룰은 `.css`나 `.js`와 같은 흔한 파일 확장자를 매칭함으로써 정적인 리소스를 타겟하곤 한다. 이는 대부분의 CDNs에서 기본 동작이다. 만일 캐시와 원본 서버가 URL 경로를 리소스에 매핑하거나 구분자를 사용하는 방식에 모순이 존재한다면, 공격자는 원본 서버에 의해 무시되지만 캐시에서는 보여지는 정적인 확장자를 갖는 동적인 리소스에 대한 요청을 만들 수 있다.

<br><br>

# #Using path mapping discrepancies

---

## Path mapping discrepancies

URL 경로 매핑은 URL 경로를 파일, 스크립트, 명령어 실행과 같이 서버의 리소스에 연관 시키는 과정을 말한다. 다양한 프레임워크와 기술에 의해 사용되는 다양한 범위의 매핑 스타일이 존재한다. 전통적인 URL 매핑과 RESTful URL 매핑이 흔한 매핑 스타일들이다.

<br>

전통적인 URL 매핑은 파일 시스템 내에 위치하는 리소스로 향하는 직접적인 경로를 나타낸다. 다음은 일반적인 예시이다.

`http://example.com/path/in/filesystem/resource.html`

- `http://example.com`은 서버를 가리킨다.
- `/path/in/filesystem/`은 서버의 파일 시스템에 있는 디렉터리 경로를 나타낸다.
- `resource.html`은 접근 되어지는 특정한 파일이다.

<br>

대조적으로, REST 스타일의 URL들은 물리적인 파일 구조에 직접적으로 매칭되지 않는다. 이들은 파일 경로를 API의 논리적인 부분에 추상화한다.

`http://example.com/path/resource/param1/param2`

- `http://example.com`은 서버를 가리킨다.
- `/path/resource/`는 리소스를 나타내는 엔드포인트이다.
- `param1`과 `param2`는 요청을 처리하기 위해 서버에 의해 사용되는 경로 파라미터이다.

<br><br>

캐시와 원본 서버가 리소소 URL 경로를 매핑하는 방법에서의 모순은 web cache deception 취약점을 야기할 수 있다.

`http://example.com/user/123/profile/wcd.css`

- REST 스타일의 URL 매핑을 사용하는 원본 서버는 이 URL을 `/user/123/profile` 엔드 포인트에 대한 요청으로 해석할 수 있고, `wcd.css`를 중요하지 않은 파라미터라고 생각하여 이를 무시하고 사용자 `123`에 대한 프로필 정보를 반환할 수 있다.

- 전통적인 URL 매핑을 사용하는 캐시는 이를 `/user/123` 내의 `/profile` 디렉터리에 위치한 `wcd.css`라는 이름을 가진 파일에 대한 요청으로 간주할 수 있다. 이는 URL 경로를 `/user/123/profile/wcd.css`로 해석한다. 만일 경로가 `.css`로 끝나는 요청에 대한 응답을 저장하도록 구성되어진 경우, 이는 마치 CSS 파일이었던 것처럼 프로필 정보를 캐싱하고 전달한다.

<br><br>

## Exploiting path mapping discrepancies

원본 서버가 URL 경로를 리소스에 매핑하는 방법을 테스트하기 위해서는, 임의의 경로 세그먼트를 타겟 엔드포인트의 URL에 추가해야 한다. 만일 응답이 기본 응답과 동일한 민감한 데이터가 여전히 포함되어 있는 경우, 이는 원본 서버가 URL 경로를 추상화하고 추가된 세그먼트를 무시한다는 것을 나타낸다. 예를 들어, `/api/orders/123`을 `/api/orders/123/foo`
로 수정해도 주문 정보가 반환되는 경우가 있다.

<br>

캐시가 URL 경로를 리소스에 매핑하는 방법을 테스트하기 위해서는, 정적인 확장자를 추가함으로써 캐시 룰에 매치되도록 경로를 수정해야 한다. 예를 들어, `/api/orders/123/foo`를 `/api/orders/123/foo.js`로 업데이트 해라. 만약 응답이 캐시되면, 1) 캐시가 정적인 확장자를 가진 전체 URL 경로를 해석한다는 것과 2) `.js`로 끝나는 요청에 대한 응답을 저장하기 위한 캐시 룰이 존재한다는 것을 의미한다.

캐시는 특정한 정적인 확장자에 기반한 룰을 가지고 있을 것이기에 `.css`, `.ico`, `.exe`를 포함한 다양한 범위의 확장자를 시도해볼 수 있다.

<br>

이제, 캐시에 저장되는 동적인 응답을 반환하는 URL을 만들어 볼 수 있다. 원본 서버가 다양한 엔드포인트에 대한 다양한 추상화 룰을 가지고 있기 때문에, 이 공격은 우리가 테스트한 특정 엔드포인트에만 국한된다.

<br>

> Burp Scanner는 스캔 중 경로 매핑 모순에 의해 발생하는 web cache deception 취약점을 자동으로 찾아낼 수 있다. Web Cache Deception Scanner BApp을 사용하여 잘못 구성된 웹 캐시를 찾아낼 수도 있다.
> {: .prompt-info}

<br><br>

## 🚩Lab: Exploiting path mapping for web cache deception

사용자 `carlos`의 API 키를 찾아내는 문제이다. web cache deception 취약점을 이용할 수 있는 부분을 찾아보자.

<br>

![image](https://github.com/user-attachments/assets/9cb8cd2a-4ea9-4ced-b70a-e50bcc361986)

우선 `wiener`의 계정으로 로그인하고 `/my-account` 경로에 진입하면 로그인한 계정인 `wiener`의 API Key를 확인할 수 있다. 이렇게 민감한 데이터가 노출되는 곳이 취약점을 공격할 수 있는 엔드포인트가 된다.

<br>

이제 burp suite에서 `/my-account` 경로에 임의의 하위 경로 `/foo`를 추가하여 `/my-account/foo`에 요청을 보내보자. 요청에 대한 응답에서 여전히 API 키가 전달되는 것을 확인할 수 있다. 이는 하위 세그먼트에 대해서 무시한다는 것을 알 수 있다. 그 다음으로 `/foo` 경로 대신 `/foo.js`를 추가하여 `/my-account/foo.js`에 요청을 보내보자.

<br>

![image](https://github.com/user-attachments/assets/506d6dc4-2ff0-48be-9548-4f52ee5a6ed4)

여전히 API 키를 반환하는 것을 확인할 수 있을 뿐아니라 `Cache-Control: max-age=30` 필드와 `X-cache: miss` 필드를 확인할 수 있는 것으로 보아 해당 페이지가 캐싱되는 것을 알 수 있다.

<br>

![image](https://github.com/user-attachments/assets/43f90915-a3e1-4d9f-9a0e-0a4992854b82)

30초 이내에 다시 같은 요청을 보내면 위와 같이 `X-cache` 필드가 `hit`로 설정된 것을 확인할 수 있다. 이는 현재 응답이 서버로부터가 아닌 캐시에서 가져온 것임을 알 수 있다.

**이 페이지는 원래 `wiener` 계정으로 로그인하여 `/my-account` 경로에 요청을 보내야 확인할 수 있는 페이지이지만 로그인 하지 않더라도 다른 사용자가 해당 경로 `/my-account/foo.js`로 요청을 보내면 요청에 대한 응답을 서버가 아닌 캐시에서 가져오기 때문에 해당 페이지를 확인할 수 있다.**

이와 같이 타겟 피해자인 `carlos`가 `/my-account/something.js` 경로로 요청을 보내게 하면 `/my-account` 페이지가 캐싱되고, 이를 공격자가 경로로 요청을 보내면 해당 페이지를 확인할 수 있으며 최종적으로 API Key를 획득할 수 있다.

<br>

![image](https://github.com/user-attachments/assets/8b5f03d9-3579-4eab-84ce-a7d41333d833)

해당 랩에서 제공하는 *exploit server*를 활용하여 carlos가 url에 접속할 수 있도록 페이로드를 작성하자. 위와 같이 `/my-account/foo.js` 경로에 접속할 수 있도록 body에 script 태그를 추가한다. payload 저장 후 *Deliver exploit to victim*을 통해 전달한다.

<br>

![image](https://github.com/user-attachments/assets/ba210079-9de5-44b4-97da-5685d950cb5f)

바로 같은 경로에 요청을 보내면 캐싱된 페이지를 확인할 수 있고, `carlos`의 API Key를 획득할 수 있다.

<br><br>
