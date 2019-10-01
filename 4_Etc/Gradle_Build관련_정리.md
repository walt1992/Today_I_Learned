# [Gradle] sourceSets과 processResoures Task

### **sourceSets** 
빌드된 프로젝트에서 컴파일되고 실행되기 위해 필요한 자바소스와 기타 리소스 셋.
프로젝트 빌드를 위해서 따로 새로운 프로젝트를 만들 필요 없이 기존 프로젝트에서 선택적으로 소스를 선택해 빌드에 반영하기 위한 목적이다. 

 > Java projects typically include resources other than source files, such as properties files, that may need processing — for example by replacing tokens within the files — and packaging within the final JAR.


### processResources task

Gradle의 Build에 포함되는 기본 테스크 중 하나로, Build에 반영될 Resource를 $buildDir에 복사하는 테스크를 기본적으로 수행한다.
이 테스크에서 Resource의 빌드시에 필요한 작업을 수행할 수 있다. 

    processResources {
        dependsOn minifyJs
        doLast {
            copy {
                from "${buildDir}/resources/main/META-INF/resources/html/scripts.build.html"
                into "${buildDir}/resources/main/META-INF/resources/html"
                rename 'scripts.build.html', 'scripts.html'
            }
        }
    }




 