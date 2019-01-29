# @JsonTypeInfo 어노테이션을 활용해 추상클래스 매핑하기

지난번에 `ObjectMapping` 객체를 활용해 Json을 class 객체로 매핑하는 것을 공부하였다([ObjectMapper 활용하기](ObjectMapper.md) ). 이번에는 ObjectMapping과 함께 활용할 때 큰 이점을 볼 수 있는 `@JsonTypeInfo`에 대해서 공부한다.

***

특정 추상클래스를 상속하는 클래스들의 Json Serialize 혹은 Deserialize를 해야 할 때 매우 유용하게 사용할 수 있는 방법이다.

다음과 같은 클래스가 있다고 가정하자.

    public abstract class Car()

    public class Benz() extends Car

    public class HyunDai() extends Car


`Car`라는 추상클래스를 상속하고 있는 `Benz`와 `Hyundai`라는 클래스를 가지고 있는 List객체를 생성해 Garage.class 객채의 변수로 등록해 보자.

    List<Car> list = new ArrayList<>();
    list.add( new Benz());
    list.add( new Hyundai());

    Garage garage = new Garage();
    garage.setMyCars( list);

만약 위에 정의한 `garage`를 Json String으로 변환해주고 싶다면 어떻게 해야 할까?
단순 JsonString으로 Serialize를 해주는 방법은 다음과 같다.


        ObjectMapper mapper = new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

        try {
            String json = mapper.writeValueAsString( garage);
            System.out.println( json);

        } catch (Exception e){
            e.printStackTrace();
        }

        >> 결과 : {"myCars":[{"door":3,"wheel":4,"hyun":"hyun"},{"door":1,"wheel":1,"ben":"ben"}]}


문제는 JsonString의 형태를 다시 클래스 객체로 Deserialize할 때 발생한다.

        try {
            garage= mapper.readValue(json, Garage.class);
            List<Car> list2 = garage.getMyCars();
        } catch (Exception e){
            e.printStackTrace();
        }


        com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `veloer.eco.Car` (no Creators, like default construct, exist): abstract types either need to be mapped to concrete types, have custom deserializer, or contain additional type information
        ce chain: veloer.eco.Garage["myCars"]->java.util.ArrayList[0])


`garage`를 다시 객체로 전환하는 과정에서 List<Car>로 정의되어있는 `MyCars`를 역직렬화 하는 과정에서 에러가 발생한다. 이는 `myCars`의 요소로 포함되어 있는 `Benz.class`와 `HyunDai.class`객체를 Car 추상클래스와 온전히 매핑을 하지 못하기 때문이다. 이러한 문제를 해결해 주기 위해서는 `Car.class`에 추가적인 설정을 해주어야 한다.


***

### 자식클래스 매핑을 위한 설정

    @Data
    @JsonTypeInfo(use = JsonTypeInfo.Id.NAME, 
            include = JsonTypeInfo.As.PROPERTY // 타입의 정보를 Property로 구분
            property = "type" , // 타입을 결정지을 Property의 이름
            ) 
    @JsonSubTypes({
            @JsonSubTypes.Type(value = Hyundai.class, name = "hyundai"), // type의 이름과 매핑될 클래스 
            @JsonSubTypes.Type(value = Benz.class, name = "benz")
    })
    public abstract class Car {


역직렬화를 하기에 앞서 위의 코드를 Car.class에 추가한 후 다시 직렬화하는 코드를 실행하면 다음과 같은 결과가 나온다.

    {"myCars":[{"type":"hyundai","door":3,"wheel":4,"hyun":"hyun"},{"type":"benz","door":1,"wheel":1,"ben":"ben"}]}

이전에는 없던 Type이라는 변수가 자동 생성되며, 해당 위에서 `@JsonSubType`에 정의한 class과 name이 매핑되어 저장된 것을 확인할 수 있다.

즉, 해당 설정을 통해 알맞는 클래스로 자동 매핑이 되기 일종의 표시를 남긴셈이다.

이제 이렇게 생성한 Json String을 다시 역직렬화를 하면 다음과 같은 결과가 나온다.

      try {
        
            garage= mapper.readValue(json, Garage.class);

            List<Car> list2 = garage.getMyCars();


            System.out.println(list2.get(0).getClass());
            System.out.println(list2.get(1).getClass());
        } catch (Exception e){
            e.printStackTrace();
        }

        // 결과
        >> class veloer.eco.Hyundai
        >> class veloer.eco.Benz           

즉, `@JsonTypeInfo`와 `@JsonSubType`의 설정을 통해 추상클래스를 상속한 객체들을 성공적으로 매핑된 것을 확인할 수 있다. 