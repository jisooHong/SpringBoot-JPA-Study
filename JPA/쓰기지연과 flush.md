### 쓰기 지연 저장소

```java
            Member member1 = new Member(140L,"A");
            Member member2 = new Member(160L,"B");

            //영속 -> 이 때 db에 저장되는건 아니다. 그럼 언제 쿼리가 날라갈까?
            em.persist(member1); 
            em.persist(member2);

            tx.commit();//모았다가 이 때 한번에 쿼리날라감
```

위의 코드에서 언제 insert 쿼리가 날라갈까?
em.persist(member1) 을 할때 저장이 되는가 싶었지만, 아니었다.

- persist를 해주면 영속성 컨텍스트의 **쓰기 지연 저장소** 에 sql문이 저장이 된다.
- 트랜잭션을 commit을 해주면 한번에 쓰기 지연 저장소에 있던 sql문들이 한번에 db에 날라간다.

![image](https://user-images.githubusercontent.com/46811084/143596551-68cff673-c41a-4ace-be8d-bc69baeac2f2.png)


### DIRTY CHECKING
: 상태 변경 검사
- JPA에서는 트랜잭션이 끝나는 시점에 변화가 있는 모든 엔티티 객체를 자동으로 DB에 반영해준다.

데이터를 조회할 때
1. 해당 엔티티 스냅샷을 만든다.
2. 트랜잭션이 끝나면 위에 스냅샷과 비교해서 다르면 update query 를 전달한다. 이 때는 영속성 컨텍스트가 관리하는 엔티티만 적용한다.

![image](https://user-images.githubusercontent.com/46811084/143596671-0d405a88-ed4d-461a-9936-8933b878d21a.png)


### Flush
: 영속성 컨텍스트의 변경내용을 데이터베이스에 반영한다. 즉 쓰기 지연 SQL의 저장소의 쿼리를 데이터베이스에 저장한다.
 ***플러시는 영속성 컨텍스트를 비우지 않는다.***

