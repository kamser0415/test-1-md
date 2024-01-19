# POJO 상품 등록 기능 구현하기
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
  
**Product,ProductPort 완성**
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

**Port 구현체 Adapter,Service 생성자 초기화**
```Java
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

// 익명클래스로 Port 구현체를 만들고나서 구현체를 생성합니다.
@BeforeEach
void setUp() {
    productPort = new ProductPort() {
        @Override
        public void save(final Product product) {
            //상품 저장
        }
    };
    productService = new ProductService(productPort);
}
```
서비스가 특정 인터페이스를 가진 포트를 의존합니다.
포트의 구현체는 데이터를 저장하는 기능을 가진 `Repository` 역할의 포트입니다.

## 전체코드
```Java
package org.example.productorderservice.product;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.util.Assert;

import java.util.HashMap;
import java.util.Map;

public class ProductServiceTest6 {

    private ProductService productService;
    private ProductPort productPort;
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository = new ProductRepository();
        productPort = new productAdapter(productRepository);
        productService = new ProductService(productPort);
    }

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

    @Test
    void 상품등록(){
        final String name = "상품명";
        final int price = 1000;
        final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
        productService.addProduct(request);
    }

    private record AddProductRequest(String name, int price, DiscountPolicy discountPolicy) {
            private AddProductRequest(final String name, final int price, final DiscountPolicy discountPolicy) {
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

    private class Product {

        private final String name;
        private final int price;
        private final DiscountPolicy discountPolicy;
        private Long id;

        public Product(final String name, final int price, final DiscountPolicy discountPolicy) {
            //검증
            Assert.hasText(name, "상품명은 필수입니다.");
            Assert.isTrue(price > 0, "가격은 0원보다 커야합니다.");
            Assert.notNull(discountPolicy, "할인정책은 필수입니다.");

            this.name = name;
            this.price = price;
            this.discountPolicy = discountPolicy;
        }

        public void assignId(final Long id) {
            //검증
            this.id = id;
        }

        public Long getId() {
            return id;
        }
    }

    private interface ProductPort {
        void save(Product product);
    }

    private class productAdapter implements ProductPort {

        private final ProductRepository productRepository;

        private productAdapter(ProductRepository productRepository) {
            this.productRepository = productRepository;
        }

        @Override
        public void save(final Product product) {
            //상품 저장
            productRepository.save(product);
        }
    }

    private class ProductRepository {

        private static Long sequence = 0L;
        private Map<Long,Product> persistence = new HashMap<>();

        public void save(final Product product) {
            //상품 저장
            product.assignId(++sequence);
            persistence.put(product.getId(),product);
        }
    }
}
```
전체 코드의 작성 순서는 아래와 같습니다.
: 
1. 매서드 시그니처 작성
2. 매서그 구현부는 이미 객체가 있다 생각하고 객체의 메서드와 인수까지 전달을 작성
3. 해당 인수가 없는 DTO라면 작성하고, 객체가 없다면 객체와 메서드도 작성합니다.
4. 메서드의 내용을 완성하지 않아도 컴파일이 가능하다면 테스트를 돌립니다.
5. 그리고 구현부에서 해당 클래스의 기능을 의존하기 때문에 구성,상속 두 방법중 하나를 선택해야합니다.
6. 상속으로 해당 객체를 확장하는게 아니라면 상속으로 처리합니다.
7. 상속은 필드에 해당 객체가 있어야하고, 의존하는 객체는 생성자에 노출시켜 의존성을 드러냅니다.
8. `@BeforeEach`로 컨테이너가 동작하듯이 객체의 초기화는 클래스내에서 이루어지지 않게 합니다.
  
## 정리
리포지토리를 완성하고, 서비스를 통합테스트하고, 컨트롤러 검증까지 테스트하는 방식과 
반대 방향으로 외부 요청부터 내부 어댑터까지 들어오는 방식의 테스트는 장단점이 있습니다.  
  
외부 요청부터 들어오는 방식으로 코드를 작성하면서 테스트를 진행할 경우에는 
외부 요청에 대한 필요한 매서드와 클래스만 생성하고 테스트를 진행하면서 내부 로직으로 들어오기 때문에
테스트 코드의 예외 케이스나, 해피케이스도 생각하면서 테스트 케이스를 작성할 수 있다는 장점이 있습니다.

## 스프링 부트 테스트로 전환하기

해당 클래스를 리포지토리, 어댑터, 포트, 서비스 순서대로 `main` 디렉토리에 클래스를 옴깁니다.
어댑터,서비스에는 `@Component`를 사용하고, 리포지토리에는 `@Repository`를 적용하여 컴포넌트 스캔 대상이 되도록 합니다.

**테스트 코드 수정 후**
```Java
@SpringBootTest
public class ProductServiceTest6 {

    @Autowired
    private ProductService productService;

    @Test
    void 상품등록(){
        final String name = "상품명";
        final int price = 1000;
        final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
        productService.addProduct(request);
    }

}
```
요청 객체도 외부로 분리합니다.  
```Java
@SpringBootTest
public class ProductServiceTest6 {

    @Autowired
    private ProductService productService;

    @Test
    void 상품등록(){
        productService.addProduct(createProductRequest());
    }

    private static AddProductRequest createProductRequest() {
        final String name = "상품명";
        final int price = 1000;
        final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
        return request;
    }

}
```  
보기편한 코드가 되었습니다.