# Express 미들웨어란?

미들웨어는 express에서 요청과 응답을 조작하여 기능을 추가하거나 나쁜 요청을 걸러내는 작업을 한다고 한다. 이렇게 얘기를 들으니 약간은 Spring의 AOP와 비슷한 역할인거 같기도 하다. 

미들웨어는 express모듈을 통해 생성된 app객체에 등록된다. 

    var express = requires('express');
    var app = express();
    app.use( [미들웨어 함수]);

미들웨어는 `app.use` 함수를 통해 등록된 순서로 실행되어 모든 미들웨어를 거치고 난 후 클라이언트에 응답을 보내게 된다.
> ### 미들웨어 한번에 등록하기 
>
> app.use(  [미들웨어 함수],  [미들웨어 함수],  [미들웨어 함수], ...)
>
> ex. app.use( logger('dev'), express.json(), ...)

## 커스텀 미들웨어 만들기

미들웨어를 만드는 방법은 간단하다. `app.use` 함수에 미들웨어로 실행될 함수를 인자로 넘겨주기만 하면 된다. 

    app.use( (req,res, next)=> {
        console.log('first middle ware')
        next();
    });

    app.use( (req,res, next)=> {
        console.log('second middleware')
        next();
    })

위와같이 미들웨어를 등록하고 npm start 후 요청을 보내면 다음과 같이 로그가 찍히는 걸 확인할 수 있다.

    first middle ware   
    second middleware

이때 각각의 미들웨어에서 next()를 호출하지 않을 경우 다음 미들웨어로 넘어가지 않는다는 것에 주의해야 한다. 

> next() 함수는 세가지 타입으로 호출할 수 있다.
> 
> * next()         - 다음 미들웨어
> * next('router') - 라우터 미들웨어로 이동
> * next(error)    - 에러핸들러로 이동


    