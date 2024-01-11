# BusinessLayerTest

요구사항
: 주문 생성을 해야합니다.  
+ 상품 번호 리스트를 받아 주문 생성하기
+ 주문은 주문 상태,주문 등록 시간을 가진다.
+ 주문의 총 금액을 계산할 수 있어야한다.  
  
서비스 계층은 단위테스트와 통합테스트로 나뉩니다.
1. 통합테스트 : 비즈니스 계층 + Persistence 계층
2. 단위테스트 : 비즈니스 계층 + Mock(Persistence)  
  
현재는 통합테스트로 상품 번호를 받아 주문을 저장하고 
주문 결과를 반환을 확인하는 테스트 입니다.  
 
서비스 계층이 얇아도 테스트 코드를 작성합니다
: 기능이 추가됨에 따라 기능의 특성에 맞는 테스트를 추가하는 것이 좋다는 원칙과도 일치합니다. 
새로운 기능이나 변경된 기능이 예상대로 동작하며 시스템 전체에 미치는 영향을 평가하기 위해 
테스트를 수행하는 것이 중요합니다.

## 테스트 코드 작성 순서
이번에는 반대로 컨트롤러(`API`)부터 테스트 코드를 작성하는 방법입니다.  

테스트 코드를 `API Controller`부터 시작하던, `Persistence Layer`부터 시작하던 
진행 방식은 동일합니다.   
  

<procedure title="테스트 작성">
    <step>
        <code>OrderController</code><span>생성 및 API 생성</span>
    </step>
    <step>
        <p>요청사항: 오더번호를 받아 주문을 한다.</p>
        <code>OrderService.createOrder(List-String order)</code><span>생성</span>
    </step>
    <step>
        <code>createOrder(String...)</code><span>테스트 코드 작성</span>
    </step>
</procedure>  
  
### createOrder 테스트 코드 작성
```Java
@DisplayName("주문 번호 리스트를 받아 주문을 생성한다.")
@Test
void createOrder(){
    //given - 테스트를 준비할 때 필요한 필드만 작성해서 문서화 한다.
    Product product1 = createProduct(HANDMADE, "001", 1000);
    Product product2 = createProduct(HANDMADE, "002", 3000);
    Product product3 = createProduct(HANDMADE, "003", 5000);
    productRepository.saveAll(List.of(product1, product2, product3));

    OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of("001", "002"))
            .build();
    //when
    OrderResponse orderResponse = orderService.createOrder(request);

    //then
    Assertions.assertThat(orderResponse.getId()).isNotNull(); // 아이디 보장
    Assertions.assertThat(orderResponse)
            .extracting("registeredDateTime", "totalPrice")
            .contains(LocalDateTime.now(),4000);
    Assertions.assertThat(orderResponse.getProducts())
            .hasSize(2)
            .extracting("productNumber", "price")
            .containsExactlyInAnyOrder(
                    Assertions.tuple("001", 1000),
                    Assertions.tuple("002", 3000)
            );
}
```

#### Given
> `Given`을 작성할 때 테스트 검증에 필요한 데이터만 매개변수로 사용합니다. 
> 그외 필요없는 변수는 내부에 숨겨서 `createOrder`가 어떤 데이터를 참조하여 동작하는지 
> `createOrder`매서드를 직접 들어가지 않도록 작성해야합니다.
```Java
Product product1 = createProduct(HANDMADE, "001", 1000);
Product product2 = createProduct(HANDMADE, "002", 3000);
Product product3 = createProduct(HANDMADE, "003", 5000);

//편의 메서드
private Product createProduct(ProductType productType, String number, int price) {
    return Product.builder()
            .productNumber(number)
            .type(productType)
            .name("메뉴 이름")
            .price(price)
            .sellingStatus(SELLING_TYPE)
            .build();
}
```  
+ 주문 생성시 필요한 `주문번호`를 `Given`작성시 보여줍니다.
+ 금액은 테스트 검증을 육안으로 확인할 수 있는 필드 입니다.   
 
#### when
테스트 코드의 결과를 확인하기 위해서 내부까지 들어가서 확인하는건 올바른 검증 방법이 아닙니다.
`캡슐화`를 제대로 하지 못했다는 의미입니다.  
```Java
OrderResponse orderResponse = orderService.createOrder(request);
```  
`void`로 주문 생성하고 마무리 할 수 있지만, 제대로 주문이 생성되었는지 응답 객체를 클라이언트에게 전달 할 수 있도록 코드를 작성합니다. 
주문에 대한 반환 클래스로 `OrderResponse`를 생성하는데, 내부에 어떤 주문을 했는지 주문에 대한 응답도 포함되었으면 합니다. 
내부 필드에 생성된 오더의 대한 정보와 주문한 음료수 정보도 포함되었으면 합니다.  
`OrderReponse` 객체 내부에 만들어놓은 `ProductResponse`를 포함시킵니다.  

[//]: # ()
[//]: # (> 현재 코드는 N+1 문제가 발생합니다.  )

[//]: # (```Java)

[//]: # (// OrderResponse 객체 내부에서 order.getProduct&#40;&#41;를 호출하면 N+1 문제가 발생할 수 있습니다)

[//]: # (public static OrderResponse of&#40;Order order&#41; {)

[//]: # (    return OrderResponse.builder&#40;&#41;)

[//]: # (            .id&#40;order.getId&#40;&#41;&#41;)

[//]: # (            .totalPrice&#40;order.getTotalPrice&#40;&#41;&#41;)

[//]: # (            .registeredDateTime&#40;order.getRegisteredDateTime&#40;&#41;&#41;)

[//]: # (            .products&#40;order.getOrderProducts&#40;&#41;.stream&#40;&#41;)

[//]: # (                    // 이 부분에서 N+1 문제가 발생하기 때문에 간접참조를 사용해야합니다.)

[//]: # (                    .map&#40;orderProduct -> ProductResponse.of&#40;orderProduct.getProduct&#40;&#41;&#41;&#41;)

[//]: # (                    .collect&#40;Collectors.toList&#40;&#41;&#41;&#41;)

[//]: # (            .build&#40;&#41;;)

[//]: # (})

[//]: # (```)
  
#### then
`createOrder()`코드를 작성하지 않고, createOrder 가 동작할 경우 
결과를 예측하면서 테스트 코드를 작성합니다.  
  
```Java
//OrderService
public OrderResponse createOrder(OrderCreateServiceRequest request, LocalDateTime registeredDateTime) {
    return null;
}
```  
### TDD  
TDD로 createOrder 작성하기
: 
1.`OrderService` 반환으로 response 객체 생성  
2.`OrderResponse`에 필요한 필드를 생성  
3.`OrderResponse`에 상품 정보 반환값도 포함시킨다.    
    ```Java
    @Getter
    public class OrderResponse {
        private Long id;
        private int totalPrice;
        private LocalDateTime registeredDateTime;
        private List<ProductResponse> products;
        
        //이하 생략
    }
    ```  
4.`createOrder` 시나리오 작성   
현재 코드는 `null`을 반환하므로 빠르게 `green`을 보도록 코드를 작성하는데, 
테스트의 방향이 흔들릴 수 있으므로 시나리오를 생각합니다.
<procedure title="주문이 들어올 경우 동작방식">
    <step>
        <p>상품 번호로 주문할 상품을 찾습니다.</p>
        <code>ProductRepository</code>를 주입받아서 조회를 해야합니다.
        <code-block>
        //JpaRepository
        List[Product] findAllByProductNumberIn(...);
        </code-block>  
        <p>검증안된 <code>Persistence Layer</code>메서드를 테스트합니다.</p>
        <code-block>
        //{주어진 인수}로 {엔티티}를 {SELECT}한다
        @DisplayName("상품번호 리스트로 상품들을 조회한다.")
        </code-block>
    </step>
    <step>
        <p>찾은 상품 번호로 주문을 생성합니다. 상품리스트로 주문 엔티티를 만드는 메서드를 만듭니다.</p>
        <code-block>
        public static Order create(List products){
            return new Order(products);
        }
        public Order(List products){
            //주어진 상품으로 주문을 생성합니다.
            this.orderStatus = OrderStatus.INIT;
            return;;
        }
        </code-block>
    </step>
    <step>
        <p><code>this.orderStatus = OrderStatus.INIT;</code> 단위테스트를 작성합니다.</p>
        <p>Order 엔티티를 생성할때 초기화 하는 값이 제대로 들어갔는지 단위 테스트를 해야합니다.</p>
        <code-block>
        public Order(List-Product products, LocalDateTime registeredDateTime) {
        this.orderStatus = OrderStatus.INIT;  //
        this.totalPrice = calculateTotalPrice(products);
        this.registeredDateTime = registeredDateTime;
        this.orderProducts = products.stream()
                .map(product -> new OrderProduct(this, product))
                .toList();
        }
        </code-block>  
        <ul>
            <li><code>orderStatus</code> 단위테스트 작성</li>
            <li><code>totalPrice</code>단위테스트 작성</li>
            <li><code>registeredDateTime</code>단위테스트 작성</li>
        </ul>
        <p>registeredDateTime을 테스트하기 어려워서 외부에서 주입 받는 방식으로 테스트를 진행합니다.</p>
    </step>
    <step>
        <p>registeredDateTime을 외부로 분리하여 테스트 진행</p>
        <code-block>
         @DisplayName("주문번호 리스트를 받아 주문을 생성한다.")
    @Test
    void createOrder() {
        // given
        // 외부에서 주입을 한다.
        LocalDateTime registeredDateTime = LocalDateTime.now();
        // when
        OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);
        </code-block>
    </step>
</procedure>

## 단위 테스트는 어디까지 작성해야할까?
데이터 계층은 유요한 데이터로만 테스트를 진행한다.
: 데이터에 엑세스 한다는 행위는 그 이전에 어딘가에서 데이터를 저장할 때 유요한 데이터들을 
필터링하거나 비즈니스 요구사항에 맞게 정제하여 저장했다는 것을 전체로 합니다. 
데이터베이스에 유효하지 않은 값들이 저장되어있다면 데이터를 신뢰할 수 없는 데이터 셋이기 때문입니다.  

> 테스트 코드를 작성시 모든 시나리오를 가정하고 테스트 코드를 작성할 수 있지만, 
> 언제나 허락된 시간과 비용 안에서 최대의 효율을 가져가면서 테스트를 선택해야하기 때문입니다.  
  
findAll,save등도 테스트를 해야할까?  
: 기본으로 제공되는 메서드는 검증 완료된 라이브러리 기능이기 때문에 굳이 비용들여 테스트를 할 필요가 없습니다.

  
<procedure title="도메인 정책상 변하는 값은 테스트를 꼭한다.">
    <step>
        <p> 도메인 정책 메소드</p>
        <code-block>
            @Getter
            @RequiredArgsConstructor
            public enum ProductType {
                HANDMADE("제조 음료"),
                BOTTLE("병 음료"),
                BAKERY("베이커리");
                private final String text;
                public static boolean containsStockType(ProductType productType) {
                    return List.of(BOTTLE, BAKERY).contains(productType);
                }
            }
        </code-block>
    </step>
    <step>
        <p>테스트 코드</p>
        <code-block>
            @DisplayName("상품 타입이 재고 관련 타입인지 체크한다.")
            @Test
            void containsStockType1(){
                //given
                ProductType handmade = ProductType.HANDMADE;
                //when
                boolean result = ProductType.containsStockType(handmade);
                //then
                Assertions.assertThat(result).isFalse();
            }
        </code-block>
    </step>
</procedure>
  

재고 관리 대상인 BOTTLE 과 BAKERY를 테스트하는 이유  
: 테스트를 통해 코드의 각 부분이 예상대로 작동하며 예외 상황에 대비할 수 있는지 
확인할 수 있습니다. 또한 코드 변경이나 업데이트가 있을 때 기존 기능들이 
영향을 받지 않도록 보장하고, 새로운 기능이나 수정된 기능이 기대한 대로 동작하는지 
확인할 수 있습니다.

간단한 코드

## 테스트 작성
### 중복회원
```Java
@DisplayName("중복되는 상품번호 리스트로 주문을 생성할 수 있다.")
@Test
void createOrderWithDuplicateProductNumbers() {}
```  
**GREEN**
```Java
public OrderResponse createOrder(OrderCreateRequest request, LocalDateTime registeredDateTime) {
    List<String> productNumbers = request.getProductNumbers();
    //product
    List<Product> products = productRepository.findAllByProductNumberIn(productNumbers);
    
    Map<String, Product> productMap = products.stream()
            .collect(Collectors.toMap(it -> it.getProductNumber(), it -> it));

    List<Product> collect = productNumbers.stream()
            .map(it -> productMap.get(it))
            .collect(Collectors.toList());

    // order - order의 생성자에서 OrderProduct를 같이 만든다.
    // 이거로 오더를 만든다.
    Order order = Order.create(collect, registeredDateTime);// 단위테스트 모두 종료
    Long saveId = orderRepository.save(order);
    Order findOrder = orderRepository.findById(saveId);
    //트랜잭션 없이 동작하려면


    return OrderResponse.of(findOrder);
}
```
**리팩토링**   
> 해당 코드 영역을 하나의 단위로 생각하고 별도의 메서드로 추출합니다.
```Java
private List<Product> findProductsBy(List<String> productNumbers) {
    List<Product> products = productRepository.findAllByProductNumberIn(productNumbers);
    Map<String, Product> productMap = products.stream()
            .collect(Collectors.toMap(it -> it.getProductNumber(), it -> it));

    List<Product> collect = productNumbers.stream()
            .map(it -> productMap.get(it))
            .collect(Collectors.toList());
    
    return collect;
}
```    
  
### 비즈니스 예외와 도메인 예외
도메인
```Java
//Stock entity
public void deductQuantity(int quantity) {
    if (isQuantityLessThan(quantity)) {
        throw new IllegalArgumentException("재고가 부족합니다.");
    }
    this.quantity -= quantity;
}
```
서비스
```Java
//OrderService
private void deductStockQuantities(List<Product> products) {
    // 재고 차감 시도
    for (String stockProductNumber : new HashSet<>(stockProductNumbers)) {
        Stock stock = stockMap.get(stockProductNumber);
        int quantity = productCounting.get(stockProductNumber).intValue();
        if (stock.isQuantityLessThan(quantity)) {
            throw new IllegalArgumentException("재고가 부족한 상품이 있습니다.");
        }
        stock.deductQuantity(quantity);
    }
}
```
도메인에도 검증 로직이 있고, 서비스에도 검증 로직이 있습니다. 
엔티티는 데이터베이스에 저장되기 때문에 검증로직에 대한 예외는 비즈니스에 대한 내용이 없습니다. 
  
서비스에 있는 검증은 엔티티에 데이터를 넣기전 해당 비즈니스 정책에 대한 메세지를 전달할 수 있습니다.  
  
중복검증을 할 필요가 있을까?
: 엔티티는 다른 비즈니스 계층에서 사용할 수 있습니다. 
그때 비즈니스마다 예외 메세지가 다를 수 있고, 비즈니스에 종속된 메세지이기 때문에 
중복되게 작성하는 방향이 있습니다.  