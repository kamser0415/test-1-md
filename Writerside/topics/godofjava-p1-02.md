# hello java

java --module-path C:\study\book\godofjava\javafx-sdk-22\lib --add-modules javafx.controls,javafx.fxml,javafx.base,javafx.web -jar godofjava-230902.jar

```Java
>java
Usage: java [options] <mainclass> [args...]
           (to execute a class)
   or  java [options] -jar <jarfile> [args...]
           (to execute a jar file)
   or  java [options] -m <module>[/<mainclass>] [args...]
       java [options] --module <module>[/<mainclass>] [args...]
           (to execute the main class in a module)
   or  java [options] <sourcefile> [args]
           (to execute a single source-file program)

 Arguments following the main class, source file, -jar <jarfile>,
 -m or --module <module>/<mainclass> are passed as the arguments to
 main class.
```  
`java` 명령어를 사용하면 자바 클래스,jar 파일, 모듈안에 있는 메인 메소드, 싱글 파일 프로그램을 실행합니다.   
  
```Java
javac
Usage: javac <options> <source files>
```  
소스파일을 자바 바이너리 코드로 컴파일합니다.  
  
윈도우 환경에서 프로그램을 실행하려면 exe,bat,com 확장자로 끝나야 합니다. 
커맨드 창 어디에서든 실행을 하기 위해서는 윈도우 환경 변수 설정에 자바 bin 폴더 위치를 등록하면 
어디 위치에서든 실행이 가능합니다.  
  
## 실행  
코드작성 > 컴파일(javac) > 실행 (java)  

1. 에디터를 활용하여 자바 소스 파일을 작성
2. 컴파일러 javac로 자바 소스 파일을 자바 바이너리 코드로 컴파일
3. 컴파일 결과를 해당 컴퓨터 디스크에 저장
4. 자바 JVM이 독립형 플랫폼으로 운영체제와 상관없이 실행된다.  
  
`java`로 실행하려는 클래스 파일은 반드시 `main()`함수가 있어야합니다.  
  
모든 클래스가 `main()`함수가 필요한 것은 아니지만, `java`명령어로 실행되는 클래스는 
반드시 `main()`함수가 있어야합니다.  
  
그러면 `main`함수 앞에 `public static void`는 왜 필요할까? 
1. public은 모든 패키지에서 자유롭게 접근이 가능하다.
2. static은 main 함수를 실행하기 위해 인스턴스를 생성하지 않아도 된다.
3. void는 main도 함수이기 때문에 반환타입을 반드시 작성해야한다.  
  
자바는 멀티쓰레드 환경이기 때문에 `main()`함수가 실행되는 환경을 main thread라고 한다.  
  
+ () 소괄호
+ {} 중괄호
+ [] 대괄호
