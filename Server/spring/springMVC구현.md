# MVC
## 설정
### pom.xml
1. war 패키징 설정
   - 서블릿/JSP 사용한 웹어플리케이션은 war로 패키징 해야한다.(외부 톰캣 사용 시)

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
