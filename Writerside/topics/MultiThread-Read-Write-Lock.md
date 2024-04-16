# Read-Write-Lock  
  
## 다루는 내용  
+ Interlocked Class
+ Read-Write Lock 패턴
+ ReaderWriterLock Class
  
## Interlocked Class
+ 원자성을 지원
  + 데이터 손실이 발생하지 않는다.
+ 가시성과 Reordering 방지는 지원안함
  + 가시성을 지원하지 않는다는 메인 메모리가 아니라 캐시 데이터를 읽어와서 연산할 수 있다.
  + Reordering은 데이터 초기화 순서가 변경되어 원하는 결과가 안나올 수 있다.  
  
## Read-Write 패턴  
멀티 쓰레드 프로그래밍에서 공유 자원에 접근할 때 문제가 발생할 수 있는 영역(`critical section`)은 동시에 두 개 이상의 스레드가 
자원을 변경하지 못하도록 `Mutex`방식을 사용합니다. 
> Mutex란  
> 뮤텍스는 이진 상태를 가지며, 잠금(lock)과 해제(unlock) 연산을 수행합니다.
주로 **임계 영역(critical section)** 에서 사용되며, 여러 스레드가 동시에 접근하지 못하도록 보호합니다.  
>   
  
모든 쓰레드가 수정하는 게 아니라, 일부의 쓰레드만 수정하고 나머지는 데이터를 조회만 하는 경우에는 여러 쓰레드가 동시에 접근해도 상관없습니다. 
따라서 데이터를 수정하려고 하는 쓰레드에게만 잠금을 걸면 되는 걸까요?  
  
아닙니다. 읽는 쪽도 잠금이 필요합니다.  
  
동시성 문제가 발생되는 이유중 하나는 데이터를 수정하는 프로그램이 완료되기전에 다른 쓰레드에서 데이터를 읽기 때문에 발생합니다.  
데이터를 변경하는 중이라면 다른 쓰레드에서는 접근을 하지 못하고 완료되어 최신 데이터로 변경되었을 때 읽어야합니다.  
  
읽기 잠금과 쓰기 잠금을 나누어 사용하는 방식을 `Read-WriteLock`이라고 합니다.  
  
### 주의사항  
읽기 잠금을 획득한 쓰레드 A가 공유 자원을 조회하고 있을 때, 다른 읽기 잠금을 얻으려는 쓰레드 B,C는 자유롭게 획득할 수 있습니다. 
수정을 하지 않기 때문에 최신 데이터를 조회만 하기 때문이죠   
다만, 읽기 잠금을 획득한 쓰레드 A가 조회하고 있는 경우에 쓰레드 B가 쓰기 잠금을 획득하려고 합니다.  
쓰레드 C,D,E,F는 읽기잠금으로 조회를 하려고 하고 있습니다. 쓰레드 실행 순서가 없는 일반적인 상황에서 
읽기 잠금이 계속 획득한다면 쓰기 잠금은 영원히 실행되지 않을 수 있습니다. 이러한 현상을 `write-starvation(쓰기 기아상태)`가 됩니다.  
  
`write-starvation`을 방지하고자 쓰기 잠금을 원하는 쓰레드가 락을 얻을 때까지 대기하는 방식도 있습니다. 
이런 구현은 너무 불필요하게 `read Lock`이 대기하는 상황이 발생할 수 있어서 효율성이 떨어집니다.  
다만 순서대로 실행되는 방식이기 때문에 공정성은 있습니다.  
  
### 코드  
```Java
public class ReadWriteLock {
    private AtomicBoolean shouldStop = new AtomicBoolean(false);
    private AtomicInteger readCount = new AtomicInteger(0);
    private AtomicInteger writeCount = new AtomicInteger(0);
    private volatile String message = new Date().toString();

    public void startUp() throws InterruptedException {
        System.out.println("Process Start!");

        List<Thread> threads = new ArrayList<>();

        for (int i = 0; i < 2; i++) {
            Thread thread = new Thread(this::write);
            thread.setName("Writer" + i);
            threads.add(thread);
        }

        for (int i = 0; i < 6; i++) {
            Thread thread = new Thread(this::read);
            thread.setName("Reader" + i);
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.start();
        }

        System.out.println("Press any key to stop...");
        Thread.sleep(5000);  // Simulate waiting for key press

        shouldStop.set(true);

        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("All done!");
    }

    private void write() {
        while (!shouldStop.get()) {
            if (!writeCount.compareAndSet(0, 1)) {
                continue;
            }

            try {
                if (readCount.get() > 0) {
                    continue;
                }

                String request = new Date().toString();
                message = request;
                System.out.println(Thread.currentThread().getName() + " : " + request);
            } finally {
                writeCount.decrementAndGet();
            }

            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    private void read() {
        while (!shouldStop.get()) {
            if (writeCount.get() > 0) {
                continue;
            }

            readCount.incrementAndGet();

            try {
                String readMessage = message;
                System.out.println(Thread.currentThread().getName() + " : " + readMessage);
            } finally {
                readCount.decrementAndGet();
            }

            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReadWriteLock().startUp();
    }
}
```  
전체 코드입니다.  
  
읽기 잠금은 세마포어 방식, 쓰기 잠금은 뮤텍스 방식을 통해서 효율적으로 쓰레드를 관리합니다.  
  
### 읽기 작업  
```Java
while (!shouldStop.get()) {
    if (writeCount.get() > 0) {
        continue;
    }

    readCount.incrementAndGet();

    try {
        String readMessage = message;
        System.out.println(Thread.currentThread().getName() + " : " + readMessage);
    } finally {
        readCount.decrementAndGet();
    }

    try {
        Thread.sleep(50);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}
```  
1. `!shouldStop.get()` 먼저 전체 로직을 실행할 수 있는지 확인합니다
2. `writeCount.get()`쓰기 잠금인지 확인합니다.
   1. 쓰기 잠금이 걸려있다면 쓰레드는 락을 `0.05초` 간격으로 확인합니다.
3. 읽기 잠금을 추가하고 로직을 수행합니다.  
  
  
### 쓰기 작업
```Java
 while (!shouldStop.get()) {
    if (!writeCount.compareAndSet(0, 1)) {
        continue;
    }

    try {
        if (readCount.get() > 0) {
            continue;
        }

        String request = new Date().toString();
        message = request;
        System.out.println(Thread.currentThread().getName() + " : " + request);
    } finally {
        writeCount.decrementAndGet();
    }

    try {
        Thread.sleep(50);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}
```
  
쓰기 잠금이 2가지를 확인합니다.  
1. 다른 멀티쓰레드에서 쓰레드 잠금이 걸려있는지 확인합니다.
2. `writeCount.compareAndSet(0, 1)` 쓰기 잠금이 없다면 쓰기잠금을 획득합니다.  
3. 그리고 읽기 잠금이 걸려있는지 확인합니다. 
   1. `writeCount.decrementAndGet()`읽기 잠금이 걸린 경우 쓰기잠금을 반납합니다.
4. 읽기잠금이 없는 경우(`readCount.get() > 0`)
   + 기타 쓰기를 실행하고 쓰기 잠금을 반납합니다.  
 
### 주의사항  
해당 코드는 동시성 문제가 발생할 수 있습니다.
```Java
if(readCount.get() > 0)
```  
이 코드는 동시성 문제가 발생할 수 있습니다. 
1. "조회": readCount.get();
2. 쓰레드 Z가 조회`readCount.get()`
3. "비교": readCount.get() > 0  
    
따라서 해당 코드영역은 수정해야합니다.  
  

  