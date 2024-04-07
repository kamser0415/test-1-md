# RaceCondition-Solution 두번째  
  
## RaceCondition 
공유 데이터를 수정하기 위한 프로그램이 순차적으로 실행되는 것이 아니라,
수정된 데이터를 공유 데이터에 반영되기 전에 다른 프로그램이 수정전 데이터 정보를 사용하여 
원자성이 보장되지 않을 때 발생한다.  
  
`RaceCondition`이 발생할 경우 `두 번의 갱실 분실 문제(Second lost updates proble)`가 발생할 수 있습니다.
  
### 두 번의 갱실 분실 문제란
재고 100개인 노트북을 구매하는 상황입니다.

| 시간 | 트랜잭션 A (구매)     | 트랜잭션 B (구매)     |
|----|-----------------|-----------------|
| 1  | 재고 확인: 100개     |                 |
| 2  |                 | 재고 확인: 100개     |
| 3  |                 | 구매 (재고 -1): 99개 |
| 4  | 구매 (재고 -1): 99개 |                 |
| 5  | 구매 확인: 99개      |                 |
| 6  |                 | 구매 확인: 99개      |  
  
두 개의 트랜잭션 A,B가 동시에 실행됩니다. 두 트랜잭션 모두 재고를 확인하고, 그다음 재고를 1개씩 줄입니다. 
하지만 `두 번의 갱실 분실 문제`가 발생하여 트랜잭션 `A`의 수정 사항이 분실하게 되었습니다.  
  
이를 해결하는 방법은 트랜잭션만으로 해결할 수 없기에 3가지 선택사항이 있습니다.  

### 갱실 분실 해결방안 세가지 
+ **마지막 커밋만 인정하기** : 트랜잭션 A의 내용은 무시하고 마지막에 커밋한 트랜잭션 B의 내용만 인정한다.
+ **최초 커밋만 인정하기** : 트랜잭션 A를 이미 수정을 완료했으므로 트랜잭션 B가 수정을 완료할때 오류가 발생한다.
+ **충돌하는 갱신 내용 병합하기** : 트랜잭션 A,B의 수정내용을 병합한다.
  
기본적인 방식은 마지막 커밋만 인정합니다. 하지만 재고처럼 수량이 정해진 경우에는 문제가 발생합니다. 
이런 상황에는 최초 커밋만 인정하는 것이 더 합리적으로 재고 수량을 관리할 수 있습니다.  
  
## 환경 구성
스프링 의존성  
```Gradle
testImplementation 'org.springframework.boot:spring-boot-starter-test'

// jpa
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
// mvc
implementation 'org.springframework.boot:spring-boot-starter-web'
// lombock
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
// db
runtimeOnly 'com.mysql:mysql-connector-j'
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
// redis - pub/sub 
implementation 'org.redisson:redisson-spring-boot-starter:3.27.2'
```  
### application.yml
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://127.0.0.1:3308/stock_example
    username: root
    password: 1234
    hikari:
      maximum-pool-size: 40
      
server:
  port: 9090 # 8080 포트는 사용중 

logging:
  level:
    org:
      hibernate:
        SQL: DEBUG
        type:
          descriptor:
            sql:
              BasicBinder: TRACE
```
  
### 마리아 DB 도커 컨테이너 사용 및 설정
```Java
# mariadb
docker pull mariadb:10.11.5

# docker volume 설정
docker volume create my-maria

# 외부 3308 - mariadb 3306 root 계정 비밀번호 1234로 초기화 및 volume 설정
docker run -d -p 3308:3306 --name mymaria -v my-maria:/etc/mysql/conf.d --env MARIADB_ROOT_PASSWORD=1234 --rm mariadb:10.11.5

# maria db 접속후 데이터베이스 생성
docker exec -it mymaria bash
mysql -u root -p
패스워드 입력: 1234
```    
도커 컨테이너는 재시작, 중지할 경우 컨테이너 내부에 있는 데이터는 컨테이너 초기 상태로 돌아갑니다. 
따라서 설정을 유지하고 싶은 경우에는 2가지 방법을 사용합니다.  
1. bindMount : 호스트 OS에서 파일을 관리
2. volume : 도커 엔진이 파일을 내부에서 관리  
  
도커는 `BindMount` 방식은 호스트에서 직접 도커 컨테이너의 파일을 관리하는 것을 권장하지 않습니다. 
도커 엔진이 내부적으로 관리하여 도커에서만 수정할 수 있는 `volume` 방식을 권장합니다.

### 테스트 환경 만들기  
마리아 디비 격리수준을 `READ-COMMITED`로 변경  
+ 사유: 동시성 제어가 필요한 환경과 맞추기 위해서
```Bash
# 격리 수준 확인하기
show variables like '%isolation';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+

# 격리 수준 변경하기 (현재 세션만 변경하려면 GLOBAL 대신 SESSION 사용)
SET GLOBAL tx_isolation = 'read-committed';

# 격리수준 확인하기
show variables like '%isolation';
+---------------+----------------+
| Variable_name | Value          |
+---------------+----------------+
| tx_isolation  | READ-COMMITTED |
+---------------+----------------+
```  
데이터 베이스 생성하기
```Bash
create database stock_example;
use database stock_example;
```  

## 문제 발생 코드만들기
Stock Entity
```Java
package org.example.stock_rt_1.domain;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@Getter
@NoArgsConstructor
public class Stock {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private long productId;

    private long quantity;

    public Stock(long productId, long quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    public void decrease(long quantity) {
        long currentQuantity = this.quantity - quantity;
        if (currentQuantity < 0) {
            throw new IllegalArgumentException("재고는 0보다 작을 수 없습니다.");
        }
        this.quantity = currentQuantity;
    }
}
```  
Service 로직
```Java
package org.example.stock_rt_1.service;

import org.example.stock_rt_1.domain.Stock;
import org.example.stock_rt_1.repository.StockRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class StockService {

    private final StockRepository stockRepository;

    public StockService(StockRepository stockRepository) {
        this.stockRepository = stockRepository;
    }

    @Transactional
    public void decreaseQuantity(long productId, long quantity) {
        Stock stock = stockRepository.findById(productId)
                .orElseThrow();
        stock.decrease(quantity);
    }
}
```
**테스트 코드**  
```Java

@SpringBootTest
class StockServiceTest {

    @Autowired
    private StockService stockService;
    @Autowired
    private StockRepository stockRepository;

    @Test
    @DisplayName("주문을 하면 재고가 감소한다.")
    void decreaseQuantity() throws InterruptedException {
        //given
        int threadCount = 100;
        final long productId = 1L;
        final long quantity = 1L;
        saveStock(productId, 100L);
        ExecutorService executorService = Executors.newFixedThreadPool(32);
        CountDownLatch count = new CountDownLatch(threadCount);

        //when
        for (int i = 0; i < threadCount; i++) {
            executorService.submit(()->{
                try {
                    stockService.decreaseQuantity(productId, quantity);
                } finally {
                    count.countDown();
                }
            });
        }
        count.await();

        //then
        Stock stock = stockRepository.findById(productId).orElseThrow();
        Assertions.assertThat(stock.getQuantity()).isZero();
    }

    private void saveStock(long productId, long quantity) {
        stockRepository.save(new Stock(productId, quantity));
    }
}
```  
테스트 결과
```Java
Expected :0L
Actual   :96L
```   
해당 문제가 발생한 이유는 `race Condition`으로 발생하여 데이터베이스는 기본 값인 마지막에 커밋된 내용을 기록하기 때문에 
중간 기록이 사라졌습니다.  
  
## 문제 해결 방법
> 전제조건은 사용하는 데이터베이스 서버는 한 대 입니다.  
  
`race Condition`가 발생하여 문제가 생기는 데이터는 크게 2가지 방식으로 처리됩니다. 
1. Lock을 획득하지 못한 쓰레드는 예외를 던지며 종료
2. Lock을 획득하지 못한 쓰레드는 대기 상태로 변환후 대기  
  
크게 2가지 방식을 어디서, 어떤 기술을 사용해서 적용할 것인지에 따라 나눌 수 있습니다.  

### 해결 위치  
1. 애플리케이션 계층 활용
   1. `synchronized` : 대기
   2. `@Version` + `Optimistic Lock` : 예외
   3. `Concurrent class` : 타임아웃
  
2. 데이터베이스 계층 활용
   1. `NamedLock` : 대기
   2. `Pessimistic Lock` : `read(예외)`, `write(대기)`  

3. Redis 활용
   1. `lattuce` 방식 : 예외
   2. `Redisson` 방식  : 대기
  
해당 작업이 무조건 실행이 되어야하는 경우에는 예외로 처리되는 방식인 경우 Lock을 얻기위해 무한 반복문으로 처리해야합니다. 
무조건 실행되어야한다면 쓰레드가 대기 상태로 가는 방식이 좋은 선택이 될 수 있습니다.  
  
## synchronized 방법
`synchronized`는 `JVM`이 자동으로 락을 관리해주기 때문에 직접 락을 관리할 필요가 없습니다.  
  
**장점**
+ 구현이 단순합니다 : 클래스나 메소드 앞에 추가하면 됩니다.
+ 가독성이 좋습니다 : 코드 내에서 명시적으로 동기화 영역이 표시됩니다.  
  
**단점**  
+ 확장이 어렵습니다 : 서버가 한대 일경우에만 동기화를 보장합니다.
+ 성능저하 : 다른 스레드들은 대기 상태가 되며, 타임아웃을 지정할 수 없습니다.
+ 유연성 부족 : 세밀하게 임계구역을 지정할 수 없으며, 매서드나 클래스가 잠금을 오래 들고 있는 경우 대기시간이 발생합니다.  
  
단일 서버에서 사용하더라도 세밀하게 조정할 수 있고 타임아웃도 지정할 수 있는 `java.util.concurrent` 패키지 내에 `ReentrantLock,Semaphore` 나 `Lock` 인터페이스를 사용하는 것이 좋습니다.


## @Version 방법
`JPA`가 제공하는 낙관적 락 방식으로 데이터를 수정할 때마다 `version`의 값을 변경합니다. 
데이터를 조회했을때 `version`값과 다를 경우 예외가 발생하여 첫번째 커밋을 반영하는 방식입니다.  

재시도를 추가하는 방법은 `commit` 시점에 `version`이 다를 경우 `OptimisticLockException`이 발생합니다. 
스프링 데이타 JPA를 사용하는 경우 이 예외를 추상화하여 `ObjectOptimisticLockingFailureException`로 변경합니다.  
  
`@Version`을 사용하는 방식은 3가지 입니다.  
`@Version`을 사용할 경우 기본적으로 JPA 낙관적 락을 기본으로 사용합니다.  
    
`LockModeType`에 상관없이 모두 `version`의 값을 변경하는 시점은 트랜잭션을 `commit`할때 발생합니다. 
처음 `select`로 버전을 읽어보고, 마지막에 커밋을 할 때 맨 처음 읽었던 `version`에 `+1`을 하여 업데이트를 합니다. 
이때 `update` 가 실패할 경우 예외가 발생합니다.
  
+ 각 엔티티당 버전 속정(`@Version`)은 하나만 있어야합니다.
  + int 또는 Integer
  + short 또는 Short
  + long 또는 Long
  + java.sql.Timestamp
+ 엔티티가 `version`필드를 추가해야하므로 `@Version`없는 낙관적 잠금도 가능합니다.
  + `@OptimisticLocking` 방식은 처음 읽어온 엔티티 스냅샷과 비교하여 동일한지 비교합니다.
  + `OptimisticLockType.ALL` : 처음 데이터의 모든 필드와 비교 
  + `OptimisticLockType.DIRTY` : 수정된 엔티티의 필드만 과거 데이터와 비교
   
**StockVersion Repository 코드**
```Java
@Lock(LockModeType.NONE)
@Query("select s from StockVersion s where s.productId = :id")
StockVersion findByIdNone(@Param("id") Long id);

@Lock(LockModeType.OPTIMISTIC)
@Query("select s from StockVersion s where s.productId = :id")
StockVersion findByIdOptimistic(@Param("id") Long id);

@Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
@Query("select s from StockVersion s where s.productId = :id")
StockVersion findByIdOptimisticIncr(@Param("id") Long id);
```
**Service 로직 코드**
```Java
@Transactional
public void decreaseVersionWithOptimistic(long productId, long quantity) {
  StockVersion stock = stockRepository.findByIdOptimistic(productId);
  stock.decrease(quantity); // quantity만큼 재고 감소 로직입니다.
}
```

### @Lock(LockModeType.NONE)
**테스트 코드**
```Java
@Test
@DisplayName("NONE 방식")
void decreaseVersionWithWhile() {
    final long productId = 1L;
    saveStock(productId, 100L);
    stockService.decreaseVersionWithNone(1L,1L);
}  
```  
**실행 로그**  
```Java
// 엔티티 조회 + VERSION
2024-04-06T17:36:12.132 : select sv1_0.id,sv1_0.product_id,sv1_0.quantity,sv1_0.version from stock_version sv1_0 where sv1_0.product_id=?
// commit 직전 update로 version도 같이 수정됩니다.
2024-04-06T17:36:12.133 : update stock_version set product_id=?,quantity=?,version=? where id=? and version=?
```  
**용도** 
+ 조회 시점부터 수정 시점(트랜잭션 종료)까지 보장됩니다.  
+ 동일한 엔티티를 읽고, 수정할 경우에 적합합니다.  
  
**방식**
+ 엔티티를 수정하여 반영할 때 처음 읽어온 `version+=1`로 업데이트를 합니다.
+ 이때 업데이트 결과가 0(`실패`)할 경우 예외가 발생합니다.  
  
> 주의사항  
> 수정하는 엔티티만 version 필드가 업데이트시 실패하면 예외가 발생하는 방식입니다. 
> 멤버 엔티티를 조회후 필드에 따라 오더 엔티티를 수정하거나 결제 엔티티를 수정하는 경우라면
> 멤버 엔티티에 대한 `REPEATABLE READ`가 보장되지 않습니다. 그 사이에 멤버가 수정되어도 검증할 수 없기 때문입니다.
  
### @Lock(LockModeType.OPTIMISTIC)  
**테스트 코드**
```Java
@Test
@DisplayName("Optimistic 방식")
void decreaseVersionWithOptimistic() {
    final long productId = 1L;
    saveStock(productId, 100L);
    stockService.decreaseVersionWithOptimistic(1L,1L);

}
```  
**실행 로그**  
```Java
# 엔티티 조회
2024-04-06T18:26:10.899+09:00 : select sv1_0.id,sv1_0.product_id,sv1_0.quantity,sv1_0.version from stock_version sv1_0 where sv1_0.product_id=?
# COMMIT 직전 엔티티 수정
2024-04-06T18:26:10.910+09:00 : update stock_version set product_id=?,quantity=?,version=? where id=? and version=?
# COMMIT 직전 조회한 엔티티 REPEATABLE READ 보장
2024-04-06T18:26:10.913+09:00 : select version as version_ from stock_version where id=?
```  
**용도**  
+ 조회한 엔티티는 조회 시점부터 트랜잭션이 종료될 때까지 다른 트랜잭션(세션)에서 변경되지 않았음을 보장합니다.
+ 같은 트랜잭션 내에 수정없이 읽기만 해도 버전을 한번더 읽어 확인합니다.  
  
**동작**
+ 트랜잭션을 `COMMIT`할 때 버전 정보를 조회해서 읽은 엔티티의 버전과 다를 경우 예외가 발생합니다.
  
**이점**  
+ `DIRTY READ`와 `NON-REPEATABLE READ`를 방지합니다.
  
> 주의사항   
> 읽은 엔티티가 변경되지 않은 엔티티일 경우 버전을 확인만 확인합니다(예외를 던지지 않음)
  

### @Lock(LockModeType.OPTIMISTIC_INCREMENT)
**테스트 코드**
```Java
@Test
@DisplayName("Optimistic_Increment 방식")
void decreaseVersionWithOptimisticIncr() {
    final long productId = 1L;
    saveStock(productId, 100L);
    stockService.decreaseVersionWithOptimisticIncr(1L,1L);
}
```  
**실행로그**
```Java
Hibernate: select sv1_0.id,sv1_0.product_id,sv1_0.quantity,sv1_0.version from stock_version sv1_0 where sv1_0.product_id=?
Hibernate: update stock_version set product_id=?,quantity=?,version=? where id=? and version=?
Hibernate: update stock_version set version=? where id=? and version=?
```  
> 낙관적 락을 사용하면서 버전을 강제로 증가합니다.  
  
**용도**  
1. 논리적인 단위의 엔티티 묶음을 관리하는 방법으로 사용할 수 있습니다.
2. 조회된 데이터가 중간에 변경될 경우 예외로 다른 동작을 추가할 수 있습니다.
  
예를 들면:  
**첫 번째 논리적인 단위로 사용하는 방법**  
게시판-첨부파일 관계일 때 처럼 사용할 수 있습니다. 첨부파일이 변경되면 해당 게시판은 수정된 내용은 없지만 
논리적으로 게시판도 수정된것으로 보고 강제로 버전을 증가할 수 있습니다.  
**데이터가 중간에 변경시 예외가 발생하게 하는 방법**  
회원의 멤버쉽 등급에 따라 이용권 금액이 달라진다면 처음 조회했던 멤버쉽 등급에서 이용권 금액을 산정하여 발급하기 전에 
멤버쉽이 변경되면 버전이 달라지기 때문에 예외가 발생하여 다른 방식으로 처리할 수 있습니다.  
  
**동작**  
엔티티를 수정하지 않아도 트랜잭션을 커밋할 때 `update`쿼리를 사용하여 버전을 증가합니다. 
만약 엔티티를 수정한다면 버전 정보는 2번 수정됩니다.  
  
**이점**  
강제로 버전을 증가하여 논리적인 단위의 엔티티 묶음을 버전 관리할 수 있습니다.  
  
## 분산락 - NamedLock  
`NamedLock`은 데이터베이스 메타데이터 Lock의 하위 시스템을 사용하여 구현되었습니다.  
+ 동일한 이름(`same name`)으로 여러 개의 잠금을 획득하는 것도 가능합니다. 
  + 다만 해당 세션이 모두 잠금을 해제 해야 사용이 가능합니다.
  + 여러 개의 잠금을 획득한 경우 교착 상태가 될 경우 `ER_USER_LOCK_DEADLOCK` 오류로 종료합니다. 이때 트랜잭션 롤백이되지 않습니다.
+ MySQL 기준 이름은 최대 64자입니다.
+ 데이터베이스 내 다른 클라이언트끼리 잠금 이름이 겹칠 수 있기때문에 잠금 이름앞에 데이터베이스 명,App 명을 사용하면 줄일 수 있습니다.
  + `stock_example.lock-stockId`
+ 여러 클라이언트개 해당 잠금을 얻는건 순서가 없습니다.
+ FOR UPDATE나 FOR SHARE 같은 경우 primary key를 사용하더라도 데이터가 없거나 보조 인덱스를 사용하는 경우 
어떤 레코드가 잠금이 될지 확신할 수 없다는 단점을 보완합니다.

분산락으로 인해 메인 서비스의 세션을 공유하면 안되기 때문에 별도의 쓰레드로 설정합니다.

### LockFunction
**간단 정리**   

| Name                | Description                                        |
|---------------------|----------------------------------------------------|
| GET_LOCK()          | 특정 이름의 잠금을 획득합니다.                                  |
| IS_FREE_LOCK()      | 지정된 이름의 잠금이 사용 가능한지 여부를 확인합니다.                     |
| IS_USED_LOCK()      | 지정된 이름의 잠금이 사용 중인지 여부를 확인하며, 사용 중이면 연결 식별자를 반환합니다. |
| RELEASE_ALL_LOCKS() | 현재 모든 명명된 잠금을 해제합니다.                               |
| RELEASE_LOCK()      | 특정 이름의 잠금을 해제합니다.                                  |

### GET_LOCK(str,timeout)
`str`이름의 잠금을 획득 하려고,`timeout`만큼 대기합니다. (`timeout`이 음수 일 경우 무한 대기)  
한 세션이 보유하고 있는 동안 다른 세션은 동일한 이름의 잠금을 얻을 수 없습니다.  

| 응답         | 결과              |
|------------|-----------------|
| 1          | 잠금 획득           |
| 0          | 시간 초과 실패        |
| NULL error | 메모리 부족 및 쓰레드 종료 |

### RELEASE_LOCK(str)
`str` 이름의 잠금을 해제 합니다.  

| 응답         | 결과                       |     
|------------|--------------------------|     
| 1          | 잠금 반납                    |     
| 0          | 이 쓰레드에 의해 잠금이 설정되지 않은 경우 |     
| NULL error | 해당 이름의 잠금이 없는 경우         |     
                                     
**잠금 확인 방법**  

`MariaDB` 기준 `information_schema.METADATA_LOCK_INFO`(플러그인 설치 필수)
`MySQL` 기준 `information_schema.metadata_locks`    

`TABLE_SCHEMA` 필드가 `NamedLock`의 이름입니다.
```
+-----------+---------------------+---------------+-----------+--------------+------------+
| THREAD_ID | LOCK_MODE           | LOCK_DURATION | LOCK_TYPE | TABLE_SCHEMA | TABLE_NAME |
+-----------+---------------------+---------------+-----------+--------------+------------+
|         6 | MDL_SHARED_NO_WRITE | NULL          | User lock | lock1        |            |
|         3 | MDL_SHARED_NO_WRITE | NULL          | User lock | lock         |            |
+-----------+---------------------+---------------+-----------+--------------+------------+
```                                                 
  
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true

  datasource:
    hikari:
      driver-class-name: org.mariadb.jdbc.Driver
      max-lifetime: 60000
      username: root
      password: 1234
      maximum-pool-size: 30 # 
      pool-name: Spring-HikariPool
      jdbc-url: jdbc:mariadb://127.0.0.1:3308/stock_example

user-lock:
  datasource:
    hikari:
      jdbc-url: jdbc:mariadb://127.0.0.1:3308/stock_example
      driver-class-name: org.mariadb.jdbc.Driver
      max-lifetime: 60000
      username: root
      password: 1234
      maximum-pool-size: 10 #
      pool-name: Lock-HikariPool

server:
  port: 9090
```

별도의 쓰레드 풀을 만들어 사용합니다.  
```Java
// 기본 데이터소스를 사용하고
@Primary
@Bean
@ConfigurationProperties("spring.datasource.hikari")
public HikariDataSource dataSource() {
    return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
// 필요한 경우에만 사용합니다.
@Bean
@ConfigurationProperties("user-lock.datasource.hikari")
public HikariDataSource userLockDataSource() {
    return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
```  
   
[배민 MySQL 분산락 처리 출처](https://techblog.woowahan.com/2631/)  
  
현재 상황에 맞게 별도로 수정했습니다.
```Java
package org.example.stock_rt_1.repository;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Repository;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@Repository
@Slf4j
public class UserLockRepository {

    public static final String GET_LOCK = "SELECT GET_LOCK(?,?)";
    public static final String RELEASE_LOCK = "SELECT RELEASE_LOCK(?)";
    private static final String EXCEPTION_MESSAGE = "LOCK 을 수행하는 중에 오류가 발생하였습니다.";
    private final DataSource dataSource;

    public UserLockRepository(@Qualifier("userLockDataSource") DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public <T> void executeWithNamedLock(String userLockName, int timeoutSeconds, Runnable runnable) {

        try(Connection conn = dataSource.getConnection()){
            try {
                log.info("start getLock = [{}], timeoutSeconds = [{}], connection = [{}]", userLockName, timeoutSeconds, conn);
                getLock(conn,userLockName,timeoutSeconds);
                log.info("success getLock = [{}], timeoutSeconds = [{}], connection = [{}]", userLockName, timeoutSeconds, conn);
                runnable.run();
            } finally {
                log.info("start getLock = [{}], timeoutSeconds = [{}], connection = [{}]", userLockName, timeoutSeconds, conn);
                releaseLock(conn, userLockName);
                log.info("start getLock = [{}], timeoutSeconds = [{}], connection = [{}]", userLockName, timeoutSeconds, conn);
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    private void releaseLock(Connection conn, String userLockName) throws SQLException {
        try(PreparedStatement preparedStatement = conn.prepareStatement(RELEASE_LOCK)){
            preparedStatement.setString(1,userLockName);
            checkResultSet(userLockName,preparedStatement,"RELEASE_");
        }
    }

    private void getLock(Connection connection,String userLockName,int timeoutSeconds) throws SQLException {
        try(PreparedStatement prepareStatement = connection.prepareStatement(GET_LOCK)){
            prepareStatement.setString(1, userLockName);
            prepareStatement.setInt(2, timeoutSeconds);

            checkResultSet(userLockName,prepareStatement,"GetLock_");
        }
    }

    private void checkResultSet(String userLockName, PreparedStatement preparedStatement, String type) throws SQLException {
        try (ResultSet resultSet = preparedStatement.executeQuery()) {
            if (!resultSet.next()) {
                log.error("USER LEVEL LOCK 쿼리 결과 값이 없습니다. type = [{}], userLockName : [{}], connection=[{}]", type, userLockName, preparedStatement.getConnection());
                throw new RuntimeException(EXCEPTION_MESSAGE);
            }
            int result = resultSet.getInt(1);
            if (result != 1) {
                log.error("USER LEVEL LOCK 쿼리 결과 값이 1이 아닙니다. type = [{}], result : [{}] userLockName : [{}], connection=[{}]", type, result, userLockName, preparedStatement.getConnection());
                throw new RuntimeException(EXCEPTION_MESSAGE);
            }
        }
    }
}
```  
**실행로직**  
```Java
public void decrementWithNamedLock(long productId, long quantity){
    userLockRepository.executeWithNamedLock(
            "stock-"+productId
            ,10, ()->stockService.decreaseQuantity(productId, quantity));
}
```  

사용되는 커넥션 수는 `메인 쓰레드풀`+`분산 락을 위한 별도 쓰레드 풀` 개입니다.  
처음부터 분산 락을 위한 쓰레드 풀을 만들지 않습니다. 해당 쓰레드를 사용할때 `max`갯수 만큼 만들어놓습니다.

   
> 아쉬운점 불필요한 try-resource-catch 구문을 제거하고 싶습니다.  
> 트랜잭션 매니저를 하나 더 등록하면 될 수 있지만 
> 트랜잭션 매니저를 등록하면 기본으로 제공하는 트랜잭션매니저는 빈으로 등록되지 않습니다.  
> 그러면 2개의 트랜잭션을 개별로 등록해야하는 과정이 생기는데 
> 차라리 try-resource-catch를 넣는게 낫다고 봅니다.  
>   

### 분산락으로 처리할 때 핵심
네임드 락을 거는 구간과 실제로직은 별개의 트랜잭션으로 관리해야합니다.   

```Java
@Transasctional
public synchroniezed void sample(){...}
```  
과 동일한 구조로 JPA는 트랜잭션이 종료될때 커밋이 되므로 그 사이 동기화가 끝나기 때문에 동시성 문제가 발생합니다.  

## Redis 활용
레디스를 Spring Boot Data Redis를 사용한다고 하면 Redis 드라이버인 `lettuce`,`jedis`,`redisson`중 필요한 드라이브를 등록하여 사용합니다.  
기본으로 제공되는 `lettuce`을 사용하는 방식과 추가로 다양한 기능을 제공하는 `redisson` 드라이버를 사용합니다 
  
[Redisson vs Lettuce](https://redisson.org/feature-comparison-redisson-vs-lettuce.html)  
    
동기화를 위해 2가지 방법을 사용할 수 있습니다.  
1. 소멸시간이 있는 명령어 `SETNX`를 사용하는 tryLock() 방식
   + 재시도 로직을 넣을 경우 직접 구현해야함
   + 무한 반복문이나 횟수 반복문을 넣어야함
2. Pub/Sub 구조를 활용하는 Lock() 방식
   + 차례대로 잠금 획득과 반납을 이용하는 방식  
  

### SETNX 방식
레디스 `SETNX` 방식은 `HashMap`에 키를 설정할 때, 이미 설정된 값이 있다면 false, 설정된 값이 없으면 true를 반환합니다.  
  
> 주의사항  
> expire을 설정한 경우 비즈니스 로직이 완료되지 않아도 setNx가 해제됩니다.  
> expire 시간을 설정하지 않거나, 적절한 시간을 설정해야 합니다.  
  
`SETNX`는 현재 `deprecated`입니다.  
`SET` 명령어를 통해 두 경우를 모두 구현할 수 있습니다.

```Shell
SET [KEY] [VALUE] [NX] [EX|PX] (a positive integer)

# expire 시간을 설정하지 않을 경우
SET lock-productId "1" NX 
127.0.0.1:6379> set lock-productId "1" NX
OK
127.0.0.1:6379> set lock-productId "1" NX
(nil)

# expire 설정한 경우
127.0.0.1:6379> SET lock-productId "1" NX EX 1
OK

// 1초 후
127.0.0.1:6379> set lock-productId "1" NX EX 1
OK
```  
+ `NX` 옵션: 키가 있다면 (nil), 없다면 OK 반환  
+ `EX` 옵션: expire 단위 `초`
+ `PX` 옵션: expire 단위 `밀리초`
  
+ NamedLock 과 비교

|           | SETNX | DB NamedLock |
|-----------|-------|--------------|
| 추가 세션 필요  | X     | O            |
| 쓰레드 대기    | X     | O            |
| 타임아웃      | X     | O            |
| 소멸 시간     | O     | X            |
| 별도 재시도 구현 | O     | X            |
   
레디스를 활용하는 경우 인프라 구축으로 인한 비용과 추가적인 레디스 유지 보수에 대한 학습이 필요합니다. 
다만 DB의 NamedLock은 추가 세션관리에 신경을 써야합니다.  
공통점은 레디스나 DB나 모니터링을 해야하는 공통점이 있습니다.  
  
### 코드 구현
**레디스 SETNX 방식 코드**
```Java
@Component
@RequiredArgsConstructor
public class RedisRetryLock {

    public static final String LOCK = "lock";
    private final StringRedisTemplate stringRedisTemplate;

    private Boolean getLock(String userLockName, @Nullable Long expireMillis) {
        ValueOperations<String, String> opsForValue = stringRedisTemplate.opsForValue();
        if (Objects.isNull(expireMillis)) {
            return opsForValue.setIfAbsent(userLockName, LOCK);
        }
        return opsForValue.setIfAbsent(userLockName,LOCK, Duration.ofMillis(expireMillis));
    }
    private void releaseLock(String userLockName) {
        stringRedisTemplate.delete(userLockName);
    }
    public void tryLock(String userLockName,Runnable runnable, @Nullable Long expireMillis) throws InterruptedException {
        if (acquireLock(userLockName, expireMillis)){
            try {
                runnable.run();
            } finally {
                releaseLock(userLockName);
            }
        } else {
            throw new RuntimeException("작업이 실패했습니다.");
        }
    }
    private boolean acquireLock(String userLockName, Long expireMillis) throws InterruptedException {
        for (int i = 0; i < 99; i++) {
            if (getLock(userLockName, expireMillis)) {
                return true;
            }
            Thread.sleep(200);
        }
        return false;
    }
}
```  
**레디스 파사드 클래스**  
```Java
@Component
@RequiredArgsConstructor
public class RedisRetryLockFacade {

    private final RedisRetryLock redisRetryLock;
    private final StockService service;

    public void decrement(long productId, long quantity) throws InterruptedException {
        redisRetryLock.tryLock("lock-"+productId,
                ()->service.decreaseQuantity(productId,quantity),null);
    }
}
```  
  
> 주의사항  
> 잠금 획득까지 횟수를 지정하거나 무한 반복으로 할 경우 재시도 시간의 컴을 줘야합니다.  
> 구현 방식은 간단하나 무한 재시도를 하는 로직일 경우 레디스에 부하가 갈 수 있습니다.  
>   
  
### 메세지 방식
`Redisson pub/sub` - 채널 하나를 만들고 락을 점유중인 쓰레드가 락 획득하려고 대기중인 쓰레드에게
해제를 알려주면 안내를 받은 쓰레드가 락 획득 시도를 하는 방식을 제공합니다.  
  
**장점**  
+ 재시도 반복 로직을 작성하지 않아도 된다.
+ 재시도로 인한 레디스 부하가 상대적으로 적다.  
  
**단점**  
+ redisson 의존성을 추가해야합니다.  
  
**실행 로직**  
```Java
@Slf4j
@Component
@RequiredArgsConstructor
public class RedissonMessageLock {

    private final RedissonClient redissonClient;

    public void tryLock(String lockName,Runnable runnable) {
        RLock lock = redissonClient.getLock(lockName);
        try {
            run(runnable, lock);
        } finally {
            lock.unlock();
        }
    }
    private void run(Runnable runnable, RLock lock) {
        try {
            if (lock.tryLock(10, 1, TimeUnit.SECONDS)) {
                runnable.run();
                return;
            }
            log.info("락 획득 실패");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```    
  
이하코드는 동일합니다.