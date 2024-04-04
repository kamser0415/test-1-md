# RaceCondition-Solution

## 정의
경합 상태는 둘 이상의 스레드가 공유 데이터에 액세스할 수 있고 동시에 공유 데이터를 변경하려고 할 때 발생합니다. 
스레드 스케줄링 알고리즘은 언제든지 스레드 간에 스왑할 수 있기 때문에 스레드가 공유 데이터에 접근하려고 시도하는 순서를 알 수 없습니다.  
따라서 데이터 변경의 결과는 스레드 스케줄링 알고리즘에 따라 달라지게 되며, 두 스레드 모두 데이터에 액세스/변경하기 위해 **경합**합니다.
  
## 설명  
**정상 동작**  

![image_156.png](image_156.png)     
  
**Race Condition 발생**  

![image_157.png](image_157.png)  
  
데이터를 읽고 수정하는 코드 사이에 다른 쓰레드가 접근할 경우 발생하는 현상입니다.    

톰캣은 멀티쓰레드로 동작하기 때문에 `Race Condition` 문제를 해결해야 합니다.
  
## 요구사항
선착순 100명에게 할인 쿠폰을 제공하는 이벤트를 진행합니다.
  
**세부 조건** 
1. 선착순 100명에게만 지급되어야 한다.
2. 101개 이상 지급되면 안된다.
3. 순간적으로 몰리는 트래픽을 버틸 수 있어야한다.
   + 다른 서비스에 차질이 생기면 안된다.

## 선착순 요구사항 문제점 
할인 쿠폰의 갯수는 총 100개입니다.  
  
테스트 방식은 다음과 같이 진행합니다.  
1. 싱글 쓰레드에서 할인 쿠폰 100개만 발행되는지 테스트하기
2. 멀티 쓰레드에서 할인 쿠폰 100개만 발행되는지 테스트하기
  
### 싱글쓰레드 테스트

**서비스 코드**
```Java
public void applyWithSQLCount(Long userId) {
    final int MAX_COUPON_COUNT = 100;
    
    // SELECT COUNT(*) FROM Coupon;
    long count = couponRepository.count();
    if (count >= MAX_COUPON_COUNT) {
        return;
    }
    couponRepository.save(new Coupon(userId));
}
```   
**싱글 쓰레드 환경에서 테스트 코드작성**
```Java
@Test
@DisplayName("선착순 100명에게만 쿠폰을 발행한다.")
public void applyBasic() {
    final int MAX_COUPON_COUNT = 100;
    for (long i = 1L; i <300L; i++) {
        applyService.applyWithSQLCount(i);
    }
    long count = couponRepository.count();
    Assertions.assertThat(count).isEqualTo(MAX_COUPON_COUNT);
}
```  
**결과**  

![image_158.png](image_158.png)  
  
싱글 쓰레드 환경에서는 공유 데이터를 수정할 때 원자성이 보장되기 때문에 
`Race Condition` 발생되지 않습니다.

### 멀티쓰레드  
멀티 쓰레드를 만들어서 테스트를 진행할 때 2가지 방법이 있습니다.  
+ 요청마다 쓰레드를 생성 -> 실행 -> 소멸을 진행
+ 쓰레드 풀을 사용하여 쓰레드를 일정 갯수를 유지하고 재사용하는 방법
  
톰캣은 쓰레드 풀을 사용하기때문에 테스트 환경도 쓰레드 풀로 테스트합니다.  
  
**쓰레드 풀 테스트코드**  
```Java
@Test
@DisplayName("데이터베이스 COUNT(*)를 사용하여 선착순 100명에게만 쿠폰을 발행한다.")
public void applyWithSQLCount2() throws InterruptedException {
    final int REQUEST_CNT = 3000;
    ExecutorService threadPool = Executors.newFixedThreadPool(32);
    CountDownLatch latch = new CountDownLatch(REQUEST_CNT);
    for (int i = 0; i < REQUEST_CNT; i++) {
        final long finalId = i;
        threadPool.submit(() -> {
            try {
                applyService.applyWithSQLCount(finalId);
            } finally {
                latch.countDown();
            }
        });
    }
    latch.await();

    long count = couponRepository.count();
    Assertions.assertThat(count).isEqualTo(100);
}
```
#### Executors.newFixedThreadPool(32)  
![image_159.png](image_159.png)   
  
`submit()`을 호출하면 `Task`를 작업 큐에 저장할 때 실행 가능한 쓰레드가 있으면 쓰레드를 찾아서 실행합니다.  
쓰레드가 없다면 새로 생성하는데 최대 **Fix:32**로 고정되어 생산하며, 모든 쓰레드가 동작중이라면 대기합니다.   

### 결과 대기
**`Thread.join()`을 사용하지 않은 이유**    

`join()`은 해당 쓰레드가 종료될때까지 호출한 쓰레드는 `Blocking`상태가 됩니다. 
쓰레드 풀을 여러 쓰레드를 재사용하기 때문에 `join()`을 사용하는건 맞지않습니다.  
  
쓰레드 풀은 비동기로 동작하기 때문에 쓰레드 풀 내부에 Task가 종료를 확인하는 방법은
+ `Future.get()` 해당 task가 종료될때까지 대기후 결과를 받을 수 있습니다
+ `CountDownLatch` task가 종료될때마다 카운팅을 합니다.  
  
#### Future.get() 방식   
`Future`란 비동기 계산의 결과를 나타냅니다.  
해당 인터페이스는 다음과 같은 기능을 제공합니다.
1. 계산 완료 확인 (`isDone()`)
2. 계산 종료까지 대기하고 결과값 받기(`get()`)
3. 계산 취소하기 (`cancel()`)  
   
```Java
@Test
@DisplayName("데이터베이스 COUNT(*)를 사용하여 선착순 100명에게만 쿠폰을 발행한다.")
public void applyWithSQLCount3() throws InterruptedException, ExecutionException {
    final int REQUEST_CNT = 3000;
    ExecutorService threadPool = Executors.newFixedThreadPool(32);
    ArrayList<Future<?>> tasks = new ArrayList<>();
    for (int i = 0; i < REQUEST_CNT; i++) {
        final long finalId = i;
        Future<?> submit = threadPool.submit(() -> {
            applyService.applyWithSQLCount(finalId);
        });
        tasks.add(submit);
    }
    // 결과를 모두 대기합니다.
    for (Future<?> task : tasks) {
        task.get();
    }
    long count = couponRepository.count();
    Assertions.assertThat(count).isEqualTo(100);
    threadPool.shutdown();
}
```

#### CountDownLatch 방식  
![image_160.png](image_160.png)

```Java  
@Test
@DisplayName("데이터베이스 COUNT(*)를 사용하여 선착순 100명에게만 쿠폰을 발행한다.")
public void applyWithSQLCount2() throws InterruptedException {
    final int REQUEST_CNT = 3000;
    ExecutorService threadPool = Executors.newFixedThreadPool(32);
    CountDownLatch latch = new CountDownLatch(REQUEST_CNT);
    for (int i = 0; i < REQUEST_CNT; i++) {
        final long finalId = i;
        threadPool.submit(() -> {
            try {
                applyService.applyWithSQLCount(finalId);
            } finally {
                latch.countDown();
            }
        });
    }
    latch.await();

    long count = couponRepository.count();
    Assertions.assertThat(count).isEqualTo(100);
    threadPool.shutdown();
}
```  
`CountDownLatch`은 인스턴스 변수로 `volatile int state`으로 쓰레드 갯수를 저장합니다. 
+ `volatile`은 CPU 캐시 메모리가 아니라 물리 메모리를 읽게하며, 코드 최적화로 발생하는 재정렬을 방지합니다.  
    + volatile은 원자성을 보장하지 않습니다.  
+ 쓰레드가 종료될 때마다 `countDown()`으로 내부 `state` 값을 감소합니다.
  + `countDown()` 메서드로 저장된 state를 감소할 때 원자성을 보장합니다.  
    ```Java
    # wait() 호출시 acquires = 1을 전달합니다.
    # 오버라이딩
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }
    # 무한반복문 종료 옵션
    if(tryAcquireShared(arg) >= 0){
        // 종료
    }
    ```  
    `await()`는 내부 무한 반복문을 돌려서 `countDown()`으로 `state = 0`이 될때까지 반복합니다. 
  
    ```Java
    org.opentest4j.AssertionFailedError: 
    expected: 100L
     but was: 119L
    Expected :100L
    Actual   :119L
    ```  
  
### 실패 이유    
할인 쿠폰 100장만 발생되어야하지만, 지금 119장이 발행되었습니다.  
   
원인은 멀티쓰레드 환경에서 동시에 변경하려고 하기 때문에 발생했습니다.  
  
1. 100개의 할인 쿠폰을 확인하는 방법은 `select count(*) from Coupon`으로 쿠폰 총 갯수를 조회
2. 여러 개의 쓰레드가 공유 데이터인 메서드에 접근 ( 컨택스트 스위칭, 다른 cpu를 통해 접근 )
3. 스레드 A와 스레드 B 모두 읽을 때는 `count`값이 99로 `count >= 100`을 통과
  
| 스레드 A                                                  | 스레드 B                                                   | 문제 발생 원인                                                                       |
|--------------------------------------------------------|---------------------------------------------------------|--------------------------------------------------------------------------------|
| `couponRepository.count()`를 호출하여 쿠폰 개수를 읽음             |                                                         |                                                                                |
|                                                        | `couponRepository.count()`를 호출하여 쿠폰 개수를 읽음              | `count`를 다시 읽기 전에 `couponRepository.count()`가 호출되어 `count` 값이 갱신되는 경우 발생할 수 있음 |
| `count` 값 확인 `count=99`                                |                                                         |                                                                                |
| `count` 값이 100 이상인지 확인하여, 100 이상이면 아무 작업도 하지 않고 메서드 종료 |                                                         |                                                                                |
|                                                        | `count` 값 확인   `count=99`                               |                                                                                |
|                                                        | `count` 값이 100 이상인지 확인하여, 100 이상이 아니면 `Coupon` 저장 작업 수행 | `count` 값이 100 미만으로 확인된 후에도 다시 `Coupon` 저장 작업이 수행될 수 있음                        |

SQL 중 `count()`함수는 MySQL InnoDB 스토리지 엔진을 기준으로 
1. 캐시로 총 레코드의 수를 기록하지 않습니다.
2. 매번 디스크에서 읽어오게 됩니다.  
   
그래서 CPU의 캐시 문제로 인한 동시성 발생하지 않을 수 있지만, 멀티 쓰레드에서 쓰기 전에 다른 쓰레드가 디스크에서 읽어오기 때문에 문제가 발생합니다.  
  
### 해결방안  
1. 어플리케이션쪽에서 동시성 문제를 해결한다.
2. 데이터베이스쪽에서 동시성 문제를 해결한다.

#### 어플리케이션 해결방안
1. Atomic,Concurrent 클래스 사용
   1. AtomicLong을 사용한다.
       + MySQL의 `auto_increment` 처럼 데이터를 넣을때마다 증가를 시키고 100개가 넘어가면 저장을 하지 않는다.
        ```
        AtomicInteger atomicInt = new AtomicInteger(0);
        
        public void findByIdWithAtomic(Long userId) {
            if (atomicInt.getAndIncrement() >= 100 ) {
                return;
            }
            couponRepository.save(new Coupon(userId));
        }
        ```  
        
   2. ConcurrentQueue 구조를 활용한다.  
        + Stack과 Queue 구조를 모두 활용 가능한 클래스를 활용하여 100개 스택을 넣어놓고 스택이 없으면 쿠폰 발행을 중지한다.
        ```Java
        private final ConcurrentLinkedDeque<Integer> stack = new ConcurrentLinkedDeque<>();
        
        # 생성자
        public ApplyService() {
            for (int i = 1; i <= 100; i++) {
                stack.offer(i); // offer 메서드를 사용하여 요소를 끝에 추가
            }
        }
        
        public void applyWithCon(Long userId) {
            Integer count = stack.pollLast();
            if (count == null) {
                // 발행 종료
                return;
            }
            couponRepository.save(new Coupon(userId));
        }
        ```  
   + **AtomicLong**등은 동시성 문제를 제어할 수 있는 방법중 하나이지만, 웹 환경에서는 단순한 메모리 공유를 제어하는 것만으로는 복잡한 요구사항을 해결하기 어렵습니다.  
   + **ConcurrentHashMap,CopyOnWriteArrayList**의 자료구조를 활용하면 구현할 수 있지만
     1. 웹 서버가 여러대로 동작할 경우 대응하기가 어렵다.
     2. 메모리에 저장되기에 불안전하며, 확장하기 어렵고, 오류 발생시 백업하기도 어렵습니다.
2. 동기화 사용(`synchronized`) 및 `lock()` 사용  
   1. synchronized  
      + 해당 메서드는 모두 임계영역이 되어 하나의 쓰레드만 입장가능하고 나머지는 대기합니다.  
      ```Java
      public synchronized void applyWithSyc(Long userId) {
          long count = couponRepository.count();
          if (count >= 100) {
              return;
          }
          couponRepository.save(new Coupon(userId));
      }
      ```
   2. lock or tryLock 사용  
      + tryLock을 사용시 lock을 획득 못하면 특정 값을 던져 대기하지 않아도 됩니다.  
      ```Java
      private final Lock lock = new ReentrantLock();
    
      public void applyWithLock(Long userId) {
          lock.lock();
          long count = couponRepository.count();
          try {
              if (count >= 100) {
                  return;
              }
              couponRepository.save(new Coupon(userId));
          } finally {
              lock.unlock();
          }
      } 
      ```
   3. 인메모리 DB (`Redis`)  
      **아래에 자세히 설명**
   
   + ![image_170.png](image_170.png)  
   + 어플리케이션 쪽에서 락을 사용하는 방법은 정말 권장하지 않습니다.
     1. 요청하나 처리하고 2000명이 대기하는 상황이 발생
     2. 선착순이 종료되어도 1900명이 해당 메서드의 락을 획득하기위해 경쟁이 발생
     3. 타임아웃을 설정해도 순간 톰캣이 커넥션의 수를 감당하기 어려움  
       

#### 데이터베이스 Lock 활용  
1. `for share` 또는 `for update`활용
   + for share - 읽기 잠금  
     읽기 잠금은 동시에 데이터를 읽기 가능, **동시에 쓰기 불가능**  
     동시에 쓰려고 할 때 하나의 트랜잭션만 적용 가능하거나 모두 실패합니다.
     ```Java
     @Transactional
     @Query(value = "SELECT COUNT(*) FROM coupon FOR SHARE", nativeQuery = true)
     long findByIdWithNativeRead();
     ```
   + for update - 쓰기 잠금  
     쓰기 잠금은 쓰기 잠금이 시작된 트랜잭션만 읽기, 쓰기가 가능합니다.  
     설정한 시간만큼 대기후 락을 획득하지 못할 경우 실패합니다.
     ```Java
     @Transactional
     @Query(value = "SELECT COUNT(*) FROM coupon FOR UPDATE", nativeQuery = true)
     long findByIdWithNative();
     ```   
   + ![image_173.png](image_173.png)  
   + ![image_174.png](image_174.png)  
   + `10,000`개 요청시 속도차이가 3초정도 발생합니다.
   + 낙관적(read) 락은 동시에 수정시 하나만 적용되거나 둘다 실패합니다. 
   + 비관적(write) 락은 동시에 수정시 하나만 적용되고 나머지는 대기합니다.  
   + 두 가지 방법 모두 `NO WAIT, SKIP LOCKED`을 활용하여 락을 획득 못하면 실패하는 옵션을 설정할 수 있습니다.  
   + 선착순 이벤트이기 때문에 비관적 락과 낙관적락은 요구사항에 맞지 않습니다. 
   
2. 접근제어격리 수준을 `Serializable`으로 변경   
   `@Transactional(isolation = Isolation.SERIALIZABLE)`    
   현재 하이버네이트에서 LockMode로 비관적락이 제대로 동작하지 않습니다.
   네이티브 쿼리를 사용해야 적용이 됩니다.  
  
3. 분산락 활용
   + 추후 추가 -  

#### 레디스 활용  
[레디스는 싱글스레드다](https://redis.io/docs/management/optimization/latency/)   
[레디스 멤캐시드 비교](https://deveric.tistory.com/65)   
[레디스 멤캐시드 비교 2](https://kinsta.com/blog/memcached-vs-redis/)    
  
> 레디스는 싱글 프로세스와 주로 싱글 스레드를 활용하는 디자인입니다. 
> 멀티플렉싱 기법을 사용하여 모든 클라이언트의 요청을 처리합니다.  
> 레디스 2.4부터 디스크 I/O와 관련된 느린 작업은 백그라운드에서 수행하기위해 스레드를 활용합니다.  
>   
  
인 메모리 디비는 `Redis`와 `Memcahed`가 있으며 모두 `key-value` 방식의 `No-SQL`입니다.  
  
현재 쿠폰 발행 프로젝트에 레디스를 선택한 이유는 다음과 같습니다. 
+ 데이터 복구  
  + 쿠폰 발행시 오류가 발생하여 데이터를 복구할 경우 스냅샷과 AOF 기능을 활용하여 멤케시드보다 데이터의 지속성과 내구성이 뛰어납니다.
+ 다양한 자료구조
  + 쿠폰의 요구사항이 변경되어 1회, 5회등 다양한 요구사항을 맞추기 위해 레디스가 제공하는 자료구조를 활용할 수 있습니다  
+ 서버를 확장할 경우
  + 자바의 기능인 동기화나 Lock는 많은 성능 저하가 발생하고 선착순에 적용하기 어렵다.
  + Atomic과 Concurrent는 빠르지만 확장성과 서버가 증가할 경우 확장이 어렵다.
  + MySQL은 과도한 트래픽이 발생할 경우 데이터베이스 커넥션으로 인해 다른 사용자가 서비스 이용이 어렵습니다.
  
대신 레디스는 물리 메모리 관리가 비효율적으로 사용할 수 있기에 메모리 모니터링이 필요합니다.
    
> 별도 설정으로 스냅샷 저장을 할 수 있습니다.  
> Redis Persistence에 대한 자세한 내용은 http://redis.io/topics/persistence를 참조하세요 .

#### 작업환경 세팅  
```Java
docker pull redis
docker run --name myredis -d -p 6379:6379 redis
```  
**사용할 명령어 incr에 대한 설명**  
레디스는 싱글 스레드로 동작하여 순차적으로 클라이언트의 요청을 처리합니다.  
`incr [key]`를 입력하면 응답값으로 해당 key의 `increment value`를 반환합니다.    
해당 명령어를 사용하여 선착순으로 100번 실행이되면 쿠폰 발행을 막는 로직을 작성할 수 있습니다.  

```Actionscript
docker exec -it myredis redis-cli
127.0.0.1:6379> incr coupon_sample
(integer) 1
127.0.0.1:6379> incr coupon_sample
(integer) 2
127.0.0.1:6379> incr coupon_sample
(integer) 3
127.0.0.1:6379> incr coupon_sample
(integer) 4
```  
  
  
스프링은 `Spring Data Redis`를 제공하여 쉽게 의존성을 추가할 수 있습니다.
```Gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```
`JdbcTemplate`과 유사한 `RedisTemplate`을 제공합니다.  
레디스를 사용하는 리포지토리를 생성합니다.   

```Java
@Repository
public class CouponCountRepository {  

    //스프링 부트가 기본 세팅하여 제공하는 인터페이스 
    private final RedisTemplate<String, String> redisTemplate;

    public CouponCountRepository(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
      
    // incr coupon_count 와 동일
    public Long increment() {
        return redisTemplate
                .opsForValue()
                .increment("coupon_count");
    }

    public void clear() {
        redisTemplate.delete("coupon_count");
    }
}
```    

**서비스 로직**
```Java
public void applyWithRedisIncr(Long userId) {
  Long count = couponCountRepository.increment();
  if (count > 100) {
      return;
  }
  couponRepository.save(new Coupon(userId));
}
```  

#### 추가 문제  
만약 발행하는 선착순 쿠폰이 `10,000`개, 동시 다발적으로 몇만명이 쿠폰 발급을 하기 위해 접속했습니다.  
  
`INSERT` 쿼리가 `10,000`개가 실행이 되어야할 때 
`INSERT` 쿼리 100개당 1분의 시간이 소요된다고 가정한다면 100분이 소요가 됩니다.  
  
그 사이에 다른 `회원가입`,`결제`등 요청이 들어온다면 애플리케이션이 서비스 이용이 불가능하게 됩니다.  
  
`10,000`개의 데이터를 한번에 저장하는 게 아니라 서비스 이용이 가능한 처리량 만큼 분할하여 처리하는 방법이 있습니다.  
  
대용량 스트리밍 데이터를 처리하기위해 선택할 수 잇는 미들웨어가 있습니다.  

1. Message Broker  
   + 대표: 레디스, 레빗엠큐
   + 메세지 브로커에 프로듀서가 전달하고 컨슈머가 받으면 즉시 또는 짧은 시간내에 삭제되는 구조 
   + 메세지 전처리 후처리를 해야함
  
2. Event Broker   
   + 키네시스, 카프카
   + 이벤트 또는 메세지라고 불리는 레코드를 하나만 보관하여 인덱스로 개별로 접근합니다.
   + 업무상 필요한 시간만큼 보관할 수 있습니다.
   + 단일 진실 공급원(SSOT)으로 모든 데이터 요소를 한 곳에서만 제어합니다.
   + 장애가 발생할 경우 해당 지점으로부터 재시작이 가능하다.
   + 많은 양의 스트림 데이터를 효과적으로 처리할 수 있습니다.  
   

[카카오 실시간 채팅 데이터 스트링 관리](https://kakaoentertainment-tech.tistory.com/109)  

이미 쿠폰 발행이 완료되어 이벤트는 종료되어 10,000명의 데이터가 유실없이 저장되어야합니다.  

1. 이벤트가 발행되어 쿠폰 결과나 쿠폰 발행 로그등 추가 작업을 해야한다.
2. 쿠폰 저장중 오류가 발생해도 재시작이 가능하다.
3. 많은 양의 데이터의 속도를 조절이나 분산할 수 있다.
4. 필요한 만큼 해당 정보를 저장하여 관리할 수 있다.  
    
**현재 상황에서는 다음과 같은 이유로 카프카를 사용했습니다.**

카프카는 레플리케이션을 통해 데이터의 안정성과 가용성을 보장합니다.  
  
**_레플리케이션은 주키퍼(Zookeeper)를 사용하여 관리됩니다._**  

레플리케이션은 각 토픽의 파티션을 여러 브로커에 복제하는 것을 의미합니다.  
예를 들어, 특정 토픽의 파티션은 리더(leader)와 여러 개의 팔로워(follower)로 구성됩니다. 
리더는 데이터를 실제로 쓰고 읽는 역할을 하며, 팔로워는 리더로부터 데이터를 복제하여 보관합니다.   

이런 방식으로, 주키퍼는 카프카 클러스터의 상태를 관리하고 클러스터 내의 각 브로커가 어떤 파티션의 리더인지, 어떤 브로커가 팔로워인지 등을 추적합니다. 주키퍼는 브로커의 메타데이터와 클러스터 구성 정보를 저장하고 클러스터의 일관성을 유지하는 데 사용됩니다.
따라서 카프카 브로커를 관리하고 클러스터의 안정성을 유지하기 위해 주키퍼를 활용하는 것은 일반적인 방법입니다. 주키퍼는 카프카 클러스터의 중앙 집중식 관리와 안정성을 보장하는 데 중요한 역할을 합니다.  
  
## 카프카   
  
![image_168.png](image_168.png)  

**도커 컴포즈 설정**
```Actionscript
# docker-compose.yml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka:2.12-2.5.0
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock  

# 도커 컴포즈 위치로 이동

# 실행        
docker-compose up -d

# 종료
docker-compose down
```
  
스프링 부트에서는 `Spring for Apache Kafka`을 활용하여 어노테이션으로 적용할 수 있습니다.    
주의사항은 이벤트 브로커와 이벤트 클라이언트(컨슈머,프로듀서) 버전이 호환성을 확인해야합니다.  
  
### 환경 구성  
  
**의존성 추가**  
```Actionscript
implementation 'org.springframework.kafka:spring-kafka'
```    
**토픽 생성**  
토픽은 SSOP를 보장하는 큐 구조이며 파티션으로 나눌 수 있습니다.  
```Docker
docker exec -it kafka kafka-topics.sh --bootstrap-server localhost:9092 --create --topic coupon_create
```  
**프로듀서 등록**  
```Java
docker exec -it kafka kafka-console-consumer.sh --topic coupon_create --bootstrap-server localhost:9092 --key-deserializer "org.apache.kafka.common.serialization.StringDeserializer" --value-deserializer "org.apache.kafka.common.serialization.LongDeserializer"
```  
**컨슈머 등록**  

```Java
docker exec -it kafka kafka-console-consumer.sh --topic coupon_create --bootstrap-server localhost:9092 --key-deserializer "org.apache.kafka.common.serialization.StringDeserializer" --value-deserializer "org.apache.kafka.common.serialization.LongDeserializer"
```
  
#### 컨슈머 인스턴스 설정
```Java
package org.example.api.config;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.LongSerializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

@Configuration
public class KafkaProducerConfig {

    /**
     * 프로듀서 인스턴스를 생성하는데 필요한 설정을 해야합니다.
     */
    @Bean
    public ProducerFactory<String, Long> producerFactory() {
        Map<String, Object> config = new HashMap<>();

        /**
         * 부트스트랩 서버설정을 로컬 호스트의 카프카를바라보도록 합니다.
         * 카프카 브로커 주소목록은 되도록이면 2개 이상의 ip와 port를 설정하도록 권장합니다.
         * 하나의 브로커가 비정상일경우 다른 정상적인 브로커와 연결이 가능하기 때문이다.
         * 실제로 애플리케이션에서는 반드시 2개이상의 브로커 정보를 넣는것을 추천한다.
         *
         * 키는 메세지를 보내면, 토픽의 파티션이 지정될때 사용됩니다.
         * 카프카 클라이언트에서 ProducerRecord 클래스를 지원합니다.
         */
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
        // 컨슈머는 데이터 -> 바이너리로 직렬화하기 위해 설정을 추가합니다.
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, LongSerializer.class);

        return new DefaultKafkaProducerFactory<>(config);
    }

    /**
     * 카프카 토픽에 메세지를 전송할 템플릿 만들기
     */
    @Bean
    public KafkaTemplate<String, Long> kafkaTemplate() {
        // 카프카팩토리를 만들때 위에서 저장한 카프카 팩토리를 사용한다
        // 구조가 JdbcTemplate 과 구조가 유사하다.
        return new KafkaTemplate<>(producerFactory());
    }
}
```  
  
### 프로듀서 등록
```Java
package org.example.api.producer;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class CouponCreateProducer {

    private final KafkaTemplate<String, Long> kafkaTemplate;

    public CouponCreateProducer(KafkaTemplate<String, Long> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    // 유저의 아이디를 전달할거기때문에
    public void create(Long userId) {
        // 토픽에 유저의 아이디를 전달하기때문에 유저의 아이디를 매개변수로 가지는 메소드를 하나 생성합니다.
        // 프로듀서 레코드의 인스턴스를 생성할 때 어느 토픽에 넣을 것인지, 어떤 키와 값을 담을 것인지 선언할 수 있습니다.

        // 현재는 coupon_create 토픽에 userId를 value 로 넣고 있습니다.
        kafkaTemplate.send("coupon_create", userId);
//      kafkaTemplate.send("coupon_create", "1",userId);
        /**
         * 위 코드처럼 키를 추가할 수 있습니다.
         * 파라미터 갯수에 따라 자동으로 오버로딩되어 인스턴스가 생성됩니다
         * 데이터가 도착할 토픽과 카프카 브로커의 호스트와 포트까지 지정했합니다.
         *
         * SEND 메서드에 파라미터로 프로듀서 레코드를 넣으면 전송이 됩니다.
         * 카프카는 이벤트 스트림인가
         *
         * 키는 파티션을 구분하는 값으로 활용됩니다.
         * 현재는 key = null인 파티션이 1개인 토픽에 전송하면 1개의 큐에 저장하게 되며
         * 파티션이 여러개일경우 라운드 로빈(번갈아가면서) 저장됩니다.
         */
    }
}  
```  
  
카프카를 애노테이션으로 사용하기전에 애노테이션을 사용하지 않고 받아보는 코드를 보면 이해하기가 쉽습니다.  
  
```Java
@Component
public class CouponCreateConsumer {

    private final ConcurrentKafkaListenerContainerFactory<String,Long> factory;

    @PostConstruct
    public void polling(){
        Consumer<? super String, ? super Long> consumer = factory.getConsumerFactory().createConsumer("group_1");
        consumer.subscribe(Arrays.asList("coupon_partition"));
        while (true) {
            long start = System.nanoTime();
            ConsumerRecords<? super String, ? super Long> records = consumer.poll(Duration.ofMillis(500));
            for (ConsumerRecord<? super String, ? super Long> consumerRecord : records) {
                System.out.println("consumerRecord = " + consumerRecord.value());
            }
            long end = System.nanoTime();
            long elapsedTimeInNanos = end - start;
            long elapsedTimeInMillis = elapsedTimeInNanos / 1_000_000; // 나노초를 밀리초로 변환
            logger.info("poll Diff = {} milliseconds", elapsedTimeInMillis);
        }
    }
    
}
```  
로그값을 확인해보면 500 밀리세컨즈동안 파티션에 등록된 데이터를 읽어오는 것을 확인할 수 있습니다.
```Java
CouponCreateConsumer : poll Diff = 506 milliseconds
CouponCreateConsumer : poll Diff = 508 milliseconds
CouponCreateConsumer : poll Diff = 516 milliseconds
CouponCreateConsumer : poll Diff = 511 milliseconds
CouponCreateConsumer : poll Diff = 515 milliseconds
CouponCreateConsumer : poll Diff = 515 milliseconds
```

> 라운드 로빈  
> 여러 대상 사이에 순차적으로 요청이 분배되는 로드 밸런싱 알고리즘  
>   
  
### 파티션 나누기
```Docker
docker exec -it kafka kafka-topics.sh --bootstrap-server localhost:9092 --create --topic coupon_partition --partitions 2
```   

`application.yml`을 통해서 설정을 추가할 수 있습니다.  
```Java
spring:
  kafka:
    producer:
      batch-size: 1
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.LongSerializer
```

![image_169.png](image_169.png)  

프로듀서가 파티션 키 값을 입력하지 않으면 라운드 로빈으로 파티션에 레코드를 보내는데, 
최신 버전은 레코드당 라운드 로빈을 처리하지 않고 지정한 배치의 수로 라운드 처리하게 됩니다.  
  

  

  
