# 스프링 DI

## DI

Dependency Injection, 의존 주입이라고 번역합니다.  
의존 객체를 직접 생성하지 않고 외부에서 전달받는 방식입니다. 

### 의존

한 객체의 변경이 다른 객체에 영향을 주는 관계입니다.
예를 들어, 한 클래스가 다른클래스의 메서드를 실행 할때 의존이라고 표현할 수 있습니다.

- 의존하는 대상을 구하는 방법  
1. 의존 대상 객체를 직접 생성  
2. DI와 로케이터


<details><summary><code>💡 피드백</code> : 의존 대상 객체를 직접 생성하는 방식의 장점과 단점이 궁금합니다.</summary>

장점은 외부 설정 없이 new로 객체를 생성하기 때문에 , 객체 생성 위치와 흐름을 코드에서 바로 확인할 수 있어 직관적이라는 점입니다.  
단점은 의존 객체의 구현체 변경 시, 객체를 생성 한 코드를 모두 수정해야 합니다. 따라서 코드의 결합도가 높습니다.  
또한 객체를 생성하는 코드가 여러 곳에 분산되면 중복이 늘어나고, 구현 클래스가 많아질 수록 관리와 확장이 어려워집니다.  
 
</details>
 
## DI를 통한 의존 처리 방법

1. 생성자 주입 방식
2. setter 메서드 방식

### 생성자 방식

빈 객체를 생성하는 시점에 완전한 상태의 객체를 사용할 수 있다.
- 파라미터 개수가 많을 경우 어떤 의존 객체를 설정하는지 알 수 없다.

### setter 메서드 방식

메서드 이름만으로 어떤 의존 객체를 설정하는지 쉽게 유추 가능하다.  
하지만 해당 객체를 사용할 시점에 의존 객체를 전달하지 않도 빈객체가 생성 되기 때문에 NullPointException이 발생할 수 있다.

- 메서드이름이 set으로 시작한다.
- 파라미터가 1개이다.
- 리턴타입이 void이다.  


<details><summary><code>💡 피드백</code> : setter 주입을 진행할때, 객체의 입장에서 NullPointerException 위험이 생기는 이유가 궁금합니다</summary>

setter 주입은 객체가 먼저 생성되고, 이후에 의존 객체가 주입됩니다.  
이 과정에서 setter 가 호출되지 않으면, 객체는 의존성이 없는 상태로 생성될 수 있습니다.  
컴파일 시점에서는 문제가 없지만 , 실행 중 의존 객체를 사용하는 순간 NullPointerException이 발생할 수 있습니다.  
반면 생성자 주입은 객체 생성 시점에 필요한 의존성을 반드시 전달 받도록 강제 함으로, 객체가 항상 완전한 상태로 생성됩니다.  

따라서 필수 의존성은 생성자 주입을 사용하고 선택적 의존성에는 setter 주입을 사용합니다.  

</details>

### 장단점


## assembler 조립기

객체를 생성해 주고 주입해주는 역할을 하는 클래스
도메인용 조립기  

- 객체 생성  
- 의존 주입  
- 필요한 객체 제공

assembler를 도메인 마다 생성 시 문제점
1. 도메인 마다 공통 적으로 필요한 객체를 중복 생성해야 된다. 
2. DataSource도 assembler마다 달라 같은 DB에 연결할 수 있지만, 같은 연결을 사용하고 있지 않다.  
따라서 트랜잭션 매니저도 달라 트랜잭션도 달라지게 된다.  
2. DataSource가 분리 되어 같은 DB를 사용하더라도 커넥션풀과 트랜잭션 컨텍스트가 분된다.
3. 하나의 유스케이스를 단일 트랜잭션으로 묶기 어렵다.

## 스프링은 DI를 지원하는 조립기

애플리케이션 전체를 다루는 범용 조립기  

- 객체 생성  
- 의존 주입  
- 생명주기 관리  
- 싱글톤 보장  

## @Configuration  

빈을 생성하는 설정 애노테이션입니다.  
빈 생성 시 CGLIB 프록시를 사용해 @Bean 메서드 호출을 가로채고, 이미 생성된 빈이 있으면 기존 객체를 반환해 싱글톤을 보장합니다.

## @Autowired

타입 기준으로 컨테이너에 등록된 빈을 찾아 주입합니다.  
같은 타입이 여러개 이면 예외(NoUniqueBeanDefinitionException)가 발생합니다. 따라서 @Qualifier("빈이름")를 사용해 빈의 이름을 지정해 줍니다.
다른 방법으로는 @Primary를 사용해 빈의 기본을 등록할 수 있습니다.

## @Import

## 싱글톤

같은 역할을 하는 객체의 정체성과 상태를 애플리케이션 전역에서 일관되게 유지하기 위해서

- 예시 
1. MemberRegisterService, ChangePasswordService에 의해 MemberDao객체가 총 2번 호출되어, 객체가 2개 생길 것 처럼 보인다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberDao memberDao() {
        System.out.println("memberDao() 호출");
        return new MemberDao();
    }

    @Bean
    public MemberRegisterService memberRegisterService() {
        return new MemberRegisterService(memberDao());
    }

    @Bean
    public ChangePasswordService changePasswordService() {
        ChangePasswordService svc = new ChangePasswordService();
        svc.setMemberDao(memberDao());
        return svc;
    }
}
```

2. 스프링에서 내부적으로 프록시를 사용하여, 이미 생성된 빈 객체가 있으면 새로 만들지 않고 기존 객체를 반환해 싱글톤을 보장한다.


```java
public class AppConfig$$EnhancerBySpring extends AppConfig {

    private Map<String, Object> singletonBeans = new HashMap<>();

    @Override
    public MemberDao memberDao() {
        if (!singletonBeans.containsKey("memberDao")) {
            singletonBeans.put("memberDao", super.memberDao());
        }
        return (MemberDao) singletonBeans.get("memberDao");
    }
}

```

3. 따라서 MemberRegisterService, ChangePasswordService에서 호출된 MemberDao는 같은 객체이다.

## 핵심  

### 3줄 요약  

1. 영향을 주는 관계를 의존이라고 합니다. 객체를 의존 주입 하지 않으면, 구현체 수정시 관련 모든 코드를 변경하게 됩니다.(결합도 높음)  
2. 빈 생성 시 같은 객체를 반환 하기 위해 스프링은 프록시를 사용해 빈을 싱글톤으로 관리한다.
3. 도메인별 assembler 방식은 공통객체와 트랜잭션 관리에 한계가 있어, 애플리케이션 단위로 객체를 관리하는 스프링 컨테이너를 사용하는 것이 적합하다.

### 용어 정리

- 도메인 : 업무의 규칙과 개념의 묶음 단위
- 애플리케이션 : 도메인을 실행하고 흐름을 제어하는 단위  
- IoC Inversion of Control : 객체의 생성과 제어 흐름을 개발자가 아니라 컨테이너가 담당하는 설계 원칙  
- CGLIB 프록시 클래스 :  
- context : 현재 실행에 필요한 모든 정보와 환경을 묶어놓은 범위
