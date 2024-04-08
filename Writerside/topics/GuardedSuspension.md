# GuardedSuspension

## 키워드  
* Write-Through, Write-Back
* CPU Cache Hit, Miss
* No Allocate, Allocate
* Memory Barrier(or Fence)
* Lock
* Quarded Suspension
  
### CPU 캐싱 정책  
CPU Cache에 write를 할 때 정책입니다.  

+ Write-Through
  + CPU Cache에도 write를 하고, main Memory에도 write를 하는 방식
+ Write-Back(Lazy write)
  + CPU Cache에만 write를 하고, Cache buffer가 main Memory에 write를 하는 방식
  
Read나 Write하려는 메모리가 CPU 캐시에 있으면 `Cache Hit`, 없으면 `Cache Miss` 라고 합니다.     
  
이때 Write를 하려고 하는데 `Cache Miss`로 캐시에 데이터가 없을 경우에
+ write No Allocation(`할당`)
  + 바로 메모리에 작성합니다.
+ write Allocation(`할당`)
  + 메인 메모리에서 `Cache`로 데이터를 `Allocation(할당)`한 뒤 Cache에 씀
  + 쓰여진 캐시 데이터는 `write-back`으로 `cache buffer`가 메모리에 write 합니다.  
  
### Dirty Flag
CPU 캐시된 데이터가 수정되었는지 저장하는 변수 혹은 메모리를 말합니다.  
`Cache buffer`가 저장된 캐시 데이터가 변경을 확인해야 메인 메모리에 쓸지 안쓸지 알 수 있기 때문에 존재합니다.  
  
`Cache Server`에 캐시된 정보가 있을 때 `Dirty Flag`를 확인하여 `RDB`에 기록 유무를 결정합니다.  
   
### 캐시 정책 이해하기  

![image_175.png](image_175.png)  
  
CPU 정책은 2가지로 정리됩니다.
1. Write-Through,No-Allocation - 캐시에 없는 데이터는 메모리에 바로쓴다.
2. Write-Back,Allocation - 캐시에 없는 데이터는 캐시에 저장한 이후 캐시 버퍼가 `Dirty Flag`를 확인하여 메모리에 기록한다.  
  
> Write-Through 방식은 바로 메모리에 작성하기 때문에 캐시로 데이터를 가져올 필요가 없기에 No-Allocation 방식과 한 세트가 됩니다.  
>   
  
[캐시 정책 출처](https://m.site.naver.com/1lwCd)
    
**메모리 시스템 구성**  

CPU가 연산을 하기 위해서는 데이터를 읽고 써야하는데 데이터는 메인 메모리에 저장되어있습니다. 
CPU와 메모리는 속도차이가 크기때문에 CPU가 직접 메모리에 접근하여 읽고 쓰는 방식보다 
중간에 캐시 메모리를 두어 메인 메모리의 데이터를 임시로 캐시 메모리에 저장하여 사용하는 방식입니다.  
  
캐시메모리의 용량이 작아서 연산에 필요한 데이터를 모두 저장할 수 없습니다.  
  
캐시메모리에없는 데이터를 저장하고 읽을때 정책에 따라 나뉩니다.  
  
### write-through  
`Write`할때 `Through` 드라이빙 쓰루 처럼 관통한다는 의미로 해석하면 됩니다.  
CPU 연산 결과를 기록할때 해당 데이터가 캐시에 없는경우 캐시메모리와 메인메모리에 모두 저장하는 정책입니다.  
  
### write-through with buffer
`Write`할 때 CPU가 직접 메인 메모리에 기록하는 방식을 사용한다면 캐시메모리를 사용할 이유가 없습니다.  
CPU가 캐시메모리에 기록하면 동시에 버퍼가 캐시 메모리에 있는 데이터를 메인 메모리에 기록합니다.  
이렇게 하므로 CPU는 캐시메모리에만 기록하면 되기 때문에 CPU가 I/O로 인한 대기하는 시간을 줄일 수 있습니다.  
   
### write back
Write-Back은 캐시 메모리에만 데이터를 기록하다가 새로운 데이터 블록을 사용하게 되면 그때 메인 메모리에 기록하는 방식입니다.  
말 그대로 기록은 뒤에서 한다는 의미입니다.  
이 방식을 사용하면 데이터 블록이 교체되지 않는 한, 메인 메모리에는 데이터는 업데이트하지 않고 캐시메모리만 계속 수정하게 됩니다.  
  
### No-Allocate
캐시메모리에 필요한 데이터 블록이 없을 때 정책입니다.  
캐시 메모리는 메인 메모리의 데이터 주소의 복사본을 가지고 있습니다.  
  
`Cache Miss`시 메인 메모리에만 수정하는 방식입니다.  
캐시메모리는 결국 메인 메모리의 복사본이기 때문에 원본인 메인 메모리만 수정하는 방식입니다. 

### Allocate  
캐시 메모리에 접근했을 대, 원하는 데이터 블록 주소가 없을 경우 메인 메모리의 데이터 블록 주소를 
캐시 메모리에 로드를 하고 캐시 메모리에 있는 데이터를 오버라이딩 하겠다는 의미입니다.  
  
### 정리
+ `Write-Through,No-Allocation`  
  + CPU가 연산에 필요한 데이터 블록이 없을 경우 메모리에서 직접 읽습니다.
  + CPU가 연산의 결과를 기록할때 캐시에 복사본 주소가 없는 경우 직접 메모리에 기록합니다.
+ `Write-Back,Allocation`
  + CPU가 연산에 필요한 데이터 블록이 없을 경우 메인 메모리 주소를 캐시 메모리에 복사하고, 거기에 기록을 합니다. 그리고 `Dirty Flag`가 기록되며 `Write-Back`으로 캐시 버퍼가 메인 메모리에 기록합니다.
  + CPU가 연산에 결과를 기록할때도 위 방식과 동일합니다. 
  
## 코드레벨  
```Java
// count = 0 으로 초기화

// countNumber Thread
while(true) {
  count += 1; // normal int
  flag = true; // flag는 volatile로 항상 메모리에서 읽어와야합니다.
}
// printCount Thread
while(true) {
  if(flag){
    foo(counter);
  }
}
```  
실행 순서를 아래와 같이 한다고 한다면,  
1. counter += 1;
2. flag = true;
3. if(flag)
4. counter += 1;
5. foo(counter);  

CPU는 대부분 write-back으로 동작합니다.  
1. CPU -> 캐시메모리 ( flag = true 할당) -> 버퍼가 메인 메모리에 기록
2. printCount 쓰레드가 volatile flag 값을 메인 메모리에서 읽어옴  
3. 그때 기록하기 때문에 `printCount`는 짧은 시간 대기
4. 그 사이에 4번 로직이 실행됨
5. foo에 인자로 counter = 2가됨
  
이때 `flag`에 `volatile`을 선언하여 항상 최신 데이터를 가져와야한다.  
  
## Memory Barrier(Fance)  
+ Reordering 방지
+ 가시성 제공  
  
Read,Write,Read And Write 3가지 종류의 메모리 베리어가 있지만, 
대부분 Full Memory Barrier(`Read And Write`)를 말합니다.  
  
저수준 프로그래밍 방식이기 때문에 권장하지 않고 자바에서는 volatile,synchonized,Lock등을 사용하면 됩니다.  
  
메모리 베리어를 사용하다가 다른 쓰레드와 부딪칠 경우 독립적으로 실행되는 구조입니다.  
Lock 블럭을 사용하면 메모리 베리어를 사용하게 되므로 원자성을 보장하게 합니다.  


## 이 패턴을 왜 쓸까?  
[Guarded-Suspension 패턴](https://12bme.tistory.com/375)  
  
자바는 멀티쓰레드 환경이고 데이터의 효율적인 관리를 위해서 공유 데이터를 사용합니다.  
이때 각각 쓰레드가 작업한 내용이 누락되지 않도록 하기 위해 동시성이 보장되는 메모리 배리어 블럭을 사용합니다.  
`Synchronized`나 `lock`등을 사용하여 해당 구간(`cretical section`)을 하나의 쓰레드만 접근할 수 있도록 하는 방식을 사용합니다.  
  
만약 공유 데이터에 작업을 추가하거나 상태를 변경하는 `produce`역할인 쓰레드와 
추가된 데이터를 변경이나 추가 작업을 하는 `consumer`역할인 쓰레드가 있을때 구조는 다음과 같습니다.  
  
[자바 공식문서](https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html)
```Java  
public void guardedJoy() {
    // Simple loop guard. Wastes
    // processor time. Don't do this!
    while(!joy) {
      // 
    }
    System.out.println("Joy has been achieved!");
}
```
`consumer`역할 쓰레드는 공유 변수 `joy`가 `true`가 될때까지 무한 반복문으로 대기를 해야합니다.  
내부 로직에는 별다른 일을 하지 않고 있지만, `guardedJoy` 메소드를 실행하는 쓰레드는 `joy`값이 변경될 때까지 서버의 리소스를 가지게 됩니다.  
  
```Java
public synchronized void guardedJoy() {
    // This guard only loops once for each special event, which may not
    // be the event we're waiting for.
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```  
  
이렇게 `critical section`과 `wait`를 통해서 하나의 쓰레드만 접속할 수 있도록 작성하고 
```Java
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```
프로듀서 쓰레드는 공유 변수를 수정하는 과정에서 락을 통해 갱신 손실을 방지하고, 작업이 완료되면 
컨슈머 쓰레드에 대기중인 쓰레드를 깨워 `joy`가 변하길 기다리는 무한 반복을 제거할 수 있습니다.  
  
### 주의사항
`notifyAll()`이나 `wait()`는 공유 객체내에 `Synchronized`나 `Lock`인 Block 내에서만 사용해야합니다.  
  
Thread는 동기화된 객체에 접근하여 `wait()`할 경우 해당 객체의 대기열에 들어갑니다.  
`notifyAll()`은 동기화된 객체에서 대기중인 쓰레드를 모두 깨워 해당 객체에 접근을 시도하도록 합니다.  
  
따라서 동기화가되지 않은 외부환경에서 사용할 경우 예외가 발생합니다.  
  
### 정리
잠금을 사용하여 공유 자원에 대한 데이터 일관성을 보장하고, 프로듀서와 컨슈머 역할의 쓰레드가 나누어진 구조일 경우 
쓰레드 대기와 대기열을 통해 불필요한 리소스 낭비를 줄일 수 있습니다.  
프로듀서는 작업이 완료된 후 해당 자원에 접근하려는 쓰레드를 제어하여 안정적인 자원관리와 쓰레드 동기화를 제공합니다.  
  
  
