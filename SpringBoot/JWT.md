## JWT란?

인증에 필요한 정보들을 암호화시킨 토큰

클라이언트가 Access Token을 HTTP header에 담아 서버에게 넘기면 서버는 이를 식별함 → Stateless

실행 순서:

로그인 시 클라이언트는 서버로부터 **유일한 토큰**을 발급받아 다음부터 서버에 접속할 때(로그인 유지 시) 토큰값을 요청헤더에 함께 실어 보낸다. 서버는 **Access Token**과 서버가 **실제로 부여한 Token이 일치하는지 확인**하여 사용자를 식별한다. 토큰이 만료되면 refresh Token을 통해 다시 서버로부터 Access Token을 재발급받는다.

## JWT 구조

Header. Payload. Signature

![image](https://user-images.githubusercontent.com/46811084/162614947-c25120a5-471f-420b-a735-249b9e576e9f.png)

1. Header(헤더)
    - algo : 서명 암호화 알고리즘
    - typ : 토큰 유형
2. Payload(내용): 실제 서버-클라이언트 인증 시 사용될 정보
    - sub: 제목
    - name: private claim , 당사자들끼리 정보를 공유하기 위해 사용
3. Signature(서명)
    - 헤더의 인코딩값과 payload의 인코딩값을 합친 후 비밀키로 해쉬화 한 뒤 사용
    - 서버의 비밀키를 알아야 복호화 할 수 있다.
    - Payload가 바뀌면 Signature가 일치하지 않기 때문에 신뢰성 있는 인증이 가능하다.
    

장점

- 데이터 위변조를 막기 때문에 신뢰성 있는 인증이 가능하다.
- 인증 정보에 대한 서버의 별도의 저장소가 필요없다. ( = Stateless )
