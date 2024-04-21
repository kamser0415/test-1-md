# AutoConfiguration
  
이 글은 김영한님의 강의를 요약한 것입니다. [출처](https://inf.run/7VBBx)   


## 정의  
스프링 부트의 자동 구성은
사용자가 추가한 의존성 라이브러리나 프레임워크에 필요한 빈 오브젝트를 설정하는 번거로움을 덜어줍니다. 
사용자는 직접 속성값 설정과 빈 등록을 할 필요없이, 스프링 부트가 제안하는 기본적인 속성 값과 권장하는 빈 오브젝트를 자동으로 등록하는 것을 말합니다.  
  
## 공부하는 이유
스프링 부트가 권장하는 설정 값을 자동으로 제공해줍니다. 
이 설정이 어떻게 이루어지는지 이해함으로써 사용자는 공통적인 실행 환경에 맞춰진 빈 오브젝트를 조정하거나 
필요에 맞게 사용자 정의를 해야할 때가 오기 때문입니다. 
사용자가 자동 등록된 빈을 수정하거나 변경하지 않으려면 그 동작 방식을 이해하는 것중요합니다.  
  
예를 들어 `JpaTransactionManager`를 커스텀 해서 등록했는데, 기존에 동작하던 코드가 예외가 발생하면서 동작하지 않는 경우가 발생할 수 있습니다.  
그럴때 어떤 원인인지 파악하기 위해서는 자동 구성을 이해할 필요가 있습니다.  
  
## 이해하기  

> 전제  
> 로컬 라이브러리를 추가하는 방식보다 원격 저장소인 (`maven`)에 저장하여 
> gradle의 dependency를 통해 주입받는 것이 안전하고 유지보수하기도 간단합니다.
  

### 간단한 라이브러리 만들기  

해당 라이브러리를 실행하는 JVM의 메모리 사용량을 조회할 수 있는 라이브러리 입니다.
해당 라이브러리의 `properties`를 설정하고, 관련된 오브젝트를 빈 오브젝트를 등록합니다. 
그리고 `/memory`로 `GET` 요청을 하면 아래와 같음 메세지를 확인할 수 있는 라이브러리 입니다.  
  
```Java
// DTO
@Getter
@ToString
public class Memory {
    private final long used ;
    private final long max;

    public Memory(final long used,final long max) {
        this.used = used;
        this.max = max;
    }
}
// properties
@Getter
public class MemoryProperties {

    private final String unit;
    private final String name;

    public MemoryProperties(String unit, String name) {
        this.unit = unit;
        this.name = name;
    }

    public int calcUnit(){
        return switch (unit) {
            case "GB" -> 1024 * 1024 * 1024; // 기가바이트로 변환하기 위해 바이트를 1024의 3승으로 나눔
            case "MB" -> 1024 * 1024; // 메가바이트로 변환하기 위해 바이트를 1024의 2승으로 나눔
            case "BYTE" -> 1; // 그 외의 경우에는 변환할 필요 없으므로 1 반환
            default -> throw new IllegalArgumentException("단위가 올바르지 않습니다.");
        };
    }
}
// MemoryFinder
@Slf4j
@Getter
public class MemoryFinder {

    private final MemoryProperties memoryProperties;

    public MemoryFinder(MemoryProperties memoryProperties) {
        this.memoryProperties = memoryProperties;
    }

    public Memory get() {
        int unit = memoryProperties.calcUnit();
        long max = Runtime.getRuntime().maxMemory()/unit;
        long total = Runtime.getRuntime().totalMemory()/unit;
        long freed = Runtime.getRuntime().freeMemory()/unit;
        long used = total-freed;

        return new Memory(used,max);
    }
    @PostConstruct
    public void init(){
        // 빈 등록 성공 확인 로직
        System.out.println("MemoryFinder.init");
    }

}
// MemoryController
@Slf4j
@RestController
@RequiredArgsConstructor
public class MemoryController {

    private final MemoryFinder memoryFinder;

    @GetMapping("/memory")
    public Memory system() {
        Memory memory = memoryFinder.get();
        log.info("memory ={}", memory);
        return memory;
    }
}
```
### 자동 구성이 없는 경우  
#### 코드
```Java
@Configuration
public class MemoryConfig {

    @Bean
    public MemoryController memoryController() {
        return new MemoryController(memoryFinder());
    }

    @Bean
    public MemoryFinder memoryFinder() {
        return new MemoryFinder(new MemoryProperties("MB","Local"));
    }
}
```  
해당 라이브러리를 사용하기 위해 사용자는 구성 정보를 만들기 위해 
1. 해당 라이브러리의 설정 단위를 알아야합니다.
2. 등록해야하는 빈 오브젝트의 의존 관계를 알아야합니다.
3. 구성 정보 클래스를 별도로 만들어 사용해야합니다.  
  
해당 라이브러리를 사용하기 위해 불필요한 학습이 필요하게 됩니다.
### 자동 구성을 설정하는 경우  
라이브러리를 만든 개발자나 스프링 부트에서는 보편적으로 권장하는 설정과 빈 오브젝트를 등록하기 위한 `@Configuration` 클래스를 생성합니다.  
```Java
@AutoConfiguration
public class MemoryAutoConfiguration {

    @Bean
    public MemoryController memoryController() {
        return new MemoryController(memoryFinder());
    }

    @Bean
    public MemoryFinder memoryFinder() {
        return new MemoryFinder(new MemoryProperties("MB","Local"));
    }
}
```  
그리고 해당 구성 정보 클래스를 스프링 부트에서 인식할 수 있도록 별도의 파일에 작성합니다.  

![image_190.png](image_190.png)  
  
```Java
memory.MemoryAutoConfiguration
```  
   
이렇게 할 경우 해당 라이브러리를 사용하는 사용자가 라이브러리를 학습하고, 의존 관계를 설정하여 `@Configuration` 클래스를 등록하지 않아도 
`/memory` 를 입력하면 해당 라이브러리가 동작합니다.
### 스프링 부트가 자동 구성을 읽는 방법
스프링 부트가 관리하는 라이브러리의 자동 구성 클래스도 있고, 라이브러리가 작성한 자동 구성 클래스도 있습니다.  
해당 `jar`파일내에 `.class` 모두 스캔하여 `@AutoConfiguration`을 가진 클래스를 찾는 방법보다 특정 위치에 자동 구성 클래스로 사용하는 
클래스의 풀네임을 작성하는것이 효율적이고 관리하기 용이합니다.  
  
약속된 위치인 `/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`에 자동 구성 정보 클래스를 작성하면 
해당 클래스를 빈 오브젝트 후보에 등록하는 방식을 사용하고 있습니다.  
  
스프링 부트 애노테이션 `@SpringBoottApplication` 내에 메타에노테이션인 `@EnableAutoConfiguration`에서 위에 작성된 파일을 읽고
해당 파일을 스프링 부트 컨테이너 후보에 등록합니다.  

```Java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader())
            .getCandidates();
    Assert.notEmpty(configurations,
            "No auto configuration classes found in "
                    + "META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```  
내부 동작에서 `ImportCandidates.load()`에서 해당 파일을 조회합니다.  
```Java
public final class ImportCandidates implements Iterable<String> {

	private static final String LOCATION = "META-INF/spring/%s.imports";

	private final List<String> candidates;
	
    public static ImportCandidates load(Class<?> annotation, ClassLoader classLoader) {
		Assert.notNull(annotation, "'annotation' must not be null");
		ClassLoader classLoaderToUse = decideClassloader(classLoader);
		String location = String.format(LOCATION, annotation.getName());
		// location = META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
		Enumeration<URL> urls = findUrlsInClasspath(classLoaderToUse, location);
		List<String> importCandidates = new ArrayList<>();
		while (urls.hasMoreElements()) {
			URL url = urls.nextElement();
			importCandidates.addAll(readCandidateConfigurations(url));
		}
		return new ImportCandidates(importCandidates);
	}
}
```  
해당 파일에 있는 ClassPath를 모두 읽어서 내부 필드에 보관을 합니다.
```Java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```
그리고 자동 구성 클래스 엔트리로 만들고 이걸 `StringArray`로 반환하여 빈 오브젝트로 등록합니다.   
  
#### 정리
`@SpringBootApplication`=> `@EnableAutoConfiguration`->`@Import(AutoConfiguartionImportSelector.calss)`-> 특정 위치 파일을 읽고 
스프링 컨테이너에 등록합니다.

### 자동 구성에 조건이 필요한 이유
`/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`에 작성된 구성 정보 클래스를 모드 빈으로 등록할 수 없습니다.  

1. 사용자의 프로퍼티 설정에 따라 빈 등록 여부를 결정할때
2. 이미 등록된 빈이 있는 경우 충돌 방지를 위해 무시해야하는 경우
3. 해당 클래스를 로드할 수 있어야 등록되는 빈인 경우  
  
등등 조건에 만족해야 해당 빈을 사용할 수 있는 경우가 있기 때문에 빈 등록할 때 조건을 추가해야합니다.  
  
[공식 홈페이지-Conditional 조건](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations)  
  
스프링 부트가 제공하는 `@ConditionalOnXXX`외에도 `interface Condition`을 구현한 클래스를 `@Conditional()` 인수로 전달하면 
해당 조건에 일치하는 클래스만 빈으로 등록될 수 있습니다.  

## 실제 AutoConfiguration 이해하기  
  
```Java
@AutoConfiguration(before = SqlInitializationAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import(DataSourcePoolMetadataProvidersConfiguration.class)
public class DataSourceAutoConfiguration {}
```  
자동 구성 정보 클래스에 등록된 클래스입니다.  
  
해당 `DataSourceAutoConfiguration`가 구성 정보 클래스로 동작하기 위해 조건이 추가되어있습니다.
1. SqlInitializationAutoConfiguration 구성 정보보다 전에 동작합니다.
2. `DataSource.class`, `EmbeddedDatabaseType.class`두 클래스가 존재해야합니다.
3. `io.r2dbc.spi.ConnectionFactory` 해당 타입의 빈이 존재하지 않아야합니다. 
  
   
```Java
@AutoConfiguration(after = DataSourceAutoConfiguration.class)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ DatabaseInitializationDependencyConfigurer.class, JdbcTemplateConfiguration.class,
		NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {}
```  
1. `DataSourceAutoConfiguration` 이후에 동작합니다
2. `DataSource.class, JdbcTemplate.class` 두 클래스가 존재해야합니다.
3. `DataSource.class` `DataSource`타입의 빈의 후보가 하나일때 동작합니다. 여러개의 `DataSource`가 존재하고 우선순위가 정해져있지 않다면 동작하지 않습니다.  
  
  