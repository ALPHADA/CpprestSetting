# CPPRESTSDK SETTING FOR OSX

MAC OS 환경에서 C++ rest를 사용하기 위한 SDK 붙이기

https://github.com/microsoft/cpprestsdk

CMAKE 사용

각종 설정 완료 이후 작성한 문서이기 때문에 일부 환경이 달라진 부분이 있을 수 있음.


### SDK 설치
cpprestsdk 사용을 위해서는 boost와 openssl 등 다양한 라이브러리들이 필요하다.     
cpprestsdk 설치시 같이 설치가 된다고 하는데 만약 되어있지 않다면 추가로 설치가 필요하다.        
sdk 설치 과정은 아래 문서를 참고할것     

공식 : https://github.com/microsoft/cpprestsdk

OSX 가이드 :  https://github.com/Microsoft/cpprestsdk/wiki/How-to-build-for-Mac-OS-X

```
$ brew update

# 각각 설치
$ brew install cpprestsdk
$ brew install boost
$ brew install openssl
...

# 한번에
$ brew install cmake git boost openssl ninja
```


아래 폴더에 설치한 라이브러리가 생성되었는지 확인한다.
```
/usr/local/include/
/usr/local/lib/
/usr/local/Cellar/

# openssl 은 아래 dir 에 있음
/usr/local/opt
```
코드에서 헤더를 참조할때 `/usr/local/include/` 폴더를 확인하기 때문에 해당 폴더에 include 할 라이브러리 헤더들이 제대로 들어가있는지 확인한다.


<details>
<summary>선택사항</summary>

ssl 안에 있는 crypto와 ssl을 linking 해주어야 하는데 mac에서는 직접 찾을수가 없어서       
쉽게 찾을 수 있도록 바로가기 링크를 추가해준다.     
  cmakelist.txt 에서 해당 경로를 직접 연결도 가능

```
// ssl 설치 링크
ln -s /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib /usr/local/lib/
ln -s /usr/local/opt/openssl/lib/libssl.1.0.0.dylib /usr/local/lib/
ln -s /usr/local/Cellar/openssl/1.0.2j/bin/openssl /usr/local/bin/openssl
```

</details>



### vscode 설정

#### C/C++ Extension 설치
Extension 탭에서 C/C++을 설치하면 우측 상단에 Build/Debug 아이콘이 생성되며 이를 클릭하면 Compiler 세팅 옵션을 선택 할 수 있다.

기본 Extension만을 이용하여 Hello World를 해보면 빌드 시 ./vscode/tasks.json이 생성되며 이 파일이 빌드 설정 파일이다.

> 참고로 C++ Extension을 설치하면 자동으로 formatter도 설치됨


#### CMake Extension 설치

cmake 관련 기능을 쉽게 사용할 수 있게 해주는 CMake Extension을 설치한다.

CMake로 Extension을 검색하면 여러가지가 뜨는데 Microsoft에서 배포한 CMake Tools를 설치하면 CMake가 동시에 설치된다.     

기본적으로 CMakeLists.txt파일이 있어야 하지만 없을 경우 'F1' > 'CMake: Quick Start' 를 실행하면 대화상자를 통해 생성이 가능하다.      

F1을 눌러 CMake을 검색하면 CMake Extension 기능을 찾을 수 있으며 주로 사용할 기능은 'CMake: Build', 'CMake: Debug', 'CMake: Run Without Debugging'이다.      

'Build' 실행 후 'Run Without Debugging' 로 c++ 프로그램을 실행한다.        


### CMAKELISTS.txt

아래는 테스트가 완료된 파일이다.    
수정이 필요한 내용이 있다면 가감하여 사용하면 된다.        

```
cmake_minimum_required(VERSION 3.15)
project(basic)

set(CMAKE_CXX_STANDARD 17)
if (APPLE)
    set(OPENSSL_ROOT_DIR ${OPENSSL_ROOT_DIR} /usr/local/opt/openssl/) # openssl path 입력
    set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.dylib CACHE FILEPATH "" FORCE)
    set(OPENSSL_SSL_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libssl.dylib CACHE FILEPATH "" FORCE)
    set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.dylib CACHE FILEPATH "" FORCE)
endif()

find_package(Boost COMPONENTS regex system filesystem REQUIRED)
find_package(cpprestsdk REQUIRED)
find_package(OpenSSL REQUIRED)

include_directories("/usr/local/lib")

add_definitions(-std=c++17)

add_executable(basic main.cpp)
target_link_libraries(basic PRIVATE 
                            ${Boost_FILESYSTEM_LIBRARY}
                            ${Boost_SYSTEM_LIBRARY}
                            ${Boost_REGEX_LIBRARY}
                            ${OpenSSL_LIBRARIES}
                            ${OPENSSL_CRYPTO_LIBRARY} 
                            cpprestsdk::cpprest)
```

### 기타 설정

'C/C++ Extension > Edit Configurations' UI 편집 결과

'.vscode' 폴더 하위에 'c_cpp_properties.json' 생성하여 수정할 수 있음.

```
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/local/include"
            ],
            "defines": [],
            "macFrameworkPath": [
                "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks"
            ],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c17",
            "cppStandard": "c++98",
            "intelliSenseMode": "macos-clang-x64",
            "configurationProvider": "ms-vscode.cmake-tools"
        }
    ],
    "version": 4
}
```

작동하는 코드는 위와 같다.    

그리고 하단의 CMake Tools 'kit for printer' 으로는 'CLang 13.0.0 x86_64-apple-darwin20.6.0'을 사용하였다.


### 덧 


#### locale_t issue

컴파일러가 잘못된 케이스로 'GCC -> CLang' 변경해주니 작동하였다.     
다만 리눅스 환경에서는 gcc로 동작했던 터라 왜 Mac os 에서는 clang 으로 동작하는지는 잘 모르겠다.


#### 'U' complict issue

cpprest가 제공하는 'U' 매크로는 플랫폼 유형의 문자열 또는 문자를 만드는데 사용하는데 간혹 다른 라이브러리가 동일한 이름을 사용하여 오류가 발생할때가 있다.

이 오류는 cpprest 내부적으로 boost 의 특정 파일을 물고 있는 경우에도 발생하는데(...) 이미 이슈로 제기된 바가 있다.    

> 수정도 검토하였지만 기존 사용자들 코드를 커버할 수 없는 문제가 있어 네이밍은 유지하기로 한듯하다.   
> https://github.com/microsoft/cpprestsdk/issues/1214


cpprest에서는 해결 방안으로 `_TURN_OFF_PLATFORM_STRING`를 제공하는데 이를 사용하면 'U' 를 정의하지 않게 된다.     


```
# cpprestsdk 설명

#ifndef _TURN_OFF_PLATFORM_STRING
// The 'U' macro can be used to create a string or character literal of the platform type, i.e. utility::char_t.
// If you are using a library causing conflicts with 'U' macro, it can be turned off by defining the macro
// '_TURN_OFF_PLATFORM_STRING' before including the C++ REST SDK header files, and e.g. use '_XPLATSTR' instead.
#define U(x) _XPLATSTR(x)
```


```
# 사용법 

#define _TURN_OFF_PLATFORM_STRING
#include <iostream>
#include <cpprest/http_listener.h>
```

보통은 문제가 있는 'U'를 사용하지 않는 방법을 채택하는듯하다.    

```
U("something") -> utility::conversions::to_string_t("something")
```

하지만 나의 경우 이미 U 템플릿을 사용중인 프로젝트여서 적용하기 어려웠다.

그래서 참조하고 있는 boost의 템플릿명을 바꾸는 것으로 대체하였다.    
충돌이 발생한 파일이 하나여서 해당 파일만 수정하면 됐기에 가능한 방법이었다.    

수정한 경로는 아래와 같다..

> /usr/local/Cellar/boost/1.81.0_1/include/boost/multiprecision/cpp_int/misc.hpp
> ```
> cpp_int_backend<MinBits1, MaxBits1, SignType1, Checked1, Allocator1> U(storage, temp_size);     
> -> cpp_int_backend<MinBits1, MaxBits1, SignType1, Checked1, Allocator1> UU(storage, temp_size);     
> 이하 연관된 코드 수정
> ```


추천하는 방법은 아니다.     
어쩔 수 없을때 사용하도록 하자.     


