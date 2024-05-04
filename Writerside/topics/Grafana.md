# Grafana
  
그라파나는 데이터를 시각화하는 프로그램입니다.  

## 사용 목적
+ **데이터베이스의 불필요성**: Grafana는 별도의 데이터베이스가 필요 없이 기존 데이터를 통합하여 정보를 제공합니다.
+ **데이터 접근성**: 사용자가 데이터 조작 방법을 몰라도 시각화된 자료를 통해 쉽게 데이터를 분석할 수 있습니다.
+ **유연성**: 다양한 대시보드로 변환하여 사용자가 원하는 데이터 형식으로 표현할 수 있습니다.  
  
## 설치 및 실행
+ [그라파나 다운로드](https://grafana.com/grafana/download)  
  
**윈도우**  
+ 압축을 풀거나 `msi`를 실행후 `bin` 폴더로 이동후 `grafana-server.exe` 실행
  
**MAC**
+ 압축을 풀거나 `msi`를 실행후 `bin` 폴더로 이동후 `./grafana-server`  

### 실행
기본 포트가 `3000`이므로 `localhost:3000`으로 접속합니다.  
+ email or username: admin
+ Password: admin 
  
## 연동  
![image_211.png](image_211.png)   
  
그라파나는 프로메테우스에 `PromQL`을 인터벌로 조회하여 결과를 시각화합니다.  
  
![image_212.png](image_212.png)  
  
**연동방법**  
+ `Home` > `DataSource` > `+ Add new data source` > `prometheus` 선택
+ `Connection`:`localhost:9090` 프로메테우스 주소 입력
+ `Save & test` 클릭  
  
## 대시보드 활용   
그라파나 대시보드를 사용하기 전에 아래 사항을 확인합니다.
1. 애플리케이션 실행
2. 프로메테우스 실행
3. 그라파나 실행  
  
### 대시보드 선택
![image_213.png](image_213.png)  
  
+ `Dashboards` > `New` > `New Dashboard` 선택시 대시보드를 생성
+ `Dashboards` > `New` > `Import` 선택시 다른 사람의 대시보드를 불러옵니다.  
  
#### 대시보드 생성  
![image_214.png](image_214.png)

1. `New DashBoard`를 선택후 `+ Add visualization`을 선택합니다.  
2. 팝업 창을 끕니다.
  
+ **오른쪽 툴바**
  + Title : 제목
  + Description : 설명
  + Standard options 
    + Unit - 그래프 데이터 사이즈 변경
    + Min - 최소값 변경
+ **아래쪽 푸터**
  + `Enter a PromQL query`에 프로메테우스에서 적용햇던 문법 작성
  + `Run queries` 실행 
  + `Apply` 적용
  + `Option`에서 `Legend:Auto` > `Custom`으로 수정
    + 범례의 이름이 변경됩니다.

#### 대시보드 불러오기
+ [그라파나 대시보드](https://grafana.com/grafana/dashboards/)

1. 왼쪽 Dashboards 메뉴 선택
2. New 버튼 선택 Import 선택
3. 불러올 대시보드 숫자( id:`4658` )를 입력하고 Load 버튼 선택
4. Prometheus 데이터소스를 선택하고 Import 버튼 선택  
  
## 문제 예시
![jvm-heap.png](jvm-heap.png)  
  
+ `Heap Memory`에 메모리 부족으로 `OOM` 동작하기 전 모습  
  