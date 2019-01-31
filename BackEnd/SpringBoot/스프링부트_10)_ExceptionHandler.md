# 스프링부트 ExceptionHandler

>_본 글은 백기선님의 [스프링부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)강좌를 수강하며 정리한 내용입니다._

특정 Exception이 발생하였을 때, 특정한 처리를 해줄 수 있는 코드를 작성할 수 있다.

다음과 같은 Exception클래스가 있다고 가정하자.

    public class SampleException extends RuntimeException {
    }


만약 위 정의한 Exception이 발생했을 때 특정한 처리를 해주고 싶다면 다음과 같은 코드를 작성하면 된다.

    @Data
    public class AppError {

    private String message;

    private String reason;
    
    }

특정 에러메시지를 담은 객체를 위와 같이 가정하여 구현하였다.
이제 SampleException이 발생하였을 때, 특정 작업을 수행하고자 한다면 `@ExceptionHandler` 어노테이션을 활용해 다음과 같이 코드를 작성할 수 있다.      

    @ExceptionHandler(SampleHandler.class)
    public AppError sampleError(SampleHandler e){
        AppError appError = new AppError();
        appError.setMessage("error.app.key");
        appError.setReason("IDK");
        return appError;
    }

만약 위에 정의한 메소드가 특정 일반 컨트롤러 안에 정의되어 있다면 그 스코프는 해당 컨트롤러에 속한 Url의 요청에 한해 작동한다. 만약 해당 기능을 전역적으로 사용하고 싶다면 새로운 컨트롤러 생성하여 `@ControllerAdvice`어노테이션을 입력해주면 된다. 

이제 /exception이라는 Url로 호출할 때, SampleException이 발생한다고 가정하면 다음과 같은 메시지가 화면에 나타날 것이다

    {"message":"error.app.key","reason":"IDK"}


위와같은 방법을 통해 특정 Exception이 발생하였을 때 특정 화면으로 전환하거나 혹은 특정 데이터를 전송해주는 등 다양한 방식으로 활용할 수 있다. 


***

### Error Status Code에 따라 다른 웹페이지 보여주기

Resource/static 혹은 Resource/templates 디렉토리 아래에 error라는 디렉토리를 생성한다.

이후 Error Status Code와 일치하는, 혹은 비슷한 패턴(5xx.html)의 이름을 가진 html파일을 생성해주면 해당 에러 상태 코드가 발생할 경우 해당 코드와 매핑되는 이름의 정적 리소스를 리턴해준다.



