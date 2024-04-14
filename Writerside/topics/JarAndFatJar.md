# Jar와 FatJar

## Jar
[Oracle Jar 프로그램 패키징](https://docs.oracle.com/javase/tutorial/deployment/jar/index.html)  
  
Jar 파일은 ZIP파일 형식으로 패키지되었습니다.  
1. JDK의 일부인 `Java Archive Tool`사용
2. `jar` 명령어를 사용
3. 데이터 압축, 아카이빙, 압축해제, 아카이브 해체
   + 압축은 데이터 크기를 줄이는 것이 중점
   + 아카이빙은 데이터 저장 구조화하고 보관하는 것이 중점  
  
### 명령어  
| 작업                      | 명령어                                                                                                |
|-------------------------|----------------------------------------------------------------------------------------------------|
| JAR 파일 생성하기             | `jar cf jar-file input-file(s)`                                                                    |
| JAR 파일 내용 보기            | `jar tf jar-file`                                                                                  |
| JAR 파일 내용 추출하기          | `jar xf jar-file`                                                                                  |
| JAR 파일에서 특정 파일 추출하기     | `jar xf jar-file archived-file(s)`                                                                 |
| JAR 파일로 패키지된 응용 프로그램 실행 | `java -jar app.jar`                                                                                |
| JAR 파일로 패키지된 애플릿 실행     | `<applet code=AppletClassName.class archive="JarFileName.jar" width=width height=height></applet>` |

>  (requires the Main-class manifest header)   
> jar파일을 java -jar app.jar 파일로 실행하려면 반드시 MANIFEST.MF 파일에 Main-class가 필수로 필요합니다.  
>   
  
### 한계  
+ `war`와 다르게 `jar`파일은 내부에 라이브러리 역할을 하는 JAR 파일을 포함할 수 없습니다. 
+ 포함한다고 하더라도 인식이 되지 않습니다.  
+ WAR는 WAS 위에서만 실행 가능하므로 WAR를 사용하지 못합니다.
+ MANIFEST 파일에 경로를 적어주면 인식이 가능하지만 Jar 파일안에 jar 파일을 항상 가지고 다녀야합니다.  
  
## FatJar  
`fat jar`또는 `uber jar`라고 불리는 방법  
  
jar 파일을 풀면 class 파일이 나오게 되고 이 파일을 Jar 파일안에 포함시키는 방법입니다.  
수 많은 라이브러리에 나오는 class때문에 뚱뚱한 `Jar`가 탄생합니다.  

```Java
//Fat Jar 생성
task buildFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    duplicatesStrategy = DuplicatesStrategy.WARN
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
```  
실행 방법
```Java
./gradlew clean buildFatJar
```  
### 장점  
+ Fat Jar 덕분에 jar 파일 하나에 필요한 라이브러리를 내장할 수 있습니다.
  
### 단점
+ 어떤 라이브러리가 포함되어있는지 확인하기 어렵습니다.
  + 모두 `.class`파일로 풀려있으므로 어떤 라이브러리가 포함되어있는지 파일명 보고 판단하기 어렵습니다.
+ 파일명 중복을 해결할 수 없습니다.
  + 클래스, 리소스 이름이 동일한 경우 (`jakarta.servlet.ServletContainerInitializer`) 같이 여러 jar 파일에 톰캣 형식에 맞는 파일이 존재하게 됩니다.
   ```Java
   \build\libs\META-INF\services
   Mode                 LastWriteTime         Length Name
   ----                 -------------         ------ ----
   -a----      2023-01-11  오후 12:11             57 jakarta.servlet.ServletContainerInitializer
   ```  
   해당 파일은 `/META-INF/services`안에 모아놓게 되는데 여기서 동일한 이름으로 충돌이 발생합니다.  
  
   