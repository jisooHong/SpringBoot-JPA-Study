## CASCADE : 영속성 전이

특정 엔티티를 ```영속상태```로 만들 때 연관된 엔티티도 함께 ```영속 상태```로 만들고 싶을 때는 어떻게 할까?

![image](https://user-images.githubusercontent.com/46811084/145340915-87ed0e0c-241c-4969-ba1c-878865e83f3c.png)

```@OneToMany(mappedBy="parent", cascade = CascadeType.PERSIST) ```    
: ```parent```를 ```영속성 컨텍스트```에 올릴때 자동으로 ```child``` 들도 ```영속성 컨텍스트```에 올라온다.

```java
Parent parent = new Parent();
Child child1 = new Child();

parent.addChild(child1);

em.persist(parent);
```
-> 이렇게 ```parent```만 ```persist```를 해줘도 ```child```도 모두 ```insert```되는 것을 볼 수 있다.

< CASECADE의 종류 >
- ```ALL```: 모두 적용
- ```PERSIST```: 영속
- ```REMOVE```: 삭제
- ```MERGE```: 병합
- ```REFRESH```
- ```DETACH```

--------------------------------------------------------------------------------------------------------------------
## 고아객체
: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제해준다. 

```java
// PARENT class
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> childList = new ArrayList<>();
```

```java
// main
Parent parent1 = em.find(Parent.class, parent.getId());
parent1.getChildList().remove(0);
em.persist(parent1);
```

-> 이렇게 ```orphanRemoval = True``` 를 해준 후 ```parent```의 ```child```의 연관관계를 끊어버리면 실제 ```child``` 는 ```delete``` 된다.

```
Hibernate: 
    /* delete hellojpa.Child */ delete 
        from
            Child 
        where
            id=?

```


---------------------------------------------------------------------------------------------------------------
## CascadeType.ALL + orphanRemovel=true
: 이렇게 두가지를 같이 쓰면 부모로부터 자식의 생명주기를 관리할 수 있다.   
자식의 Persist도 부모를 통해 한번에 가능하고, 부모로부터 자식의 삭제도 가능하다.  
