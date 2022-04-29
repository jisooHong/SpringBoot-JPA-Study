# 트랜잭션 적용 ver3 - 트랜잭션 매니저 사용
![image](https://user-images.githubusercontent.com/46811084/165892273-298cfc3d-fd8a-47e2-bb4b-97e0f659bd4e.png)

→ 대부분의 어플리케이션은 이렇게 3가지로 구조를 나누게 된다.

이 중 Service 는 비즈니스로직을 처리하는 곳이다. 

직전에 실습해본 트랜잭션 적용 ver2의 service는 트랜잭션을 시작하기 위해 jdbc 접근 기술인

`javax.sql.DataSource` , `java.sql.Connection` ,
`java.sql.SQLException`

 이 세가지를 사용했다.

만약 jdbc에서 jpa로 데이터접근기술을 변경하게 되면 service안의 코드도 변경해야한다. 즉 유지보수를 하기 힘들다. (jpa와 jdbc는 데이터 접근 코드가 다르다)
  
![image](https://user-images.githubusercontent.com/46811084/165892218-1313e69c-ed6f-4ec9-8842-e2b2fa21b47d.png)   
→ service는 jdbc 트랜잭션에 의존해있다. 여기서 jpa로 기술을 바꾸면?

![image](https://user-images.githubusercontent.com/46811084/165892502-a7fefc77-0737-40bc-a648-3d5b1c1b54e1.png)   
→ 해당 서비스는 사용할 수 없게 된다. jpa로 안에 코드를 싹 바꿔야함

따라서! 서비스에서는 특정 트랜잭션에 의존하지 않고 트랜잭션 기능을 추상화한 인터페이스를 사용하는게 적절하다. 

![image](https://user-images.githubusercontent.com/46811084/165892951-1861eb08-6c8e-4e07-92b3-4692f0e2ac72.png)   
→ 스프링은 미리 이런 고민을 다 해놨다. 스프링에서 제공해주는 `PlatformTransferManager` 인터페이스를 사용하여 트랜잭션을 사용하면 된다!
