MySQL에서 사용자 계정을 생성하는 방법이나 각 계정의 권한을 설정하는 방법은 다른 DBMS와는 조금 차이가 있다. 대표적으로 MySQL의 사용자 계정은 단순히 사용자의 아이디뿐 아니라 해당 사용자가 어느 IP에서 접속하고 있는지도 확인한다. 또한 MySQL 8.0 버전부터는 권한을 묶어서 관리하는 역할(Role)의 개념이 도입됐기 때문에 각 사용자의 권한으로 미리 준비된 권한 세트(Role)를 부여하는 것도 가능하다.

# 3.1 사용자 식별

MySQL의 사용자는 다른 DBMS와는 조금 다르게 사용자의 계정뿐 아니라 사용자의 접속 지점(클라이언트가 실행된 호스트명이나 도메인 또는 IP 주소)도 계정의 일부가 된다. 따라서 MySQL에서 계정을 언급할 때는 다음과 같이 항상 아이디와 호스트를 함께 명시해야 한다. 아이디와 IP 주소를 감싸는 역 따옴표는 MySQL에서 식별자를 감싸는 따옴표 역할을 하는데, 이는 종종 홀따옴표로도 바뀌어서 사용되기도 한다. 다음의 사용자 계정은 항상 MySQL 서버가 기동 중인 로컬 호스트에서 svc_id라는 아이디로 접속할 때만 사용될 수 있는 계정이다. 만약 사용자 계정에 다음과 같은 계정만 등록돼 있다면 다른 컴퓨터에서는 svc_id라는 아이디로 접속할 수 없음을 의미한다.
```
`svc_id`@`127.0.0.1`
```

만약 모든 외부 컴퓨터에서 접속이 가능한 사용자 계정을 생성하고 싶다면 사용자 계정의 호스트 부분을 % 문자로 대체하면 된다. 즉, % 문자는 모든 IP 또는 모든 호스트명을 의미한다. 사용자 계정 식별에서 또 한 가지 주의해야 할 점은 서로 동일한 아이디가 있을 때 MySQL 서버가 해당 사용자의 인증을 위해 어떤 계정을 선택하느냐다. 예를 들어, 다음과 같은 2개의 사용자 계정이 있는 MySQL 서버가 있다고 해보자.

```
'svc_id'@'192.168.0.10' ( 이 계정의 비밀번호는 123)
'svc_id'@'%' (이 계정의 비밀번호는 abc)
```

IP 주소가 192.168.0.10인 PC에서 이 MySQL 서버에 접속할 때 MySQL 서버가 첫 번째 계정 정보를 이용해 인증을 실행할지, 아니면 두 번째 계정 정보를 이용할지에 따라 이 접속은 성공할 수도 있고 실패할 수도 있다. MySQL은 둘 중에서 어떤 것을 선택할까? 권한이나 계정 정보에 대해 MySQL은 범위가 가장 작은 것을 항상 먼저 선택한다. 즉, 위의 두 계정 정보 가운데 범위가 좁은 것은 %가 포함되지 않은 'svc_id'@'192.168.0.10'이기 때문에 IP가 명시된 계정 정보를 이용해 이 사용자를 인증하게 된다. 이 사용자가 IP가 192.168.0.10인 PC에서 'svc_id'라는 아이디와 abc라는 비밀번호로 로그인하면 '비밀번호가 일치하지 않는다' 라는 이유로 접속이 거절될 것이다. 의도적으로 이처럼 중첩된 계정을 생성하지는 않겠지만 실수로 자주 이 같은 상황이 발생할 수 있으므로 사용자 계정을 생성할 때 주의해야 한다.

# 사용자 계정 관리

## 시스템 계정과 일반 계정

MySQL 8.0부터 계정은 SYSTEM_USER 권한을 가지고 있느냐에 따라 시스템 계정(System Account)과 일반 계정(Regular Account)으로 구분된다. 여기서 소개하는 시스템 계정은 MySQL 서버 내부적으로 실행되는 백그라운드 스레드와는 무관하며, 시스템 계정도 일반 계정과 같이 사용자를 위한 계정이다. 시스템 계정은 데이터베이스 서버 관리자를 위한 계정이며, 일반 계정은 응용 프로그램이나 개발자를 위한 계정 정도로 생각하면 이해하기 쉬울 것이다. 

시스템 계정은 시스템 계정과 일반 계정을 관리(생성 삭제 및 변경)할 수 있지만 일반 계정은 시스템 계정을 관리할 수 없다. 또한 다음과 같이 데이터베이스 서버 관리와 관련된 중요 작업은 시스템 계정으로만 수행할 수 있다.

- 계정 관리(계정 생성 및 삭제, 그리고 계정의 권한 부여 및 제거)
- 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

이렇게 시스템 계정과 일반 계정의 개념이 도입된 것은 DBA 계정에는 SYSTEM_USER 권한을 할당하고 일반 사용자를 위한 계정에는 SYSTEM_USER 권한을 부여하지 않게 하기 위해서다.

## 참고

일반적으로 데이터베이스에서는 계정과 사용자라는 말을 혼용하는데, 이번 장에서는 설명의 편의를 위해 '사용자'와 '계정'이라는 단어를 다음과 같이 구분해서 사용하겠다.

- 사용자: MySQL 서버를 사용하는 주체(사람 또는 응용 프로그램)
- 계정: MySQL 서버에 로그인하기 위한 식별자(로그인 아이디)

MySQL 서버에는 다음과 같이 내장된 계정들이 있는데, 'root'@'localhost'를 제외한 3개의 계정은 내부적으로 각기 다른 목적으로 사용되므로 삭제되지 않도록 주의하자.

- 'mysql.sys'@'localhost' : MySQL 8.0부터 기본으로 내장된 sys 스키마의 객체(뷰나 함수, 그리고 프로시저)들의 DEFINER로 사용되는 계정
- 'mysql.session'@'localhost': MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- 'mysql.infoschema'@'localhost': information_schema에 정의된 뷰의 DEFINER로 사용되는 계정

위에 언급한 3개의 계정은 처음부터 잠겨(account_locked 칼럼) 있는 상태이므로 의도적으로 잠긴 계정을 풀지 않는 한 악의적인 용도로 사용할 수 없으므로 보안을 걱정하지는 않아도 된다.

```sql
mysql>  SELECT user, host, account_locked FROM mysql.user WHERE user LIKE 'mysql.%';

+------------------+-----------+----------------+

| user             | host      | account_locked |

+------------------+-----------+----------------+

| mysql.infoschema | localhost | Y              |

| mysql.session    | localhost | Y              |

| mysql.sys        | localhost | Y              |

+------------------+-----------+----------------+

3 rows in set (0.01 sec)
```

## 계정 생성

MySQL 5.7 버전까지는 GRANT 명령으로 권한의 부여와 동시에 계정 생성이 가능했다. 하지만 
MySQL 8.0 버전부터는 계정의 생성은 CREATE USER 명령으로, 권한 부여는 GRANT 명령으로 구분해서 실행하도록 바뀌었다. 계정을 생성할 때는 다음과 같은 다양한 옵션을 설정할 수 있다.

- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
- 기본 역할(Role)
- SSL 옵션
- 계정 잠금 여부

일반적으로 많이 사용되는 옵션을 가진 CREATE 명령은 다음과 같다.

```sql
mysql> CREATE USER 'user'@'%'

    ->     IDENTIFIED WITH 'mysql_native_password' BY 'password'

    ->     REQUIRE NONE

    ->     PASSWORD EXPIRE INTERVAL 30 DAY

    ->     ACCOUNT UNLOCK

    ->     PASSWORD HISTORY DEFAULT

    ->     PASSWORD REUSE INTERVAL DEFAULT

    ->     PASSWORD REQUIRE CURRENT DEFAULT;

Query OK, 0 rows affected (0.02 sec)
```

위 예제의 각 옵션을 하나씩 살펴보자.

## IDENTIFIED WITH

사용자의 인증 방식과 비밀번호를 설정한다. IDENTIFIED WITH 뒤에는 반드시 인증 방식(인증 플러그인의 이름)을 명시해야 하는데, MySQL 서버의 기본 인증 방식을 사용하고자 한다면 IDENTIFIED BY 'password' 형식으로 명시해야 한다. MySQL 서버에서는 다양한 인증 방식을 플러그인 형태로 제공하며, 다음 4가지 방식이 가장 대표적이다.

- Native Pluggable Authentication: MySQL 5.7 버전까지 기본으로 사용되던 방식으로, 단순히 비밀번호에 대한 해시(SHA-1 알고리즘) 값을 저장해두고, 클라이언트가 보낸 값과 해시값이 일치하는지 비교하는 인증 방식이다.
- Caching SHA-2 Pluggable Authentication: MySQL 5.6 버전에 도입되고 MySQL 8.0 버전에서는 조금 더 보완돤 인증 방식으로, 암호화 해시값 생성을 위해 SHA-2(256비트)알고리즘을 사용한다. Native Authentication과의 가장 큰 차이는 사용되는 암호화 해시 알고리즘의 차이이며, SHA-2 Authentication은 저장된 해시값의 보안에 더 중점을 둔 알고리즘으로 이해할 수 있다. Native Authentication 플러그인은 입력이 동일 해시값을 출력하지만, Caching SHA-2 Authentication은 내부적으로 Salt 키를 활용하며, 수천 번의 해시 계산을 수행해서 결과를 만들어 내기 때문에 동일한 키 값에 대해서도 결과가 달라진다. 이처럼 해시값을 계산하는 방식은 상당히 시간 소모적이어서 성능이 매우 떨어지는데, 이를 보안하기 위해 MySQL 서버는 해시 결괏값을 메모리에 캐시해서 사용하게 된다. 그래서 인증 방식의 이름에 'Caching'이 포함된 것이다. 이 인증 방식을 사용하려면 SSL/TLS 또는 RSA 키페어를 반드시 사용해야 하는데, 이를 위해 클라이언트에서 접속할 때 SSL 옵션을 활성화해야 한다.
- PAM Pluggable Authentication: 유닉스나 리눅스 패스워드 또는 LDAP(Lightweight Directory Access Protocol) 같은 외부 인증을 사용할 수 있게 해주는 인증 방식으로, MySQL 엔터프라이즈 에디션에서만 사용 가능하다.
- LDAP Pluggable Authentication: LDAP을 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식으로, MySQL 엔터프라이즈 에디션에서만 사용 가능하다.

MySQL 5.7 버전까지는 Native Authentication이 기본 인증 방식으로 사용됐지만 MySQL 8.0 버전부터는 Caching SHA-2 Authentication이 기본 인증으로 바뀌었다. 하지만 Caching SHA-2 Authentication은 SSL/TLS 또는 RSA 키페어를 필요로 하기 때문에 기존 MySQL 5.7 까지의 연결 방식과는 다른 방식으로 접속해야 한다. 그래서 보안 수준은 좀 낮아지겠지만 기존 버전과의 호환성을 고려한다면 Caching SHA-2 Authentication 보다는 Native Authentication 인증 방식으로 계정을 생성해야 할 수도 있다. 만약 MySQL 8.0에서도 Native Authentication을 기본 인증 방식으로 설정하고자 한다면 다음과 같이 MySQL 설정을 변경하거나 my.cnf 설정 파일에 추가하면 된다.

```sql
// Native Authentication을 기본 인증 방식으로 설정
mysql> SET GLOBAL default_authentication_plugin="mysql_native_password"
```

CREATE USER 또는 ALTER USER 명령을 이용해 MySQL 서버의 계정을 생성 또는 변경할 때 연결 방식과 비밀번호 옵션, 자원 사용과 관련된 여러 옵션을 설정할 수 있다. 

## REQUIRE

MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다. 만약 별도로 설정하지 않으면 비암호화 채널로 연결하게 된다. 하지만 REQURE 옵션을 SSL로 설정하지 않았다고 하더라도 Caching SHA-2 Authentication 인증 방식을 사용하면 암호화된 채널만으로 SSL/TLS 서버에 접속할 수 있게 된다.

## PASSWORD EXPRIE

비밀번호의 유효 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 default_password_lifetime 시스템 변수에 저장된 기간으로 유효 기간이 설정된다. 개발자나 데이터베이스 관리자의 비밀번호는 유효 기간을 설정하는 것이 보안상 안전하지만 응용 프로그램 접속용 계정에 유효 기간을 설정하는것은 위험할 수 있으니 주의하자. PASSWORD EXPRIE 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD EXPIRE: 계정 생성과 동시에 비밀번호의 만료 처리
- PASSWORD EXPIRE NEVER: 계정 비밀번호의 만료 기간 없음
- PASSWORD EXPRIE DEFAULT: default_password_lifetime 시스템 변수에 저장된 기간으로 비밀번호의 유효 기간을 설정
- PASSWORD EXPIRE INTERVAL n DAY: 비밀번호의 유효 기간을 오늘부터 n일자로 설정

## PASSWORD HISTORY

한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션인데, PASSWORD HISTORY 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD HISTORY DEFAULT: password_history 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용 할 수 없다.
- PASSWORD HISTORY n: 비밀번호의 이력을 최근 n개까지만 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.

한 번 사용했던 비밀번호를 사용하지 못하게 하려면 이전에 사용했던 비밀번호를 MySQL 서버가 기억하고 있어야 하는데, 이를 위해 MySQL 서버는 mysql DB의 password_history 테이블을 사용한다. 

```sql
mysql> SELECT * FROM mysql.password_history;

Empty set (0.00 sec)  
```
(설정하지 않아서 없다 ㅎ)

## PASSWORD REUSE INTERVAL

한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 password_reuse_interval 시스템 변수에 저장된 기간으로 설정된다. PASSWORD REUSE INTERVAL 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD REUSE INTERVAL DEFAULT: password_reuse_interval 변수에 저장된 기간으로 설정
- PASSWORD REUSE INTERVAL n DAY: n 일자 이후에 비밀번호를 재사용할 수 있게 설정

# PASSWORD REQUIRE

비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호(변경하기 전 만료된 비밀번호)를 필요로 할지 말지를 결정하는 옵션이며, 별도로 명시되지 않으면 password_require_current 시스템 변수의 값으로 설정된다. PASSWORD REQUIRE 절에 사용 가능한 옵션은 다음과 같다.

- PASSWORD REQUIRE CURRENT: 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- PASSWORD REQUIRE OPTIONAL: 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- PASSWORD REQUIRE DEFAULT: password_require_current 시스템 변수의 값으로 설정

## ACCOUNT LOCK / UNLOCK

계정 생성 시 또는 ALTER USER 명령을 사용해 계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 여부를 결정한다.

- ACCOUNT LOCK: 계정을 사용하지 못하게 잠금 
- ACCOUNT UNLOCK: 잠긴 계정을 다시 기능 상태로 잠금 해제

# 비밀번호 관리

## 고수준 비밀번호

MySQL 서버의 비밀번호는 유효기간이나 이력 관리를 통한 재사용 금지 기능뿐만 아니라 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 있다. MySQL
 서버에서 비밀번호의 유효성 체크 규칙을 적용하려면 validate_password 컴포넌트를 이용하면 되는데, 우선 다음과 같이 validate_password 컴포넌트를 설치해야 한다. validate_password 컴포터는트는 MySQL
서버 프로그램에 내장돼 있기 때문에 INSTALL COMPONENT 명령의 file:// 부분에 별도의 파일 경로를 지정하지 않아도 된다.

```sql
mysql> INSTALL COMPONENT 'file://component_validate_password';

Query OK, 0 rows affected (0.75 sec)

  

mysql> SELECT * FROM mysql.component;

+--------------+--------------------+------------------------------------+

| component_id | component_group_id | component_urn                      |

+--------------+--------------------+------------------------------------+

|            1 |                  1 | file://component_validate_password |

+--------------+--------------------+------------------------------------+

1 row in set (0.00 sec)
```

validate_password 컴포넌트가 설치되면 다음과 같이 컴포넌트에서 제공하는 시스템 변수를 확인할 수 있다.

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'validate_password%';

+-------------------------------------------------+--------+

| Variable_name                                   | Value  |

+-------------------------------------------------+--------+

| validate_password.changed_characters_percentage | 0      |

| validate_password.check_user_name               | ON     |

| validate_password.dictionary_file               |        |

| validate_password.length                        | 8      |

| validate_password.mixed_case_count              | 1      |

| validate_password.number_count                  | 1      |

| validate_password.policy                        | MEDIUM |

| validate_password.special_char_count            | 1      |

+-------------------------------------------------+--------+

8 rows in set (0.02 sec)
```

비밀번호 정책은 크게 다음 3가지 중에서 선택할 수 있으며, 기본값은 MEDIUM으로 자동 설정된다.

- LOW: 비밀번호의 길이만 검증
- MEDIUM: 비밀번호의 길이를 검증하며, 숫자와 대소문자, 그리고 특수문자의 배합을 검증
- STRONG: MEDIUM 레벨의 검증을 모두 수행하며, 금칙어가 포함됐는지 여부까지 검증

비밀번호의 길이는 validate_password.length 시스템 변수에 설정된 길이 이상의 비밀번호가 사용됐는지를 검증하며, 숫자와 대소문자, 특수문자 validate_password.mixed_case_count 와 validate_password.number_count, validate_password.special_char_count 시스템 변수에 설정된 글자 수 이상을 포함하고 있는지 검증한다. 그리고 마지막으로 금칙어는 validate_password.dictinonary_file 시스템 변수에 설정된 사전 파일에 명시된 단어를 포함하고 있는지를 검증한다.

MySQL 서버에서는 기본적으로 비밀번호에 'qwerty'나 '1234'와 같이 연속된 단어를 사용해도 아무런 에러 없이 설정된다. 하지만 높은 수준의 보안을 요구하는 서비스에서는 비밀번호를 사전에 명시되지 않은 단어들로 생성하도록 제어해야 할 수도 있다. 이러한 요구사항이 필요한 경우에는 validate_password.dictionary_file 시스템 변수에 금칙어들이 저장된 사전 파일을 등록하면 된다.

## 이중 비밀번호

일반적으로 많은 응용 프로그램 서버들이 공용으로 데이터베이스 서버를 사용하기 때문에 데이터베이스 서버의 계정 정보는 응용 프로그램 서버로부터 공용으로 사용되는 경우가 많다. 이러한 구현 특성으로 인해 데이터베이스 서버의 계정 정보는 쉽게 변경하기가 어려운데, 그중에서도 데이터베이스 계정의 비밀번호는 서비스가 실행 중인 상태에서 변경이 불가능했다. 그래서 서비스에서 데이터베이스 계정의 비밀번호는 처음 설정한 상태로 몇 년 동안 사용되는 경우가 많다. 데이터베이스 계정의 비밀번호는 보안을 위해 주기적으로 변경해야 하지만 지금까지는 서비스를 모두 멈추지 않고서는 비밀번호를 변경하는 것은 불가능한 일이었다.

이 같은 문제점을 해결하기 위해 MySQL 8.0 버전부터는 계정의 비밀번호로 2개의 값을 동시에 사용할 수 있는 기능을 추가했다. MySQL 서버 메뉴얼에서는 이 기능을 '이중 비밀번호(Dual Password)'라고 소개한다. 

MySQL 서버의 이중 비밀번호 기능은 하나의 계정에 대해 2개의 비밀번호를 동시에 설정할 수 있는데, 2개의 비밀번호는 프라이머리(Primary)와 세컨더리(Secondary)로 구분된다. 최근에 설정된 비밀번호는 프라이머리 비밀번호이며, 이전 비밀번호는 서컨더리 비밀번호가 된다. 이중 비밀 비밀번호를 사용하려면 다음과 같이 기존 비밀번호 변경 구문에 RETAIN CURRENT PASSWORD 옵션만 추가하면 된다.

# 권한(Privilege)

MySQL 5.7 버전까지 권한은 글로벌(Global) 권한과 객체 단위의 권한으로 구분됐다. 데이터베이스나 테이블 이외의 객체에 적용되는 권한을 글로벌 권한이라고 하며, 데이터베이스나 테이블을 제어하는 데 필요한 권한을 객체 권한이라 한다. 객체 권한은 GRANT 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야 하며, 반대로 글로벌 권한은 GRANT 명령에서 특정 객체를 명시하지 말아야 한다. 예외적으로 ALL(또는 ALL PRIVILEGES)은 글로벌과 객체 권한 두 가지 용도로 사용될 수 있는데, 특정 객체에 ALL 권한이 부여되면 해당 객체에 적용될 수 있는 모든 객체 권한을 부여하며, 글로벌로 ALL이 사용되면 글로벌 수준에서 가능한 모든 권한을 부여하게 된다.
(자세한 권한의 예시는 메뉴얼을 참고하자)

사용자에게 권한을 부여할 때는 GRANT 명령을 사용한다. GRANT 명령은 다음과 같은 문법으로 작성하는데, 각 권한의 특성(범위)에 따라 GRANT 명령의 ON 절에 명시되는 오브젝트(DB나 테이블)의 내용이 바뀌어야 한다. 

```sql
mysql> GRANT privilege_list ON db.table TO 'user'@'host';
```

MySQL 8.0 버전 부터는 존재하지 않는 사용자에 대해 GRANT 명령이 실행되면 에러가 발생하므로 반드시 사용자를 먼저 생성하고 GRANT 명령으로 권한을 부여해야 한다. GRANT OPTION 권한은 다른 권한과 달리 GRANT 명령의 마지막에 WITH GRANT OPTION을 명시해서 부여한다. privilege_list에는 구분자(,)를 써서 앞의 표에 명시된 권한 여러 개를 동시에 명시할 수 있다. TO 키워드 뒤에는 권한을 부여할 대상 사용자를 명시하고, ON 키워드 뒤에는 어떤 DB의 어떤 오브젝트에 권한을 부여할지 결정할 수 있는데, 권한의 범위에 따라 사용하는 방법이 달라진다. 간단하게 예제를 한 번 살펴보자.

글로벌 권한
```
mysql> GRANT SUPER ON *.* TO 'user'@'localhost';
```

글로벌 권한은 특정 DB나 테이블에 부여될 수 없기 때문에 글로벌 권한을 부여할 때 GRANT 명령의 ON 절에는 항상 * . *  을 사용하게 된다.  * . *  은 모든 DB의 모든 오브젝트(테이블과 스토어드 프로시저나 함수 등)를 포함해서 MySQL 서버 전체를 의미한다. CREATE USER 나 CREATE ROLE과 같은 글로벌 권한은 DB 단위나 오브젝트 단위로 부여할 수 있는 권한이 아니므로 항상  * . *  로만 대상을 사용할 수 있다. 

DB 권한
```sql
mysql> GRANT EVENT ON  *.* TO 'user'@'localhost';
mysql> GRANT EVENT ON employees.* TO 'user'@'localhost'l
```

DB 권한은 특정 DB에 대해서만 권한을 부여하거나 서버에 존재하는 모든 DB에 대해 권한을 부여할 수 있기 때문에 위의 예제와 같이 ON절에  * . *  이나 employees.* 모두 사용할 수 있다. 여기서 DB라 함은 DB 내부에 존재하는 테이블뿐만 아니라 스토어드 프로그램들도 모두 포함된다.

하지만 DB 권한만 부여하는 경우에는(DB 권한은 테이블에 대해 부여할 수 없기 때문에) employees.department 와 같이 테이블까지 명시할 수 없다. DB 권한은 서버의 모든 DB에 적용할 수 있으므로 대상에  * . *  을 사용할 수 있다. 또한 특정 DB에 대해서만 권한을 부여하는 것도 가능하기 때문에 db.* 로 대상을 설정하는 것도 가능하다. 하지만 오브젝트 권한처럼 db.table 로 오브젝트(테이블)까지 명시할 수는 없다.

테이블 권한
```sql
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user'@'localhost';
```

테이블 권한은 첫 번째 예제와 같이 서버의 모든 DB에 대해 권한을 부여하는 것도 가능하며, 두 번째 예제와 같이 특정 DB의 오브젝트에 대해서만 권한을 부여하는 것도 가능하다. 그리고 세 번째 예제와 같이 특정 DB의 테이블에 대해서만 권한을 부여하는 것도 가능하다.

테이블의 특정 칼럼에 대해서만 권한을 부여하는 경우에는 GRANT 명령의 문법이 조금 달라져야 한다. 칼럼에 부여할 수 있는 권한은 DELETE를 제외한 INSERT, UPDATE, SELECT로 3가지이며, 각 권한 뒤에 칼럼을 명시하는 형태로 부여한다. employees DB의 department 테이블에서 dept_name 칼럼을 업데이트할 수 없게 권한을 부여하려면 다음과 같이 GRANT 명령을 사용하면 된다. 이 경우 SELETE나 INSERT는 모든 칼럼에 대해 수행하 수 있지만 UPDATE는 dept_name 칼럼에 대해서만 수행할 수 있다.

```sql
mysql> GRANT SELECT, INSERT, UPDATE(dept_name) ON employees.department TO 'user'@'localhost';
```

여러 가지 레벨이나 범위로 권한을 설정하는 것이 가능하지만 테이블이나 칼럼 단위의 권한은 잘 사용하지 않는다. 칼럼 단위의 권한이 하나라도 설정되면 나머지 모든 테이블의 모든 칼럼에 대해서도 권한 체크를 하기 때문에 칼럼 하나에 대해서만 권한을 설정하더라도 전체적인 성능에 영향을 미칠 수 있다. 칼럼 단위의 접근 권한이 꼭 필요하다면 GRANT 명령으로 해결하기보다는 테이블에서 권한을 허용하고자 하는 컬럼만으로 별도의 뷰(VIEW)를 만들어서 사용하는 방법도 생각해볼 수 있다.  뷰도 하나의 테이블로 인식되기 때문에 뷰를 만들어 두면 뷰의 칼럼에 대해 권한을 체크하지 않고 뷰 자체에 대한 권한만 체크하게 된다.

각 계정이나 권한에 부여된 권한이나 역할을 확인하기 위해서는 SHOW GRANT 명령을 사용할 수도 있지만 표 형태로 깔끔하게 보고자 한다면 mysql DB의 권한 관련 테이블을 참조하면 된다.

# 역할(ROLE)

MySQL 8.0 버전부터 권한을 묶어서 역할(Role)을 사용할 수 있게 됐다. 실제 MySQL 서버 내부적으로 역할(Role)은 계정과 똑같은 모습을 하고 있다. 간단히 MySQL 8.0에서 역할을 사용하는 예시를 한번 살펴보자.

우선 CREATE ROLE 명령을 이용해 role_emp_write라는 이름의 역할을 정의한다.

```
mysql> CREATE ROLE
		role_emp_read,
		role_emp_write;
```

위의 CREATE ROLE 명령에서는 빈 껍데기만 있는 역할을 정의한 것이며, 다음과 같이 GRANT 명령으로 각 역할에 대해 실질적인 권한을 부여하면 된다. 역할의 이름대로 'role_emp_read' 역할에는 employees DB의 모든 객체에 대해 읽기(SELET) 권한만 부여했으며, 'role_emp_write' 역할에는 employees DB의 모든 객체에 대해 데이터 변경(INSERT, UPDATE, DELETE) 권한을 부여했다.

```sql
mysql> GRANT SELECT ON employees.* TO role_emp_read;
mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```

기본적으로 역할은 그 자체로 사용될 수 없고 계정에 부여해야 하므로 CREATE USER 명령으로 reader와 writer라는 계정을 생성해보자.

```
mysql> CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
mysql> CREATE USER wirter@'127.0.0.1' IDENTIFIED BY 'qwerty';
```

CREATE USER 명령으로 계정은 생성됐지만 실제 이 계정들은 아무런 권한이 부여되지 않았으므로 employees DB에 대해 아무런 쿼리도 실행할 수 없는 상태다. 이제 GRANT 명령으로 reader와 writer 계정에 역할을 부여해보자.
```sql
mysql> GRANT role_emp_read TO reader@'127.0.0.1';
mysql> GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```

reader 계정에는 role_emp_read 역할만 부여했으며, writer 계정에는 role_emp_read와 role_emp_write 역할을 부여했다. SHOW GRANT 명령으로 계정이 가진 권한을 확인해봐도 role_emp_read 역할이 잘 부여된 것을 확인할 수 있다.

그런데 지금 상태에서 reader나 writer 계정으로 로그인해서 employees DB의 테이터를 조회하거나 변경하려고 하면 권한이 없다는 에러를 만나게 될 것이다.

실제 역할은 부여돼 있지만 계정의 활성화된 역할을 조회해 보면 role_emp_read 역할이 없음을 확인할 수 있다.

reader 계정이 role_emp_read 역할을 사용할 수 있게 하려면 다음과 같이 SET ROLE 명령을 실행해 해당 역할을 활성화해야 한다. 일단 역할이 활성화되면 그 역할이 가진 권한은 사용할 수 있는 상태가 되지만 계정이 로그아웃됐다가 다시 로그인하면 역할이 활성화되지 않은 상태로 초기화돼 버린다. 

MySQL 서버의 역할이 불편하고 수동적으로 보이는데, 이는 MySQL 서버의 역할이 자동으로 활성화 되지 않게 설정돼 있기 때문이다. 사용자가 MySQL 서버에 로그인할 때 역할을 자동으로 활성화할지 여부를 activate_all_roles_on_login 시스템 변수로 설정할 수 있다. 다음과 같이 active_all_roles_on_login 시스템 변수가 ON으로 설정되면 매번 SET ROLE 명령으로 역할을 활성화하지 않아도 로그인과 동시에 부여된 역할이 자동으로 활성화된다.

```sql
mysql> SET GRLOBAL activate_all_roles_on_login=ON;
```
