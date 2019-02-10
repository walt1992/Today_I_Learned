# Lombok 어노테이션 활용하기
 ###### 앞으로 나오는 모든 내용은 jojoldu님의 [블로그](https://jojoldu.tistory.com/250?category=635883)의 내용들을 참고해 작성하였습니다.
  스프링 개발을 더 쉽게 만들어준다는 Lombok이라는 것을알게 되어 간단히 정리해 보기로 하였습니다.
 **Lombok을 사용하면 DTO클래스에서 수행해야하는 다양한 설정들을 매우매우 간단하게 할 수 있다는 장점이 있다고 합니다**. 즉, 자바에서 모델객체를 생성하는 데에 있어 getter, setter, toString 등의 메소드를 설정하는 과정이 매우 귀찮게 느껴질 수도 있는데 lombok을 사용하게 되면 이러한 과정을 매우 간편하게 처리할 수 있습니다. 
 먼저 Lombok을 사용하기 위해서는 Dependency 설정을 해야 합니다.
 + Maven
```
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.2</version>
    <scope>provided</scope>
</dependency>
 ```
 + Gradle 
 ```
https://mvnrepository.com/artifact/org.projectlombok/lombok
provided group: 'org.projectlombok', name: 'lombok', version: '1.18.2'
```
 또한, Lombok은 의존성만 추가해서는 IDE에서 바로 사용할 수 없기 때문에 IDE에 맞게 사용환경을 구성할 필요가 있습니다.
 >[Intellij Lombok 설치](http://blog.woniper.net/229)
>
>[Eclipse Lombok 설치](http://countryxide.tistory.com/16)
 모두 완료되면 사용 준비는 끝났습니다. 
 너무 깊게 들어가는 것은 초보자인 나의 정신건강에 해로우니... 많이 활용되는 어노테이션을 기준으로 정리하도록 하겠습니다. 
 ***
 ### @Data
 **@Data는 Lombok의 종합세트**같은 녀석입니다. 이를 통해 아래에 소개되는 모든 어노테이션들을 한번에 설정할 수 있으니, 다음에 나오는 녀석들을 참고하세요!
   
+ **@toString** - 해당 메소드의 모든 필드를 출력하는 toString 메소드를 생성합니다.
 + **@EqualsAndHashCode** - hashcode와 equals 메소드를 생성합니다.
 + **@Getter / @Setter** - 말 그대로 getter함수와 Setter 함수를 생성합니다.
 + **@NoArgsConstructor** - 파라미터를 요구하지 않는 생성자를 생성합니다.  
   > (access =AccessLevel.PROTECTED) 를 추가하게 될 경우 기본생성자의 접근 권한을 protected로 제한하게 됩니다. 
 + **@RequiredArgsConstructor** - 파라미터를 요구하는 생성자를 생성합니다. 
 + **@AllArgsConstructor** - 모든 인자를 가진 생성자를 생성합니다. 
 이렇게 언급한 어노테이션을 삽입하게 될 경우, Class안에는 보이지 않지만 객체에서 사용할 수 있는 메소드들이 정의됩니다. **매번 DTO를 생성할 때마다 매우 귀찮은 작업을 했어야 했는데 매우 간편하게 작업을 할 수 있습니다.**
 ### @Builder
 다수의 멤버필드를 가진 클래스의 경우 객체를 생성했을 때 각각의 변수들을 setter 함수를 통해 입력하는 것이 매우 번거로운 작업입니다. 이는 생성자에 파라미터를 입력해 객체를 생성하는 것 또한 마찬가지이죠. 
 또한 앞서 설명한 
 `@NoArgsConstructor(access =AccessLevel.PROTECTED)`
 을 사용하게 될 경우 멤버필드 변수에 값을 채워넣기가 매우 애매합니다. 
 또한 `@RequiredArgsConstructor` 와 `@AllArgsConstructor` 또한 유지보수의 측면에서 매우매우 좋지않은 결과를 가져오기도 하기 때문에 활용하는것을 지양하는 편입니다. (이에 대한 이유는 뒤에 나오는 링크를 참고하세요)
 이때 @Bulider 어노테이션을 활용해 매우 간단하고 명료하게 멤버필드를 선언해 줄 수 있습니다. 이 경우. 예제코드와 같은 Builder패턴을 활용할 수 있음을 확인할 수 있습니다. 
       @Data 
      @Builder
      class Test(){
        private int a;
        private int b;
        private int c;
        
        @Singular
        private List<int> list; 
      
      }
      
      
      public static void main(String[] args){
          
        Test t = new Test.builder()
                          .a(1)
                          .b(2)
                          .c(3)
                          .list(3)
                          .list(4)
                          .build();
      }
  
 이처럼 매우 명료하고 간단하게 멤버필드변수를 선언해 줄 수 있습니다. 만약 멤버필드 변수중 Collection이 있을 경우 @Singular 어노테이션을 위와같이 선언해줄 경우, 모든 원소를 Collection객체에 담아 넣는 것이 아닌, 값을 하나하나 따로 넣어줄 수도 있습니다. 
 (추가 : DTO클래스에서 Builder를 사용했고, Mybatis를 통해 객체를 DB로 부터 불러오는 작업을 했는데, 생성자를 찾을 수 없다는 에러가 났습니다. Builder어노테이션을 활용하게 될 경우, 파라미터에 값을 입력함으로써 멤버필드 변수를 채우는 생성자를 사용할 수 없는 듯 보입니다. 자세한 부분은 추후 더 공부해봐야 할 것 같습니다. )
 ### @NonNull 
 @NonNull 어노테이션은 변수에 붙이는 어노테이션입니다. 변수에 들어온 값이 Null인지 아닌지를 판단해 주며, 만약 Null이 넘어올 경우 NullPointerException을 발생시킵니다. 
 이렇게 대략적으로 Lombok에서 지원하는 어노테이션 중 핵심인 부분들에 대해 알아보았습니다. Lombok은 코드를 작성할 때 매우 편리할 뿐만 아니라 추후 유지보수에도 매우 유용한 어노테이션들을 지원하니, 잘 활용하면 좋을 듯 합니다.
 다만, 몇가지 이유에 있어서 Lombok의 무분별한 사용을 경계하기도 합니다. 그 이유는 
 http://kwonnam.pe.kr/wiki/java/lombok/pitfall
 를 참고해 주세요.