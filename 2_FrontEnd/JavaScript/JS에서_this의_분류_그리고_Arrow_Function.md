# JS에서 this의 종류, 그리고 Arrow Function

자바스크립트에서 this는 어느 상황에 쓰이느냐에 따라 서로 다른 대상을 참조하게 된다. 코드를 작성할때면 이 부분에 항상 신경쓰면서 작성을 하긴 하지만, 가끔 나도 현재 this가 가르키는 것이 무엇인지 간혹 헷갈리는 경우가 있어 간단하게 정리해보려 한다. 

일반적으로 Java와 같은 언어에서 this는 객체 인스턴스 자신을 의미하게 된다. 그러나 자바스크립트는 `this`를 어느 맥락에서 사용하느냐에 따라 그 참조하는 값이 완전히 달라지게 된다. 

## 1. 함수 내부

일반적인 함수 내부에서 this를 사용하게 될 경우 참조하게 되는 값은 `window`객체이다(node에서는 global을 참조하게 된다). 

## 2. 메서드 내부 

함수와 메서드의 차이는 해당 함수가 특정 객체 프로퍼티의 일부인지 여부의 차이이다. 만약 **특정 오브젝트의 프로퍼티로서 함수가 정의되어 있다면 해당 함수는 메서드**라고 부른다.

이렇게 정의한 메서드의 내부에서 `this`의 값은 함수가 속해있는 객체 그 자체가 된다.

    function func(){
        return this;
    }

    const obj = {
        method : func
    }

    console.log(func() === window); // true
    console.log(obj.method() === obj); // true

위와같이 이미 정의되어 있는 함수를 객체의 프로퍼티값으로 할당할 경우 기존 `window`를 참조하던 this는 `obj`를 참조하게 된다.

## 3. 생성자 함수 내부

ES6이후 자바스크립트에도 여타 객체지향 언어처럼 class라는 개념이 생겼다. 그러나 이러한 개념이 없던 이전에는 생성자 함수라는 것을 사용해 객체지향의 형태로 코드를 작성했다. 생성자 함수는 일반 class와 같이 특정 포맷을 지닌 객체를 생성하기 위한 함수이다. 이러한 생성자 함수 내부에서의 this는 생성자 함수를 자기 자신을 가르키게 된다.

    function Test(a, b){
        this.a;
        this.b;
    }

    Test.prototype.func(){
        console.log(this.a, this.b);
    }

    const test = new Test(1, 2);
    test.func(); // 1, 2

또한 위의 예시에서 처럼 생성자 함수의 프로토타입을 정의할 경우에도 그 내부에서 참조하게 되는 `this`는 생성자 함수 자신이 된다.


## 4. 콜백함수에서의 this

자바스크립트를 사용하다보면 콜백함수를 상당히 많이 사용하게 된다. 그러나 이러한 콜백함수에서의 this는 특정하기 어렵다. 즉 어떤 경우는 일반 함수에서와 같이 window를 참조하게 되지만, 특별한 경우 콜백함수에 자체적으로 다른 값이 this로 바인딩 되기도 한다. 그 대표적인 경우가 `addEventListener에` 정의되는 콜백함수이다. 

특정 DOM에 이벤트를 바인딩하는 addEventListener함수의 인자로 전달되는 콜백함수 내부에서의 this는 이벤트가 바인딩되는 타깃DOM객체가 된다. 

    document.getElementById('#test').addEventListener('click', function(e){
        console.log(this === document.getElementById('#test')); // true 
    })

---

이처럼 자바스크립트에서의 `this`는 함수가 어떤 환경에서 실행되느냐에 따라 다른 값으로 바인딩 된다는 점에서 여간 까다로운일이 아닐 수 없다. 이처럼 함수가 호출될 때마다 `this`의 값이 새롭게 바인딩되는 것이 싫다면 사용할 수 있는 것이 바로 ES6의 `Arrow Function`이다. 

다음 예제를 보자.

    function Test(){
        this.a;
        setTimeout(function(){
            console.log(this);
        })
    }
    const test = new Test(1);

위 코드를 실행할 경우 this는 어떤 값이 출력되게 될까? 바로 `window`이다. 만약 이때의 this를 생성자 함수를 통해 생성한 객체로 바인딩해서 사용하고 싶다면 어떻게 해야할까?

기존의 방식이라면 다음 두가지 경우를 고려해 볼 수 있을 것이다.
    
* **클로저를 활용한 방식**

        function Test(){
            this.a;
            const that = this;
            setTimeout(function(){
                console.log(that);
            })
        }
        var test = new Test(1);

* **call 혹은 apply를 활용한 방식**

        function Test(){
            this.a;
            setTimeout((function(){
                console.log(this));
            }).apply(this,[]))
        }
        var test = new Test(1);

그러나 매번 코드를 작성할 때마다 이런식의 조치를 취하는 것은 여간 귀찮고 불편한 일이 아닐 수 없다. 그렇다면 Arrow Function을 사용하게 되면 어떻게 될까?

앞서 말했듯, Arrow Function은 새로운 this를 바인딩하지 않는다. 즉, 함수가 정의된 그 시점에서의 상위 스코프 this에 해당하는 값을 그대로 사용할 수 있게 된다.

* **Arrow Function을 사용한 방식**

        function Test(){
            this.a;
            setTimeout(() => {
                console.log(this);
            })
        }
        var test = new Test(1);


---

이렇게 자바스크립트 코드를 작성할 때 한번씩 헷갈리는 `this`를 정리해 보았다. _(정리함에 있어 참고한 책은 호랑이책, `코어자바스크립트`와 MDN이다.)_ 역시 평소 알고있던 개념이라도 한번쯤은 나만의 언어로 정리하는 것이 매우 의미가 있는 것같다. 훨씬 더 명확해진 느낌이랄까? 앞으로 자바스크립트에 대한 핵심 개념을 시간날 때마다 정리하는 습관을 길러야겠다.

이로써 오늘도 의미있는 One day, One commit 완료! 