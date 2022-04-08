## JPQL
: 객체지향적 쿼리 언어

### JPQL을 왜 사용할까?

- JPA는 ```엔티티 객체```를 중심으로 개발을 한다. 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색한다.      
```em.find()```로 아이디로 검색이 가능하지만 특정조건을 가진 데이터를 검색하기에는 ```em.find()```는 한계가 있다.   
  따라서 ```JPA```는 SQL을 추상화한 ```JPQL```이라는 ```객체지향 쿼리 언어```를 사용한다.
- JPQL은 SQL을 추상화해서 특정 데이터 베이스 SQL에 의존하지 않는다. 

### jpql 문법   

select m from Member as m where m.age > 18   

여기서 주의할 점은
1) ```별칭```은 필수로 적어줘야한다.
2) 테이블 이름이 아닌, ```엔티티```이름을 사용한다. 
3) 엔티티와 속성은 대소문자를 구분한다.

#### ```TypeQuery```와 ```Query```
- ```TypeQuery``` : 반환값이 명확할 때 사용, 즉 타입이 한개로 통일되어야하고 정확해야한다.   
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class); 
```

- ```Query```: 반환값이 명황하지 않을 때 사용, 타입이 한개로 통일되어있지 않거나, 잘 모를 때 사용
```java
Query query = em.createQuery("SELECT m.username, m.age from Member m"); 
```

#### 결과조회 API
- ```getResultList()``` : 결과가 하나 이상일 때 리스트를 반환해준다. 결과가 없을 시에는 ```빈 리스트```를 반환한다.
(```Exception```이 터지지 않는다)
- ```getSingleResult()``` : 결과가 정확히 하나 일 때 단일 객체를 반환해준다.   
  이 때 주의할점은
  1) 결과가 없을 시에는 ``` javax.persistence.NoResultException ```
  2) 결과가 여러개일시에는 ``` javax.persistence.NonUniqueResultException ```
  예외가 터진다.


#### 파라미터 바인딩
1) ```이름```기준 파라미터 바인딩
```java
SELECT m FROM Member m where m.username=:username
query.setParameter("username", "지수");
```

2) ```위치```기준 파라미터 바인딩
```java
SELECT m FROM Member m where m.username=?1
query.setParameter(1, usernameParam)
```

--> 되도록이면 ```이름```기준 파라미터 바인딩을 쓰자 !   
```위치```기준 파라미터바인딩은 중간에 질의문을 추가할 경우 헷갈릴 수 있기 때문 


