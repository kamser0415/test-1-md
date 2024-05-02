# @Profile  
  
외부 설정 파일을 조회할 때 `Environment`를 사용합니다.  
애플리케이션을 시작할때 `SpringApplication.run()`을 호출하게 되고 `Environment`는 다음과 같은 필드를 갖습니다.  
  
![image_198.png](image_198.png)
  
## Profile이란
스프링은 애플리케이션 구성의 일부인 빈을 분리하여 등록할 수 있는 방법을 제공합니다. 
특정 환경에서만 사용할 수 있도록 하는 것이 목적이며, `@Component`나 `@Configuration`등과 같이 빈 등록시
`@Profiles`를 통해 제한할 수 있습니다.   
  
제한하는 방법은 `spring.profiles.active`프로퍼티 값의 활성 프로필 작성합니다. 
예를 들면 `spring.profiles.active=dev,hsqldb`나 커맨드라인 인수 옵션인 `--spring.profiles.active=dev,hsqldb`로 설정합니다.  
  
## 동작 원리  
  
내부에서는 `@Conditional`을 통해 활성 프로필과 일치하는지 확인후 등록합니다.

```Java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
	String[] value();
}
```  
```Java
if (attrs != null) {
    for (Object value : attrs.get("value")) {
        if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
            return true;
        }
    }
    return false;
}
return true;
```  
  
`@Conditional`에서 `ProfileCondition.class`의 메소드를 확인해보면 
`@Profile(value)`에서 value 값과 `ActiveProfiles`를 비교하여 일치하는지 여부를 반환하는 메서드로 
해당 빈 오브젝트의 등록 여부를 결정합니다.  
   
배포 환경에 따라 실제 외부네트워크가 아니라 가짜 `mock`역할을 하는 클래스를 등록할 수 있습니다.
```Java
@Slf4j
@Configuration
public class PayConfig {

    @Bean
    @Profile("default")
    public LocalPayClient localPayClient() {
        log.info("LocalPayClient 빈 등록");
        return new LocalPayClient();
    }

    @Bean
    @Profile("prod")
    public ProdPayClient prodPayClient() {
        log.info("ProdPayClient 빈 등록");
        return new ProdPayClient();
    }
}
```  
