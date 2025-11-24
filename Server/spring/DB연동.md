# Spring DB 연동

## DB 연동 방식

## JDBC API

### JDBC란
**Java DataBase Connectivity**  
Java에서 DBMS와 통신하기 위한 표준 API.

### JDBC 사용 방식

#### DriverManager

1. `DriverManager`에게 `getConnection()` 호출하여 DB 정보로 연결(Connection 생성)
2. Connection 객체에서 `prepareStatement(String sql)` 또는 `createStatement()`로 쿼리 전송
3. 쿼리 결과(ResultSet 등) 처리
4. Connection 닫기(해제)

<details>
<summary>Connection</summary>

DB와 통신을 할 수 있게 해주는, 연결을 표현한 객체이다.

</details>

<details>
<summary>DriverManager</summary>

JDBC 드라이버를 가져오고 URL등의 입력된 정보를 바탕으로 DB연결(Connection)을 생성해 주는 클래스
  
  
Connection 생성할 수 있는 DriverManager클래스의 메소드  
- `getConnection(String url, String user, String password)` : url에 DB URL 정보, user에 사용자 계정, password에 계정 비밀번호를 넣는다.
- `getConnection(String url)` : url에 (DB URL, user, password) 정보 들어가 있다.
- `getConnection(String url, Properties info)` : url에 DB URL정보가 들어 있고, Properties에 key, value로 user와 password가 들어가 있다.  


`DB URL` 구성요소  
- jdbc(프로토콜) : Subprotocol(DBMS종류)://Subname(접속 대상 위치 정보(host:port:databaseName))[옵션(?key=value)]

</details>

<details>
<summary>Driver</summary>

특정 장치(또는 시스템)와 OS/프로그램 사이를 이어주는 소프트웨어 인터페이스이다.
이 인터페이스를 구현한 프로그램을 XXX드라이버라고 한다.

</details>

**문제점**

- 매번 Connection을 새로 만드는 작업은 비용이 큼  
- Connection 생성/종료 코드가 반복됨  

<details>
<summary>Connection 비용이 큰 이유</summary>

Connection을 생성하기 위해 소켓 생성, 인증, 리소스 할당 과정이 필요하다.  
비용 비교 기준은 CPU 내부에서 끝나는 단순 연산이다.

- 소켓 생성과 인증 과정은 네트워크 IO가 포함되며, 패킷이 오가는 데 물리적 시간이 필요해 CPU 연산보다 훨씬 느리다.  
- DBMS에서는 세션 생성, 메모리·버퍼 초기화, 스레드/프로세스 준비 등의 작업을 수행하는데, 이 리소스 할당 과정이 전체 비용의 대부분을 차지한다.

</details>

**해결방법**

- DB 연결을 여러 개 미리 생성해 두고 재사용하기(Connection Pool)
- 개발자가 직접 Connection을 열고 닫는 반복 코드를 제거하고, 공통화된 관리 제공

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

---

## JPA, MyBatis

- JDBC API의 중복·반복 코드 문제를 해결하기 위해 등장한 프레임워크들  
- 내부적으로 JDBC를 사용하지만, SQL 작성과 매핑 코드를 간소화한다

<details>
<summary><code>피드백</code> DB 연동 방식을 바꿀 일이 생겼을 때(MySQL → PostgreSQL, JDBC → JPA),
어떤 구조로 프로젝트를 짜두면 변경 비용이 줄어드나요?</summary>

3-Layer 아키텍처를 적용하고 Repository를 인터페이스 기반으로 주입(DI)하면  
DB 종류나 연동 기술이 바뀌어도 Service와 Controller 코드를 수정할 필요가 없어  
변경 비용을 크게 줄일 수 있습니다.

- **DB 종류 변경(MySQL → PostgreSQL)**  
  DataSource 인터페이스 덕분에 설정만 변경하면 교체할 수 있어 코드 변경이 없습니다.

- **DB 기술 변경(JDBC → JPA)**  
  Repository **구현체**만 교체하면 되므로 변경 비용이 발생하지만  
  Service와 Controller는 영향을 받지 않습니다.

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

### 2. JdbcTemplate 조회 쿼리

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

예:  
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

# 3줄 요약
1. Java 프로그램에서 DB와 연동 하려면 JDBC를 사용해야된다.  
2. Spring에서 JDBC를 사용하는 방법중 하나는 JdbcTemplate이고, DataSource 객체를 생성해 JdbcTemplate에 주입 해 주어야 사용할 수 있다.  
3. Connection pool은 connection을 미리 연결해 두고 관리한다.