# WAR와 JAR
  
## JAR 소개  
자바는 여러 클래스와 리소스를 묶어서(`java achive`)라는 압축 파일을 만들 수 있습니다.  
이 파일은 JVM 위에서 실행되거나 다른 자바 프로젝트의 라이브러리로 제공됩니다.  
직접 실행하는 경우 `main()`메서드가 필요하고, `MANIFEST.MF` 파일에 실행할 메인 메서드가 있는 클래스를 지정해야합니다.  
  
스프링 부트 프로젝트(`jar`)을 빌드하고 나서 `jar` 파일을 압축하고 `MANIFEST.MF` 파일을 보면 해당 프로젝트의 `main()` 메소드를 확인할 수 있습니다.  
```Java
jar tf stock_rt_1-0.0.1-SNAPSHOT.jar
```  
jar tf 명령어로 압축해제 후 `/META-INF/MANIFEST.MF`를 확인해보면 확인할 수 있습니다. 
```Java
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.launch.JarLauncher
# main() 메서드가 있는 클래스가 지정되어있습니다.  
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
  
참고 사이트: [jar 파일이란](https://velog.io/@wpdlzhf159/Java-jar란)  
  
## WAR 소개  
WAR(Web Application Archive)라는 이름으로 WAR 파일은 웹 어플리케이션 **서버(WAS)에 배포할 때 사용하는 파일**입니다.
  
### 구조  
+ WEB-INF
  + classes: 실행 클래스 모음
  + lib: 라이브러리 모음(`jar`)
  + web.xml: 웹 서버 배치 설정 파일(생략가능)
+ index.html: 정적리소스
  
+ WEB-INF: 폴더 하위는 자바 클래스와 라이브러리, 그리고 설정 정보가 들어있습니다.
+ 그외 나머지영역은 HTML,CSS 같은 정적 리소스가 사용됩니다.  
  
### 배포
1. Tomcat 실행 `./startup.bat`
2. `tomcat폴더/webapps`에 빌드 파일 복사
3. 빌드파일 이름을 `ROOT.war`으로 변경
4. `tomcat폴더/webapps/ROOT.war`을 톰캣이 압축해제하여 사용
  

## 정리 
jar 파일은 JVM에서 바로 실행할 수 있는 파일이며, was 파일은 WAS(웹서버)에서 실행할 수 있는 java 아카이브 파일입니다.  
그래서 내장형 톰캣을 사용하면 jar으로 실행할 수 있는 이유가 그런 이유입니다.