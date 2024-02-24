# 과제 3일차

## 키워드

익명 클래스  
[오라클 공식문서](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)
  
람다  
  
함수형 프로그래밍  
  
`@FunctionalInterface`

스트림 API  
  
메소드 레퍼런스  

  
## 자바의 람다식은 왜 등장했을까?  
기존의 자바에서는 익명 클래스를 통해 동적으로 동작을 전달하는 방식을 지원했습니다. 이를 통해 인터페이스나 추상 메서드를 구현하여 인스턴스를 생성하고 실행하는 방식으로 기능을 변경할 수 있었습니다. 그러나 익명 클래스를 사용하는 방식은 코드의 장황함과 가독성 문제를 야기할 수 있었습니다.

익명 클래스를 사용하면 클래스를 정의하고 인스턴스를 생성하는 번거로운 과정을 거쳐야 했고, 이는 코드를 이해하는데 오랜 시간이 걸리고 가독성이 떨어지는 원인이 되었습니다. 또한, 코드의 재사용성과 유지보수성이 떨어지는 문제가 있었습니다.

람다식은 이러한 문제를 해결하기 위해 등장했습니다. 람다식은 함수형 프로그래밍의 개념을 도입하여 코드를 간결하고 명확하게 작성할 수 있도록 지원합니다. 익명 클래스와는 달리 람다식은 함수의 형태로 직접 작성되며, 메서드의 매개변수로 전달하거나 변수에 할당하여 사용할 수 있습니다.

람다식을 사용하면 익명 클래스보다 훨씬 간결한 코드를 작성할 수 있으며, 이는 코드의 가독성을 높이고 유지보수성을 향상시킵니다. 따라서 자바에서 람다식이 등장한 주요 이유 중 하나는 코드의 가독성과 유지보수성을 개선하기 위함입니다.

참조 : 오라클 공식문서,모던 인 자바액션
  
## 람다식과 익명 클래스는 어떤 관계가 있을까? - 람다식의 문법은 어떻게 될까?  
  
[자바 8은 함수형 프로그래밍이 도입되었을까?](https://tecoble.techcourse.co.kr/post/2021-09-30-java8-functional-programming/)  
  
함수형 인터페이스를 표현식을 통해서 작성하는 방식을 람다식이라고 합니다.  
  
함수형 인터페이스는 추상 메서드를 하나만 가지고 있는 인터페이스를 말하며 
 
`@FunctionalInterface`라는 어노테이션을 추가하여 컴파일 오류를 확인할 수 있습니다.  
  
익명 클래스는 인수로 전달해야하는 클래스를 지역 클래스를 익명으로 상속하여 구현하는 방식입니다.  
  
자바는 객체지향언어이면서 클래스 기반이기 때문에 함수만 별도로 사용할 수 없고 클래스 내부에 선언되고 
인스턴스가 함수로 사용할 수 있습니다.  
  
다른 언어와 다르게 자바는 함수형 인터페이스를 사용하기 위해서 클래스를 임시로 만들어야하기 때문에 
람다식은 익명 클래스의 일부라고 생각이 됩니다.  
  
  
### 람다식의 문법
람다식으로 변환할 수 있는 함수형 인터페이스를 선언합니다.  
  
```Java
public interface add {
    // 추상 메서드가 하나이다
    int calculate(int a, int b);

    default void print() {
        System.out.println("함수형 인터페이스");
    }
}
```  
  
람다식은 해당 입력 인수 타입과 갯수, 그리고 반환 타입만 일치하면 됩니다.   
  
```Java
public class Homework3Day {

    @Test
    @DisplayName("동적 파라미터")
    void DynamicParameter(){
        Addable addable = (a, b) -> { return a + b; };
        int result = addable.calculate(5, 4);
        Assertions.assertThat(result).isEqualTo(9);
    }

}

interface Addable {
    // 추상 메서드가 하나이다
    int calculate(int a, int b);

    default void print() {
        System.out.println("함수형 인터페이스");
    }
}

```

```Java
Addable addable = (a, b) -> { return a + b; };
```  
+ (a,b): 파라미터 리스트
+ -> : 화살표 , 람다의 파라미터 리스트와 바디를 구분합니다.
+ 람다 바디: 파라미터 리스트를 더한 값을 반환하는 표현식입니다.  
  
**람다 바디 표현 방법**  
1. (a,b) -> a + b; (paramters) -> expression;
2. (a,b) -> { return a + b ; }; (parameters) -> { statements; } ;
  
표션식 스타일과 블럭 스타일이 있습니다.  

**표현식(Expressions)**:
표현식은 값을 계산하고 결과를 반환하는 코드 구문입니다.
표현식은 변수, 상수, 연산자, 메서드 호출 등으로 이루어져 있습니다.
표현식은 항상 값을 가지며, 그 값을 반환합니다.  

**문장(Statements)**:
문장은 프로그램의 실행 단위를 나타내는 코드 구문입니다.
문장은 하나 이상의 표현식을 포함할 수 있으며, 프로그램의 흐름을 제어하거나 작업을 수행합니다.
문장은 세미콜론(;)으로 끝납니다.  
  
  
**메서드 래퍼런스**  
클래스 이름이나 인스턴스 명 뒤에 `::`를 사용하여 참조할 수 있습니다.  
```Java
interface Addable {
    // 추상 메서드가 하나이다
    int calculate(int a, int b);

    default void print() {
        System.out.println("함수형 인터페이스");
    }
}

class Exam {

    public int add(Addable addable, int a, int b) {
        return addable.calculate(a, b);
    }
}

public class Main {
    public static void main(String[] args) {
        // 메서드 래퍼런스를 사용하여 Addable 인터페이스를 구현
        Addable addable = Integer::sum;

        Exam exam = new Exam();
        int result = exam.add(addable, 10, 20);
        System.out.println("Result: " + result); // Output: Result: 30
    }
}
```