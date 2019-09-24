# Element.getBoundingClientRect()

자바스크립트를 활용해 동적으로 컴포넌트를 생성하여 절대위치에 Append해야하는 경우가 종종 있다. 혹은 특정 컴폰너트의 크기, 위치값을 추출해 와야 한다거나. 

이런 작업은 생각보다 귀찮다. 타겟 요소의 위치값을 얻기 위해 offsetWidth, offsetTop 등 offset을 활용해 가져오지만, 해당 프로퍼티 명도 매번 헷갈려 자주 실수를 하게 된다.

이번에도 회사에서 동적으로 생성되는 컨텍스트 메뉴 컴포넌트를 직접 구현해야 할 일이 있었다. 2-depth로 이루어진 컨텍스트 메뉴였기에 1-depth의 메뉴에 호버를 할 경우 2-depth의 메뉴가 바로 옆에 동적으로 생성되는 UI를 구현해야 했다. 

어떻게 해야 구현해야 할까 고민하다가 해당 아이디어를 얻기 위해 타 서드파티 컴포넌트 라이브러리의 소스를 들여다 보았다. 그곳에서 알게된 함수가 바로 `Element.getBoundingClientRect()`이다.

처음보는 함수였기에 뭐에쓰는 놈인가 하여 직접 사용해 보았다. 그때 리턴된 값은 다음과 같았다.

    DOMRect {x: 1321, y: 706, width: 202, height: 50, top: 706, …}
         bottom: 756
         height: 50
         left: 1321
         right: 1523
         top: 706
         width: 202
         x: 1321
         y: 706
         __proto__: DOMRect

즉 DOMRect 객체가 리턴되는데, 이 안에는 해당 DOM요소의 전체 viewport기준으로 위치와 크기에 대한 값이 포함되어 있다. 이 객체를 활용해서 원하는 타겟의 위치 정보를 쉽게 계산할 수 있다. 다음이 내가 사용한 예시이다. 

    const subMenuRect = subMenu[0].getBoundingClientRect();
    const isFitRight = bodyRect.right >= subMenuRect.right; // 화면 우측 여유공간 유무
    const isFitBottom = bodyRect.bottom >= subMenuRect.bottom; // 화면 하단여유공간 유무
    if( !isFitRight) subMenu.css({left : left - subMenuRect.width })
    if( !isFitBottom) subMenu.css({top : bodyRect.bottom - subMenuRect.height})
    subMenu.css( {visibility : 'visible'}); // subMenu 포지션 잡은 후 화면 노출

위의 코드는 동적으로 생서되는 컨텍스트메뉴의 2-depth 메뉴의 위치를 계산하는 로직이다. 즉, 생성되는 컨텍스트메뉴가 화면의 우측과 하단에서 가려지는 일이 없도록 위치를 재조정 한다. 

확실히 기존 offset을 활용하는 것 보다 훨씬 쉽고 간결하고, 또 직관적이다. 