# Thread-Per-Message
  
## 개념  
각 메세지(명령 또는 요청)마다 새로운 `Thread`를 할당하여 처리하는 방식입니다.  
이 패턴은 멀티쓰레드 환경에서 사용할 수 있으며, 메세지를 의뢰하는 측과 메세지를 수행하는 측이 다른 쓰레드에서 동작합니다.  
  
예를 들면 쿠팡이츠로 도미노에서 피자를 주문하고, 교촌에서 치킨을 주문한다고 하려고 합니다.  
우리는 도미노에서 피자를 주문하고 피자가 배달되기 전까지 기다리지 않고 교총에서 치킨을 주문합니다.  
  
이처럼 주문(`Message`)을 하는 손님(`Main Thread`)와 주문을 준비하는 식당(`Worker Thread`)가 다르다는 의미입니다.  
  
## 구성요소 
#### Client(의뢰자)  
클라이언트는 Host에게 요청합니다. Host가 어떻게 그 요구를 실행하는지 클라이언트는 알지 못합니다.
#### Host  
호스트는 클라이언트의 요청을 받으면 새로운 쓰레드를 새롭게 만듭니다. 
새롭게 만들어진 쓰레드는 `Helper`를 통해서 요구 처리(`Handle`)합니다.

#### Helper(수행자)  
자신이 가진 기능을 수행만 합니다.   
  
정리하면, 클라이언트는 호스트에게 요청하고, 호스트는 새롭게 쓰레드를 만들고, 수행자에게 작업을 위임하고 돌아옵니다.  
이렇게 하다보니 클라이언트와 호스트간의 응답성이 좋아지고, 지연 시간이 줄어듭니다.  
  
## 사용 환경 
### 응답성을 높이고 지연 시간이 줄어들어야 할때
실제 수행하는 작업이 시간이 오래걸리는 경우, I/O 작업으로 인해 대기가 발생하는 경우가 발생합니다. 
다른 요청들은 해당 작업이 끝날때까지 대기해야하는 상황이 발생하지 않아도 될때 사용합니다.

### 처리 순서가 상관 없을 경우  
새로운 쓰레드를 생성하고 수행자에게 작업을 요청한 순서대로 진행되지 않을 수 있습니다. 
따라서 이 패턴은 순서가 의미 없는 작업에 사용해야합니다.

### 결과가 불필요한 경우에 사용한다.  
클라이언트는 본인이 요청한 결과를 기다리지 않습니다. `Handle`의 실행 결과를 `request`측에서 얻을 수 없습니다.  
따라서 처리의 결과가 필요 없는 경우에만 사용하게 됩니다. 
상호작용이 필요없는 이벤트를 통지하는 경우가 해당됩니다.  
  
> 결과가 필요한 경우에는 Future 패턴을 사용합니다.  
>   
  
### 메소드 호출과 새 쓰레드 -> 메세지 송신  
보통의 경우에는 쓰레드를 호출하면 처리가 완료되어야 제어가 돌아옵니다.  
지금 호스트의 메소드를 호출한 것도 보통의 메서드이기 때문에 처리가 완료된 후 돌아옵니다.  
하지만 정말 실행되어야하는 핵심 로직인 `Handle`은 동작이 되었는지 알 수가 없습니다.  
  
호스트 측에서 새 쓰레드를 생성하고 작업을 위임하고 있다는 것을 확장하여 
**_메소드 호출과 쓰레드 기동을 통해 "비동기 메세지 송신"을 실현하고 있습니다_**  
  
넓게 보면 멀티쓰레드판 Proxy 패턴이나 멀티쓰레드의 Adapter 패턴이라고 할 수 있습니다.


## 트레이드 오프
이 패턴을 사용 여부는 `Handle의 처리에 걸리는 시간`과 `쓰레드 기동에 걸리는 시간`과 트레이드 오프가 발생됩니다.  
무조건 새로운 쓰레드를 생성하여 `handle`에 작업을 위임하는 것보다 하나의 쓰레드에서 처리하는게 더 빠를 수 있습니다.  
  
## 코드  
### 클라이언트
```Java
public class MyHouse {
    public static void main(String[] args) {
        System.out.println("==== 배달음식 주문시작 ====");
        Coupang coupang = new Coupang();
        coupang.order("메가커피", 10);
        coupang.order("교촌치킨", 20);
        coupang.order("도미노 피자", 30);
        System.out.println("==== 배달음식 주문종료 ====");
    }
}
```  
### 호스트
```Java
public class Coupang {
    private final Helper helper = new Helper();
    public void order(final String  foodName, final int waitTime) {
        System.out.printf("주문 접수( 예상소요 분: " + waitTime + ", 음식명:" + foodName + " ) BEGIN\n");
        new Thread(() -> helper.handle(waitTime, foodName)).start();
        System.out.printf("주문 접수( 예상소요 분: " + waitTime + ", 음식명:" + foodName + " ) END\n");
    }
}
```
### 수행자
```Java
package org.messageper;

public class Helper {
    public void handle(int waitTime, String foodName) {
        System.out.printf("준비중( " + waitTime + ", " + foodName + " ) BEGIN\n");
        for (int i = 0; i < waitTime; i++) {
            slowly();
            System.out.println(waitTime+"분째 만드는 중 = " + foodName);
        }
        System.out.printf("준비중( " + waitTime + ", " + foodName + " ) END\n");
    }
    private void slowly(){
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```  
### 콘솔
```Java
==== 배달음식 주문시작 ====
주문 접수( 예상소요 분: 10, 음식명:메가커피 ) BEGIN
주문 접수( 예상소요 분: 10, 음식명:메가커피 ) END   --- 메소드 완료
주문 접수( 예상소요 분: 20, 음식명:교촌치킨 ) BEGIN
주문 접수( 예상소요 분: 20, 음식명:교촌치킨 ) END   --- 메소드 완료
주문 접수( 예상소요 분: 30, 음식명:도미노 피자 ) BEGIN
준비중( 10분 소요예정, 메가커피 ) BEGIN  
준비중( 20분 소요예정, 교촌치킨 ) BEGIN
주문 접수( 예상소요 분: 30, 음식명:도미노 피자 ) END --- 메소드 완료
==== 배달음식 주문종료 ====
준비중( 30분 소요예정, 도미노 피자 ) BEGIN
0분째 만드는 중 = 교촌치킨
0분째 만드는 중 = 메가커피
0분째 만드는 중 = 도미노 피자
    :
9분째 만드는 중 = 교촌치킨
9분째 만드는 중 = 메가커피
준비완료( 메가커피 ) END
9분째 만드는 중 = 도미노 피자
    :
18분째 만드는 중 = 도미노 피자
19분째 만드는 중 = 교촌치킨
준비완료( 교촌치킨 ) END
19분째 만드는 중 = 도미노 피자
20분째 만드는 중 = 도미노 피자
    :
28분째 만드는 중 = 도미노 피자
29분째 만드는 중 = 도미노 피자
준비완료( 도미노 피자 ) END
```
  
메소드는 순차적으로 실행했지만 기다리지 않고 바로 다음 주문을 할 수 있습니다. 
메소드 호출과 쓰레드 기동을 통해 `비동기 메세지 송신`을 실현하고 있다는 것을 다시 확인해야합니다.

### 참고자료  
[자바-Thread-Per-Message패턴](https://12bme.tistory.com/379)  
