# BDD 스타일로 작성하기  

<procedure title="BDD,Behavior Driven Development">
    <step>
    <p>TDD에서 파생된 개발 방법</p>
    </step>
    <step>
    <p>함수 단위에 테스트에 집중하기보다,함수가 어떤 역할을 해야하는지
    <code>특정 시나리오에 기반한 테스트케이스(TC) 자체</code>에 집중하여 테스트한다.</p>
    </step>
    <step>
    <p>개발자가 아닌 사람이 봐도 이해할 수 있을 정도의 추상화 수준(레벨)을 권장</p>
    </step>
</procedure>

## Given / When / Then

Given 
: 시나리오 진행에 필요한 모든 준비 과정(객체, 값, 상태 등) 테스트를 하기위한 모든 단계를 준비하는 과정
  
When 
: 테스트하려는 TC 시나리오를 진행을 시킵니다.

Then 
: TC 시나리오 진행에 대한 결과 명시 그리고 상태가 어떤식으로 변화가 됐는지 검증 `AssertThat` 등으로 검증

<procedure title="정리">
    <step>
    <p>어떤 환경에서(Given)</p>
    </step>
    <step>
    <p>어떤 환경에서(Given)</p>
    </step>
    <step>
    <p>어떤 행동을 진행했을 때(When)</p>
    </step>
</procedure>  

> `DisplayName`과도 연관이 되어서 테스트 내용을 정확히 문서로 남길 수 있습니다.
  
예시코드
```java
@Test
@DisplayName("주문목록에 담긴 음료의 총 금액이 일치하는지 확인한다.") -- 내가한건 테스트관점
@DisplayName("주문 목록에 담긴 상품들의 총 금액을 계산할수 있다.")-- 도메인 관점
void calculateTotalPrice() {
    //given : 테스트에 필요한 객체,값을 세팅하는 과정
    CafeKiosk cafeKiosk = new CafeKiosk();
    Americano americano = new Americano();
    Latte latte = new Latte();
    cafeKiosk.add(americano);
    cafeKiosk.add(latte);

    //when : 한줄인 경우가 많다, 수행하는 메서드를 실행
    int totalPrice = cafeKiosk.calculateTotalPrice();
 
    //then : 검증단계
    assertThat(totalPrice).isEqualTo(8500);
}
```