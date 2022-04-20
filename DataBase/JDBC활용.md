JDBC는 데이터베이스 연결을 위한 자바 표준 인터페이스

대표적으로 Connection, Sql문 전달, 결과값 반환 이 세가지를 표준으로 제공한다.

1. Connection

```java
try {
	  Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
	  log.info("get connection = {}, class={}",connection,connection.getClass());
	  return connection;
}catch (SQLException e){
	  throw new IllegalStateException(e);
}
```

- `DriverManager`을 통한 `Connection`
- `DriverManager`은 각각의 DB드라이버를 관리하는 역할을 한다. **매번** `DriverManager`을 통해 `Connection`을 맺어준다. `DriverManager`을 이용하면 “매번” 맺어준다는게 중요하다.
- `URL, USERNAME, PASSWORD` 정보를 이용해 연결한다.

1. SQL 문 전달 및 반환

```java
//저장
public Member save(Member member) throws SQLException{
    String sql = "insert into member(member_id, money) values (?, ?)";

    Connection con = null;
    PreparedStatement pstmt = null; // 데이터베이스에 sql 날리는거 preparedStatement는 statement와 다르게 파라미터 바인딩가능

    try{
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1,member.getMemberId());
        pstmt.setInt(2,member.getMoney());
        pstmt.executeUpdate();
        return member;
    }catch (SQLException e){
        log.error("db error",e);
        throw e;
    }finally {
        close(con,pstmt,null);
    }
}

//조회
public Member findById(String memberId) throws SQLException{
        String sql = "select * from member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1,memberId);

            rs = pstmt.executeQuery();
            if (rs.next()){ // 실행해줘야함
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else{
                throw new NoSuchElementException("memberNotFound memberId = " + memberId);
            }

        }catch(SQLException e){
            log.error("db error", e);
            throw e;
        }finally {
            close(con,pstmt,rs);
        }
    }
```

- `PreparedStatement`는 `Statement`의 자식타입으로 `(?, ?)` 와 같이 파라미터 바인딩이 가능하다.
- `executeUpdate()` 은 결과  `raw` 개수인 `int` 값을 반환해준다.
- `executeQuery()` 는 결과 값인 `ResultSet`을 반환시켜준다.
- 0 번째 값을 가리키고 있던 `Cursor`는 `rs.next()`로 다음 `raw`로 이동한다.
- 순서대로 `ResultSet`, `PreparedStatement`, `Connection`을 꼭 닫아줘야한다!!
- 닫아주지 않으면 리소스 누수로 나중에 커넥션이 부족해져 장애가 발생할 수 있다.

> 삭제로직 작성과 테스트를 해보자
> 

```java
//삭제 로직
public void delete(String memberId) throws SQLException{
	  String sql = "delete from member where member_id = ?";
	
	  Connection con = null;
	  PreparedStatement pstmt = null;
	
	  try{
	      con = getConnection();
	      pstmt = con.prepareStatement(sql);
	      pstmt.setString(1,memberId);
	      pstmt.executeUpdate();
	  }catch (SQLException e){
	      log.info("db error",e);
	      throw e;
	  }finally {
	      close(con,pstmt,null);
	    }
}

//테스트코드 작성
//delete
repositoryV0.delete(memberV0.getMemberId());
Assertions.assertThatThrownBy(()-> repositoryV0.findById(memberV0.getMemberId()))
        .isInstanceOf(NoSuchElementException.class);
```

→ 삭제 테스트 코드는 이미 삭제해준 데이터를 찾았을 때 `NoSuchElementException` 익셉션이 터지는지 확인하도록 작성해야한다.

참고로 NoSuchElementException은 findById() 때 try-catch문 Exception이 발생하도록 구현했다.
