# 스프링부트 Rest 클라이언트
>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._



Rest Client란 현재 어플리케이션에서 Remote에 있는 다른 Rest Service를 호출할 때 사용하는 개념인 듯 하다.
예를들어 현재 구현된 Web 서버와는 별도로 다른 작업을 담당하는 또 다른 Spring Rest API와 통신하고 싶을 떄 사용한다.

대표적으로 RestTemplate와 WebClient 두 객체를 활용할 수 있는데 두 클래스의 가장 큰 차이점은 바로 요청을 **Synchronous**하게 처리하느냐(RestTemplate) 혹은 **Asynchonous**하게 처리하느냐(WebClient)의 차이이다.

`RestTemplate`는 Blocking I/O 기반의 Synchronous API이다. 이말인 즉, 한번에 하나의 요청만을 보내고 해당 요청에 대한 응답을 받고 나서야 다음 차례의 요청을 보내는 방식으로 작동한다.
`WebClient`는 Non-Blocking I/O 기반의 Asynchronous API이다. 즉, 여러 요청을 순서대로 보내고 Response를 받는대로 다음 작업을 진행한다. 다음 예제를 보자

***

두개의 Spring 프로젝트가 있으며 A라는 어플리케이션의 서버가 구동되는 시점에서 B라는 Rest API에 특정 요청을 보낸다고 가정해보자.

###B Rest API의 Controller

    @RestController
    public class SamplController {

        @GetMapping("/hello")
        public String hello() throws InterruptedException {
            Thread.sleep(5000l);
            return "hello";
        }


        @GetMapping("/my")
        public String my() throws InterruptedException {
            Thread.sleep(3000l);
            return "my";
        }

    }


B Rest API의 SampleController에는 두개의 핸들러가 존재한다. `/hello` URL과 매핑되는 핸들러는 요청을 받은 후 5초 후에 "hello"라는 문자열을 반환하며, `/my` URl과 매핑되는 핸들러는 3초 후에 "my"라는 문자열을 반환한다고 가정해보자.

이제 A 어플리케이션에서 B Rest API의 두개의 핸들러에 대해 /hello와 /my 순서로 요청을 보낸다고 가정하면 어떤 일이 발생할까?

결론부터 말하면 `RestTemplate`를 활용할 경우 약 8초 이상의 시간이 소요될 것이며, `WebClient`를 사용할 경우 약 5초의 시간이 소요될 것이다. 이제 두 Rest Client를 활용한 Rest Service호출의 예제를 살펴보자.


## RestTemplate를 활용한 예제

    Component
    public class RestRunner implements ApplicationRunner {

        @Autowired
        RestTemplateBuilder restTemplateBuilder;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            RestTemplate restTemplate= restTemplateBuilder.build(); // Blocking I/O >> 요청을 Synchronous하게 처리함

            StopWatch stopWatch = new StopWatch();
            stopWatch.start();

            String helloResult = restTemplate.getForObject("http://localhost:8080/hello",String.class);
            System.out.println(helloResult);

            String myResult = restTemplate.getForObject("http://localhost:8080/my", String.class);
            System.out.println(myResult);


            stopWatch.stop();

            System.out.println(stopWatch.prettyPrint());
        }

스프링부트에서 RestTemplate를 사용하기 위해서는 관련 Bean객체를 주입해 주어야한다. 이때 `@Autowired`가 되어야 하는 객체는 RestTemplate 자체가 아닌, `RestTemplateBulider`객체이며 이 객체를 받아 RestTemplate객체를 생성해 활용해야 한다.

    RestTemplate restTemplate= restTemplateBuilder.build();


앞서 언급했듯 RestTemplate는 요청을 Synchronous하게 처리한다. 따라서 `http://localhost:8080/hello`에 대한 요청을 보낸 후, 그에 대한 응답이 있지 않고서는 다른 요청을 보내지 않는다. 따라서 `http://localhost:8080/my`에 요청을 보내기 위해서는 `/hello`에 설정해 놓은 5초라는 시간이 모두 지난 후 hello라는 문자열을 리턴받을 때 까지 기다려야 한다. 따라서 너무나도 당연하게 `hello` 이후 `my`가 순서대로 콘솔에 찍히게 된다. 바로 앞서 `RestTemplate`을 사용할 경우 최소 8초가 소요된다는 것이 바로 이 때문이다.

이제 WebClient에 대해 살펴보자
## WebClient 활용하기 

먼저 다음과 같은 디펜던시 설정이 필요하다.
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

`webflux`라는 디펜던시 설정을하면 WebClent를 활용할 수 있다.


    @Component
    public class RestRunner implements ApplicationRunner {

        @Autowired
        WebClient.Builder builder;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            WebClient webClient = builder.build(); // None Blocking


            StopWatch stopWatch = new StopWatch();
            stopWatch.start();

            Mono<String> helloMono = webClient.get().uri(""http://localhost:8080/hello")
                                                    .retrieve()
                                                    .bodyToMono(String.class);

            helloMono.subscribe( s-> {
                System.out.println(s);
                if(stopWatch.isRunning()){
                    stopWatch.stop();
                }
                System.out.println(stopWatch.prettyPrint());
                stopWatch.start();
            });

            Mono<String> myMono = webClient.get().uri("http://localhost:8080/my")
                                            .retrieve()
                                            .bodyToMono(String.class);

            myMono.subscribe(s -> {
                System.out.println(s);
                if(stopWatch.isRunning()){
                    stopWatch.stop();
                }
                System.out.println(stopWatch.prettyPrint());
                stopWatch.start();
            });

`WebClient`역시 `RestTemplate`와 마찬가지로 `Builder`를 Bean으로 등록한 후 해당 빌더를 활용해 객체를 생성하는 방식으로 사용할 수 있다.

    @Autowired
    WebClient.Builder builder;
    
위와 같이 `bulder`를 `@Autowired`하여 객체를 주입받으면 다음과 같이 `WebClient`객체를 생성해 활용할 수 있다.

    WebClient webClient = builder.build();

    Mono<String> helloMono = webClient.get().uri(""http://localhost:8080/hello") // 해당 Uri로 Get요청을 보낸 후
                                    .retrieve() // 받아와서
                                    .bodyToMono(String.class); // Body를 mono타입으로 변환한다.

한가지 유의할 점은 위의 코드를 작성한다고 하여 요청을 바로 보내는 것은 아니라는 점이다. 위처럼 Mono객체를 생성한 후 Mono의 `subscribe()`메소드를 실행해야 비로서 요청을 보내게 된다. 아래가 바로 해당 코드이다.

    helloMono.subscribe( s-> {
        System.out.println(s);
        if(stopWatch.isRunning()){
            stopWatch.stop();
        }
        System.out.println(stopWatch.prettyPrint());
        stopWatch.start();
    });

>즉, 요청에 대한 정보를 담은 Mono 객체를 생성하면 하나의 객체로 여러번 요청을 보낼 수 있다.
>추가적으로 mono.block()를 사용할 경우 요청을 보내고 그에 따른 응답을 바로 객체로 받을 수 있다. Mono에 대한 공부는 나중에 따로 하는걸로    

자 이제 다시 어플리케이션을 실행하면 어떤 결과가 발생할까? 먼저 총 소요시간은 5초보다 조금 긴 시간이 걸릴 것이다. 그리고 print된 내용을 보면 `/hello`와 `/my` 순으로 요청을 보냈음에도 불구하고

    my
    hello

순으로 응답을 받은 것을 확인할 수 있다.

***

이렇게 간략히 RestTemplate와 WebClient를 활용해 Rest API를 호출하는 방법을 공부하였다. 

기존 강의에는 하나의 프로젝트내에서 자기자신에게 요청을 보내는 방식으로 진행하였지만, 혼자 연습하는 과정에서 두개의 프로젝트를 동시에 띄워둔 후 Rest Client를 활용해 요청을 받고 응답을 보내는 과정을 테스트해 보았고 그 과정이 매우 재미있었다. 

또, 실제 서비스되고있는 사이트 (예를 들어 네이버)에도 해당 기능을 활용해 HTML을 응답받아 올 수 있다는 것을 테스트를 통해 알게 되었으며, 이 기능을 활용하여 웹 크롤링과 같은 기능을 구현해 볼 수 있지 않을까 생각이 들었는데... 실제 자바를 활용해 웹크롤링을 어떻게 하는지 모르겠지만, 나중에 한번 시도해보면 재미있을 것 같다는 생각이 들었다. 

일단 다음에는 인강을 통해 앞서 살펴본 두개의 Rest Client를 어떻게 커스터마이징하는지 정리할 예정이다. 


