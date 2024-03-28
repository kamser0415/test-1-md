# 기술력

## 꾸준히 수준 높은 해결책을 만들어 내는 능력
> 문제 인식부터 문제 해결을 시작하는 것이다.  
  
문제가 발생한 근본적인 원인을 파악하고 해결하는 문제의 범위(Scope)를 정확하게 정리해야 적절한 해결책을 만들어 낼 수 있습니다.   

문제에 대한 **해결책을 제안서로 작성**합니다.  
  
### 제안서  
+ 문제의 원인
+ 해결 방법 1 장/단점
+ 해결 방법 2 장/단덤
+ 해결 방법 3 장/단점  
+ 해결 방법 분석  
+ 최종 의견 제안   
  
**ppt** 가 아니라 단순한 텍스트 워드 한페이지 정도로 정리합니다.  
  
해결 방법은 해결책들에 필요한 리소스나 시간, 단기/장기 솔루션인지 작성합니다. 
간략하게 비교하고 그 해결책마다 장단점을 작성하고, 장단점에는 오래걸리지만 원인을 제거할 수 있다 등 
그 방식을 구현하기 위해서 필요한 운영 비용이나, 다른 비용은 어떤게 발생하는지 등을 정리하는 것을말합니다.  
  
해결 방법 1,2,3중 하나만 사용하는 것이 아닙니다.  
임시 방편인 `Mitigation`만 가지고 해결하는 것도 아니고, 근본적인 원인을 찾아 수정하는 것만 옳은 것도 아닙니다.  
  
다양한 해결책들의 비용과 리소스를 고려해 현재 주어진 상황에 가장 적절한 해결책을 찾는 것이 중요합니다. 
때로는 단기적으로 임시방편에 해당하는 해결책을 선택하고 중/장기적으로 리소스와 비용이 크지만 보다나은 해결책을 선택할 수도 있습니다.  
  
> 임시방편과 발생 원인 수정은 반드시 같이 수행해야합니다.  
   
예시 상황)  
배포를 하다가 소스코드에 오류가 발생했다.  
서버가 전체에 한번에 배포되는 방식이 아니라 점차 배포 서버의 영역을 넗힌다.  
1. 이전 버전으로 돌린다.
2. 근본적인 원인을 찾아서 수정한다.
3. 배포를 일시적으로 중지한다.  
  
모든 방법을 사용하여 오류를 막습니다. 
1. 오류를 가진 배포 버전이 확산되지 않도록 먼저 배포를 일시적으로 중지합니다.
2. 배포된 서버는 이전 버전으로 돌립니다.
3. 근본적인 원인을 찾아서 수정합니다.   
  
소스코드에 문제가 있기 때문에 배포팀에 문제가 없다고 생각하는 것보다 
PR전에 확인 할 수 있는 방법이 있는지, 배포하기 전에 오류가 발생하는 것을 미리 확인하는 로직이 잘못된 것인지 
그게 수정되었다면 잘못된 소스코드가 들어와도 배포되기 전에 막을 수 있었습니다.  
  
버그를 수정하는 것과 더불어 버그가 발견되지 못했는지, 그러한 시스템을 갖추지 못했는지에 대한 생각까지 해야합니다.  
  
## 엔지니어링 원칙과 업계 베스트 프랙티스를 받아들이기  
엔지니어링 원칙은 학습하는 것보다 적용하는 것이 어렵습니다.  
  
레거시 코드를 수정할 때 어떤 오류가 날지 무서워서 배포나 코드 수정하기가 무서운 경우에는 
리팩토링을 해야합니다.  
  
> 리팩토링을 할 때, 테스트 코드가 없다면 테스트 코드를 먼저 작성합니다.  
  
베스트 프렉티스와 트랜드와 헷갈리면 안됩니다. 
단순하게 많이 써서 사용하는 게 아니라 현재 기술의 한계를 다른 기술을 적용하여 해결할 때 
그 기술을 접목 시켜 사용해야합니다.   
  
### 엔지니어링 원칙
+ Don't Repeat Yourself. : 반복되는 소스 코드를 작성하지 않습니다.
+ Keep It Simple Stupid. : 함수는 단순하고 여러 일을 할 수있는 똑똑한 함수가 되면안됩니다.
+ You Aren't Gonna Need It. : 미리 만들어서 하지 않습니다.
+ SOLID 디자인 원칙
  
### 안티 패턴
+ 오버 엔지니어링
+ 이른 성능 최적화
  
### 베스트 프렉티스 
+ 테스트 작성하기
+ 리팩토링
+ 클린 코드  
  
   
##  개발 방법과 품질을 높이는 방법  
소프트웨어 개발 프로세스 전반 걸쳐서 (설계,코딩,빌드,배포 등) 프로세스를 더 견고하고, 빠르고, 유연하게 개선할 수 있는 방업을 찾아 개선합니다.  
  
자바, 스프링을 잘 아는 것은 개발 영역중 일부입니다.
개발 프로세스에서 병목이 생기는 지점이 어디인지 파악하고,
개발 프로세스 플로우가 순조롭지 못한지 빠르게 새로운 기능을 개발해서 배포했을 때 왜 적용을 하지 못할까 고민을 하는 겁니다.  
  
품질 관련 측면에서 시스템에 장애가 났을 때 처리하고 끝내는게 아니라 
  
### 사후 리뷰 postmorterm
장애 사후 리뷰를 작성합니다.  
+ 장애 상세 정보
+ 장애 범위, 일시
+ 장애 처리방법,일시 -> 임시방편(`Mitigation`)은 어떤 것을 사용했는지
+ 장애 근본 원인
+ 장애 방지 대책: 동일한 문제가 발생하지 않기 위해 해야하는 일을 작성합니다.  
  
여러가지 해결책을 제안하고, 그 해결책들이 언제 적용될 수 있는지 파악합니다.
  
그 다음 문제는 이 문제를 애초에 발견하지 못한 이유를 꼼꼼히 따져보는 회의가 필요합니다.  
동일한 장애가 발생하지 않게 처리해야합니다.  
  
### 사전 리뷰 Premorterm  
사건이 발생하기 전에 이런 문제가 발생하면 이런 식으로 대응하고, 막으려면 이런한 시스템을 만들어야한다는 리뷰 입니다.  
  
> 중요한 건, 사건이 발생한 것처럼 리뷰를 작성하는 방식입니다.
> 특히 새로운 시스템을 개발할 때 필요합니다.  
  
## 기술 증진을 시키기 위한 방법을 찾는다.  
기술의 근본적인 자료(래퍼런스와 API)를 찾아서 학습할 수 있는 능력을 키워야 합니다.  
학습하여 이해한 것들은 반드시 코딩하며 검증합니다.  
  
**_특히, 가공된 자료를 통해 학습하는 경우 본인이 이해한 것이 사실인지 "근본적인 자료" 또는 "코딩"을 통해 확인하는 습관을 가지는 것이 좋습니다._**  
  
가공된 자료를 볼때 비판적인 사고로 정확한 자료인지 검증하며 확인합니다.  
  
그리고 영어가 익숙하지 않더라도 근본적인 래퍼런스 문서를 읽어야합니다. 
한 페이지를 읽는데 1시간이 걸려도 4~5년 뒤에는 10분 정도면 한 페이지를 읽을 수 있고, 
아는 내용이면 1~2분 정도면 읽을 수 있습니다.

## 동료 또는 다른 개발자의 기술력을 증진시키는 것을 돕기  

시간에 중요한 일인 긴급한 일이 아니라면, 동료들이 막혀있다면 그 문제를 풀어주는 역할을 하는 것이 중요합니다.  
  
**세미나 또는 컨퍼런스 발표하기**  
내가 공부한 것을 어떻게 전달해야하는지 연습하다보니 내용을 100%이해하게 된다.  
한시간 발표를 할 때 20번 정도 연습을 한다고 합니다. 
+ 청중을 보면서 잘 따라오는지 속도를 조절해야합니다.  
+ 다른 청중 앞에서 대면하는 것이 중요합니다.
  
이것보다 더 중요한건 팀원을 도와주는게 중요합니다.  
  
## 정리
**기술력을 증진시키는 방법**  
  
+ 꾸준히 수준 높은 해결책 만들기
+ 엔지니어링 원칙과 베스트 프렉티스 적용
+ 지속적인 개발 방법과 품질 개선
+ 기술력 증진에 필요한 학습
+ 동료 및 다른 개발자의 기술력 증진 돕기  
  
