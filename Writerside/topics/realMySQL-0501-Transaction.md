# 트랜잭션

트랜잭션은 논리적인 작업 단위가 모두 적용되거나 적용되지 않도록 보장해주는 것을 말합니다. 
트랜잭션이 없다면 논리적인 작업 단위중 오류가 발생하면 전체 작업을 원 상태로 복구되지 않고 
일부 작업만 적용되는 `Partial Update(부분 업데이트)`로 인해 데이터 정합성이 깨지게 됩니다.  
  
데이터 정합성이 깨진다면 데이터는 신뢰할 수 없는 정보가 되어 사용할 수 없습니다. 
  
트랜잭션 기능을 지원하지 않는다면 `IF ELSE` 구문으로 실패할 경우 클렌징 코드까지 추가해야합니다.

```SQL
try{
    START TRANSACTION;
    INSERT INTO member_;
    INSERT INTO bank_;
    COMMIT;
} catch(exception e){
    ROLLBACK;
}
```  
트랜잭션 시작 명령어와 예외 없이 동작한 경우 `COMMIT`을 사용하고 중간에 예외가 발생한다면 `ROLLBACK`으로 모든 작업을 취소합니다.
  
## 주의사항  
트랜잭션 또한 DBMS의 커넥션과 동일하게 꼭 필요한 최소의 코드만 적용하는 것이 좋습니다.  
예를 들어 사용자가 게시판에 게시물을 작성하고 저장 버튼을 클릭할 경우에 아래와 같은 의사 코드로 작성할 수 잇습니다.

1. 처리 시작
   + 데이터베이스 커넥션 생성
   + 트랜잭션 시작
2. 사용자 정보를 데이터베이스에서 조회
3. 사용자의 글쓰기 내용 검증
4. 첨부로 업로드된 파일 검증 및 파일 서버에 저장
5. 사용자의 입력 내용을 DBMS에 저장
6. 첨부 파일 URL을 DBMS에 저장
7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
8. 게시물 등록에 대한 알람 메일 전송
9. 알람 메일 발송 이력을 DBMS에 저장
   + 트랜잭션 종료(`COMMIT`)
   + 데이터베이스 커넥션 반납
+ 처리 완료  
     
트랜잭션은 데이터베이스에서 데이터 정합성과 논리적인 작업 단위를 원자성으로 관리해주는 기능입니다. 
프로그래밍 코드나 외부 네트워크 통신까지 묶어서 사용할 경우 커넥션 소유 시간이 길어지게 됩니다.  
  
커넥션은 제한된 자원이기 때문에 온라인 환경에서 커넥션 소유 시간이 길어지면 사용 가능한 여유 커넥션의 개수도 감소하게 되고 
커넥션이 부족하게 되면 웹 요청을 대기하는 상황이 발생합니다.  
  
더 위험한건 `8번`으로 외부 네트워크를 통한 작업이 트랜잭션에 포함되는 경우입니다. 
`FTP`나`SMTP`와 같이 원격서버에 통신할 때 외부 네트워크가 응답이 없거나 예외가 발생한 경우 DBMS에 까지 영향을 끼칩니다.  
  
`7번`과 같은 단순한 조회의 경우에는 트랜잭션으로 묶을 필요가 없고 `9번`은 `5번,6번`과 다른 별도의 작업이기 때문에 
다른 트랜잭션으로 분리하는 것이 커낵션 관리에 좋습니다.  
  
### 트랜잭션 보완
1. 처리 시작
2. 사용자 정보를 데이터베이스에서 조회
3. 사용자의 글쓰기 내용 검증
4. 첨부로 업로드된 파일 검증 및 파일 서버에 저장
   + 트랜잭션 시작
5. 사용자의 입력 내용을 DBMS에 저장
6. 첨부 파일 URL을 DBMS에 저장
   + 트랜잭션 종료
7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
8. 게시물 등록에 대한 알람 메일 전송
   + 트랜잭션 시작
9. 알람 메일 발송 이력을 DBMS에 저장
   + 데이터베이스 커넥션 반납(`COMMIT`)
+ 처리 완료  
  
정리하면, 프로그래밍 코드이 트랜잭션 내에 포함되어 불필요한 커넥션 유지를 없애고, 외부 네트워크와 같이 
제어할 수 없는 경우가 포함되지 않도록 해야합니다.  
  
> 특히 외부 네트워크 통신이 트랜잭션에 포함된다면 DBMS가 높은 부하 상태로 빠지거나 위험한 상태로 갈 수 있습니다.