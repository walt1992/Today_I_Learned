# 테스트용 properties 설정하기

테스트를 할 때, 테스트에 알맞는 프로퍼티를 설정하고 싶을 경우가 종종 생긴다. 나는 대표적으로 테스트용 DB에 테스트하고 싶을 때 주로 사용하는 방법이다.

만약 테스트코드를 작동할 때에만 사용하는 DB를 설정하고 싶다면 테스트 코드에 다음과 같은 설정을 하면 된다. 

### 1) 테스트 코드 클래스에서 Properties 변경하기

    @RunWith(SpringRunner.class)
    @SpringBootTest(properties = "spring.datasource.url='jdbc:postgresql://localhost:5432/springboottestdb'")
    public class AccountRepositoryTest {
            // Test Code
    }    

위와같이 `@SpringBootTest` 어노테이션에 테스트용 프로퍼티 내용을 입력해주면 실제 프로퍼티 파일에 설정되어 있는 프로퍼티의 내용을 오버라이드한다.


### 2) 테스트용 properties 파일 만들기
또한, 좀 더 상세한 설정을 변경하여 테스트하고 싶다면 테스트를 위한 application.properties 파일을 생성해 등록해 주면 되는데, 그 경로는 다음과 같다.

    test
    ㄴ resources
        ㄴ application.properties

위와같이 테스트용 application.properties를 등록하게 되면 해당 파일에 설정된 properity는 어플리케이션의 메인 프로퍼티를 오버라이드하게 된다. 

단, 해당 방법을 상용하기 위해서는 테스트코드 상 다음 설정이 반드시 되어 있어야 한다.

    @RunWith(SpringRunner.class)
    @SpringBootTest

### 3) @TestPropertySource 어노테이션 활용하기

`@TestPropertySource`를 사용해 직접 테스트에 활용될 프로퍼티 파일을 지정해 줄 수 있다.

    @TestPropertySource(locations = "classpath:applications.properties")

이때 프로퍼티 파일의 위치는 main 디렉토리 혹은 test 디렉토리아래의 resources 디렉토리 안에만 존재하면 어디든 무관하게 작동한다.


