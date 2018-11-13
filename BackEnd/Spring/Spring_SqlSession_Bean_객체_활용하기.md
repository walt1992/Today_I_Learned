# SqlSessionFactory Bean객체를 활용해 DB연결하기

Spring 프로젝트를 진행하는 중, sql파일을 [java에서 실행시키는 기능을 구현](/java/Java에서_sql파일_실행하기.md)해야 했다.

가장 먼저 해야할 일은 DB에 Connect하는 것이었다. 처음에는 일반 JDBC의 방식대로 다음과 같이 코드를 작성하였다.

    Connection conn = DriverManager.getConnection("URL","USER","PWD");
    

그러나, Spring을 사용할 경우 이미 Bean으로써 DB connection pool을 등록해 두기 때문에 이와같이 따로 DB에 접근할 필요가 없었다.

그에 따라 다음과 같이 코드를 변경하였다 .

먼저, **DB연결을 위한 `sqlSession `Bean객체**는 다음과 같이 등록되어 있다.


    <bean id="defaultSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"    name="defaultSqlSessionFactory">
        <property name="dataSource" ref="defaultDataSource" />
        <property name="mapperLocations" value="${datasource.default.mapperLocations}" />
        <property name="configLocation" value="classpath:mybatis-config.xml" />
    </bean>
  


이렇게 등록된 Bean객체를 불러와 **@Autowired**를 통해 초기화를 시켜준다.

 ### 초기화

     @Autowired SqlSessionTemplate defaultSession;

이후, `SqlSession`객체를 생성해 다음과 같이 설정해주면 DB연결이 간단하게 끝난다.

 ### Connection


    SqlSession sess = defaultSession.getSqlSessionFactory().openSession();
    Connection conn = sess.getConnection();



이렇게 DB에 수행해야할 작업이 끝난 후

    conn.close()
    sess.close()

는 잊지 말자. 이와 관련해 나는 다음과 같은 메소드를 생성해 활용하였다. 한 Class에서 여러번의 DB 커넥션이 이루어질 경우 매우 유용하며 코드 또한 깔끔하게 유지할 수 있다. 

    private void close() throws Exception{
            try {
                if( rs !=null ) rs.close();
                if( reader !=null ) reader.close();
                if( fis !=null ) fis.close();
                if( conn !=null ) conn.close();
                if( sess !=null ) sess.close();
            } catch (Exception e) {
                logger.error("Failed JDBC Pool or IO Close");
                throw e;
            }
        }

***

> *사실 처음부터 이러한 방법에 대해서 생각해보지 않았던 것은 아니다. 다만, 아직 Spring의 Bean객체 의존성 주입에 대한 개념을 평소 간과하며 기계적으로 코드를 작성하다보니 직접 실행에 옮길 생각을 하지 못한 것 같다. 그리고 mybatis에 너무 의존적이었구나 라는 반성도 하게 된다. 이러한 개념은 이외에 많은 부분에 있어 앞으로 코드를 작성하는 데에 효율성을 높혀줄 수 있으리라 생각한다.*