# warm-up-section-05-01-스프링부트이모저모

## build.gradle
빌드 스크립트라고불리며, gradle을 이용해 프로젝트를 빌드하고 의존성을 관리하기 위해 작성되었습니다.  
+ jvm 언어로 모두 사용 가능합니다  
  
### plugins 블락
```Gradle
plugins {
	id 'org.springframework.boot' version '2.7.6'
	id 'io.spring.dependency-management' version '1.0.12.RELEASE'
	id 'java'
}
```  
**역할**  
블럭안에 플러그인을 추가할 수 있도록 합니다.  
  
````Gradle
id 'org.springframework.boot' version '2.7.6'
````  
1. 스프링을 빌드했을 때 실행 가능한 `jar`파일이 나오게 도와줍니다. 
2. 스프링 애플리케이션을 실행할 수 있게 도와주고
3. 또 다른 플러그 인들이 잘 적용될 수 있게 해준다.  
  
```Gradle  
id 'io.spring.dependency-management' version '1.0.12.RELEASE'
```  
외부 라이브러리,프레임 워크의 버전관리에 도움을 주고 서로 얽혀있는 의존성을 처리하는데 도와준다.  
```Gradle
id 'java'
```  
+ Java 프로젝트를 개발하는데 필요한 기능들을 추가해줍니다.  
+ jvm gradle 플러그인을 사용할 수 있는 기반을 마련해준다.  
  
```Gradle
id 'name' version 'version' 형식으로 추가할 수 있다.
```  
  
### 프로젝트와 관련된 설정
```Gradle
group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'
```  
+ group: 프로젝트를 만드는 그룹에 대한 설정과 빌드의 결과물에 프로젝트 정보가 담기게 된다.
+ version: 0.0.1 snapshot을 변경하면 jar 파일도 반영된다.
+ sourceCompatibility: JDK 버전을 의미한다.  
  
### repositories 블락
```Gradle
repositories {
	mavenCentral()
}
```  
  
### dependencies 블락
```Gradle
implementation 'org.springframework.boot:spring-boot-starter-web'
runtimeOnly 'com.h2database:h2'
```  
+ implementation: 이 뒤에 오는 라이브러리나 프레임워크는 항상 사용한다는 의미
+ runtimeOnly: 코드를 실행할 경우에만 사용한다는 의미
+ testimplementation: 테스트 코드를 컴파일하거나 실행할 때 사용한다.  
  
의존성 설정할때 언제 사용할지 명시해주는 의미입니다.  
  
## yml
```yaml
key=value 
```  
value로 들어갈 수 있는 값
1. true/false의 boolean 타입
2. 숫자 (그냥 숫자)  
3. 문자열(큰따옴표가 없어도 문자로 인식)  
4. 배열도 들어갈 수 있다.
    ```yaml
     person:
       name: lomom
       age: 322
       dogs:
        - 초코
        - 망고
     ```
   이렇게 `-`로 배열을 나타낼 수 있습니다.  
  
**주석(#) 을 사용**  
  
## properties  
파일마다 구획을 구분해서 사용한다.
```Actionscript
application-dev.properties
```  
 
## Lombok
보일러 플레이트 코드를줄일 수 있습니다.  
  
## 마이그레이션 툴이 있으면 빨갛게 알려줌
