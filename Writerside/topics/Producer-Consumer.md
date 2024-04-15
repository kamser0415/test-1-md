# Producer-Consumer

## 키워드  
+ Concurrent Collection
+ Producer Consumer 패턴
+ Server Thread Model   
  
## Producer Consumer  
![image_188.png](image_188.png)  
  
[이미지 출처](https://medium.com/@karthik.jeyapal/system-design-patterns-producer-consumer-pattern-45edcb16d544)  
  
멀티쓰레드 환경이나 분산 시스템환경에서 자주 사용하는 프로듀서-컨슈퍼 패턴입니다.  
동시성 문제가 없는 `Queue` 공유 자원에 작업을 올리는 프로듀서와 주기적으로 작업을 읽어 처리하는 컨슈머로 역할을 나누어 처리하는 방식입니다.  
  
프로듀서는 컨슈머에게 `notify`처럼 신호를 주지 않고 작업을 큐에 추가만 하면되고 
컨슈머는 큐에 등록된 작업에 대한 처리만 신경쓰면 됩니다.  

즉, 멀티스레드 환경에서 작업을 분리하여 효율적으로 처리하는 패턴
  
### 장점  
+ 프로듀서와 컨슈머는 독립적으로 동작하므로 모듈화가 가능합니다.
+ 프로듀서와 컨슈머의 수를 조절하여 시스템의 작업량을 조절할 수 있습니다.
+ 프로듀서가 오류가 발생해도 컨슈머는 등록된 작업만 처리하면되고, 반대로 컨슈머가 오류가 발생해도 프로듀서는 등록만하면 나중에 등록된 작업만 처리하면 됩니다.  
  
### 단점  
+ 동기화가 복잡할 수 있습니다. 
  + 생산자와 소비자는 공유된 자원에 동시에 접근하므로 동기화가 필요합니다.
  + 동시에 생산자가 큐에 데이터를 업데이트하는 경우 갱실 문제가 발생할 수 있습니다.  
+ 생산자와 소비자의 속도차이
  + 생상자 속도가 빠를 경우 큐가 넘칠 수 있습니다.
  + 소비자가 많은 경우 다른 소비자가 대기할 수 있습니다.  
+ 다중 생산자 및 다중 소비자의 관리:
 + 실제 시나리오에서는 여러 개의 생산자와 소비자가 동시에 작업할 수 있습니다.
 + 이로 인해 동일한 메시지가 다른 소비자에 의해 처리될 수 있습니다.
+ 데드락(Deadlock) 위험:
  + 잘못된 구현으로 인해 데드락이 발생할 수 있습니다. 예를 들어, 생산자와 소비자가 서로를 기다리는 상태에 빠지는 경우입니다.
  
### 코드  
```Java
class Producer implements Runnable {
    private final BlockingQueue<Integer> sharedQueue;

    public Producer(BlockingQueue<Integer> sharedQueue) {
        this.sharedQueue = sharedQueue;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                System.out.println("Produced: " + i);
                sharedQueue.put(i);
                Thread.sleep(100);
            } catch (InterruptedException ex) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

class Consumer implements Runnable {
    private final BlockingQueue<Integer> sharedQueue;

    public Consumer(BlockingQueue<Integer> sharedQueue) {
        this.sharedQueue = sharedQueue;
    }

    @Override
    public void run() {
        while (true) {
            try {
                System.out.println("Consumed: " + sharedQueue.poll(2000,TimeUnit.MILLISECONDS));
            } catch (InterruptedException ex) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

public class ProducerConsumerTest {
    public static void main(String[] args) {
        BlockingQueue<Integer> sharedQueue = new LinkedBlockingQueue<>();

        Thread producerThread = new Thread(new Producer(sharedQueue));
        Thread consumerThread = new Thread(new Consumer(sharedQueue));

        producerThread.start();
        consumerThread.start();
    }
}
```  
`poll(숫자,단위)`  
이렇게 Queue에 등록된 데이터가 있으면 읽어오고, 데이터가 없는 경우 지정한 시간만큼 대기하고 null을 반환합니다.  
컨슈머는 계속 반복하여 작업을 꺼내서 처리하는 일을 합니다.

## Server Thread Model  
![image_187.png](image_187.png)  
  
`NIC`는 네트워크 인터페이스 카드로 외부 네트워크와 통신할때 데이터를 수신하고 전송하는 역할을 합니다.  
전송받은 데이터를 빠르게 읽어 메모리에 올리는 작업을 (`I/O Thread`)가 하고, 
메모리에 올려진 작업을 처리하는 작업을 (`WorkerThead`)가 합니다.  
  
CPU가 사용할 수 있는 작업 쓰레드 갯수는 한정적인 상황에서 무거운 작업이 많은 `WockerThread`에 대부분의 쓰레드가 사용되어진다면 
네트워크 카드가 가진 메모리를 비워주지 못하게 되고 서버의 과부하가 발생됩니다.  
  
따라서 패킷에 저장된 데이터를 `I/O`하는 쓰레드의 역할을 분리하여 안정적으로 사용할 수 있도록 합니다.  
  
