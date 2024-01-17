# Junit_테스트하기 - 자동테스트

<procedure title="단위 테스트" id="unit-test-info">
    <step>  
        <p>작은 코드 단위를 독립적으로 검증하는 테스트</p>
        <p>ㄴ 클래스 or 매서드</p>
    </step>
    <step>
        <p>검증 속도가 빠르고 안정적이다.</p>
    </step>
</procedure>
  
> **작은** 코드 단위를 **독립적**으로 검증하는 테스트  
> 외부에 의존하지 않은 클래스 단위로 검증을 하기 때문에 검증속도가 빠르고, 
> 안정적인 테스트를 할 수 있습니다.  
> **단위 테스트부터 꼼꼼하게 잘 작성하는 것이 중요한 일입니다.** 

작은
: **작은 코드의 단위**라고 하면 일반적으로 클래스 혹은 메소드 단위를 이야기합니다.   
내가 만든 **클래스 하나**에 대해서 검증을 하거나 혹은 클래스 내에 있는 **메소드 단위**로 검증을 하는 것을 말합니다.
  
독립적
: 독립적이라는 의미는 외부 네트워크(`DB`)를 이용한 기능이 들어있지 않아야합니다.  
외부 상황에 의존하지 않은 테스트하는 클래스나 메소드만 검증할 수 있어야 합니다.


## JUnit5  
Junit5
: - 단위 테스트를 위한 테스트 프레임워크
  - XUnit - Kent Beck  
단위 테스트를 위한 테스트 프레임워크입니다.  
[Junit Doc](https://junit.org/junit5/)

## AssertJ
AssertJ
: - 테스트 코드 작성을 원할하게 돕는 테스트 라이브러리
- 풍부한 API, 메서드 체이닝 지원   
메소드 체이닝으로 깔끔한 테스트 코드 작성이 가능합니다.  
[AssertJ / Fluent assertions for java](https://joel-costigliola.github.io/assertj/index.html)  
  
### dependencies
```Gradle
dependencies {
    // Junit5 와 AssertJ가 같이 있는 boot-starter 입니다.
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
  
### 테스트 코드  

테스트 코드 목적
: 키오스크에 아메리카노를 주문과 취소시 담김 음료 수 확인
  
```Java
@Test
void remove(){
    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano = new Americano();

    cafeKiosk.add(americano);
    assertThat(cafeKiosk.getBeverages()).hasSize(1);

    cafeKiosk.remove(americano);
    assertThat(cafeKiosk.getBeverages()).hasSize(0);
    assertThat(cafeKiosk.getBeverages()).isEmpty();
}
```
> `Junit` 테스트 코드 작성시 최종 단계에서 사람이 확인할 필요가 없어졌습니다.  
> 결과만 통과했는지 체크만 하면 됩니다.  
> `단위 테스트`가 많이 작성이 된다면 실제 구현 내용이 변경되더라도 지속적으로 
> 테스트 코드를 수행한다면 `프로덕션 코드가` 정상 동작하고 있는지 수시로 체크할 수 있습니다.