이곳에는 유용한 자바스크립트 라이브러리를 정리한다. 

그때그때 새롭게 알게 될 때마다 정리해야지. 

***





+ ## [HTML2CANVAS](https://html2canvas.hertzen.com/)

지정한 html의 DOM Element들을 canavas의 형태로 변환시켜주는 라이브러리이다. 이를 통해 [이미지를 다운로드 및 저장](../html2canvas로_이미지_서버_저장.md)할 수 있다.

> **Note** : ***복제된 canvas는 원본 html의 css를 온전히 담지 못하는 경우가 발생한다. 이런 경우는 대부분 CSS가 Inline이 아닌 다른 방식으로 적용되어 있기 때문이다.***


+ ## [pagemap](https://larsjung.de/pagemap/)

현재 페이지의 html구성 요소들을 지정해 미니맵의 형태로 보여준다. 화면에서 지정된 html태그의 childeNode들을 트래킹하여 미니맵을 구성하며, childNode 중에서도 원하는 태그들만 선택적으로 적용할 수 있다. 