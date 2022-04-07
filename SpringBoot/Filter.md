# Filter

# Filter란?

필터(Filter)는 J2EE 표준 스펙 기능으로 디스패처 서블릿(Dispatcher Servlet)에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공한다.

<img src="https://user-images.githubusercontent.com/46811084/162199430-76ceb1bd-ef31-484a-97f2-e45e40865f65.png" width="700" height="250"/>

- spring bean으로 등록은 되지만 spring container에 관리가 되는게 아닌 web container에 관리가 된다.
- Filter 는 스프링의 독자적인 기능이 아닌 자바 서블릿에서 제공하는 기능이다.
- 주로 요청에 대한 인증, 권한 체크를 할때 사용된다.
- 스프링컨텍스르로 관리가 되지 않기 때문에 스프링과 무관한 자원에 대해서 동작한다.
- Filter은 FilterChain 을 통해 여러 필터가 연쇄적으로 동작할 수 있다.

<img src="https://user-images.githubusercontent.com/46811084/162199871-0b54dd15-17f1-4cab-8b43-18fc67dc5a93.png" width="700" height="250"/>

---

`servlet` 의 Filter 인터페이스

```java
public interface Filter{
	public default void init(FilterConfig filterConfig) throws ServletException{}

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
	public default destroy(){}
}
```

- init()
    
    : 필터 객체를 초기화하고 서비스에 추가하기 위해 사용
    
- doFilter()
    
    : request, response가 필터를 거칠 때 수행되는 메소드
    
    
- destroy()
    
    : 필터가 소멸될때 수행된다.
    
    ---
    
    ### doFilter로 로그인검증여부를 확인해보자
    
    - session이 없다면 ```httpResponse.sendRedirect()```로 로그인 페이지로 보내기
    - redirect 시 꼭 return을 해줘야한다. return 이 없을 시 다음 필터, 디스패처 서블릿으로 넘어감
    
    ```java
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        String requestURI = httpRequest.getRequestURI();
        HttpSession session = httpRequest.getSession(false);
        try {
            if (!PatternMatchUtils.simpleMatch(unAuthList, requestURI)) {  
                if (session == null || session.getAttribute("SESSION-KEY") == null) {
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
    								// 리다이렉트로 로그인 후 원래 보려고 했던 페이지로 이동되어있어야하니까 requestURI를 함께 redirect 해줌
                    return;
                }
            }
    
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("LoginValidFilter 종료");
        }
    }
    ```
    
    해당 필터 빈으로 등록해주기
    
    ```java
    @Configuration
    public class Config{
    
        @Bean
        public FilterRegistrationBean firstFilterRegister() {
            FilterRegistrationBean registrationBean = new FilterRegistrationBean(new FirstFilter());
            registrationBean.setOrder(1); //필터의 순서를 명시할 수 있다.
            return registrationBean;
        }
    }
    ```
    
