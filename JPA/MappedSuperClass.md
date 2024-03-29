## MappedSuperClass ( 매핑 정보 상속 )

![image](https://user-images.githubusercontent.com/46811084/145061555-24884135-68cc-469d-8c3e-191d7429ec53.png)

이처럼 공통 컬럼을 한번에 사용하고 싶을 때 사용한다.   
```extends 공통클래스``` 형식으로 사용

주의 할 점
- 상속 관계 매핑과는 확연히 다르다!
- 상속 관계 매핑은 테이블과 관련이 있는 반면 ( ```@Entity``` 어노테이션 사용 ) , ```MappedSuperClass```는 엔티티가 아니다. 즉 테이블이 만들어지지 않는다.
- 공통의 속성만 내려주는 역할
- 따라서 <**부모타입 조회**>가 불가능하다.
- 직접 생성해서 사용할 일이 없기 때문에 추상 클래스로 만드는 걸 권장한다. 
---------------------------------------------------------------------------------------------------------------------------------------------
1. 공통 클래스 생성
```java
@MappedSuperclass
public class BaseEntity {
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
    
    //getter , setter 생성
}
```

2. 공통클래스를 사용할 클래스는 ```public class Member extends BaseEntity```로 사용 가능하다.

3. BaseEntity 의 속성들이 Member 테이블 컬럼으로 생성된 것을 확일할 수 있다.  

![image](https://user-images.githubusercontent.com/46811084/145063122-b76018bc-7cfb-4590-b9f8-affd1707b56e.png)
