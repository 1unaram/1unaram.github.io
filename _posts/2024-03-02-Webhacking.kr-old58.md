---
title: '[Wargame] Webhacking.kr old-58 (Javascript)'
date: 2024-03-02 00:00:00 +0900
last_modified_at: 2024-03-02
categories: [Wargame, webhacking.kr]
tags: [webhacking, webhacking.kr]
published: True
---

## 🚩 문제 파악

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e51a397e-16fa-4004-a6b6-28169eee9988)

문제에 접속하면 채팅창 모습을 확인할 수 있다. 아무 값을 입력하니 명령어를 찾을 수 없는 문구를 확인할 수 있는 것으로 보아 CLI를 구현해 놓은 것으로 유추할 수 있다.

<br>

`help` 명령어를 입력하니 사용가능한 명령어를 확인할 수 있었고, 다른 명령어를 실행한 결과 위와 같은 응답을 확인할 수 있다.

`id` 명령어를 통해 현재 사용자가 admin 권한을 가지고 있는 것을 확인하였으나, `flag` 명령어를 사용할 수 없는 것을 확인할 수 있다.

소스코드를 확인해보자.

<br>

```javascript
$(function () {
    var username = "guest";
    var socket = io();
    $('form').submit(function(e){
    e.preventDefault();
    socket.emit('cmd',username+":"+$('#m').val());
    $('#m').val('');
    return false;
    });
    socket.on('cmd', function(msg){
    $('#messages').append($('<li>').text(msg));
    });
});
```

`script` 태그 내에서 위와 같은 javascript 코드를 확인할 수 있었다. 해당 스크립트의 역할을 분석하면 다음과 같다.

1. `var socket = io();`

    위 명령을 통해 **Socket.io** 모듈을 사용한다. Socket.io는 node.js 기반 실시간 웹 애플리케이션 지원 라이브러리로, 실시간 채팅을 구현할 때 사용된다.

2. `socket.emit('cmd',username+":"+$('#m').val());`

    socket.emit 명령어는 서버 쪽에서 event를 발생시키는 함수로, 첫 번째 인자는 event의 이름을, 두 번째 인자로는 해당 소켓을 통해 클라이언트에게 메시지를 전달한다.
    사용자가 입력한 값을 cmd 명령어로 실행한다. 이때, username 변수 값을 명령 앞에 붙여 사용하는데 이는 해당 사용자의 권한으로 명령어를 실행함을 의미한다. 여기서 username 변수 값은 'guest'로 되어있다.

3. `socket.on('cmd', function(msg){ $('#messages').append($('<li>').text(msg)); });`

    socket.on 명령어는 첫 번째 인자로 받은 event가 클라이언트에서 emit 되면, 두 번째 인자로 받은 콜백 함수를 실행한다. 응답을 수신하는 기능으로 위에서 전달한 명령어를 실행한 결과 값을 출력하는 역할을 한다.


<br>

우리는 socket 통신을 통해 admin으로 flag 명령어를 실행하고 결과 값을 수신 받으면 되는 문제이다.

---

<br><br>


## 🚩 문제 풀이

---

```javascript
var socket = io();  // 소켓 통신을 시작

socket.emit('cmd','admin:flag');  // cmd 이벤트로 admin:flag를 실행하도록 한다
socket.on('cmd', function(res) {console.log(res)});  // cmd 명령 실행 값을 콘솔 창에 출력한다.
```

콘솔 창에서 위의 코드를 실행한다.

<br>

![image](https://github.com/1unaram/1unaram.github.io/assets/37824335/e454d185-a177-45ad-8a0f-67adad24bc0e)

그럼 위와 같이 flag 값을 획득할 수 있다!


