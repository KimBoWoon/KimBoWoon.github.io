---
title: "[Android] JNI 사용하기"
date: 2018-03-31 01:23:45
categories:
- Android
---

# 예제 코드
* https://github.com/KimBoWoon/Android-Practice/tree/master/JNI

# 오류 수정
* jar 파일을 만들 때 gradle 명령어를 조심히 사용하자
* jar 파일을 만들고 나서 확인해보니 ```libs.com.android.~``` 로 시작해서 클래스를 못찾은거 같다
* 인터넷에서 본 exportJar 라는 gradle task가 안된다.
* ```build/intermediates/bundle```라는 폴더가 생성이 안되서 command line으로 실행

# JNI(Java Native Interface) 란
* 자바가 다른 언어들과 상호작용을 할 수 있도록 도와주는 인터페이스
* 자바로 구현하기에는 성능이 많이 떨어진다고 생각되면 JNI를 사용해 구현할 수 있다
* 다른 하드웨어를 제어하기위해 사용하기도 함
* C/C++를 사용

# Example
1. 안드로이드 Empty Activity 프로젝트를 만든다.
2. ```sdk tools```에서 CMake와 LLDB, NDK를 설치한다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/sdk-setting.png)

3. ```File -> New Module```를 선택한다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/new-module.png)

4. android library를 선택한다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/android-library.png)

5. android library를 생성하고 난 뒤에 ```File -> Setting```에 ```Tool -> ExternalTool```을 다음과 같이 설정한다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/setting.png)
	* javah
        ```text
        program : javah.exe
        argument : -classpath "$Classpath$" -v -jni $FileClass$
        working directory : $ProjectFileDir$\<라이브러리 이름>\src\main\jni
        ```
    * NDK-Build
        ```text
        program : ndk-build.cmd
        working directory : $ProjectFileDir$\<라이브러리 이름>\src\main
        ```
    * NDK-Clean
        ```text
        program : ndk-build.cmd
        argument : clean
        working directory : $ProjectFileDir$\<라이브러리 이름>\src\main
        ```
	* ```javah.exe``` : java class 대로 Header File을 만들어준다. 설치된 jdk에 bin에 존재한다.
	* ```ndk-build.cmd``` : C/C++로 작성된 코드를 인터페이스에 맞게 build 시켜준다. 설치된 android-sdk에 ndk-bundle에 존재한다.(2번에 NDK를 설치하면 생성됨)
	* 각각의 환경변수가 설정되어 있지 않으면 절대경로를 넣어 설정한다.

6. ```src -> main``` 아래에 jni폴더를 생성한다. NDK-Build를 진행하게 되면 여기에 ```*.so```파일이 생성된다.
7. 이제 자신의 패키지명 아래에 java class를 생성한다. class에는 메소드만 선언해주고 나중에 ```javah.exe```를 사용하면 jni폴더 아래에 헤더 파일이 생성된다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/create-class.png)
    * ```System.loadLibrary()```은 전달인자와 같은 이름의 모듈을 로드시킨다는 의미
    * 나중에 mk 파일에서 LOCAL_MODULE의 이름을 적어주면 된다.

8. class를 만들고 난 뒤에 ```javah.exe```를 실행한다. 단, 먼저 빌드가 되어 있어야 순조롭게 진행된다.

    ![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/make-project.png)

    ![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/error-javah.png)
    * 실패 했을 때

    ![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/success-javah.png)
    * 성공 했을 때

9. 지금은 참조를 제대로 못하고 있지만 Link를 해주면 정상적으로 참조를 하게 된다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/c-header.PNG)

10. Android.mk와 Application.mk 파일을 만들어 준다.
	* LOCAL_MODULE := (원하는 module 이름)
	* FILES := (나중에 만들 cpp file 이름)
	* APP_ABI := all : android cpu가 지원하는 모든 ABI에 대해 ```*.so``` 파일을 만든다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/create-mk-file.PNG)

11. ```Link C++ Project With Gradle```을 진행한다.

    ![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/link.png)

    ![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/mk-build-gradle.PNG)
    * 위와 같이 ```build.gradle```에 mk 파일의 경로가 추가된걸 볼 수 있다.

12. 이제 jni 폴더 아래에 cpp 파일을 만들어 원하는대로 구현한다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/make-csource.png)

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/create-cpp.PNG)

13. ```ndk-build```를 실행한다. 실행하면 다음과 같이 모든 ABI에 대해 ```*.so``` 파일이 만들어진것을 볼수있다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/excute-ndk-build.png)

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/after-excute-ndk-build.PNG)

14. 만들어둔 java class에 대해서는 jar 파일을 만들어야 연동이 가능하다. jar파일이 없으면 해당하는 메소드를 찾을 수 없다고 오류가 발생한다. ```build.gradle```에 다음과 같은 task를 넣는다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/export-jar.PNG)

15. gradle 창에 other으로 가면 exportJar가 생성된걸 볼 수 있고 실행하면 jar파일이 만들어진것을 볼 수 있다.

	![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/export-jar-gradle.PNG)

16. 이제 이걸 사용하려면 원하는 프로젝트에 ```app -> libs```에 jar 파일을 넣고 ```src -> main -> jniLibs```에 ```<abi name>/*.so```를 넣고 ```build.gradle```에 ```implementation files('libs/(만들어둔 jar 파일 이름))```을 넣으면 사용이 가능하다.

# 실행 결과
![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/complate.png)

* x와 y를 더한 값을 반환하는 JNI Library를 만들었고 Toast로 결과값을 확인

# 참조
* https://developer.android.com/studio/projects/android-library.html?hl=ko
* http://duzi077.tistory.com/134
