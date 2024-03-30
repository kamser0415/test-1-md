# MultiThread

## Process
프로세스는 중앙처리장치(프로세서,CPU)에서 실행 중인 프로그램 또는 작업으로 정의합니다.  
프로세스는 기본적으로 중앙처리장치(프로세서)와 주기억장치를 운영체제에게 할당 받아야 실행할 수 있습니다.  

운영체제에게 프로세스는 2가지 역할로 나눌 수 있습니다.
1. 자원 할당의 단위
2. 작업(`task`)의 단위  
  
프로세스는 각각 독립된 메모리 공간을 갖습니다. 운영체제에서 프로세스끼리 동일한 메모리 공간을 공유하지 못하도록 제어하기때문이죠  
  
운영체제가 프로세스를 실행하기 위해 프로세서를 할당하는데 그때 할당하는 단위가 프로세스입니다.  

### 프로세스간 통신 방법
+ 프로세스 끼리 메모리 공유가 안되기에 통신을 해야합니다.  
+ HTTP(L7) 통신 방법, TCP(L4) 방식 중 장단점을 보고 파악
+ MSA를 지향하는 현재에서는 프로세스간 통신은 필수입니다.  
  
그러면 왜 멀티 프로세스 방식을 선택하는 이유는 서버 머신 한대로 서버를 동작시키는건 한계가 있습니다.  
그래서 효율을 생각하면 서버의 성능 업그레이드보다 서버의 갯수를 늘리는게 효율적이기 때문입니다.  
  
그러면 프로세스간 통신을 위해서 프로토콜을 만들고 통신하는 코드를 만드는 작업이 번거롭고 
다른 언어를 사용한다면 패킷 모델을 주고 받는 것도 리소스가 발생합니다.  
  
그래서 Google Protobuf나 Apache Avro를 사용하여 공통 패킷 모듈을 만들 수 있습니다.  
HTTP 통신으로 Json도 가능하지만 필드 추가나 오타로 인한 경우 원인 찾기도 어렵습니다.  

  
## Thread  
![image_153.png](image_153.png)  
  
운영체제가 하나의 프로세스 내애 여러 Thread를 지원하는 것을 다중 쓰레딩(kernel-level multi thread)라고 합니다.  
하나의 사용자가 하나의 프로세스를 실행할 수 있으며, 해당 프로세스 내에 하나의 쓰레드만 동작하게 되었습니다.  
  
사용자가 증가할 수록 매번 프로세스를 실행하고 중지하는 과정은 비효율적이기 때문에 하나의 프로세스에 여러개의 쓰레드를 만들어 동작하는 방향으로 개선했습니다.  
프로세스를 하나 만드는 것보다 쓰레드 하나를 추가하는게 비용이 더 작기때문입니다.  
  
싱글 쓰레드 환경에서 다중 쓰레드 환경으로 변화가 되었고, 프로세스는 자원 할당의 단위로 의미를 갖게 됩니다.  
  
![image_154.png](image_154.png)  
  
멀티 쓰레드 환경이 된 이유는 프로세스 내에 단일 쓰레드 일 경우 연산이 오래걸리는 작업이나, I/O 발생으로 인해
대기를 하는 시간을 줄일 수 있습니다. 프로세스로 이 방식을 구현한다면 쓰레드를 만드는 것보다 더 많은 자원과 시간이 필요하며 
쓰레드간 컨택스트 스위칭 비용이 프로세스간 컨택스트 스위칭 비용보다 더 적습니다.   
  
> 프로세스간 스위칭은 비용이 발생할까  
> 프로세스 마다 메모리 공간이 다르기 때문에 연산에 필요한 자원을 가져오고, 데이터 오류를 방지하기 위해 
> 캐시를 초기화하는 과정이 발생하기 때문입니다.
 
그리고 쓰레드 내부에서는 메모리가 공유되지만, 프로세스는 메모리 공유가 안되어 커널을 개입한 통신을 해야합니다.  
  
프로세스는 메인 쓰레드를 갖고, 해당 쓰레드가 종료하면 프로세스도 종료합니다.   

**메인 쓰레드 로직**
```Java
public static void main(String[] args) throws InterruptedException {
    System.out.println("Process Start!");

    //Thread
    Thread threadA = new Thread(new DoWork("threadA", 1000));
    Thread threadB = new Thread(new DoWork("threadB", 1500));

    threadA.start();
    threadB.start();

    //Thread 종료대기
    threadA.join();
//  threadB.join();

    System.out.println("All fin");
}
```  
**Runable 구현**
```Java
 static class DoWork implements Runnable {

    private final String name;
    private final int millis;

    public DoWork(String name, int millis) {
        this.name = name;
        this.millis = millis;
    }

    @Override
    public void run() {
        System.out.println("Doing work...");
        sleep();
        System.out.println("Work done!!"+name);
    }

    private void sleep() {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```  
  
### start와 join  
메인 쓰레드 스택에 threadA,B start()가 추가되어 실행되면 스택에서 제거됩니다.  
이후 `threadA.join()`을 만나면 `Main thread`는 대기상태가 됩니다.  
`threadA`의 모든 작업이 종료되면 그때부터 `Main thread`의 나머지 작업이 시작됩니다.  
  
```Java
// threadA.join() 의 경우
Process Start!
[threadB ]Doing work... sleep = 1000
[threadA ]Doing work... sleep = 500
[threadA ]Work done !!
theadA가 종료될때까지 MainThread 는 대기상태가 됩니다.
All fin #####
[threadB ]Work done !!
```  
쓰레드A가 종료되야 나머지 코드를 진행합니다.  
만약 `join()`이 없는 경우에는 아래와 같습니다.
```Java
Process Start!
theadA가 종료될때까지 MainThread 는 대기상태가 됩니다.
All fin #####
[threadA ]Doing work... sleep = 500
[threadB ]Doing work... sleep = 1000
[threadA ]Work done !!
[threadB ]Work done !!
```  
  
메인쓰레드는 나머지 쓰레드가 시작되면 이후부터는 자신의 나머지 로직을 모두 수행하고 `threadA,B`가 종료되기까지 기다립니다.  