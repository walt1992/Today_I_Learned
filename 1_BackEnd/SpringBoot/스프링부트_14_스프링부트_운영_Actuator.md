# Springboot Actuator


>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._
### Actuator란?

스프링케이션 애플리케이션 운영중에 주시할 수 있는 여러 정보들을 EndPoint라는 개념을 활용하여 제공해주는 모듈이다.
어플리케이션이 사용하는 자바의 버전부터 사용중인 CPU와 RAM의 정보를 실시간 차트로 확인하는 것은 물론이며 쓰레드, Bean객체의 정보, 등록된 Class 수 등 매우 다양한 정보를 확인할 수 있다.

([레퍼런스 참고](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints))

 
### Actuator 사용을 위한 디펜던시 설정


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>


### JMX를 통해 조회하기

콘솔에 `jconsole`이라는 명령어를 입력하면 다음과 같은 프로그램이 실행되며 리스트에서 조회하고자 하는 어플리케이션을 선택하면 정보를 확인할 수 있다.


![jconosole](/img/jconsole.png)
![jconosole](/img/jconsole-main.png)


이 중 Mbean탭에 들어가 org.springframework.boot폴더를 선택하면 EndPoint를 확인할 수 있다.

![jconosole](/img/jconsole-mbean.png)


jconsole과 비슷한 GUI로 확인 하는 또 다른 방법으로는 `JviaulVm`을 활용하는 방법이다.

이 역시 콘솔에 `jvisualvm`을 입력하면 자동 실행되는데 Java 11 Version 이후부터는 따로 설치를 해주어야 한다고 한다.

프로그램을 실행시키면 다음과 같은 화면에서 정보를 확인할 수 있다. 전반적으로 jconsole보다 깔끔한 디자인을 제공한다.

![jvisualVM](/img/jvisualVM.png)

`JviualVM`에서 Mbean을 조회하기 위해서는 별도의 플러그인을 설치해 주어야 하며 상단 탭의 Tools > Plugins에서 MBean을 검색 및 설치 후 프로그램을 재실행 하면 Mbean을 조회할수 있다.

![jvisualVM](/img/jvisualVM-Mbean.png)



### HTTP에서 확인하기

JMX에서 Endpoint를 조회하기 위해서는 디펜던시설정 이외의 별다른 설정을 필요하지 않다. 그러나 해당 정보들을 Http에서 링크를 통해 조회하기 위해서는 다음과 같은 설정이 필요하다.

    management.endpoints.web.exposure.include=*
    management.endpoints.web.exposure.exclude=env,beans

이는 기본적으로 Endpoint에는 어플리케이션과 관련되어 예민한 정보들을 많이 포함하고 있기 때문에 기본적으로 비공개상태로 유지하기 때문이다.
이제 위와같은 설정을 하고 난 후 URL에 다음과 같이 입력하면 EndPoint와 관련된 Hateos 정보들을 확인할수 있다.

    http://localhost:8080/acutator


   ![actuator-http](/img/actuator-http.png)


이제 원하는 Endpoint의 url을 선택해 접속하여 원하는 정보를 확인할 수 있다. 다음은 httptrace(최근 발생했던 http요청의 정보를 확인하는 Endpoint)를 조회하여 얻은 json데이터를 보기 좋게 파싱한 예시이다.

![httptrace-example](/img/httptrace-example.png)


***

이렇게 Actuator를 활용하면 어플리케이션과 관련되어 다양한 세부정보를 조회할 수 있다. 그러나 위에 제시된 방법으로 조회하기에는 다소 불편한점이 많은 듯 하다.

이러한 단점을 보완하여 더 좋은 환경에서 Endpoint를 조회할 수 있는 방법이 존재한다. 해당 내용은 [스프링부트 운영 관리자](스프링부트_15_스프링부트_운영_관리자.md) 포스팅을 참고하면 된다.
