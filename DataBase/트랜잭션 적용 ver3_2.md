## 트랜잭션 적용 ver3_2
> 트랜잭션 템플릿을 사용하여 트랜잭션 반복 코드를 줄이자.
> 

```java
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

→ 트랜잭션을 적용하는 부분을 보면 `try - catch` 구문안에 트랜잭션이 성공하면 `commit` , 실패하면 `rollback` 이 된다. 모든 트랜잭션을 이렇게 동작한다.

트랜잭션템플릿을 사용하면 이 코드를 줄일 수 있다!!

```java
public class MemberServiceV3_2 {
	private final TransactionTemplate txTemplate;
	private final MemberRepositoryV3 memberRepository;
	
	public MemberServiceV3_2(PlatformTransactionManager transactionManager, MemberRepositoryV3 memberRepository) {
	    this.txTemplate = new TransactionTemplate(transactionManager);
	    this.memberRepository = memberRepository;
	}
	
	public void accountTransfer(String fromId, String toId, int money) throws SQLException {
	
	    txTemplate.executeWithoutResult((status) -> { // 트랜택션 시작하는 템플릿
	        try {
	            bizLogic(fromId,toId,money); // 정상 수행되면 커밋
	        } catch (SQLException e) {
	            throw new IllegalStateException(e);//언체크예외외
	       }
	    });
	
	}
```

→ `TransactionTemplate` 을 이용하여 비즈니스로직이 정상수행되면 `commit`, 아니면 `rollback` 이 수행된다. 

하지만 아직까지 서비스에 트랜잭션을 처리하는 코드가 남아있다. 서비스에 순수 비즈니스 로직만을 사용하기 위해서는 어떻게 해야할까? 다음에는 이를 해결하기 위해 `트랜잭션 AOP`를 사용할 예정이다.
