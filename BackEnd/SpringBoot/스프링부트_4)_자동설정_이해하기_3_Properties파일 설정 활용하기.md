# Properties파일설정 활용하기 
>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._


이전 포스트에서 자동설정 jar파일을 만들어 빈객체를 생성하는 것을 연습하였다. (이전 [포스트](스프링부트_자동설정_이해하기_2_자동설정_jar파일_만들기.md) 참고)

그러나 위의 경우, jar파일에 등록되어 있는 bean객체가 다양한 상황에 활용되기에는 다소 **자율성**이 부족하다는 단점이 존재한다. 이를 해결하고자 새로운 bean객체를 프로젝트 내부에서 생성해 주기 또한 여간 까다로운 작업이 아닐 수 없다. 

### **따라서 properties를 통해 bean에 등록될 객체의 내용을 정의하는 방법에 대해 알아본다.**

먼저, jar파일을 새롭게 install하기 위해 다음과 같은 설정을 추가해야 한다. 먼저 properties에서 활용하기 위한 설정을 위해 HolomanPorperties라는 클래스를 생성한다. 

    @ConfigurationProperties("holoman")
    public class HolomanProperties {

        private String name;
        private int howLong;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getHowLong() {
            return howLong;
        }

        public void setHowLong(int howLong) {
            this.howLong = howLong;
        }
    }

해당 클래스에는 Properties에서 활용하기 위한 변수들을 설정한다. 이후 `@ConfigurationProperties`에 해당 properties를 활용하기 위한 prefix(이 경우 holoman)를 설정한다.

이후, 해당 Properties파일을 Bean객체와 바인딩하는 단계를 거쳐야 한다. 이전 포스트에서 생성해 놓은 HolomanConfiguration에 다음과 같은 설정을 추가하자.

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    @EnableConfigurationProperties(HolomanProperties.class)
    public class HolomanConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public Holoman holoman( HolomanProperties properties){
            Holoman holoman = new Holoman();
            holoman.setHowLong(properties.getHowLong());
            holoman.setName(properties.getName());
            return holoman;
        }
    }

먼저 `@EnableConfigurationProperties`에 해당 Configuration클래스와 바인딩할 Properties클래스를 명시한다. 이후 Bean객체에 설정하고 싶은 멤버필드의 변수값에 위와 같이 properties를 통해 초기화 될 수 있도록 설정하면 새로운 jar 파일을 install할 준비가 끝났다. 

    mvn install



이제 기존 메인 프로젝트로 돌아와 maven을 refresh한 후, /resources 디렉토리 밑에 application.properties라는 파일을 생성한다. 

해당 properties파일에 위에 등록한 prefix명.[변수명]의 형태로 bean객체의 멤버필드 변수를 초기화하는 설정을 할 수 있다. 

    holoman.name : properties
    holoman.how-long : 100


이후 어플리케이션을 다시 실행시키면 

    Holoman{name='Properties', howLong=100}

과 같은 결과를 얻을 수 있다. 


*** 
이로써 스프링부트에서 어떤 원리로 Propertis에 등록한 내용이 bean객체 생성과 연관이 되는지 알 수 있었다. 실제 회사에서 수 많은 properties 설정을 해야만 했는데 이번을 계기로 그 원리를 간략하게나마 이해할 수 있게 되었다. 