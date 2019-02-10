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

또한, Postgres DB에 Account 클래스와 매핑될 테이블을 생성해 준다.



### 테스트 코드

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
            Account existAccount = accountRepository.findByUsername(account.getUsername());
            assertThat(existAccount).isNotNull();

            Account notExistAccount = accountRepository.findByUsername("ecoveloper");
            assertThat(notExistAccount).isNull();
        }
    }

ㅇ