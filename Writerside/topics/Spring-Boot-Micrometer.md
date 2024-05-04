# 마이크로미터 라이브러리
  
모니터링 툴을 사용하는 이유는 
문제를 식별하고 대응하기 위한 중요한 도구입니다. 
시스템이나 네트워크에서 발생하는 문제를 빠르게 감지하여 조취를 취함으로써 
문제를 예방하고 시스템을 안정적으로 운영하는데 도움이 됩니다.  
  
모니터링 툴마다 사용하는 데이터 구조가 다를 수 있습니다. 
만약 `A 모니터링 툴`을 사용하다가 `B 모니터링 툴`로 변경해야한다면 
측정하는 코드까지 모두 변경해야 하는 문제가 발생됩니다.  
  
## 추상화  
마이크로미터는 애플리케이션 파사드로 불리며,
`jdbc`와 `DataSource` 처럼 추상화를 통해 
다양한 모니터링 지표를 마이크로 미터가 정한 표준 방법으로 작성합니다.  
  
마이크로미터는 사용하는 모니터링 툴에 맞게 지표를 변환하여 전달합니다.  
    
[마이크로미터 docs](https://micrometer.io/docs/)  
  
## 메트릭 목록 확인
스프링 액츄에이터를 사용하면 마이크로미터에서 사용되는 메트릭 포멧에 맞게 자동으로 지표 수집용 빈을 `@AutoConfiguration`을 등록합니다.  
   
### 엔드포인트 확인 방법
`http://localhost:8080/actuator/metrics`  
  
```json
{
  "names": [
    "application.ready.time",
    "application.started.time",
    "disk.free",
    "disk.total",
    "executor.active",
    "executor.completed",
    "executor.pool.core",
    "executor.pool.max",
    "executor.pool.size",
    "jvm.info",
    "jvm.memory.committed",
    "jvm.memory.max",
    "jvm.memory.usage.after.gc",
    "jvm.memory.used",
    "jvm.threads.daemon",
    "jvm.threads.live",
    "jvm.threads.peak",
    "jvm.threads.states",
    "logback.events",
  ]
}
```  
  
### 상세보기
metrics 엔드포인트는 다음과 같은 패턴을 사용해서 더 자세히 확인할 수 있다.  
`http://localhost:8080/actuator/metrics/{name}`  

```json
// http://localhost:8080/actuator/metrics/jvm.memory.used
{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 131172848
    }
  ],
  "availableTags": [
    {
      "tag": "area",
      "values": [
        "heap",
        "nonheap"
      ]
    },
    {
      "tag": "id",
      "values": [
        "G1 Survivor Space",
        "Compressed Class Space",
        "Metaspace",
        "CodeCache",
        "G1 Old Gen",
        "G1 Eden Space"
      ]
    }
  ]
}
```  
+ 현재 메모리 사용량을 확인할 수 있습니다.  
  
### Tag 필터  
`availableTags`에 다음과 같은 항목이 있습니다.  
+ `tag:area,value[heap,nonheap]`  
  
해당 Tag를 기반으로 정보를 필터링해서 확인할 수 있습니다.  
`tag=KEY:VALUE`과 같은 형식을 사용합니다.  
+ `http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap`  
  
```json
{
  "name": "jvm.memory.used",
  "description": "The amount of used memory",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 56313920
    }
  ],
  "availableTags": [
    {
      "tag": "id",
      "values": [
        "G1 Survivor Space",
        "G1 Old Gen",
        "G1 Eden Space"
      ]
    }
  ]
}
```  
태그 필터는 `SQL`의 `WHERE`처럼 특정 엔드포인트(`FROM`)에 있는 특정 카테고리의 값을 읽어온다는 것과 유사합니다.  
+ `SELECT * FROM Member WHERE gender='F'`와 유사합니다.
  
#### 예제
tag 필터에 조건이 여러 개인 경우 사용하기 유용합니다.  
+ `tag=uri:/log&tag=status:200`  
  
검색 조건이 `tag = /log`와 `status:200` 을 만족하는 결과만 조회할 수 있습니다.  
```Java
/metrics/http.server.requests?tag=uri:/log&tag=status:200
```   

### 옵션 활성화  
톰캣 메트릭이 제공하는 정보를 모두 사용하려면 옵션을 활성화합니다.  
옵션을 켜지 않는다면 `tomcat.session`만 노출됩니다.  
```yaml
server:
  tomcat:
    mbeanregistry:
      enabled: true
```  

## 지원하는 메트릭 정보
[Supported Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported)  
  

  
