# 컴포넌트 List에서 index를 key로 사용하면 안되는 이유

리액트에서 컴포넌트를 List로 구성할 경우 별도의 key를 지정해주지 않으면 다음과 같은 에러가 발생한다. 

    index.js:1 Warning: Each child in a list should have a unique "key" prop.

이때 무심코 key로 사용하게 되는 값은 map 함수의 두번째 인자로 넘어오는 index이다. 그러나 이러한 index 값을 key로 사용할 경우 문제가 발생할 수 도 있다.

일단, 기본적으로 별도의 key를 지정하지 않을 경우 리액트는 자체적으로 이 index값을 key로 지정하게 된다. 만약 index를 key로 지정하였을 때 아무런 문제가 없다면 굳이 이렇게 에러를 발생시키면서까지 경고를 주지는 않을 것이다.

그렇다면 index를 key로 사용하지 말아야 하는 이유는 무엇일까? 컴포넌트 list에서 key의 역할은 무엇일까?

---

다음과 같은 컴포넌트의 리스트가 있다고 가정해보자

    const indexItems = this.state.items.map( (el, index) => {
        return <Child key ={index} name ={el.name}></Child>
    })

    => 렌더링 이후
    
    <Child key = '0' name ='A'>
    <Child key = '1' name ='B'>
    <Child key = '2' name ='C'>

만약 부모컴포넌트의 State에 다음과 같은 변화가 생겼다면 어떻게 될까?

    this.setState({
        items : [{name : "New"}, ...this.state.items]
    })

이 경우 위의 아이템은 다음과 같이 바뀌게 된다.

    <Child key = '0' name ='New'>
    <Child key = '1' name ='A'>
    <Child key = '2' name ='B'>
    <Child key = '4' name ='C'>


인간은 위의 컴포넌트에서 변경된 것은 단순히 아이템의 순서라는 것을 인지 할 수있지만, 리액트는 그렇지 못한다. 따라서 리액트는 이렇게 변경된 컴포넌트 리스트의 변경사항을 효율적으로 비교하기 위해 `key`를 사용하게 된다. 

즉, 변경 전과 변경 후의 컴포넌트 리스트에서 key값을 활용해 어떤 컴포넌트가 새로 생성되었고, 어떤 변화가 있었는지를 파악할 수 있게 된다. 이를 위의 예시에 대입해 이야기해보면, 실제 `name = 'C'`에 해당하는 마지막 컴포넌트는 이전부터 존재했던 컴포넌트임에도 불구하고 리액트의 입장에서는 `key = '4'`를 가진 새로운 컴포넌트로 인지하게 된다는 것이다. 

이는 리액트의 라이프사이클 훅과 함께 보면 더욱 명확히 이해할 수 있다. Child 컴포넌트에 다음과 같이 `shouldComponentUpdate` 활용해 `props`의 변경사항 내역을 추적해 보자.

    shouldComponentUpdate(nextProps, nextState){
        console.log( nextProps.name, this.props.name)
        return nextProps.name === this.props.name;
    }

다시 처음으로 돌아가 'New'를 추가하게 되면 콘솔에는 다음과 같이 로그가 찍힌다.

    New0 A
    A B
    B C

모든 컴포넌트들의 props는 변경되는 것으로 인지를 하며 결국엔 컴포넌트를 새로 렌더링하게되는 비효율이 발생하게 된다.

이는 단순 컴포넌트를 다시 렌더링하는 비효율적인 상황이외에도 데이터의 정합성에도 문제를 일으키게 된다. 

만약 각각의 Child내에 고유의 state가 있다고 가정하자. props로 전달되는 데이터와 별도로 state가 존재할 경우 state는 props가 아닌 컴포넌트의 key를 따라가게 되므로 컴포넌트가 추가된 전 후에 있어 컴포넌트간 state가 어긋나는 상황이 발생하게 된다. 이러한 예시는 [이곳](http://jsbin.com/wohima/edit?js,output)에서 제대로 확인해 볼 수 있다.

이처럼 컴포넌트 리스트를 활용할 때에는 반드시 컴포넌트의 index가 아닌 별도로 고유의 key값을 사용할 수 있도록 유의하여야 한다. 