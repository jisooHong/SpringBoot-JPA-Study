## JPA 구동 방식
![image](https://user-images.githubusercontent.com/46811084/143590283-18a37a5e-5ff8-4420-a868-887d6b706298.png)
1. META-INF/persistence.xml 에서 데이터베이스 설정 정보를 조회한다. 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value=" jdbc:h2:tcp://localhost/~/hello-jpa"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.jdbc.batch_size" value="10"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```
2. EntityManagerFactory 생성
- 이때 주의할 점은 엔티티매니저팩토리는 하나만 생성하여 애플리케이션 전체에서 공유해야한다.

3. EntityManager 생성
- 필요할 때마다 항상 생성해주고 close() 해줘야한다.
- 스레드간 공유 불가능하다.
- 꼭 한 트랜잭션 안에서 데이터를 관리해야한다.

```java
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member1 = new Member(140L,"A");
            em.persist(member2);
            tx.commit();
        }catch (Exception e){
            tx.rollback();
        }finally {
            em.close();
        }
        emf.close();
```
