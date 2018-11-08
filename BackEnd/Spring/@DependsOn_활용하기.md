# Bean객체 생성 시점 정하기

스프링에서 Bean객체가 생성되고 의존성 주입이 될 때, 특정 Bean객체가 정상적으로 생성되기 위해서는 타 Bean객체에 의존적이어야 하는 경우가 종종 생긴다. 

나의 경우는 DB의 스키마를 DBeaver에서 직접 DDL을 통해 입력하는 것이 아닌, 어플리케이션을 실행할 당시에 스프링에서 자동으로 [JDBC를 통해 쿼리를 특정 경로에 저장된 .sql파일 통째로 Execute하는 기능](../Java/Java에서_sql파일_실행하기.md)을 구현하였는데 
> ###### _(알아보니 이런 기능은 별로 좋지않은 기능이라도 한다. 자세한건 모르지만 보안과 관련된 부분이라고... 어쨌든, 단순히 협업에 있어서 팀원들과 편하게 작업을 하기 위해 구현했던 기능이었다)_


이 경우 `@PostConstruct`에 의해 미리 실행되는 메소드 내에서의 CRUD작업이 발생할 경우 문제가 되었다. 따라서 **Bean 객체를 초기화하는 특정 메소드들이 실행되기 이전에 DB스키마를 업데이트하는 기능을 실행**시켜야만 했다. 

그렇게 알게된 것이 `@DependsOn`이라는 어노테이션이었다.

***


### @DependsOn( name = "Target Bean" )

`@DependsOn`
 은 특정 Bean객체간의 의존성 주입 시점을 정해줄 수 있는 어노테이션이다. 

다음의 코드를 보면 쉽게 이해가 갈 것이다. 

```
@Component("test1")
@DependsOn(name = "test2")
public class Test1()

@Component("test2")
public class Test2()
```

위의 예시에서 Bean으로 등록되는 순서는 **Test1 > Test2** 이다. 
이 설정을 통해 Bean 객체로 등록되는 Class들의 등록 순서를 정해줄 수 있다. 

***

한가지 아쉬운 점은 나의 경우 이러한 방법으로 원하는 기능을 구현하였을 때, **DB를 조회해야하는 모든 @PostConstruct가 적용된 메소드마다 해당 설정을 해주어야** 한다는 한계점이 있었다. 

다른 방법이 있는지 열심히 찾아보았지만, 이렇다할 성과를 얻지는 못하였다. ([내가 올린 질문](https://stackoverflow.com/questions/53202669/how-to-execute-method-before-postconstruct-work))

그래도 꽤 유익한 시간이었다. 
