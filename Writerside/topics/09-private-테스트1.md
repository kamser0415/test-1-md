# private 테스트
  
private 메서드
: 프로덕션 코드를 작성하다보면 어떤 객체 안에 `public`메서드가 있고, 
내부에 로직이 길어지거나 혹은 이제 어떤 의미를 부여하고 싶을 때 `private`메서드로 추출하여 **추상화 과정**을 거치게 됩니다.  
  
`private`메서드가 추출되어 이 메소드를 테스트하고 싶을 수 있습니다.  
  
> private 메서드는 할 필요가 없습니다.   
> 하려고 해서도 안됩니다.
> 
{ style="warning"}  

## 객체를 분리할 시점
`private`메서드를 테스트하고 싶을 때 메서드를 분리해야하는 시점인지 질문을 던져야합니다.  
  
```Java
private String nextProductNumber(){
    String latestProductNumber = productRepository.findLatestProduct();
    if (latestProductNumber == null) {
        return "001";
    }
    Integer latestProductNumberInt = Integer.parseInt(latestProductNumber);
    Integer nextProductNumberInt = latestProductNumberInt + 1;
    return String.format("%03d", nextProductNumberInt);
}
```  
 
`public`메서드는 인터페이스 API 입니다.  
예를 들어 리포지토리 API를 사용하는 서비스 계층은 클라이언트가 됩니다.  
서비스 계층 API를 사용하는 컨트롤러 계층은 클라이언트가 됩니다.  
  
테스트 코드는 **클라이언트 입장**에서 사용하는 인터페이스 API를 검증하는 코드입니다. 
인터페이스 API는 입력과 출력에 대해서 클라이언트와 서버가 약속한 결과에 대한 검증할 뿐이지 내부 로직이 어떻게 동작하는지 알 필요가 없습니다. 
이게 객체지향에서 말하는 `캡슐화`라고 합니다.  
  
그래서 서버가 내부 로직으로 감춘 `private`매서드는 클라이언트 입장에서는 알필가 없습니다. 
그러면 테스트를 진행할 필요도 없다는 의미입니다.  

```Java
@DisplayName("신규 상품을 등록한다. 상품 번호는 최근 상품 번호에서 +1 증가한 값이다.")
@DisplayName("기존 상품이 없는 신규 상품을 등록한다. 상품 번호는 001 이다.")
```
`ProductService`가 `ProductRepository`의 API를 테스트를 한다고 생각해서 
테스트 내용을 살펴보면 신규 상품을 등록하는 역할과 상품번호를 만드는 역할을 같이 테스트하는게 
맞는지 모르겠다는 생각이 들 수 있습니다.  
  
이럴때는 `private` 메서드를 테스트 하는게 아닌, 객체를 분리할 시점인지 생각해야합니다.  
  
## private 메서드 분리하기  
`@Component`는 서버의 하나의 기능을 말합니다. 
클라이언트가 요청하여 **별도의 기능으로 동작**필요하면 객체를 분리할 시점이라고 생각합니다. 또는 하나의 공개 API안에 많은 기능이 있다면 
> 책임을 분리할 시점입니다.
```Java
@RequiredArgsConstructor
@Component
public class ProductNumberFactory {

    private final ProductRepository productRepository;

    public String createNextProductNumber() {
        String latestProductNumber = productRepository.findLatestProductNumber();
        if (latestProductNumber == null) {
            return "001";
        }

        int latestProductNumberInt = Integer.parseInt(latestProductNumber);
        int nextProductNumberInt = latestProductNumberInt + 1;

        return String.format("%03d", nextProductNumberInt);
    }

}
```  
해당 `Factory`는 클라이언트 입장에서 별도의 설정을 하지 않아도 호출시 
원하는 결과를 반환해주는 기능을 합니다.  
  
계층은 비즈니스 로직이 포함되어 있기 때문에 서비스 계층과 같이 통합 테스트를 통해서 검증할 수 있습니다.  
   
