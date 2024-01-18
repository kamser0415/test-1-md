# 상품 등록 기능 구현하기
> 상품 -> 주문 -> 결제
  
상품
: 상품을 등록할 때는 이름, 가격, 할인 정책이 필요하며,
상품을 등록 수정 삭제하는 기능이 있다.
  
주문 
: 상품과 주문 수량으로 주문을 할 수 있다.
주문을 저장, 수정, 삭제하는 기능이 있다.

결제
: 주문과 카드번호로 결제를 할 수 있다.
결제를 저장, 수정, 삭제하는 기능이 있다.


## 부제: 순수 자바로 요구사항 테스트하기  
상품 등록 비즈니스 로직
: 상품의 이름, 가격, 할인 정책을 서비스에 전달하면 
서비스는 `Port`가 원하는 `DTO`로 변환후 `Port`에게 상품을 저장을 요청합니다.  
`Port`는 인터페이스로 구현체인 `Adapter`가 실제 저장 로직을 수행하게 되며
`Adapter`는 리포지토리 처럼 `DB`에 접근하는 라이브러리를 의존하여 데이터를 저장한다.

## 1. ServiceTest 작성
```Java
@Test
void 상품등록(){
    productService.addProduct(request);
}
```
클라이언트가 요청할 때 전달하는 정보와 어떤 서비스에게 요청하는지부터 시작합니다.  
이렇게 코드를 작성하면 불필요한 클래스나 메서드를 만들지 않고, 클라이언트가 요청할 때 
필요한 메서드와 클래스만 생성할 수 있습니다.  
  
## 2. 요청 객체 완성
```Java
@Test
void 상품등록(){
    final String name = "상품명";
    final int price = 1000;
    final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
    productService.addProduct(request);
}

private static class AddProductRequest {
    private final String name;
    private final int price;
    private final DiscountPolicy discountPolicy;

    public AddProductRequest(final String name, final int price, final DiscountPolicy discountPolicy) {
        this.name = name;
        this.price = price;
        this.discountPolicy = discountPolicy;
        //검증
        Assert.hasText(name, "상품명은 필수입니다.");
        Assert.isTrue(price > 0, "가격은 0원보다 커야합니다.");
        Assert.notNull(discountPolicy, "할인정책은 필수입니다.");
    }
}

private enum DiscountPolicy {
    NONE
}
```  
서비스 메서드에 전달하는 클라이언트 요청 객체를 코드로 작성합니다.

## 3.서비스 구현체 완성
```Java
private ProductService productService;

private class ProductService {
    public void addProduct(final AddProductRequest request) {
        throw new RuntimeException("미구현");
    }
}

@Test
void 상품등록(){
    final String name = "상품명";
    final int price = 1000;
    final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
    productService.addProduct(request);
}

@BeforeEach
void setUp(){
    productService = new ProductService();
}
```
실행하면, `addProduct` 매서드까지 실행하여 테스트 코드는 실패합니다.  

## 4. 서비스 메서드 구현부 완성
```Java
public void addProduct(final AddProductRequest request) {
    final Product product = new Product(request.name(), request.price(), request.discountPolicy());
    productPort.save(product);
}
```  
현재 Product , ProductPort의 구현체 없이 먼저 테스트 코드를 작성하고 있습니다.  
  
**Product,Product 완성**
```Java
private ProductPort productPort;

private class ProductService {
    private final ProductPort productPort;

    private ProductService(ProductPort productPort) {
        this.productPort = productPort;
    }

    public void addProduct(final AddProductRequest request) {
        final Product product = new Product(request.name(), request.price(), request.discountPolicy());
        productPort.save(product);
    }
}

private class Product {

    private final String name;
    private final int price;
    private final DiscountPolicy discountPolicy;

    public Product(final String name, final int price, final DiscountPolicy discountPolicy) {
        //검증
        Assert.hasText(name, "상품명은 필수입니다.");
        Assert.isTrue(price > 0, "가격은 0원보다 커야합니다.");
        Assert.notNull(discountPolicy, "할인정책은 필수입니다.");
        
        this.name = name;
        this.price = price;
        this.discountPolicy = discountPolicy;
    }

}

private interface ProductPort {
    void save(Product product);
}
```  
서비스 구현체는 `Port`에게 저장 기능을 위임하는데 `Port`구현체를 직접생성해야하지만 
`DI`를 통해서 외부에서 주입을 해서 해결합니다. 만약 `DI`를 안했다면 구현체를 직접 호출하게되고 
사용하는 메서드는 동일할 지라도 외부에서 서비스 로직이 의존하는 `Port`를 수정하지 못하고, 어떤걸 의존하는지 알 수 없습니다.  
  
