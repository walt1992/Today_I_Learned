# 스프링부트 JPA 설정 & 테스트

>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._

JPA에 대해 간략히 정리된 내용은 [JPA/Entity_생성하기](Entity_생성하기.md)를 참고하면 된다.

***

### JPA사용을 위한 설정
JPA를 사용하기 위해서는 다음 디펜던시 설정을 추가해 주어야 한다.

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

또한, 서비스 DB로 사용될 postgresql과 테스트 환경에서 사용될 인메모리 DB인 h2 디펜던시도 추가한다.

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
         <!--Postgres-->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>


현재 DB설정 정보는 다음과 같다.

### Application.properties

    spring.datasource.url= jdbc:postgresql://localhost:5432/springboot
    spring.datasource.username=minsik
    spring.datasource.password=pass
    spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
***

테스트를 진행하기에 앞서 먼저 다음과 같은 Entity 클래스를 생성한다.

### Account.class

    @Entity
    @Data
    public class Account {

        @Id
        @GeneratedValue(strategy= GenerationType.IDENTITY)
        private Long id;

        private String username;
        private String password;
    }   

(`@Data` 어노테이션과 관련된 내용은 [Spring/Lombok_어노테이션_활용하기](../Spring/Lombok_어노테이션_활용하기.md)참고)


### 스키마 초기화하기 
물론 테이블을 직접 DDL명령어를 활용해 생성해 주어도 되나 자동으로 Entity와 매핑될 스키마를 자동 생성해 줄 수 있다.

매핑될 테이블을 **자동으로 생성**되도록 하고 싶다면, application.properties에 다음과 같은 옵션을 추가하면 된다.

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.generate-ddl=true
    spring.jpa.show-sql=true // 로그에 sql를 찍어주는 옵션

`spring.jpa.hibernate.ddl-auto`에는 update 이외에도 여러 설정이 가능하다.

+ **create** : 현재 DB에 있는 스키마를 모두 삭제하고 새롭게 스키마를 생성한다.
+ **validate** : 현재 DB의 스키마가 Entity클래스와 정상적으로 매핑이 되는지 검증하는 옵션이다. 만약, 일치하지 않는다면 에러가 발생한다. 실제 운영시에는 해당 옵션을 설정해 주어야 한다.
+ **update** : Entity에 변경사항이 생겼을 경우 DB스키마에 변경 내용을 적용한다. 단, Entity의 변수명이 변경되는 경우는 인식하지 못하고 추가 혹은 삭제 되었을 경우에만 갱신된다.





# 테스트 코드 작성하기

테스트 코드를 작성하기 위해서는 두가지 방법이 있다. 어플리케이션 전체를 구동하는 방법과 JPA를 위한 클래스 Bean객체만 생성해 가볍게 테스트 하는 방법.

전자의 경우는 JPA테스트에 한정되어 있지 않은 너무 보편적인 방법이라 굳이 언급하지 않아도 될 것 같다.

여기서 정리할 것은 `@DataJpaTest`어노테이션을 활용한 테스트이다. 이 어노테이션은 어플리케이션 전체를 실행하는 것이 아닌, JPA를 활용한 작업을 위한 객체(@Repository와 @Entity)만 Bean으로 등록한다는 점에서 매우 가벼운 테스트가 가능하다.

이 경우 DB는 인메모리 데이터베이스인 h2를 사용하게 된다.

### `@DataJpaTest`를 활용한 테스트 코드

    @RunWith(SpringRunner.class)
    @DataJpaTest 
    public class AccountRepositoryTest {

        @Autowired
        DataSource dataSource;
        @Autowired
        JdbcTemplate jdbcTemplate;
        @Autowired
        AccountRepository accountRepository;
        @Test
        public void di() throws Exception{
            try(Connection connection = dataSource.getConnection()){
                DatabaseMetaData metaData = connection.getMetaData();
                System.out.println(metaData.getURL());
                System.out.println(metaData.getUserName());
                System.out.println(metaData.getDriverName());
            }

            Account account = new Account();
            account.setUsername("minsik");
            account.setPassword("pass");

            Account newAccount = accountRepository.save(account);

            assertThat(newAccount).isNotNull();
            Account existAccount = accountRepository.findByUsername(account.getUsername()); // accountRepository에 메서드만 생성해 주면 자동 적용됨
            assertThat(existAccount).isNotNull();

            Account notExistAccount = accountRepository.findByUsername("ecoveloper");
            assertThat(notExistAccount).isNull();
        }
    }

