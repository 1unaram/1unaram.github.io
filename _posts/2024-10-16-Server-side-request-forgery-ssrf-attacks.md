---
title: '[PortSwigger] Server-side request forgery (SSRF) attacks'
date: 2024-10-16 00:00:00
categories: [Study, PortSwigger]
tags: [webhacking, portswigger]
published: False
---

> [[PortSwigger Academy Server-side request forgery (SSRF) attacks](https://portswigger.net/web-security/learning-paths/ssrf-attacks)] 을 수강하고 정리하였습니다.
{: .prompt-info }

<br><br>

# #What is SSRF?

---

서버 사이드 요청 위조(SSRF)는 공격자가 서버 측 애플리케이션이 의도되지 않은 위치로 요청을 보내도록 만드는 웹 보안 취약점이다.

일반적인 SSRF 공격에서, 공격자는 서버가 조직의 인프라 내에서 내부 전용 서비스에 연결하도록 할 수 있다. 다른 경우에는, 서버가 임의의 외부 시스템에 연결을 하도록 만드는 것이 가능할 수도 있다. 이는 인증 크레덴셜과 같은 민감한 데이터를 유출시킬 수 있다.

<br><br>

# #What is the impact of SSRF attacks?

---

성공적인 SSRF 공격은 자주 조직 내에서 인가받지 않은 동작이나 데이터에 대한 접근을 발생시킨다. 이는 취약한 애플리케이션이나 애플리케이션이 소통할 수 있는 다른 백엔드 시스템에서가 될 수 있다. 몇몇의 경우에는 SSRF 취약점이 공격자가 임의의 명령어 실행을 수행할 수 있게끔 할 수도 있다. 외부의 서드파티 시스템에 연결을 유발할 수 있는 SSRF 익스플로잇으로 인해 악의적인 공격이 발생할 수 있다. 이는 취약한 애플리케이션을 호스팅하는 조직에서 발생한 것으로 보일 수 있다.

<br><br>

# #Common SSRF attacks

---

SSRF 공격은 신뢰 관계를 악용하여 취약한 애플리케이션에서 공격의 권한을 상승시키거나 인가되지 않은 동작을 수행한다. 이러한 신뢰 관계는 서버와 관련하여 존재하거나 같은 조직 내의 다른 백엔드 시스템과 관련할 수 있다.

<br>

## SSRF attacks against the server

**서버를 대상으로 하는** SSRF 공격에서 공격자는 루프백 네트워크 인터페이스를 통해서 애플리케이션이 애플리케이션을 호스팅하고 있는 서버로 다시 HTTP 요청을 보내도록 할 수 있다. 여기에는 일반적으로 호스트 `127.0.0.1`이나 `localhost`와 같은 호스트명이 포함된 URL을 제공하는 작업이 포함된다.

<br>

예를 들어, 사용자가 한 아이템의 재고가 특정한 상점에 있는지를 확인할 수 있도록 하는 쇼핑 애플리케이션이 있다고 해보자.

<br>

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

제고 정보를 제공하기 위해서는 애플리케이션은 다양한 백앤드 REST API 쿼리를 해야한다. 이것은 프론트엔드 HTTP 요청을 통해 관련 백엔드 API 엔드포인트에 URL을 전달함으로써 진행된다. 사용자가 한 아이템의 재고 상태를 볼 때, 그들의 브라우저는 위와 같은 요청을 수행할 것이다.

<br>

서버는 `/admin` URL의 내용을 가져오고 이를 사용자에게 전달한다. 공격자는 `/admin` URL에 방문할 수 있으나, 정상적으로는 관리자 기능은 인가된 사용자에게만 접근할 수 있어야 한다. 이 말은 공격자가 관심있는 그 어느 것도 볼 수 없다는 뜻이다. 그러나, `/admin` URL에 대한 요청이 로컬 시스템에서 온다면, 정상적인 접근 제어는 우회된다. 이 애플리케이션은 관리자 기능에 대한 전체 액세스 권한을 부여하는데, 이는 요청이 신뢰하는 곳에서부터 오는 것처럼 보이기 때문이다.

<br>

왜 애플리케이션은 이러한 방식으로 동작하며, 암묵적으로 로컬 시스템에서부터 오는 요청을 신뢰하는가? 이는 다양한 다음의 이유들이 있다.

<br>

- 접근 통제 확인은 애플리케이션 서버 앞에 놓여있는 다른 구성요소에서 구현될 수 있다. 서버에 다시 연결되면, 검사는 우회된다.
- 재해 복구 목적으로, 애플리케이션은 로컬 시스템으로부터 오는 어떠한 사용자에게든 로그인 없이 관리자 접근을 허용할 수 있다. 이는 관리자가 자격 증명을 잃어버렸을 때 시스템을 복구할 수 있는 방법을 제공한다. 이것은 오직 완전히 믿을 수 있는 사용자가 서버에서 직접 왔다고 가정한다.
- 관리자 인터페이스는 애플리케이션과 다른 포트 번호에서 서비스 되고, 사용자에게 직접적으로 노출되지 않는다.

<br>

로컬 시스템에서 오는 요청이 평범한 요청과 다르게 다루어지는 이러한 종류의 신뢰 관계는 SSRF를 치명적인 취약점으로 만들곤 한다.

<br>

## 🚩Lab: Basic SSRF against the local server

문제 설명과 같이 이 랩은 내부 시스템으로부터 데이터를 가져오는 재고 확인 기능을 가지고 있다. 임의의 상품 페이지의 하단에 도시별로 재고를 조회할 수 있는 기능이 존재하는 것을 확인할 수 있다. 재고를 조회할 때 요청하는 패킷에는 `stockApi` 라는 필드가 존재하고 이곳에 요청하고자 하는 API의 URL을 전달하는 것을 알 수 있다.

<br>

![image](https://github.com/user-attachments/assets/3f17cb83-fc42-4bfd-a263-10eb90e95e49)

이 필드 값에 `http://localhost/admin/delete?user=carlos`와 같이 `carlos` 사용자를 지우기 위한 url과 파라미터명을 유추하여 요청 url을 구성한 후에 패킷을 전송하면, 응답으로 **"Missing parameter 'username'"**를 받을 수 있다. 이를 통해 파라미터명이 `username`이라는 사실을 알아낼 수 있다.

<br>

이제 요청 url을 `http://localhost/admin/delete?username=carlos`로 구성하여 패킷을 전송하면 랩을 해결할 수 있다.
