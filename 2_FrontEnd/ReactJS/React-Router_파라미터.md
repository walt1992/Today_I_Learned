#React-Router 파라미터

리액틀 라우터를 사용할 때 라우트의 경로에 특정값을 넣어 파라미터를 전달할 수 있다.

전달 방법으로는 `params`를 사용하는 것과 `query`를 사용하는 방식이 존재한다.
라우트로 설정한 컴포넌트는 3가지의 props를 전달받게 된다. 

![react-router-props](/img/react-router-props.png)

* **history** - push, replace를 통해 다른 경로로 이동하거나 앞 뒤 페이지로 전환 가능

* **location** - 현재 경로에 대한 정보를 지니고 있고 URL 쿼리 정보를 가지고 있음

* **match** - 어떤 라우트에 매칭되었는지에 대한 정보와 params에 대한 정보를 가지고 있음


### params 추가하기

    import React, {Component} from "react";
    import {Route, Switch} from 'react-router-dom';
    import {Home, About} from 'pages';

    class App extends Component{
        render(){
            return (
                <div>
                    {/* exact는 반드시 path와 동일한 문자열 요청에 대해서만 라우팅을 해주어야 한다는 의미
                        만약 exact가 빠질 경우 두 페이지가 모두 화면에 보여지는데 이는 /about에도 /가 포함되어 있기 때문이다.  */}
                    <Route exact path ='/' component={Home}/> 
                    {/* Switch는 react-router-dom의 컴포넌트로 매칭되는 첫번째 라우트만 보여준다 */}
                    <Switch> 
                        {/* /:name을 통해 name이라는 이름의 파라미터를 설정해 줄수 있다. */}
                        <Route path ="/about/:name" component={About}/>
                        <Route path ="/about" component={About}/>
                    </Switch>

                </div>
            )
        }
    }

위와 같이 컴포넌트의 URL path에 `/:name`과 같은 형식으로 params를 설정해 줄 수 있다. 이렇게 설정된 params는 해당 컴포넌트에서 다음과 같이 받아 활용할 수 있다. 

    const About = ({match}) =>{
        return (
            <div>
                <h2>About {match.params.name}</h2>
            </div>
        )
    }

이렇게 설정된 params은 다음과 같이 확인할 수 있다. 

![react-router-props](/img/react-router-props-params.png)

### URL query활용하기

query를 사용하기 위해서는 쿼리를 파싱하기 위한 라이브러리를 따로 설치해 주어야 한다.

    $yarn add query-string

앞서 말했듯 url의 정보를 가지고 있는 객체는 location이므로 컴포넌트에서 해당 객체를 받아와 쿼리를 파싱해 주어야 한다.

    import React from 'react';
    import queryString from 'query-string';
    const About = ({location, match}) =>{
        // 쿼리를 파싱
        const query = queryString.parse(location.search);
        console.log(query);

        // 쿼리로 넘어온 값은 모두 문자열이므로 
        // 별도의 타입에 맞게 설정해주어야 함
        const detail = query.detail ==='true';
        return (
            <div>
                <h2>About {match.params.name}</h2>
                { detail && 'detail : blahblah'}
            </div>
        )
    }

    export default About;


![react-router-props](/img/react-router-props-URLquery.png)

URL을 통해 넘어온 쿼리는 location객체에 다음과 같이 넘겨져 오는 것을 확인할 수 있다.