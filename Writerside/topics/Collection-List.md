# Collection-List

Collection을 상속하여 확장한 List 인터페이스를 구현한 클래스를 말합니다.  
  
특징은 배열처럼 순서가 보장이 되며 확장이 가능합니다.  
  
구현체로 Vector, ArrayList, Stack, LinkedList등이 있다.  
  
자료구조인 만큼 `Thread safe`인 구현체와 아닌 구현체로 나뉜다.  
  
`Vector`는 내부에 자바에서 제공하는 `Synchronized`를 사용하여 동시성에 안전하며, 
`ArrayList`는 동시성 제어를 포기하고 속도를 선택했다.  
  
`Stack`은 LIFO을 지원하기 위해 구현되었으며, "호출된 순서를 기억하는 메서드다"  
  
`Abstract`가 붙은 클래스는 그 인터페이스의 일부 기능을 구현했다는 의미다.  
백터나 ArrayList나 구현한 인터페이스는 동일하다.  
  
`ArrayList`는 List인터페이스를 구현하였는데 특징은 
데이터를 특정 인덱스에 넣거나, 맨뒤에 넣는다.  
  
맨 앞에 넣는 메서드는 없다.  
  
대신 데이터를 꺼낼때는 특정 인덱스를 지정해서 꺼낼 수 있다.  
  
동일한 객체를 구분하는 방법은 equals를 사용한다.  
```Java
int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
        for (int i = start; i < end; i++) {
            if (es[i] == null) {
                return i;
            }
        }
    } else {
        for (int i = start; i < end; i++) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    return -1;
}
```  

객체로 인수를 전달해서 삭제할때나 찾을때 모두 `equals`나 참조값을 비교하기 때문에 
`equals`를 오버라이딩해서 동일한 객체의 속성을 선택해야합니다.  
  
그리고 확장 가능한 배열이기 때문에 기본적으로 배열을 내부에 보관하고 있습니다.  
그리고 데이터가 들어올때마다 size라는 변수값을 변경하여 배열크기와 비교하여 자동으로 배열 사이즈를 변경하는 구조인데  
  
이때 전송을 하게 되면 불필요한 배열까지 전송하게 되므로 `trimToSize()`를 통해  
저장된 데이터의 갯수만큼 배열 크기를 줄일 수 있습니다,