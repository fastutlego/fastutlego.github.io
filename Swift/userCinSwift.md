Swift Framework 에서 C/C++ 코드 사용하기
===================================

### Swift에서 C/C++ 사용법 정리

앱프로젝트에서 C나 C++파일을 사용시 bridge-header를 설정해서 해당 파일에 헤더(.h)를 import 후 처리하면 되는데, framework에서는 bridge-header를 설정 할 수 없음.
따라서 프레임웍에서 C파일을 사용하는 방법을 메모라이징함.

***

1. 프로젝트의 설정 - Build Parses 에서 Link Binary With Libarary에 C 모듈을 import함.

2. C/C++을 Objective-C 로 사용할 수 있도록 Wrapper Class 작성
(C - .m, C++ - .mm)

3. 프로젝트의 설정 - Headers에 C 모듈의 헤더와 래퍼클래스의 헤더를 첨부함

4. 프레임웍에 module.modulemap 파일 작성 후, 아래와 같이 작성
프레임웍은 TestFramework, C의 Objective-C Wrapper Class는 WrapperClass 파일로 명명함.

moudle.modulemap

    framework module TestFramework {
        umbrella header "TestFramework.h"

        header "WrapperClass.h"

        export *
        module * { export * }
    }

5. 이후 framework에서 Wrapper Class를 호출 할 수 있음.


Reference article: 
<https://medium.com/allatoneplace/challenges-building-a-swift-framework-d882867c97f9>
