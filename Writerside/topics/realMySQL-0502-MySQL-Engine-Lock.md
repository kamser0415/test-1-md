# MySQL 엔진의 잠금  
    
MySQL 엔진의 잠금은 모든 스토리지 엔진에 영향을 끼칩니다.  
  
## 글로벌락
+ 동작 방식 : 수동
+ 범위 : MySQL 서버 전체
+ 사용목적 : `mysqldump`로 일관된 백업이 필요할때
+ 한계 : 글로벌락 동안 `DML`사용을 할 수 없어 모든 DML이 대기상태가 되어 서버 지연발생
    
**SQL**
```SQL
FLUSH TABLES WITH READ LOCK;
-- Perform backup operations here
UNLOCK TABLES;
```
  
### 주의사항
MySQL 서버 내에 모든 테이블에 읽기 잠금을 걸기 전에 먼저 테이블에 플러시를 해야하기 때문에 
테이블에 실행중인 모든 종류의 쿼리가 완료되어야합니다.  
  
만약 먼저 실행중인 쿼리가 오래 지연되고 글로벌락을 실행한다면 
이후에 모든 변경 쿼리가 적용되기까지 `INSERT,UPDATE,DELETE` 쿼리등이 아주 오랜시간 대기하게 됩니다.  
  
### 한계  
MySQL이 InnoDB 스토리지 엔진을 사용하면서 트랜잭션을 지원하게 되었고, 데이터 일관성을 위해 모든 데이터를 변경하는 작업을 멈출 필요가 없어졌습니다.  
  
> 트랜잭션이 없다면 중간에 데이터가 변경되다가 오류가 발생할 경우 데이터 일관성이 깨지지만,
> 트랜잭션을 지원하여 데이터 일관성이 보장되기 때문입니다.  
>   
  
따라서 글로벌 락보다 가벼운 락이 필요합니다.  

또한 글로벌 락을 사용하여 백업할 경우 시간이 지연될 수 록 레플리카 서버에 적용해야하는 동기화가 많아집니다.
소스서버는 최신데이터이지만, 백업 데이터를 레플리카 서버에 동기화되기까지 서비스가 중지될 수 있습니다.
  
또한 백업 중간에 실수로 DDL을 사용하여 복제가 실패하게 되면 처음부터 다시 백업을 해야하며 서버 지연이 더 길어지게 됩니다.
## 백업 락
+ 동작 방식 : 수동
+ 범위 : MySQL 서버 전체
+ 사용목적 : 
  + 레플리카 서버에서 서버 지연없이 백업을 해야하는 경우
  + 실수로 DDL 실행시 백업이 실패하지 않고 중지한다.
+ 장점 : 일반적인 DML 사용가능하다.
```SQL
LOCK INSTANCE FOR BACKUP;

-// 백업실행

UNLOCK INSTANCE;
```  

### 주의사항
+ 데이터베이스 및 테이블의 모든 객체 생성 및 변경, 삭제 변경 불가
+ REPAIR TABLE과 OPIMIZE TABLE 명령 불가
+ 사용자 관리 및 비밀번호 변경 불가  

### 사용목적
백업 중간에 실수로 DDL을 사용하여 복제가 실패하는게 아니라 중지하여 서버 지연을 줄일 수 있습니다.  
  
## 테이블락  
+ 동작 방식 : 수동, 자동
+ 범위 : 개별 테이블 단위
+ 한계 : 온라인 트랜잭션 환경에서는 사용하기 어렵다.

```SQL
LOCK TABLES table_name [ READ | WRITE ]
//...
UNLOCK TABLES 
```  
  
### 자동 동작(묵시적)  
MyISAM 및 MEMORY:
+ DDL 명령어: 테이블 락 발생
+ DML 명령어: 테이블 락 발생 (읽기 또는 쓰기 락)  
  
InnoDB:
+ DDL 명령어: 테이블 락 발생
+ DML 명령어: 주로 행 수준 락 발생, 테이블 락은 드뭄  
  
묵시적 락은 실행되어 명령어가 종료되면 자동으로 해제됩니다.  
  
## 네임드락
+ 동작 방식 : 수동
+ 범위 : 임의의 사용자 지정 문자열
+ 사용목적 :
    + 여러 클라이언트, 단일 DB 서버에서 상호 동기화를 할 때
    + 배치 프로그램으로 많은 레코드가 변경되어 데드락이 발생하여 최소화할 때
+ 장점 : 간단한 구현 가능
+ 단점 : 별도의 커넥션을 사용해야함  
    
**GET_LOCK(str,wait_sec)**
```SQL
-- // my_lock 이라는 문자열에 잠금을 획득한다.
-- // 이미 잠금을 사용중이라면 2초동안 기다립니다. ( 2초 이후 자동 잠금 해제됨 )

// session-1
SELECT GET_LOCK('my_lock',2);

// session-2
select get_lock('my_lock',2);
+-----------------------+
| get_lock('my_lock',2) |
+-----------------------+
|                     0 |
+-----------------------+
1 row in set (2.00 sec)
```  
  
**SELECT IS_FREE_LOCK(str)**
```SQL
-- // 해당 str이 잠금인 경우 0, 잠금이 없는 경우 1
select is_free_lock('my_lock');
+-------------------------+
| is_free_lock('my_lock') |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)
```  
  
**SELECT RELEASE_LOCK(str)**  
```SQL
-- // 해당 str의 잠금을 해제한 경우 1, 해제하지 못한 경우 0 이나 null 반환
select release_lock('my_lock');
+-------------------------+
| release_lock('my_lock') |
+-------------------------+
|                       1 |
+-------------------------+
```  
  
해당 문자열에 중첩으로 잠금을 누적할 수 있으며, 한번에 해제도 가능합니다.
```SQL
select get_lock('my_lock',2);
+-----------------------+
| get_lock('my_lock',2) |
+-----------------------+
|                     1 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select get_lock('my_lock',2);
+-----------------------+
| get_lock('my_lock',2) |
+-----------------------+
|                     1 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select get_lock('my_lock',2);
+-----------------------+
| get_lock('my_lock',2) |
+-----------------------+
|                     1 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select release_all_locks();
+---------------------+
| release_all_locks() |
+---------------------+
|                   3 |
+---------------------+
1 row in set (0.00 sec)
```  
  
같은 커넥션에서 동일한 이름의 네임드 락을 3번 누적하고, 한번의 잠금을 해제할 수 있습니다.  
  
## 메타데이터락
+ 동작 방식 : 자동
+ 범위 : 데이터베이스 객체( 테이블이나 뷰 등)
+ 발생원인 :
  + 테이블의 이름이나 구조를 변경하는 경우
  
### 주의사항
+ `Table not found 'rank'`
    ```SQL
    -- // 배치 프로그램으로 별도 임시테이블(rank_new)에 서비스용 랭킹 데이터 생성
    
    -- // 배치가 완료되면 현재 서비스용 랭킹 테이블(rank)을 rank_backup으로 백업하고
    -- // 새로 만들어진 랭킹 테이블(rank_new)를 서비스용으로 대체하는 경우
    
    RENAME TABLE rank TO rank_backup , rank_new TO rank;
    ```  
    `RENAME` 명령문에 두 개의 RENAMME을 한번에 실행하면 실제 애플리케이션에서 `Table not found 'rank'`같은 상황이 발생하지 않지만
    ```SQL
    RENAME TABLE rank TO rank_backup;
    RENAME TABLE rank_new TO rank; 
    ```
    으로 변경하는 경우 명령어 사이에 실행되는 쿼리에 예외가 발생할 수 있습니다.  
### 테이블 구조 변경시  
1. 테이블 구조를 변경할 때 시간이 오래 걸린다면 이후 조회 작업이 발생할 때마다 언두 로그가 발생하게 됩니다.
2. 언두 로그 버퍼 크기도 고민해야하는 문제 발생
3. DDL은 단일 스레드이기 때문에 많은 시간이 필요하게 됩니다.  
  
이럴때 새로운 구조의 테이블을 생성하고 4개의 스레드를 이용하여 신규 테이블로 복사합니다.
```SQL
INSERT INTO access_log_new SELECT * FROM access_log WHERE id >=0 AND id < 10000;
    :
```  
가장 최근 데이터(`1시간 전이나 하루 전`)까지 복사하고 남은 데이터를 복사할 때 테이블 잠금을 사용하기 때문에 
**잠금 시간을 최소화하는게 중요합니다.**  
  
```SQL
SET autocommit = 0; // BEGIN 이나 START TRANSACTION 은 사용하면 안됩니다.

-- // 작업대상 테이블 2개에 테이블 쓰기 락을 적용
LOCK TABLE access_log_new WRITE, access_log WRITE;

-- // 남은 데이터 복사
SELECT MAX(id) as @MAX_ID FROM access_log_new;
INSERT INTO access_log_new SELECT * FROM access_log WHERE pk > @MAX_ID;
COMMIT;

-- // 새로운 데이터가 복사되면 RENAME으로 테이블 변경
RENAME TABLE access_log TO access_log_old, access_log_new TO access_log;
UNLOCK TABLES;

-- // 불필요한 테이블 삭제
DROP TABLE access_log_old;
```
> set autocommit = 0 을 사용하는 이유  
> 명시적 트랜잭션을 사용할 경우 DDL은 바로 서버에 반영이 됩니다.  
> 남은 데이터 복사후 테이블 이름이 변경하고 테이블을 삭제하는 명령어를 사용하면
> 데이터를 복사하기 전에 테이블이 삭제되는 오류를 방지할 수 있습니다.
