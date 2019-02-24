
# Java로 .sql파일 실행해 테이블 생성하기

팀장님이 재미있는 기능하나 구현해보라며 

> _**DB버젼이 바뀔때 마다 특정 경로에 있는 변경된 스키마 DDL이 적힌 sql파일을 Java에서 실행시키는 기능**_

을 주문하셨다.

물론 여러모로 실용성에 의문이 가는 기능이기는 하지만, **DB버전이 바뀔 때 마다 팀원들이 직접 DDL을 입력해 DB를 변경하는 것이 너무 귀찮을 것**이라고 여겼기 때문인 듯 하다. 

이와 비슷한 기능을 제공하는 라이브러리로 [Flyway](https://flywaydb.org/)가 있다고는 하나, 과금정책이 있는 오픈소스이기에 Community Ver이라 할지라도 사용하기가 꺼려졌다 _(공짜버젼이 따로 있는 이유가 있겠지...라는 단순한 생각에 의해서였다)_

어쨋든 설령 구현한다 할지라도 활용할지 안할지도 모를 _(물론 활용하더라도 실제 서비스에는 활용하면 절대 안되는 기능이다)_ 해당 기능을 구현하기 위해서 약 3일 동안 꽤나 고생했다. 역시...자바 공부는 소홀히하면 안된다. 

 서론은 각설하고, 해당 기능을 구현하기 위해서 여러가지 다른 이슈들도 존재했지만 그에 앞서 프로젝트의 ClassPath에 있는 .sql 파일을 가져와 통째로 쿼리를 실행하는 방법을 정리해본다. 

***

프로젝트의 Schema 정보를 관리하는 것은 매우 중요하다. 변경이 될 때마다 원본이 적힌 sql파일(ddl.sql)에 반영하고 그때그때 변경/추가된 정보를 담은 sql파일(ver_no.sql)을 따로 보관하고는 한다. 

내가 구현해야 하는 기능은 이 두가지를 모두 실행시킬 수 있는 로직을 짜는 것이었다. 즉, 프로젝트를 처음 실행시킬 경우에는 ddl.sql을, 그렇지 않을 경우 버전정보를 비교해 ver_no.sql을 실행시켜야 하는 것이다. 

이러한 파일은 프로젝트의 **src/main/resource**밑의 패키지에 보관해 놓았으며, 해당 경로에서 파일을 불러와 `Statement.executeQuery`를 실행해야 했다.



# ClassPath 리소스 불러오기


이렇게 ClassPath에 추가되어있는 파일을 불러오기 위해서 다음과 같은 코드를 작성했다.


+ **하나의 파일 가져오기**
 ```
  URL ddlUrl = ClassLoader.getSystemClassLoader().getResource( "com/a/schema/origin/ddl.sql");
```


+ **여러개의 파일 가져오기**
 ```
    ClassLoader loader = Thread.currentThread().getContextClassLoader();
    URL url = loader.getResource( "com/a/schema/update");
    String path = url.getPath();
    File[] files = new File( path).listFiles();
 ```

위 코드에 대한 자세한 설명은 생략한다(지금은 생각하고 싶지 않아...)



# java에서 sql 파일 실행하기

사실 파일을 불러오는 것은 쉽게 알아낼 수 있었다. 문제는 그렇게 불러온 쿼리들을 **_어떻게 처리할 것이냐_**  문제였다.

처음에는 `SemiColon(;)`을 기준으로 쿼리를 하나하나 나누어 `Statement.executeQuery()`를 활용해 쿼리를 날렸다. 이를 통해 대부분의 쿼리가 잘 작동하였지만, 문제는 PL/SQL을 실행시킬때 생겼다. PL/SQL은 다음과 같은 형태를 지닌다

```
create OR REPLACE function name() returns trigger AS $name$
    BEGIN
        IF NEW.state = 'Processing' THEN
            ~~~~~~~~~~ ;
        END IF;
            ~~~~~~~~~~;
    END;
$name$ LANGUAGE plpgsql;
```

즉, 하나의 PL/SQL문에는 여러개의  `SemiColon(;)`이 포함되어 있어 이를 기준으로 `split`을 하게될 경우 문제가 생겼다.

따라서 나는 sql 파일을 line단위로 읽어들여 execute하는 것이 아닌 sql파일 자체를 한번에 실행시킬 수 있는 방법을 찾아보아야 했다.


# ScriptRunner

그렇게 다시 열심히 검색을 하다가 알게된 클래스가 바로 `ScriptRunner.class`이다.
해당 클래스는 ibatis에서 지원한다고하는데, 자세한건 알아보지 않았다.

> 어쨋든! 덕분에 열심히 작성한 나의 sql을 나누고, 합치고, 쏘았던 코드를 모두 지워버려도 상관없었.....다.
> _(클래스의 내부를 들여다보니 내가짠 코드와 로직이 비슷해서 나름 뿌듯했다)_

사용법은 다음과 같다. 

 + **sql 파일의 쿼리를 따로 실행시키기**
```
    ScriptRunner runner= new ScriptRunner(conn);
 
    fis = new FileInputStream( fileUrl);
    reader = new InputStreamReader( fis);
    runner.runScript( reader);;
```

 >이 방법으로 실행할 경우 결과는 내가 직접 작성했던 코드와 동일했다. 
 이유는 내부를 들여다보니 알 수 있었다.

```
  private static final String DEFAULT_DELIMITER = ";";
```

>더이상의 설명은 생략한다...

+ **sql 파일의 쿼리를 한번에 실행시키기**
  
  >이를 위해서는 위의 코딩에 한줄만 추가해주면 된다. 

  ```
    runner.setSendFullScript(true);  
  ```

  >즉 쿼리를 실행하기 전에 `setSendFullScript(true)`을 설정함으로써 쿼리를 한번에 실행시키겠다는 것이다. 
  >여기서 의문이었던 것은, 왜 저런 기능을 **메소드의 다형성을 활용해 파라미터로 받는 것이 아닌, setter함수를 통해 받는것일까였다**. 

  >저러한 옵션이 있다는 것을 알게된 것도 클래스의 내부를 들여다 봤기에 알게된 것이다. 라이브러리에서 제공하는 클래스 내부도 한번씩 들여다보는 습관을 들여야지..



***

뭔가 다 쓰고 보니 내가 깊게 공부한것 같지는 않다... 다만 알아보는 시간이 좀 오래걸렸을 뿐. 어찌저찌 쓰다보니 글은 이렇게 길어졌지만, 그래도 꽤나 기억에 남기고 싶은 과정들이었던지라 이렇게 열심히 써보았다.([StackOverFlow에 작성한 질문](https://stackoverflow.com/questions/53203743/how-to-execute-create-trigger-sql-in-java))

이제 기능이 활용되는지에 대한 여부만 잘 기다려봐야지..

아 그리고 이 외에 발생했던 다른 문제점들 즉, [Bean객체 초기화방법](../Spring/Bean객체_초기화.md)에 대한 문제와 [Bean객체 생성순서](../Spring/@DependsOn_활용하기.md)에 대한 문제도 여기 있으니 참고하시길. 
