# Collection-List

## 리스트를 왜 사용할까?  
객체들을 순서대로 저장하는 배열은 정적으로 크기가 고정되어있습니다.  
데이터를 추가하거나 삭제할때 배열의 크기가 변경되어야한다면 해당 로직을 개발자가 직접 작성해야합니다.  
List는 데이터의 용량에 따라 동적으로 크기를 변경할 수 있도록 도와주고 데이터의 순서를 보장합니다.  
그러므로 배열보다 유연하게 데이터를 관리할 수 있는 기능을 제공합니다.  

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
  
## 제네릭을 사용하는 이유
List 목록형 인터페이스를 상속한 확장 가능한 배열인 ArrayList는 다양한 자료형이 들어올 경우 
내부에 어떤 참조 자료형이 들어어왔는지 확인을 해야한다.  
제네릭을 사용하여 타입의 안정성과 불필요한 타입확인 과정을 생략 할 수 있기 때문에 
제네릭을 사용한다.  
  
## stack의 상속문제  
ArrayList나 Vector는 LIFO,FIFO 같은 자료구조는 아닙니다.  
LIFO,FIFO 구조를 보장하지 않고, 인덱스로 접근하고 삭제할 수 있기 때문이죠.  
그러면 Vector를 상속한 Stack 클래스는 LIFO 구조를 보장합니다.  
  
Vector를 상속하여 Stack클래스는 LIFO 자료구조를 완전하고 일관적으로 사용할 수 있는 메소드가 부족합니다.  
컬렉션 프레임 워크가 나온 이후 Dequeue 인터페이스를 사용한 ArrayDeqeue나 LikeList를 권장합니다.  
  
### Dequeue와 비교
Stack 클래스 전용 메서드
+ push
+ pop
+ peek
+ search   
  
자료구조에 활용할 수 있는 간단한 메서드만 가지고 있지만, Dequeue는 다음과 같은 메서드를 제공합니다.  
+ add
+ addAll
+ addFirst
+ addLast
+ contains
+ descendingIterator
+ element
   
.. 그외 다양한 자료구조 메서드를 제공합니다.  
  
## 주의사항  
컬렉션을 사용하는 이유는 데이터를 저장할 때 크기를 신경쓰지 않아도 된다는 것인데 
데이터의 맨 앞에 있는 내용을 삭제하면 순서를 지키키위해 아래 로직이 실행됩니다.
```Java
final int newSize;
if ((newSize = size - 1) > i)
    System.arraycopy(es, i + 1, es, i, newSize - i);
es[size = newSize] = null;
```  
+ `i`는 삭제하려는 인덱스
+ `size`는 현재 데이터 갯수  
    
List에 데이터가 5개 저장되어있으면 마지막 인덱스는 4가 됩니다.  
+ remove(lastIdx)
  + 5(`size`) - 1 > 4(`i`) = false
  + `list[4]` = null 으로 마무리
+ 그외 (첫번째 인덱스를 지운다면)
  + 5(`size`) - 1 > 0 (`i`) = true
  + System.arraycopy(es, i + 1, es, i, newSize - i)
  + 얕은 복사로 기존 배열에 한칸씩 당기는 작업이 실행됩니다.
