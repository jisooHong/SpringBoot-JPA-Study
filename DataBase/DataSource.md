# DataSource
커넥션을 하는 방법은

1) `DriverManager`을 이용하기

2) `ConnectionPool`을 이용하기

이 두가지 방법이 있다.

이 때 , `DriverManager로` 커넥션을 얻어오다가 `HikariCp`로 커넥션을 얻어오기 위해서는 애플리케이션 코드를 바꿔야한다. 

따라서 자바는 `DataSource` 라는 커넥션획득을 추상화한 인터페이스를 제공해준다. 

![image](https://user-images.githubusercontent.com/46811084/164177738-e8ec142e-8819-4843-90ea-9e9c2a8f86e4.png)

대부분의 커넥션 풀은 DataSource를 구현해놓았다. 

따라서 커넥션 연결방법을 바꾸고 싶다면 애플리케이션 코드전체를 변경할 필요 없이 기술만 갈아끼우면 된다. 

또한 DriverManager을 사용한 연결방법도 DriverMangerDataSource도 사용하다가 커넥션 풀로 손쉽게 사용할 수 있다.

---

## `DriverManagerDataSource` VS `DriverManger`

```java

//DriverManger을 통해 획득
DriverManager.getConnection(URL,USERNAME,PASSWORD);

//DriverManagerDataSource를 통해 획득
1) 초기화
DriverManagerDataSource dataSource = new DriverManagerDataSource(URL,USERNAME,PASSWORD);

2) 사용
dataSource.getConnection()
```

- `DirveManager`은 커넥션을 연결 할 때마다 URL,USERNAME,URL을 매번 선언해줘야한다. 따라서 어플리케이션 전반에 URL,USERNAME,PASSWORD가 떠다니고 레파지토리는 매번 이 정보들에 의존한다.
- 그에 반해 `DriverManagerDataSource` 는 초기화 과정에서만 딱 한번 URL,USERNAME, PASSWORD를 전달하고, 사용 부분에서는 어플리케이션에 넘겨주지 않는다. 즉 설정과 사용을 분리했다. 레파지토리는 정보에 의존하지 않고 `DataSource`만 주입받아서 사용하면 된다.

```java
16:07:23.869 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test]
16:07:24.105 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = conn0: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class org.h2.jdbc.JdbcConnection
16:07:24.158 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test]
16:07:24.162 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = conn1: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class org.h2.jdbc.JdbcConnection
```

→ 매번 Connection을 새로 생성해준다.

---

## `HikaraCP` 활용하기

> 커넥션풀 오픈소스인 HikaraCP를 사용해보자
> 

Repository.class

```java
//Repository.class

private final DataSource dataSource;

public MemberRepositoryV1(DataSource dataSource){ // dataSource 주입받기
    this.dataSource = dataSource;
}

// CRUD 코드 생략

public Connection getConnection(){
	Connection connection = dataSource.getConnection()
	log.info("get connection = {} , class = {}",connection,connection.getClass());
  return connection;
}
```

→ 생성자를 통해 `dataSource` 의존성을 주입받는다. ( 생성자로 의존성을 주입받는 이유는 여러 이점이 있기 때문.. 다음에 정리하겠음)

RepositoryTest.class

```java
HikariDataSource dataSource = new HikariDataSource();
dataSource.setJdbcUrl(URL);
dataSource.setUsername(USERNAME);
dataSource.setPassword(PASSWORD);
repository = new MemberRepositoryV1(dataSource);
```

→ `HikariSource`를 초기화 해준 후 `Repository`에 의존성을 주입해준다. 만약 `HikariCP`말고 다른것을 쓰고 싶음 이 부분만 바꾼 후 갈아끼우면 된다.

실행 로그

```java
16:23:34.156 [main] DEBUG com.zaxxer.hikari.HikariConfig - maximumPoolSize................................10
16:23:34.486 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn1: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.491 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn2: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.495 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn3: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.500 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn4: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.504 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn5: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.508 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn6: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.512 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn7: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.516 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn8: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.526 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection conn9: url=jdbc:h2:tcp://localhost/~/test user=SA
16:23:34.527 [HikariPool-1 connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - After adding stats (total=10, active=0, idle=10, waiting=0)
16:23:34.581 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = HikariProxyConnection@1168420930 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class com.zaxxer.hikari.pool.HikariProxyConnection
16:23:34.583 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - resultSize = 1
16:23:34.583 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = HikariProxyConnection@380812044 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class com.zaxxer.hikari.pool.HikariProxyConnection
16:23:34.587 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = HikariProxyConnection@1171434979 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class com.zaxxer.hikari.pool.HikariProxyConnection
16:23:34.591 [main] INFO hello.jdbc.repository.MemberRepositoryV1 - get connection = HikariProxyConnection@464400749 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA , class = class com.zaxxer.hikari.pool.HikariProxyConnection
```

- maximumPoolSize는 default값이 10임을 확인
- Connection이 10개가 만들어진 후 이를 차례대로 가져다 쓰는 것을 확인할 수 있다.
- 여기서 conn0만 계속 가져다 쓰는 이유는 conn0 을 사용하고 반환하기 때문이다.

> ConnectionPool 연결을 확인해보자
> 

test.class

```java
@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
    //커넥션 풀링 : 어플리케이션이 실행될 때 미리 커넥션을 여러개 만들어준다.
    HikariDataSource dataSource = new HikariDataSource();
    dataSource.setJdbcUrl(URL);
    dataSource.setUsername(USERNAME);
    dataSource.setPassword(PASSWORD);
    dataSource.setMaximumPoolSize(10); //default:10
    dataSource.setPoolName("MyPool");

    useDataSource(dataSource);
    Thread.sleep(1000); //풀에 추가되는 걸 보기 위해
}

private void useDataSource(DataSource dataSource) throws SQLException{
    Connection connection1 = dataSource.getConnection();
    Connection connection2 = dataSource.getConnection();
    Connection connection3 = dataSource.getConnection();
    Connection connection4 = dataSource.getConnection();
    Connection connection5 = dataSource.getConnection();
    Connection connection6 = dataSource.getConnection();
    Connection connection7 = dataSource.getConnection();
    Connection connection8 = dataSource.getConnection();
    Connection connection9 = dataSource.getConnection();
    Connection connection10 = dataSource.getConnection();
    Connection connection11 = dataSource.getConnection();
}
```

로그

```java
16:42:57.157 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn1: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.163 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn2: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.166 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn3: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.171 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn4: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.179 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn5: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.184 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn6: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.187 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn7: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.193 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn8: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.197 [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn9: url=jdbc:h2:tcp://localhost/~/test user=SA
16:42:57.252 [MyPool housekeeper] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Pool stats (total=10, active=10, idle=0, waiting=1)
16:42:57.254 [MyPool housekeeper] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Fill pool skipped, pool is at sufficient level.
16:43:27.212 [main] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Timeout failure stats (total=10, active=10, idle=0, waiting=0)
```

→ 커넥션풀에 모든 커넥션을 다 써서 1개가 Waiting 되어 있는 것을 확인할 수 있다.

일정 시간이 지나면 Timeout 으로 실패한다.
