MySQL 서버는 사람의 머리 역할을 담당하는 MySQL 엔진과 손발 역할을 담당하는 스토리지 엔진으로 구분할 수 있다. 그리고 손과 발의 역할을 담당하는 스토리지 엔진은 핸들러 API를 만족하면 누구든지 스토리지 엔진을 구현해서 MySQL 서버에 추가해서 사용할 수 있다.

먼저 MySQL 의 쿼리를 작성하고 튜닝할 때 필요한 기본적인 MySQL 엔진의 구조를 훑어보자.

MySQL 서버는 다른 DBMS에 비해 구조가 상당히 독특하다. 사용자 입장에서 보면 거의 차이가 느껴지지 않지만 이러한 독특한 구조 때문에 다른 DBMS에서는 가질 수 없는 엄청난 혜택을 누릴 수 있으며, 반대로 다른 DBMS에서는 문제되지 않을 것들이 가끔 문제가 되기도 한다.

# MySQL의 전체 구조

![](https://velog.velcdn.com/images/harperkwon/post/4a410dd0-4be0-4f65-8b5a-66ffee73fd6d/image.png)


MySQL은 일반 사용 RDBMS와 같이 대부분의 프로그래밍 언어로부터 접근 방법을 모두 지원한다. MySQL 공의 C API부터 시작해 JDBC나 ODBC, 그리고 .NET의 표준 드라이버를 제공하며, 이러한 드라이버를 이용해 C/C++, PHP, 자바, 펄, 파이썬, 루비나 .NET 및 코볼까지 모든 언어로 MySQL 서버에서 쿼리를 사용할 수 있게 지원한다.

MySQL 서버는 크게  MySQL 엔진과 스토리지엔진으로 구분할 수 있다. 

## MySQL 엔진

MySQL 엔진은 클라이언트로부터의  접속 및 쿼리 요청을 처리하는 커넥션 핸들러와 SQL 파서 및 전처리기, 쿼리의 최적화된 실행을 위한 옵티마이저가 중심을 이룬다. 또한 MySQL은 표준 SQL(ANSI SQL) 문법을 지원하기 때문에 표준 문법에 따라 작성된 쿼리는 타 DBMS와 호환되어 실행될 수 있다.

## 스토리지 엔진

MySQL 엔진은 요청된 SQL 문장을 분석하거나 최적화하는 등 DBMS의 두뇌에 해당하는 처리를 수행하고, 실제 데이터를 디스크 스토리지에 저장하거나 디스크 스토리지로부터 데이터를 읽어오는 부분은 스토리지 엔진이 전담한다. MySQL 서버에서 MySQL 엔진은 하나지만 스토리지 엔진은 여러 개를 동시에 사용할 수 있다. 다음 예제와 같이 테이블이 사용할 스토리지 엔진을 지정하면 이후 해당 테이블의 모든 읽기 작업이나 변경 작업은 정의된 스토리지 엔진이 처리한다.

```mysql
mysql> CREATE TABLE test_table (fd1 INT, fd2 INT) ENGINE=INNODB;
```

위 예제에서 test_table은 InnoDB 스토리지 엔진을 사용하도록 정의했다. 이제 test_table에 대해 INSERT, UPDATE, DELETE, SELECT... 등의 작업이 발생하면 InnoDB 스토리지 엔진이 그러한 처리를 담당한다. 그리고 각 스토리지 엔진은 성능 향상을 위해 키 캐시(MyISAM 스토리지 엔진)나 InnoDB 버퍼 풀(InnoDB 스토리지 엔진)과 같은 기능을 내장하고 있다.

## 핸들러 API

MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야 할 때는 각 스토리지 엔진에 쓰기 또는 읽기를 요청하는데, 이러한 요청을 핸들러(Handler) 요청이라 하고, 여기서 사용되는 API를 핸들러 API 라고 한다. InnoDB 스토리지 엔진 또한 이 핸들러 API를 이용해 MySQL 엔진과 데이터를 주고 받는다. 이 핸들러 API를 통해 얼마나 많은 데이터(레코드) 작업이 있었는지는 `SHOW GLOBAL STATUS LIKE 'Handler%';` 명령으로 확인할 수 있다.

```MYSQL
mysql> SHOW GLOBAL STATUS LIKE 'Handler%';

+----------------------------+-------+

| Variable_name              | Value |

+----------------------------+-------+

| Handler_commit             | 575   |

| Handler_delete             | 8     |

| Handler_discover           | 0     |

| Handler_external_lock      | 6273  |

| Handler_mrr_init           | 0     |

| Handler_prepare            | 0     |

| Handler_read_first         | 40    |

| Handler_read_key           | 1718  |

| Handler_read_last          | 0     |

| Handler_read_next          | 4032  |

| Handler_read_prev          | 0     |

| Handler_read_rnd           | 0     |

| Handler_read_rnd_next      | 158   |

| Handler_rollback           | 0     |

| Handler_savepoint          | 0     |

| Handler_savepoint_rollback | 0     |

| Handler_update             | 329   |

| Handler_write              | 8     |

+----------------------------+-------+

18 rows in set (0.04 sec)
```


## MySQL 스레딩 구조

![](https://velog.velcdn.com/images/harperkwon/post/744faaf3-408e-412c-94bd-4fdc8a6ebc6b/image.png)


MySQL 서버는 프로세스 기반이 아니라 스레드 기반으로 작동하며, 크게 포그라운드(Foreground) 스레드와 백그라운드(Background) 스레드로 구분할 수 있다. MySQL 서버에서 실행 중인 스레드의 목록은 다음과 같이 perfromance_schema 데이터베이스의 threads 테이블을 통해 확인할 수 있다.

```MYSQL
mysql> USE performance_schema;

Reading table information for completion of table and column names

You can turn off this feature to get a quicker startup with -A

  

Database changed

mysql> SELECT thread_id, name, type, processlist_user, processlist_host 

    ->        FROM threads 

    ->        ORDER BY type, thread_id;

+-----------+---------------------------------------------+------------+------------------+------------------+

| thread_id | name                                        | type       | processlist_user | processlist_host |

+-----------+---------------------------------------------+------------+------------------+------------------+

|         1 | thread/sql/main                             | BACKGROUND | NULL             | NULL             |

|         2 | thread/mysys/thread_timer_notifier          | BACKGROUND | NULL             | NULL             |

|         4 | thread/innodb/io_ibuf_thread                | BACKGROUND | NULL             | NULL             |

|         5 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |

|         6 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |

|         7 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |

|         8 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |

|         9 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |

|        10 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |

|        11 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |

|        12 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |

|        13 | thread/innodb/page_flush_coordinator_thread | BACKGROUND | NULL             | NULL             |

|        14 | thread/innodb/log_checkpointer_thread       | BACKGROUND | NULL             | NULL             |

|        15 | thread/innodb/log_flush_notifier_thread     | BACKGROUND | NULL             | NULL             |

|        16 | thread/innodb/log_flusher_thread            | BACKGROUND | NULL             | NULL             |

|        17 | thread/innodb/log_write_notifier_thread     | BACKGROUND | NULL             | NULL             |

|        18 | thread/innodb/log_writer_thread             | BACKGROUND | NULL             | NULL             |

|        19 | thread/innodb/log_files_governor_thread     | BACKGROUND | NULL             | NULL             |

|        24 | thread/innodb/srv_lock_timeout_thread       | BACKGROUND | NULL             | NULL             |

|        25 | thread/innodb/srv_error_monitor_thread      | BACKGROUND | NULL             | NULL             |

|        26 | thread/innodb/srv_monitor_thread            | BACKGROUND | NULL             | NULL             |

|        27 | thread/innodb/buf_resize_thread             | BACKGROUND | NULL             | NULL             |

|        28 | thread/innodb/srv_master_thread             | BACKGROUND | NULL             | NULL             |

|        29 | thread/innodb/dict_stats_thread             | BACKGROUND | NULL             | NULL             |

|        30 | thread/innodb/fts_optimize_thread           | BACKGROUND | NULL             | NULL             |

|        31 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |

|        32 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |

|        33 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |

|        37 | thread/innodb/buf_dump_thread               | BACKGROUND | NULL             | NULL             |

|        38 | thread/innodb/clone_gtid_thread             | BACKGROUND | NULL             | NULL             |

|        39 | thread/innodb/srv_purge_thread              | BACKGROUND | NULL             | NULL             |

|        40 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |

|        41 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |

|        42 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |

|        44 | thread/sql/signal_handler                   | BACKGROUND | NULL             | NULL             |

|        45 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |

|        43 | thread/sql/event_scheduler                  | FOREGROUND | event_scheduler  | localhost        |

|        47 | thread/sql/compress_gtid_table              | FOREGROUND | NULL             | NULL             |

|        48 | thread/sql/one_connection                   | FOREGROUND | root             | localhost        |

+-----------+---------------------------------------------+------------+------------------+------------------+

39 rows in set (0.01 sec)
```

전체 44개의 스레드가 실행중이며, 그중에서 41개의 스레드가 백그라운드 스레드이고 나머지 3개만 포그라운드 스레드로 포시돼 있다. 그런데 이 중에서 마지막  `thread/sql/one_connection` 스레드만 실제 사용자의 요청을 처리하는 포그라운드 스레드다. 백그라운드 스레드의 개수는 MySQL 서버의 설정 내용에 따라 가변적일 수 있다. 동일한 이름의 스레드가 2개 이상씩 보이는 것은 MySQL 서버의 설정 내용에 의해 여러 스레드가 동일 작업을 병렬로 처리하는 경우다.

## 포그라운드 스레드(클라이언트 스레드)

포그라운드 스레드는 최소한 MySQL 서버에 접속된 클라이언트의 수만큼 존재하며, 주로 각 클라이언트 사용자가 요청하는 쿼리 문장을 처리한다. 클라이언트 사용자가 작업을 마치고 커넥션을 종료하면 해당 커넥션을 담당하던 스레드는 다시 스레드 캐시(Thread cache)로 되돌아간다. 이때 이미 스레드 캐시에 일정 개수 이상의 대기 중인 스레드가 있으면 스레드 캐시에 넣지 않고 스레드를 종료시켜 일정 개수의 스레드만 스레드 캐시에 존재하게 한다. 이때 스레드 캐시에 유지할 수 있는 최대 스레드 개수는 `thread_cache_size` 시스템 변수로 설정한다.

포그라운드 스레드는 데이터를 MySQL의 데이터 버퍼나 캐시로부터 가져오며, 버퍼나 캐시에 없는 경우에는 직접 디스크의 데이터나 인덱스 파일로부터 데이터를 읽어와서 작업을 처리한다. MyISAM 테이블은 디스크 쓰기 작업까지 포그라운드 스레드가 처리하지만(MyISAM도 지연된 쓰기가 있지만 일반적인 방식은 아님) InnoDB 테이블은 데이터 버퍼나 캐시까지만 포그라운드 스레드가 처리하고, 나머지 버퍼로부터 디스크까지 기록하는 작업은 백그라운드 스레드가 처리한다.

## 백그라운드 스레드

MyISAM의 경우에는 별로 해당 사항이 없는 부분이지만 InnoDB는 다음과 같은 여러 가지 작업이 백그라운드로 처리된다.

- 인서트 버퍼(Insert Buffer)를 병합하는 스레드
- 로그를 디스크로 기록하는 스레드
- InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드
- 데이터를 버퍼로 읽어 오는 스레드
- 잠금이나 데드락을 모니터링하는 스레드

모두 중요한 역할을 하지만 그중에서도 가장 중요한 것은 로그 스레드(Log thread)와 버퍼의 데이터를 디스크로 내려쓰는 작업을 처리하는 쓰기 스레드(Write thread)일 것이다. MySQL 5.5 버전부터 데이터 쓰기 스레드와 데이터 읽기 스레드의 개수를 2개 이상 지정할 수 있게 됐으며, innodb_write_io_threads와 innodb_read_io_threads 시스템 변수로 스레드의 개수를 설정한다. InnoDB에서도 데이터를 읽는 작업은 주로 클라이언트 스레드에서 처리되기 때문에 읽기 스레드는 많이 설정할 필요가 없지만 쓰기 스레드는 아주 많은 작업을 백그라운드로 처리하기 때문에 일반적인 내장 디스크를 사용할 때는 2~4 정도, DAS나 SAN과 같은 스토리지를 사용할 때는 디스크를 최적으로 사용할 수 있을 만큼 충분히 설정하는 것이 좋다.

사용자의 요청을 처리하는 도중 데이터의 쓰기 작업 지연(버퍼링)되어 처리될 수 있지만 데이터의 읽기 작업은 절대 지연될 수 없다. 그래서 일반적인 사용 DBMS에는 대부분 쓰기 작업을 버퍼링해서 일괄 처리하는 기능이 탑재돼 있으며, InnoDB 또한 이러한 방식으로 처리한다. 하지만 MyISAM은 그렇지 않고 사용자 스레드가 쓰기 작업까지 함께 처리하도록 설계돼 있다. 이러한 이유로 InnoDB에서는 INSERT, UPDATE, DELETE 쿼리로 데이터가 변경되는 경우 데이터가 디스크의 데이터 파일로 완전히 저장될 때까지 기다리지 않아도 된다. 하지만 MyISAM에서 일반적인 쿼리는 쓰기 버퍼링 기능을 사용할 수 없다.

## 메모리 할당 및 사용 구조

![](https://velog.velcdn.com/images/harperkwon/post/f0492e26-12e9-4375-9916-14fcef3470f1/image.png)


MySQL 에서 사용되는 메모리 공간은 크게 글로벌 메모리 영역과 로컬 메모리 영역으로 구분할 수 있다. 글로벌 메모리 영역의 모든 메모리 공간은 MySQL 서버가 시작되면서 운영체제로부터 할당된다. 운영체제의 종류에 따라 다르겠지만 요청된 메모리 공간을 100% 할당해줄 수도 있고, 그 공간만큼 예약해두고 필요할 때 조금씩 할당해주는 경우도 있다. 각 운영체제의 메모리 할당 방식은 상당히 복잡하며, MySQL 서버가 사용하는 정확한 메모리의 양을 측정하는 것 또한 쉽지 않다. 그냥 단순하게 MySQL의 시스템 변수로 설정해 둔 만큼 운영체제로부터 메모리를 할당받는다고 생각해도 된다.

글로벌 메모리 영역과 로컬 메모리 영역은 MySQL 서버 내에 존재하는 많은 스레드가 공유해서 사용하는 공간인지 여부에 따라 구분되며, 각각 다음과 같은 특성이 있다.

## 글로벌 메모리 영역

일반적으로 클라이언트 스레드의 수와 무관하게 하나의 메모리 공간만 할당된다. 단, 필요에 따라 2개 이상의 메모리 공간을 할당받을 수도 있지만 클라이언트의 스레드 수와는 무관하며, 생성된 글로벌 영역이 N개라 하더라도 모든 스레드에 의해 공유된다. 대표적인 글로벌 메모리 영역은 다음과 같다.

- 테이블 캐시
- InnoDB 버퍼 풀
- InnoDB 어댑티브 해시 인덱스
- InnoDB 리두 로그 버퍼

## 로컬 메모리 영역

세션 메모리 영역이라고도 표현하며, MySQL 서버상에 존재하는 클라이언트 스레드가 쿼리를 처리하는 데 사용하는 메모리 영역이다. 대표적으로 위 사진과 같이 커넥션 버퍼와 정렬(소트) 버퍼 등이 있다. 클라이언트가 MySQL 서버에 접속하면 MySQL 서버에서는 클라이언트 커넥션으로부터의 요청을 처리하기 위해 스레드를 하나씩 할당하게 되는데, 클라이언트 스레드가 사용하는 메모리 공간이라고 해서 클라이언트 메모리 영역이라고도 한다. 클라이언트와 MySQL 서버와의 커넥션을 세션이라고 하기 때문에 로컬 메모리 영역을 세션 메모리 영역이라고도 표현한다.

로컬 메모리는 각 클라이언트 스레드별로 독립적으로 할당되며 절대 공유되어 사용되지 않는다는 특징이 있다. 일반적으로 글로벌 메모리 영역의 크기는 주의해서 설정하지만 소트 버퍼와 같은 로컬 메모리 영역은 크게 신경 쓰지 않고 설정하는데, 최악의 경우(가능성은 희박하지만)에는 MySQL 서버가 메모리 부족으로 멈춰 버릴 수도 있으므로 적절한 메모리 공간을 설정하는 것이 중요하다. 로컬 메모리 공간의 또 한 가지 중요한 특징은 각 쿼리의 용도별로 필요할 때만 공간이 할당되고 필요하지 않은 경우에는 MySQL이 메모리 공간을 할당조차도 하지 않을 수도 있다는 점이다. 대표적으로 소트 버퍼나 조인 버퍼와 같은 공간이 그러하다. 그리고 로컬 메모리 공간은 커넥션이 열려 있는 동안 계속 할당된 상태로 남아 있는 공간도 있고(커넥션 버퍼나 결과 버퍼) 그렇지 않고 쿼리를 실행하는 순간에만 할당했다가 다시 해제하는 공간(소트 버퍼나 조인 버퍼)도 있다.

대표적인 로컬 메모리 영역은 다음과 같다.

- 정렬 버퍼(Sort buffer)
- 조인 버퍼
- 바이너리 로그 캐시
- 네트워크 버퍼

## 플로그인 스토리지 엔진 모델

![](https://velog.velcdn.com/images/harperkwon/post/f55148cb-731d-4f4d-b27a-47a09f1ae443/image.png)


MySQL의 독특한 구조 중 대표적인 것이 바로 플러그인 모델이다. 플러그인해서 사용할 수 있는 것이 스토리지 엔진만 있는 것은 아니다. 전문 검색 엔진을 위한 검색어 파서(인덱싱할 키워드를 분리해내는 작업)도 플러그인 형태로 개발해서 사용할 수 있으며, 사용자의 인증을 위한 Native Authentication과 Caching SHA-2 Authentication 등도 모두 플러그인으로 구현되어 제공된다. MySQL은 이미 기본적으로 많은 스토리지 엔진을 가지고 있다. 하지만 이 세상의 수많은 사용자의 요구 조건을 만족시키기 위해 기본적으로 제공되는 스토리지 엔진 이외에 부가적인 기능을 더 제공하는 스토리지 엔진이 필요할 수 있으며, 이러한 요건을 기초로 다른 전문 개발 회사 또는 사용자가 직접 스토리지 엔진을 개발하는 것도 가능하다.

MySQL에서 쿼리가 실핼되는 과정을 크리 다음과 같이 나는 다면 거의 대부분의 작업이 MySQL 엔진에서 처리되고, 마지막 '데이터 읽기/쓰기' 작업만 스토리지 엔진에 의해 처리 된다.

![](https://velog.velcdn.com/images/harperkwon/post/1f1ac9f5-6047-4032-b5a4-c4485d7dbe57/image.png)


그림 4.5의 각 처리 영역에서 '데이터 읽기/쓰기' 작업은 대부분 1건의 레코드 단위(예를 들어, 특정 인덱스의 레코드 1건 읽기 또는 마지막으로 읽은 레코드의 다음 또는 이전 레코드 읽기와 같이)로 처리된다. 그리고 MySQL을 사용하다 보면 '핸들러(Handler)'라는 단어를 자주 접하게 될 것이다. 핸들러라는 단어는 MySQL 서버의 소스코드로부터 넘어온 표현인데, 이는 우리가 매일 타고 다니는 자동차로 비유해 보면 쉽게 이해할 수 있다. 사람이 핸들(운전대)을 이용해 자동차를 운전하듯이, 프로그래밍 언어에서는 어떤 기능을 호출하기 위해 사용하는 운전대와 같은 역할을 하는 객체를 핸들러(또는 핸들러 객체)라고 표현한다. MySQL 서버에서 MySQL 엔진은 사람 역할을 하고 각 스토리지 엔진은 자동차 역할을 하는데, MySQL 엔진이 스토리지 엔진을 조정하기 위해 핸들러라는 것을 사용하게 된다.

MySQL에서 핸들러라는 것은 개념적인 내용이라서 완전히 이해하지 못하더라도 크게 문제되지는 않지만 최소한 MySQL 엔진이 각 스토리지 엔진에게 데이터를 읽어오거나 저장하도록 명령하려면 반드시 핸들러를 통해야 한다는 점만 기억하자. 나중에 MySQL 서버의 상태 변수라는 것을 배울 텐데, 이러한 상태 변수 가운데 'Handler_'로 시작하는 것이 많다는 사실을 알게 될 것이다. 'Handler_'로 시작하는 상태 변수는 'MySQL 엔진이 각 스토리지 엔진에게 보낸 명령의 횟수를 의미하는 변수'라고 이해하면 된다. MySQL에서 MyISAM이나 InnoDB와 같은 다른 스토리지 엔진을 사용하는 테이블에 대해 쿼리를 실행하더라도 MySQL의 처리 내용은 대부분 동일하며, 단순히 '데이터 읽기/쓰기' 영역의 처리만 차이가 있을 뿐이다. 실질적인 GROUP BY나 ORDER BY 등 복잡한 처리는 스토리지 엔진 영역이 아니라 MySQL 엔진의 처리 영역인 '쿼리 실행기'에서 처리된다.

그렇다면 MyISAM이나 InnoDB 스토리지 엔진 가운데 뭘 사용하든 별 차이가 없는 것 아닌가, 라고 생각할 수 있지만 그렇진 않다. 여기서 설명한 내용은 아주 간략하게 언급한 것일 뿐이고, 단순히 보이는 '데이터 읽기/쓰기' 작업 처리 방식이 얼마나 달라질 수 있는가는 차차 이해할 수 있을 것이다. 여기서 중요한 내용은 '하나의 쿼리 작업은 여러 하위 작업으로 나뉘는데, 각 하위 작업이 MySQL 엔진 영역에서 처리되는지 아니면 스토리지 엔진 영역에서 처리되는지 구분할 줄 알아야 한다'는 점이다. 사실 여기서는 스토리지 엔진의 개념을 설명하기 위한 것도 있지만 각 단위 작업을 누가 처리하고 'MySQL 엔진 영역'과 '스토리지 엔진 영역'의 차이를 설명하는 데 목적이 있다. 이제 설치된 MySQL 서버(mysqld)에서 지원되는 스토리지 엔진이 어떤 것이 있는지 확인해보자.

```MYSQL
mysql> SHOW ENGINES;

+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+

| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |

+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+

| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |

| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |

| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |

| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |

| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |

| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |

| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |

| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |

| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |

+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+

9 rows in set (0.01 sec)
```

Support 칼럼에 표시될 수 있는 값은 다음 4가지다.

- YES: MySQL 서버(mysqld)에 해당 스토리지 엔진이 포함돼 있고, 사용 가능으로 활성화된 상태임
- DEFAULT: 'YES'와 동일한 상태이지만 필수 스토리지 엔진임을 의미함(즉, 이 스토리지 엔진이 없으면 MySQL이 시작되지 않을 수도 있음을 의미한다)
- NO: 현재 MySQL 서버(mysqld)에 포함되지 않았음을 의미함
- DISABLED: 현재 MySQL 서버(mysqld)에는 포함됐지만 파라미터에 의해 비활성화된 상태임

MySQL 서버에 포함되지 않은 스토리지 엔진을 사용하려면 MySQL 서버를 다시 빌드(컴파일)해야 한다. 하지만 MySQL 서버가 적절히 준비만 돼 있다면 플러그인 형태로 빌드된 스토리지 엔진 라이브러리를 다운로드해서 끼워 넣기만 하면 사용할 수 있다. 또한 플러그인 형태의 스토리지 엔진은 손쉽게 업그레이드할 수 있다. 스토리지 엔진뿐만 아니라 모든 플러그인의 내용은 다음과 같이 확인할 수 있다. `SHOW PLUGINS` 명령으로 스토리지 엔진뿐 아니라 인증 및 전문 검색용 파서와 같은 플러그인도 (설치돼 있다면) 확인할 수 있다.

```MYSQL
mysql> SHOW PLUGINS;

+---------------------------------+----------+--------------------+---------+---------+

| Name                            | Status   | Type               | Library | License |

+---------------------------------+----------+--------------------+---------+---------+

| binlog                          | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| sha256_password                 | ACTIVE   | AUTHENTICATION     | NULL    | GPL     |

| caching_sha2_password           | ACTIVE   | AUTHENTICATION     | NULL    | GPL     |

| sha2_cache_cleaner              | ACTIVE   | AUDIT              | NULL    | GPL     |

| daemon_keyring_proxy_plugin     | ACTIVE   | DAEMON             | NULL    | GPL     |

| CSV                             | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| MEMORY                          | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| InnoDB                          | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| INNODB_TRX                      | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMP                      | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMP_RESET                | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMPMEM                   | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMPMEM_RESET             | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMP_PER_INDEX            | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CMP_PER_INDEX_RESET      | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_BUFFER_PAGE              | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_BUFFER_PAGE_LRU          | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_BUFFER_POOL_STATS        | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_TEMP_TABLE_INFO          | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_METRICS                  | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_DEFAULT_STOPWORD      | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_DELETED               | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_BEING_DELETED         | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_CONFIG                | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_INDEX_CACHE           | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_FT_INDEX_TABLE           | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_TABLES                   | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_TABLESTATS               | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_INDEXES                  | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_TABLESPACES              | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_COLUMNS                  | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_VIRTUAL                  | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_CACHED_INDEXES           | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| INNODB_SESSION_TEMP_TABLESPACES | ACTIVE   | INFORMATION SCHEMA | NULL    | GPL     |

| MyISAM                          | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| MRG_MYISAM                      | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| PERFORMANCE_SCHEMA              | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| TempTable                       | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| ARCHIVE                         | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| BLACKHOLE                       | ACTIVE   | STORAGE ENGINE     | NULL    | GPL     |

| FEDERATED                       | DISABLED | STORAGE ENGINE     | NULL    | GPL     |

| ngram                           | ACTIVE   | FTPARSER           | NULL    | GPL     |

| mysqlx_cache_cleaner            | ACTIVE   | AUDIT              | NULL    | GPL     |

| mysqlx                          | ACTIVE   | DAEMON             | NULL    | GPL     |

| mysql_native_password           | ACTIVE   | AUTHENTICATION     | NULL    | GPL     |

+---------------------------------+----------+--------------------+---------+---------+

45 rows in set (0.01 sec)
```

MySQL 서버에서는 스토리지 엔진뿐만 아니라 다양한 기능을 플러그인 형태로 지원한다. 인증이나 전문 검색 파서 또는 쿼리 재작성과 같은 플러그인이 있으며, 비밀번호 검증과 커넥션 제어 등에 관련된 다양한 플러그인이 제공된다. 그뿐만 아니라 MySQL 서버의 기능을 커스텀하게 확장할 수 있게 플러그인 API가 매뉴얼에 공개돼 있으므로 기존 MySQL 서버에서 제공하던 기능들을 확장하거나 완전히 새로운 기능들을 플러그인을 이용해 구현할 수도 있다. 플러그인에 대한 자세한 정보는 MySQL 매뉴얼을 참고하자.

## 컴포넌트

MySQL 8.0부터는 기존의 플러그인 아키텍처를 대체하기 위해 컴포넌트 아키텍처가 지원된다. MySQL 서버의 플러그인은 다음과 같은 몇 가지 단점이 있는데, 컴포넌트는 이러한 단점들을 보완해서 구현됐다.

- 플러그인은 오직 MySQL 서버와 인터페이스할 수 있고, 플러그인끼리는 통신할 수 없음
- 플러그인은 MySQL 서버의 변수나 함수를 직접 호출하기 때문에 안전하지 않음(캡슐화 안 됨)
- 플러그인은 상호 의존 관계를 설정할 수 없어서 초기화가 어려움

MySQL 5.7 버전까지는 비밀번호 검증 기능이 플러그인 형태로 제공됐지만 MySQL 8.0의 비밀번호 검증 기능은 컴포넌트로 개선됐다. 컴포넌트의 간단한 사용법을 비밀번호 검증 기능 컴포넌트를 통해 살펴보자.

```MYSQL
SELECT * FROM mysql.component;

+--------------+--------------------+------------------------------------+

| component_id | component_group_id | component_urn                      |

+--------------+--------------------+------------------------------------+

|            1 |                  1 | file://component_validate_password |

+--------------+--------------------+------------------------------------+

1 row in set (0.00 sec)
```

## 쿼리 실행 구조

![](https://velog.velcdn.com/images/harperkwon/post/5edca752-440b-4595-bb19-bad110703420/image.png)


### 쿼리 파서

쿼리 파서는 사용자 요청으로 들어온 문장을 토큰(MySQL이 인식할 수 있는 최소 단위의 어휘나 기호)으로 분리해 트리 형태의 구조로 만들어 내는 작업을 의미한다. 쿼리 문장의 기본 문법 오류는 이 과정에서 발견되고 사용자에게 오류 메시지를 전달하게 된다.

### 전처리기

파서 과정에서 만들어진 파서 트리를 기반으로 쿼리 문장에 구조적인 문제점이 있는지 확인한다. 각 토큰을 테이블 이름이나 칼럼 이름, 또는 내장 함수와 같은 개체를 매핑해 객체의 존재 여부와 객체의 접근 권한 등을 확인하는 과정을 이 단계에서 수행한다. 실제 존재하지 않거나 권한상 사용할 수 없는 개체의 토큰은 이 단계에서 걸러진다.

## 오티마이저

옵티마이저란 사용자의 요청으로 들어온 쿼리 문장을 저렴한 비용으로 가장 빠르게 처리할지를 결정하는 역할을 담당하며, DBMS의 두뇌에 해당한다고 볼 수 있다. 

### 실행 엔진

옵티마이저가 두뇌라면 실행 엔진과 핸들러는 손과 발에 비유할 수 있다. 실행 엔진이 하는 일을 더 쉽게 이해할 수 있게 간단하게 예를 들어 살펴보자. 옵티마이저가 GROUP BY를 처리하기 위해 임시 테이블을 사용하기로 결정했다고 해보자.

1. 실행 엔진이 핸들러에게 임시 테이블을 만들라고 요청
2. 다시 실행 엔진은 WHERE 절에 일치하는 레코드를 읽어오라고 핸들러에게 요청
3. 읽어온 레코드들을 1번에서 준비한 임시 테이블로 저장하라고 다시 핸들러에게 요청
4. 데이터가 준비된 임시 테이블에서 필요한 방식으로 데이터를 읽어 오라고 핸들러에게 다시 요청
5. 최종적으로 실행 엔진은 결과를 사용자나 다른 모듈로 넘김

즉, 실행 엔진은 만들어진 계획대로 각 핸들러에게 요청해서 받은 결과를 또 다른 핸들러 요청의 입력으로 연결하는 역할을 수행한다.

### 핸들러(스토리지 엔진)

앞에서 잠깐 언급한 것처럼 핸들러는 MySQL 서버의 가장 밑단에서 MySQL 실행 엔진의 요청에 따라 데이터를 디스크로 저장하고 디스크로부터 읽어 오는 역할을 담당한다. 핸들러는 결국 스토리지 엔진을 의미하며, MyISAM 테이블을 조작하는 경우에는 핸들러가 MyISAM 스토리지 엔진이 되고, InnoDB 테이블을 조작하는 경우에는 핸들러가 InnoDB 스토리지 엔진이 된다.

### 복제

MySQL 서버에서 복제(Replication)는 매우 중요한 역할을 담당하며, 지금까지 MySQL 서버에서 복제는 많은 발전을 거듭해왔다. 그래서 MySQL 서버의 복제에 관해서는 별도의 장에서 다르기로 하고, 기본적인 복제의 아키텍처 또한 뒤에서 알아보자.

### 쿼리 캐시

MySQL 서버에서 쿼리 캐시(Query Cache)는 빠른 응답을 필요로 하는 웹 기반의 응용 프로그램에서 매우 중요한 역할을 담당했다. 쿼리 캐시는 SQL의 실행 결과를 메모리에 캐시하고, 동일 SQL 쿼리가 실행되면 테이블을 읽지 않고 즉시 결과를 반환하기 때문에 매우 빠른 성능을 보였다. 하지만 쿼리 캐시는 테이블의 데이터가 변경되면 캐시에 저장된 결과 중에서 변경된 테이블과 관련된 것들은 모두 삭제(Invalidate)해야 했다. 이는 심각한 동시 처리 성능 저하를 유발한다. 또한 MySQL 서버가 발전하면서 성능이 개선되는 과정에서 쿼리 캐시는 계속된 동시 처리 성능 저하와 많은 버그의 원인이 되기도 했다.

결국 MySQL 8.0으로 올라오면서 쿼리 캐시는 MySQL 서버의 기능에서 완전히 제거되고, 관련된 시스템 변수도 모두 제거됐다. MySQL 서버의 쿼리 캐시 기능은 아주 독특한 환경(데이터 변경은 거의 없고 읽기만 하는 서비스)에서는 매우 훌륭한 기능이었지만 이런 요건을 가진 서비스는 흔치 않다. 실제 쿼리 캐시 기능이 큰 도움이 됐던 서비스는 거의 없었다. 이 같은 이유로 MySQL 서버에서 쿼리 캐시를 제거한 것은 좋은 선택이라고 생각한다. 
실제로 큰 도움은 되지 않았지만 수많은 버그의 원인으로 지목되는 경우가 많았기 때문이다.

### 스레드 풀

MySQL 서버 엔터프라이즈 에디션은 스레드 풀(Thread Pool) 기능을 제공하지만 MySQL 커뮤니티 에디션은 스레드 풀 기능을 지원하지 않는다. 여기서는  MySQL 엔터프라이즈 에디션에 포함된 스레드 풀 대신 Percona Server에서 제공하는 스레드 풀 기능을 살펴보고자 한다.

우선 MySQL 엔터프라이즈 스레드 풀 기능은 MySQL 서버 프로그램에 내장돼 있지만 Percona Server의 스레드 풀은 플러그인 형태로 작동하게 구현돼 있다는 차이점이 있다. 만약 MySQL 커뮤니티 에디션에서도 스레드 풀 기능을 사용하고자 한다면 동일 버전의 Percona Server에서 스레드 풀 플러그인 라이브러리를 MySQL 커뮤니티 에디션 서버에 설치해서 사용하면 된다.

스레드 풀은 내부적으로 사용자의 요청을 처리하는 스레드 개수를 줄여서 동시 처리되는 요청이 많다하더라도 MySQL 서버의 CPU가 제한된 개수의 스레드 처리에만 집중할 수 있게 해서 서버의 자원 소모를 줄이는 것이 목적이다. 많은 사람들이 MySQL 서버에서 스레드 풀만 설치하면 성능이 그냥 두 배쯤 올라갈 거라고 기대하는데, 스레드 풀이 실제 서비스에서 눈에 띄는 성능 향상을 보여준 경우는 드물었다. 또한 스레드 풀은 앞에서 소개한 것처럼 동시에 실행 중인 스레드들을 CPU가 최대한 잘 처리해낼 수 있는 수준으로 줄여서 빨리 처리하게 하는 기능이기 때문에 스케줄링 과정에서 CPU 시간을 제대로 확보하지 못하는 경우에는 쿼리 처리가 더 느려지는 사례도 발생할 수 있다는 점에 주의하자. 물론 제한된 수의 스레드만으로 CPU가 처리하도록 적절히 유도한다면 CPU의 프로세서 친화도(Processor affinity)도 높이고 운영체제 입장에서는 불필요한 컨텍스트 스위치(Context switch)를 줄여서 오버헤드를 낮출 수 있다.

스레드 그룹의 모든 스레드가 일을 처리하고 있다면 스레드 풀은 해당 스레드 그룹에 새로운 작업 스레드(Worker thread)를 추가할지, 아니면 기존 작업 스레드가 처리를 완료할 때까지 기다릴지 여부를 판단해야 한다. 스레드 풀의 타이머 스레드는 주기적으로 스레드 그룹의 상태를 체크해서 `thread_pool_stall_limit` 시스템 변수에 정의된 밀리초만큼 작업 스레드가 지금 처리 중인 작업을 끝내지 못하면 새로운 스레드를 생성해서 스레드 그룹에 추가한다. 이때 전체 스레드 풀에 있는 스레드의 개수는 `thread_pool_stall_limit` 시스템 변수에 설정된 개수를 넘어설 수 없다. 즉, 모든 스레드 그룹의 스레드가 각자 작업을 처리하고 있는 상태에서 새로운 쿼리 요청이 들어오더라도 스레드 풀은 `thread_pool_stall_limit` 시간 동안 기다려야만 새로 들어온 요청을 처리할 수 있다는 뜻이다. 따라서 응답 시간에 아주 민감한 서비스라면 `thread_pool_stall_limit` 시스템 변수를 적절히 낮춰서 설정해야 한다. 그렇다고 해서 thread_pool_stall_limit을 0에 가까운 값으로 설정하는 것은 권장하지 않는다. `thread_pool_stall_limit` 을 0에 가까운 값으로 설정해야 한다면 스레드 풀을 사용하지 않는 편이 나을 것이다.

Percona Server의 스레드 풀 플러그인은 선순위 큐와 후순위 큐를 이용해 특정 트랜잭션이나 쿼리를 우선적으로 처리할 수 있는 기능도 제공한다.  이렇게 먼저 시작된 트랜잭션 내에 속한 SQL을 빨리 처리해주면 해당 트랜젝션이 가지고 있던 잠금이 빨리 해제되고 잠금 경합을 낮춰서 전체적인 처리 성능을 향상시킬 수 있다.


![](https://velog.velcdn.com/images/harperkwon/post/2a187eab-ae78-47fe-b914-8e8ef17bc544/image.png)


### 트랜잭션 지원 메타데이터

데이터베이스 서버에서 테이블의 구조와 스토어드 프로그램 등의 정보를 데이터 딕셔너리 또는 메타데이터라고 하는데, MySQL 서버는 5.7 버전까지 테이블의 구조를 FRM 파일에 저장하고 일부 스토어드 프로그램 또한 파
( * TRN, * TRG, * PAR,..) 기반으로 관리했다.  하지만 이러한 파일 기반의 메타데이터는 생성 및 변경 작업이 트랜잭션을 지원하지 않기 때문에 테이블의 생성 또는 변경 도중에 MySQL 서버가 비정상적으로 종료되면 일관되지 않은 상태로 남는 문제가 있었다. 많은 사용자들이 이 같은 현상을 가리켜 '데이터베이스나 테이블이 깨졌다'라고 표현한다.

MySQL 8.0 버전부터는 이러한 문제점을 해결하기 위해 테이블의 구조 정보나 스토어드 프로그램의 코드 관련 정보를 모두 InnoDB 의 테이블에 저장하도록 개선됐다. MySQL 서버가 작동하는 데 기본적으로 필요한 테이블들을 묶어서 시스템 테이블이라고 하는데, 대표적으로 사용자의 인증과 권한에 관련된 테이블들이 있다. MySQL 서버 8.0 버전부터는 이런 시스템 테이블을 모두 InnoDB 스토리지 엔진을 사용하도록 개선했으며, 시스템 테이블과 데이터 딕셔너리 정보를 모두 모아서 mysql DB에 저장하고 있다. mysql DB는 통째로 mysql.ibd라는 이름의 테이블스페이스에 저장된다. 그래서 MySQL 서버의 데이터 디렉터리에 존재하는 mysqlibd라는 파일은 다른 * .ibd 파일과 함께 특별히 주의해야 한다.

MySQL 8.0 버전부터 데이터 딕셔너리와 시스템 테이블이 모두 트랜잭션 기반의 InnoDB 스토리지 엔진에 저장되도록 개선되면서 이제 스키마 변경 작업 중간에 MySQL 서버가 비정상적으로 종료된다고 하더라도 스키마 변경이 완전한 성공 또는 완전한 실패로 정리된다. 기존의 파일 기반 메타데이터를 사용할 때와 같이 작업 진행 중이 상태로 남으면서 문제를 유발하지 않게 개선된 것이다.

MySQL 서버에서 InnoDB 스토리지 엔진을 사용하는 테이블은 메타 정보가 InnoDB 테이블 기반의 딕셔너리에 저장되지만 MyISAM 이나 CSV 등과 같은 스토리지 엔진의 메타 정보는 여전히 저장할 공간이 필요하다. MySQL 서버는 InnoDB 스토리지 엔진 이외의 스토리지 엔진을 사용하는 테이블들을 위해 SDI(Serialized Dictionary Information) 파일을 사용한다. InnoDB 이외의 테이블들에 대해서는 SDI 포맷의 * .sid 파일이 존재하며, 이 파일은 기존의 * FRM 파일과 동일한 역할을 한다. 그리고 SDI는 이름 그대로 직렬화(Serialized)를 위한 포맷이므로 InnoDB 테이블들의 구조도 SDI 파일로 변환할 수 있다. ibd2sdi 유틸리티를 이용하면 InnoDB 테이블스페이스에서 스키마 정보를 추출할 수 있는데, 다음 예제는 mysql DB에 포함된 테이블의 스키마를 JSON 파일로 덤프한 것이다. ibd2sdi 유틸리티로 추출한 테이블의 정보 중에는 MySQL 서버에서 SHOW TABLE 명령으로는 확인할 수 없단 mysql.tables 딕셔너리 데이터를 위한 테이블 구조도 볼 수 있다.

```
cat tables_schema.txt

TABLE_CATALOG: varchar(512) NOT NULL DEFAULT 'def'

TABLE_SCHEMA: varchar(64) NOT NULL

TABLE_NAME: varchar(64) NOT NULL

TABLE_TYPE: varchar(64) NOT NULL

ENGINE: varchar(64) DEFAULT NULL

VERSION: bigint DEFAULT NULL

ROW_FORMAT: varchar(10) DEFAULT NULL

TABLE_ROWS: bigint DEFAULT NULL

AVG_ROW_LENGTH: bigint DEFAULT NULL

DATA_LENGTH: bigint DEFAULT NULL

MAX_DATA_LENGTH: bigint DEFAULT NULL

INDEX_LENGTH: bigint DEFAULT NULL

DATA_FREE: bigint DEFAULT NULL

AUTO_INCREMENT: bigint DEFAULT NULL

CREATE_TIME: datetime DEFAULT NULL

UPDATE_TIME: datetime DEFAULT NULL

CHECK_TIME: datetime DEFAULT NULL

TABLE_COLLATION: varchar(32) DEFAULT NULL

CHECKSUM: bigint DEFAULT NULL

CREATE_OPTIONS: varchar(255) DEFAULT NULL

TABLE_COMMENT: varchar(2048) NOT NULL
```
