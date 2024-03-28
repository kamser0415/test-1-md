# 변수  
  
어떤 프로그래밍 언어든 데이터를 어디에 담아야 하며, 담아두는 곳을 변수라고 합니다.
담아 두는 것에는 항상 이름을 정해 주어야 합니다.  
  
변수는 네 가지 종류가 있습니다. 나누는 기준은 생명주기가 다르기 때문입니다.  

+ 지역 변수 Local variable
  + 지역 변수를 선언한 중괄호 내에서 선언된 변수
  + 해당 메서드가 종료가 되면 소멸된다.
+ 매개 변수 parameter
  + 메소드에 넘겨주는 변수
  + 메소드가 호출될 때 생명이 시작되고,메소드가 끝나면 소멸된다
+ 인스턴스 변수 instance variable
  + 메소드 밖에,클래스 안에 선언된 변수,static이라는 예약어가 없어야합니다.
  + 클래스에서 상태(state)를 나타냅니다.
  + 객체가 생성될 때 생명이 시작되고, 그 객체를 참조하고 있는 다른 객체가 없으면 소멸된다.
+ 클래스 변수 class variables
  + 인스턴스 변수 앞에 static 예약어가 있습니다.  
  + 클래스가 처음 호출 될 때 생명이 시작되고, 자바 프로그램이 끝날 때 소멸된다.
  
```Java
public class VariableClass {
    int instanceVariable;
    static int classVariable;
    
    void method(int parameter){
        int localVariable;    
    }
}
```
  
지역 변수에 추가로 알아야 하는 키워드  
1. final
   + final을 붙이는 지역 변수는 재할당이 금지된다.
2. lambda
3. 초기화
  + [공식문서 설명](https://docs.oracle.com/javase/specs/jls/se16/html/jls-4.html#jls-4.12.5)  
  + 컴파일러는 초기화되지 않는 지역변수를 사용하려고 시도하는 경우 컴파일 에러를 발생시킵니다.
  + java에서 자동으로 초기화 하지 않는 이유는 코드의 안전성과 신뢰성을 유지하기 위함입니다.
  
