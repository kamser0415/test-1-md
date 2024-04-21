# Properties

## 외부 설정이란?  
자바에서의 외부 설정은 애플리케이션의 값을 내부 코드가 아닌 외부에서 제공하는 것을 의미합니다.  
OS 환경 변수, Java 시스템 속성, 자바 커맨드 라인 인수 또는 외부 파일을 통해 제공할 수 있습니다.  
이런 방식을 통해 애플리케이션의 설정 값을 유연하게 변경할 수 있습니다.  
  
## 외부 설정을 배우는 목적  
애플리케이션의 코드는 동일하지만 배포환경마다 다른 속성 값을 사용하기 위해서 코드를 수정하여 배포하는 방식은 
유연성도 떨어지고, 동일한 소스코드가 아닐 수 있습니다.  
  
그래서 실행 시점에 외부 설정을 주입하여 유연하게 배포를 할 수 있습니다.  
  
## 전달 방식
![image_191.png](image_191.png)  
  
외부 설정에서 전달하는 방식은 4가지 방법이 있습니다.
+ OS 환경변수: OS에서 지원하는 외부 설정, 해당 OS를 사용하는 모든 프로세스에서 참조 가능
+ 자바 시스템 속성: 자바에서 지원하는 외부설정, 해당 JVM에서 사용가능
+ 자바 커맨드라인 인수: 커맨드 라인에서 전달하는 외부 설정, 실행시 `main(args)` 메서드에서 사용
+ 외부 파일(`properties`): 프로그램에서 외부파일을 읽어서 사용
  + 약속된 위치에 파일을 읽도록 합니다 - `data/setting/...`
  + 개발서버 - 'test.properties'
  + 운영서버 - 'live.properties'  

### OS 환경 변수
자바 프로그램내에서 수정이 불가능하며, 서버마다 속성 값을 다르게 저장하여 애플리케이션에서 참조하는 방식입니다.  
모든 프로세스에서 공유되는 값이나 변하지 않는 값을 사용하는 경우에 필요한 방식입니다.  
  
### 자바 시스템 속성
실행한 JVM 안에서 접근 가능한 외부 설정입니다. 
추가로 자바가 내부에서 미리 설정해두고 사용할 수 있으며 변경도 가능합니다.  
  
지정 방식은 `java -Durl=dev -jar app.jar`  
`-D` 옵션을 통해 `key=value` 형식으로 전달되며 순서에 주의해야합니다.
+ `java -Durl=devdb -Dusername=dev_user -Dpassword=dev_pw -jar app.jar`  
  
또는 자바 코드를 통해서 저장도 가능합니다.
+ 설정 : `System.setProperty(propertyName, "propertyValue")`
+ 조회 : `System.getProperty(propertyName)`  
  
코드로 설정하는 경우에는 외부로 설정을 분리하는 효과인 유연성은 떨어집니다.

### 자바 커맨드라인 인수  
Java 실행시점에 외부 설정 값을 `main(args)` 메서드의 파라미터로 전달되는 방식이며 
다른 방식과 다르게 `Spring`으로 인식하여 개발자가 추가 수정을 해야합니다.  
  
+ `java -jar app.jar dataA dataB`  
+ 구분자는 스페이스바로 합니다.  
  
개발자가 직접 key=value 형식으로 파싱해야하는 과정이 생깁니다.    
### 자바 커맨드라인 옵션 인수  
커맨드라인인수는 개발자가 직접 `key-value`로 파싱하는 과정이 필요하지만, 스프링에서는 `key=value`로 인식할 수 있도록 
별도의 클래스를 제공합니다.  
  
+ `--key=value 형식으로 --username=kms --username=goguma`  
  
동일한 key값에 여러개의 value를 갖을 수 있습니다.  

### 외부파일
![image_192.png](image_192.png)  
  
OS 환경 변수, 자바 시스템 속성, 자바 커맨드 라인 인수 방식은 전달하는 방식은 문제가 있습니다.
1. 변경사항 확인 어려움
2. 인자가 많아질 경우 유지보수가 어려움  
  
`\build\lib`에 위치한 빌드된 결과물 옆에 외부 파일을 추가하여 참조할 수 있습니다.  
  
```Java
# application.properties

// 테스트 서버 
url=dev.db.com
username=dev_user
password=dev_pw

// 라이브 서버
url=prop.db.com
username=prop_user
password=prop_pw
```  
  
빌드 결과물을 실행하는 위치 옆에 속성 파일을 만들어 읽을 수 있습니다.   
  
+ 외부 설정을 별도의 파일로 관리하면 설정 파일 자체를 관리하기 작업이 추가가 됩니다.
+ 서버가 증가할 수 록 해당 설정파일을 유지보수하기 어렵습니다.
+ 설정 파일을 별도로 관리하기에 변경이력을 git을 통해 관리하는 방식을 사용할 수 있습니다. 
+ 다만, 해당 파일의 변경이력이 애플리케이션 속성 관련 코드의 변경이력과 같이 확인하기 복잡해집니다.
+ 관련 설정을 확인하기 위해서 해당 외부파일을 별도로 조회하는 과정이 필요합니다.  

### 내부 파일 분리
![image_194.png](image_194.png)  
  
설정 파일을 외부에서 별도로 관리하는 방식은 번거롭습니다. 
설정을 변경할 때마다 서버에 들어가서 각 변경사항을 수정해야합니다.(자동화도 가능합니다.)  
추가로 외부 설정을 확인하기 위해서 서버에 접속해야하는 불필요한 과정이 발생합니다.  
  
이때 모든 서버의 외부 설정을 애플리케이션에 포함하고, 실행 시점에 옵션을 주는 방식으로 유연하게 배포할 수 있습니다.  
  
#### 배포서버별 파일
+ application-dev.properties
+ application-prop.properties
  
두 설정 파일을 애플리케이션에서 빌드하고 실행 시점에 자바 시스템 설정으로 지정하는 방식입니다.  
  
설정 파일 규칙은 `application-{profile}.properties`으로 `profile`위치에 필요한 파일명을 만들고, 실행시점에 입력하면 됩니다.  
```Java
// java 시스템 속성 방식
java -Dspring.profiles.active=dev -jar external-0.0.1-SNAPSHOT.jar
// java 커맨드라인 인수 방식
java -jar external-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
```
#### 하나의파일로 관리
![image_195.png](image_195.png)  
  
```Java
url=default.db.com
my_username=default_user
password=default_pw

#---

spring.config.activate.on-profile=local
url=basic.db.com
my_username=basic_user
password=basic_pw

#---

spring.config.activate.on-profile=dev
url=dev.db.com
my_username=dev_user
password=dev_pw

#---

spring.config.activate.on-profile=prod
url=prod.db.com
my_username=prod_user
password=prod_pw

#---

password=must
```  
+ `#---` 으로 구분할 수 있습니다
+ `#---` 구분자 위 아래로 `#` 주석을 사용할 경우 정상적으로 동작하지 않을 수 있습니다.
  
```Java
// 자바 시스템 속성 방식
java -Dspring.profiles.active=dev -jar external-0.0.1-SNAPSHOT.jar
// 자바 커맨드 라인 속성 방식
java -jar external-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
```
### 설정 데이터 적용 순서
+ 단순하게 문서를 위에서 아래로 순서대로 읽으면서 값을 설정한다. 이때 기존 데이터가 있으면 덮어쓴다.
+ 논리 문서에 `spring.config.activate.on-profile`옵션이 있으면 해당 프로필을 사용할 때만 논리 문서를 읽습니다.  
  
### 부분적용
```Java
database=myriadb
username=local_user
passowrd=1234

#---

spring.config.activate.on-profile=dev
database=mysql
```  
이렇게 작성되어있고 `-Dspring.profiles.active=dev`으로 실행한다면 `database`의 속성값만 변경됩니다.  
  
## 속성 추상화  
![image_193.png](image_193.png)  
  
각 속성 변수마다 조회나 저장하는 방식이 다 다릅니다. 
하지만 모두 `Map` 구조로 `key,value`를 가지고 있기 때문에 각 외부 설정을 읽는 구현체를 만들고 
`Environment` 추상화 클래스를 구현하여 동일한 메서드로 동작할 수 있게 했습니다.  
  
## 우선순위-전체
[Externalized Configuartion](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)  
  
우선순위는 위에서 아래로 적용됩니다. 아래가 더 우선순위가 높습니다.  
### 자주 사용되는 우선순위  
+ 설정 데이터 파일(`application.properties > yml`) yml보다 properties 파일이 높습니다.
+ OS 환경변수
+ 자바 시스템 속성 (`-Dusername=`)
+ 커맨드라인 옵션인수 (`--username=`)
+ `@TestPropoertiesSource` 테스트 사용
  
### 설정 데이터파일 우선순위
+ jar 내부 application.properties
+ jar 내부 프로필 적용된 application-{profile}.properties
+ jar 외부 application.properties
+ jar 외부 application-{profile}.properties
  
### 우선순위 이해방법  
+ 더 유연한 것이 우선 순위를 갖습니다.(변경하기 어려운 파일보다 실행시 매번 변경이 가능한 인수가 높습니다.)
+ 범위가 넓은 것보다 좁은 것이 우선순위를 갖습니다.
  + OS 환경 변수 < 자바 시스템 < 자바 커맨드 라인 < 자바 테스트 속성
    
  
### 덮어씌우는게 아니다
예를 들어 List에 저장된 순서로 생각하면 됩니다.  
```Java
List<Environment> env = new ArrayList<Environment>();
env.add(new TestProperties());
env.add(new CommendLine());
env.add(new VMOptions());
env.add(new OSvariable());
env.add(new PropertiesFiles());
```  
이순서대로 저장하고 반복문을 돌아서 먼저나온 애들의 값을 사용한다고 생각하면 됩니다.  
  