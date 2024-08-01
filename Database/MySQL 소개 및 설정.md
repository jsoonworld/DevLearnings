
# MySQL 소개

MySQL은 소스가 공개된 오픈소스 데이터베이스로 알려져 있지만 처음부터 MySQL이 오픈소스였던 것은 아니었다. TcX 사내에서 사용되다가 일반인에게 공개가 되고 썬마이크로시스템, 오라클로 거쳤고 엔터프라이즈 에디션, 커뮤니티 에디션으로 유지되어오고 있다.

# 왜 MySQL 인가?

다른 DBMS와 비교할 때 MySQL의 경쟁력은 무엇이고 왜 MySQL을 사용해야 할까?
MySQL과 오라클을 비교해 본다면 당연히 MySQL의 경쟁력은 가격이나 비용이다. 만약 페이스북이 가진 데이터를 모두 오라클 RDBMS에 저장하면 페이스북은 망할 것이라는 말이 있다. 앞으로 10년은 지금까지 만들어지던 데이터의 몇백 배, 아니 몇천 배로 늘어날 것이다. "어떤 DBMS를 사용해야 할지 모르겠습니다. 어떤 DBMS가 좋은가요?" 다음 질문은 많은 사람들로부터 듣는 질문이라고 한다. 이런 질문들 받을 때마다 저자는 "자기가 가장 잘 활용할 수 있는 DBMS가 가장 좋은 DBMS입니다."라고 답변한다고 한다. 

- 안정성
- 성능과 기능
- 커뮤니티나 인지도

성능이나 기능은 돈이나 노력으로 해결되지만 안정성은 그렇지 않다.
DBMS 선택에서 중요한 요소인 안정성과 성능, 그리고 기능은 DB-Engines의 점수 부여 기준이 아니다. DB -Engines.com에서 점수 산정의 시준은 DBMS별 활용도에 집중하고 있는데, 결국 이런 활용도의 중심에는 DBMS의 안정성과 기능, 그리고 성능이 모두 포함돼 있기 때문일 것이다.

# MySQL 서버 설치

MySQL 서버는 다음과 같이 다양한 형태로 설치할 수 있지만 가능하다면 리눅스의 RPM이나 운영체제별 인스톨러를 이용하기를 권장한다.

- Tar 또는 Zip으로 압축된 버전
- 리눅스 RPM 설치 버전(윈도우 인스톨러 및 macOS 설치 패키지)
- 소스코드 빌드

단순 설치 과정에 대한 설명은 넘어가겠습니다.

# MySQL 서버의 시작과 종료

여기서는 거의 대부분의 서비스 환경에서 사용되는 리눅스 운영체제에서 MySQL 서버의 설정 파일을 비롯해 MySQL 서버를 시작, 종료하는 방법을 살펴보겠다.

# 설정 파일 및 데이터 파일 준비

리눅스 서버에서 Yum 인스톨러나 RPM을 이용해 MySQL 서버를 설치하면 MySQL 서버에 필요한 프로그램과 디렉터리들은 일부 준비되지만 트랜잭션 로그 파일과 시스템 테이블이 준비되지 않았기 때문에 아직 MySQL 서버를 시작할 수 없다. 우선 MySQL 서버가 설치되면 /etc/my.cnf 설정 파일이 준비되는데, 이 설정 파일에는 MySQL 서버를 실행하는 데 꼭 필요한 3~4개의 아주 기본적인 설정만 기록돼있다. 실제 서비스용으로 사용하기에는 많이 부족한 상태지만 간단히 테스트용으로 MySQL 서버를 실행한다면 이 정도로도 충분히 MySQL 서버를 실행할 수는 있다. (자세한 진행과정은 책을 참고하자!)

## What is Yum?

Yum(Yellowdog Updater, Modified)은 RPM 기반의 패키지 관리 시스템을 사용하는 리눅스 배포판에서 사용되는 패키지 관리 도구이다. Yum은 의존성 해결, 패키지 설치, 업그레이드, 제거 등을 간편하게 할 수 있도록 도와준다. 주로 CentOS, RHEL(Red Hat Enterprise Linux) 및 Fedora 같은 배포판에서 사용된다.

## What is RPM?

RPM(레드햇 패키지 매니저, Red Hat Package Manager)은 리눅스 운영 체제에서 소프트웨어 패키지를 관리하는 포맷이자 패키지 관리 시스템이다. RPM 기반 시스템은 RPM 패키지를 사용하여 소프트웨어의 설치, 업그레이드, 제거 등을 처리한다. 주로 Red Hat 계열의 리눅스 배포판(CentOS, Fedora, RHEL 등)에서 사용된다.

# 시작과 종료

유닉스 계열 운영체제에서 RPM 패키지로 MySQL을 설치했다면 자동으로 /usr/lib/systemd/system/mysqld.service 파일이 생성되고, systemctl 유틸리티를 이용해 MySQL을 기동하거나 종료하는 것이 가능하다.

- 시작
```
linux> systemctl start mysqld
```
- 종료
```
linux> systemctl stop mysqld
```
- 원격으로 MySQL 서버 셧다운 (MySQL 서버에 로그인한 상태에서 권한을 가진 상태여아 함.)
```
mysql> SHUTDOWN;
```

MySQL 서버에서는 실제 트랜잭션이 정상적으로 커밋돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일(리두 로그)에만 기록돼 있을 수 있다. 심지어 MySQL 서버가 종료되고 다시 시작된 이후에도 계속 이 상태로 남아있을 수도 있다. 사용량이 많은 MySQL 서버에서는 이런 현상이 더 일반적인데, 이는 결코 비정상적인 상황이 아니다. 하지만 MySQL 서버가 종료될 때 모든 커밋된 내용을 데이터 파일에 기록하고 종료하게 할 수도 있는데, 이 경우에는 다음과 같이 MySQL 서버의 옵션을 변경하고 MySQL 서버를 종료하면 된다.

```
mysql> SET GLOBAL innodb_fast_shutdown=0;
linux> systemctl stop mysqld.service


## 또는 원격으로 MySQL 서버 종료 시
mysql> SET GLOBAL innodb_Fast_shutdwon=0
mysql> SHUTDOWN;
```

이렇게 모든 커밋된 데이터를 데이터 파일에 적용하고 종료하는 것을 클린 셧다운(Clean shutdown)이라고 표현한다. 클린 셧다운으로 종료되면 다시 MySQL 서버가 기동할 때 별도의 트랜잭션 복구 과정을 진행하지 않기 때문에 빠르게 시작할 수 있다.

## 주의

MySQL 서버가 시작되거나 종료될 때는 MySQL 서버(InnoDB 스토리지 엔진)의 버퍼 풀 내용을 백업하고 복구하는 과정이 내부적으로 실행된다. 실제 버퍼 풀의 내용을 백업하는 것이 아니라, 버퍼 풀에 적재돼 있던 데이터 파일의 데이터 페이지에 대한 메타 정보를 백업하기 때문에 용량이 크지 않으며, 백업 자체는 매우 빠르게 완료된다. 하지만 MySQL 서버가 새로 시작될 때는 디스크에서 데이터 파일들을 모두 읽어서 적재해야 하므로 상당한 시간이 걸릴 수도 있다. 혹시 MySQL 서버의 시작 시간이 오래 걸린다면 MySQL 서버가 버퍼 풀의 내용을 복구하고 있는지 확인해보는 것이 좋다. 

# 서버 연결 테스트

MySQL 서버가 시작 됐다면 서버에 직접 접속해보자.
```
linux> mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock
linux> mysql -uroot -p --host=127.0.0.1 -- port=3306
linux> mysql -uroot -p
```

첫 번째 예제는 MySQL 소켓 파일을 이용해 접속하는 예제다. 두 번째 예제는 TCP/IP 를 통해 127.0.0.1(로컬 호스트)로 접속하는 예제인데, 이 경우에는 포트를 명시하는 것이 일반적이다. 로컬 서버에 설치된 MySQL이 아니라 원격 호스트에 있는 MySQL 서버에 접속할 때는 반드시 두 번째 방법을 사용해야 한다. MySQL 서버에 접속할 때는 호스트를 localhost로 명시하는 것과 127.0.0.1로 명시하는 것이 각각 의미가 다르다. --host=localhost 옵션을 사용하면 MySQL 클라이언트 프로그램은 항상 소켓 파일을 통해 MySQL 서버에 접속하게 되는데, 이는 'Unix domain socket'을 이용하는 방식으로 TCP/IP 를 통한 통신이 아니라 유닉스의 프로세스 간 통신 (IPC; Inter Process Commnuication)의 일종이다. 하지만 127.0.0.1을 사용하는 경우에는 자기 서버를 가리키는 루프백(loopback) IP 이기는 하지만 TCP/IP 통신 방식을 사용하는 것이다.

세번 째 방식은 별도로 호스트 주소와 포트를 명시하지 않는다. 이 경우에는 기본값으로 호스트는 localhost가 되며 소켓 파일을 사용하게 되는데, 소켓 파일의 위치는 MySQL 서버의 설정 파일에서 읽어서 사용한다. MySQL 서버가 기동될 때 만들어지는 유닉스 소켓 파일은 MySQL 서버를 재시작하지 않으면 다시 만들어 낼 수 없기 때문에 실수로 삭제하지 않도록 주의한다. 유닉스나 리눅스에서 mysql 클라이언트 프로그램을 실행하는 경우에는 mysql 프로그램의 경로를 PATH 환경변수에 등록해 둔다.

MySQL 서버에 접속했다면 SHOW DATABASES 명령으로 데이터베이스의 목록을 확인할 수 있다. 

# 서버 설정

일반저으로 MySQL 서버는 단 하나의 설정 파일을 사용하는데, 리눅스를 포함한 유닉스 계열에서는 my.cnf라는 이름을 사용하고, 윈도우 계열에서는 my.ini라는 이름을 사용한다. MySQL 서버는 시작될 대만 이 설정 파일을 참조하는데, 이 설정 파일의 경로가 고정돼 있는 것은 아니다. MySQL 서버는 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 my.cnf 파일을 사용하게 된다.

또한 직접 MySQL 을 컴파일해서 설치한 경우에는 이 디렉터리가 다르게 설정될 수도 있다. 만약 설치 된 MySQL 서버가 어느 디렉터리에서 my.cnf 파일을 읽는지 궁금하다면 다음과 같이 mysqld 프로그램을 '--verbose --help' 옵션을 주어 실행해 보면 된다. mysqld 프로그램은 MySQL 서버의 실행 프로그램으로 서비스용으로 사용되는 서버에서 이미 MySQL 서버가 실행 중인데 다시 mysqld 프로그램을 시작한다거나 하지 않도록 주의해야 한다. 가능하다면 두 번째 예제처럼 mysql 클라이언트 프로그램(첫 번째 예제의 mysqld 서버 프로그램이 아닌)으로 확인해 보는 것이 좋다.

```
mysqld --verbose --help

...
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
...

shell> mysql --help
...
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
...

```

두 명령 모두 상당히 많은 내용이 출력된다. 내용 상단에 'Default options are read ...' 라는 부분을 보면 MySQL 서버나 클라이언트 프로그램이 어디에 있는 my.cnf(또는 my.ini) 파일을 참조하는지 확인할 수 있다.
위 예제에서는 다음과 같은 순서로 파일을 찾고 있다.

1. /etc/my.cnf 파일
2. /etc/mysql/my.cnf 파일
3. /usr/etc/my.cnf 파일
4. ~/.my.cnf 파일

실제 MySQL 서버는 단 하나의  설정 파일(my.cnf)만 사용하지만 설정 파일이 위치한 디렉터리는 여러 곳일 수 있다는 것이다. 이러한 특성은 MySQL 사용자를 상당히 혼란스럽게 하는 부분이기도 하다. 만약 실수로 1번과 2번 디렉터리에 각각 my.cnf 파일을 만든 경우, MySQL 서버가 어느 디렉터리의 my.cnf 파일을 참조해서 기동했는지 알아내기가 쉽지 않다. 이러한 경우에는 위 예제의 명령으로 MySQL 서버가 어느 디렉터리의 my.cnf 파일을 먼저 읽는지(우선순위가 높은지)를 확인할 수 있다. 이러한 파일 가운데 1, 2, 4번 파일은 어느 MySQL에서나 동일하게 검색하는 경로이며, 3번 파일은 컴파일 될 때 MySQL 프로그램에 내장된 경로다. 즉, 컴파일할 때 설정한 MySQL 의 홈 디렉터리나 MySQL 홈 디렉터리 밑의 etc 디렉터리에 있는 my.cnf 파일이 표시된다.

MySQL 서버용 설정 파일은 주로 1번이나 2번을 사용하는데, 하나의 장비(서버 머신)에 2개 이상의 MySQL 서버(인스턴스)를 실행하는 경우에는 1번과 2번은 충돌이 발생할 수 있으므로 공유된 디렉터리가 아닌 별도 디렉터리에 설정 파일을 준비하고 MySQL 시작 스크립트의 내용을 변경하는 방법을 사용한다. 그러나 하나의 서버에 2개 이상의 MySQL 서버(인스턴스)를 실행하는 형태로 서비스한 경험은 별로 없으며, 그렇게 사용하는 곳도 흔하지는 않다고 한다.

# 설정 파일의 구성

MySQL 설정 파일은 하나의 my.cnf나 my.ini 파일에 여러 개의 설정 그룹을 담을 수 있으며, 대체로 실행 프로그램 이름을 그룹명으로 사용한다. 예를 들어, mysqldump 프로그램은 [mysqldump] 설정 그룹을, myslqd 프로그램은 설정 그룹의 이름이 [mysqld]인 영역을 참조한다. 그리고 mysqld_safe 프로그램은 [mysqld_safe]와 [mysqld] 섹션을 참조한다.

```
[mysqld_safe]
malloc-lib = /opt/lib/libtcmalloc_minimal.so

[mysqld]
socket = /usr/local/mysql/tmp/mysql.sock
port = 3306

[mysql]
default-character-set = utf8mb4
socket = /usr/local/mysql/tmp/mysql.sock
port = 3304

[mysqldump]
default-character-set = utf8mb4
socket = /usr/local/mysql/tmp/mysql.sock
port = 3305

```

### 섹션별 설명

#### [mysqld_safe]

이 섹션은 `mysqld_safe` 스크립트의 설정을 정의한다. `mysqld_safe`는 MySQL 서버를 안전하게 시작하고 모니터링하는 데 사용되는 스크립트이다.

- **malloc-lib**: 이 옵션은 MySQL 서버가 사용할 메모리 할당 라이브러리를 지정한다. 여기서는 `/opt/lib/libtcmalloc_minimal.so`를 사용하도록 설정되어 있다. 이는 Google의 TCMalloc 라이브러리로, 메모리 할당 및 해제를 최적화하여 성능을 향상시킬 수 있다.

#### [mysqld]

이 섹션은 실제 MySQL 서버 데몬(`mysqld`)의 설정을 정의한다.

- **socket**: MySQL 서버가 사용할 유닉스 소켓 파일의 경로를 지정한다. 여기서는 `/usr/local/mysql/tmp/mysql.sock`로 설정되어 있다. 유닉스 소켓 파일은 로컬 호스트에서 MySQL 서버와 클라이언트가 통신할 때 사용된다.
- **port**: MySQL 서버가 수신 대기할 TCP/IP 포트를 지정한다. 여기서는 기본 포트인 `3306`으로 설정되어 있다.

#### [mysql]

이 섹션은 MySQL 클라이언트 프로그램(`mysql`)의 설정을 정의한다.

- **default-character-set**: MySQL 클라이언트가 사용할 기본 문자 세트를 지정한다. 여기서는 `utf8mb4`로 설정되어 있어, 4바이트 UTF-8 인코딩을 지원한다.
- **socket**: MySQL 클라이언트가 사용할 유닉스 소켓 파일의 경로를 지정한다. 여기서는 `/usr/local/mysql/tmp/mysql.sock`로 설정되어 있다.
- **port**: MySQL 클라이언트가 연결할 때 사용할 포트를 지정한다. 여기서는 `3304`로 설정되어 있다.

#### [mysqldump]

이 섹션은 MySQL 덤프 도구(`mysqldump`)의 설정을 정의한다.

- **default-character-set**: `mysqldump`가 사용할 기본 문자 세트를 지정한다. 여기서는 `utf8mb4`로 설정되어 있다.
- **socket**: `mysqldump`가 사용할 유닉스 소켓 파일의 경로를 지정한다. 여기서는 `/usr/local/mysql/tmp/mysql.sock`로 설정되어 있다.
- **port**: `mysqldump`가 연결할 때 사용할 포트를 지정한다. 여기서는 `3305`로 설정되어 있다.

### 요약

- **[mysqld_safe]**: MySQL 서버를 안전하게 시작하고 모니터링하는 데 사용되는 스크립트의 설정을 정의한다. 여기서는 TCMalloc 라이브러리를 메모리 할당에 사용하도록 설정한다.
- **[mysqld]**: MySQL 서버 데몬의 설정을 정의한다. 소켓 파일 경로와 TCP/IP 포트를 지정한다.
- **[mysql]**: MySQL 클라이언트 프로그램의 설정을 정의한다. 기본 문자 세트, 소켓 파일 경로, 포트를 지정한다.
- **[mysqldump]**: MySQL 덤프 도구의 설정을 정의한다. 기본 문자 세트, 소켓 파일 경로, 포트를 지정한다.

이 예제는 간략한 MySQL 설정 파일의 구성을 보여주는데, 이 설정 파일이 MySQL 서버만을 위한 설정 파일이라면 [mysqld] 그룹만 명시해도 무방하다. 하지만 MySQL 서버뿐 아니라 MySQL 클라이언트나 백업을 위한 mysqldump 프로그램이 실행될 때도 이 설정 파일을 공용으로 사용하고 싶다면 [mysql] 또는 [mysqldump]등의 그룹을 함께 설정해 둘 수 있다. 일반적으로 각 그룹을 사용하는 프로그램은 성격이 다르며, 각 프로그램이 필요로 하는 설정 내용이 상이하므로 이 예제처럼 중복되는 설정이 나열되는 경우는 거의 없지만 socket 이나 port 같은 설정은 모든 프로그램에 공통으로 필요한 설정값이라서 위와 같이 각 설정 그룹에 여러번 설정된 것이다.

이 예제의 설정 파일을 사용하는 MySQL 서버 (mysqld) 프로그램은 3306 포트를 사용한다. 하지만 MySQL 클라이언트(mysql) 프로그램은 3304 포트를 이용해 MySQL 서버에 접속하려고 할 것이다. 즉, 설정 파일의 각 그룹은 같은 파일을 공유하지만 서로 무관하게 적용된다는 의미다.

# MySQL 시스템 변수의 특징

MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위해 이러한 값을 별도로 저장해 둔다.MySQL 서버에서는 이렇게 저장된 값을 시스템 변수(System Variables) 라고 한다. 각 시스템 변수는 다음 예제와 같이 MySQL 서버에 접속해 SHOW VARIABLES 또는 SHOW GLOBAL VARIABLES 라는 명령으로 확인할 수 있다.

```
mysql> SHOW GLOBAL VARIABLES;

mysql> SHOW GLOBAL VARIABLES;

+-----------------------------+---------------------------+
| Variable_name               | Value                     |
+-----------------------------+---------------------------+
| activate_all_roles_on_login | OFF                       |
| admin_address               |                           |
| admin_port                  | 33062                     |
| auto_increment_increment    | 1                         |
| auto_increment_offset       | 1                         |
| autocommit                  | ON                        |
| basedir                     | /usr/local/mysql/         |
| binlog_cache_size           | 32768                     |
| character_set_client        | utf8mb4                   |
| character_set_connection    | utf8mb4                   |
| character_set_database      | utf8mb4                   |
| character_set_filesystem    | binary                    |
| character_set_results       | utf8mb4                   |
| character_set_server        | utf8mb4                   |
| character_set_system        | utf8mb3                   |
| collation_connection        | utf8mb4_general_ci        |
| collation_database          | utf8mb4_general_ci        |
| collation_server            | utf8mb4_general_ci        |
| back_log                    | 80                        |
| max_connections             | 151                       |
| innodb_buffer_pool_size     | 134217728                 |
| innodb_log_file_size        | 50331648                  |
| query_cache_size            | 1048576                   |
| version                     | 8.0.21                    |
| uptime                      | 123456                    |
+-----------------------------+---------------------------+
```

시스템 변수(설정) 값이 어떻게 MySQL 서버와 클라이언트에 영향을 미치는지 판단하려면 각 변수가 글로벌 변수인지 세션 변수인지 구분할 수 있어야 한다. 그리고 이를 위해서는 우선 글로벌 변수와 세션 변수가 무엇이고 서로 어떤 관계가 있는지 명확히 이해해야 한다. MySQL 서버의 메뉴얼에서 시스템 변수(Server System Variables)를 설명한 페이지를 보면 MySQL 서버에서 제공하는 모든 시스템 변수의 목록과 간단한 설명을 참고할 수 있다.

```
+--------------------------+-----------+-------------+------------+-------------+---------+
| Variable_name            | Cmd-Line  | Option file | System Var | Var Scope   | Dynamic |
+--------------------------+-----------+-------------+------------+-------------+---------+
| auto_increment_increment | Yes       | Yes         | Yes        | Global      | Yes     |
| autocommit               | Yes       | Yes         | Yes        | Both        | Yes     |
| basedir                  | Yes       | Yes         | Yes        | Global      | No      |
| character_set_server     | Yes       | Yes         | Yes        | Global      | Yes     |
| max_connections          | Yes       | Yes         | Yes        | Global      | Yes     |
| innodb_buffer_pool_size  | Yes       | Yes         | Yes        | Global      | Yes     |
| version                  | No        | No          | No         | Global      | No      |
+--------------------------+-----------+-------------+------------+-------------+---------+

```

시스템 변수가 가지는 5가지 속성의 의미는 다음과 같다.

- Cmd-Line: MySQL 서버의 명령행 인자로 설정될 수 있는지 여부를 나타낸다. 즉, 이 값이 'Yes'이면 명령행 인자로 이 시스템 변수의 값을 변경하는 것이 가능하다는 의미다.
- Option file: MySQL의 설정 파일인 my.cnf(또는 my.ini)로 제어할 수 있는지 여부를 나타낸다. 옵션 파일이나 설정 파일 또는 컨피규레이션 파일 등은 전부 my.cnf(또는 my.ini) 파일을 지칭하는 것으로 같은 의미로 사용된다.
- System Var: 시스템 변수인지 아닌지를 나타낸다. MySQL 서버의 설정 파일을 작성할 때 각 변수명에 사용된 하이픈('-')이나 언더스코어(' _ ')의 구분에 주의해야 한다. 이는 MySQL 서버가 예전부터 수많은 사람들의 손을 거쳐오면서 생긴 일관성 없는 변수의 명명 규칙 때문이다. 어떤 변수는 하이픈으로 구분되고 어떤 시스템 변수는 언더스코어로 구분되는 등 애매모호한 부분이 있는데, 뒤늦게 이런 부분을 언더스코어로 통일해가는 중이다. 현재 MySQL 8.0에서는 모든 시스템 변수들이 ('_ ' )를 구분자로 사용하도록 변경된 것으로 보인다. 그리고 명령행 옵션으로만 사용 가능한 설정들은 하이픈을 구분자로 사용한다.
- Var Scope: 시스템 변수의 적용 범위를 나타낸다. 이 시스템 변수가 영향을 미치는 곳이 MySQL 서버 전체 (Global, 글로벌 또는 전역)를 대상으로 하는지, 아니면 MySQL 서버와 클라이언트 간의 커넥션(Session, 세션 또는 커넥션)만인지 구분한다. 그리고 어떤 변수는 세션과 글로벌 범위에 모두 적용(Both)되기도 한다. 
- Dynamic: 시스템 변수가 동적인지 정적인지 구분하는 변수이다.

# 글로벌 변수와 세션 변수

MySQL의 시스템 변수는 적용 범위에 따라 글로벌 변수와 세션 변수로 나뉘는데, 일반적으로 세션별로 적용되는 시스템 변수의 경우 글로벌 변수뿐만 아니라 세션 변수에도 동시에 존재한다. 이러한 경우 MySQL 메뉴얼의 'Var Scope'에는 'Both'라고 표시된다.

- 글로벌 범위의 시스템 변수는 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 마치는 시스템 변수를 의미하며, 주로 MySQL 서버 자체에 관련된 설정일 때가 많다. MySQL 서버에서 단 하나만 존재하는 InnoDB 버퍼 풀 크기(innodb_buffer_pool_size) 또는 MyISAM의 키 캐시 크기(key_buffer_size) 등이 가장 대표적인 글로벌 영역의 시스템 변수다.
- 세션 범위의 시스템 변수는 MySQL 클라이언트가 MySQL 서버에 접속할 때 기본으로 부여하는 옵션의 기본값을 제어하는 데 사용된다. 다른 DBMS에서도 비슷하겠지만 MySQL 에서도 각 클라이언트가 처음에 접속하면 기본적으로 부여하는 기본값을 가지고 있다. 별도로 그 값을 변경하지 않은 경우에는 그대로 값이 유지되지만, 클라이언트의 필요에 따라 개별 커넥션 단위로 다른 값을 변경할 수 있는 것이 세션 변수다. 여기서 기본값은 글로벌 시스템 변수이며, 각 클라이언트가 가지는 값이 세션 시스템 변수다. 각 클라이언트에서 쿼리 단위로 자동 커밋을 수행할지 여부를 결정하는 autocommit 변수가 대표적인 예라고 볼 수 있다. autocommit 을 ON 으로 설정해 두면 해당 서버에 접속하는 모든 커넥션은 기본으로 자동 커밋 모드로 시작되지만 각 커넥션에서 autocommit 변수의 값을 OFF 로 변경해 자동 커밋 모드를 비활성화할 수도 있다. 이러한 세션 변수는 커넥션별로 설정값을 서로 다르게 지정할 수 있으며, 한번 연결된 커넥션의 세션 변수는 서버에서 강제로 변경할 수 없다.
- 세션 범위의 시스템 변수 가운데 MySQL 서버의 설정 파일(my.cnf 또는 my.ini)에 명시해 초기화할 수 있는 변수는 대부분 범위가 'Both'라고 명시돼 있다. 이렇게 'Both'로 명시된 시스템 변수는 MySQL 서버가 기억만 하고 있다가 실제 클라이언트와의 커넥션이 생성되는 순가에 해당 커넥션의 기본값으로 사용되는 값이다. 그리고 순수하게 범위가 세션(Sesstion)이라고 명시된 시스템 변수는 MySQL 서버의 설정 파일에 초깃값을 명시할 수 없으며, 커넥션이 만들어지는 순간부터 해당 커넥션에서만 유효한 설정 변수를 의미한다.

# 정적 변수와 동적 변수

MySQL 서버의 시스템 변수는 MySQL 서버가 기동 중인 상태에서 변경 가능한지에 따라 동적 변수와 정적 변수로 구분된다. MySQL 서버의 시스템 변수는 디스크에 저장돼 있는 설정 파일(my.cnf)을 변경하는 경우와 이미 기동 중인 MySQL 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우로 구분할 수 있다. 디스크에 저장된 설정 파일의 내용은 변경하더라도 MySQL 서버가 재시작하기 전에는 적용되지 않는다. 하지만 SHOW 명령으로 MySQL 서버에 적용된 변숫값을 확인하거나 SET 명령을 이용해 값을 바꿀 수도 있다. 만약 변수명을 정확히 모른다면 SQL 문장의 LIKE 처럼 SHOW 명령에서 % 문자를 이용해 패턴 검색을 하는 것도 가능하다.

```
mysql> SHOW GLOBAL VARIABLES LIKE '%max_connections%';

+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+

mysql> SET GLOBAL max_connections=500;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+

```

하지만 SET 명령을 통해 변경되는 시스템 변숫값이 MySQL의 설정 파일인 my.cnf 파일에 반영되는 것은 아니기 때문에 현재 기동 중인 MySQL의 인스턴스에서만 유효하다. MySQL 서버가 재시작하면 다시 설정 파일의 내용으로 초기화되기 때문에 설정을 영구히 적용하려면 my.cnf 파일도 반드시 변경해야 한다. MySQL 8.0 버전부터는 SET PERSIST 명령을 이용하면 실행 중인 MySQL 서버의 시스템 변수를 변경함과 동시에 자동으로 설정 파일로도 기록된다. SHOW나 SET 명령에서 GLOBAL 키워드를 사용하면 글로벌 시스템 변수의 목록과 내용을 읽고 변경할 수 있으며, GLOBAL 키워드를 빼면 자동으로 세션 변수를 조회하고 변경한다.

일반적으로 글로벌 시스템 변수는 MySQL 서버의 기동 중에는 변경할 수 없는 것이 많지만 실시간으로 변경할 수 있는 것도 있다. my.cnf 설정 파일을 변경할 때 MySQL 서버를 재시작하는 경우가 많은데, 사실 변경하고자 하는 값이 동적 변수라면 SET 명령으로 간단히 변숫값을 변경할 수 있으며, 굳이 MySQL 서버를 재시작하지 않아도 된다. 이처럼 동적으로 시스템 변숫값을 변경하는 경우 SET 명령으로 시스템 변수를 변경하면 my.cnf 설정 파일에는 변경 내용이 기록되지 않는다. 설정 파일까지 내용을 변경하고자 한다면 SET PERSIST 명령을 사용해야 한다. SET PERSIST 명령을 사용하는 경우 변경된 시스템 변수는 my.cnf 파일이 아닌 별도의 파일에 기록된다.

시스템 변수의 범위가 'Both'인 경우(글로벌이면서 세션 변수인)에는 글로벌 시스템 변수의 값을 변경해도 이미 존재하는 커넥션의 세션 변숫값은 변경되지 않고 그대로 유지된다. 동적으로 변경 가능한 join_buffer_size라는 Both 타입 변수로 한번 확인해 보자.

```
-- 새로운 세션에서 초기 값 확인
mysql> SHOW GLOBAL VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 262144  |
+------------------+---------+

mysql> SHOW SESSION VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 262144  |
+------------------+---------+

-- 새로운 터미널 또는 클라이언트에서
mysql> SHOW SESSION VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 262144  |
+------------------+---------+

mysql> SET GLOBAL join_buffer_size = 1048576;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 1048576 |
+------------------+---------+

-- 기존 세션에서
mysql> SHOW SESSION VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 262144  |
+------------------+---------+

-- 새로운 터미널 또는 클라이언트에서
mysql> SHOW SESSION VARIABLES LIKE 'join_buffer_size';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| join_buffer_size | 1048576 |
+------------------+---------+

```

- 글로벌 시스템 변수 `join_buffer_size` 값을 변경하면, 새로운 커넥션에 대해서는 변경된 값이 적용되었다.
- 이미 존재하는 커넥션의 세션 변수 값은 변경되지 않고 그대로 유지되었다.

# SET PERSIST

MySQL 서버의 시스템 변수는 동적 변수와 정적 변수로 구분되는데, 동적 변수의 경우 MySQL 서버에서 SET GLOBAL 명령으로 변경하면 즉시 MySQL 서버에 반영된다. 예를 들어, MySQL 서버의 max_connections 라는 시스템 변수가 있는데, 이 시스템 변수는 MySQL 서버로 접속할 수 있는 최대 커넥션의 개수를 제한하는 동적 시스템 변수다. MySQL 서버에 커넥션을 많이 사용 중이라면 최대 연결 가능 커넥션의 개수를 더 늘리기 위해서 다음과 같이 MySQL 서버의 시스템 변수를 즉시 변경하게 될 것이다.

```
mysql> SET GLOBAL max_connections=5000;
```

문제는 이렇게 변경한 후 MySQL 서버의 설정 파일에서도 이 내용을 적용해야 하는데, 응급조치를 하다 보면 MySQL 서버의 설정 파일에 변경 내용을 적용하는 것을 잊어버릴 때도 있다. 그리고 시간이 지나서 MySQL 서버를 재시작하면 다시 예전의 max_connections 시스템 변수의 값으로 MySQL 서버가 시작되고, 이로 인해 장애가 반복적으로 발생하게 된다. 

MySQL 8.0 버전에서는 이러한 문제점을 보안하기 위해 SET PERSIST 명령을 도입했다.

```
mysql> SET PERSIST max_connections = 5000;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 5000  |
+-----------------+-------+

```

위 예제와 같이 SET PERSIST 명령으로 시스템 변수를 변경하면 MySQL 서버는 변경된 값을 즉시 적용함과 동시에 별도의 설정 파일(mysqld-auto.cnf)에 변경 내용을 추가로 기록해 둔다. 그리고 MySQL 서버가 다시 시작될 때 기본 설정 파일(my.cnf)뿐만 아니라 자동 생성된 mysqld-auto.cnf 파일을 같이 참조해서 시스템 변수를 적용한다. 즉, SET PERSIST 명령을 사용하면 MySQL 서버 설정 파일 (my.cnf)에 변경 내용을 수동으로 기록하지 않아도 자동으로 영구 변경이 되는 것이다.

SET PERSIST 명령은 세션 변수에는 적용되지 않으며, SET PERSIST 명령으로 시스템 변수를 변경하면 MySQL 서버는 자동으로 GLOBAL 시스템 변수의 변경으로 인식하고 변경한다. 현재 실행 중인 MySQL 서버에서는 변경 내용을 적용하지 않고 다음 재시작을 위해 mysqld-auto.cnf 파일에만 변경 내용을 기록 해두고자 한다면 SET PERSIST_ONLY 명령을 사용하면 된다.

```
mysql> SHOW GLOBAL VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+

mysql> SET PERSIST max_connections = 500;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+

mysql> SET PERSIST_ONLY max_connections = 1000;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+

```

SET PERSIST 명령이나 SET PERSIST_ONLY 명령으로 시스템 변수를 변경하면 다음과 같이 JSON 포맷의 mysqld-auto.cnf 파일이 생성된다. mysqld-auto.cnf 파일에는 변경된 시스템 변수의 이름과 설정값, 그리고 추가로 언제 누구에 의해 시스템 변수가 변경됐는지 등의 정보도 함께 기록된다. 

```
{
  "Version": "1.0",
  "mysql_server": {
    "max_connections": {
      "Value": 500,
      "Source": "user",
      "Set_time": "2024-08-01 12:00:00",
      "Set_by": "root@localhost"
    },
    "join_buffer_size": {
      "Value": 8388608,
      "Source": "user",
      "Set_time": "2024-08-01 12:05:00",
      "Set_by": "root@localhost"
    }
  }
}

```

SET PERSIST 또는 SET PERSIST_ONLY 명령으로 변경된 시스템 변수의 메타데이터는 다음과 같이 performance_schema.variables_info 뷰와 performance_schema.persksted_variables 테이블을 통해 참조할 수도 있다.


```
-- max_connections 값을 변경하고 영구적으로 저장
mysql> SET PERSIST max_connections = 500;
Query OK, 0 rows affected (0.00 sec)

-- join_buffer_size 값을 변경하고 다음 재시작 시 적용되도록 저장
mysql> SET PERSIST_ONLY join_buffer_size = 8388608;  -- 8MB
Query OK, 0 rows affected (0.00 sec)


mysql> SELECT * FROM performance_schema.variables_info
       WHERE VARIABLE_NAME IN ('max_connections', 'join_buffer_size');


mysql> SELECT * FROM performance_schema.persisted_variables
       WHERE VARIABLE_NAME IN ('max_connections', 'join_buffer_size');


+------------------+----------------+-------------------+----------------+----------------+-----------+----------------+---------------+
| VARIABLE_NAME    | VARIABLE_VALUE | VARIABLE_SOURCE   | DEFAULT_VALUE  | READ_ONLY     | DATA_TYPE | VARIABLE_SCOPE | ENUM_VALUE    |
+------------------+----------------+-------------------+----------------+----------------+-----------+----------------+---------------+
| max_connections  | 500            | COMPILED          | 151            | NO             | BIGINT    | GLOBAL         |               |
| join_buffer_size | 8388608        | PERSISTED         | 262144         | NO             | BIGINT    | BOTH           |               |
+------------------+----------------+-------------------+----------------+----------------+-----------+----------------+---------------+


+------------------+---------------+---------------------+--------------------+----------------+
| VARIABLE_NAME    | VARIABLE_VALUE | VARIABLE_SOURCE    | SET_TIME           | SET_BY         |
+------------------+----------------+---------------------+--------------------+----------------+
| max_connections  | 500            | PERSIST            | 2024-08-01 12:00:00| root@localhost |
| join_buffer_size | 8388608        | PERSIST_ONLY       | 2024-08-01 12:05:00| root@localhost |
+------------------+----------------+---------------------+--------------------+----------------+

```

SET PERSIST나 SET PERSIST_ONLY 명령으로 추가된 시스템 변수의 내용을 삭제해야 할 때도 있다. 이때 mysqld-auto.cnf 파일의 내용을 직접 변경하다가 내용상 오류를 만드는 경우 MySQL 서버가 시작되지 못할 수도 있다. 그래서 mysqld-auto.cnf 파일의 내용을 삭제해야 하는 경우에는 다음과 같이 RESET PERSIST 명령을 사용하는 것이 안전하다.


```
-- max_connections 변수의 영구 설정 삭제
mysql> RESET PERSIST max_connections;
Query OK, 0 rows affected (0.00 sec)

-- join_buffer_size 변수의 영구 설정 삭제
mysql> RESET PERSIST join_buffer_size;
Query OK, 0 rows affected (0.00 sec)
```

# my.cnf 파일

MySQL 8.0 서버의 시스템 변수는 대략 570개 수준이며, 사용하는 플러그인이나 컴포넌트에 따라 시스템 변수의 개수는 더 늘어날 수도 있다. 물론 모든 시스템을 공부해야 하는 것은 아니지만 MySQL 서버를 제대로 사용하려면 시스템 변수에 대한 이해가 상당히 많이 필요하다.






