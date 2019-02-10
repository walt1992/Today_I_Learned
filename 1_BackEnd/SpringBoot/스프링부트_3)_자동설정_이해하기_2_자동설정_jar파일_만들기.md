# 스프링부트 자동설정 이해하기 2) 자동설정 jar파일 만들기 
>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._


이번 포스트는 Autoconfoguration을 직접 만들어 실제 스프링부트 프로젝트에서 어떻게 활용될 수 있는지 연습해 본다. 

먼저 Autoconfiguration을 제공할 새로운 스프링부트 프로젝트를 생성한다. 
나의 경우 minsik-springboot-stater라는 이름으로 프로젝트를 생성하였다. 

프로젝트가 생성되었다면 (maven의 경우) pom.xml 파일에 다음과 같은 디펜던시 설정을 해준다.


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>


이후, 자동설정에 활용할 클래스를 만들어 Bean객체로 등록해 준다. 

예를들어 다음과 같은 클래스를 만들 수 있다. 


###

    public class Holoman {

        String name;
        int howLong;

        public String getName() {
            return name;
        }

        public int getHowLong() {
            return howLong;
        }

        public void setName(String name) {
            this.name = name;
        }

        public void setHowLong(int howLong) {
            this.howLong = howLong;
        }

        @Override
        public String toString() {
            return "Holoman{" +
                    "name='" + name + '\'' +
                    ", howLong=" + howLong +
                    '}';
        }
    }

이후 위와 같은 클래스를 빈객체로 등록해 줄 수 있는 다른 Configuration 클래스를 만든다. (원래 이러한 자동설정파일은 새로운 프로젝트로 만드는게 일반적이라고 한다.)



    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class HolomanConfiguration {

        @Bean
        public Holoman holoman(){
            Holoman holoman = new Holoman();
            holoman.setHowLong(5);
            holoman.setName("Minsik");
            return holoman;
        }

이 Configuration클래스는 이전에 만든 Holoman이라는 클래스 객체를 자동 생성해 초기화를 시켜주는 클래스이다. 스프링 혹은 스프링부트 프로젝트에서 `@Configuration` 설정을 해놓을 경우 자동으로 의존성 주입이 발생하는 것 처럼 이곳에서도 해당 설정이 반드시 필요하다. 

이후, META-INF폴더 밑에 `spring.fatories`라는 설정파일을 생성해 다음과 같은 설정을 한다. 

    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
        web.ecoveloper.HolomanConfiguration


이는 `@EndableAutoCofiguration` 어노테이션이 자동설정을 실행할 때 이곳에 명시된 경로를 스캔하라는 것을 지시한다. (`@EndableAutoCofiguration`에 대한 내용은 [이곳](스프링부트_자동설정_이해하기_1_@SpringBootApplication.md) 참고)

이후, 생성된 프로젝트를 빌드해 jar파일로 변환해주어야 한다. 
        
콘솔에서

    mvn install

을 입력하게 되면 다른 메이븐 프로젝트에서 생성된 jar파일을 가져다가 사용될 수 있도록 로컬 메이븐 저장소에 설치하게 된다. 

이제 해당 jar파일을 사용할 프로젝트의 pom.xml파일에 

    <dependency>
        <groupId>web.ecoveloper</groupId>
        <artifactId>minsik-spring-boot-starter</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency> 

과 같은 설정을 해주면 방금 생성한 jar파일이 자동설정시 의존성주입이 발생한다. 이를 확인해 보기 위해 다음과 같은 클래스 파일을 생성한다 .

    @ComponentScan
    public class HolomanRunner implements ApplicationRunner {

        @Autowired
        Holoman holoman;

        @Override
        public void run(ApplicationArguments args) {
            System.out.println(holoman);
        }
    }


ApplicationRunner는 스프링부트 어플리케이션이 시작될 때 자동으로 실행시키고 싶은 bean을 만들고 싶을때 사용되는 인터페이스라고 한다.

이후 프로젝트를 실행시키고 나면, 

    Holoman{name='Minsik', howLong=5}

과 같이 콘솔에 찍히는 것을 확인할 수 있다. 그렇다면 만약, HolomanRunner 클래스아래에 다음과 같이 새로운 Holoman빈객체를 등록하는 코드를 작성하면 어떻게 될까?

    @Bean
    public Holoman Holoman(){
        Holoman holoman = new Holoman();
        holoman.setName("Ecoveloper");
        holoman.setHowLong(70);

        return holoman;
    }

이 경우에도 어플리케이션을 실행하면 이전과 동일하게 

    Holoman{name='Minsik', howLong=5}

라는 결과가 출력되는 것을 알 수 있다. 이는 Main함수가 정의된 Class에 설정되어 있는 `@SpringBootApplication`이라는 어노테이션이 의존성 주입을 시작할 때, 빈객체를 생성하는 순서와 관련되어 있는 문제이며 해당 내용은 [이곳](스프링부트_자동설정_이해하기_1_@SpringBootApplication.md)을 참고하면 된다.

위의 내용을 요약하자면, `@SpringBootApplication`은 `@ComponentScan`과 `@EndableAutoConfiguration` 두가지 어노테이션을 활용하는데, `@ComponentScan`을 통해 먼저 빈 객체를 생성하고 이후 `@EndableAutoConfiguration`를 통해 autoconfigure에 등록된 객체들을 빈으로 등록한다. 

위의 경우, 이러한 특성때문에 우리가 프로젝트에서 생성한 빈객체를 `@EndableAutoConfiguration`이 빈객체를 생성하는 과정에서 덮어쓰게 된 것이라 할 수 있다. 
***

그렇다면 `@ComponentScan`을 통해 생성된 빈객체가 항상 우선시되도록 하기 위해서는 어떻게 해야 할까?

Autoconfiguration을 통해 등록되는 빈 객체에 다음와 같은 어노테이션을 추가하면 된다.

    @Bean
    @ConditionalOnMissingBean
    public Holoman holoman(){
        Holoman holoman = new Holoman();
        holoman.setHowLong(5);
        holoman.setName("Minsik");
        return holoman;
    }

  `@ConditionalOnMissingBean`은 `@ComponentScan`에 의해 생성된 타입의 객체가 없을 경우 해당 설정을 빈객체로 등록하겠다는 어노테이션이다. 

  이후 다시 `mvn install`을 실행한 후, 기존 프로젝트로 돌아와 실행하면 다음과 같은 결과가 나오는 것을 확인할 수 있다.

     Holoman{name='Ecoveloper', howLong=70}

