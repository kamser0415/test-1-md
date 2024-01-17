# Mockito로 Subbing하기

> **목표** : `Mocking`을 언제,어떻게 사용해야하는지 배우기  

## 구현 기능
: 사용자가 메일 전송 버튼을 누르면 기간내 매출을 전송해준다.
+ 가정 : `Order`가 생성되고 고객이 결제를 하고 결제 완료라는 상태로 
데이터가 저장되어있다.  
+ 미흡 : 원래대로라면 결제 완료 시간을 기록하는 필드가 있어야 한다.


```Java
@Getter
@RequiredArgsConstructor
public enum OrderStatus {
    // 이하 상태 Enum 생략
    PAYMENT_COMPLETED("결제완료"),
       
    private final String text;
}
```
### 시나리오
```Java
@Service
@RequiredArgsConstructor
public class OrderStatisticsService {

    public void sendOrderStatisticsMail(LocalDateTime orderDate,String email) {
        // 해당 일자에 결제 완료된 주문들을 가져와서
        // 총 매출 합계를 계산하고
        // 메일 전송
    }
}
```  
## 리포지토리 테스트
```Java
@DisplayName("주문 중에서 전달된 주문 상태와 시작시간이상부터 종료시간미만으로 주문을 조회한다.")
@Test
void findOrdersBy(){
    //given
    LocalDateTime startDateTime = LocalDateTime.of(2024, 1, 12, 10, 0, 0);
    LocalDateTime endDateTime = LocalDateTime.of(2024, 1, 12, 22, 0, 0);
    Product product1 = createProduct(HANDMADE, "001", 1000);
    Product product2 = createProduct(HANDMADE, "002", 3000);
    Product product3 = createProduct(HANDMADE, "003", 5000);
    productRepository.saveAll(List.of(product1, product2, product3));

    OrderCreateRequest requestByStart = createOrder( "002");
    OrderCreateRequest requestByEnd = createOrder("003");
    OrderResponse orderResponse1 = orderService.createOrder(requestByStart, startDateTime);//제대로 동작한지 응답이 보고싶다.
    OrderResponse orderResponse2 = orderService.createOrder(requestByEnd,endDateTime);//제대로 동작한지 응답이 보고싶다.

    //when
    List<Order> findOrders = jpaOrderRepository2.findOrdersBy(startDateTime, endDateTime, OrderStatus.INIT);

    //then
    assertThat(findOrders).size().isOne();
    assertThat(findOrders).extracting("id","registeredDateTime")
            .contains(
                    Assertions.tuple(orderResponse1.getId(),startDateTime)
            );
}
```  
  
경계성 예외 테스트 
: `A <= TIME AND TIME < B`로 범위를 테스트 해야한다면,  
1. `A-1` 포함이 안된다.
2. `A`포함이 된다. (=)
3. `B-1`는 포함이 된다. (<)
4. `B`가 포함이 안된다 .  
경계값을 테스트 해야합니다.  
  
## 외부 네트워크(메일 전송)
```Java
public boolean sendOrderStatisticsMail(LocalDate orderDate, String email) {
    // 회원 조회
    boolean result = mailService.sendMail()
    return true;
}
```
메일 전송을 하는 외부 네트워크 기술이 비즈니스 로직에 추가가 됩니다.  
통합 테스트를 진행한다면 메일 전송 객체 때문에 테스트를 진행할 때마다 외부 네트워크를 
이용하게 됩니다.

## Mocking(Stubbing)
Mock에 대한 행위를 우리가 원하는 행위로 정의하고 그걸 `Stubbing` 이라고 합니다.  
  
테스트 실행마다 외부 네트워크를 사용하면서 발생하는 소요 시간과 비용을 낭비하지 않기 위해서 
가짜 외부 네트워크 객체를 등록합니다.  

통합테스트시 특정 행위시 같은 결과만 반환하는 가짜 외부 네트워크 객체를 만들어 컨테이너에 등록합니다.

## Stubbing 테스트
```Java
@SpringBootTest
class OrderStatisticsServiceTest {

    @Autowired
    private OrderStatisticsService orderStatisticsService;

    @MockBean
    MailSendClient mailSendClient;

}
```
외부 네트워크 객체를 `MockBean`으로 등록합니다.  
```Java
Mockito.when(mailSendClient.sendEmail(ArgumentMatchers.anyString(),ArgumentMatchers.anyString(),ArgumentMatchers.anyString(),ArgumentMatchers.anyString()))
                .thenReturn(true);
```
`when` 인수로 테스트에 수행할 Mock객체의 행위를 지정합니다. 
이때 어떤 인수가 들어올지 모르기 때문에 `ArgumentMatchers`를 이용해서 동작할 수 있습니다.  
  
```Java
//when
boolean result = orderStatisticsService.sendOrderStatisticsMail(LocalDate.of(2024, 1, 12), "roro@naver.com");
```  
테스트 코드를 실행하면, `Mock`객체는 등록된 행위는 항상 우리가 지정한 결과를 반환합니다.  
  
외부 네트워크를 이용하는 로직을 테스트 코드를 동작하면서 발생하는 시간과 비용을 `Mock`으로 `Stubbing`으로 
대체할 수 있습니다.
  
## 테스트 코드 반환이 Boolean
`sendOrderStatisticsMail` 테스트 코드의 결과가 `Boolean` 이라서
다른 `createOrder`처럼 결과를 오브젝트로 반환하지 않는 경우,
내부 로직으로 히스토리에 저장하는 로직이 포함되어있는 경우에는 
데이터 베이스를 직접 조회해서 테스트 하는 방식으로 진행해야합니다.
```Java
List<MailSendHistory> histories = mailSendHistoryRepository.findAll();
        assertThat(histories).hasSize(1)
            .extracting("content")
            .contains("총 매출 합계는 12000원입니다.");
```
