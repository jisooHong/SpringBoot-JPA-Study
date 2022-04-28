# 트랜잭션 적용 ver1,2
계좌이체와 같은 비즈니스 로직이 동작할 때 트랜잭션이 적용되어 있지 않으면 어떻게 될까?

```java
@RequiredArgsConstructor
public class MemberServiceV1 {
    private final MemberRepositoryV1 memberRepositoryV1;

    public void accountTransfer(String fromId,String toId, int money) throws SQLException {
        Member fromMember = memberRepositoryV1.findById(fromId);
        Member toMember = memberRepositoryV1.findById(toId);

        memberRepositoryV1.update(fromId,fromMember.getMoney()-money);
        validation(toMember);
        memberRepositoryV1.update(toId,toMember.getMoney()+money);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")){
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

Member의 아이디값이 “ex”일 경우 예외가 발생하여 비즈니스로직이 중단된다.

즉 이체가 정상처리 되지 않는다. 

```java
@Test
    @DisplayName("이체 중 예외 발생")
    void accountTransferEx() throws SQLException {
        //given
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);
        memberRepository.save(memberA);
        memberRepository.save(memberEx);

        //when
        Assertions.assertThatThrownBy(()-> memberService.accountTransfer(memberA.getMemberId(),MEMBER_EX,2000))
                .isInstanceOf(IllegalStateException.class);
        //then 검증
        Member findMemberA = memberRepository.findById(memberA.getMemberId());
        Member findMemberB = memberRepository.findById(memberEx.getMemberId());
        Assertions.assertThat(findMemberA.getMoney()).isEqualTo(8000);
        Assertions.assertThat(findMemberB.getMoney()).isEqualTo(12000);

    }
```

→ 아이디값이 ex이기 때문에 `IllegalStateException` 예외가 발생했다. A의 계좌에는 2000원이 출금됐지만 ex의 계좌에는 2000원이 입금되지 않아야한다. 따라서 이 테스트 케이스는 실패한다.

따라서 계좌이체 등 한가지 단위의 일을 할 때는 트랜잭션이 적용되어야한다. 

중간에 예외발생등으로 실패하면 rollback이 되어야 한다.

![image](https://user-images.githubusercontent.com/46811084/165774620-c35c4273-06b0-4e16-a584-f1e192cb0aeb.png)

비즈니스 로직 전 후에 트랜잭션을 적용시키자. 그 전에 트랜잭션이 시작할거면 커넥션이 생성되어야한다. 

같은 세션을 사용하기 위해서는 커넥션은 트랜잭션 실행중에는 항상 같아야한다!!

파라미터를 통해 커넥션 정보를 넘기면 커넥션을 같게 유지할 수 있다. 

```java
public class MemberServiceV2 {
    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;

    public void accountTransfer(String fromId,String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try{
            con.setAutoCommit(false); //트랜잭션 시작
            //비즈니스 로직 수행
            bizLogic(con, fromId, toId, money);
        }catch (Exception e){
            con.rollback();
            throw new IllegalStateException(e);
        }finally {
            release(con);
        }
    }

    private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(con, fromId);
        Member toMember = memberRepository.findById(con, toId);

        memberRepository.update(con, fromId,fromMember.getMoney()- money);
        validation(toMember);
        memberRepository.update(con, toId,toMember.getMoney()+ money);
        con.commit(); //성공 시 커밋
    }

    private void release(Connection con) {
        if(con != null){
            try{
                con.setAutoCommit(true); // 커넥션 풀에 돌려주기 전에 오토커밋 true로 다시 바꿔주기
                con.close();
            }catch (Exception e){
                log.info("error",e);
            }
        }
    }
```

- `con.setAutoCommit(false)`: 트랜잭션을 시작한다
- `bizLogic` 을 실행할 때 커넥션은 항상 동일해야한다. 따라서 `repository`에 `connection`정보를 함께 파라미터로 넘겨준다.
- 커넥션을 닫아줄 때 기본값인 `con.setAutoCommit(true);` 로 변경해주자~!

```java
//MemberRepositoryV2
public Member findById(Connection con, String memberId) throws SQLException{
      String sql = "select * from member where member_id = ?";

      PreparedStatement pstmt = null;
      ResultSet rs = null;

      try{
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
          // 커넥션은 여기서 닫지 않는다.
          JdbcUtils.closeResultSet(rs);
          JdbcUtils.closeStatement(pstmt);
      }
 }

public void update(Connection con, String memberId, int money) throws SQLException{
    String sql = "update member set money=? where member_id=?";

    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try{
        pstmt = con.prepareStatement(sql);
        pstmt.setInt(1,money);
        pstmt.setString(2,memberId);
        int resultSize = pstmt.executeUpdate(); // 쿼리 수행후의 raw수 반환
        log.info("resultSize = {}",resultSize);
    }catch(SQLException e){
        log.error("db error", e);
        throw e;
    }finally {
        //커넥션 닫으면 안됨됨            JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(pstmt);
    }
}
```

- `bizLogic` 에서 사용하는 `findById` 와 `update` 메소드에 `connection`을 파라미터로 받아와서 사용한다. `getConnection()` 또 받아 오지 않는 것에 주의해야한다!!
- 주의할 점!! `connection`을 닫아주면 안된다
