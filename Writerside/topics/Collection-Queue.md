# Collection-Queue
  
## 필요한 이유  
List와 동일하게 데이터 저장 순서가 지켜진다. 
다만 차이점은 List는 데이터가 먼저 저장된 객체를 꺼내는 목적이 아니라 순서를 유지하기 위한 컬렉션이라면 
Queue는 먼저 들어온 객체를 꺼내는데 집중된 컬렉션이다.  
  
## LinkedList  
Queue 구현체로 데이터를 Node라는 객체로 관리한다.  
Node안에는 자기 앞과 뒤만 기억하기 때문에 전체적인 저장 순서를 알지 못한다.  
  
그래서 데이터를 찾는 List 메서드인 `get(idx)`의 메서드 구현부를 확인해보면 
```Java
// 컬렉션에 저장된 size를 반으로 으로 나눈다.
if (index < (size >> 1)) {
    Node<E> x = first;
    for (int i = 0; i < index; i++)
        x = x.next;
    return x;
} else {
    Node<E> x = last;
    for (int i = size - 1; i > index; i--)
        x = x.prev;
    return x;
}
```  
`size/2`보다 작으면 처음부터 `next`로 찾고, 크면 `pre`로 찾는다. 
그래서 인덱스를 알고 있으면 바로 찾을 수 있는 `ArrayList`와는 속도 차이가 발생한다.  
  
다만 데이터를 삭제하고 중간에 추가하거나 맨 앞에 추가하는 경우에는 `LinkedList`가 훨씬 빠르다  
`ArrayList`는 내부에 배열로 관리하기 때문에 맨 뒤에 저장하는 경우가 아니면 데이터를 모두 한칸씩이나 저장하는 컬렉션 크기만큼 이동하는 반복문을 실행한다.  
  
`LinkedList`는 첫번째 노드를 찾아서 그 앞에 노드와 연결해주면 끝난다.  
  
### 장점  
FIFO 자료구조의 인터페이스인 `Queue`를 상속한 `Deque`를 구현했다.  
Stack 클래스보다 일관적이고 완성된 메서드를 제공하는 `Queue`를 구현하엿고, 
모든 요청을 FIFO으로 처리하면 맨 앞에 있는 데이터만 꺼내는게 아니라 앞 뒤로 데이터를 넣고 뺄수 있는 `Deque`를 구현하였습니다.  
  
List 인터페이스를 구현했지만, `ArrayList`와 다르게 초기 용량을 설정할 수 없는데 
배열처럼 붙어있는 주소구조가 아니라 Node 구조로 메모리 주소가 떨어져있으므로 설정하지 못한다.  
  
중복된 메소드 명이 있지만 명시적인 `addFirst,removeFirst`같은 메소드를 사용한다 내부에서도 명령어는 동일하게 동작한다.  
  
`Collection`을 구현하여 데이터 조회기능인 `contaions`를 구현했는데 위 메소드와 동일하게 
처음부터 찾던지 맨뒤부터 찾는 방식이기 때문에 데이터가 많을 경우에는 `Set`같은 자료구조를 활용하거나 
특정 인덱스로 접근하여 찾아오려면 `ArrayList`를 사용하는게 낫다.  
  
다만 처음부터 조회하는 경우나 맨 뒤에서부터 조회하는 경우에는 `Iterator`를 상속한 `ListIterator`를 내부에 가지고 있으므로 
확장된 기능을 사용할 수 있다.