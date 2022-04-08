## JPA란?
java persistence api
orm 기술 표준
orm이란? : object -relational mapping(객체 관계 매핑) 중간에 orm framework가 이 둘을 매핑해준다. orm은 객체와 RDB 두 기둥 위에 있는 기술

![image](https://user-images.githubusercontent.com/46811084/143529863-a0b2bb39-bdfd-40bb-b829-b15c2e229f4f.png)

#### JPA는 Application과 JDBC 사이에서 작동한다.



### jpa를 왜 사용해야하는가?
### : sql중심적 개발에서 객체 중심으로 개발한다. 

### jpa의 장점
- 패러다임의 불일치를 해결해줌
- persist하면 알아서 insert쿼리 두개를 날려준다. 
- find할때도 마찬가지 join해서 알아서 가져와줌
- 자유로운 객체 그래프를 탐색할 수 있다. 지연로딩 옵션이 있음(원할때 가능)

### jpa특징
1. 1차 캐시와 동일성 보장 
: 같은 트랜잭션에서 두번 이상 조회할때 캐싱해서 받아옴 = sql 한번만 실행
2. 트랜잭션을 지원하는 쓰기 지연 (버퍼링)
	- 트랜색션을 커밋할 때까지 INSERT SQL을 모은다.
	- JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송 

3. 지연로딩과 즉시로딩: jpa는 옵션하나로 튜닝이 된다. 
지연로딩: 객체가 실제 사용될 때 로딩
member가져올때 team을 가져오지 않음 team 쓸 일 있을때만 가져옴

즉시로딩 : JOIN SQL로 한번에 연관된 객체까지 미리 조회
