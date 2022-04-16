# Http 상태코드
- 1XX (Informational)  : 요청이 수신되어 처리중
- 2XX (Successful) : 요청 정상 처리
- 3XX (Redirection) : 요청을 완료하려면 추가 행동이 필요
- 4XX (Client Error) : 클라이언트 오류, 잘못된 문법 등으로 서버가 요청을 수행할 수 없음
- 5XX (Server Error) :  서버 오류, 서버가 정상 요청을 처리하지 못함

---

2XX: 성공

- 200 OK → GET시 응답을 잘 줄 때 사용
- 201 Created → POST로 서버에서 자원을 잘 생성했을 때 사용
- 202 Accepted → 요청이 접수는 됐지만 처리가 완료되지 않음 ( 배치 처리 같은 곳에 사용 )
- 204 No Content → 응답 페이로드 본문에 보낼 데이터가 없을 때

---

3XX: 리다이렉션

요청을 완료하기 위해 유저 에이전트(웹브라우저) 의 추가 조치 필요

리다이렉션이란?

![image](https://user-images.githubusercontent.com/46811084/163679911-024ab62e-80d5-4c4a-a2e7-610128b28c52.png)


> 영구 리다이렉션
> 
- 특정 리소스의 uri가 영구적으로 이동
- 301 Moved Permanently : 리다이렉트시 요청 메서드가 GET으로 변하고, **본문 자체가 제거**될 수 있음 → 실무에서는 대부분 301 씀
- 308 Permanent Redirect : 요청 메소드와 바디가 그대로 남아있음

![image](https://user-images.githubusercontent.com/46811084/163679925-055cff5e-d5bc-4911-aac7-06a91bf42109.png)


> 일시 리다이렉션: 일시적인 변경, 주문완료 후 주문 내역 화면으로 이동 , PRG (POST / REDIRECT / GET)
> 
- 302 Found : 리다이렉트시 요청 메서드가 GET으로 변하고 본문이 제거될 수 있음
- 307 Temporary Redirect : 리다이렉트시 요청메서드와 본문을 유지한다.
- 303 See Other : 302와 기능 같음
- **304 NOT MODIFIED** : 클라이언트에게 리소스가 수정되지 않았음을 알려준다. 따라서 클라이언트는 로컬 PC에 저장된 캐시를 재사용하면 된다( 캐시로 리다이렉트한다 )

> **PRG** : POST/REDIRECT/GET
> 

EX)

주문완료 후 새로고침을 해주면? → POST요청이 또 가서 중복주문이 될 수 있다.

따라서 주문완료가 되면 주문 완료 페이지를 띄어줄 수 있게 302 FOUND로 응답하고 GET 요청을 통해 주문 완료 페이지를 띄우게 해준다.  

이 때 새로고침이 들어오면 처음에 보낸 POST 요청이 아닌 REDIRECT 요청인 GET이 실행되기 때문에 증복주문은 없어짐

![image](https://user-images.githubusercontent.com/46811084/163679939-ce19feb7-3c67-44fd-845a-347bb5b1b931.png)
> 특수 리다이렉션: 결과 대신에 캐시를 사용해라
> 

---

4XX: 클라이언트 오류, 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 떄문에 똑같은 재시도가 실패함, 요청 수정해야함

- 400 BAD REQUEST : 요청 구문, 메시지 등등 오류, 요청 파라미터가 잘못되거나 API스펙이 맞지않을 때! 서버 개발자들이 명확하게 보내줘야함
- 401 Unauthorized : 클라이언트가 해당 리소스에 대한 **인증**이 필요함
    - 인증( Authentication) : 본인이 누구인지 (로그인체크)
- 403 Forbidden: 인증은 됐지만 **인가가 안됨**
    - 인가(Authoriztaion) : 권한이 있는지, 인증이 있어야 **인가**가 가능
- 404 Not Found : 요청 리소스를 찾을 수 없음 , url등을 잘못 보냈을때

---

5XX: 서버 오류 , null point exception , 디비 터질때등등

- 500 Internal Sever Error:  서버 내부 오류 대부분 이거
- 503 Service Unavailable : 서버 일시적 과부화
