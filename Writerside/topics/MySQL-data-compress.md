# HDD는 저렴한데 데이터 압축이 왜 필요할까?

## 메모리는 비싸다.
**버퍼풀과 더티 페이지 최적화** 
: InnoDB 스토리지 엔진은 디스크에 저장된 데이터를 메모리에 캐싱하여 Disk I/O를 줄이기 위해 버퍼풀을 사용합니다. 또한, 더티 페이지를 통해 랜덤 I/O를 최소화합니다. 데이터 압축은 이러한 최적화 작업을 향상시키는 데 도움을 줄 수 있습니다.

**제한된 메모리 공간 관리**
: 데이터 크기가 클수록 버퍼풀에 저장되는 데이터 양이 많아지므로, 메모리 공간을 효율적으로 관리하는 것이 중요합니다. 데이터 압축을 통해 메모리 공간을 절약할 수 있으며, 제한된 메모리 공간을 효과적으로 활용하여 데이터 액세스 성능을 유지할 수 있습니다.

**백업 및 복구**
: 압축된 데이터는 백업 및 복구 작업 시에도 이점을 제공합니다. 더 작은 백업 파일 크기는 저장 공간을 절약하고, 데이터 전송 속도를 향상시킵니다. 또한, 압축된 데이터를 복구하는 데 필요한 시간 및 리소스도 줄일 수 있습니다.  
  
  
MySQL에서는 두가지 방식으로 데이터 압축을 구분합니다.
+ 테이블 압축
+ 페이지 압축
  
## 페이지 압축
> MySQL 서버가 디스크에 저장하는 시점에 데이터 페이지가 압축되는 방식  
> `Transparent Page Compression`
  
반대로 데이터를 읽을때 압출 해제후 버퍼풀에 적재하는 방식으로 버퍼풀에 적재할 때는 압축이 해제된 상태로 
데이터 페이지를 관리합니다.  

페이지 압축은 MySQL 서버가 디스크에 데이터를 저장할 때 페이지 단위로 나누어 압축하는 방식입니다. 이 방식은 테이블의 크기가 클 경우 동일한 페이지 크기로 나누어 저장하여 압축된 용량을 디스크에 효율적으로 저장할 수 있습니다.
  
### 문제점  
운영체제가 펀치홀을 지원해야 합니다. 펀치홀은 데이터를 압축한 후에도 해당 영역을 운영체제뿐만 아니라 하드웨어에서도 지원해야합니다. 펀치홀 기능이 지원되지 않으면 압축된 데이터를 복사하는 과정에서 펀치홀이 사라져서 원본 크기로 복사될 수 있습니다.

## 테이블 압축
**장점**
1. 펀치홀 기능에 의존하지 않음: 펀치홀 기능을 지원하지 않는 환경에서도 사용할 수 있습니다. 따라서 운영체제나 하드디스크의 제약을 받지 않고 데이터를 압축할 수 있습니다.
2. 압축률 제어: KEY_BLOCK_SIZE 값을 조정하여 압축된 데이터의 크기를 제어할 수 있습니다. 이를 통해 디스크 공간을 효율적으로 활용할 수 있습니다.
3. 데이터 압축 및 해제 속도: 압축된 데이터의 크기를 제어할 수 있어 데이터의 압축 및 해제 속도를 조절할 수 있습니다.

### 전제조건
```SQL
# 별도의 테이블 스페이스 활성화
SET GLOBAL innodb_file_per_table = ON;

# 압축 테이블 생성시 명시
CREATE TABLE compressed_table ( 
    //... 
) ROW_FORMAT = COMPRESSED
  KEY_BLOCK_SIZE = 8; 
```  
+ InnoDB 페이지 크기는 `32,64KB`인 경우 압축 적용 불가
+ InnoDB 페이지 크기보다 작아야함 ( 16KB => 4,8)
  
### 동작방식
1. 16KB 데이터 페이지 압축
   1. 압축된 결과가 8KB 이하라면 디스크 저장( 압축 완료 )
   2. 압축된 결과가 8KB 이상이라면 원본 페이지를 스플릿(split)해서 2개의 페이지에 8KB씩 저장
2. 나뉜 페이지에 각각 1번의 페이지 압축을 반복실행  
  
### 한계  
1. 데이터 처리 성능: 압축된 데이터는 버퍼풀에서 압축된 상태와 압축 해제된 상태를 모두 유지해야 하므로 공간 활용이 떨어질 수 있습니다. 또한, 데이터를 조회하거나 커맨드 쿼리를 작성할 때 압축된 데이터의 크기에 따라 처리 성능이 저하될 수 있습니다.
   + 압축된 테이블과 압축 해제된 테이블 2가지를 버퍼풀에서 관리하므로 메모리 낭비가 발생
2. 압축률 변동성: 빈번한 데이터 변경이 발생할 경우 데이터 크기에 따라 압축 해제 후 재압축을 수행해야 하므로 압축률이 변동할 수 있습니다. **_압축 알고리즘은 많은 CPU 자원을 소모합니다._**  
     
## 압축 테이블 선정방법  
### 압축 용량 선택
```SQL
# 압축 상황 조회 SQL
mysql> SELECT
    -> table_name, index_name, compress_ops, compress_ops_ok,
    -> (compress_ops-compress_ops_ok)/compress_ops*100 as compress_failure_pct
    -> from information_schema.INNODB_CMP_PER_INDEX;
    
# 4KB
+------------------+--------------+--------------+-----------------+----------------------+
| table_name       | index_name   | compress_ops | compress_ops_ok | compress_failure_pct |
+------------------+--------------+--------------+-----------------+----------------------+
| employees_comp4k | PRIMARY      |        18635 |           13478 |              27.6737 |
| employees_comp4k | ix_firstname |         8320 |            7653 |               8.0168 |
| employees_comp4k | ix_hiredate  |         7766 |            6721 |              13.4561 |
+------------------+--------------+--------------+-----------------+----------------------+

# 8KB
+------------------+--------------+--------------+-----------------+----------------------+
| table_name       | index_name   | compress_ops | compress_ops_ok | compress_failure_pct |
+------------------+--------------+--------------+-----------------+----------------------+
| employees_comp4k | PRIMARY      |         8092 |            6593 |              18.5245 |
| employees_comp4k | ix_firstname |         1996 |            1996 |               0.0000 |
| employees_comp4k | ix_hiredate  |         1391 |            1381 |               0.7189 |
+------------------+--------------+--------------+-----------------+----------------------+
```  
`compress_failure_pct`은 전체 압축 시도 횟수 - 압축 성공 횟수로 다시 압축을 재시도 했다는 의미입니다.  
  
4KB에서 8KB로 변경하여도 압축 실패육( 압축 재시도 확률 )이 높아서 해당 테이블에는 압축을 설정할때 고려할게 있습니다.  
4KB나 8KB나 압축결과는 큰 차이가 발생하지 않기에 압축 실패율이 낮은 **8KB** 로 설정하는게 낫습니다.  
  
### 압축실패율
압축 실패율이 높다고 압축하지 말아야한다는 의미가 아닙니다.  
단순히 INSERT 용도로 사용되는 로그 테이블인 경우에는 한번 INSERT 된 후에 변경도 없고 조회도 많지 않습니다.   
한 번정도 압축 시도가 증가한다고 해도 전체적인 데이터 파일의 크기를 줄일 수 있다면 압축을 하는것이 좋습니다.  

**압축 실패가 낮다** 압축실패율이 낮아도 자주 I/O가 발생하는 테이블이라면 압축은 고려하지 않는것이 좋습니다.  
  
#### 테이블 압축 튜닝 설정도 제공됩니다.
+ innodb_cmp_per_index_enabled = on : 테이블 압축이 사용된 테이블의 모든 인덱스별로 압축 성공 및 압축 실행 횟수를 수집하도록 설정하며 information_schema.INNODB_CMP_PER_INDEX 테이블에 기록된다.
+ innodb_cmp_per_index_enabled = off : 비활성화 할경우 테이블 단위의 압축 성공 및 압축 실행 횟수만 수입되고 테이블 단위로 수집된 정보는 information_schema.INNODB_CMP 테이블에 기록된다.
+ innodb_compression_level : InnoDB의 테이블 압축은 zlib 압축 알고리즘만 지원하는데 이때 innodb_compression_level 시스템 변수를 이용해 0~9 까지 압축률을 선택할수 있는데 압축률이 작을수록 속도는 빨라지지만 저장공간은 커지며 압축률 속도는 CPU 자원 소모량과 동일한 의미이다.
+ innodb_compression_failure_threshold_pct , innodb_compression_pad_pct_max : 테이블 단위로 압축 실패율이 innodb_compression_failure_threshold_pct 값보다 커지면 압축을 실행하기 전 원본 데이터 페이지의 끝에 의도적으로 일정 크기의 빈 공간을 추가하여 그 공간의 압축률을 높여 압축 결과가 key_block_size 보다 작아지게 만드는 효과를 내며 이때 추가하는 빈 공간을 패딩 이라고 하고 추가할수 있는 패딩의 최대 크기가 innodb_compression_pad_pct_max 이다.
+ innodb_log_compressed_pages : MySQL 서버가 비정상 종료 되었다가 다시 시작될 경우 압축 알고리즘의 버전 차이가 있더라도 복구 과정이 실패하지 않도록 InnoDB스토리지 엔진은 압축된 데이터 페이지를 그대로 리두 로그에 기록하는데 압축 알고리즘을 업데이트 할때 도움이 되지만 데이터 페이지를 통째로 리두 로그에 저장하는 것은 리두 로그의 증가량에 상당한 영향을 줄수도 있다.
압축을 적용한 리두 로그 용량이 매우 빠르게 증가한다거나 버퍼 풀로부터 더티 페이지가 한번에 많이 기록되는 팬턴으로 바뀌었다면 innodb_log_compression_pages 시스템 변수를 비활성화 하고 모니터링 해보는것이 좋다.