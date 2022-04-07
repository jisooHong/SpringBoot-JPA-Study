Interceptor
-----------------
인터셉터(Interceptor)는 J2EE 표준 스펙인 필터(Filter)와 달리 Spring이 제공하는 기술로써, 디스패처 서블릿(Dispatcher Servlet)이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다.

![image](https://user-images.githubusercontent.com/46811084/162201120-96e59503-bbec-41b4-9203-f17336f72333.png)

- controller가 실행되기 전에 가로채서 행위한다.
- 필터와 달리 스프링 컨텍스트가 관리한다.
- 따라서 spring context 안의 Controller(=handler)에 관한 요청과 응답에 대해 처리한다.
- 주로 로그인체크, 권한체크, 프로그램 실행시간 계산작업 로그확인 등에 쓰인다.

![image](https://user-images.githubusercontent.com/46811084/162201186-343a66e0-a09b-4702-a384-8da5ec7c1f6b.png)

`HandlerInterceptor` 인터페이스를 구현해서 사용한다.

- `Prehandler()` : Interception point before the execution of a handler. Called after HandlerMapping determined an appropriate handler object, but before HandlerAdapter invokes the handler. = handler mapping 직후, handler adapter 이전
- `PostHandler()` : Interception point after successful execution of a handler. Called after HandlerAdapter actually invoked the handler, but before the DispatcherServlet renders the view. = 즉 컨트롤러 실행 직후지만 veiw가 렌더링 되기 전
- `afterCompletion()` : Callback after completion of request processing, that is, after rendering the view. = view 렌더링 후

---

`Interceptor`로 컨트롤러 실행 전 로그인 확인 여부 확인하기

```java
public class LoginSessionInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // controller 실행 전에 실행
        log.info("prehandle 시작");

        HttpSession session = request.getSession(false);
        if(session == null || session.getAttribute(SESSION_ID) == null){
            //로그인이 안되어 있음
            response.sendRedirect("/member/error");
            return false;
        }
        return true;
    }
}
```

`WebMvcConfigurer`를 implements한 `WebConfig` 에 `interceptor` 등록하기

번외로 여기서 `argumentResolver`도 등록해준다!

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    //환경 설정 등록해주는 곳
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //로그인이 필요했던 시점에만 적용해주기
        registry.addInterceptor(new LoginSessionInterceptor()).order(1).addPathPatterns("/member/find","/member/my","/member/logout");
    }
}
```
