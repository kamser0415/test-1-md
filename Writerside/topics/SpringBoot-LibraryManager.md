# 스프링 부트 라이브러리 관리  
  
스프링 부트는 개발자가 라이브러리를 고민하지 않고 사용할 수 있도록 제공합니다.  
+ 외부 라이브러리 버전 관리
+ 스프링 부트 스터터 제공 

## 버전관리
```Gradle
plugins {
    id 'org.springframework.boot' version '3.0.2'
    // 버전 관리 플러그인
    id 'io.spring.dependency-management' version '1.1.0' //추가
    id 'java'
}
```    

`dependency-management`을 사용 전
```Gradle
implementation 'org.springframe work:spring-webmvc:6.0.4'
```  
`dependency-management`을 사용 후  
```Gradle
implementation 'org.springframework:spring-webmvc'
```
  
### dependency-management 관리  
`io.spring.dependency-management` 플러그인을 사용하면 
현재 스프링 부트 버전에 맞는 라이브러리 버전을 관리합니다.  
  
`spring-boot-dependencies`에 있는 다음 `bom` 정보를 참고합니다.  
`gradle`플러그인을 사용하기에 개발자는 확인할 수 없습니다.  
  
[버전 정보 bom](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-dependencies/build.gradle)  
  
#### BOM(Bill of materials)  
자재 명세서란 제품 구성하는 모든 부품들에 대한 목록을 말합니다.  
부품이 복잡한 요소들로 구성된 조립품의 경우에는 계층적인 구조로 작성될 수 있습니다.  
  
#### 관리 목록
[Managed Dependency Coordinates](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.coordinates)  

+ 버전을 선언하지 않고 이러한 아티팩트 중 하나에 종속성을 선언하면 표에 나열된 버전이 사용됩니다.  
  
#### 관리하지 않는 경우  
이럴 겨웅에는 라이브러리의 버전을 직접 입력합니다.  
```Gradle
implementation 'org.yaml:snakeyaml:1.30'
```  
  
### 관리 버전 변경  
스프링 부트에서 관리하는 버전을 재정의하는 데 사용할 수 있는 모든 속성을 제공합니다.  
  
![image_186.png](image_186.png)  
  
```Gradle
    repositories {
    mavenCentral()
}

ext['tomcat.version'] = '10.1.4'

dependencies {
    //..
}
```  
[Version Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.properties)  
  
## 스타터  
웹 프로젝트 기능을 실행하려면 많은 라이브러리를 사용해야합니다. 
이때 개발자가 고민하지 않아도 스프링 부트가 제시하는 기본적인 라이브러리를 제공합니다.  
  
```Gradle
dependencies {
 //3. 스프링 부트 스타터
 implementation 'org.springframework.boot:spring-boot-starter-web'
}
```  
해당 스타터만 의존해도 아래와 같은 라이브러리가 추가됩니다.
```Gradle
// Logging 관련 라이브러리
// - 로깅 구현체 및 관련 모듈
Gradle: ch.qos.logback:logback-core:1.4.5
Gradle: org.slf4j:slf4j-api:2.0.6
        :

// JSON 처리 관련 라이브러리
// - JSON 데이터의 직렬화 및 역직렬화 지원
// - JSON 데이터 형식 및 애노테이션
Gradle: com.fasterxml.jackson.core:jackson-core:2.14.1
Gradle: com.fasterxml.jackson.core:jackson-databind:2.14.1
        :

// 웹 애플리케이션 개발 관련 라이브러리
// - 메트릭 수집, 웹 서버 임베딩, 웹 애플리케이션 개발 관련 모듈
Gradle: org.apache.tomcat.embed:tomcat-embed-core:10.1.4
Gradle: org.springframework.boot:spring-boot-starter-web:3.0.2
        :

// Spring Framework 관련 라이브러리
// - 스프링 프레임워크 핵심 모듈 및 웹 개발 지원 모듈
Gradle: org.springframework:spring-aop:6.0.4
Gradle: org.springframework:spring-beans:6.0.4
        :

// 기타 유틸리티 관련 라이브러리
// - 자바 어노테이션 API, YAML 파서
Gradle: jakarta.annotation:jakarta.annotation-api:2.1.1
Gradle: org.yaml:snakeyaml:1.33
```  
  
### 스프링 부트 스타터 목록
[Spring Starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)  
  
### 이름 패턴  
+ **공식** : `spring-boot-starter-*`
+ **비공식** : `mybatis-spring-boot-starter`
  + 타사 스타터는 일반적으로 프로젝트 이름으로 시작합니다