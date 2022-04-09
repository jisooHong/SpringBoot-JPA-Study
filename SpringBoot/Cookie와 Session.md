# 쿠키와 세션

HTTP 프로토콜은 서버 자원절약 및 서버 증진을 위해 기본적으로 클라이언트의 정보를 서버가 저장하지 않는 **Stateless** 와 클라이언트와 서버의 연결을 유지하지않는 **Connectionless** 속성을 가지고 있다.

### Stateless(무상태)

![image](https://user-images.githubusercontent.com/46811084/162576688-03f4f51b-c748-40f0-b4ef-37ba8238f927.png)

### Connectionless(비연결성)

![image](https://user-images.githubusercontent.com/46811084/162576697-84bfc458-b708-4a4c-bf18-1a30794d922e.png)

하지만 이로 인해 사용자를 인식하지 못해 같은 사용자가 요청을 하더라도 매번 새로 인증을 해야하는 문제점이 있다. 이를 보완하기 위해서 쿠키와 세션을 사용한다.

---

## 쿠키

![image](https://user-images.githubusercontent.com/46811084/162576708-8aa9bcaa-ef2c-470d-a64c-8492b6dfe37c.png)

클라이언트가 어느 웹사이트에 방문할 때, 클라이언트의 정보를 클라이언트가 저장한다.

1. 클라이언트가 서버 에 (최초) 방문할 때, 서버는 저장정보를 응답헤더의 `Set-Cookie`에 담아 response한다.
2. 다음 접속 시 클라이언트는  서버에게 전에 전달 받은 정보를 요청헤더의 `Cookie`에 담아서 request 한다. → 따라서 보안에 취약함, 민감정보는 쿠키에 저장하지 않는게 좋다.
3. 서버는 `Cookie`에 담긴 정보를 가지고 사용자를 인식한다.

`key-value` 형태의 `set-cookie` 예시

set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; [domain=.google.com](http://domain%3D.google.com/); Secure

- expires : 쿠키 만료일
- domain: 쿠키를 전송할 도메인 , domain을 명시할 경우는 서브 도메인도 쿠키 접근이 가능하고 domain을 명시하지 않는 경우는 해당 도메인만 쿠키 접근
- path: 이 경로를 포함한 하위 페이지에만 쿠키 접근 ( 주로 / 로 설정 )
- Secure : https 에만 쿠키 전송

쿠키의 단점:

- 클라이언트에 저장을 하기 때문에 보안 취약
- 용량제한(도메인당 20개, 1 쿠키당 400kb)
- 쿠키의 사이즈가 커질수록 네트워크에 과부화가 심해진다.

---

## 세션

사용자 정보 파일을 서버에 저장하고 관리한다.

1. 클라이언트가 서버에게 요청
2. 서버측에서 처음 보는 클라이언트면 고유한 sessionId를 만들어서 클라이언트에게 전달
3. 클라이언트는 서버에서 받은 sessionId를 쿠키에 저장하고 이후 요청에서는 sessionId를 함께 담아 요청
4. 서버는 sessionId에 따라 세션저장소를 조회해서 응답

장점:

- 서버에서 세션이 삭제되거나 브라우저를 닫으면 삭제되므로, 쿠키보다 보안에 좋음
- 저장데이터 용량 제한 없음

단점

- 클라이언트가 많아지면 서버 과부화가 생길 수 있다.
- 세션아이디를 중간에 낚아채 클라이인트인척 위장 할 수 있다.
