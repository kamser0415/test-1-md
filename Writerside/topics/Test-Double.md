# Test Double

출처 :[Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)

## Dummy
실제 객체를 모방하기만 하여 아무런 동작도 안하고 행이도 안하는 깡통 객체입니다.  
활용할 일은 잘 없습니다.

## Fake
페이크 객체는 단순한 형태로 동일한 기능을 수행합니다. 
실제 객체와 동일한 기능을 수행하지만 간단한 형태로 수행을 합니다.
그래서 실제 프로덕션에서 사용하기에 기능이 부족한 객체를 말합니다. 
  
예시로 `FakeRepository`가 있습니다.
`Map`처럼 인메모리에서 데이터를 저장하는 방식입니다.
(인메모리 데이터베이스가 좋은 예입니다).

## Stub
주로 테스트 중에 이루어지는 호출에 대해 미리 정해진 답변을 제공하며, 
테스트에 프로그램된 내용 이외에는 대부분 반응하지 않습니다.

## Spy
Spy(스파이)는 호출된 방식에 기반하여 일부 정보를 **기록**하는 `스텁(Stub)`입니다. 
여기에 속하는 한 가지 예시는 메일 서비스로, 
얼마나 많은 메시지가 전송되었는지를 기록할 수 있습니다.  
  
몇번 호출이 되었는지, 타임아웃이 어떻게 되었는지 이런 정보를 기록해서
개발자에게 제공합니다. 실제 객체와 거의 유사하게 동작할 수 있고 
일부만 `Stub`을 할 수 있는 객체라고 생각하면 됩니다.

## Mock
행위에 대한 기대를 정의하고 이 행위를 했을때 어떤 과정을 거칠거야,
어떤 결과가 나올거야 이런 것들을 개발자가 명세하고 그에 따라 동작하도록 
우리가 프로그래밍을 해서 만든 객체를 Mock 이라고 합니다.

## Stub 과 Mock의 차이

> 비슷한 점: 모두 가짜 객체고, 요청 한것에 대해 지정한 답을 리턴해달라는건 비슷

### 검증하려는 목적이 다르다
> `Double` 중에서 `목(Mock)`만이 행위 검증을 요구합니다.   
> 다른 더블들은 주로 `상태 검증`을 사용할 수 있습니다. 
> `Mock`은 실제 협력(의존) 객체와 상호 작용하는 동안 실제 객체와 같이 동작해야 하므로, 
> 연습 단계에서 다른 `Double`과 유사하게 행동하지만   
> 
> **_설정 및 검증 단계에서는 차이가 있습니다._**

Stub 
: 상태 검증(State Verification)
Stub객체를 행위를 호출했을 때, 그로인해 객체에 내부 값들이 변경되는 것을 
객체의 메서드로 확인하여 검증하는 것을 말합니다.  
```Java
public class MailServiceStub implements MailService {
  private List<Message> messages = new ArrayList<Message>();
  public void send (Message msg) {
    messages.add(msg);
  }
  // 객체 메서드로 검증
  public int numberSent() {
    return messages.size();
  }
}
public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    MailServiceStub mailer = new MailServiceStub();
    order.setMailer(mailer);
    order.fill(warehouse);
    //내부 메서드로 객체의 내부 필드나 상태를 검증합니다.
    assertEquals(1, mailer.numberSent());
  }
```
**내부적인 상태 변화에 대해 검증**하려고 하는것.

Mock 
: 행위 검증(Behavior Verification)
`when → send mail → return` 등 행위에 대한 값을 검증한다.
```Java
public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    Mock warehouse = mock(Warehouse.class);
    Mock mailer = mock(MailService.class);
    order.setMailer((MailService) mailer.proxy());
    
    mailer.expects(once()).method("send");
    //우리가 행위에 대한 결과를 지정합니다.
    warehouse.expects(once()).method("hasInventory")
      .withAnyArguments()
      .will(returnValue(false));
    
    order.fill((Warehouse) warehouse.proxy());
}
```

## 정리
Stub은 행위를 호출했을 경우 실제 객체의 내부 필드나 속성이 어떻게 되었는지 검증하는 방식이라면, 
Mock은 행위를 호출할때 우리가 원하는 결과를 반환하게 만드는 것입니다.  

- `상태 검증`이란 메소드가 수행된 후, 객체의 상태를 확인하여 올바르게 동작했는지를 확인하는 검증법입니다.
- `행위 검증`이란 메소드의 리턴 값으로 판단할 수 없는 경우, 특정 동작을 수행하는지 확인하는 검증법입니다.  