# 외부 설정 사용 방식  
  
애플리케이션은 실행 환경에 따라 구성 정보가 다릅니다. 
테스트 서버, 라이브 테스트 서버, 라이브 서버등 배포 환경마다 다른 설정 값을 사용해야합니다. 
  
매번 다른 설정 값을 코드로 변경하여 배포하는 방식이나 내부 파일을 변경하여 배포하는 방식은 유연성이 떨어지며 애플리케이션을 재활용하기 어렵습니다.  
  
자바는 다양한 외부 설정 방식을 제공하여 실행 시점에 설정 값을 파일이나 인수로 전달했습니다.  
외부 설정 방식에 따라 프로퍼티(속성)을 읽기 위해 사용하는 클래스와 메소드가 다르기 때문에 코드를 유연하게 작성하기 어렵습니다.  
  
스프링은 외부 설정을 `Environment` 인터페이스로 추상화하여 아래와 같이 외부 설정 파일을 읽고 우선수위를 통해 프로퍼티의 값을 읽어올 수 있습니다.
   
그러면 스프링 컨테이너를 사용하여 외부 속성 값을 사용하기 위해서는 `Environment` 구현체를 사용해야하기 때문에 
어떤 프로퍼티 읽기 방식이든지 `Environment`의 `getProperty`를 사용합니다.  

```Java
#SpringApplication
ConfigurableEnvironment prepareEnvironment(...) 
// 스프링 부트 내부에서 ConfigurableEnvironment를 준비합니다.
```
  
## 외부 설정 클래스를 만들어야하는 이유  
프로퍼티 파일(`application.properties`)에 작성된 속성 값이 혼자서 사용할 수 있는 경우가 아니라면 
대부분 구성 정보 클래스를 사용하기 위해 관련된 프로퍼티 속성을 함께 사용합니다.  
예를 들면 데이터 베이스를 연결하기 위해 `url,password,username,maxConnection`등의 값이 필요합니다.  
  
자바에서 제공하는 `Property` 자료구조를 활용할 수 있지만, 클래스로 사용할 경우 관련 메소드를 추가하여 프로퍼티 클래스 내에서 사용할 수 있는
메소드나 검증 메소드를 제공할 수 있습니다.  
  
프로퍼티 클래스는 스프링 컨테이너에 등록하여 사용하면 같은 프로퍼티를 사용하는 구성 정보 클래스는 의존성 주입으로 
직접 프로퍼티 클래스를 구현하지 않아도 되므로 유지보수와 재사용성이 높아집니다. 
또한 스프링 컨테이너의 관리를 받을 수 있으므로 빈 전처리기, 후처리기등을 활용할 수 있습니다.  
  
### 프로퍼티 클래스를 빈으로 등록하지 않을 경우  
1. `Environment`에 대한 직접 의존성: 프로퍼티를 사용하기 위해 외부 설정이 필요한 구성 정보 클래스에서 직접 의존해야합니다.
   필요한 프로퍼티 이름을 확인하기 어렵기에 가독성이 떨어집니다.
2. 바인딩 코드 중복: 동일한 프로퍼티 클래스를 사용하는 여러 구성 정보 클래스에서 바인딩 코드를 직접 구현해야합니다.
   이는 중복된 코드를 만들며, 유지보수성을 낮출 수 있습니다.
3. 수정 및 확장의 어려움: 직접적으로 프로퍼티 클래스를 생성하여 의존하는 경우, 새로운 설정이 추가될때마다 관련된 구성 정보 클래스를 수정해야합니다. 유연성과 확장성을 제한할 수 있습니다.  
  
와 같은 문제점이 발생합니다.  
  
### Properteis Conversion  
스프링 부트는 `external application properties`(외부 응용프로그램 속성)이 바인딩 될때 자료형에 따라 알맞은 단위로 변환합니다.  
+ Durations - `ns,us,ms,s,m,h,d`
+ Periods - `y,m,w,d`
+ DataSizes - `B,KB,MB,GB,TB`  
  
[properties.conversion](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.conversion)  
  
## Environment
```Java
# run 실행시
// 자바 커맨드 옵션 인수 생성
ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
// 환경 객체 생성
ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
// 생략
// 스프링 컨테이너 초기화
refreshContext(context);
```    
  
스프링 컨테이너를 초기화 하기전에 스프링 부트는 `커맨드 라인 옵션 인수`와 프로퍼티를 읽어서 `Environment` 객체를 만듭니다.  
어떤 방식이던 내부 동작은 `Environment`를 활용하기에 동작원리는 동일합니다.  
  
### 코드  
프로퍼티 클래스
```Java
@Slf4j
public class MyDataSource {

    private String url;
    private String username;
    private String password;
    private int maxConnection;
    private Duration timeout;
    private List<String> options;

    public MyDataSource(String url, String username, String password, int maxConnection, Duration timeout, List<String> options) {
        this.url = url;
        this.username = username;
        this.password = password;
        this.maxConnection = maxConnection;
        this.timeout = timeout;
        this.options = options;
    }
    @PostConstruct
    public void init() {
        log.info("url = {}",url);
        log.info("username = {}",username);
        log.info("password = {}",password);
        log.info("maxConnection = {}",maxConnection);
        log.info("timeout = {}",timeout);
        log.info("options = {}",options);
    }
}
```
Environment를 직접 사용하여 프로퍼티 클래스 빈 등록
```Java
@Slf4j
@Configuration
public class MyDataSourceEnvConfig {

    private final Environment env;

    public MyDataSourceEnvConfig(Environment env) {
        this.env = env;
    }
    @Bean
    public MyDataSource myDataSource() {
        String url = env.getProperty("my.datasource.url");
        String username = env.getProperty("my.datasource.username");
        String password = env.getProperty("my.datasource.password");
        int maxConnection = env.getProperty("my.datasource.etc.max-connection",Integer.class);
        Duration timeout = env.getProperty("my.datasource.etc.timeout", Duration.class);
        List<String> options = env.getProperty("my.datasource.etc.options", List.class);
        return new MyDataSource(url, username, password, maxConnection, timeout, options);
    }
}
```  

`Environment`를 주입받아 사용하는 경우 어떤 외부 설정을 사용하더라도 동일한 조회 방법으로 프로퍼티 값을 사용할 수 있습니다.  
  
### 한계 
1. `Environment`를 직접 사용하여 프로퍼티를 조회하는 `env.getProperty()`를 반복호출 하고 있습니다.
2. 어떤 프로퍼티를 묶어서 사용하는지 한눈에 보기 어렵습니다.
3. 자료형에 맞는 타입을 직접 명시해야합니다.
4. 필드가 늘어날때마다 반복적으로 바인딩 로직을 추가해야합니다.
  
## @Value
`Environment`를 직접 사용하는 방식보다 더 나은 방식을 스프링 컨테이너로 제공합니다.   

```Java
@Slf4j
public class MyDataSourceValueConfig {

    @Value("${my.datasource.url}")
    private String url;
    @Value("${my.datasource.username}")
    private String username;
    @Value("${my.datasource.password}")
    private String password;
    @Value("${my.datasource.etc.max-connection}")
    private int maxConnection;
    @Value("${my.datasource.etc.timeout}")
    private Duration timeout;
    @Value("${my.datasource.etc.options}")
    private List<String> options;

    @Bean
    public MyDataSource myDataSource1() {
        if (maxConnection > 999) {
            throw new IllegalArgumentException();
        }
        return new MyDataSource(url, username, password, maxConnection, timeout, options);
    }

    @Bean
    public MyDataSource myDataSource2(
              @Value("${my.datasource.url}") String url
            , @Value("${my.datasource.username}") String username
            , @Value("${my.datasource.password}") String password
            , @Value("${my.datasource.etc.max-connection:2}") int maxConnection
            , @Value("${my.datasource.etc.timeout}") Duration timeout
            , @Value("${my.datasource.etc.options}") List<String> options) {
        return new MyDataSource(url, username, password, maxConnection, timeout, options);
    }
}
```
`@Value("${...}")`가 있는 스프링 빈 오브젝트는 빈 전처리기를 통해 값이 주입됩니다.  
순서는 `Constructor -> 빈 전처리기 -> @PostConstructor`  
  
```Java
@Slf4j
@Component
public class MyComponentValueConfigV2 {

    @Value("${my.datasource.url}")
    private String url;

    public MyComponentValueConfigV2() {
        log.info("===== constructor : start =====");
        log.info("url = {}", url);
    }

    @PostConstruct
    public void init() {
        log.info("===== postConstructor : start =====");
        log.info("url = {}", url);
    }
}
```  
코드를 실행하면 확인 할 수 있습니다.
```Java
===== constructor : start =====
url = null
===== postConstructor : start =====
url = local.db.com
```  
![image_197.png](image_197.png)
  
1. `PropertySourcesPlaceholderConfigurer`는 `${...}`를 바인딩 하는 빈 프로세스 입니다.
2. 스프링 부트는 `@AutoConfiguration`으로 해당 빈을 등록합니다.
3. `PropertySourcesPlaceholderConfigurer.postProcessBeanFactory`로 컨택스트에 등록된 클래스를 조회합니다.
4. 내부 동작으로 `processProperties(beanFactory, mergedProps)`를 호출하게 되어 프로퍼티를 바인딩을 합니다.
5. `resolveRequiredPlaceholders` 호출  
  
```Java
protected String getPropertyAsRawString(String key) {
  return getProperty(key, String.class, false);
}
```  
따라가다보면 해당 프로퍼티를 조회하는 것을 확인할 수 있습니다.  
  
```Java
if (value != null) {
   if (resolveNestedPlaceholders && value instanceof String string) {
       value = resolveNestedPlaceholders(string);
   }
   logKeyFound(key, propertySource, value);
   return convertValueIfNecessary(value, targetValueType);
}
```  
프로퍼티 값을 `@Value`로 바인딩할때 자료형을 확인하여 변환해주는 로직이 `convertValueIfNecessary`입니다.  
여기서 `@Value`의 한계가 나타납니다.  
  
### 장점
+ 메소드 호출, 타입 명시를 하지 않아도 됩니다.
+ `SpEL` 표현식을 지원합니다.
  
### 한계  
+ 프로퍼티 속성이 많아 지거나 수정시 변경해야되는 코드가 많아집니다.
+ 중첩된 구성 불가합니다 : 정적 멤버 클래스를 활용하기 어렵습니다.  
+ `@Validated`등 유효성 검사 애노테이션을 활용하지 못합니다.
+ 유효성 검사에 대한 로직이 생성자를 통해서 검증하다보니 코드 가독성이 나빠집니다.  
  
## @ConfigurationProperties  
**Type-safe Configuration Properties**  
스프링은 외부 설정의 묶음 정보를 객체로 변환하는 `ConfigurationPropertiesBinder`클래스를 제공합니다.  
이것을 **타입 안전한 설정 속성** 이라고 합니다.  
```Java
private final ApplicationContext applicationContext;
private final PropertySources propertySources;
private final Validator configurationPropertiesValidator;
private final boolean jsr303Present;
private volatile Validator jsr303Validator;
private volatile Binder binder;
```  
가지고 있는 필드만 보더라도 다양한 기능을 지원하는것을 확인할 수 있습니다.  
  
`Binder.bind`는 `@ConfigurationProperties(prefix="my.datasource")`의 prefix로 입력한 값 뒤에 있는 프로퍼티를 읽어
해당 클래스에 프로퍼티 값을 바인딩 합니다.  
  
```Java
@Validated
@Getter
@ConfigurationProperties("my.datasource")
public class MyDataSourcePropertiesV3 {
    // 자바빈 프로퍼티 방식으로 getter/setter 필수
    @NotEmpty
    private String url;
    @NotEmpty
    private String username;
    @NotEmpty
    private String password;
    private Etc etc;

    @ConstructorBinding // 생성자가 두개 이상일 경우에는 이걸 사용한다. 하나 일 경우에는 생략가능
    public MyDataSourcePropertiesV3(String url, String username, String password,  Etc etc) {
        this.url = url;
        this.username = username;
        this.password = password;
        this.etc = etc;
    }

    @Getter
    public static class Etc {
        @Min(1)
        @Max(999)
        private int maxConnection;
        @DurationMin(seconds = 1)
        @DurationMax(seconds = 60)
        private Duration timeout;
        private List<String> options = new ArrayList<>();

        public Etc(int maxConnection, Duration timeout, List<String> options) {
            this.maxConnection = maxConnection;
            this.timeout = timeout;
            this.options = options;
        }
    }
}
```
  
`@ConfigurationProperties`가 동작하는 방식은 다음과 같습니다.
1. **빈 후처리가 모든 빈 오브젝트를 순회**하여 `ConfigurationProperties` 애노테이션이 있는지 확인합니다.
2. `prefix`로 묶어진 프로퍼티를 `Binder.bind`를 통해 해당 프로퍼티 클래스의 필드와 동일할 경우 바인딩을 합니다.  
  
이때 프로퍼티에 속성이 추가되더라도 클래스의 필드만 추가한다면 자동으로 값이 바인딩 됩니다. 
별도의 바인딩 로직을 작성할 필요가 없습니다.  
  
`Binder`는 외부에서 가져온 프로퍼티(`Environment`)를 오브젝트의 프로퍼티(기존)에 하나씩 세팅하는 대신 자동으로 이름을 비교해서
새로운 오브젝트를 반환하는 유틸 클래스입니다.  

### 프로퍼티 클래스 빈 등록하기
+ `@Component`
+ `@EnableConfigurationProperteis(Class)`
+ `@ConfigurationPropertiesScan({"hello.datasource"})` 
  
결국은 `@ConfigurationProperties`를 사용하기 위해서는 해당 프로퍼티 클래스가 빈으로 등록되어야 
빈 프로세스가 스캔하여 바인딩을 할수 있습니다.  
  
따라서 직접 프로퍼티 클래스를 컴포넌트 스캔 대상이 되게하거나 해당 프로퍼티 클래스를 사용하는 구성 클래스에서 추가하는 방법이 있습니다.  
`@EnableConfigurationProperteis(Class)` 방식은 `@AutoConfiguration`에서 명시적으로 어떤 프로퍼티 클래스를 사용할지 지정할 수 있기 때문에 사용하지만, 
개발자가 만들어 사용하는 프로퍼티 클래스는 하나의 패키지로 관리하는 것이 유지보수하기 좋기 때문에 `@ConfigurationPropertiesScan`을 사용하는 것을 권장합니다.
  
  
### 초기화 방식
1. 자바 빈 프로퍼티 방식
   + `get/set`으로 데이터를 바인딩하는 방식 
2. 생성자 초기화 방식
   + 생성자 메서드의 인수로 바인딩 되는 방식
  
프로퍼티 클래스는 애플리케이션이 시작 된 이후에 변경되지 않아야합니다. 
`setter`를 통해서 수정이 될수 있기 때문에 중간에 데이터를 변경하는 실수가 발생하지 않게 생성자 초기화 방식을 사용합니다.  
  
`@ConstructorBinding`은 스프링 부트 3.0 이전에는 명시적으로 작성해야하지만, 3.0 이후 부터는 생성자가 하나 일경우에는 
작성하지 않아도 됩니다. `@Autowired`와 유사합니다.  
  
생성자가 두 개이상일 경우에는 `@ConstructorBinding`을 사용하여 어떤 생성자로 초기화할지 명시해야합니다.
  
### 검증 추가
`@Validated`를 통해서 검증을 추가 할 수 있습니다.  
  
### 장점  
+ ~~프로퍼티 속성이 많아 지거나 수정시 변경해야되는 코드가 많아집니다.~~
+ ~~중첩된 구성 불가합니다 : 정적 멤버 클래스를 활용하기 어렵습니다.~~
+ ~~@Validated등 유효성 검사 애노테이션을 활용하지 못합니다.~~
+ ~~유효성 검사에 대한 로직이 생성자를 통해서 검증하다보니 코드 가독성이 나빠집니다.~~  
  
`@Value`가 가진 한계를 해결 할 수 있습니다.