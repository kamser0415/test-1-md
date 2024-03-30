# CPU 최적화에 따른 문제  
  
## CPU Cache
  
![image_155.png](image_155.png)  
  
Memory Access 속도보다 CPU의 속도가 빠르기 때문에 속도 차이로 인한 문제가 Memory wall이 발생합니다.  
그래서 속도 차이를 줄이기 위해 CPU에 Cache를 두고 필요한 데이터를 미리 Memory에서 읽어옵니다.  
  
이때 성능 최적화를 위해 cache로 인해 멀티쓰레드 환경에서는 문제가 발생합니다.  
  
### Stale Data
+ Memory 데이터를 읽지 않고 캐시 데이터를 읽어서 발생하는 문제  
  
| 시간순  | A    | B     | M     |
|------|------|-------|-------|
| 10ns |      |       | false |
| 11ns |      | false |       |
| 12ns | true |       | true  |
| 13ns |      | false |       |
    
```Java
static boolean stop = false ; // 10ns

new Thead(()-> stop = true).start(); // A 
new Thead(()-> {
    int counter = 0;
    while(stop == false){
        counter++;
    }
}).start(); // B
```  
1. 메모리에 stop = false 가 저장됩니다.
2. B쓰레드를 코어2에서 실행할때 캐시에 stop = false로 저장합니다.
3. 반복문을 실행하는 사이 코어1에서 A쓰레드를 실행하여 메모리에 저장된 값인 stop = true로 변경합니다.  
4. 코어2에서 메모리가 아닌 캐시에서 데이터를 읽어서 반복문이 유지됩니다.  
  
## ReOrdering (재정렬)  
```Java
static int y=0;
static int x=0;

new Thead(()-> {
    x = 100;
    y = 50;
}).start(); // A 
new Thead(()-> {
    if( y > x ) {
        // Do Something 
    }
}).start(); // B
```  
CPU는 코드 최적화를 위해 코드의 순서가 변경될 수 있습니다.  
1. 메모리에 `y=0`,`x=0`으로 초기화
2. 쓰레드 A가 실행되면서 코드 최적화로 `y = 50;`이 먼저 할당됨.
3. 그 사이 쓰레드 B가 실행되어 `y = 50 ,x = 0` 인 상태라서 반복문이 실행되는 구조  
  
## 해결 방법  
  
> 재정렬, 최신 데이터 읽기, 원자성 보장입니다.  
>  
  
### Visibility(가시성)
+ 가시성을 부여하여 해당 코드는 반드시 MainMemory에서 값을 읽거나 씀
```Java
private volatile static boolean shouldStop = false;
```  
  
사용하는 이유는 중요한 설정이나 상태 값과 같은 경우 `volatile`키워드를 사용하면 다른 쓰레드에서 해당 변수를 
로컬 캐시가 아닌 메모리에서 읽도록 합니다.  
  
대신 원자성은 보장되지 않습니다. 
여러 쓰레드가 동시에 `volatile` 변수에 접근하여 값을 수정하는 경우에 읽는 과정과 쓰는 과정에서 생기는건 방지할 수 없습니다.
  
### Atomicity(원자성)  
+ 여러 쓰레드에서 메모리를 공유하면 원하지 않던 오작동을 하게 됨
+ `i++` : 이건 Automic Operation이 아닙니다.
  + 변수 i의 값을 읽어옴 (Read)
  + 읽어온 값에 1을 더함 (Add)
  + 결과를 변수 i에 쓰기 (Write)
  + 이 과정을 수행하기 때문에 그 사이에 데이터가 일관되지 않을 수 있습니다.  
+ 하나의 쓰레드에서만 해당 데이터를 수정할 수 있도록 하는걸 `Atomic Operation`이라고 합니다.  
  + `critical section(임계구역)`을 지정하여 하나의 쓰레드만 접근할 수 있도록 합니다.  
  
    
## 관련 키워드  
1. volatile - 가시성
2. synchronized - 임계구역
3. lock - 임계구역
4. Interlocked.Increment
5. Memory Barrier