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

<details><summary><code>💡 피드백</code> : 의존이랑 결합도는 어떻게 구분하는게 좋을까요?</summary>
의존은 한 객체가 다른 객체를 사용하는 관계 자체를 말하고, 결합도는 한 클래스의 변경이 다른 클래스의 변경을 얼마나 강제하는지를 나타내는 정도입니다.

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

<details>
<summary><code>💡 피드백</code> : CGLIB 방식이 항상 적용되나요? 예외 케이스도 있나요?</summary>

@Configuration은 기본적으로 CGLIB 프록시를 사용해
@Bean 메서드 간 호출을 가로채 싱글톤을 보장합니다.  

다만 proxyBeanMethods=false 설정, Lite mode(@Component 등),
static @Bean 메서드, AOT 환경에서는
@Configuration 클래스가 CGLIB 프록시로 생성되지 않습니다.  

또한 CGLIB은 상속 기반 프록시이기 때문에
final 클래스나 final 메서드에는 적용할 수 없습니다.
</details>


## @Autowired

타입 기준으로 컨테이너에 등록된 빈을 찾아 주입합니다.  
같은 타입이 여러개 이면 예외(NoUniqueBeanDefinitionException)가 발생합니다. 따라서 @Qualifier("빈이름")를 사용해 빈의 이름을 지정해 줍니다.
다른 방법으로는 @Primary를 사용해 빈의 기본을 등록할 수 있습니다.

## @Import

다른 설정 클래스 또는 빈 정의를 현재 설정에 포함시키기 위해 사용  

* @Import는 ComponentScan보다 먼저 처리됨
* Spring Boot 자동 설정의 핵심 메커니즘


### 다중 import 

``` 
@Import({AppConf1.class, AppConf2.class})
```
### 전이적 import  

A를 등록하면 B와 C 모두 컨테이너에 등록됨  

```
@Import(B.class)
class A { }

@Import(C.class)
class B { }
```

### import 대상



<details> <summary> @Configuration 클래스</summary>  

    ```
    @Configuration
    public class ConfigA {

        @Bean
        public String beanA() {
            return "A";
        }
    }

    @Configuration
    @Import(ConfigA.class)
    public class MainConfig {
    }
    ```

    📌 결과  
    beanA가 컨테이너에 등록됨

</details>

<details> <summary>일반 클래스 (빈으로 등록됨)</summary>  

    ```
    public class SimpleService {

        public void hello() {
            System.out.println("hello");
        }
    }

    @Configuration
    @Import(SimpleService.class)
    public class AppConfig {
    }
    ```
    📌 결과  
    SimpleService가 @Component 없이도 빈으로 등록

</details>


<details> <summary>ImportSelector</summary>

    ```
    public class MyImportSelector implements ImportSelector {

        @Override
        public String[] selectImports(AnnotationMetadata metadata) {
            return new String[] {
                "com.example.ConfigA",
                "com.example.ConfigB"
            };
        }
    }

    @Configuration
    @Import(MyImportSelector.class)
    public class AppConfig {
    }
    ```
    📌 결과  
    selectImports()에서 반환한 클래스들이 빈으로 등록  
    동적 설정 구성 가능

</details>


<details> <summary>DeferredImportSelector</summary>

    ```
    public class MyDeferredImportSelector
            implements DeferredImportSelector {

        @Override
        public String[] selectImports(AnnotationMetadata metadata) {
            return new String[] {
                "com.example.LateConfig"
            };
        }
    }

    @Configuration
    @Import(MyDeferredImportSelector.class)
    public class AppConfig {
    }
    ```
    📌 특징  
    모든 사용자 설정 처리 이후에 실행  
    Spring Boot 자동 설정에서 사용됨

</details>


<details> <summary>ImportBeanDefinitionRegistrar</summary>  

    ```
    public class MyRegistrar
            implements ImportBeanDefinitionRegistrar {

        @Override
        public void registerBeanDefinitions(
                AnnotationMetadata metadata,
                BeanDefinitionRegistry registry) {

            RootBeanDefinition beanDefinition =
                    new RootBeanDefinition(CustomService.class);

            registry.registerBeanDefinition(
                    "customService", beanDefinition);
        }
    }

    @Configuration
    @Import(MyRegistrar.class)
    public class AppConfig {
    }
    ```
    📌 결과  
    코드로 빈 이름 + 빈 정의를 직접 등록  
    가장 저수준, 가장 강력한 방식
</details>




## getBean()

- getBean()이란 빈객체를 구할때 사용하는 메서드  

* getBean()은 BeanFactory인터페이스에 정의 되어 있음  
* AbstractApplicationContext에 getBean()구현 되어 있음
* 모든 객체를 빈으로 만들 필요는 없음
* 의존 주입 대상은 컨테이너를 통한 라이프사이클 및 제어하는것이 좋음으로 빈으로 관리 하는게 좋음

### 발생할 수 있는 예외  

- getBean("빈이름", 빈.class)
  - 빈 이름이 없을 때:
    NoSuchBeanDefinitionException
  - 빈 이름은 존재하나 타입이 다를 때:
    BeanNotOfRequiredTypeException

- getBean(빈.class)
  - 해당 타입의 빈이 없을 때:
    NoSuchBeanDefinitionException
  - 같은 타입의 빈이 여러 개이고
    @Primary / @Qualifier로 결정 불가할 때:
    NoUniqueBeanDefinitionException

### 빈이름 결정
빈 이름은 다음 중 하나로 결정됨
  - @Bean 메서드명
  - @Bean("이름")
  - @Component 계열 클래스명(camelCase)
  - @Component("이름")
  - XML id
  - alias

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

<details><summary><code>💡 피드백</code> : DI와 IoC 관계를 한 문장으로 정리하면 어떻게 표현할수있나요?</summary>

IoC는 객체의 생성과 주입, 제어를 개발자가 아닌 컨테이너가 담당하도록 하는 설계방식 입니다.  
DI는 (IoC 환경에서) 한 객체가 필요로 하는 다른객체를 외부에서 주입받아 사용하는 방식입니다.
</details>

-CGLIB : Code Generation Library. 런타임에 바이트코드를 생성하여 대상 클래스를 상속한 프록시 클래스를 만들어내는 라이브러리입니다. 인터페이스 없이도 프록시 생성이 가능합니다.

- CGLIB 프록시 클래스 : 대상 클래스를 상속받아 생성된 프록시 클래스로 메서드 호출 전/후에 부가 로직을 삽입할 수 있습니다.

- JDK 동적 프록시 : 인터페이스를 기반으로 프록시 객체를 생성합니다.  

- context : 현재 실행에 필요한 모든 정보와 환경을 묶어놓은 범위
