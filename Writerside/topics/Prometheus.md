# Prometheus
  
## 설치 및 실행  
### 설치 경로
+ [공식 사이트](https://prometheus.io/download/)  
+ 윈도우 사용자 - `window-amd64`
  + `prometheus.exe` 실행
+ MAC 사용자 - `darwin-amd64`
  + `./prometheus`
  
## 아키텍처  
![image_201.png](image_201.png)  
  
프로메테우스는 설정된 시간 간격으로 `HTTP` 요청을 통해 지정된 인스턴스의 메트릭 정보를 수집합니다.  
이 메트릭 정보는 스프링 부트 애플리케이션 또는 다른 서비스에서 생성되고 노출되는 정보입니다.  
  
프로메테우스가 가져온 메트릭 정보는 `PromQL`이라는 질의 언어를 사용하여 사용자는 메트릭 데이터를 시각화하거나
분석하여 원하는 형태로 사용할 수 있습니다.  
  
스프링 부트에서 프로메테우스를 사용하려면 2가지 작업이 필요합니다.  
  
### 준비
1. 애플리케이션 설정: 프로메테우스가 애플리케이션의 메트릭을 가져갈 수 있도록 애플리케이션에서 프로메테우스 포멧에 맞춰 메트릭 만들기
2. 프로메테우스 설정: 프로메테우스가 애플리케이션의 메트릭을 주기적으로 수집하도록 설정  
  
### 애플리케이션 설정  
프로메테우스는 `json` 포멧은 이해하지 못합니다. 
마이크로미터 프로메테우스 구현체를 사용하면 Json 정보를 프로메테우스 포멧에 맞춰 변경합니다.  
  
**build.gradle추가**  
```Gradle
implementation 'io.micrometer:micrometer-registry-prometheus'
```  
+ 마이크로미터 프로메테우스 구현 라이브러리를 추가한다.
+ 스프링 부트와 액츄에이터가 자동으로 마이크로미터 프로메테우스 구현체를 빈으로 등록한다.
+ 액츄에이터 프로메테우스 메트릭 수집 엔트포인트가 자동으로 추가됩니다.
  + `/actuator/prometheus`  
  
**실행**  
```text
http://localhost:8080/actuator/prometheus
```  
  
**결과**  
```text
# HELP tomcat_threads_config_max_threads
# TYPE tomcat_threads_config_max_threads gauge
tomcat_threads_config_max_threads{name="http-nio-8080",} 200.0
# HELP tomcat_sessions_alive_max_seconds
# TYPE tomcat_sessions_alive_max_seconds gauge
tomcat_sessions_alive_max_seconds 0.0
# HELP tomcat_cache_access_total
# TYPE tomcat_cache_access_total counter
tomcat_cache_access_total 0.0
# HELP jvm_info JVM version info
# TYPE jvm_info gauge
```  
  
### 포멧차이  
+ 프로메테우스는 `.` 대신 `_` 포멧을 사용합니다.
  + `jvm.info` > `jvm_info`
+ 누적으로 증가하는 숫자 메트릭은 카운터라고 합니다
  + `logback.events` > `logback_events_total` 카운터 메트릭은 `total`이 관례로 붙는다
+ `http.server.requests` 이 메트릭은 내부에 추가 정보가 있습니다.
  + `http_server_requests_seconds_count` : 요청 수
  + `http_server_requests_seconds_sum` : 시간 합(요청수의 시간을 합함)
  + `http_server_requests_seconds_max` : 최대 시간(가장 오래걸린 요청 시간)
    + **단) 일정 시간(`1~3`분) 내에서 max를 구합니다.**
    + 이유는 제일 느린것만 가지고 있다면 의미없는 지표가 될 수 있습니다.  
  
## 수집설정  
프로메테우스 폴더에 있는 `prometheus.yml` 파일을 수정하여 
프로메테우스의 `pull` 의 인터벌 시간을 설정할 수 있습니다.  
  
```yaml
global:
 scrape_interval: 15s
 evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
    - targets:
      # - alertmanager:9093
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
    - targets: ["localhost:9090"]
#추가
  - job_name: "spring-actuator"
    metrics_path: '/actuator/prometheus'
    scrape_interval: 1s
    static_configs:
      - targets: ['localhost:8080']
```  
+ `job_name`: 수집하는 이름, 임의의 이름을 설정하여 사용
  + 어떤 일을 수행하는 프로세스로 백그라운드에서 실행됩니다.
+ `metrics_path`: 수집할 경로를 지정합니다.  
+ `scrape_interval`: 수집할 주기를 설정합니다.
+ `targets`: 수집할 서버의 IP,PORT를 지정합니다.  
  
> 주의사항  
> 수집 주기가 짧을 애플리케이션 성능에 영향을 줄 수 있습니다. 
> 운영에서는 `10s~1m` 정도를 권장합니다. (시스템 상황에 따라서 다릅니다)  
>   
  
**_설정이 종료되면 프로메테우스 서버를 종료하고 다시 실행합니다._**  
  
## 프로메테우스 연동 확인     
```yaml
- job_name: "prometheus"
  static_configs:
  - targets: ["localhost:9090"]
```  
프로메테우스 `Http` 주소로 접속하여 프로메테우스 상황을 GUI로 확인할 수 있습니다.

![image_204.png](image_204.png)   

**Configuration**에서 `prometheus.yml`에 입력한 부분 추가 확인
+ `http://localhost:9090/config`  
  
**Targets**에서 연동이 잘 되었는지 확인  
+ `http://localhost:9090/targets`

![image_203.png](image_203.png)  


+ `prometheus` : 프로메테우스 자체에서 제공하는 메트릭 정보이다. (프로메테우스가 프로메테우스 자신의 메트릭을 확인하는 것이다.)
+ `spring-actuator` : 우리가 연동한 애플리케이션의 메트릭 정보이다.
+ State 가 **UP** 으로 되어 있으면 정상이고, **DOWN** 으로 되어 있으면 연동이 안된 것이다.
  
## PromQL  
![image_205.png](image_205.png)  
  
+ **태그,레이블**: `error , exception , instance , job , method , outcome , status , uri` 등은 메트릭 정보를 구분하여 사용하기 위한 태그로 
  마이크로미터에서는 `tag`, 프로메테우스는 `lable`이라 합니다.
+ 숫자 : 1,2,4,11977 은 해당 메트릭의 값입니다.  
  
## 기본 기능  
+ Table : `Evaluation time`을 수정하여 과거 시간 조회 가능
+ Graph : 메트릭을 그래프로 조회 가능
  
### 필터  
레이블을 기준으로 필터를 적용할 수 있습니다. 필터는 중괄호 `{}` 문법을 사용합니다.  
  
#### 레이블 일치 연산자
`logback_events_total`을 기준으로 설명합니다.  

조건 `level`이 `info`인 경우  
+ `=` 제공된 문자열과 정확히 동일한 레이블 선택
  + `logback_events_total{level='info'}`
+ `!=` 제공된 문자열과 같지 않은 레이블 선택
  + `logback_events_total{level!='info'}`
+ `=~` 제공된 문자열과 정규식이 일치하는 레이블 선택
  + `logback_events_total{level=~'in.*'}`
+ `!~` 제공된 문자열과 정규식이 일치하지 않은 레이블 선택  
  + `logback_events_total{level!~'in.*'}`  
  
## 연산자  
+ `+` (덧셈)
+ `-` (빼기)
+ `*` (곱셈)
+ `/` (분할)
+ `%` (모듈로)
+ `^` (승수/지수)
  
#### sum
값의 합계를 구합니다.
`sum(logback_events_total{level=~'info|error'})`  
  
#### sum by
`SQL`의 group by 기능과 유사합니다.  
`sum by(method, status)(http_server_requests_seconds_count)`  
```text
{method="GET", status="404"} 1
{method="GET", status="200"} 13750
```  
  
#### count
메트릭 자체의 수 카운트  
`count(logback_events_total{level=~'info|error'})`  
```text
2
```  
메트릭 종류가 `error`과`info`만 존재하기 때문입니다.  
  
#### topk
`topk(3, http_server_requests_seconds_count)`  
상위 3개 메트릭 조회  
  
#### 오프셋 수정자
`http_server_requests_seconds_count offset 1h`
offset 1h 과 같이 나타낸다. 현재를 기준으로 특정 과거 시점의 데이터를 반환한다.
  
#### 범위 벡터 선택기
`logback_events_total{level=~'info|error'}[1m]`  
마지막 인수로 `[1m]`,`[60s]`와 같이 표현합니다. 지난 1분간 모든 값을 기록합니다.  
이때 요청에 대한 기록이 나열됩니다.  

![image_206.png](image_206.png)  
  
**범위 벡터 선택기는 차트에 바로 표현할 수 없습니다.** 데이터로 확인이 가능하며, 
차트에 표현하기 위해서 약간의 가공이 필요합니다.  
  
## 게이지와 카운터  
메트릭은 크게 보면 게이지와 카운터로 분류합니다.  
  
### 게이지(Gauge)
+ 증감이 발생하여 기록하는 현재의 속성 값
+ 예) CPU 사용량 , 메모리 사용량, 사용중인 커넥션
+ 비즈니스 로직 예) 재고, 기간별 회원 가입/탈퇴 수  
  
**증감의 값을 그대로 차트에 표현하면 된다**  
  
![image_207.png](image_207.png)  
  
### 카운터(Counter)  
+ 단순한 누적량을 기록한 속성 값
+ 예) HTTP 요청 수, logback error 수량  
+ 비즈니스 로직) 회원 가입 수, 당일 거래 횟수  
  
![image_208.png](image_208.png)  
  
계속 증가하는 그래프는 정보로 활용하기 어렵기 때문에 가공이 필요하며, 
프로메테우스는 가공 함수를 제공합니다.  
  
#### increase()  
지정한 시간 단위별로 증가를 확인할 수 있습니다.  
마지막에 `[시간]`을 사용하여 범위 벡터를 선택해야 합니다.  
+ increase(http_server_requests_seconds_count{uri="/log"}[1m])
![image_209.png](image_209.png)  
   
#### rate()
범위 벡터에서 초당 평균 증가율을 계산합니다.  
`increase()`는 숫자를 직접 카운트한다면,`rate()`는 여기에 초당 평균을 나누어 계산합니다.  
`rate(data[1m])`은 `[1m]`이라고 하면 60초가 기준이 되므로 기준 값에 60을 나눈 수입니다.
+ `rate(logback_events_total{level='info'}[1m])`  
1분 간격으로 초당 얼마큼 증가가 되는지 확인할 수 있습니다  
  
#### irate()  
`rate()`와 유사하지만, 초당 순간 등가율을 계산합니다. 급격하게 증가한 내용을 확인하기 좋습니다.
+ `irate(logback_events_total{level='info'}[1m])`  
  
![image_210.png](image_210.png)  
  
+ `11:08:00`~ `11:13:00` 까지 9 정도의 빠른 요청이 있다는 걸 알 수 있습니다.  
  
## 정리  
게이지는 메트릭 정보를 그대로 그래프에 사용해도 정보로 가치가 있습니다. 
하지만 카운터는 가치가 있는 정보로 활용하려면 가공이 필용합니다.  

### 한계  
프로메테우스는 `100%`로 일치하는 메트릭을 제공하지 않습니다. 실제 정확한 정보를 확인하려면 
직접 쿼리를 작성해야합니다.  
  
+ [프로메테우스 Query Doc](https://prometheus.io/docs/prometheus/latest/querying/basics/)  
