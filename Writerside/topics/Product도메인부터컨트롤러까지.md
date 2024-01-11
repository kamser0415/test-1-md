# Product 도메인부터 컨트롤러까지

## 요구사항
요구사항
: + 티오스크 주문을 위한 상품 후보 리스트 조회하기 API
+ 상품의 판매 상태: 판매중, 판매 보류,판매 중지
    + 판매중, 판매보류중인 상태의 상품을 화면에 보여준다.
+ ID,상품번호,상품타입,판매상태,상품이름,가격 (API 정의)

## 환경 설정
### 패키지 구조
[설명](https://www.inflearn.com/questions/862350)

기존 구조
: `api/controller/domain`
`api/service/domain`


변경 구조
: `api/domain/service`
`api/domain/controller`

>  이유
>  애플리케이션이 확장되면 계층별로 도메인을 나누는 것보다 도메인을 기준으로 계층을 나누는게
> 유지보수하기 더 쉽기 때문입니다.

#### 도식화 { collapsible="true"}
```text
sample.cafekiosk.spring
│
├── api
│   ├── product
│   │   ├── controller
│   │   │   └── ProductController
│   │   └── service
│   │       ├── ProductResponse
│   │       └── ProductService
│
└── domain
    ├── base
    │   ├── BaseEntityAdmin
    │   └── BaseEntityTime
    │
    └── product
        ├── JpaProductRepository
        ├── Product
        ├── ProductRepository
        ├── ProductRepositoryImpl
        ├── ProductSellingStatus
        └── ProductType
```  

### 프로퍼티 설정 {collapsible="true"}
```yaml
spring:
  profiles:
    default: local

  datasource:
    url: jdbc:h2:mem:~/cafeKioskApplication2
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: none
# 프로파일 잘못 설정으로 인한 문제를 방지하기 위함

---
spring:
  config:
    activate:
      on-profile: local

  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true
    defer-datasource-initialization: true # (2.5~) Hibernate 초기화 이후 data.sql 실행
    # 테이블 정보가 있어야 sql을 날릴수 잇기 때문에 나중에 실행하도록 설정
  h2:
    console:
      enabled: true
      path: /h2-console
---
spring:
  config:
    activate:
      on-profile: test

  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true

  sql:
    init:
      mode: never
# 테스트시에는 sql을 실행하지 않도록 설정
# 화면에 sql이 보이지 않도록 설정
```    

## 전체 코드 작성 순서
<procedure title="코드 작성 흐름" id="required_test">
    <step>
      <p>엔티티 코드 작성(Product)</p>
    </step>
    <step>
      <p>BaseEntity 생성</p>
    </step>
    <step>
      <p>ProductRepository(JPA interface) 생성</p>
    </step>
    <step>
      <p><code>API/domain/service/</code>ProductService 생성</p>
    </step>
    <step>
      <p>서비스 계층용 DTO(<code>ProductResponse</code>) 생성</p>
    </step>
    <step>
      <p>요구상항에 맞는 API용<code>Repository</code>메서드 생성</p>
    </step>
    <step>
      <p><code>ProductResponse</code>에 toEntity용 <code>of</code> 추가-빌더활용</p>
    </step>
    <step>
      <p><code>api/domain/controller/ProductController</code> API 메서드 추가</p>
    </step>
</procedure>  

## 남기고 싶은 내용
### ProductRepository 생성
강의에서는 `JpaRepository`를 상속한 인터페이스를 `ProductService`에 바로 주입을 했습니다.  
이런 경우 `QueryDSL`이나 `JDBC`를 사용하다가 다른 데이터 접근 방법을 적용해야하는 경우
테스트 코드를 새로 작성하거나 `API Service` 로직에 영향을 줄 수 있습니다.

+ ProductRepository ( 인터페이스 )
```Java
public interface ProductRepository {
    List<Product> findAllBySellingStatusIn(List<ProductSellingStatus> sellingStatuses);
}
```
+ Jpa 구현 빈 오브젝트
```Java
public interface JpaProductRepository extends JpaRepository<Product, Long> {
    List<Product> findAllBySellingStatusIn(List<ProductSellingStatus> sellingStatuses);
}
```  
+ 리포지토리 인터페이스 구현체
```Java
@Repository
@RequiredArgsConstructor
public class ProductRepositoryImpl implements ProductRepository{

    private final JpaProductRepository jpaProductRepository;

    @Override
    public List<Product> findAllBySellingStatusIn(List<ProductSellingStatus> sellingStatuses) {
        return jpaProductRepository.findAllBySellingStatusIn(sellingStatuses);
    }
}
```  

현재 구조는 `Service`가 `ProductRepository` 인터페이스를 의존합니다.  
클라이언트는 해당 인터페이스 메서드와 반환값을 알면 내부 로직을 몰라도 상관없습니다.

다만 `JpaRepository` 를 상속한 인터페이스 빈 오브젝트는 생명 주기는 스프링 컨테이너가 관리합니다.
`DI`를 통해서 필요한 의존성을 추가할 수 있지만, 관련된 기술을 제거하고 새 기술을 적용하기 따라롭습니다.

### ProductService 생성
> 테스크 코드에 의문점
>
+ 현재 서비스 코드
```Java
public List<ProductResponse> getSellingProducts() {
    // TODO 판매중인 것과 판매 보류인 상품들을 가져와서 ProductResponse로 변환하여 반환
    List<Product> products = productRepository.findAllBySellingStatusIn(ProductSellingStatus.forDisplay());
    return products.stream().map(ProductResponse::of)
            .collect(Collectors.toList());
}
```  
서비스계층은 리포지토리 계층을 의존하고 있습니다.   
`Service -> 리포지토리`  
서비스 계층은 리포지토리에 있는 데이터를 활용해도 상관 없습니다.
참조 방향은 동일하기 때문입니다. (`ServiceDTO`->`Entyty`)

<procedure title="forDisplay()" id="for_display_">
     <step>
     <p>클라이언트에서 리포지토리 결과를 제한하기</p>
     </step>
     <step>
     <p>엔티티 필드인 `enum`에 매서드 생성하기</p>
     </step>
</procedure>  

#### 차이
도메인에서 중요하게 여기는 비즈니스 정책에 따라 서비스 로직이 달라집니다.
> 현재 키오스크 도메인은 어떤 카테고리 메뉴를 선택해도 `보여주는`관점에서
> 동일합니다. (`ELLING_TYPE`, `HOLD`)
>
어느 컨트롤러나 서비스에서 `Product` 도메인을 사용할 때 클라이언트에게 보여주는 조건이 동일하면
엔티티 클래스나 엔티티 클래스의 특정 필드에서 관리를 합니다.

도메인 정책이 클라이언트의 요구 사항에 따라 보여주는 기준이 달라진다면 외부로 부터 받는게 맞습니다.ㄴ

+ 도메인에서 클라이언트의 요청에 따라 달라지는 경우
```Java
public List<ProductResponse> getSellingProducts(SearchCondition) {
      // TODO 판매중인 것과 판매 보류인 상품들을 가져와서 ProductResponse로 변환하여 반환
      List<Product> products = productRepository.findAllBySellingStatusIn(SearchCondition cond);
      return products.stream().map(ProductResponse::of)
              .collect(Collectors.toList());
  }  
```  
+ 도메인에 공통 정책인 경우
```Java
public List<ProductResponse> getSellingProducts() {
  // TODO 판매중인 것과 판매 보류인 상품들을 가져와서 ProductResponse로 변환하여 반환
  List<Product> products = productRepository.findAllBySellingStatusIn(ProductSellingStatus.forDisplay());
  return products.stream().map(ProductResponse::of)
  .collect(Collectors.toList());
}  
```  
### ProductService 용 DTO 생성
위와 마찬가지로 `ProductService`는 `Entity`를 만들고 소멸하고 생성해도 상관없습니다.
```Java
@Getter
public class ProductResponse {

    @Builder // 빌러는 private 으로 그대로 쓰지 못하게 막는다.
    // 질문1: Builder 를 사용하는 이유
    // 답변1: ProductResponse의 생성자를 private으로 막아놓고, ProductResponse를 생성할 때는 Builder를 사용하도록 하여, ProductResponse의 생성을 제한하고, ProductResponse의 생성을 제한함으로써, ProductResponse의 불변성을 보장하기 위해서 사용한다.
    private ProductResponse(Long id, String productNumber, String name, int price, ProductType type, ProductSellingStatus sellingStatus) {
        this.id = id;
        this.productNumber = productNumber;
        this.name = name;
        this.price = price;
        this.type = type;
        this.sellingStatus = sellingStatus;
    }

    public static ProductResponse of(Product product) {
        return ProductResponse.builder()
                .id(product.getId())
                .productNumber(product.getProductNumber())
                .name(product.getName())
                .price(product.getPrice())
                .type(product.getType())
                .sellingStatus(product.getSellingStatus())
                .build();
    }
}
```  
생성자로 만들기에는 경우의 수마다 추가를 해야합니다.
빌더패턴을 이용하면 `명시적인 초기화`,`불필요한 프로퍼티 메소드 제거`,`불변성`등이 있습니다.  
  