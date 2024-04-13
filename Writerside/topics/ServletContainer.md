# ServletContainer

참고 자료: [서블릿 컨테이너의 이해](https://yangbongsoo.gitbook.io/study/servlet_container)  
  
## 개념
서블릿 컨테이너는 클라이언트와 서버간 소켓 통신에 필요한 TCP/IP 연결 관리와 HTTP 프로토콜 해석등 네트워크 기반 작업을 
추상화하여 웹 기능을 수행할 수 있는 실행 환경을 제공합니다.  
  
개발자는 직접 TCP/IP 연결과 HTTP 프로토콜 해석 및 작성을 하지 않고 WAS가 제공하는 API를 통해 네트워크에 접근할 수 있습니다.

## 서블릿이란  
`javax.servlet.Servlet` 인터페이스를 구현한 클래스를 서블릿으로 간주합니다.  
서블릿은 클라이언트로 부터 HTTP 요청을 받고, 요청에 따라 기능을 수행한 후 HTTP 응답을 반환하는 
웹 컴포넌트 입니다.  
  
서블릿은 `main()`메서드가 없이 서블릿 컨테이너에 의해 생명주기가 관리됩니다.  
이를 통해 웹 애플리케이션의 동적인 기능을 구현하고 웹 요청을 처리할 수 있습니다.  

## 초기화 방식  
서블릿 컨테이너를 실행하는 시점에 필요한 초기화 작업이 있습니다.  
필요한 서블릿이나 필터를 등록해야하는데 이때 스프링을 사용한다면 디스패처 서블릿과 스프링 컨테이너를 등록해야합니다.  
+ 과거에는 web.xml을 사용했지만 자바코드로 초기화도 지원합니다.
  
![image_181.png](image_181.png)  
  
+ 출처가 사라졌습니다.  
  
[톰캣과 자바 클래스로더의 차이점](https://yangbongsoo.gitbook.io/study/understanding_the_tomcat_classpath)  
  
```Java
public interface ServletContainerInitializer {
    public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
}
```  
서블릿 컨테이너는 실행전에 `ServletContainerInitializer.class`를 구현한 클래스나 상속한 클래스를 찾아서 등록합니다.  
여기서 애플리케이션에 필요한 기능을 초기화 하거나 등록할 수 있습니다.  
+ `HandlesTypes`에 지정된 클래스 유형으로 확장되거나 구현된 클래스 집합을 호출합니다.  
  
`resources/META-INF/services/jakarta.servlet.ServletContainerInitializer`  
해당 위치에 `WAS`가 읽어서 스캔할 클래스를 확인하기 때문에 작성해야합니다.  
  
```Java
# jakarta.servlet.ServletContainerInitializer 파일 내
hello.container.MyContainerInitV1
hello.container.MyContainerInitV2
```  
  
![image_182.png](image_182.png)  
  
### 어노태이션 방식  
```Java
@WebServlet(urlPatterns = "/test")
public class TestServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("TestServlet.service");
        resp.getWriter().println("test");
    }
}
```  
HttpServlet을 상속하여 해당 클래스를 서블릿으로 등록합니다.

### 프로그래밍 방식  
내용과 형식은 상관없고 인터페이스가 필요합니다.
```Java
public interface AppInit {
   void onStartup(ServletContext servletContext);
}
```  
해당 클래스의 구현체를 작성합니다.  
```Java  
public class AppInitV1Servlet implements AppInit {

    @Override
    public void onStartUp(ServletContext servletContext) {
        System.out.println("AppInitV1Servlet.onStartUp");
        ServletRegistration.Dynamic helloServlet =
                servletContext.addServlet("helloServlet", new HelloServlet());
        helloServlet.addMapping("/hello-servlet");
    }
}
```  
WAS가 호출할 수 있도록 ServletContainerInitializer 구현한 클래스를 작성합니다.  
+ `@HandlesTypes`에 작성한 인터페이스를 추가합니다.   
```Java
@HandlesTypes(AppInit.class)
public class MyContainerInitV2 implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        // set 인수에 AppInit을 구현한 Class 정보가 들어온다.
        for (Class<?> appInitClass : set) {
            try {
                //new
                AppInit appInit = (AppInit) appInitClass.getDeclaredConstructor().newInstance();
                appInit.onStartUp(servletContext);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```  
그러면 `AppInit.class`를 상속한 클래스 메타데이터(Class) 집합을 호출합니다.  

  
### 장단점  
+ 애노테이션 방식 : 구현이 간단하지만 조건에 따라 서블릿 등록을 결정할 수 없습니다.  
+ 프로그래밍 방식
  + `path` 경로를 상황에 따라 바꾸어 외부 설정을 읽어 등록할 수 있습니다.
  + 서블릿 자체도 특정 조건에 따라 if문으로 분기해서 등록하거나 뺄수 있습니다.
  + 서블릿을 직접 생산하기 때문에 필요한 생성자 인자를 결정할 수 있습니다.  
  
### 정리  
서블릿 컨테이너 초기화중 클래스로더를 통해 `ServletContainerInitializer`의 파일을 스캔합니다.  
+ `META-INF/lib`와 `META-INF/classes`안에서 조회를 합니다.  
  
리플렉션을 통해서 인스턴스를 생성하고 반복문을 돌면서 `onStartup`을 실행함  
  
애노테이션 방식은 `Tomcat.class`에서 클래스를 확인한뒤 `addServlet()`으로 등록합니다.
```Java
private static boolean hasAsync(Servlet existing) {
   boolean result = false;
   Class<?> clazz = existing.getClass();
   WebServlet ws = clazz.getAnnotation(WebServlet.class);
   if (ws != null) {
       result = ws.asyncSupported();
   }
   return result;
}
```

## 초기화 시점이 다른 이유  
![image_180.png](image_180.png)    

**Gradle**  
`implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'`  
  
의존성을 추가한 것을 확인해보면 우리 코드가 사용할 서블릿 컨테이너에 대한 정보가 없습니다. 
이 말은 어떤 서블릿 컨테이너를 사용해도 웹 기능이 동작할 수 있는 서블릿을 등록할 수 있다는 의미입니다.  
  
1. **단일 책임 원칙**  
   + 서블릿 컨테이너를 초기화 하는 역할인 톰캣과 서블릿을 초기화하는 모듈을 분리합니다.
2. **의존성 분리**  
   + 서블릿을 초기화 하는 과정에서 서블릿 컨테이너 등록하는 방법에 대한 의존성을 제거 할 수 있습니다.
   + 서블릿 컨테이너가 변경되어도 서블릿 초기화 하는 코드는 수정하지 않아도 됩니다.
   + 프로그래밍 방식이나 자바 코드 방식등 상황에 맞게 사용할 수 있습니다.  
  
### 정리  
서블릿 컨테이너가 변경되어도 소스 코드를 수정하지 않아도 됩니다.  
  