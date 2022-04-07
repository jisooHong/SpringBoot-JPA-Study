# 어노테이션과 ArgumentResolver을 이용한 반복코드 생략

`httpSession.getAttribute(”user”)` 

→ 로그인 기능이 있는 웹 개발에서는 IndexController 뿐만 아니라 다른 컨트롤러 메소드에서 세션값이 필요하면 그 때마다 직접 세션에서 값을 가져와야한다. 같은 코드가 계속해서 반복되는 것은 불필요하다.

따라서 컨트롤러의 인자로 세션값을 바로 받아올 수 있도록 변경해보자.

1. `@LoginUser` 어노테이션 생성

```java
@Target(ElementType.PARAMETER) //파라미터로 선언된 객체에서만 사용할 수 있다.
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

`@Target` 과 `@Retention`과 같이 어노테이션 위에 작성하는 어노테이션을 메타 어노테이션이라고 한다. 

`@Target` : Returns an array of the kinds of elements an annotation type can be applied to. 즉 , 해당 어노테이션에 어디에 사용할 수 있는지 명시

- `ElementType` 종류
    
    ```java
    public enum ElementType {
        /** Class, interface (including annotation type), or enum declaration */
        TYPE,
    
        /** Field declaration (includes enum constants) */
        FIELD,
    
        /** Method declaration */
        METHOD,
    
        /** Formal parameter declaration */
        PARAMETER,
    
        /** Constructor declaration */
        CONSTRUCTOR,
    
        /** Local variable declaration */
        LOCAL_VARIABLE,
    
        /** Annotation type declaration */
        ANNOTATION_TYPE,
    
        /** Package declaration */
        PACKAGE,
    
        /**
         * Type parameter declaration
         *
         * @since 1.8
         */
        TYPE_PARAMETER,
    
        /**
         * Use of a type
         *
         * @since 1.8
         */
        TYPE_USE,
    
        /**
         * Module declaration.
         *
         * @since 9
         */
        MODULE
    }
    ```
    

`@Retention` : 해당 어노테이션이 선언된 대상의 메모리를 언제까지 유지할것인지에 결정하는 어노테이션, 3가지 속성이 있다.

- RetentionPolicy.SOURCE : Annotations are to be discarded by the compiler. = 컴파일할때 사라짐, 주석처럼 사용가능
- RetentionPolicy.CLASS: Annotations are to be recorded in the class file by the compiler but need not be retained by the VM at run time. This is the default behavior. = 즉 컴파일할때는 (.java를 바이트 코드로 바꿀 때) Class 파일에 기록되지만 런타임할때(클래스로더가 바이트 코드를 로드할때) 가상머신에 안담김,
- RetentionPolicy.RUNTIME : Annotations are to be recorded in the class file by the compiler and retained by the VM at run time, so they may be read reflectively. = 런타임시까지 유지됨

2 . LoginUserArgumentResolver 클래스 생성

→ HandlerMethodArgumentsResolver interface를 구현한 클래스

Strategy interface for resolving method parameters into argument values in the context of a given request. = 요청이 들어올때 메소드 파라미터를 인자값으로 받아오면서 사용하는 전략인터페이스

```java
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    private final HttpSession httpSession;

    //컨트롤러 메서드의 특정 파라미터를 지원하는지 판단
    // 파라미터에 @LoginUser 어노테이션이 붙어 있고, 파라미터 클래스 타입이 SessionUser.class 일 경우 true 반환
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    // 파라미터에 전달할 객체 생성
    // 세션에서 객체를 가져옴
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```

 

`HandlerMethodArgumentResolver`에는 두가지 메소드가 있다.

- *boolean* supportsParameter(MethodParameter parameter);
    
    =  조건에 맞는 메소드인지 확인하는 기능
    
- Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory)
    
    = 파라미터에 전달할 객체 생성
    

3. index 수정

session값을 항상 받아왔다면 파라미터를 통해서 미리 SessionUser을 세팅해놓음

```java
@GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts",postsService.findAllDesc());

        // SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if (user != null){
            model.addAttribute("userName",user.getName());
        }
        return "index";
    }
```
---
> resolver의 동작과정

1. **`Client Request` 요청**
2. **`Dispatcher Servlet`에서 해당 요청 처리**
3. **`Client Request`에 대한 `Handler Mapping`**
    
    3.1 **`RequestMapping`에 대한 매칭** (**`RequestMappingHandlerAdapter`가 수행**)
    
    3.2 **`Interceptor` 처리**
    
    3.3 **`Argument Resolver` 처리** <-- **`Argument Resolver` 실행 지점**
    
    3.4 **`Message Converter` 처리**
    
4. **Controller Method invoke**

---

번외로, “Argument resolver를 사용할 때 , `@LoginUser` 어노테이션을 왜 썼는가? 안써도 되지않나?” 에 대한 질문을 받았었는데 곰곰이 생각해보니 당장 해당 플젝에서는 어노테이션을 사용안해도될듯하다.

그렇게 생각한 이유는 

- SessionUser 객체를 사용하는 controller은 현재로써는 IndexConroller의 index 메소드밖에 없기 때문!  `@LoginUser` 어노테이션이 없어도 ArgumentResolver는 동작한다.

하지만 다른 컨트롤러에서도 사용하도록 확장시킬 수 있기 때문에 사용을 권장한다.
