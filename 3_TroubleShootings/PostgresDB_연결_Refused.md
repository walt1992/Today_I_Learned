# Postgreg 연결 거부 현상



### Docker로 생성한 Postgres 컨테이너 실행하기

Docker에 대해서 잘 모르지만 현재 듣고있는 인강에서 도커를 사용해 Postgres 컨테이너를 생성하였으며 그 정보는 다음과 같다. 

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    bde691234f90        postgres            "docker-entrypoint.s…"   13 seconds ago      Up 12 seconds       0.0.0.0:5432->5432/tcp   postgres_boot
        

이후 다음 명령어를 입력해 Postgres 컨테이너를 실행하였다.

    $ docker exec -i -t postgres_boot bash

다음 명령어를 입력해 postgres_boot 프로세스가 정상 실행되었는지 확인한다.

    root@bde691234f90:/# ps ax | grep postgres
        1 ?        Ss     0:00 postgres
    59 ?        Ss     0:00 postgres: checkpointer
    60 ?        Ss     0:00 postgres: background writer
    61 ?        Ss     0:00 postgres: walwriter
    62 ?        Ss     0:00 postgres: autovacuum launcher
    63 ?        Ss     0:00 postgres: stats collector
    64 ?        Ss     0:00 postgres: logical replication launcher
    75 pts/0    S+     0:00 grep postgres


이제 미리 등록한 데이터 베이스에 접근하는 명령어를 실행하였다.

    root@bde691234f90:/# psql -U minsik springboot
    psql (11.1 (Debian 11.1-3.pgdg90+1))
    Type "help" for help.

    springboot=#

이렇게 정상적으로 springboot라는 데이터베이스에 접속하는 것까지 확인하였고, 이제 Springboot 프로젝트에서 해당 DB에 연결해 테이블을 생성해보는 예제를 진행중이었다.


### DB에 연결하기 위한 클래스 소스코드

    @Component
    public class PostgresqlRunner implements ApplicationRunner {

        @Autowired
        DataSource dataSource;

        @Autowired
        JdbcTemplate jdbcTemplate;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            System.out.println("RUN");
            try(Connection connection = dataSource.getConnection()) {
                System.out.println(dataSource.getClass());
                System.out.println(connection.getMetaData().getURL());
                System.out.println(connection.getMetaData().getUserName());

                Statement statement = connection.createStatement();
                String sql = "CREATE TABLE ACCOUNT (ID INTEGER NOT NULL, name VARCHAR(255) );";
                statement.executeUpdate(sql);
            } catch (Exception e ){
                e.printStackTrace();
            }

            jdbcTemplate.execute("INSERT INTO ACCOUNT VALUES (2, 'Minsik')");
        }
    }


어플리케이션이 실행되는 시점에서 Account라는 테이블을 생성하고 간단한 데이터를 Insert하는 쉬운 예제였는데 이 단계에서 다음과 같은 에러가 발생하였다.


    org.postgresql.util.PSQLException: Connection to localhost:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
        at org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:245) ~[postgresql-42.2.2.jar:42.2.2]
        at org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:49) ~[postgresql-42.2.2.jar:42.2.2]
        at org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:195) ~[postgresql-42.2.2.jar:42.2.2]
        at org.postgresql.Driver.makeConnection(Driver.java:452) ~[postgresql-42.2.2.jar:42.2.2]
        at org.postgresql.Driver.connect(Driver.java:254) ~[postgresql-42.2.2.jar:42.2.2]


## 해결 방법

**원인은 Docker**에 있었다. 내 노트북의 운영체제는 `Window 10 home으로 Docker for Windows를 사용할 수 없는 환경`이다. 따라서 `Docker Toolbox를 사용해야 하는데, 이는 가상머신 기반`으로 작동하게 된다(그래서인지 나는 설치한 적도 없는 오라큭 Virtual Machine이 설치가 되어 있었던 거군..) 즉, Docker를 사용해 Postgres를 설치한 공간도 가상머신이었던 것(내가 수강하는 강의는 Mac환경에서 진행되므로 Docker for mac을 사용한다는 점에서 나와 차이가 있었다.)

어쩃든, 그래서인지 강의에서 본대로 Postgres를 정상적으로 실행시켰음에도 불구하고 활성화 되어있는 포트의 정보를 확인했을 때 postgres전용 포트로 설정한 5432가 활성화되어 있지 않았던 것이었다. 

그래서 가상머신의 포트 포워딩 규칙에서 가상머신에 대한 포스트 포트와 게스트포트를 5432로 변경해주니 어플리케이션에서 정상적으로 Postgres에 접속할 수 있었으며 포트정보에서도 5432포트가 활성화 되어 있는 것을 확인할 수 있었다.

회사에서도 가상머신을 사용하지만, 기존 세팅되어있는 것을 그대로 사용해 로컬환경에서 테스트하기 위해 껐다 켰다하는 정도로만 사용하고 있어 자세히 알지는 못하지만 어렴풋한 기억에 비슷한 설정을 했던 경험이 있었고 그것이 이번 문제의 해결책이 되었다.

결론은 빨리 나도 맥북을....
 
