# 스프링부트 HttpMessageConverters

>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._

Http 요청 본문을 객체로 변경하거나 객체를 Http 응답 본문으로 변경할 때 사용한다.
Controller에 @RequestBody 어노테이션을 사용하면 해당 설정을 할 수 있다.

        @PostMapping("/user/create") // RestController를 사용하면 RequestBody를 생략해도 된다.
        // 만약 RestController가 명시되어있지 않다면 String 리턴에 대해 그에 알맞은 URL을 View Resolver에서 찾음
        public User create( @RequestBody User user){

            return user;
        }

원칙적으로는 Return값을 명시하는 부분 또한 `@RequestBody` 어노테이션을 명시해주어야 하지만, 컨트롤러를 빈객체로 등록하기 위한 설정에서 `@RestController`어노테이션을 사용하였다면 이는 생략해도 무방하다. 

이로써 Http 요청의 본문 혹은 응답 본문을 원하는 형태로 변경해 받아오는 설정을 끝냈다.
이를 확인할 수 있는 테스트 코드는 다음과 같이 작성할 수 있다. 


### Json 형태

    @Test
    public void createUser_JSON() throws Exception{
        String userJson = "{\"username\":\"minsik\", \"password\" :\"123\"}";
        mockMvc.perform(post("/user/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8) // Http 요청으로부터 받는 컨텐츠의 타입은 무엇인가?
                .accept(MediaType.APPLICATION_JSON_UTF8) // 응답으로 어떤 타입을 원하는가
                .content(userJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.username", is(equalTo("minsik"))))
                .andExpect(jsonPath("$.password", is(equalTo("123"))));

    }

### XML형태

XML형태로 Http 요청을 처리하기 위해서는 다음과 같은 의존성 주입이 우선되어야 한다.

    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
        <version>2.9.6</version>
    </dependency>

    

    @Test
    public void createUser_XML() throws Exception{
        String userJson = "{\"username\":\"minsik\", \"password\" :\"123\"}";
        mockMvc.perform(post("/user/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8) // Http 요청으로부터 받는 컨텐츠의 타입은 무엇인가?
                .accept(MediaType.APPLICATION_XML) // 응답으로 어떤 타입을 원하는가
                .content(userJson))
                .andExpect(status().isOk())
                .andExpect(xpath("/User/username").string("minsik"))
                .andExpect(xpath("/User/password").string("123"));

    }



