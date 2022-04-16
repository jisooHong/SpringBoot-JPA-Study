좋은 URI설계는? → 리소스와 행위가 잘 분리된거

## 리소스? (= Representation)

ex) 회원 목록조회, 회원 수정, 회원 삭제

→ “회원”  자체가 리소스

HTTP METHOD 로 리소스의 행위를 판별할 수 있다.

## HTTP METHOD

1) GET : 리소스 조회

```html
GET /search?q=hello&hl=ko HTTP/1.1
HOST: www.google.com
```

서버에 전달하고 싶은 데이터는 쿼리 파라미터 혹은 쿼리 스트링을 통해서 전달한다.

메세지 바디로 전달할 수도 있지만, 실무에서 사용하지않음(지원하지 않는 서버가 많아서)

```html
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 34

{
	"username": "JISU"
	"age": 20
}
```

2) POST : 요청 데이터를 처리

```html
POST /members HTTP/1.1
Content-Type: application/json

{
	"username":"hello"
	"age":20
}
```

- 메세지 바디를 통해서 서버로 요청 데이터를 전달한다.
- 새 리소스를 등록한다.(리소스를 모르고 있음)
- 요청 데이터를 처리할 때 , 대용량 프로세스를 처리해야하는 경우 ex)start-delivery, 이 때 새 리소스가 형성되지 않을 수도 있다.
- 다른 메소드로 처리하기 애매할 때

3) PUT : 리소스를 **완전히** 대체, 해당 리소스가 없으면 생성

- 클라이언트가 리소스를 알고있고 URI를 지정한다.
- 리소스를 완전히 대체하기 때문에 모든 필드를 지정해줘야한다. → 따라서 **PATCH**로 부분 변경 사용해서 보완

```html
PUT /members/100 HTTP/1.1
Content-Type: application/json
{
	"username": "hello",
	"age": 20
}
```

4) PATCH: 리소스 부분 변경

5) DELETE: 리소스 삭제

## HTTP METHOD 특징

1) 안전

: 호출해도 리소스가 변경되지 않는다. (로그 장애 같은건 고려하지 않는다. 오로지 리소스 변화만) 

ex) GET , HEAD(get에서 바디부분만 뺀거)

2) 멱등

: 한번 호출하든 두 번 호출하든 결과가 똑같다. f(f(x)) = f(x)

ex) GET, PUT, DELETE

자동복구메커니즘에 사용 가능( 요청처리가 잘 안된거 같을 때 delete를 여러번 해도 되는가? )

멱등은 외부 요인으로 중간에 리소스가 변경되는 것 까지는 고려하지않음, 동일한 사용자가 동일한 요청을 한것만 고려

3) 캐시가능

: 응답 결과 리소스를 캐시해서 사용해도 되는가?

GET, HEAD, POST, PATCH 캐시 가능

실무에서는 GET만 주로 캐시로 사용한다. POST등은 바디도 함께 캐시해야해서 쉽지않음
