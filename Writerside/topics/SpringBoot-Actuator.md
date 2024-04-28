# Actuator(엑츄에이터)  
  
  

[스프링 부트 액추에이터 웹 API 문서](https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/)  
  
## 프로덕션 준비 기능  
개발자는 애플리케이션을 개발할 때 기능 요구사항만 개발하는 것이 아닙니다.  
서비스를 실제 운영하는 단계에서 올리게 되면 개발자들이 해야하는 업무는 서비스에 문제를 모니터링하고 
지표들을 심어서 감시하는 활동입니다.  
  
운영 환경에서 서비스할 때 필요한 이런 기능들을 **프로덕션 준비 기능** 이라고 합니다.  
프로덕션을 운영에 배포할 때 준비해야하는 비 기능적 요소들을 말합니다.  
+ 지표(`metric`)
+ 추적(`trace`)
+ 감사(`auditing`)
+ 모니터링  
  
애플리케이션이 동작하는지, 로그 정보는 정상으로 설정되었는지, 커넥션풀은 얼마나 사용되는지 확인할 수 있어야합니다.  
  
스프링 부트는 액추에이터를 통해 프로덕션 준비 기능을 의존성만 추가해도 다양한 편의 기능들을 제공합니다. 

### 의존성 추가
```Actionscript
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // Actuator 추가
    implementation 'org.springframework.boot:spring-boot-starter-actuator'

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    //test lombok 사용
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
}
```  
  
## 사용 방법  
![image_199.png](image_199.png)  
  
스프링 부트에 의존성을 추가하면 자동으로 관련된 빈을 스프링 컨테이너에 등록합니다.
스프링 특성상 엑츄에이터 빈을 커스텀하고 등록할 수 있고, 속성 값만 변경할 수 도 있습니다,  
  
엑추에이터를 사용하기 위해서 
1. 엑추에이터 엔드포인트(`특정 기능을 하는 빈`) 활성화 (`on,off`)
2. 엑추에이터 엔드포인트 노출 허용
  
### 엔드포인트 설정 방법
```yaml
management:
  endpoint:
    shutdown:
      enabled: true # 엔드포인트 활성화
  endpoints:
    web: # jmx: 도 가능합니다.
     exposure:
     include: "*" # 엔드포인트 모두 노출
   # exclude: "shutdown,env" env,shutdown 엔드포인트는 노출하지 않는다.
```
    
해당 방법은 모든 활성화된 엔드포인트를 `web`을 통해서 볼 수 있도록 허용했다는 의미입니다.  
활성화 위치인 (`web`)은 `HTTP`이며, 다른 JMX를 통해서 노출을 선택하거나 모두 노출할 수 있습니다.  


`http://localhost:8080/actuator` 실행하면 엔드포인트가 활성화 + 노출된 엑추에이터 목록을 확인할 수 있습니다.  
  
```yaml
"_links": {
  "self": {
    "href": "http://localhost:8080/actuator",
    "templated": false
  },
  "beans": {
    "href": "http://localhost:8080/actuator/beans",
    "templated": false
  },
      :
}
```  
엑추에이터가 제공하는 기능을 확인 할 수 있습니다.   
  
## 다양한 엔드포인트  
각각의 엔드포인트를 통해서 개발자는 애플리케이션 내부의 수 많은 기능을 관리하고 모니터링할 수 있습니다.  
  
[엔드포인트 메뉴얼](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)
  
#### 엔드포인트 목록
+ beans : 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
+ conditions : condition 을 통해서 빈을 등록할 때 평가 조건과 일치하거나 일치하지 않는 이유를 표시한다.
+ configprops : @ConfigurationProperties 를 보여준다.
+ env : Environment 정보를 보여준다.
+ **health** : 애플리케이션 헬스 정보를 보여준다.
+ httpexchanges : HTTP 호출 응답 정보를 보여준다. HttpExchangeRepository 를 구현한 빈을 별도로 등록해야 한다.
+ info : 애플리케이션 정보를 보여준다.
+ **loggers** : 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
+ **metrics** : 애플리케이션의 메트릭 정보를 보여준다.
+ **mappings** : @RequestMapping 정보를 보여준다.
+ **threaddump** : 쓰레드 덤프를 실행해서 보여준다.
+ shutdown : 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어 있다  
  
## Health(헬스정보)  
헬스 정보를 사용하여 애플리케이션의 문제가 발생할 경우 빠르게 파악할 수 있습니다.  

`http://localhost:8080/actuator/health`
  
#### 기본동작  
추가 설정을 하지 않고 `health`정보를 확인하면 애플리케이션의 전체 동작 상황을 확인할 수 있습니다. 
애플리케이션, DB네트워크, 기타 연결이나 상태가 하나라도 **"UP"** 이 아닐 경우 `status`는 변경됩니다.  
```Java
# 기본 동작
{"status": "UP"}
```  

#### 상세보기 설정
```Java
# .application
management.endpoint.health.show-details=always
# .yml
management:
 endpoint:
   health:
    show-details: always
#   show-components: always
```

**show-details** : always 일 경우  
```Java
"status": "UP",
"components": {
    "db": {
        "status": "UP",
        "details": {
            "database": "H2",
            "validationQuery": "isValid()"
        }
    },
    "diskSpace": {
        "status": "UP",
        "details": {
            "total": 511152484352,
            "free": 341093076992,
            "threshold": 10485760,
            "path": "C:\\study\\inflean\\spring\\spring-boot-active\\boot-source-study\\start\\actuator-04-28\\.",
            "exists": true
        }
    },
    "ping": {
        "status": "UP"
    }
}
```  
각각의 항목이 자세하게 노출되는 것을 확인할 수 있습니다. 

**show-components** : always 일 경우  
```Java
"status": "DOWN",
"components": {
    "db": {
        "status": "UP"
    },
    "diskSpace": {
        "status": "UP"
    },
    "myCustom": {
        "status": "DOWN"
    },
    "ping": {
        "status": "UP"
    }
}
```
각 컴포넌트마다 최종 상태인 `status`를 가지고 있습니다. 여기서 `DOWN`으로 `status`를 가지게 되니 
최종 `status`는 `DOWN`이 되었습니다.  
  
### 직접 구현하기  
```Java
@Component
public class MyCustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        boolean status = getStatus();
        if (status) {
            return Health.up()
                    .withDetail("slave-1", "value1") // 사내에서 상용 Indicator 를 직접 만들어서 통신하기 위한 목적
                    .withDetail("key2", "value2")
                    .build();
        }
        return Health.down()
                .withDetail("key3","key3")
                .withDetail("key4","key4")
                .build();
    }
    boolean getStatus() {
        // 상용 프로그램의 체크하고 싶은 네트워크의 상태를 들고옵니다.
        return System.currentTimeMillis() % 2 == 0;
    }
}
```
`HealthIndicator`를 상속하여 `Health health()` 메소드를 오버라이딩하여 
특정 네트워크나 상황을 체크한 뒤 `Health` 객체를 만들어서 반환하면 됩니다.  
빌더의 첫번째 값에 따라 결과 상태와 `HttpStatusCode`가 달라집니다.  

+ `Health.up()` : status = "UP" , HttpStatusCode = "200"
+ `Health.down()` : status = "DOWN" , HttpStatusCode = "503"  
  
#### 참고
+ [헬스 기본 지원 기능](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.auto-configured-health-indicators)
+ [헬스 기능 직접 구현하기](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.writing-custom-health-indicators)
  
  
## 애플리케이션 정보  
`/info` 엔드 포인트는 애플리케이션의 기본 정보를 노출합니다.  

| ID    | 이름                         | 설명                       | 전제 조건                                     |
|-------|----------------------------|--------------------------|-------------------------------------------|
| build | BuildInfoContributor       | 빌드 정보 노출                 | META-INF/build-info.properties 리소스가 있어야 함 |
| env   | EnvironmentInfoContributor | 이름이 info로 시작하는 환경 속성을 노출 | 없음                                        |
| git   | GitInfoContributor         | Git 정보 노출                | git.properties 리소스가 있어야 함                 |
| java  | JavaInfoContributor        | 자바 런타임 정보 노출             | 없음                                        |
| os    | OsInfoContributor          | 운영 체제 정보 노출              | 없음                                        |

+ `env,java,os`을 사용하려면 `management.info.<id>.enable = true`으로 설정해야합니다.
+ `build,git`은 `enable=true`이지만, 전제 조건이 있어야합니다.
  
### 전제조건 설정 방법
`build`와 `git`은 별도의 전제조건을 만족하기 위해서는 아래와 같이 설정하면 됩니다.

#### build 추가
```Java
// build.gradle
springBoot {
    buildInfo()
}
```  

#### git repository
해당 프로젝트는 `git repository`가 있어야합니다.
```Java
// 터미널
$ git init
```   
추가로 `git` 정보를 노출하기 위해 `git-properties` 플러그인을 추가합니다.
```Actionscript
plugins {
 ...
 id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
}
```  
자세한 정보가 필요하다면 프로퍼티 설정을 추가합니다.  
```Actionscript
management:
  info:
    git:
      mode: full
```
[git generate info](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.build.generate-git-info)  

  
### 커스텀 info 만들기  
`env`가 활성화되어 있으면 `info.*`의 프로퍼티는 `Environment`의 `info` 프로퍼티로 들어갑니다.
```Java
// 하드코딩
info.app.encoding=UTF-8
info.app.java.source=17
info.app.java.target=17
```  
다른 프로퍼티를 참조하는 방법
```Actionscript
// OS 환경 변수 조회
info.app.java.source=${java.version}

// 현재 프로퍼티내의 on-profile 참조
spring:
  config:
    activate:
      on-profile: default
      
info:
  app:
    name: hello-actuator
    company: yh
    app-name-copy: "${info.app.name}"
    on-profile: "${spring.config.activate.on-profile}"
    not-search-property: "${not.search.is.same.print}"
    not-search-property-default: "${not.search.is.same.print:empty}"
```  
`/info`로 조회를 하면 다음과 같이 `placeholder`가 변경됩니다. 
다만 해당 프로퍼티를 찾을 수 없는 경우 문자 그대로를 출력합니다. `SpEL`을 사용하기 때문에 기본값도 설정이 가능합니다.
```Actionscript
"app": {
    "name": "hello-actuator",
    "company": "yh",
    "app-name-copy": "hello-actuator",
    "on-profile": "default",
    "not-search-property": "${not.search.is.same.print}"
    "not-search-property-default": "empty"
}
```  
  
## Logger(로거)
`/loggers` 엔드포인트를 사용하면 전체 혹은 특정 패키지의 로깅과 관련된 정보를 확인하고, 
실시간으로 변경할 수 있습니다.  
  
> 단 메모리에서 변경하기 때문에 애플리케이션이 재시작이 된다면 프로퍼티 설정을 따라갑니다.  
>   
   
프로퍼티 파일에 특정 패키지의 로깅 레벨을 설정하고 실행합니다.  
```yaml
# application.yml 로깅 레벨 설정
logging:
  level:
    hello.controller: debug
```  
  
### 엔드포인트 호출
`http://localhost:8080/actuator/loggers`  
  
```JSON
{
    "levels": [
        "OFF",
        "ERROR",
        "WARN",
        "INFO",
        "DEBUG",
        "TRACE"
    ],
    "loggers": {
        "ROOT": {
            "configuredLevel": "INFO",
            "effectiveLevel": "INFO"
        },
        "hello.controller": {
            "configuredLevel": "DEBUG",
            "effectiveLevel": "DEBUG"
        },
        "hello.controller.LoggerController": {
            "effectiveLevel": "DEBUG"
        }
    }
}
```  
로깅의 레벨 목록과 패키지와 클래스가 로깅레벨을 확인할 수 있습니다.  
여기에 특정 패키지의 로깅 레벨도 확인이 가능합니다.  
  
#### 상세 조회
`http://localhost:8080/actuator/loggers/{로거 이름}` 다음과 같은 패턴으로 
특정 로거 이름을 기준으로 조회할 수 있습니다.
  
`http://localhost:8080/actuator/loggers/hello.controller`
```JSON
{
    "configuredLevel": "DEBUG",
    "effectiveLevel": "DEBUG"
}
```  
  
### 실시간 로그 레벨 변경  
해당 기능이 필요한 이유는 개발 서버는 `DEBUG`을 설정하고, 운영 서버는 요청이 많기 때문에 
성능이나 디스크에 영향을 적게하기 위해 `INFO`이상의 레벨을 설정합니다.  
  
서비스 운영중에 문제가 발생하여 급하게 `DEBUG`나 `TRACE`로그를 남겨서 확인해야한다면 
`property`를 수정후 재시작을 하지 않아도 로그 레벨을 변경할 수 있습니다.  
  
**_POST METHOD_** 로 요청해야합니다.  
`http://localhost:8080/actuator/loggers/hello.controller` 로 요청하고 body는 json으로 다음과 같이 요청을 실행하면
```JSON
{
 "configuredLevel": "TRACE"
}
```  
성공적으로 수정이 된다면 `Response StatusCode`는 **_204(`Not content`)_** 으로 응답 코드가 돌아옵니다.  
  
#### 변경 확인
해당 url로 `GET`으로 요청하면 다음과 같이 변경된 내용을 확인할 수 있습니다.
```JSON
{
    "configuredLevel": "TRACE",
    "effectiveLevel": "TRACE"
}
```  
  
## HTTP 요청 응답 기록  
`HTTP` 요청과 응답의 과거 기록을 확인하고 싶을 경우에 사용하는 엔드포인트 입니다. 
**전제 조건** 은 `HttpExchangeRepository` 인터페이스의 구현체를 빈으로 등록하면 됩니다.  
  
**_httpexchanges_** 엔드포인트를 사용할 수 있습니다.  
  
스프링 부트는 기본으로 `InMemoryHttpExchangeRepository` 구현체를 제공합니다.  
```Java
@Configuration
public class InMemoryHttpExchangeRepositoryConfig {
    @Bean
    public InMemoryHttpExchangeRepository httpExchangeRepository() {
        return new InMemoryHttpExchangeRepository();
    }
}
```  
  
### 한계  
기능이 단순하고, 제한이 많기 때문에 개발 단계에서 사용합니다.  
실제 운영 서비스에서는 모니터링 툴인 네이버의 핀포인트나 Zipkin 같은 다른 기술을 사용하는 것을 권장합니다.  
  
  
## 엑츄에이터와 보안  
**HTTP** 를 통해서 엑츄에이터 기능이 활성화된 애플리케이션의 내부 정보를 많이 노출할 수 있습니다. 
그래서 외부 인터넷 망이 공개된 곳에서 사용하는 것은 보안상 좋지 않습니다.  
  
그러다보니 두가지 해결 방안이 있습니다.  
1. **엑츄에이터를 다른 포트에서 실행**  
   + 외부 네트워크는 `8080` 포트로만 접근할 수 있도록 설정
   + 내부 네트워크는 다른 포트로 설정하여 사용합니다.
     + `management.server.port=9092`으로 설정하면 액츄에이터 접근 포트가 변경됩니다.
2. **액츄에이터 url 경로에 인증 설정**
   + `/actuator`에 접근할 때 `필터,인터셉터,시큐리티`등을 사용하여 인증한뒤 접근할 수 있도록 설정합니다.  
     
### 엔드포인트 경로 변경
```yaml
management:
  endpoints:
    web:
      base-path: "/manage/api"
```  
이렇게 설정할 경우 `/actuator/{엔드포인트}` 대신에 `/manage/api/{엔드포인트}`로 접근할 수 있습니다.