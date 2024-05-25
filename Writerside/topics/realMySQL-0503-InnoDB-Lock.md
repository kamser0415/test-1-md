# InnoDB 스토리지 엔진 잠금
 
## 레코드 락
레코드 자체만 잠그는 것을 말한다.  
  
### 특징  
InnoDB는 레코드를 잠근하는게 아니라 인덱스를 사용하여 레코드를 잠금합니다.  
보조 인덱스를 사용하여 잠금을 설정하는 경우 갭 락이나 넥스트 키 락을 사용하지만,
유니크 인덱스나 프라이머리 키를 사용하는 경우 레코드 자체만 잠금을 설정합니다.  
  
## 갭락 
갭 락은 레코드 자체가 아니라 인접한 레코드까지 같이 잠금이 됩니다.  
갭 락은 레코드와 레코드 사이의 가나격에 새로운 레코드가 생성(`INSERT`)되는 것을 제어하여 
데이터 일관성을 유지할 수 있습니다.  
  
## 넥스트 키 락
+ [레플리카 복제 방식](https://dev.mysql.com/doc/refman/8.0/en/binary-log-formats.html)  
+ [next-key-lock 동작](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-next-key-locks)  

+ STATEMENT 포멧 바이너리 로그를 사용하는 MySQL 서버에서는 REPEATABLE READ 격리 수준을 사용해아한다
+ 넥스트 키 락은 트랜잭션 격리수준인 `REPEATABLE READ`에서 동작합니다.  
+ `innod_locks_unsafe_for_binlog`은 8.0.0 에서 사라졌습니다.  
  
레플리카 서버와 소스 서버의 동일한 쿼리 결과를 보장하기 위해 동작하기 때문에 
데드락이 발생되거나 다른 트랜잭션이 기다리게 하는 일이 자주 발생하므로 줄이는 것이 좋습니다.  
  
## 자동 증가 락
자동 증가하는 숫자를 추출하기 위해 `AUTO_INCREMENT`라는 칼럼 속성을 제공합니다.   
내부적으로 테이블 수준의 잠금을 사용하여 새로운 레코드를 저장할 때만 발생합니다.  

`AUTO_INCREMENT` 속성은 테이블당 하나만 가지고 있으므로 여러개를 INSERT하면 대기하는 시간이 발생합니다.
해당 속성에 명시적으로 값을 넣어도 자동 증가락은 발생합니다.  
  
+ [자동 증가락 설정](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)
+ 글로벌 옵션으로 설정시 주의해야합니다.  
+ 기본은 2로 설정되어있다.
  
`sysvar_innodb_autoinc_lock_mode = 2` 인 경우 
인터리빙(`끼워넣다`) 모드로 경량화된 래치(뮤텍스)를 사용하므로 연속된 자동 증가 값을 보장하지 않지만, 
대용량 SELECT - INSERT를 실행하더라도 다른 커넥션에서 INSERT가 가능한 동시성 처리가 높은 설정입니다.  
  
유니크만 보장하므로 소스 코드와 레플리카 서버가 복제 포멧이 STATEMENT 라면 자동 증가값이 달라질 수 있습니다.  

만약 복제 포멧이 STATEMENT라면 `sysvar_innodb_autoinc_lock_mode = 1`로 변경하여 자동증가 값이 연속적으로 나올 수 있게 변경해야합니다.  
  
## 인덱스와 잠금
InnoDB의 잠금과 인덱스는 중요한 연관 관계가 있습니다.
레코드를 잠금하는게 아니라 변경해야할 레코드를 찾기위해 검색된 인덱스의 레코드를 잠금합니다.  
  
만약 인덱스가 없는 데이터를 잠금한다면 해당 데이터를 조회하는 과정에 사용된 인덱스는 모두 잠금처리됩니다.  
  
## 레코드 수준의 잠금 확인 및 해제
레코드 수준의 잠금은 테이블 잠금과 다르게 사용하지 않는다면 발견하기 어렵습니다.  
게다가 InnoDB 스토리지 엔진 수준에서 잠금이기 때문에 MySQL 서버 엔진에서는 레코드 잠금을 조회하기 어렵기에 
`MySQL 5.1`버전부터 레코드 잠금을 확인할 수 있도록 제공합니다.  
  
```SQL
SHOW PROCESSLIST;
+----+-----------------+---------+-------+------------------------+---------------------------------------------------------------+
| Id | User            | Command | Time  | State                  | Info                                                          |
+----+-----------------+---------+-------+------------------------+---------------------------------------------------------------+
|  5 | event_scheduler | Daemon  | 11263 | Waiting on empty queue | NULL                                                          |
|  8 | root            | Sleep   |    35 |                        | NULL                                                          |
|  9 | root            | Query   |    28 | updating               | UPDATE employees SET birth_date = NOW() WHERE emp_no = 100001 |
| 10 | root            | Query   |    25 | updating               | UPDATE employees SET birth_date = NOW() WHERE emp_no = 100001 |
| 11 | root            | Query   |     0 | init                   | show processlist                                              |
+----+-----------------+---------+-------+------------------------+---------------------------------------------------------------+  
```  
+ [State 상태 공식문서](https://dev.mysql.com/doc/refman/8.0/en/general-thread-states.html)  
  
`State = updating` 스레드에서 업데이트할 행을 검색하고 있으며 이를 업데이트하고 있습니다.  
실행하려는 문장은 `Info` 칼럼에서 확인할 수 있으며 `KILL [iD]`를 통해서 해당 스레드를 제거할 수 있습니다.
