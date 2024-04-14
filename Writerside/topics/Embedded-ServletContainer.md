# Embedded-ServletContainer  

## 환경 구성
톰캣 라이브러리를 의존하여 코드로 톰캣을 실행하는 방식입니다.  
```Gradle
dependencies {
    //스프링 MVC 추가
    implementation 'org.springframework:spring-webmvc:6.0.4'

    //내장 톰켓 추가 라이브러리에 접근해서 실행할 수 있다.
    implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'
}
```  

## 실행코드  
공통적인 코드 내용은 다음과 같습니다.
1. 톰캣 인스턴스 생성
2. 커넥트 생성 및 설정
3. 서블릿 컨테이너 생성
4. 서블릿 등록
5. 서블릿 매핑
6. 톰캣 실행

### 서블릿 방식
```Java
public class EmbedTomcatServletMain {
    public static void main(String[] args) throws LifecycleException {
        System.out.println("EmbedTomcatServletMain.main");
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);
        Context context = tomcat.addContext("", "/");
        tomcat.addServlet("", "helloServlet", new HelloServlet());
        context.addServletMappingDecoded("/hello-servlet", "helloServlet");
        tomcat.start();
    }
}
```  

### Connector
HTTP 프로토콜과 관련된 설정을 가진 클래스입니다. 
클라이언트로부터의 요청을 받고 이를 처리할 서비스(service) 객체에 위임합니다.  
  
주로 특정 포트에 요청을 수신하고, 웹 요청 객체와 웹 응답 객체를 생성하는 메서드를 갖고 있습니다.  
  
**Connector는 WAS마다 다른 구현체를 가집니다.**  
웹 서버마다 가진 목적에 맞게 활용할 수 있도록 요구사항에 맞춰진 클래스를 개별로 사용합니다.  

### Service  
서비스는 여러 개의 `Connector`를 그룹화하여 하나의 애플리케이션 단위로 구성합니다. 
각 서비스는 독립된 애플리케이션 단위로 클라이언트의 요청을 받고 처리할 수 있는 컨테이너를 포함합니다.  
이때 서비스는 하나의 컨테이너를 갖습니다.

### Container  
서블릿의 라이프사이클을 관리하는 컴포넌트입니다. 서블릿의 인스턴스를 생성하고 초기화하며, 
요청을 받아드려서 서블릿에 전달하여 해당 서블릿이 요청을 처리할 수 있도록 합니다.
  
### DispatcherServlet 방식
```Java
public class EmbedTomcatSpringMain {
    public static void main(String[] args) throws LifecycleException {
        System.out.println("EmbedTomcatSpringMain.main");
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

        // 스프링 컨테이너 생성
        AnnotationConfigWebApplicationContext springContainer = new AnnotationConfigWebApplicationContext();
        springContainer.register(HelloConfig.class);
//        springContainer.refresh();

        // 스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
        DispatcherServlet dispatcher = new DispatcherServlet(springContainer);
        
        // 디스패처 서블릿을 등록하기
        Context context = tomcat.addContext("", "/");
        tomcat.addServlet("", "dispatcher", dispatcher);
        
        // 경로 매핑
        // 디스패처 서블릿을 호출하게 되고 스프링 컨테이너 안에 있는 Controller 클래스 안의 매핑된 애들을 호출
        context.addServletMappingDecoded("/","dispatcher");
        tomcat.start();
    }
}
```  

### refresh()  
스프링 컨테이너는 `refresh()`를 실행해야합니다.  
이유는 웹 요청이 들어왔을대 해당 서블릿을 생성하기 때문입니다.  
  
컨트롤러 코드
```Java
@RestController
public class HelloController {

    public HelloController() {
        System.out.println("HelloController.HelloController");
    }

    @PostConstruct
    public void run() throws InterruptedException {
//        Thread.sleep(3000);
        System.out.println("HelloController.run");
    }

    @GetMapping("/hello-spring")
    public String hello() {
        System.out.println("Thread.currentThread().getName() = " + Thread.currentThread().getName());
        System.out.println("HelloController.hello");
        return "hello spring!";
    }

}
```  

```Java
4월 14, 2024 3:08:14 오후 org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["http-nio-8080"]
// 웹 요청 후
HelloController.HelloController
HelloController.run
```  
처음 요청이 들어올 경우 그때 컨테이너를 초기화 합니다.  
컨테이너 초기화 작업이 오래 걸리는 네트워크 연결이나 지연이 생기는 경우에는 컨테이너 초기화 전까지 
동작하지 않기 때문에 웹 서버가 시작하기 전에 스프링 컨테이너는 초기화가 되어야합니다.  
