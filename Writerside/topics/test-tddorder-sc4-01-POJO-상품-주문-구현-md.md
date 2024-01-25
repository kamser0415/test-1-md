# 응용 테스트 방법

## RED
테스트 코드를 먼저 작성합니다.  
```Java
@Test
void 상품주문(){

    final Long productId = 1L;
    final int quantity = 2; //이미 주문하고 있다.

    final CreateOrderRequest request = new CreateOrderRequest(productId,quantity);
    orderService.createOrder(request);
    
}
```  
> `orderService.createOrder(request);`부터 작성하여 시작합니다.  
>   

클라이언트는 주문 인터페이스를 사용하기 위해서 상품의 고유번호, 주문 수량을 서비스에게 전달하면 
주문 서비스는 주문을 생성합니다.
  
+ request를 이너 클래스로 생성
+ orderService를 이너 클래스로 생성
+ orderService.createOrder 매서드 생성 후 테스트 실행
+ **RED 발생**

### 전체 코드
```Java
public class OrderServiceTest {
    private OrderService orderService;

    @BeforeEach
    void setUp(){
        orderService = new OrderService();
    }

    private class OrderService {
        public void createOrder(final CreateOrderRequest request) {
            throw new RuntimeException("아직 구현되지 않았습니다.");
        }
    }

    @Test
    void 상품주문(){

        final Long productId = 1L;
        final int quantity = 2; //이미 주문하고 있다.

        final CreateOrderRequest request = new CreateOrderRequest(productId,quantity);
        orderService.createOrder(request);
        
    }
    
    private record CreateOrderRequest(Long productId, int quantity) {

        private CreateOrderRequest {
            Assert.notNull(productId, "상품 ID는 필수입니다.");
            Assert.isTrue(quantity > 0, "수량은 0보다 커야합니다.");
        }
    }
}
```
OrderService.createOrder
: 내부 순서
1. 포트에게 해당 번호의 상품을 가져오라고 합니다
2. 해당 상품으로 오더를 만듭니다.
3. 포트에게 오더를 저장하라고 합니다.
  
## 주문 생성 작성
```Java
private class OrderService {
    public void createOrder(final CreateOrderRequest request) {
        //상품 조회
        //== 강사님
        //orderPort.getProductById(request.productId());
        
        productPort.getProduct(request.productId());
        //오더 생성
        final Order order = new Order(request.productId(), request.quantity());
        //오더 저장
        orderPort.save(order);
    }
}
```
ProductPort 의존할 경우
: 
1. OrderService가 ProductPort를 의존하려면 ProductPort는 `public` 접근제어자를 가져야한다.
2. ProductPort를 의존함으로 해당 포트가 가진 메서드만 Stubing하여 테스트 코드가 간결해진다.  
3. package-private이 깨지게 되어 접근 범위가 넓어진다.  
  
OrderPort로 다 해결할 경우
: 
1. ProductPort를 의존하지 않기 때문에 같은 패키지내에서 package-private이 보장된다.
2. OrderPort와 ProductPort와 중복되는 코드가 발생한다.

> 강의와 다르게 productPort를 의존하도록 해보겠습니다.  
>   

```Java
public class OrderServiceTest {

    private OrderService orderService;
    private ProductPort productPort;
    private OrderPort orderPort;
    private OrderRepository orderRepository;

    @BeforeEach
    void setUp(){
        productPort = new StubProductPort();
        orderRepository = new OrderRepository();
        orderPort = new OrderAdapter(orderRepository);
        orderService = new OrderService(productPort, orderPort);
    }

    private static class StubProductPort implements ProductPort {
        @Override
        public void save(final Product product) {}
        @Override
        public Product getProduct(final Long productId) {
            return new Product("상품명", 1000, DiscountPolicy.NONE);
        }
    }

    private static class OrderAdapter implements OrderPort {

        private final OrderRepository orderRepository;

        private OrderAdapter(OrderRepository orderRepository) {
            this.orderRepository = orderRepository;
        }

        @Override
        public Long save(final Order order) {
            return orderRepository.save(order);
        }
    }

    private static class OrderRepository {
        private Long sequence = 0L;
        private final Map<Long, Order> persistence = new HashMap<>();

        public Long save(final Order order) {
            order.assignId(++sequence);
            persistence.put(order.getId(), order);
            return order.getId();
        }
    }

    private class OrderService {

        private final ProductPort productPort;
        private final OrderPort orderPort;

        private OrderService(ProductPort productPort, OrderPort orderPort) {
            this.productPort = productPort;
            this.orderPort = orderPort;
        }

        public Long createOrder(final CreateOrderRequest request) {
            //상품 조회
            final Product product = productPort.getProduct(request.productId());

            //오더 생성
            final Order order = new Order(product, request.quantity());

            //오더 저장
            return orderPort.save(order);
        }
    }

    @Test
    @DisplayName("주문생성")
    void 주문생성(){
        //given
        final Long productId = 1L;
        final int quantity = 2;
        final CreateOrderRequest request = new CreateOrderRequest(productId,quantity);

        //when
        Long orderid = orderService.createOrder(request);

        //then
        assertThat(orderid).isEqualTo(1L);
    }
    
    private record CreateOrderRequest(Long productId, int quantity) {
        private CreateOrderRequest {
            //validation
            Assert.notNull(productId, "상품 ID는 필수입니다.");
            Assert.isTrue(quantity > 0, "상품 수량은 0보다 커야합니다.");
        }

    }

    private static class Order {

        private final Product productId;
        private final int quantity;
        private Long id;

        public Order(final Product productId, final int quantity) {
            //도메인 검증
            Assert.notNull(productId, "상품 ID는 필수입니다.");
            Assert.isTrue(quantity > 0, "상품 수량은 0보다 커야합니다.");

            this.productId = productId;
            this.quantity = quantity;
        }

        public void assignId(final Long orderId) {
            this.id = orderId;
        }

        public Long getId() {
            return id;
        }
    }

    private interface OrderPort {
        Long save(Order order);
    }
}
```

## 강사님 전체 POJO 코드
```Java
public class OrderServiceTest {
    private OrderService orderService;
    private OrderPort orderPort;

    private OrderRepository orderRepository;
    //클라이언트 입장에서 TDD 시작합니다.

    @BeforeEach
    void setUp() {
        orderRepository = new OrderRepository();


        //구현된 어댑터를 사용하면 JpaRepository 의 메서드를 구현해야하기 때문입니다.
        orderPort = new OrderPort() {
            @Override
            public Product getProductById(final Long productId) {
                return new Product("상품", 1000, DiscountPolicy.NONE);
            }

            @Override
            public void save(final Order order) {
                orderRepository.save(order);
            }
        };
        orderService = new OrderService(orderPort);
    }

    private class OrderService {
        private final OrderPort orderPort;

        private OrderService(OrderPort orderPort) {
            this.orderPort = orderPort;
        }

        public void createOrder(final CreateOrderRequest request) {
            // 상품 조회
            Product product = orderPort.getProductById(request.productId());
            
            // 오더 생성
            final Order order = new Order(product, request.quantity());

            // 오더 저장
            orderPort.save(order);
        }
    }

    @Test
    void 상품주문() {

        final Long productId = 1L;
        final int quantity = 2; //이미 주문하고 있다.

        final CreateOrderRequest request = new CreateOrderRequest(productId, quantity);
        orderService.createOrder(request);
    }

    private record CreateOrderRequest(Long productId, int quantity) {

        private CreateOrderRequest {
            Assert.notNull(productId, "상품 ID는 필수입니다.");
            Assert.isTrue(quantity > 0, "수량은 0보다 커야합니다.");
        }
    }

    private interface OrderPort {
        Product getProductById(Long productId);

        void save(Order order);
    }

    private class Order {
        private final Long productId;
        private final int quantity;
        private Long id;

        public Order(final Long productId, final int quantity) {
            //validation
            Assert.notNull(productId, "상품 ID는 필수입니다.");
            Assert.isTrue(quantity > 0, "수량은 0보다 커야합니다.");

            this.productId = productId;
            this.quantity = quantity;
        }

        public void assignId(final Long productId) {
            this.id = productId;
        }

        public Long getId() {
            return id;
        }

    }

    private class OrderAdapter implements OrderPort {

        private final ProductRepository productRepository;
        private final OrderRepository orderRepository;

        private OrderAdapter(ProductRepository productRepository, OrderRepository orderRepository) {
            this.productRepository = productRepository;
            this.orderRepository = orderRepository;
        }

        @Override
        public Product getProductById(final Long productId) {
            return productRepository.findById(productId).orElseThrow(
                    () -> new IllegalArgumentException("상품이 존재하지 않습니다."));
        }

        @Override
        public void save(final Order order) {
            orderRepository.save(order);
        }
    }

    private class OrderRepository {
        private Long sequence = 0L;
        private Map<Long, Order> persistence = new HashMap<>();

        public void save(final Order order) {
            order.assignId(++sequence);
            persistence.put(order.getId(),order);
        }
    }
}
```  

## 정리  
테스트 코드를 먼저 작성하다보니 데이터를 저장한 다음 제대로 저장했는지 확인하려면 테스트 코드에서 
Port,Service,Repository에 아직 데이터 저장과 관련없는 별도의 메서드를 추가로 작성해야합니다.  
  
차라리 저장된지 확인하기 위해 저장된 OrderId라도 반환하면 검증이 편할거같아서 수정했습니다.

처음부터 public으로 접근제어를 모두 열어 놓는게 아니라 , 코드를 작성하면서 
public으로 변경해야하는 애들만 변경하도록해서 private을 유지하도록 합니다.