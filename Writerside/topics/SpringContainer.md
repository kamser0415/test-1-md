# SpringContainer

[Spring Bootstranpping](https://medium.com/chequer/tomcat-spring-bootstrapping-sequence-2편-spring-e19705529132)  
  
## 등록  
![image_183.png](image_183.png)  
  
스프링 컨테이너를 사용하기 위해서 스프링 `webmvc`의 의존성을 추가합니다.  
```Java
dependencies {
    // 서블릿
    implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'
    // 스프링 MVC
    implementation 'org.springframework:spring-webmvc:6.0.4'
}
```  
  
어노테이션을 사용하여 스프링 컨테이너에게 해당 클래스와 메서드의 역할을 명시합니다.  
```Java
@RestController
public class HelloController {
    @GetMapping("/spring-container")
    public String hello() {
        return "spring-container";
    }
}
```  
이제 스프링 컨테이너를 사용하기 위해서 구성정보 클래스를 추가합니다.   
```Java
@Configuration
public class HelloConfig {
    @Bean
    public HelloController helloController() {
        return new HelloController();
    }
}
```
```Java
AnnotationConfigWebApplicationContext springContainer = new AnnotationConfigWebApplicationContext();
//springContainer.register(HelloConfig.class);
springContainer.register(HelloController.class);
```   
구성정보를 넘기는 방법은 `xml`이나 자바 코드 그외 방식으로도 가능합니다.  
이때 구성 정보 클래스를 넘기는게 관례이기 때문에 `@Configuration`클래스를 통해 설정 정보를 전달했지만 
`@RestController`내부에도 `@Component` 어노테이션으로 빈 오브젝트를 명시하기 때문에 그래도 넘겨도 실행됩니다.  
  
## HandlesType 사용  
```Java
public class AppInitV2Spring implements AppInit {

    @Override
    public void onStartUp(ServletContext servletContext) {
        System.out.println("AppInitV1Spring.onStartUp");

        // 스프링 컨테이너를 만들고
        AnnotationConfigWebApplicationContext springContainer = new AnnotationConfigWebApplicationContext();
        springContainer.register(HelloConfig.class);

        // 디스패처 서블릿을 만들고,스프링 컨테이너를 등록
        DispatcherServlet dispatcherServlet = new DispatcherServlet(springContainer);

        // 디스패처 서블릿은 서블릿 컨테이너에 등록한다.
        // 웹 요청을 받을 path를 매핑한다.
        ServletRegistration.Dynamic dispatcherV2 = servletContext.addServlet("dispatcherV2", dispatcherServlet);
        dispatcherV2.addMapping("/spring/*");
    }
}
```  
`AppInit`인터페이스를 구현하여 `@HandlesTpye`에 의해 호출이 되어 실행됩니다.  
  
## WebApplicationInitializer 사용  
```Java
package org.springframework.web;

public interface WebApplicationInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```  
스프링 `web`에서 제공하는 인터페이스를 구현하는 방식도 가능합니다.  

```Java
# \spring-web-6.0.4.jar\META-INF\services\jakarta.servlet.ServletContainerInitializer
org.springframework.web.SpringServletContainerInitializer
```  
해당 클래스도 서블릿 컨테이너의 규칙에 맞게 작성되어잇습니다.  
  
해당 `WebApplicationInitializer` 인터페이스도 `AppInit`처럼 `@HandlesTypes`에 작성되어 있습니다.  

```Java
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public SpringServletContainerInitializer() {
    }
    public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
    }
}
```  
  
## 정리  
스프링 `web mvc`는 다음 클래스를 제공합니다.  
+ `org.springframework.web.servlet.DispatcherServlet`
+ `org.springframework.web.SpringServletContainerInitializer`  
  
스프링 `web mvc`는 `spring-context`를 의존하여 어노테이션으로 웹 설정이 가능한 컨택스트를 제공합니다.
+ `org.springframework.web.context.support.AnnotationConfigWebApplicationContext`  
  
