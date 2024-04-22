# WorkerThread 패턴  
  
## 비유  
크몽에 단기간 일을 하기 위한 사람을 모집한다고 합니다. 
클라이언트는 크몽(`channel`)에 작업에 필요한 사람을 구하려면 정해진 포멧에 맞춰 글을 작성합니다 (`Request`)
작성할때 급여, 계약 기간등 작성 내용을 확인하고(`argument Evaluation`) 신청(`invocation`)합니다  
  
크몽은 들어온 신청내용을 확인하여 우선 순위를 매겨 프리랜서가 볼 수 있는 `Queue`에 저장합니다.  
  
프리랜서는 들어온 작업을 `Queue`에서 꺼내어 계약을 맺고 작업을 시작(`execute`)합니다  
  
## 정의  
워커쓰레드 패턴은 클라이언트는 채널이 요구하는 구조에 맞게 작업을 요청하면 채널은 해당 작업을 워커 쓰레드가 실행하도록 관리하는 것을 말합니다.  
  
### ClientThread
+ 일을 `Request` 인스턴스 형태로 표현합니다.
+ `putRequest`하는 것이 일의 리퀘스트를 신청하는 것에 해당합니다.
+ `putRequest`하면 일이 실행되는 것을 기다리지 않고 다른 작업을 수행합니다.  
  
### WorkerThread
+ `Request`를 대기합니다.
+ `Request`를 획득합니다.
+ `execute`를 호출하여 `Request`를 실행합니다.  
  
## 코드  
**Main 클래스**  
채널을 만들고 작업자를 초기화 한 다음 작업을 신청할 클라이언트를 연결해주는 메인 클래스입니다.  
```Java
public class MainClass {
    public static void main(String[] args) {
        Channel channel = new Channel(5); // 워커쓰레드 갯수
        channel.startWorkers();
        new ClientThread("Alice",channel).start();
        new ClientThread("Bobby",channel).start();
        new ClientThread("Chris",channel).start();
    }
}
```  
**Request 클래스**  
작업에 대한 약속된 규격이라고 이해하고 있습니다. 처리해야하는 작업과 의뢰자의 이름과 번호를 가지고 있습니다.  
식당으로 치면 주문서와 동일합니다.
```Java
public class Request {

    private final String name; // 의뢰자
    private final int number; // 리쿼스트 번호
    private static final Random random = new Random();

    public Request(String name, int requestNumber) {
        this.name = name;
        this.number = requestNumber;
    }
    public void execute(){
        System.out.println(Thread.currentThread().getName() + " executes " + this);
        try{
            Thread.sleep(random.nextInt(1000));
        } catch (InterruptedException e) {
            //
        }
    }
    @Override
    public String toString(){
        return "[ Request from " + name + " No. " + number + "]";
    }
}
```  
**클라이언트**   
클라이언트 쓰레드는 작업의 인스턴스를 만들어 작업 수행을 관리하는 채널에 전달하는 일을 합니다.
```Java
public class ClientThread extends Thread {
    private final Channel channel;
    private static final Random random = new Random();

    public ClientThread(String name, Channel channel) {
        super(name);
        this.channel = channel;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; true; i++) {
                Request request = new Request(getName(), i);
                channel.putRequest(request);
                Thread.sleep(random.nextInt(1000));
            }
        } catch (InterruptedException e){
            //
        }
    }
}
```
**채널 관리자**  
채널 클래스는 작업을 주고 받기 위해서 공유 데이터를 가지고 있습니다. 
이때 효율적으로 쓰레드를 관리하기 위해 `Guarded Suspension`패턴을 통해 `wait()`와 `notifyAll()`로 자원을 효율적으로 사용합니다.  
```Java
public class Channel {
    private static final int MAX_REQUEST = 100;
    private final Request[] requestQueue;
    private int tail; // 다음에 putRequest 하는 주소
    private int head; // 다음에 takeRequest 하는 주소
    private int count; // request 수

    private final WorkerThread[] threadPool;

    public Channel(int threads) {
        this.requestQueue = new Request[MAX_REQUEST];
        this.head = 0;
        this.tail = 0;
        this.count = 0;

        threadPool = new WorkerThread[threads];
        for (int i = 0; i < threadPool.length; i++) {
            threadPool[i] = new WorkerThread("Worker-" + i, this);
        }
    }

    public void startWorkers() {
        for (int i = 0; i < threadPool.length; i++) {
            threadPool[i].start();
        }
    }

    public synchronized void putRequest(Request request) {
        while (count >= requestQueue.length) {
            try {
                wait();
            } catch (InterruptedException e) {
                //
            }
        }
        requestQueue[tail] = request;
        tail = (tail + 1) % requestQueue.length;
        count++;
        notifyAll();
    }

    public synchronized Request takeRequest() {
        while (count <= 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                //
            }
        }
        Request request = requestQueue[head];
        head = (head + 1) % requestQueue.length;
        count--;
        notifyAll();
        return request;
    }
}
```  
**작업자**  
작업자는 채널 관리자에서 일을 꺼내어 작업을 수행만 합니다.  
워커쓰레드는 `Request` 객체의 `execute`라는 메소드를 갖고 있다는 것만 알고 있고 
해당 작업을 실행하는 일만 합니다.
```Java
public class WorkerThread extends Thread {

    private final Channel channel;

    public WorkerThread(String name, Channel channel) {
        super(name);
        this.channel = channel;
    }

    @Override
    public void run() {
        while (true) {
            Request request = channel.takeRequest();
            request.execute();
        }
    }
}
```
  
## Thread-Per-Message와 차이  
매번 일을 요청하면 새로은 쓰레드를 만드는 `Thread-Per-Message`와는 다르게 
쓰레드의 갯수를 유지하는 방식으로 자원을 관리합니다.  
  
## 사용 목적
클라이언트가 일을 수행할 때 매번 대기하지 않도록 새 쓰레드에게 작업을 실행하도록 하는 방식은 
비효율적입니다. 새 쓰레드를 생성하고 소멸하는 과정보다는 **_쓰레드를 재활용_** 을 통해서 자원을 관리합니다.  
  
### 용량의 제어
채널을 생성할 때 작업 쓰레드의 갯수를 조절할 수 있습니다. 
  
작업 쓰레드의 갯수를 조절한다는 것은
1. 작업 쓰레드가 많을 수록 병렬로 일을 수행할 수 있습니다.
2. 일의 양보다 작업의 쓰레드가 많다면 불필요한 메모리를 낭비하게 되는걸 조절할 수 있습니다.  
  
### Request의 수
채널은 `Request`를 저장하는 공간을 가지고 있습니다.  
클라이언트가 전달하는 `Request`의 양이 작업 쓰레드가 소화하는 양보다 많아 `Queue` 공간이 다 차게 된다면
클라이언트는 대기상태로 빠지게 됩니다. 이를 방지하기 위해서 `Queue`의 공간을 늘리게 된다면 
클라이언트와 작업 쓰레드의 처리 속도의 차이를 극복할 수 있습니다.  
  
다만, `Request`를 보유하기 위해 대량의 메모리를 소비하게 됩니다. 용량과 리소스의 트레이드 오프가 발생합니다.  

**_공유자원을 저장하는 것은 항상 트레이드 오프가 발생합니다._** `Producer-Consumer`패턴도 동일합니다.  
  
## invocation과 execution 분리  
클라이언트는 `Request`라는 클래스의 인스턴스를 생성할 때 인수를 확인하고, 채널에 건네주는 메소드 기동을 합니다.  
워커는 `Request`의 `execute`의 작업을 실행만 합니다.  
  
메소드의 기동(`invocation`)과 메소드의 실행(`execution`)을 분리합니다.    
  
### 응답성의 향상  
nvocation과 execution을 분리하지 않으면 execution에 시간이 걸리는 경우 invocation의 처리까지 늦어지게 됩니다. 그러나 invocation과 execution을 분리해 두면 execution하는 쪽이 시간이 걸려도 invocation을 하는 쪽이 먼저 다음 단계로 나갈 수 있습니다. 이것은 응답성의 향상으로 이어집니다.  

### 실행 순서의 제어  
invocation과 execution을 분리하지 않으면 invoke하자마자 execute해야 합니다. 그러나 invocation과 execution을 분리해두면 invocke한 순서와는 관계없이 execute할 수 있습니다. 이것은 Request에 대해서 우선 순위를 설정하고 Channel이 Worker에 Request를 건네주는 순서를 제어하면 실현됩니다.  
  
### 취소 가능, 반복 실행 가능
invocation과 execution이 분리되어 있으면 "invocation은 일어나지만 execution은 취소한다"라는 기능을 실현할 수도 있게됩니다. invoke한 결과는 Request라는 객체가 되어 있으므로 그 Request를 보존해 두는 것도 반복해서 execute할 수도 있게 됩니다.  
한번만 `invocation` 하더라도 채널에서 삭제하지 않는다면 워커쓰레드가 여러번 실행이 가능합니다.  
  
### 분산 처리의 길
invocation과 execution이 분리되어 있으면 invoke하는 컴퓨터와 execute하는 컴퓨터를 나누기 쉽게 됩니다. Request에 해당하는 개체를 네트워크를 사용해서 다른 컴퓨터에 보내는 것입니다.  
이를 통해 빠르게 여러가지 작업을 invocation하는 측과 `Request`를 확인하여 적합한 `execute`로 실행할 수 있는 서버에 전달이 가능합니다.  
  
## Runnable 인터페이스  
`java.lang.Runnable` 인터페이스를 사용하여 일의 내용을 표현하는 객체를 만들어 채널에 `Request`로 전달할 수 있습니다.  
  
`Runnable` 객체는 실행하기 위한 객체뿐만 아니라, 메소드의 인수나, Queue에 넣을 수 있고, 네트워크에 전달이나, 파일에 보관할 수 있습니다.  

## 다형적으로 사용
지금은 `Request`의 인스턴스만 사용하고 있지만, 작업 쓰레드는 `Request`의 `execute`메소드를 실행하는 것 말고는 알지못합니다.  
  
`Requset`의 서브 클래스를 만들어 전달한다고 하더라도 작업 쓰레드에서 동작할 때 별다른 문제가 발생하지 않습니다.  
작업의 정의를 다양하게 하여 사용하는 방식으로 일의 종류를 나누어도 상관이 없습니다.  
  
## 한명의 Worker
만약 워커가 하나라면 해당 자원을 공유하는 쓰레드도 하나이기 때문에 싱글쓰레드가 되므로 베타제어가 가능합니다.


### 출처
[Worker Thread 패턴](https://12bme.tistory.com/377)
