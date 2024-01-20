# test_tddorder_sc1-03-JPA 적용하기.md

목표
: 메모리 저장 방식에서 JPA를 활용하여 H2 데이터 베이스로 저장하도록 코드를 수정합니다.
  
## RestAssured 주의사항
[@Transactinal 미적용 - 공식문서](https://docs.spring.io/spring-boot/docs/3.1.8/reference/html/features.html#features.testing.spring-boot-applications)

번역
: 만약 당신의 테스트가 @Transactional 어노테이션을 사용하면, 
기본적으로 각 테스트 메서드의 끝에서 트랜잭션을 롤백합니다. 
그러나 `RANDOM_PORT` 또는 `DEFINED_PORT`와 함께 이 구성을 사용하는 경우, 
실제 서블릿 환경을 묵시적으로 제공하므로 HTTP 클라이언트와 
서버는 별도의 스레드에서 실행되며 따라서 별도의 트랜잭션에서 실행됩니다. 
서버에서 시작된 어떠한 트랜잭션도 이 경우에 롤백되지 않습니다.

정리하자면,  
1. 서버와 클라이언트가 분리되어서 동작한다.
2. 클라이언트가 롤백을해도 서버는 트랜잭션 경계가 다르기 때문에 데이터는 유지된다.
3. 따라서, 데이터 격리가 어렵다.  
  
## 트랜잭션 격리 방법
1. @BeforeEach
    ```Java
    @AfterEach
    void tearDown(){
        repository.deleteAllInBatch();
          :
    }
    ```
2. EntityManager 사용  
    [우아한 테크세미나 ATDD](https://www.youtube.com/watch?v=ITVpmjM4mUE)
    테스트 케이스마다 엔티티 매니저에 저장된 테이블을 초기화 합니다.
    ```Java
    //엔티티 자바 코드로 초기화하기
    import com.google.common.base.CaseFormat;
    
    @Component
    public class DatabaseCleanUp {
        @PersistenceContext
        private EntityManager entityManager;
    
        private List<String> tableNames;
    
        @PostConstruct
        public void init() {
            final Set<EntityType<?>> entities = entityManager.getMetamodel().getEntities();
            tableNames = entities.stream()
                    .filter(e -> isEntity(e) && hasTableAnnotation(e))
                    .map(e -> {
                        String tableName = e.getJavaType().getAnnotation(Table.class).name();
                        return tableName.isBlank() ? CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, e.getName()) : tableName;
                    })
                    .collect(Collectors.toList());
    
            final List<String> entityNames = entities.stream()
                    .filter(e -> isEntity(e) && !hasTableAnnotation(e))
                    .map(e -> CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, e.getName()))
                    .toList();
    
            tableNames.addAll(entityNames);
        }
    
        private boolean isEntity(final EntityType<?> e) {
            return null != e.getJavaType().getAnnotation(Entity.class);
        }
    
        private boolean hasTableAnnotation(final EntityType<?> e) {
            return null != e.getJavaType().getAnnotation(Table.class);
        }
    
        @Transactional
        public void execute() {
            entityManager.flush();
            entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY FALSE").executeUpdate();
            for (final String tableName : tableNames) {
                entityManager.createNativeQuery("TRUNCATE TABLE " + tableName).executeUpdate();
                entityManager.createNativeQuery("ALTER TABLE " + tableName + " ALTER COLUMN ID RESTART WITH 1").executeUpdate();
            }
            entityManager.createNativeQuery("SET REFERENTIAL_INTEGRITY TRUE").executeUpdate();
        }
    }    
    ```  
    **Api Test 통합 환경 수정**
    ```Java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    @Slf4j
    public class ApiTest {
    
        @Autowired
        protected DatabaseCleanUp databaseCleanUp;
    
        @LocalServerPort
        private int port;
    
        @BeforeEach
        void setUp() {
            //API 요청
            //상품 조회 restAssured
            log.info("RestAssured-port: {}",RestAssured.port);
            if (RestAssured.port == RestAssured.UNDEFINED_PORT) {
                RestAssured.port = port;
                //처음 시작할 때 테이블의 이름을 가져오기
                databaseCleanUp.init();
            }
            //시작마다 모든 테이블을 초기화
        }
    }
    ```
    > 기본적으로 RestAssured 의 포트는 `-1`로 초기화가 됩니다. 테스트 마다 포트가 초기화가 안될 경우 
    > 스프링 부트가 만든 랜덤 포트번호를 주입합니다.
  
## JPA로 변경
테스트는 인베디드 H2 데이터베이스를 사용합니다.  
  
1. ProductRepository 변경
    ```Java
    interface ProductRepository extends JpaRepository<Product,Long> {
    
    }
    ```
2. Product 변경
    Product 를 Entity로 사용할 수 있게 변경합니다.
    ```Java
    @Entity
    @Table(name="products")
    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    class Product {
    
        @Id
        @GeneratedValue
        private Long id;
        private int price;
        private String name;
        private DiscountPolicy discountPolicy;
        
    }
    ```  
  
> 여기까지 의문점 2개
> 1. DatabaseCleanup의 초기화 방식을 왜 인터페이스를 사용했을까?
> 2. 헥사고날 아키넥쳐는 Port에 머가 올지 모르는 상황에서 유연하게 하기 위해 사용하는 아키텍쳐인데 
> 이미 서비스 계층에서 `Product`에 Entity가 붙어있어서 Jpa를 사용한다고 이미 알고 있다. 
> 
{ style="warning"}
