# Generic

제네릭은 참조 자료형을 불필요한 캐스팅 과정 없이 효율적으로 사용할 수 있는 자바 기술입니다.

제네릭은 JDK 5에 출시가 되었습니다. 자바는 이전 버전 호환성을 위해 컴파일 이후 `.class` 파일에 제네릭 타입을 소거합니다.

컴파일 시점에는 제네릭 제약 조건을 검사하고, 런타임시에는 제네릭 타입을 제거합니다.

[관련 문서](https://www.devinline.com/2014/02/how-generics-works-internally-in-java.html)  
[관련 문서](https://m.site.naver.com/1lAwK)

## 소거
```Java
class GenericsSample<T extends Number> {
    private T data;
    private T[] intArray;
    public GenericsSample(T data){
        this.data = data;
    }
    public T getData(){
        return data;
    }
 // methods for finding sum of all numbers of intArray.
}
```  
컴파일 시점에는 제네릭 타입으로 Number를 상속한 클래스만 올수 있지만 
컴파일이 되면 Number는 소거가 되고 제네릭을 Number로 변환합니다.  
  
```Java
class GenericsSample {
    private Number data;
    private Number[] intArray;
    public GenericsSample(Number data){
        this.data = data;
    }
    public Number getData(){
        return (Number) data;
    }
 // methods for finding sum of all numbers of intArray.
}
```    
  
부모 Node와 자식 IntegerNode 가 있다고 하면
```Java
public class Node<T> {
    private T t;
    public Node(T t) {
        this.t = t;
    }
    public T getT() {
        return t;
    }
    public void setT(T t) {
        System.out.println("Node<T> 클래스의 메서드 호출");
        this.t = t;
    }
}
public class IntegerNode extends Node<Integer> {
    private Integer data;

    public IntegerNode(Integer integer) {
        super(integer);
    }
    @Override
    public void setT(Integer i) {
        System.out.println("IntegerClass에서 호출");
        this.data = data + 1000;
    }
}
```  
부모 노드가 컴파일이 되면 제네릭 소거로 아래와 같이 변경됩니다.
```Java
public class Node {
    private Object t;
    public Node(Object t) {
        this.t = t;
    }
    public Object getT() {
        return t;
    }
    public void setT(Object t) {
        System.out.println("Node<T> 클래스의 메서드 호출");
        this.t = t;
    }
}
```
Integer 노드를 부모 노드로 형변환을 하고 부모 노드의 메소드를 호출하여 실행하면
컴파일 시점에서는 오류를 못잡습니다.
```Java
public class Main {
    public static void main(String[] args) {
        IntegerNode integerNode = new IntegerNode(1);
        Node node = integerNode;
        node.setT("hello");
    }
}
```  
하지만 런타임시에 예외가 발생합니다.
```Java
Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer
	at org.example.generic2.IntegerNode.setT(IntegerNode.java:3)
	at org.example.Main.main(Main.java:10)
```  
이유는 컴파일 시점에서는 T가 선언되지 않아 Object가 들어올 수 있고, 문자열이 인수로 전달이 됩니다.  
```Java
public void setT(Object t) {
    System.out.println("Node<T> 클래스의 메서드 호출");
    this.t = t;
}
@Override
public void setT(Integer i) {
    System.out.println("IntegerClass에서 호출");
    this.data = data + 1000;
}
```  
타입이 소거되면서 오버 라이디잉 아니라 오버 로딩으로 처리하게 되고 
자바는 이런 간격을 제거하고자 브릿지 메서드를 컴파일 시점에 생성합니다.
```Java
### IntegerNode 클래스 내부 메서드
public synthetic bridge setT(Ljava/lang/Object;)V
L0
LINENUMBER 3 L0
ALOAD 0
ALOAD 1
CHECKCAST java/lang/Integer
INVOKEVIRTUAL org/example/generic2/IntegerNode.setT (Ljava/lang/Integer;)V
RETURN
L1
LOCALVARIABLE this Lorg/example/generic2/IntegerNode; L0 L1 0
MAXSTACK = 2
MAXLOCALS = 2
```  
여기서 브릿지 메서드가 실행이 되고 문자열을 `CHECKCAST java/lang/Integer` Integer로 캐스팅하려다 보니 예외가 발생합니다.  
그래서 제네릭 타입을 상속하여 오버라이딩을 할때 부모 클래스로 업캐스팅을 하는 경우에 부모 클래스의 자료형을 명시적으로 입력해야 오류를 방지할 수 있습니다.  
  
컴파일 시점에 제네릭 타입을 검사하기 때문에 제네릭 타입보다 하위타입은 가능하지만, 상위 타입은 예외가 발생합니다.  
```Java
public class Main {
    public static void main(String[] args) {
        Node<?> node = new Node<String>();
    }
}
```  
## 매개변수 제네릭
메소드의 인자로 제네릭을 사용하면 타입 안정성과 불필요한 작업이 필요없습니다. 
그러면 매개 변수에 타입을 지정하기 위해서는 메소드 앞에 제네릭 타입을 지정할 수 있습니다.  

```Java
public static  <T extends Node<Integer>> void getMember(T t) {}
```   

위 같이 사용하면 T로 들어올 수 있는 참조 자료형은 `Node<Integer>`나 상속한 참조 자료형 객체만 들어올 수 있습니다.
  
## static
제네릭 클래스는 컴파일 시점에 검증을 합니다. 그러면 `public static class Member<?>`는 컴파일 시점에 검증을 할 수 없습니다.  
`<?>`는 가상 문자로 클래스가 오지 않습니다. 객체를 생성할때 결정하기 때문입니다.  
  
대신 `public static <T> void methodName(T t)`는 가능합니다.  
여기서 T는 지역변수로 컴파일 시점과는 상관없기 때문입니다.