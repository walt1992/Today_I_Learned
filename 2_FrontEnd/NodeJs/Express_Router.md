    # Express Router

Express의 Router는 Spring으로 따졌을 때 Controller의 역할을 하는 녀석이다. 
주로 요청 URL에 따라 별도의 라우터로 분리해 사용하며 app.js에서 url별 미들웨어로 등록되어 활용된다.

> 라우터 등록 예

    app.use('/', defaultRouter);
    app.use('/users', usersRouter);

개별 라우터는 express.router()로 생성되며 use, get, post, patch, delete의 각기 다른 메서드로 요청을 처리할 액션을 등록한다.

>use - http 메서드에 상관없이 url이 일치하는 모든 요청에 동작한다.

    var express = require('express');
    var router = express.router();
    
    // /users url 호출에 모두 실행
    router.use('/users', function(req, res, next){
        // do something
    })
    
    // /users url 호출 중 get메서드일 경우에만 실행
    router.get('/users', function( req, res, next){
        // do something
    })

    router.post('/users', function( req, res, next){
        // do something
    })

    moduel.exports= = router;


라우터에 등록된 콜백함수의 인자값을 통해 `next()`함수가 전달된다. 여느 미들웨어와 같이 `next()` 함수를 호출하게 되면 해당 라우터에 등록된 다음 콜백함수로 이어지게 된다. 

    router.get('/', function(req, res, next) {
        // 실행 1
        next();
    }, function(req, res, next) {
        // 실행 2
        next('route');
    }, function(req, res, next) {
        // 실행 안됨
        next();
    })

    router.get('/', function(req,res,next){
        // 실행 3
    })

만약 위와같이 next('route')를 실행하게 될 경우 그 다음 이어지는 미들웨어는 건너뛰고 다음 라우터가 진행된다.


## Route URL 파라미터

    router('/user/:id', function(req, res, next){
        // req.params.id로 id 조회 가능
    })

이 역시 스프링 컨으롤러에서 제공하는 기능이다. 역시 대부분 비슷비슷하구나..
`url/:[파라미터 이름]`의 형태로 라우터를 등록하게 되면
url/pms3300 이라는 요청에 대해 해당 라우터 안에서 `req.params.[파라미터 이름]`을 통해 해당 값을 받아올 수 있다.

이런 패턴의 라우터는 가급적 다른 라우터보다 뒤에 배치하는 것이 좋다.

## 라우터의 Response 함수

클라이언트의 요청에 따라 응답되는 메서드들이 다양하게 있는데 대표적으로 다음과 같다.

* **res.send()** - 버퍼데이터, 문자열, HTML, JSON 등 전달
* **res.fendFile(파일경로)** - 파일을 응답으로 보내줌
* **res.json()** - json데이터
* **res.redirect(주소)** - 다른 라우터로 이동
* **res.render('템플릿 파일 경로', {변수})** - 템플릿 엔진 렌더링 시 사용

이러한 응답은 하나의 동일한 주소의 라우터에서 단 번만 보내야 한다.

    router.get('/', function(req, res, next) {
        res.send({ test: 'tests' });
        next()
    });
    router.get('/', function(req, res, next) {
        res.render('index', { title: 'Express' });
    });

> Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
at ServerResponse.setHeader (_http_outgoing.js:482:11)
