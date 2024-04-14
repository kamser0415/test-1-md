# SpringApplication.run  
  
## 편리한 부트 클래스  
스프링 부트가 제공하는 클래스와 static 메소르를 사용하여 스프링 컨테이너 초기화 및 톰캣을 실행할 수 있습니다.  
  
### 기존코드   
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
### 템플릿 메서드 사용  
현재 기존코드는 `구성 정보 클래스.class`와 S`tring[] args`를 제외하고는 코드 내용은 동일합니다.  
  
해당 내용을 메소드로 추출하여 별도의 클래스에서 관리하는 방식으로 코드를 수정합니다.
```Java
public class MySpringApplication {
    public static <T> void run(Class<T> configClass, String[] args) {
        System.out.println("MySpringApplication.run args=" + List.of(args));

        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

        // 스프링 컨테이너 생성
        AnnotationConfigWebApplicationContext springContainer = new AnnotationConfigWebApplicationContext();
        springContainer.register(configClass);
        springContainer.refresh();

        // 스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
        DispatcherServlet dispatcher = new DispatcherServlet(springContainer);

        // 디스패처 서블릿을 등록하기
        Context context = tomcat.addContext("", "/");
        tomcat.addServlet("", "dispatcher", dispatcher);

        // 경로 매핑
        // 디스패처 서블릿을 호출하게 되고 스프링 컨테이너 안에 있는 Controller 클래스 안의 매핑된 애들을 호출
        context.addServletMappingDecoded("/", "dispatcher");
        // check => uncheck
        try {
            tomcat.start();
        } catch (LifecycleException e) {
            throw new RuntimeException(e);
        }
    }
}
```  
그리고 구성정보 클래스를 전달할 때 `@ComponentScan`을 사용하여 패키지내 `@Component`를 빈으로 등록하도록 수정할 수 있습니다.  
  
구성정보 클래스와 그외 다양한 메타애노테이션을 추가할 수 있습니다.
```Java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ComponentScan
public @interface MySpringBootApplication {}
```    
+ `main()` 메서드를 호출할때 초기화 탬플릿이 있던 메소드(`run`)를 호출합니다.
+ 구성 정보 클래스로 `@ComponentScan` 클래스를 가진 메타 클래스를 전달합니다.
+ 내부에서 초기화가 시작됩니다.  
  
```Java
@MySpringBootApplication
public class MySpringBootMain {
    public static void main(String[] args) {
        System.out.println("MySpringBootMain.main");
        MySpringApplication.run(MySpringBootMain.class, args);
    }
}
```  
  
### 핵심  
+ 스프링 컨테이너를 초기화 한다.
+ WAS(내장 톰캣)을 생성한다.  
  
## 초기화 과정 - 스프링 컨테이너
`SpringApplication.run()`실행시 아래 코드가 실행됩니다.  
```Java
# run 실행시
public ConfigurableApplicationContext run(String... args) {
    ConfigurableApplicationContext context = null;
    context = createApplicationContext();
}
```  
`createApplicationContext()`가 스프링 컨테이너를 생성합니다.  
  
```Java
protected ConfigurableApplicationContext createApplicationContext() {
    return this.applicationContextFactory.create(this.webApplicationType);
}
```
`createApplicationContext()`로 생성할때 `applicationContextFactory`의 메서드를 호출하는데 
이때 `this.webApplicationType`을 보고 맞는 Context를 생성합니다.  
  
**this.webApplicationType**  
```Java
private static final String WEBMVC_INDICATOR_CLASS = "org.springframework.web.servlet.DispatcherServlet";
private static final String WEBFLUX_INDICATOR_CLASS = "org.springframework.web.reactive.DispatcherHandler";
private static final String JERSEY_INDICATOR_CLASS = "org.glassfish.jersey.servlet.ServletContainer";

static WebApplicationType deduceFromClasspath() {
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
            && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    for (String className : SERVLET_INDICATOR_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    return WebApplicationType.SERVLET;
}
```
해당 메서드는 클래스로더로 `WEBFLUX` 인지 `MVC`인지 확인하고 알맞은 타입을 지정합니다.  
  
`this.applicationContextFactory.create(this.webApplicationType)`의 create를 실행하면  
```Java
private <T> T getFromSpringFactories(WebApplicationType webApplicationType,
        BiFunction<ApplicationContextFactory, WebApplicationType, T> action, Supplier<T> defaultResult) {
    for (ApplicationContextFactory candidate : SpringFactoriesLoader.loadFactories(ApplicationContextFactory.class,
            getClass().getClassLoader())) {
        T result = action.apply(candidate, webApplicationType);
        if (result != null) {
            return result;
        }
    }
    return (defaultResult != null) ? defaultResult.get() : null;
}
```  
내부에서 `getFromSpringFactories`을 호출합니다.  
`WebApplicationType`은 현재 `WEBMVC`으로 `ApplicationContextFactory` 후보중에서 `ServletWebServerApplicationContextFactory`가 호출됩니다.  
   
내부에서 `createContext()`를 호출하게 되며 우리가 `web mvc`를 사용하는 스프링컨테이너를 생성하는 것을 확인할 수 있습니다. 
```Java
private ConfigurableApplicationContext createContext() {
    if (!AotDetector.useGeneratedArtifacts()) {
        return new AnnotationConfigServletWebServerApplicationContext();
    }
    return new ServletWebServerApplicationContext();
}
```    

## 서블릿 컨테이너 실행
```Java
run 메소드 일부
// 생략
context = createApplicationContext();
// 생략
refreshContext(context);
// 생략
```
`refreshContext(context);` 을 호출하면 아래 메서드가 실행됩니다.
```Java
// SpringApplication 
private void refreshContext(ConfigurableApplicationContext context) {
    if (this.registerShutdownHook) {
        shutdownHook.registerApplicationContext(context);
    }
    refresh(context);
}
```
`ServletWebServerApplicationContextFactory`의 refresh가 호출되고 상위 클래스에게 위임합니다.
```Java
// ServletWebServerApplicationContextFactory
@Override
public final void refresh(){
    super.refresh();
} 
```   
`AbstractApplicationContext`가 호출되면서 
```Java
// AbstractApplicationContext
public void refresh() throws BeansException, IllegalStateException {
    // 생략
    this.onRefresh();
    // 생략
}
```  
다시 탬플릿 메서드로 자식 클래스의 `onRefresh()`가 호출됩니다.  
```Java
@Override
protected void onRefresh() {
    super.onRefresh();
    try {
        createWebServer();
    }
    catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start web server", ex);
    }
}
```
```Java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
    if (this.disableMBeanRegistry) {
        Registry.disableRegistry();
    }
    Tomcat tomcat = new Tomcat();
```  
코드를 따라가다보면 `new Tomcat()`이 생성되는 것을 확인할 수 있습니다.  
  
## 정리  
스프링 부트는 디자인 패턴중 템플릿 메소드 패턴을 사용하여 컨택스트와 웹 서버 등을 초기화하는 시점에 
알맞은 인스턴스를 생성해줍니다.  
  
