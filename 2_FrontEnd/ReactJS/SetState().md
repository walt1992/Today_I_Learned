#SetState()가 비동기인 이유

리액트를 연습하던 중 다음과 같은 코드를 간단히 작성하였다.

    handleAddTodo = (e) => {
        e.preventDefault();
        this.setState({
            date : new Date().toDateString()
        });        
        console.log(this.state)
        this.props.addTodo(this.state);
        this.setState({
            name : '',
            date : ''
        })
    }

하위 컴포넌트에서 상위 컴포넌트에 State를 전달하는 코드이다. 

상위 컴포넌트로 State를 전달하기 전에 `date` 요소에 특정 value를 입력해 넘겨주는 것이 본래의 의도이다. 그러나 나의 기대와는 달리 date에 원하는 값이 제대로 들어가지 않은 채로 상위 컴포넌트에 전달되었다.

~~나는 `setState()`가 비동기로 state를 업데이트하리라 짐작하였고 찾아보니 그 이유가 맞아 떨어졌다.~~ ~~따라서 이를 해결하기 위해 **async/await를 사용**하여 다음과 같이 코드를 수정하였다.~~

    handleAddTodo = async (e) => {
        e.preventDefault();
        await this.setState({
            date : new Date().toDateString()
        });
        this.props.addTodo(this.state);
        console.log(this.state)
        ;
        this.setState({
            name : '',
            date : ''
        })
    }

만약 setState가 정말 비동기 함수라면 Promise를 리턴하도록 설계해놓지 않았을까? 하는 생각이 들었다. 그러나 정작 해당 함수의 리턴값은 아무것도 없기에 혹시나 하는 마음에 다시 알아보니 `setState()`는 엄밀히 비동기함수가 아닌 일종의 **lazy한 함수**라는 것을 알게 되었다. 다음은 [React Document](https://reactjs.org/docs/react-component.html#setstate)에서 발췌한 내용이다.

>Think of setState() as a request rather than an immediate command to update the component. For better perceived performance, React may delay it, and then update several components in a single pass. React does not guarantee that the state changes are applied immediately.
***

즉, `setState()`는 컴포넌트가 리렌더링되는 등의 과정에서 성능을 고려하여 배치작업의 형태로 업데이트를 수행하는 듯 보인다.

따라서 이어지는 문장은 다음과 같다.

>This makes reading this.state right after calling setState() a potential pitfall. Instead, use componentDidUpdate or a setState callback (setState(updater, callback)), either of which are guaranteed to fire after the update has been applied.

`setState()`를 호출한 후 this.state에 접근하는 방식은 옳지 않으며, **setState()의 callback함수**를 사용하거나 **componentDidUpdate()함수를 활용**할 것을 제안한다.

나는 현 이슈에 대해 CallBack함수를 활용하는 것이 옳다고 생각하여 다음과 같이 코드를 다시 수정하였다.

    handleAddTodo = (e) => {
        e.preventDefault();
        this.setState({
            date : new Date().toDateString()
        }, () =>{
            this.props.addTodo(this.state);
            this.setState({
                name : '',
                date : ''
            })
        });
    }


### setState()가 
