## JDBC 등장이유

SQL 벤더사는 MySQL, Oracle 등 여러 개가 있다. 과거에는 벤더사마다 데이터베이스에 connection, SQL문 전달, 결과값 반환 하는 방법이 달랐다. 

![image](https://user-images.githubusercontent.com/46811084/164028284-6c0febd0-2e41-4776-9778-1bac7064871b.png)

따라서 개발자는 다른 데이터베이스로 변경 시 데이터베이스 작동 코드도 변경해야하는 번거로움이 있었다.

JDBC(JAVA Database Connectivity) 는 이를 해결해주기 위해 생긴 표준 인터페이스이다   


> JDBC 드라이버란?
> 

각 벤더사는 JDBC 를 사용하기 위해 구현해 놓은 Driver을 라이브러리로 제공한다. 이것을 JDBC Driver라고 한다. 

![image](https://user-images.githubusercontent.com/46811084/164028896-2ee7e0d2-c30c-4fe7-afac-44425b6d0aa3.png)

→ JDBC 인터페이스를 구현해놓은 각각의 드라이버를 Library에 등록하여 사용한다.

JDBC로 많은 이점이 생겼지만 연결,전달과정을 표준화 한 것이지 SQL문 자체는 표준화 되지 않았다. JAVA ORM인 JPA(Java Persistence API)를 사용하면 개발자가 SQL을 작성하지 않아도 되는 편리함이 있다.

---
## SQL Mapper와 ORM 기술

최근에는 JDBC를 직접 사용하기 보다는 JAVA Application 과 JDBC사이에서 작동하는 SQL Mapper와 ORM기술들을 통해서 JDBC를 보다 편리하게 사용할  수 있다.

> SQL Mapper
> 

![image](https://user-images.githubusercontent.com/46811084/164032197-cb117e43-89c6-49d0-81bf-78dd6b3f86e0.png)


- SQL 응답결과를 객체로 Mapping 해주는 장점이 있다.
- 하지만 여전히 SQL문을 직접 작성해줘야한다.

> ORM
> 

![image](https://user-images.githubusercontent.com/46811084/164032088-e6c62a99-42cb-40fa-b6cf-c1e9cc2ba798.png)


- 객체를 관계형 데이터 베이스와 mapping 해주는 장점이 있다.
- SQL문도 JPA 에서 알아서 작성해준다.
