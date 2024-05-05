# metrics
메트릭은 일반적인 용어로 수치 측정값을 말합니다.  

사용자가 측정하고자 하는 대상마다 메트릭은 다를 수 있습니다.
예를 들어, 애플리케이션이 느리다는 것을 발견하고 애플리케이션에 무슨 일이 발생했는지 알아내기 위한 필요한 정보가 있습니다.  
1. 요청 수가 많은 경우
2. 다른 네트워크 통신이 원할하지 않는 경우
3. 특정 요청 시간이 길어진 경우  
  
등등 사용자는 매트릭을 사용하여 애플리케이션의 성능을 모니터링하고 문제를 파악하는데 도움을 줍니다.  
  
메트릭을 기록할때 나오는 용어로 `time series`와 함께 사용합니다. 
시계열(`time series`)이란 시간이 지남에 따라 변화를 기록하는 것을 말합니다.  
  
> 메트릭을 활용하여 애플리케이션의 성능이나 상태를 파악할 수 있습니다.  
>   
  
## 사용자 정의 메트릭  
기본적인 `CPU`,`JVM`,`Thread Pool`은 스프링 엑츄에이터 및 마이크로미터를 활용하여 
메트릭을 확인할 수 있습니다.  
  
비즈니스에 특화된 부분을 모니터링 하고 싶은 경우가 있습니다. 
예를 들어) 주문수, 취소수, 재고 수량같은 메트릭이 있습니다. 
필요한 이유는 애플리케이션 성능이나 동작에 이상은 없지만 갑자기 취소수가 증가하거나 
주문 수가 감소하는 등 기술적인 메트릭으로 확인할 수 없는 비즈니스 문제를 빠르게 파악할 수 있습니다.  
  
예를 들어 특정 지역 회원의 주문이 감소하여 확인해보니 눈이나 비로 인해 운송이 불가능한 경우 메모리 사용량이나 요청 응답시간에는 
지표로 확인할 수 없기 때문에 이런 비즈니스 메트릭을 활용하여 문제를 인식할 수 있습니다.  
  
### 서비스 코드  
메트릭을 적용할 서비스 코드입니다.  
  
**주문 서비스**  
```Java
@Slf4j
@Service
public class OrderService implements OrderService {

    private AtomicInteger stock = new AtomicInteger(100);

    @Override
    public void order() {
        log.info("주문");
        stock.decrementAndGet();
    }

    @Override
    public void cancel() {
        log.info("취소");
        stock.incrementAndGet();
    }

    @Override
    public AtomicInteger getStock() {
        return stock;
    }
}
```  
`AtomicInteger`는 데이터 수정시 원자성을 보장하는 클래스입니다. 
주문,취소가 동시에 발생할 경우 갱실 손실이 발생하지 않도록 보장합니다.

## MeterRegistry  
마이크로미터 기능을 제공하는 핵심 컴포넌트입니다.  
스프링을 통해서 주입 받아서 사용하고, 이곳을 통해서 카운터와 게이지등를 등록합니다.
  
동일한 ID를 가진 `Meter`를 여러번 등록하는 경우, 첫번째만 등록되고 나머지는 무시됩니다. 
복합키로 관리되는 값은 `name`과 `tag`이므로 카운터나 게이지에 등록할 때 주의해야합니다.  

## 카운터(Counter) 메트릭
카운터는 단일 단조 증가하는 **누적** 메트릭으로, 그 값은 오직 **증가**하거나 재시작 시 **0으로 재설정**될 수 있습니다.    
마이크로미터에서 감소하는 기능도 지원하지만, 감소한다면 게이지를 활용하는 것이 습니다.  
  
**예시**
+ `완료된 작업 수`: 배치 작업이나 스케줄된 작업이 완료될 때마다 카운터를 증가시켜 작업의 성공적인 완료를 모니터링합니다.
+ `발생한 오류 수`: 애플리케이션에서 발생하는 오류를 추적하여 시스템의 안정성을 평가합니다.
+ `사용자 로그인 횟수`: 사용자가 시스템에 로그인할 때마다 카운터를 증가시켜 사용자 활동을 분석합니다.
+ `파일 다운로드 수`: 사용자가 파일을 다운로드할 때마다 카운터를 증가시켜 인기 있는 콘텐츠를 파악합니다.  
  
프로메테우스에서는 일반적으로 카운터의 이름 마지막에 `_total`을 붙여 표기합니다.  
  
누적 메트릭이기 때문에 기간과 같이 사용하여 정보로 활용합니다.  
  
### 프로그래밍 방식  
```Java
@Slf4j
@Service
public class OrderService implements OrderService {

    private final MeterRegistry meterRegistry;

    private AtomicInteger stock = new AtomicInteger(100);

    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void order() {
        log.info("주문");
        stock.decrementAndGet();

        Counter.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "order")
                .description("order")
                .register(meterRegistry)
                .increment();
    }

    public void cancel() {
        log.info("취소");
        stock.incrementAndGet();
        //태그로 구분하기 때문이다.
        Counter.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "cancel")
                .description("order")
                .register(meterRegistry)
                .increment();
    }

    @Override
    public AtomicInteger getStock() {
        return stock;
    }
}
```  
#### 동작 방식  
1. `new Counter(name,tags)` 생성
2. `meterRegistry`에 등록 (최초 1회 Map 구조)
3. 없는 경우 등록, 있는 경우 기존꺼 반환
4. `increment()` 실행  
  
해당 코드를 아래와 같이 변경도 가능합니다.  
```Java
meterRegistry.counter("my.order", Tags.of("class", this.getClass().getName(),"method","order")).increment();
// 내부 동작코드
public Counter counter(String name, Iterable<Tag> tags) {
    return Counter.builder(name).tags(tags).register(this);
}
```    
내부 동작 코드는 동일합니다.  

### 선언적 방식   
```Java
@Slf4j
public class OrderServiceV2 implements OrderService {

    private AtomicInteger stock = new AtomicInteger(100);

    @Override
    @Counted("my.order")
    public void order() {
        log.info("주문");
        stock.decrementAndGet();
    }

    @Override
    @Counted("my.order")
    public void cancel() {
        log.info("취소");
        stock.incrementAndGet();
    }

    @Override
    public AtomicInteger getStock() {
        return stock;
    }
}
```
AOP를 활용하여 적용하려는 메소드 레벨에 `@Counter`를 선언합니다.  
  
이때 AOP를 사용할 수 있게 `CountedAspect`를 빈으로 등록합니다.
```Java
@Configuration
public class CounterAespectConfig {
    @Bean
    public CountedAspect countedAspect(MeterRegistry meterRegistry) {
        //@Counter 동작하기 위해서 AOP 빈 등록
        return new CountedAspect(meterRegistry);
    }
}
```
`CountedAspect`클래스의 로직을 잠깐 확인하면 아래와 같이 태그를 대신 생성합니다. 
```Java
public CountedAspect(MeterRegistry registry, Predicate<ProceedingJoinPoint> shouldSkip) {
    this(registry, pjp -> Tags.of(
            // "class",this.getClass().getName();
            "class", pjp.getStaticPart().getSignature().getDeclaringTypeName(), 
            // "method","order" or "cancel";
            "method",pjp.getStaticPart().getSignature().getName()), shouldSkip);
}
```      

## 타이머(Timer) 메트릭  
타이머 메트릭은 대량의 짧은 실행 이벤트를 추적합니.  
예를 들면 `HTTP`요청 실행 시간이 될 수 있으며, 짧은 실행은 주관적이지만 1분 미만으로 가정합니다(공식문서 설명).  
  
카운터와 유사하며,`Timer`를 사용시 실행시간도 함께 측정 가능합니다.  
+ `seconds_count`: 누적 실행 수 - `카운터`
+ `seconds_sum`: 실생 시간의 합 - `sum`
+ `seconds_max`: 최대 실행시간 (가장 오래 걸린 실행 시간) - `게이지`
+ 내부에 타임 윈도우 개념이 있어 `1~3분` 마다 최대 실행 시간이 다시 계산됨   
  + ```Java
    public TimeWindowMax(Clock clock, DistributionStatisticConfig config) {
        this(clock, config.getExpiry().toMillis(), config.getBufferLength());
    }
    ```
  + ![image_216.png](image_216.png)
  
아무것도 설정하지 않을 경우 `1분`으로 설정되며 최대 3분까지 데이터를 보관할 수 있습니다.  
  
별도로 설정도 가능합니다.
```yaml
management:
  metrics:
    distribution:
      expiry:
        http: 2m
        server: 2m
        requests: 2m
```  
다만 스프링부트 `3.0.2` 버전에서 바인딩 오류가 있어서 `http.server`처럼 `.`가 있을 경우 바인딩이 안되고 있습니다.  
각각 구분자를 제외하고 입력하면 정상적으로 변경됩니다.  
  
### 프로그래밍 방법
```Java
@Slf4j
public class OrderService implements OrderService {

    private final MeterRegistry meterRegistry;

    private AtomicInteger stock = new AtomicInteger(100);

    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void order() {
        Timer timer = Timer.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "order")
                .description("order")
                .register(meterRegistry);

        timer.record(()->{
            log.info("주문");
            stock.decrementAndGet();
            sleep(500);
        });
    }

    public void cancel() {
        Timer timer = Timer.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "cancel")
                .description("cancel")
                .register(meterRegistry);

        timer.record(()->{
            log.info("취소");
            stock.incrementAndGet();
            sleep(200);
        });

    }

    public AtomicInteger getStock() {
        return stock;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis + new Random().nextInt(200));
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```  
`Timer` 인터페이스를 사용하는 방식으로 아래와 같이 짧게 작성도 가능합니다.
```Java
 meterRegistry.timer("my.order", 
              Tags.of("class",this.getClass().getName(),
                      "method","cancel"))
              .record(()->{
                  log.info("취소");
                  stock.incrementAndGet();
                  sleep(200);
              });
```  
타이머를 사용할때는 `record()`를 사용합니다.
다양한 람다함수를 사용할 수 있으며 시간을 측정할 함수를 포함하면 됩니다.  
  
`sleep()`을 추가한 이유는 `Timer`의 max를 활용하기 위해서 랜덤으로 설정합니다.

### 선언적 방법
```Java
@Slf4j
@Service
@Timed(value="my.order")
public class OrderService implements OrderService {

    private AtomicInteger stock = new AtomicInteger(100);
    
    public void order() {
        log.info("주문");
        stock.decrementAndGet();
        sleep(500);
    }
    public void cancel() {
        log.info("취소");
        stock.incrementAndGet();
        sleep(200);
    }
    public AtomicInteger getStock() {
        return stock;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis + new Random().nextInt(200));
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```  
`@Timed`를 적용할 AOP 클래스도 빈으로 등록합니다.
```Java
@Configuration
public class TimedAspectConfig {
    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }
}
```  
  
## 구현방식 비교
프로그래밍식으로 작성할 경우 비즈니스 로직에 추가되어 `AOP`를 통해 관심사를 분리할 수 있습니다.  
메서드나 클래스 단위가 아닌 특정 구역에 대한 메트릭을 기록하고 싶을때에는 프로그래밍식을 사용할 수 있습니다.  
  
## Gauge(게이지)
게이지는 임의로 상하로 움직일 수 있는 단일 숫자 값을 나타내는 메트릭입니다.  
+ 메트릭 측정 대상의 상태를 숫자로 확인 할 수 있습니다.
+ 값이 증가하거나 감소할 수 있음
+ 예) CPU 사용량, 메모리 사용량, 커넥션 수  
    
### 프로그래밍 방식
게이지를 활용하는 방법은 선언적 방식이 없습니다.  
사용 방법은 `MeterRegistry`에 등록할 마이크로미터 `Gauge`를 등록합니다. 
`name`과 `tag`를 추가하고 마이크로미터에서 `name`으로 조회할 경우 게이지로 측정할 메트릭 값을 반환하면 됩니다.  

[커스텀 메트릭 등록](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#actuator.metrics.registering-custom)  
  
**컴포넌트에서 `MeterRegistry`를 주입받아 사용하는 방법**
```Java
@Slf4j
@Service
public class OrderServiceV5 implements OrderService {

    private AtomicInteger stock = new AtomicInteger(100);

    public OrderServiceV5(MeterRegistry meterRegistry) {
        meterRegistry.gauge("my.stock",
                Tags.of("class", this.getClass().getName(),
                        "method", "stock"), this,
                service -> {
                    log.info("custom gauge");
                    return this.getStock().get();
                });
    }
    
    public AtomicInteger getStock() {
        return stock;
    }
}
```  
  
**MeterBinder 사용**
```Java
@Slf4j
@Configuration
public class StockMetricsConfig {

    private final String PREFIX = "my.";

    @Bean
    public MeterBinder queueSize(OrderServiceV0 orderServiceV0) {
        return (registry) -> Gauge.builder(PREFIX+"stock",
                        orderServiceV0::getStock)
                .register(registry);
    }
}
```  
#### MeterBinder 장점
1. `MeterBinder`를 사용하면 올바른 종속관계가 설정되며, 메트릭 값이 검색될때 빈이 사용되도록 보장합니다.
2. `MeterBinder`를 활용하면 여러 구성요소나애플리케이션 간에 반복적으로 등록해야하는 경우 활용할 수있습니다.  
  
```Java
@Component
public class CustomOrderMeterBinder implements MeterBinder {

    private final Tags tag;
    private final OrderService orderService;

    public CustomOrderMeterBinder(OrderService orderService) {
        this.tag = Tags.of("class", this.getClass().getName());
        this.orderService = orderService;
    }

    @Override
    public void bindTo(MeterRegistry registry) {
        Gauge.builder("my.stock", orderService,
                        service -> service.getStock().get())
                .tags(tag).description("주문 재고수량")
                .register(registry);
        Gauge.builder("my.stock1", orderService,
                        service -> service.getStock().get())
                .tags(tag).description("주문 재고수량1")
                .register(registry);
        Gauge.builder("my.stock2", orderService,
                        service -> service.getStock().get())
                .tags(tag).description("주문 재고수량2")
                .register(registry);
        Gauge.builder("my.stock3", orderService,
                        service -> service.getStock().get())
                .tags(tag).description("주문 재고수량3")
                .register(registry);
    }
}
```  

## 정리
메트릭은 100% 정확한 숫자를 보는데 사용하는 것이아닙니다. 