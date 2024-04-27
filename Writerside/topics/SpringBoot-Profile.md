# @Profile  
  
외부 설정 파일을 조회할 때 `Environment`를 사용합니다.  
`SpringApplication.run()`을 호출하게 되면 `Environment`는 다음과 같은 필드를 갖습니다.  
  
![image_198.png](image_198.png)
  
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
