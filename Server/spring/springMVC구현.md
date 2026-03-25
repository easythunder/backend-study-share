# MVC
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

### Configuration
1. 서블릿 디스패처
2. View Resolver
   - view path
   - string으로, view 파일 매칭

### web.xml

## 구현
### 컨트롤러  

1. @Controller annotation
2. @RequestParam
    요청의 파라미터
    인자 : value(파라미터 값의 이름), require(treu : 파라미터 없을때 400 Bad request, false : 파라미터 없을때 파라미터 값 null로 치환)
3. Modle
    View로 서비스 될 내용을 담는 객체
4. View
    - JSP기준
    보여주는 페이지

디스패처 서블릿 :

## 빌드 도구
웹 애플리케이션을 어디서든 실행 시키기 위해 패키징해주는 도구
- 클래스파일 생성
- 의존 관리
- jar/war로 패키징
  
### 메이븐, 그레이들
- 개발 시 컴파일된 클래스와 의존성 클래스는 target/build 디렉토리에 생성 된다.
- 패키징 시 서블릿 스펙에 맞게 WEB-INF/classes, WEB-INF/lib구조로 포함된다.

- 예외 발생 시
- ClassNotFoundException, NoSuchMethodError 발생 시 classpath 문제
- dependency 존재 여부 확인 -> scope 확인 -> 의존성 버전 충돌 여부 확인
