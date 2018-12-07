# X축으로 Overflow되는 현상 해결하기

오랜만에 주말여유가 생겨 미루고 있던 블로그 UI작업을 진행중이었다. 사이드바의 디자인을 얼추 마친 후, 바디에서의 목록을 보여주는 작업을 진행중, 계속해서 X축으로 overflow가 되는 현상이 생겼다.

화면을 보면 다음과 같다. 


![블로그 X축 overFlow](/img/blog_x_overflow.gif "Bean객체 초기화 및 소멸시기")


블로그의 전체 화면은 display flex로 해놓은 후, 화면 구성을 sidebar와 body, 그리고 right 등 3개의 세션으로 나누어 놓았다.

이때, sidebar와 right의 경우 특정 px을 주어 크기를 고정하였고, body의 경우 `flex : 1 1 auto` 옵션을 주어 세션의 크기를 설정하였다. 

    /*sidebar*/
    .blog-sidebar{
      height: 100%;
      min-width: 350px;
      flex-direction: column;
      background-color: #4a4f6a;
    }

    /*body*/
    .blog-body{
      height: 100%;
      flex: 1 1 auto;
      display: flex;
      background-color: #f8f8f8
    }

    /*right*/
    .blog-right{
      width: 450px;
    }


그런데 왜 X축으로 overflow되는 현상이 발생하는 것일까?, 단순히 body에 있는 content들이 overflow되는 것이 아닌, body 크기 자체가 늘어나는 현상이 발생해 꽤나 애를 먹었다.

overflow가 생겨난 시점은 list에 서의 아이템 타이틀을 보이는 것과 같이 매우 길게 설정했을 경우 발생하였다. 이에 대해` white-space: nowrap` 과 `overflow:hidden`설정을 해 두었음에도 body의 크기는 어느 정도까지 커진 후 해당 옵션이 적용되었다. 

이러한 오류는 생각보다 간단하게 `.blog-body`에 `overflow-x:hidden` 옵션을 하나 추가하니 해결되었다. 

또한, sidebar와 right에 각각 width가 아닌 `min-width`로 옵션을 변경함으로써 전체적인 틀을 다음과 같이 보정할 수 있었다. 


![X축 overFlow 수정](/img/fixed_overflow_x.gif "Bean객체 초기화 및 소멸시기")


일단 이런 방식으로 해결하긴 하였지만, 여전히 왜 그런 현상이 나타난 것인지는 의문이다. 

아무래도 CSS를 체계적으로 공부하지 않은 탓이겠지... 또한 flex를 너무 남발하듯 사용했기 때문일 수도 있으리라 생각도 들었다. 이부분은 UI디자인을 더 진행해 나가면서 하나하나 알아가볼 생각이다. 