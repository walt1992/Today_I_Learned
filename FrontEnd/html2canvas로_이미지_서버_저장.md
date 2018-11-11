# html2canvas로 화면 캡쳐하기 & 서버 전송하기

간혹 현재 보고있는 화면을 캡쳐해 이미지로 저장하는 기능을 구현하고 싶을 경우가 생긴다. [html2canvas](https://html2canvas.hertzen.com/)는 설정한 화면의 html태그를 canvas형태로 전환하여 이미지로 저장할 수 있도록 도와주는 라이브러리이다. 경우에 따라서 매우 활용범위가 넓은 라이브러리라고 생각하는 만큼, 기능을 구현함에 있어 어려움이 있었던 부분에 대해 Trouble shooting의 형태로 정리해본다. 

먼저 라이브러리의 기본적인 사용법은 어렵지 않다. 

    html2canvas(document.body).then(function(canvas) {
     // addtinal code...
    });


위의 코드처럼 설정해 줄 경우, promose로 return되는 canvas객체를 원하는 용도에 맞게 활용하면 된다. 예를들어 캡쳐된 이미지를 다운받을 수도 있으며, 서버단에 보내 저장할 수 도있다. 나의 경우 두가지를 모두 구현했어야만 했는데, **다운로드** 기능의 경우는 [stackoverflow](https://stackoverflow.com/questions/41165865/download-html2canvas-image-to-desktop)를 통해 비교적 쉽게 구현이 가능했다. 

문제는 생성한 canvas 객체를 서버에 이미지로 저장하는 것이었다. 

***
## canvas 객체 서버에 저장하기 

먼저 html2canvas함수를 통해 canvas객체를 return 받았다고 가정한다.


### ISSUE.1 canvas객체 blob로 전환하기

 **Canvas객체는 toDataUrl()이라는 함수를 통해 Base64로 엔코딩**할 수 있다. 이로써 하나의 이미지 URL을 생성할 수 있는데, 이것이 완전한 이미지 객체라고 할 수는 없다. (자세한 부분은 다시 공부해봐야 한다) 이를 해결하기 위해 Base64로 엔코딩 되어 있는 이미지를 Blob로 변환시켜 FormData에 넣어주면 이미지 객체를 서버에 전송할 준비가 된다고 할 수있다.  

이를 위해 검색을 통해 **Base64를 Blob로 전환하는 코드** 를 가져왔다. 

 

        function b64toBlob( b64Data, contentType, sliceSize) { 

            contentType = contentType || ''; 
            sliceSize = sliceSize || 512; 
            var byteCharacters = atob(b64Data); 
            var byteArrays = []; 

            for ( var offset = 0; offset < byteCharacters.length; offset += sliceSize) { 

              var slice = byteCharacters.slice( offset, offset + sliceSize); 
              var byteNumbers = new Array( slice.length); 
              for ( var i = 0; i < slice.length; i++) { 
                byteNumbers[i] = slice.charCodeAt(i); 
              } 

 

              var byteArray = new Uint8Array( byteNumbers); 
              byteArrays.push( byteArray); 

            } 

            var blob = new Blob( byteArrays, {type: contentType}); 
            return blob; 
        }         

 

역시나, 자세한 내용은 다시 공부해야한다. ([blob란? ](http://iamawebdeveloper.tistory.com/106 ) )



이 함수에 Base64를 파라미터로 전달하면 blob로 전환되어 리턴이되고, 그 값을  

 
     formdata.append( "thumbnailImg", blob,"testfile.png"); 

처럼 넣어주면 파라미터로 서버에 전송할 준비는 끝난다.  


## ISSUE 2. 캡쳐된 이미지를 파일로 만들어 서버에 $http 통신을 통해 전달하기 

 

Angular와 Spring boot에 아직 익숙하지가 않은 터라 많은 어려움이 있었다. Server에서 받는 

원리는 기존 Spring과 동일하나, 파라미터로 보내는 것이 매우 힘들었는데,  

전달하고자 하는 파라미터를 FromData() 객체에 담아 보내니 제대로 보내졌다.  

이때, 파라미터에는 이미지 파일 또한 포함되어 있었는데, 이는 config설정을 통해 해결하였다.  

 

   

                var formdata = new FormData(); 

                formdata.append( "thumbnailImg", blob,"testfile.png"); 
                formdata.append( "dashboardid", $scope.dashboard.id); 
                 
                var config = { 
                    transformRequest: angular.identity, 
                    headers:{'content-type':undefined} 
                } 
                
                $http.post( "app/dashboard/screenshot", formdata, config).then(function(secc){ 
                    alert("succ"); 
                },function(err){ 
                    alert("err");
                }    

***
이렇게 서버에 정상적으로 전달된 이미지의 객체는 `MultipartFile`의 형태로 객체를 전달받을 수 있다. 이후의 절차는 기본 파일 업로드와 동일하다. 