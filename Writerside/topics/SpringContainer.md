# SpringContainer  

## 목적
서블릿 컨테이너에 등록할 디스패처 서블릿과 디스패처 서블릿이 필요한 컴포넌트를 호출해주는 스프링 컨테이너를 통합하는 과정을 확인해야합니다.

[Spring Bootstranpping](https://medium.com/chequer/tomcat-spring-bootstrapping-sequence-2편-spring-e19705529132)  
  
## 구성 정보 등록 
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
  
![image_184.png](image_184.png)  

스프링 컨테이너는 구성 메타데이터와 비즈니스 로직(`POJO`)를 스프링 컨테이너에 전달해야 사용할 수 있습니다.  
  
구성 메타데이터를 전달하는 방법은 `BeanDefinition` 인터페이스를 구현한 메타 데이터를 전달하면 됩니다.  
자바 코드 방식, XML, GROOVY 등으로 메타데이터를 스프링 컨테이너에게 전달하면 됩니다.  

지금은 어노테이션을 사용하여 스프링 컨테이너에게 메타데이터를 전달합니다.  
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
이때 구성 정보 클래스를 넘기는게 관례이기 때문에 `@Configuration`클래스를 통해 설정 정보를 전달했지만 
`@RestController`내부에도 `@Component` 어노테이션으로 빈 오브젝트를 명시하기 때문에 그래도 넘겨도 실행됩니다.  
  
## HandlesType 사용   
서블릿 컨테이저가 제공하는 `@HandlersTypes`를 활용하여 서블릿 컨테이너에게 초기화를 호출 받도록 합니다. 
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
스프링 컨테이너에서 제공하은 인터페이스를 활용하는 방식도 가능합니다.  
스프링 컨테이너도 서블릿 컨테이너게 제시하는 API 사양에 맞게 구현해야합니다.  

```Java
package org.springframework.web;

public interface WebApplicationInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```   
해당 `WebApplicationInitializer` 인터페이스도 `AppInit`처럼 `@HandlesTypes`에 작성되어 있습니다.  

```Java
# \spring-web-6.0.4.jar\META-INF\services\jakarta.servlet.ServletContainerInitializer
org.springframework.web.SpringServletContainerInitializer
```  
`\META-INF\services\jakarta.servlet.ServletContainerInitializer`의 구조를 지켜야 
서블릿 컨테이너가 해당 파일에 작성된 클래스 정보를 확인하여 호출합니다.

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
  
