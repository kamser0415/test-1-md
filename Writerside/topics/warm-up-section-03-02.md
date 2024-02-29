# 스프링 컨테이너  
  
`UserController`의 의아한 점

1. static이 아닌 코드를 사용하려면 인스턴스화가 필요하다.
    ```Java
    @RestController
    public class UserController {
        private final UserService userService;
        
        public UserController(JdbcTemplate jdbcTemplate){
            this.userService = new UserService(jdbcTemplate);
        }
    }
    ```  
    인스턴스를 생성하기 위해서는 생성자 함수를 사용해야합니다.
    그런데 현재 코드를 보면 생성자 호출하는 코드가 없습니다.
2. `JdbcTemplate`를 설정하고 인스턴스를 인수로 전달하지 않았습니다.  
    UserController가 JdbctTemplate를 사용할 수 있는 이유?  
  
## SpringBean
`@RestController`는 해당 클래스를 스프링 컨테이너에 스프링 빈으로 등록합니다.  
  
스프링 빈은 스프링 컨테이너가 관리하는 인스턴스 객체로 컨테이너마다 고유하게 존재합니다.  
  
스프링 서버 내부에 컨테이너를 만들어 관리를 합니다.  
  
그러면 `JdbcTemplate`은 스프링 부트가 기본적으로 시작할 때 등록합니다.

의존성을 추가하여 자동 등록 빈에서 관리하는 패키지 목록에 포함될 경우 빈으로 등록합니다.  
  
## 스프링 컨테이너를 사용하는 이유

의존성 주입이 필요한 클래스가 1개라면 직접 new로 생성해도 됩니다.  
만약 수십, 수백까지가 된다면, `BookRepository bookrRepository = new (...)`을 모두 변경해야합니다.  
  
강한 결합인 경우 상위 클래스인 `BooKService`가 의존하는 `BookRepository`를 수정하려면 
모든 코드를 수정해야하고, 테스트로 검증해애합니다.  
  
결합도를 낮추고자 인터 인스턴스를 의존해도 코드 수정에 대한 범위가 줄어들 분이지
기존 생성자 코드를 직접 클래스에서 수정해야합니다.  
  
스프링 컨테이너는 인수로 전달받은 클래스,메소드등의 애노테이션을 확인하여 
스프링 빈으로 등록하고, 스프링 빈 객체가 필요한 인수가 스프링 부트 컨테이너 내분에 관리 대상일 경우
대신 주입을 해줍니다.  
  
이렇게 의존성에 대한 제어가 클래스 내부가 아닌 외부에서 결정되므로,  
이걸 제어의 역전 (`IoC`)를 한다고하고 스프링 부트는 `IoC` 컨테이너라고 합니다.  
  
정리하면,
스프링 컨테이너를 사용하면 의존성 주입에 필요한 제어를 외부에서 관리하기 때문에 
객체지향적으로 코드를 작성할 수 있습니다.  
  
## 스프링 컨테이너를 다루는 방법
