# AOP(관점지향프로그래밍)
: 객체별로 처리했던것을(OOP) 관점별(AOP)로 처리하는 행위


객체지향프로그래밍 (OOP)로 객체를 재사용함으로써 반복되는 코드를 많이 줄일 수 있었다. 하지만 객체의 재사용에도 불구하고 반복되는 코드들이 존재한다. 

![image](https://user-images.githubusercontent.com/46811084/162377376-e57e5433-c038-4d4a-a530-1d5c29a352f8.png)


예를들면 트랜잭션 , 로깅(시간 체크 등), 권한 체크 등 공통된 코드를 ‘**횡단적 관심사**’ 라고 한다. `OOP`에서는 이 기능들을 횡단으로 각 객체의 횡단으로 입력하였다면 `AOP`를 통해 종단으로 처리가 가능하다.


이 횡단적 관심사를 `Aspect` 클래스로 모듈화해준다. `AOP`는 이 모듈을 필요한 곳에서 **호출하지 않고** 미리 등록해주어 필요한 곳에 자동 실행되게 한다. 즉 호출부분까지 `AOP`로 뽑아내는것

## 실행위치와 순서

![image](https://user-images.githubusercontent.com/46811084/162377073-882f1109-d439-4406-8b24-f12557359237.png)

![image](https://user-images.githubusercontent.com/46811084/162377139-d80e0219-b7ba-40f5-a440-252247d45b9f.png)

사진 출처: [https://blog.naver.com/platinasnow/220035316135](https://blog.naver.com/platinasnow/220035316135)

## AOP 요소

`@Aspect` : 해당 클래스를 AOP클래스로 사용한다는 어노테이션

`@Around`

- @Around 에서 지정한 대상 객체의 메소드 실행 전, 후 공통 로직을 수행한다.
- @Around를 사용한다면 메소드의 첫번째 파라미터는 꼭 `ProceedingJointPoint` 로 두기.
- `Object object = proceedingJoingPoint.proceed()` 전 후로 공통로직을 작성해준다.
- 즉, `Object object = proceedingJoingPoint.proceed()` 이 대상 메소드로 치환되는것을 의미.

`@Pointcut` : 어느 것들을 대상으로 공통로직을 실행할지 설정해주기

- 클래스나 메소드명을 직접 입력해도 되고, 정규식을 사용해도된다.
- 꼭 `@Pointcut`을 지정하지 않아도 되고 바로 `@Around`에 적어줘도 된다.

---

Spring에서 AOP를 구현하는 방법은 두가지가 있다.

1. Annotation버전

`@TimeChecker` 어노테이션이 붙은 모든 메소드를 대상으로 수행시킨다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TimeChecker {
}
```

```java
@Around("@annotation(com.example.jpaExample.annotation.TimeChecker)")
    public Object timeChecker(ProceedingJoinPoint proceedingJoinPoint){
        Object object = null;
        try {
            long start = System.currentTimeMillis();
            //이 때 들어감
            object = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();
            log.info("타임체커 실행 = 실행시간 {}", (end-start));
        }catch(Throwable throwable){
            log.info("타임체커 오류!");
        }
         return object;
    }
```

2. PointCut 버전

```java
		@Pointcut("execution(* com.example.jpaExample.web.MemberController.*(..))") 
		//*(..)란 전체 메소드를 의미
    public void pointcut(){}

		@Around("pointcut()")
    public Object timeCheckerV2(ProceedingJoinPoint proceedingJoinPoint){
        Object object = null;
        try {
            long start = System.currentTimeMillis();
            //이 때 들어감
            object = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();
            log.info("타임체커 실행 = 실행시간 {}", (end-start));
        }catch(Throwable throwable){
            log.info("타임체커 오류!");
        }
        return object;
    }
```

AOP는 build.gradle 에 **`org.springframework.boot:spring-boot-starter-aop`**  을 ****등록해주어야 사용할 수 있다.

난 참고로 `'org.springframework.boot:spring-boot-starter-web'` 을 등록해주었다.

![image](https://user-images.githubusercontent.com/46811084/162377238-4452a20f-36a6-4ff9-9399-cbe63919826b.png)

`spirng-web` > `spring-webmvc` 하단에 `spring-aop`가 존재한다.
