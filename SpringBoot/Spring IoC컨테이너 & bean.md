## Spring IoC(Inversion of Control)
: 객체의 라이프사이클 관리을 개발자가 아닌 스프링프레임워크 런타임엔진인 ```스프링 컨테이너'```가 하는 것을 의미한다.

## 스프링 컨테이너
: 스프링 런타임 엔진으로 객체의 생성부터 소멸까지의 라이프사이클을 알아서 관리해줍니다. ( 이 때 알아서는, 요청이 들어올 때를 의미합니다)
## Bean
:  ```spring container```안에서 관리되는 객체들을 의미합니다.

bean을 등록하는 방법은 크게 두가지가 있습니다.
1) Component scan을 통한 등록   
  ```@component```어노테이션을 통해 bean 추가   
   이 때  ```@Controller```, ```@Service```. ```@Repository``` 인터페이스 위에 ```@Component```가 명시되어 있습니다.    
   따라서 이 세가지 어노테이션을 붙이는 클래스도 모두 빈으로 등록될 수 있습니다. 


2) 직접 자바로 등록하기   
  클래스위에```@Configuration```, 메소드 위에 ```@bean``` 어노테이션을 명시하여 빈 등록 가능
  
  ## DI(Dependency injection) 의존성 주입
  : 의존관계에 있는 객체를 스프링 컨테이너로부터 받아와 사용하는 것을 의미합니다. 
  
  ### DI 방법
  1) 생성자로 주입
  2) setter로 주입 -> null 문제 발생 가능
  3) @Autoried 필드 주입 -> null 문제 발생 가능
 
 -> 이 세가지 중 ```생성자```로 주입을 가장 스프링에서 권장함   
 
 ```생성자```를 통한 의존성 주입의 장점
 1) 의존성 주입을 누락하는 것을 방지해줍니다. (null 이슈 X)
 2) final 키워드만을 사용하기 때문에 불변한 속성을 가집니다.
 3) 생성자를 호출하는 시점에 딱 한번만 호출하는 것이 보장됩니다.
