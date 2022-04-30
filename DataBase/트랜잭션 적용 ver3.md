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

![image](https://user-images.githubusercontent.com/46811084/166101805-bef36a68-1dcf-456e-8caa-853ab61b3eac.png)    

→ 또한 우리는 ver2 에서 같은 커넥션을 유지하기 위해 파라미터로 커넥션 정보를 넘겨서 사용했다. 트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관하고 커밋 시 보관된 커넥션을 꺼내와 종료한다. 

코드를 살펴보자

```java
//service.class
private final PlatformTransactionManager transactionManager; // 트랜잭션 매니저를 주입받는다.
public void accountTransfer(String fromId,String toId, int money) throws SQLException {
    //트랜잭션 시작
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

    try{
        //비즈니스 로직 수행
        bizLogic(fromId, toId, money);
        transactionManager.commit(status);
    }catch (Exception e){
        transactionManager.rollback(status);
        throw new IllegalStateException(e);
    }
}
```

→ `transactionManager.getTransaction(new DefaultTransactionDefinition())`으로 트랜잭션을 시작한다. 이미 시작된 트랜잭션이 존재하면 그 트랜잭션에 참여한다.

 

```java
private void close(Connection con, Statement stmt, ResultSet rs){ //순서 지키기
    JdbcUtils.closeResultSet(rs);
    JdbcUtils.closeStatement(stmt);
    // 주의! 트랜잭션 동기화를 사용하라면 DataSourceUtils를 사용해야 한다.
    DataSourceUtils.releaseConnection(con,dataSource);
}

private Connection getConnection() throws SQLException {
    // 주의! 트랜잭션 동기화를 사용하라면 DataSourceUtils를 사용해야 한다.
    Connection con = DataSourceUtils.getConnection(dataSource);
    log.info("get connection = {} , class = {}",con,con.getClass());
    return con;
}
```

→ `DataSourceUtils.getConnection(dataSource)` 로 트랜잭션동기화 매니저에 커넥션이 있으면 가져오고 커넥션이 없으면 새로 만들어준다!

`con.close()`를 하면 트랜잭션이 `commit` 되기 전에 커넥션이 닫히게된다. 트랜잭션이 끝날때까지 유지되어야한다!!

 `DataSourceUtils.releaseConnection(con,dataSource);` 로 커넥션을 닫아주지않고 유지해준다.
 
 ```java
//test
@BeforeEach
void before(){
    DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    memberRepository = new MemberRepositoryV3(dataSource);

    PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);

    memberService = new MemberServiceV3_1(transactionManager,memberRepository);
}
```

→ JDBC기술을 사용하기 때문에 `new DataSourceTransactionManager(dataSource)` 

jpa 등을 사용하고 싶을땐 jpa구현기술로 갈아치우면된다.
