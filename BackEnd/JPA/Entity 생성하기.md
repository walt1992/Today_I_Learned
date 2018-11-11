#### (이 포스팅은 jojoldu님의  [2) 스프링부트로 웹 서비스 출시하기 - 2. SpringBoot & JPA로 간단 API 만들기](https://jojoldu.tistory.com/251) 튜토리얼을 참고하여 작성된 게시물입니다. Spring boot를 활용하여 간단한 웹서비스를 구축 및 배포하시고 싶으신 분들은 해당 게시물을 참고하여 주세요)





# JPA란 무엇인가?



**JPA란 Java Persistent API의 약자**로서, JavaSE, JavaEE를 위한 영속성 관리와 ORM을 위한 표준 기술입니다. ORM은 Object Relational Mapping의 약자로, RDB 테이블을 객체지향적으로 사용하기 위한 기술입니다. 



우리가 흔히 사용하는 RDB 테이블은 객체지향적인 특징이 없고 자바같은 언어로 접근하기가 쉽지 않습니다. ORM은 이러한 RDB 테이블을 객체지향적으로 사용할 수 있도록 발전된 기술이며, 그 표준 기술로는 Hibernate, OpenJPA, TopLink Essencial 등이 있으며 이러한 구현체들의 표준 인터페이스가 바로 JPA 입니다. 



 이러한 JPA의 장점은 



 + 객체지향적으로 데이터를 관리할 수 있기에 복잡한 쿼리가 아닌 비즈니스 로직을 통한 프로그래밍에 집중할 수 있습니다.
 + 테이블을 생성하고 변경, 관리하기 쉽습니다.
 + 빠른 개발이 가능합니다. 

# Entity 생성하기



자바에서 Entity 클래스는 하나의 테이블 객체를 표현한 것입니다. 



Class의 상단에 `@Entity` Annotation을 넣어 주는 것으로 **테이블과 매핑될 클래스를 지정**할 수 있습니다. 



```
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 기본생성자 자동 추가, 
                                                      기본생성자의 접근권한 설정
@Getter // 
@Entity // 테이블과 링크될 클래스이다. 언더스코어 네임밍(_)으로 이름을 매칭한다.
        // SalesManager.java >> sales_Manager table
public class Posts extends BaseTimeEntity{
```





# Column 설정하기


Entity 클래스가 테이블과 매핑이 된다는 것에서 눈치채셨을 테지만, 해당 클래스 안에 존재하게 되는 멤버필드 변수들은 곧 해당 테이블의 컬럼과 같은 역할을 하게 됩니다. 이러한 컬럼에도 여러가지 설정을 할 수 있는데 이와 관련해 알아보겠습니다. 





**@Column**



멤버필드 변수 위에 해당 어노테이션을 기입함으로써 컬럼임을 지정합니다. 



```
    @Column(length = 500, nullable = false) // 굳이 선언하지 않아도 컬럼이 되나 ,
                                            // 기본값에 변형을 주기 위해서 사용한다 .
    private String title; // 문자열의 경우 varchar(255)가 기본 옵션이다.
```




이러한 컬럼에는 여러가지 옵션이 존재합니다. 



+ name : 컬럼의 네임을 따로 명시합니다
+ length : 속성의 길이를 지정합니다.
+ insertable : 컬럼에 대하여 insert 허용여부를 설정합니다.
+ updatable : 컬럼에 대하여 update 허용여부를 설정합니다. 


그 외에도 여러 옵션이 존재하니 더 궁금하신 분들은 [참고](http://itdoc.hitachi.co.jp/manuals/3020/30203Y2110e/EY210078.HTM)하세요.



**@id** 

**@GeneratedValue**
```
    @Id // 테이블의 PK가 된다
    @GeneratedValue(strategy = GenerationType.AUTO)// PK의 생성규칙,스프링부트  2.0이상에서는 이와 같은 옵션이 필수
    private long id;
```

테이블의 PK조건을 지정하기 위한 어노테이션입니다.

Id의 타입은 가급적이면 Long 타입으로 지정하는 것을 추천합니다. 





**@OneToMany**

**@ManyToMany**

**@ManyToOne**



테이블간에는 서로 참조관계라는 것이 존재합니다. 이 세가지 어노테이션은 그러한 관계를 보여주는 어노테이션 입니다. 관련 포스팅은 [이곳](https://jacojang.github.io/jpa/java/hibernate/2016/12/27/jpa-chapter6-%EB%8B%A4%EC%96%91%ED%95%9C_%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84_%EB%A7%A4%ED%95%91.html)을 참고하세요.


***


이렇게 기본적인 Entity클래스를 구성하는 기본적인 방법들에 알아보았습니다. 



기존에 Spring을 한번이라도 사용해 보셨던 분들이라면 이러한 Entity와 흔히 DTO, VO라고 하는 클래스와 매우 흡사한 모양을 하고 있다는 것을 느끼셨으리라 생각합니다. 그렇다고 한들, **Entity클래스를 절대 Request, Response를 위한 DTO클래스로 사용하시면 안되고, 따로 DTO클래스를 생성해 활용하셔야 합니다.** 



저도 아직 익숙치 않은 상태에서 이것저것 자료를 찾아보며 정리한 것이니 혹시 잘못된 내용이 있다면 따끔히 지적해주시면 감사하겠습니다!





