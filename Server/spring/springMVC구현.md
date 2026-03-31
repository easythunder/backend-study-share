# SpringMVC

## Server

### Web Server VS Web Application Server
- Server : 서비스를 제공하는 컴퓨터
- Web Server : 웹에서 서비스를 제공하는 컴퓨터, http 요청을 받아 정적 리소스를 제공하거나, 동적요청을 was로 전달하는 서버 
    - 어떤 서비스? ip를 통해 컴퓨터에 접근해서 파일을 볼수 있게하는 서비스
- Web Application Server : 웹서버가 파일을 전달할때 여러 값들을 처리하기위해 애플리케이션을 사용해 여러 값을 처리 해주는 서버, 애플리케이션을 실행하여 동적인 응답을 생성하는 서버

서버 구조
```
[Client]
   ↓
[Nginx]  ← 웹서버
   ↓
[Tomcat] ← WAS
   ↓
[Spring] ← 애플리케이션
```
- 그럼 Apache는? WebServer?  
: Apache HTTP Server (WebServer), Apache Tomcat(WAS), Apache Kafka(메시징시스템)

### 톰켓 

WAS, 서블릿 컨테이너

## 설정
### pom.xml
1. WAR 패키징 설정
   - 외부 톰캣 사용 시 war로 패키징 해야한다.
   - WAR는 서블릿 컨테이너(톰캣)가 사용하는 classpath 구조(WEB_INF)를 따르기 위해 사용된다. 

<details><summary><code>💡 피드백</code> : 외부 톰캣을 사용하는 이유가 따로 있을까요?</summary>

책에서 Spring MVC를 서블릿 기반 구조로 구현하는 과정을 보여주기 위해 외부 톰캣을 사용하였고, 서블릿 컨테이너에 맞춰서 WAR로 패키징 하기 위해 pom.xml에 WAR 패키징 방식을 설정합니다.  
 
</details>
<!-- 
- 서블릿 컨테이너
- 외부 톰캣 VS 내장 톰캣
-->

### @Configuration
Configuration 클래스의 설정 목록

1. @EnableWebMvc
    - MVC를 사용한다는 어노테이션 

2. WebMVCConfiguer
    - WebMVC 설정 인터페이스
    - configureDefaultServletHandling()
    - configureViewResolvers()

3. Dispatcher Servlet
      - 모든 요청을 받아 컨트롤러로 위임하는 Front Controller
      - 매핑경로 '/'로 주었을때 JSP/HTML/CSS 등을 바르게 처리 하기 위한 설정을 추가해야한다.(=> 정적리소스 처리 방식 추가)

4. View Resolver
   - view path
   - string으로, view 파일 매칭
   - jsp(접두사, 접미사)

<details><summary><code>💡 피드백</code> : 모든 요청을 DispatcherServlet이 먼저 받으면, 그다음에는 어떤 기준으로 적절한 컨트롤러를 찾는지 궁금해요.
</summary>

DispatcherServlet이 모든 요청을 받은 다음, HandlerMapping이 요청 URL + HTTPMethod 기준으로 실행할 컨트롤러를 찾습니다.
 
</details>

<details><summary><code>💡 피드백</code> : /로 매핑했을 때와 *.do처럼 특정 패턴으로 매핑했을 때의 차이가 궁금해요.
</summary>

- / : 모든 요청을 스프링이 처리합니다. 따라서 정적 리소스도 스프링이 처리합니다.
- 특정 패턴 : 특정 패턴에 해당되는 리소스만 스프링이 처리합니다.

<정적 리소스를 스프링이 처리하게 되면>
- 정적 리소스를 처리하는 핸들러를 생성하지 않아 404 오류 발생 합니다.
- 정적 리소스(= 그대로 응답해주면 되는 파일)는 로직이 없어, 스프링을 거치게 된다면 성능과 시간 낭비가 생깁니다.
 
</details>
   
### web.xml
서블릿 컨테이너가 사용할 서블릿을 등록
URL과 매핑, 초기 파라미터 설정을 정의하는 배포서술자

1. DispatcherServlet
2. contextClass
3. contextConfigLocation
4. servlet filter



## 구현

### 컨트롤러  

1. @Controller annotation  
2. @GetMapping()
    - Http 매서드 annotiation
    - DispatcherServlet이 받은 경로 기준으로 매핑된다.
3. @RequestParam
    - 요청의 파라미터  
    - 인자 : value(파라미터 값의 이름, view에서 EL로 표기해 값을 사용할 수 있다.), require(true : 파라미터 없을때 400 Bad request, false : 파라미터 없을때 파라미터 값 null로 치환)

<details><summary><code>💡 피드백</code> : @RequestParam으로 받은 값의 검증은 어떻게 처리하는 것이 좋은지 궁금해요
</summary>

@RequestParam은 단순히 값을 바인딩하는 역할만 하기 때문에, 검증은 별도로 처리해야 한다고 생각합니다.
실무에서는 파라미터가 단순한 경우에는 @validated와 Bean Validation 어노테이션(@min, @notblank 등)을 함께 사용해 검증합니다.
다만 파라미터가 많아지거나 구조가 생기면 DTO로 분리하고 @Valid를 사용하는 방식으로 확장합니다.
검증 실패에 대한 예외는 GlobalExceptionHandler에서 일관된 형태로 처리합니다.
 
</details>
<details><summary><code>💡 피드백</code> : View Resolver가 어떤 방식으로 View를 찾는지 궁금해요
</summary>

ViewResolver는 설정된 prefix와 suffix를 기준으로 View를 찾습니다.  

예를 들어  
/WEB-INF/view/ 와 .jsp로 설정되어 있다면,
컨트롤러에서 "hello"를 반환했을 때
/WEB-INF/view/hello.jsp 파일을 찾아 실행합니다.  
```
prefix + viewName + suffix
```
prefix와 suffix설정은 @configuration에서 ViewResolver.jsp(String prefix, String suffix)에서 합니다.

</details>

4. Modle
    - View로 서비스 될 내용을 담는 객체
5. View
    - JSP기준
    보여주는 페이지

디스패처 서블릿 : 모든 요청을 받아 컨트롤러로 위힘하는 Front Conttoller


## 빌드 도구
웹 애플리케이션을 어디서든 실행 시키기 위해 패키징해주는 도구
- 클래스파일 생성
- 의존 관리
- jar/war로 패키징
  
### 메이븐, 그레이들
- 개발 시 컴파일된 클래스와 의존성 클래스는 target/build 디렉토리에 생성 된다.
- 패키징 시 서블릿 스펙에 맞게 WEB-INF/classes, WEB-INF/lib구조로 포함된다.

<details><summary><code>💡 피드백</code> : WEB-INF/lib 구조를 사용하는 이유가 있을까요?
</summary>

서블릿컨테이너의 classpath가 WEB-INF/lib, WEB-INF/classes임으로 해당 디렉토리에서 클래스를 로드합니다. 
따라서 해당 위치에 클래스 파일과 라이브러리를 배치해야 애플리케이션이 실행됩니다.
 
</details>

- 예외 발생 시
- ClassNotFoundException, NoSuchMethodError 발생 시 classpath 문제
- dependency 존재 여부 확인 -> scope 확인 -> 의존성 버전 충돌 여부 확인

<details><summary><code>💡 피드백</code> : ClassNotFoundException, NoSuchMethodError 예시
</summary>

- ClassNotFounException : .class 파일이 classpath에 없어서 클래스 로딩 하지 못할때 발생하는 예외  
- NoSuchMethodException : .class는 있지만 라이브러리 버전이 달라서 메서드를 찾지 못할 때 발생하는 애러
 
</details>

- 서블릿 컨텍스트 : 서블릿 컨텍스트는 하나의 웹 애플리케이션 단위로 생성되는 객체로, 해당 애플리케이션 내의 서블릿들이 공통으로 사용하는 설정 정보와 자원을 공유하기 위한 실행 환경입니다.
<details><summary><code>💡 피드백</code> : 서블릿 컨텍스트 경로가 왜 필요한지 궁금해요
</summary>

서블릿 컨텍스트 경로는 하나의 서버에서 여러 웹 애플리케이션을 구분하기 위해 필요합니다.
클라이언트 요청이 들어오면, 서버는 URL의 컨텍스트 경로를 기준으로 어떤 애플리케이션에 요청을 전달할지 결정합니다.
예를 들어 /shop, /blog 같은 경로로 서로 다른 애플리케이션을 구분할 수 있습니다.
이 과정이 있어야 요청이 올바른 애플리케이션과 서블릿으로 전달됩니다.

</details>
<details><summary><code>💡 피드백</code> : 서블릿 컨텍스트를 사용하는 대표적인 실제 예시가 궁금해요.
</summary>

ServletContext는 애플리케이션 전체에서 공통으로 사용하는 데이터를 공유할때 주로 사용됩니다.
파일업로드, 설정값을 context-param으로 동록해 모든 서블릿이 참조하거나, 접속자 수와 같은 전역 데이터를 저장해 여러 컴포넌트에서 공유하는데 활용됩니다.
 
</details>

