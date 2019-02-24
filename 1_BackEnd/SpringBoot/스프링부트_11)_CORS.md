# 스프링부트 CORS

>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._

CORS란 `Cross Origin Resource Sharing`의 약자로서 `Single-Origin Policy`를 보완하기 위해 생겨난 기술이다.

`Single-Origin Policy`는 같은 Origin에서만 리소스를 Share할 수 있다는 의미이며, CORS는 서로 다른 Origin끼리 호출할 수 있는 것을 의미한다.

Origin이란 

* URL스키마 (http, https)
* hostname (localhost)
* 포트 (8090, 8090 )

의 조합한 것을 의미한다.

이런 하나의 Origin은 또 다른 API의 Origin을 호출할 수 없다. 스프링 부트는 이러한 한계를 보완할 수 있는 CORS기능을 별다른 설정없이 가능하도록 지원한다.

***

예를 들어 다음과 같은 간단한 API 서버가 있다고 가정하자.

### API Server ( port: 8090)
    
    @SpringBootApplication
    @RestController
    public class Application {

        public static void main(String[] args) {
            SpringApplication.run(Application.class);
        }

        @GetMapping("/hello")
        public String hello(){
            return "hello";
        }
       

그리고 위의 API 서버로 요청을 보내는 새로운 클라이언트 프로젝트를 생성해 다음과 같이 http 통신을 보내본다.

### Client ( port: 18090)

    <body>
    Hello this is client
    </body>
    <script src="//code.jquery.com/jquery-3.2.1.min.js"></script>

    <script>
        $(function(){
            $.ajax("http://localhost:8090/hello")
                .done( (msg) =>{
                    alert(msg);
                })
                .fail( () =>{
                    alert("fail");
                })
        })
    </script>


두개의 프로젝트를 모두 실행시킨 후, Client에서 API 서버로 보낸 http Request는 당연 `Fail`이 된다.
이는 앞서 언급했듯 하나의 Origin에서 다른 Origin을 호출하는 것은 원칙적으로 막혀있기 때문이다. 이를 해결하는 방법이 CORS이며 그 설정은 간단하다. 

### CORS 설정하기 1)- @CrossOrigin

`@CrossOrigin` 어노테이션을 활용해 원하는 Mapping Url 혹은 Controller에 접근을 허용하는 Origin을 설정할 수 있다.


    @CrossOrigin(origins = "http://localhost:18090")
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
    
위와같이 특정 메소드에 `@CrossOrigin`어노테이션을 설정할 경우 해당 Mapping 주소로 오는 요청에 한해서만 설정한 Origin에 접근을 허용한다.

만약 전체 컨트롤러에 해당 설정을 해주고 싶다면 단순하게 어노테이션의 위치를 컨트롤러 전역에 설정해주면 된다.

### CORS 설정하기 2)- 전역에 설정하기

만약 특정 메소드 혹은 컨트롤러를 넘어 전역적으로 모든 요청에 대하여 접근을 허용해 주고 싶다면 다음과 같이 설정하면 된다. 

* Web 설정 클래스 생성해 설정하기

    @Configuration
    public class WebConfig implements WebMvcConfigurer {

        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
                    .allowedOrigins("http://localhost:18090");
        }
    }

`WebMvcConfigurer`는 스프링 MVC에서 제공하는 기능을 그대로 가져가면서 추가로 설정할 수 있는 인터페이스이다.

이제 위와 같이 설정을 하게 되면 모든 컨트롤러에 일일이 설정할 필요 없이 글로벌하게 CORS설정을 할 수 있다. 