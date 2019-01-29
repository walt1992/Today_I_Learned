# Object Mapper 클래스 활용하기

자바의 클래스 객체를 json형태로 변환하거나 클라이언트로 부터 받아온 Json String을 다시 클래스 객체로 Mapping해야 하는 경우가 종종 생긴다. 이럴때 활용하기 좋은 클래스이다. 나는 회사에서 ES Query를 생성하기 위해 이 클래스를 활용하기도 하였다. 

가장먼저 디펜던시를 추가해 준다.

### 디펜던시 추가 

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.4</version>
    </dependency>
 
 ***

### Serialize

    ObjectMapper objectMapper = new ObjectMapper();
    Car car = new Car("yellow", "renault");
    objectMapper.writeValue(new File("target/car.json"), car);


위와같이 활용할 경우 `writeValue`메소드를 활용하여 car라는 객체를 Json의 형태로 변환하여 파일에 입력할 수 있다.

### Json 형태의 String을 객체로 Mapping하기

반대로 `readValue` 메소드를 활용하여 Json형태의 String을 java 객체로 Mapping할 수 있다. 

    String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
    Car car = objectMapper.readValue(json, Car.class);  

    Car car = objectMapper.readValue(new File("target/json_car.json"), Car.class); // 파일에 입력되어 있는 json mapping

    Car car = objectMapper.readValue(new URL("target/json_car.json"), Car.class);
    // URL주소로 부터 읽어오기

### Collection 형태로 반환하기

Json String이 배열의 형태로 존재할 경우 `TypeReference.class` 를 함께 활용해 List객체로 반환 받을 수 있다.

    String jsonCarArray = "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : \"Red\", \"type\" : \"FIAT\" }]";
    List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>(){});

이러한 원리는 Map형태의 Json String을 변환하는데도 활용할 수 있다.

    String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
    Map<String, Object> map = objectMapper.readValue(json, new TypeReference<Map<String,Object>>(){});


### 추가지원 기능

간략하게 ObjectMapper객체의 기능을 살펴보았다. 이 이외에도 Serialize 및 Deserialize가 되는 방식을 사용자가 직접 정의할 수 있는 기능 등 유연성있게 Mapping할수 있는 기능들을 지원한다.

이와 관련된 정보는 [이곳](https://www.baeldung.com/jackson-object-mapper-tutorial)을 참고.

또한 ObjectMapper 클래스는 @jsonTypeInfo 어노테이션과 함께 활용할 때 그 활용도는 배가된다. [@JsonTypeInfo 활용하기 참고](@JsonTypeInfo.md)
