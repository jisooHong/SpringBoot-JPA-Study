# HTTP API 설계방식

### 1)  POST 기반 등록

> 회원 관리 시스템
> 

회원목록 : /members → GET

회원등록: /members → POST

회원조회: /members/{id} → GET

회원수정: /members/{id} → PATCH,PUT,POST ( 수정은 고민을 해봐야한다)

회원삭제: /members/{id} → DELETE

- **컬렉션(Collection) : 서버가 관리하는 리소스 디렉토리**
- 응답 URI가 `/members/10` 이면  여기서 컬렉션은 `/members` 가 된다.
- 서버가 리소스의 URI 를 생성하고 관리해준다.
- 클라이언트는 등록될 URI를 모른다. ( 서버가 DB와 함께 관리한다 )

---

### 2) PUT 기반 등록

> 파일관리 시스템
> 

파일목록: /files →GET

파일조회: /files/{filename} →GET

**파일등록: /files/{filename} → PUT (기존것을 덮어버려야하기 떄문에 PUT이 적절)**

파일삭제: /files/{filename} → DELETE

파일 대량 등록: /files → POST

- **스토어: 클라이언트가 관리하는 리소스 저장소**
- PUT으로 신규 자원을 등록한다.
- 클라이언트가 URI 를 알고 있어야한다.
- 여기서 스토어는 /files

→ 대부분 실무에서는 POST 기반의 등록을 사용한다. 

 

---

### HTML FORM 사용

:  **GET과 POST만** 사용가능하다!!

> 회원 관리 시스템
> 

회원목록 : /members → GET

회원등록 폼: /members/new → GET

회원등록: /members → POST

회원조회: /members/{id} → GET

회원수정 폼: /members/{id}/edit → GET

회원수정: /members/{id}/edit, /members/{id} → POST

**회원삭제: /members/{id}/delete → POST** 

**컨트롤 URI** 

- GET과 POST만 지원하므로 제약이 있다.
- 이런 제약 떄문에 **동사**로 된 리소스 경로를 사용해야한다.
- /new, /edit, /delete 가 컨트롤 URI이다.
- HTTP 메서드로 해결하기 애매한 경우 사용

→ 실무에서 많이 사용한다.
