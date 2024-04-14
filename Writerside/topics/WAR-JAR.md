# Executable WAR 와 JAR  
  
[Spring-Boot-Build-Format](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html)  
 
## Executable WAR 
WAR(Web Application Archive)라는 이름으로 WAR 파일은 웹 어플리케이션 **서버(WAS)에 배포할 때 사용하는 파일**입니다.
  
### 구조  
```Java
example.war
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-WEB-INF
    +-classes
    |  +-com
    |     +-mycompany
    |        +-project
    |           +-YourClasses.class
    +-lib
    |  +-dependency1.jar
    |  +-dependency2.jar
    +-lib-provided
       +-servlet-api.jar
       +-dependency3.jar
```  
+ 사전 정의된 구조를 사용함 (WEB-INF, META-INF)
+ 웹 관련 자원을 포함함 (JSP, Servlet, JAR, Class, XML, HTML, Javascript)   
+ WEB-INF: 폴더 하위는 자바 클래스와 라이브러리, 그리고 설정 정보가 들어있습니다.  
+ 별도의 웹서버(WEB) or 웹 컨테이너(WAS) 필요
+ JAR파일의 일종으로 웹 애플리케이션 전체를 패키징하여 `WAS`에 배포하기 위한 압축파일입니다.  

### WEB-INF  
개발자가 직접 작성한 class파일과 jar 파일, JSP일 경우 view 파일들까지 포함되어있는 디렉토리입니다.  
`Tomcat`의 기본 구조와 `was File Structure` 구조와 유사합니다.  
`war`포멧은 웹서버의 구조 규칙을 지키고 있습니다. 외장 WAS나 JSP를 사용해야한다면 WAR를 이용해야합니다.   
  
### META-INF
`war` 파일은 JVM에 단독으로 실행할 수 없지만 Spring Boot에서 bootWar로 빌드할 경우 단독으로 실행가능합니다.  
```Java
Start-Class: com.example.YourClasses
Main-Class: org.springframework.boot.loader.WarLauncher
```  
`WarLauncher`가 `WAR`를 실행할 수 있도록 외부 의존성을 가져옵니다.

![image_185.png](image_185.png)
  
### 배포
1. Tomcat 실행 `./startup.bat`
2. `tomcat폴더/webapps`에 빌드 파일 복사
3. 빌드파일 이름을 `ROOT.war`으로 변경
4. `tomcat폴더/webapps/ROOT.war`을 톰캣이 압축해제하여 사용
  
### 단점
+ 서버에 별도 `WAS`를 설치해야한다.
+ xml 파일로 웹 서버 설정을 해야한다.
+ 톰캣 버전 변경하려면 설정과 자바 버전등 확인해야는 과정이 생긴다.  

## Executable JAR 소개
자바는 `Member.java` 파일을 `javac`로 컴파일하여 `Member.class`파일을 생성할 수 있습니다.  
여러개의 `.class`파일과 리소스를 묶어서 관리할 수 있도록 `java archive`인 **_Jar_** 압축파일을 제공합니다.

```Java
jar --create --file stock_rt_1-0.0.1-SNAPSHOT.jar -C build/classes/java/main/ .
```  
+ `--create`: `jar` 명령어 옵션으로 jar 파일을 생성합니다.
+ `--file`: `jar` 파일의 이름을 지정합니다.
+ `-C` : 클래스(`.class`)들을 찾을 디렉토리를 설정합니다.
+ ` . ` : 해당 디렉토리 경로에 있는 모든 `.class`를 지칭합니다.

### 구조  
```Java
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-BOOT-INF
    +-classpath.idx
    +-layers.idx
    +-classes
    |  +-mycompany
    |     +-project
    |        +-YourClasses.class
    +-lib
       +-dependency1.jar
       +-dependency2.jar
```  
스프링 부트 공식문서에서 제공하는 Executable JAR파일 구조는 다음과 같습니다.
+ BOOT-INF
+ META-INF
+ org
  
### BOOT-INF  
개발자가 작성한 클래스파일과 의존성 주입을 통한 jar파일(lib)로 구성되었습니다.  
+ **classpath.idx** 
  + 클래스패스에 추가되어야하는 jar이름들의 목록을 제공
  + 클래스패스에 추가되어야 하는 순서를 나타낸다.
+ **layers.idx**
  + 도커 이미지를 효율적이고 빠르게 배포할 수 있도록 도와준다.

## MANIFEST.MF 이란
이 파일은 JVM 위에서 실행되거나 다른 자바 프로젝트의 라이브러리로 사용할 수 있는 컨테이너입니다.  
클래스(`.class`) 파일이 하나라면 `main()`메소드는 거기 안에 있겠지만,
만약 `.class` 파일이 수백개가 되고 `main()`메소드가 여러개가 있는 경우가 생길 수 있는 `jar` 파일은
`MANIFEST.MF`에 `main()`을 실행할 클래스 이름, 그외 import할 jar파일등 정보가 작성되어있습니다.

스프링 부트 프로젝트(`jar`)을 빌드하고 나서 `jar` 파일을 압축하고 `MANIFEST.MF` 파일을 보면 해당 프로젝트의 `main()` 메소드를 확인할 수 있습니다.
```Java
jar xf stock_rt_1-0.0.1-SNAPSHOT.jar
```  
jar xf 명령어로 압축해제 후 `/META-INF/MANIFEST.MF`를 확인해보면 확인할 수 있습니다.
```Java
Manifest-Version: 1.0
# main() 메서드가 있는 클래스가 지정되어있습니다.  
Main-Class: org.springframework.boot.loader.launch.JarLauncher
# 스프링 부트가 Main-Class후에 호출하는 Class 이름입니다.
Start-Class: org.example.stock_rt_1.StockRt1Application
Spring-Boot-Version: 3.2.4
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Build-Jdk-Spec: 17
Implementation-Title: stock_rt_1
Implementation-Version: 0.0.1-SNAPSHOT
```  
  
## 정리  
war 파일은 이미 서버에 WAS가 구성되어 웹 응용 프로그램만 배포할 때 사용합니다. 
만약 컨테이너 기술처럼 빠르게 코드로 웹 서버 설정하고 배포 및 실행해야하는 경우라면 JAR가 유용합니다.  
  
내장 톰캣을 사용하면 매번 배포하고 실행할 때마다 톰캣 버전, 설정, WAR파일명 변경등 불필요한 과정을 생략할 수 있습니다.

### 참고
참고 사이트: [jar 파일이란](https://velog.io/@wpdlzhf159/Java-jar란)  
참고 사이트: [jar ,war 배포 방법 비교](https://hye0-log.tistory.com/27)