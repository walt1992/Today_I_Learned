# 스프링부트 자동설정 이해하기 1) @SpringBootApplication

>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._

`@SpringBootApplication`은 Spring Boot의 자동설정, 즉 Bean객체를 등록하기 위해 설정한 어노테이션이다. 

그 활용은 보통 스프링부트 어플리케이션을 실행하는 Class에 등록해 놓으며 다음과 같이 활용된다. 

    package web.ecoveloper.springinit;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }

그렇다면, `@SpringBootApplication`는 어떻게 스프링빈객체를 생성할까?

***

`@SpringBootApplication`의 내부를 보면 다음과 같은 설정이 되어있다.    
    
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(
        excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
    )
    public @interface SpringBootApplication {

여기서 눈여겨 볼 어노테이션은 `@ComponentScan`과  `@EnableAutoConfiguration`이다. 

이 두가지 어노테이션 역시 빈객체의 생성을 담당하는 어노테이션이며, 위의 순서대로 빈객체를 생성한다. 

 ### @ComponentScan

 `@ComponentScan`은 등록된 클래스가 속해있는 패키지를 포함한 하위 패키지를 스캔하여 다음과 같은 어노테이션이 입력되어 있는 클래스를 빈객체로 생성한다.

* @Component
* @Configuration 
* @Repository 
* @Service
* @Controller
* @RestController
 

 ### @EnableAutoConfiguration
@ComponentScan으로 인해 class들이 빈으로 등록되고 나면  `@EnableAutoConfiguration`이 본인의 역할을 수행한다. 
사실상 스프링부트가 웹 어플리케이션으로써 실행되기 위해 반드시 필요한 어노테이션이다. 

`@EnableAutoConfiguration`은 Spring-boot-autoconfigure의 메타파일을 읽어 그 안에 등록된 Configuration들을 Bean객체로 등록한다. 

![Spring-boot-autoconfigure ](/img/spring-boot-autoconfigure.gif "Spring-boot-autoconfigure")

즉, 위에 표시한 파일에 등록된 Configuration파일들을 Bean객체로 등록하는데 자동설정되는 부분은 다음과 같다.

    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
    org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
    org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
    
    --more

Auto Configure에 등록된 파일들을 열어보면 모두 `@Configuration ` 어노테이션설정이 되어 있는 것을 알 수 있다. 즉, 이 클래스들을 찾아 조건에 만족되는 클래스를 빈객체로 등록하면서 스프링부트 어플리케이션 구동을 위한 자동설정을 하게 된다. 


이처럼 `@SpringBootApplication`은 `@ComponentScan`과 `@EnableAutoConfiguration` 두가지 어노테이션을 통해 스프링부트 어플리케이션을 실행시키기 위한 빈객체 생성을 할 수 있게 된다. 