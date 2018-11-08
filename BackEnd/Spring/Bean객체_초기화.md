
***
# Bean 객체 초기화

스프링을 통해 Bean객체가 생성될 때, 필요에 따라 Bean객체를 초기화할 필요가 있다. 

여기에는 매우 다양한 경우가 있을 것 같은데 특정 Bean객체의 값에 의존적인 어떤 작업이 어플리케이션의 로딩 직후 바로 이루어 져야 한다거나 하는 것이 그 예이다. 

이 글에서는 Bean객체가 생성되고 DI가 이루어진 직후 Bean객체를 초기화해줄 수 있는 두가지 대표적인 방법에 대해 알아본다. 



+ **@PostConstruct**
  
> `@PostConstruct`는 초기화하고 싶은 메소드 위에 기입함으로써 해당 초기화 기능을 수행할 수 있다. 

```
@PostConstruct 
public void init()
```


+ **InitializingBean**
 
> `InitializingBean`은 인터페이스로 구현을 하게 되면 `afterProperties()`라는 추상메소드를 오버라이드 함으로써 원하는 초기화 기능을 구현할 수 있다. 


```
@Component("VersionManageService")
public class VersionManageService implements InitializingBean {

        @Override
    public void afterPropertiesSet() throws Exception {
        
```



***
# @PostConstruct vs InitializingBean 

그렇다면 이 두가지 기능에는 어떤 차이가 있을까? 더 디테일한 차이점이 있을 수도 있겠지만 내가 판단하는 차이점은 바로 **시점차이**이다.

![Bean객체 초기화 및 소멸시기 ](/img/th3V1.jpg "Bean객체 초기화 및 소멸시기")

위 그림은 Bean객체의 라이프사이클이다. 많은 정보가 담겨있는 이미지이지만, `@PostConstruct `와 ` InitializingBean `의 관점에서만 보면 `@PostConstruct `가 ` InitializingBean `보다 앞서 실행되는 것을 알 수 있다. 나는 이 차이가 가장 큰 차이로 이해하였으며, 빈객체간의 복잡한 의존관계를 사용해야 할 필요가 있다면 참고하는 것이 좋을 것으로 보인다. 
##### (더 자세한 내용은 [이곳](https://stackoverflow.com/questions/50448238/spring-why-is-afterpropertiesset-of-initializingbean-needed-when-we-have-also-c) 참고)


***

내가 이 두가지 기능에 대해 알게 된 계기는 스프링 어플리케이션이 실행되기 이전에 DB의 version을 체크하고, 최신버전이 존재한다면 특정 경로에 [저장된 sql 파일을 DB에 적용하는 기능](../Java/Java에서_sql파일_실행하기.md)을 구현하기 위해서였다. 

그러나 해당 기능을 구현하기 위해서는 위에 명시한 두가지의 기능보다 먼저 수행되어야 한다는 문제점이 존재하였다. 이를 해결하기 위해서 다양한 방법을 찾아보았고, 그 해결책으로 [@DependsOn](@DependsOn_활용하기.md)이라는 어노테이션을 알게 되었다. 

