며칠 전 네이버 면접을 보면서 커넥션풀에 대해 질문을 받았다. 어디선가 들어본 것 같지만(?) 대답을 못해서.. 이번 기회에 알아보도록 하겠다..

# Connection Pool

말 그대로 connection을 모아둔 수영장을 의미한다.

지금까지 우리는 DriverManager의 getConnection()을 받아왔다. 이 과정은 꽤 복잡하다.

![image](https://user-images.githubusercontent.com/46811084/164159587-bb52fa00-d912-4a43-858a-8dfaafb47bd0.png)

TCP/IP 를 거칠 때 3 way handShaking 과정을 통해 DB에 접근한다. 이 과정은 네트워크 동작이기 때문에 시간이 소모된다. 사용자는 요청을 할 때마다 이처럼 시간을 낭비해야한다. 

![image](https://user-images.githubusercontent.com/46811084/164161916-d600f3fe-f29e-45d7-86be-35dde2d6433e.png)

→ 따라서 미리 애플리케이션이 시작 될 때 커넥션을 N개 만들어 놓는다. dafault값은 10개이다.

![image](https://user-images.githubusercontent.com/46811084/164161837-8742f5e9-0df9-4e37-b3eb-0d4723013681.png)

- 사용자는 커넥션풀에서 커넥션을 꺼내 쓰고 반납하면 된다. ( = 연결을 끊는게 아니라 반납! ), 즉 재사용이 가능하다.
- 커넥션 풀 안의 커넥션은 모두 DB와 TCP/IP로 연결이 되어있기 때문에 언제든지 SQL문을 즉시 DB에 전달할 수 있다.

> 커넥션풀을 사용하는 데에는 여러 이점이 있다.
> 
1. 시간 절약 : 커넥션 하는 시간을 줄일 수 있다.
2. DB 보호 : 커넥션 풀안의 커넥션 개수를 지정할 수 있기 때문에 DB의 무한 사용을 막아줄 수 있다.

커넥션풀은 오픈소스로 구현되어있다.

대표적으로 `HikariCp`, `commons-dbcp2` , `tomcat-jdbc pool` 이 있다. 

스프링부트2.0은 그 중 `HikariCp`를 기본으로 제공해준다.
