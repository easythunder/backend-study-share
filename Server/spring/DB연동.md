# Spring DB 연동

## JDBC API

### JDBC란
**Java DataBase Connectivity**  
Java에서 DBMS와 통신하기 위한 Java 표준 SQL 인터페이스 API

**JDBC의 목적**  
- Java 코드로 DB에 접속하고 SQL을 실행하고 결과를 받을 수 있게 함
- DBMS 종류가 달라져도 동일한 코드 패턴으로 개발 가능(드라이버만 교체)


### JDBC 사용 방식

1. JDBC 드라이버 로딩하기
2. DBMS서버와 접속하기(Connection 생성)
3. Statemet 또는 PreparedStatement  객체 생성하기
4. SQL문 실행 하기
5. 자원 해제


<!-- 
#### DriverManager

1. `DriverManager`에게 `getConnection()` 호출하여 DB 정보로 연결(Connection 생성)
2. Connection 객체에서 `prepareStatement(String sql)` 또는 `createStatement()`로 쿼리 전송
3. 쿼리 결과(ResultSet 등) 처리
4. Connection 닫기(해제)
-->
### JDBC 구성요소

#### 🔗 Connection

DB와 통신을 할 수 있게 해주는, 연결을 표현한 객체이다.

#### 🔌 Driver

- DBMS 벤더(MySQL, PostgreSQL, Oracle 등)가 제공하는 **JDBC 통신 프로토콜 구현체**
- java.sql.Driver 인터페이스를 구현한 클래스
- JDBC 규약에 따라 Connection 생성, PreparedStatement 생성 등을 수행

<details><summary>Driver</summary>

특정 장치(또는 시스템)와 OS/프로그램 사이를 이어주는 소프트웨어 인터페이스이다.
이 인터페이스를 구현한 프로그램을 XXX드라이버라고 한다.

</details>

<details><summary><code>💡 피드백</code> : 왜 JDBC Driver는 왜 DBMS 벤더마다 제공해야하나요? 표준 인터페이스가 있는데 각자 구현하는데 이유가 있나요?

</summary>

DBMS 마다 내부 프로토콜, 실행 엔진, 전송 형식, SQL 방언, 네트워크 규격 든 내부 구조가 다르기 때문에
각 벤더가 표준 인터페이스에 맞춰서 구현해 줘야 합니다.

그래야 사용자가 표준화된 방식으로 연결, 쿼리, 결과 조회를 할 수 있습니다.
</details>

#### 📤 Statement

Connection을 사용해 SQL을 실행 할수 있는 객체

Connection을 사용해 DB에 SQL문을 전송하고, DB에서 처리된 결과를 다시 자바프로그램쪽으로 전달 할 수 있게 해주는 객체

Connection이 도로라면, Statement는 데이터를 옮기는 트럭정도로 생각하면 된다.

<details>
<summary>Statement 🆚 PreparedStetment</summary>

**Statement**
```
Statement stmt = connection.createStatement();
stmt.executeUpdate("insert into test values (" + id + ", '" + password + "')");
```

- SQL을 그대로 문자열 상태로 전송하는 방식

- SQL Injection 위험 있음

- 매 실행마다 SQL을 새로 분석/컴파일하므로 비효율적

**PreparedStatement**

Statement를 확장한 인터페이스.
SQL 템플릿을 미리 컴파일하고, 바인딩 변수(?)를 사용해
반복 실행 시 성능 최적화 + SQL Injection 방지 효과 제공.
```
PreparedStatement pstmt =
    conn.prepareStatement("insert into test values(?, ?)");
pstmt.setString(1, id);
pstmt.setString(2, pwd);
pstmt.executeUpdate();
```

PreparedStatement의 핵심:

SQL Injection 방지

미리 컴파일된 SQL(Precompiled Statement)

실행 계획 캐싱(반복 실행 성능 ↑)

</details>

<details><summary><code>💡 피드백</code> : query()와 update()가 둘 다 PreparedStatement 기반이라면, Statement는 언제 사용하나요?</summary>

Statement는 파라미터 바인딩이 필요 없는 단순 SQL을 실행 하거나, DDL처럼 한번만 실행할 SQL을 다룰 때 사용합니다.

</details>

#### 📦 ResultSet

SQL 실행 결과를 담는(가지고 있는) 객체

#### 🚪 DriverManager

JDBC 드라이버를 가져오고 URL등의 입력된 정보를 바탕으로 DB연결(Connection)을 생성해 주는 클래스
  
  
Connection 생성할 수 있는 DriverManager클래스의 메소드  
- `getConnection(String url, String user, String password)` : url에 DB URL 정보, user에 사용자 계정, password에 계정 비밀번호를 넣는다.
- `getConnection(String url)` : url에 (DB URL, user, password) 정보 들어가 있다.
- `getConnection(String url, Properties info)` : url에 DB URL정보가 들어 있고, Properties에 key, value로 user와 password가 들어가 있다.  


DB URL 구성요소  
`jdbc(프로토콜) : Subprotocol(DBMS종류)://Subname(접속 대상 위치 정보(host:port:databaseName))[옵션(?key=value)]`


**문제점**

- 매번 Connection을 새로 만드는 작업은 비용이 큼  
- Connection 생성/종료 코드가 반복됨  

**해결방법**

- DB 연결을 여러 개 미리 생성해 두고 재사용하기(Connection Pool)
- 개발자가 직접 Connection을 열고 닫는 반복 코드를 제거하고, 공통화된 관리 제공


<details>
<summary>Connection 비용이 큰 이유</summary>

Connection을 생성하기 위해 소켓 생성, 인증, 리소스 할당 과정이 필요하다.  
비용 비교 기준은 CPU 내부에서 끝나는 단순 연산이다.

- 소켓 생성과 인증 과정은 네트워크 IO가 포함되며, 패킷이 오가는 데 물리적 시간이 필요해 CPU 연산보다 훨씬 느리다.  
- DBMS에서는 세션 생성, 메모리·버퍼 초기화, 스레드/프로세스 준비 등의 작업을 수행하는데, 이 리소스 할당 과정이 전체 비용의 대부분을 차지한다.

</details>

<details>
<summary>Connection Pool</summary>

Connection을 미리 여러 개 만들어 두고 “재사용”하는 방식.  
Tomcat JDBC Pool, HikariCP 같은 라이브러리가 관리한다.  
Connection을 계속 생성/종료하는 비용을 줄이고, 성능을 크게 높인다.

</details>

---

## DataSource

- DriverManager 대신 **Connection Pool 을 다루는 표준 인터페이스**
- Spring은 내부적으로 `DataSource`를 통해 Connection을 얻는다.
- Connection Pool 구현체(Tomcat JDBC, HikariCP 등)가 DataSource를 구현한다.  
- DataSource는 인터페이스이기 때문에, DB가 바뀌거나 connection pool이 바뀌어도 비즈니스코드 변경없이, DataSource구현체 클래스와 설정만 바꾸면 된다.

---

## JPA, MyBatis

- JDBC API의 중복·반복 코드 문제를 해결하기 위해 등장한 프레임워크들  
- 내부적으로 JDBC를 사용하지만, SQL 작성과 매핑 코드를 간소화한다

<details>
<summary><code>💡 피드백</code> : DB 연동 방식을 바꿀 일이 생겼을 때(MySQL → PostgreSQL, JDBC → JPA),
어떤 구조로 프로젝트를 짜두면 변경 비용이 줄어드나요?</summary>

3-Layer 아키텍처로 설계하고 DataSource로 DB연결을 추상화 하면 DB종류 변경 시 설정 파일만 수정하면 되고,  
Repository를 인터페이스 기반으로 구현하고 의존성을 주입(DI)하면 DB 접근 프레임워크 변경 시 결합도가 낮아져 변경 비용을 줄일 수 있습니다.

</details>

---

# Spring에서의 DB 연동 방식

## 1. DataSource 설정

Spring은 DataSource 빈을 등록하고,  
DB 연동이 필요한 빈들은 이 DataSource를 **주입받아 Connection을 사용한다.**

<details>
<summary>빈(Bean)</summary>

Spring 컨테이너에서 생성·관리되는 객체.

</details>

<details>
<summary>주입(Injection)</summary>

Spring이 빈을 다른 빈에 자동으로 넣어주는 기능.  
Constructor Injection(생성자 주입), Setter Injection(세터 주입) 등이 있음.

</details>

---

## 2. JdbcTemplate 사용

### JdbcTemplate이란

Spring이 제공하는 JDBC 편의 클래스.  
Connection 생성, PreparedStatement 생성, 예외 처리, 리소스 정리 등을 자동으로 수행해  
개발자는 **SQL과 매핑 코드만 작성**하면 된다.

### 1. JdbcTemplate 객체 생성

```java
JdbcTemplate template = new JdbcTemplate(dataSource);
```

DataSource를 생성자 주입하여 생성한다.

### 2. 🔍 query()
- **SELECT 다건 조회**에 사용

**주요 메소드**

`JdbcTemplate`의 `query()` 메서드를 사용하여 DB 데이터를 조회한다.

```java
List<T> query(String sql, RowMapper<T> rowMapper)
List<T> query(String sql, Object[] args, RowMapper<T> rowMapper)
List<T> query(String sql, RowMapper<T> rowMapper, Object... args)
```

- `RowMapper<T>` : ResultSet → 객체 변환 담당


<details>
<summary>Object... 은 참조 타입일까?</summary>

`...` 은 **가변 인자(Varargs)** 로, 마지막 파라미터에서 주어진 타입(T)의 인자를 0개 이상 받을 수 있다.

**예시:**  
`Object... args` → `Object arg1, Object arg2, ...` 여러 개 전달 가능

- Java 5에서 추가된 문법  
- 가변인자 작성 시 혼합 배열 표기법 사용하지 못한다.(컴파일 애러)  
- 예시 int[][]... i은 가능 하지만 int[]... i[]은 가능 하지 않다.  
- 컴파일 시 `T...` → `T[]` 배열로 변환됨 (언어 사양에 명시됨)  
- 즉, `Object... args` 는 내부적으로 `Object[] args` 와 동일  

</details>


<details>
<summary>Java Version History</summary>

| 변환 방식 | 브랜드           | 기술버전(JDK) |
|----------|------------------|----------------|
| 1        | Java 1.0         | JDK 1.0        |
|          | Java 1.1         | JDK 1.1        |
| 2        | Java 2 (1.2)     | JDK 1.2        |
|          | Java 2 (1.3)     | JDK 1.3        |
|          | Java 2 (1.4)     | JDK 1.4        |
| 3        | Java 5           | JDK 1.5        |
|          | Java 6           | JDK 1.6        |
|          | Java 7           | JDK 1.7        |
|          | Java 8           | JDK 1.8        |
| 4        | Java 9           | JDK 9          |
|          | Java 10          | JDK 10         |
|          | ...              | ...            |
|          | Java 11 (LTS)    | JDK 11         |
|          | Java 17 (LTS)    | JDK 17         |
|          | Java 21 (LTS)    | JDK 21         |

</details>


### 3. 🔍 queryForObject()
- **SELECT 결과가 정확히 1건일 때 사용**
    - 결과 0건 → `EmptyResultDataAccessException`
    - 결과 2건 이상 → `IncorrectResultSizeDataAccessException`

**주요 메소드**
```
T queryForObject(String sql, Class<T> requiredType)
T queryForObject(String sql, Class<T> requiredType, Object… args)
T queryForObject(String sql, RowMapper<T> rowMapper)
T queryForObject(String sql, RowMapper<T> rowMapper, Object… args)
```

</details>

<details><summary><code>💡 피드백</code> : 굳이 query()를 쓰지 않고 queryForObject()를 쓰는 이유가 뭔가요?</summary>

query()를 사용하게 되면 결과 값이 0건, 1건, 2건 이상일 때를 직접 검사해야 합니다.
하지만 queryForObject()를 사용하면 결과 개수에 따라 예외를 발생 시키므로, 코드를 더욱 간결하게 작성 할 수 있고 안전하다.

안전한 이유
예외를 정확히 처리하고 관리 할수 있어서 더 안전하다.
예외 조건을 명확히 하지 않으면 문제를 감지하기 어렵고, 오류를 처리하거나 추적하기도 어려워지기 때문이다.

</details>

### 4. 🔧 update()
- **INSERT, DELETE, UPDATE 실행**
- 내부적으로 **항상 PreparedStatement**를 사용함

**주요 메소드**
```
int update(String sql)
int update(String sql, Object… args)
```
- 반환값: **변경된 행(row)의 개수**

### 🧩 PreparedStatementCreator 사용 메소드
```
List<T> query(PreparedStatementCreator psc, RowMapper<T> rowMapper)
int update(PreparedStatementCreator psc)
int update(PreparedStatementCreator psc, KeyHolder generatedKeyHolder)
```

### 🏗 PreparedStatementCreator
[PreparedStatement]("")를 **직접 생성하고 옵션을 제어하기 위한 인터페이스**

> ※ 참고: `query()`와 `update()`는 내부적으로 모두 PreparedStatement 기반으로 동작한다.

**추상 메소드**
```
PreparedStatement createPreparedStatement(Connection connection)
```

**언제 사용하는가?**
- 단순 SQL 실행 이상이 필요할 때:
  - 자동 증가 키 반환(`auto increment`)
  - BLOB / CLOB 데이터 처리
  - PreparedStatement 생성 옵션 조정 (fetch size, timeout 등)
  - JDBC 드라이버 특수 기능 사용
  - Batch 튜닝
- 인덱스 파라미터(`?`)에 값을 `setXXX()`로 직접 지정해야 할 때


### 🔑 KeyHolder
자동 증가 컬럼(PK)을 조회하기 위한 객체

**사용 방식**
```
int update(PreparedStatementCreator psc, KeyHolder generatedKeyHolder)
```
PreparedStatementCreator 구현 시 

preparedStatement(String sql, String[] keyColumnNames)메소드를 사용하면 KeyHolder 객체에서 getKey() 로 자동 증가된 값을 확인 할 수 있다.

```
Long id = keyHolder.getKey().longValue();
```

getKey()는 Number 타입이므로 longValue()또는 intValue()로 변환하여 사용한다.


</details>

<details><summary><code>💡 피드백</code> : 자동 증가 PK를 KeyHolder로 받지 않고 SELECT LAST_INSERT_ID() 등을 직접 호출하면 어떤 문제점이 있나요?</summary>  

LAST_INSERT_ID()는 MySQL에서 현재 커넥션에서 마지막으로 수행된 AUTO_INCREMENT 값을 반환합니다.

커넥션 풀 환경에서는 커넥션을 여러 스레드에서 재사용할 수 있기 때문에, 내가 실행한 INSERT가 아닌 다른 스레드의 INSERT의 결과를 가져오는 동시성 문제가 발생할 수 있습니다.

한 트랜잭션 안에서 여러 테이블에 INSERT를 수행하면 어떤 테이블의 AUTO_INCREMENT값인지 명확하게 구분되지 않아 데이터 무결성이 깨질 위험이 있습니다.

LAST_INSERT_ID()는 MySQL의 고유 기능으로, 다른 DBMS마다 다른 방식으로 PK를 조회 해야 합니다.


</details>

## 🧯 Exception

### 🔍 Error와 Exception

`Error` : JVM 레벨에서 발생하며, 애플리케이션 코드로 복구할 수 없는 문제  
`Exception` : 프로그램이 처리하여 복구할 수 있는 문제  

```
Exception
├─ CheckedException
| ├─ IOException
| └─ SQLException
|
└─ UnCheckedException(RuntimeException)
  └─ IndexOutOfBoundsException

```

**Exception을 상속받는 클래스**  

1. Checked Exception: 반드시 try-catch로 처리하거나 throws로 선언해야 하며,
   처리하지 않으면 컴파일 오류가 발생한다.
2. Unchecked Exception(RuntimeException): 컴파일러가 예외 처리를 강제하지 않으며
   프로그램 실행 중 발생할 수 있다.  



#### 예외 처리 방식
1. `try-catch` : 해당 위치에서 예외를 직접 처리하는 방식
2. `throws` : 예외 처리를 호출한 메서드로 위임하는 방식

- `throw` 키워드를 사용하면 예외 객체를 직접 생성하여 던질 수 있다.  


### ⚠️ DB 연동 시 Exception  

JDBC가 설계될 때, 대부분의 메서드가 Checked Exception을 상속받은 SQLException을 던지도록 정의되었다.
이는 DB 벤더마다 서로 다른 오류 코드와 메시지를 제공하기 때문에, 개발자가 반드시 예외를 처리하도록 강제하기 위한 설계이다.

스프링에서는 JDBC의 SQLException을 그대로 사용하지 않고,
이를 RuntimeException을 상속받은 DataAccessException으로 변환하여 던진다.  

스프링에서 DataAccessException을 사용하는 이유는 다음과 같다:
- 연동 기술에 상관없이 동일한 예외 계층으로 처리하기 위해서이다.
Spring JDBC의 SQLException, JPA의 PersistenceException, Hibernate의 HibernateException 등
다양한 기술에서 발생하는 예외를 모두 DataAccessException 계층으로 변환한다.

따라서 개발자는 JDBC, JPA, Hibernate 등 어떤 기술을 사용하더라도 결국 DataAccessException만 처리하면 된다.
또한 DataAccessException은 RuntimeException 계열이므로,
CheckedException과 달리 컴파일 시점에 강제로 예외 처리를 요구하지 않는다.

<details>
<summary><code>💡 피드백</code>Checked Exception이 과도하면 코드에 어떤 문제가 생기나요?</summary>

메소드가 사용되는 곳에서 모두 예외 처리를 해주어야 하기 때문에, 불필요한 try/catch와 throws 선언이 누적되어 전체 코드가 복잡해 질 수 있습니다.
</details>

<details>
<summary><code>💡 피드백</code> : DataAccessException으로 변환해주는 핵심 컴포넌트는 무엇인가요?</summary>

- JDBC : SQLExceptionTanslator
- JPA, Hibernate : PersistenceExceptionTranslator

각 연동 기술에서 발생하는 예외를 스프링의 공통 예외 계층인
DataAccessException으로 변환해주는 컴포넌트입니다.

</details>


### 🗂️ DataAccessException 하위 클래스  

- **DuplicateKeyException** : 회원가입 시 중복 키 삽입 등
- **QueryTimeoutException** : DB 쿼리 시간 초과
- **BadSqlGrammarException** : SQL 문법 오류
- **CannotGetJdbcConnectionException** : DB 커넥션 획득 실패

<details>
<summary><code>💡 피드백</code> : DataAccessException은 어떤 방식으로 예외를 계층화하고 있나요?</summary>

데이터접근과정에서 발생한 문제의 재시도 가능 여부와 원인을 기준으로 예외를 분류 합니다.  

- Transient : 일시적 문제이며 동일 작업을 다시 시도하면 성공할 수 있음. 
- NonTransient : 재시도해도 절대 성공 불가능 (SQL 문법 오류, 제약 위반 등)  

```
DataAccessException
├── TransientDataAccessException
│     ├── DeadlockLoserDataAccessException
│     └── CannotGetJdbcConnectionException
│
└── NonTransientDataAccessException
      ├── BadSqlGrammarException
      ├── DataIntegrityViolationException
      │       └── DuplicateKeyException
      └── DataAccessResourceFailureException

```

</details>

<details>
<summary><code>💡 피드백</code> : QueryTimeoutException : DB 쿼리 시간 초과에서 쿼리 말고 커넥션 단계에서도 발생할 수 있나요?</summary>

SQLTimeoutException은 SQLException의 하위 클래스이기 때문에, 스프링의 SQLExceptionTranslator가 이를 QueryTimeoutException(DataAccessException 계열)으로 변환합니다.  

따라서 실제로 쿼리가 실행되지 않고 커넥션 단계에서 타임아웃이 발생하더라도 QueryTimeoutException이 발생할 수 있습니다.

</details>

<details>
<summary><code>💡 피드백</code> : BadSqlGrammarException : SQL 문법 오류에서 SQL 문법 오류 이외에 예외 발생하는 상황이 있나요?</summary>

DB가 SQL을 이해하지 못하는 모든 오류에서 발생합니다.  
SQL 문법이 잘못되었거나, 테이블·컬럼·스키마가 존재하지 않거나, DB가 SQL을 파싱할 수 없는 모든 상황에서 BadSqlGrammarException이 발생할 수 있습니다.

</details>


---


# 3줄 요약
1. Java 프로그램에서 DB와 연동 하려면 JDBC를 사용해야된다.  
2. Spring에서 JDBC를 사용하는 방법중 하나는 JdbcTemplate이고, DataSource 객체를 생성해 JdbcTemplate에 주입 해 주어야 사용할 수 있다.  
3. Connection pool은 connection을 미리 연결해 두고 관리한다.

**추가 요약**
1. select = query(), queryForObject()(1건 조회)  
insert delect update = update()  
모두 preparedStatement를 내부적으로 사용
2. preparedStatementCreator 사용으로 preparedStatement 옵션 지정 가능