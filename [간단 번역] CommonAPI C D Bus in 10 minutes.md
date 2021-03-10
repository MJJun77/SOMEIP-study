(( study 용으로 작업 중... ))

# CommonAPI C++ in 10 minutes (with D-Bus)
## 목차
* Step 1: Preparation / Prerequisites
* Step 2: Build the CommonAPI Runtime Library
* Step 3: Build the CommonAPI D-Bus Runtime Library
* Step 4: Write the Franca file and generate code
* Step 5: Write the client and the service application
* Step 6: Build and run

## Step 1: Preparation / Prerequisites
본 설명은 CommonAPI 3.1.10 으로 테스트되었고, standard Linux distribution (글쓴이는 Xubuntu 14.04 사용) 을 쓴다고 가정한다. code generator 는 java 8 runtime environment 가 필요하다는 걸 주지하자.

그리고 이 패키지들도 필요하다(debian/ubuntu names): cmake cmake-qt-gui libexpat-dev expat default-jre

## Step 2: Build the CommonAPI Runtime Library
CommonAPI source 를 다운받자 : 
```
$ git clone https://github.com/GENIVI/capicxx-core-runtime.git
Cloning into "capicxx-core-runtime"...
remote: Counting objects: 3959, done.
remote: Total 3959 (delta 0), reused 0 (delta 0), pack-reused 3959
Receiving objects: 100% (3959/3959), 3.11 MiB | 1.96 MiB/s, done.
Resolving deltas: 100% (2132/2132), done.
Checking connectivity... done.
```

이제 CommonAPI runtime library 를 다른 dependency 없이 빌드 가능하다:
```
$ cd capicxx-core-runtime/
$ ls
AUTHORS  cmake  CMakeLists.txt  CommonAPI.pc.in  commonapi.spec.in  docx  doxygen.in  include  INSTALL  LICENSE  README  src
<.>/capicxx-core-runtime$ mkdir build
<.>/capicxx-core-runtime$ cd build
<.>/capicxx-core-runtime/build$ cmake ..
-- The C compiler identification is GNU 4.8.2
-- The CXX compiler identification is GNU 4.8.2
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Project name: libcommonapi
-- This is CMake for Common API C++ Version 3.1.10.
-- CMAKE_INSTALL_PREFIX set to: /usr/local
-- BUILD_SHARED_LIBS is set to value: ON
-- MAX_LOG_LEVEL is set to value: DEBUG
-- USE_FILE is set to value: OFF
-- USE_CONSOLE is set to value: OFF
-- RPM packet version set to r0
-- Build type: Debug
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.26") 
-- checking for module 'automotive-dlt >= 2.11'
--   package 'automotive-dlt >= 2.11' not found
-- Found Doxygen: /usr/bin/doxygen (found version "1.8.6") 
-- asciidoc found
-- Configuring done
-- Generating done
-- Build files have been written to: ./build
<.>/capicxx-core-runtime/build$ make
Scanning dependencies of target CommonAPI
[ 10%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/Address.cpp.o
[ 20%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/ContainerUtils.cpp.o
[ 30%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/IniFileReader.cpp.o
[ 40%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/Logger.cpp.o
[ 50%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/LoggerImpl.cpp.o
[ 60%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/MainLoopContext.cpp.o
[ 70%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/Proxy.cpp.o
[ 80%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/ProxyManager.cpp.o
[ 90%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/Runtime.cpp.o
[100%] Building CXX object CMakeFiles/CommonAPI.dir/src/CommonAPI/Utils.cpp.o
Linking C shared library libCommonAPI.so
[100%] Built target CommonAPI
```

위에 보여진 내 출력 로그와 유사하게 출력되어야 한다.  Doxygen 이나 asciidoc 을 찾을 수 없다는 warning msg 는, documentation 작업이 필요한 게 아니면 문제 없다.  `automotive-dlt` package 는 DLT log msg 가 필요한 경우가 아니라면 필요없다.  위 로그에서 마지막 두 라인에서 보여지듯 libCommonAPI.so 이 빌드된 걸 확인하자.

:exclamation: CommonAPI runtime library 는 `make install` 호출을 통해 /usr/local (또는 installation prefix 를 사용해서 다른 경로 지정할 수도 있다.) 에 install 될 수 있다.  하지만 보통 이건 원하는 결과가 아니므로, 본 설명에서는 install 과정 없이 진행한다.

## Step 3: Build the CommonAPI D-Bus Runtime Library
(*주:이름이 비슷해서 헷갈릴 수 있는데) 이제 Step2. CommonAPI Runtime Library 다음으로는 CommonAPI-D-Bus (*주: 아래 보면 dbus-runtime 인데 용어가 혼동스럽다.) 를 다운받자:
```
$ git clone https://github.com/GENIVI/capicxx-dbus-runtime.git
Cloning into 'capicxx-dbus-runtime'...
remote: Counting objects: 4549, done.
remote: Total 4549 (delta 0), reused 0 (delta 0), pack-reused 4549
Receiving objects: 100% (4549/4549), 1.03 MiB | 941.00 KiB/s, done.
Resolving deltas: 100% (3404/3404), done.
Checking connectivity... done.
```
D-Bus runtime library 빌드 진행을 위해서는 패치 적용을 먼저 해야 한다.  CommonAPI-D-Bus 는 standard libdbus library 를 사용하지 않아 patched version 이 필요하다.(*주: 이게 왜 필요한지 궁금해서 이 git 에 들어가 보니, 다음과 같은 설명이 있다: CommonAPI-D-Bus needs some api functions of libdbus which are not available in actual libdbus versions.)
다시 말하자면 CommonAPI D-Bus runtime 빌드하기 전에 libdbus 의 patch / build 가 선행되어야 한다. 
freedesktop.org 에서 관리하는 actual libdbus library 를 다운받아 압축을 풀자: 
```
$ wget http://dbus.freedesktop.org/releases/dbus/dbus-1.10.10.tar.gz
$ tar -xzf dbus-1.10.10.tar.gz
$ cd dbus-1.10.10/
```
화면 출력 내용은 표시하지 않았다.  CommonAPI D-Bus source directory 에 패치를 적용하자.
capi-dbus-add-send-with-reply-set-notify.patch 적용 방법은 다음과 같다:
```
for i in ../capicxx-dbus-runtime/src/dbus-patches/*.patch; do patch -p1 < $i; done
patching file dbus/dbus-connection.c
Hunk #1 succeeded at 3500 (offset 18 lines).
patching file dbus/dbus-connection.h
```
:exclamation: 모든 패치가 정상 적용되었는지 확인이 필요하다. 나는 D-Bus 1.6.x, 1.8.x and 1.10.x. 로 확인해 보았다.  다른 버전에 대해서는 테스트를 안해 봐서, patch 가 정상 동작하는지 보장할 수 없다.

이제 autotools(*주: configure 명령어) 를 사용해서 libdbus 를 빌드하고, .libs-directory 에 .so library 가 정상 생성되었는지 확인하자:
```
<.>/dbus-1.10.10$> ./configure
<.>/dbus-1.10.10$> make
```
:exclamation: libdbus 를 autotools 로 빌드하길 추천한다.  안그러면 CommonAPI C++ D-Bus 빌드시 중요한 몇가지 파일들을 miss 할 수 있다.
```

이제 CommonAPI runtime library 빌드가 가능하다.  지금 우리는 CommonAPI and libdbus 의 uninstall 버전을 사용하고 있음을 상기하자.  아래와 같이 build directory 를 생성 후 작업하자:
```
<.>/capicxx-dbus-runtime$ mkdir build
<.>/capicxx-dbus-runtime$ cd build
```
다음 step 을 위해 D-Bus directory 를 PKG_CONFIG_PATH 환경 변수에 추가한 뒤, 늘 하던 것처럼 CMake / make 를 실행한다 :
```
<.>/capicxx-dbus-runtime/build$ export PKG_CONFIG_PATH="<my-dbus-path>/dbus-1.10.10"
<.>/capicxx-dbus-runtime/build$ cmake -DUSE_INSTALLED_COMMONAPI=OFF -DUSE_INSTALLED_DBUS=OFF ..
<.>/capicxx-dbus-runtime/build$ make
```
문제가 없다면 libCommonAPI-DBus.so 가 build dir 에 생성되었을 것이다. working dir path 에 space 가 없어야 할 것이다? (Make sure that there are no empty spaces in the path of your working directory.)

## Step 4: Write the Franca file and generate code
이 모든 준비를 마친 후 본격적으로 true CommonAPI application 작성을 시작하자.(정확히는 두 개). 하나는 sayHello method 를 offer 하는 service 이고, 하나는 이 method 를 호출하는 client 이다.
CommonAPI services 를 위해 offered interfaces 의 기술은 Franca IDL 로 작성된다. Franca IDL 라는 걸 모르더라도 걱정하지 말자. 쉽고 빠르게 배울 수 있다. (*주: ...라고 한다.  이건 쉽겠지만, 윈도우 포팅은 안되겠지...)

빈 dir 을 생성하고 HelloWorld.fidl 파일을 만들자.
_HelloWorld.fidl:_
```
$ mkdir project
$ cd project/
<.>/project$ mkdir fidl
<.>/project$ cd fidl
<.>/project/fidl$ vi HelloWorld.fidl
```
fidl 파일의 내용을 아래와 같이 작성한다.
```
package commonapi

interface HelloWorld {
  version {major 1 minor 0}
  method sayHello {
    in {
      String name
    }
    out {
      String message
    }
  }
}
```
준비되었다!  HelloWorld interface 를 인스턴스화하는 service 가 sayHello function 을 제공한다.

다음 step 은 code generation 이다.  이를 위해 code generator 가 필요하다.  tools repositories 에서 code generator 를 가져올 수 있다.  repository 에서 다운받아 빌드하면 된다.  이건 특별한 점은 없지만 일반적으로 사람들에게 익숙치 않은 Maven install 이 필요하다. (*주: Maven 은 Java 기반, 다양한 역할을 수행하는 project management tool 이라고 한다..)  그래서 가장 빠른 방법은 github 에서 이미 빌드된 code generator binary 를 다운받는 것이다.  네 개의 버전이 있다(*주: 윈도우용도 있다 !!) : Linux, Windows and 64bit variants.  `uname -m`으로 자신의 platform 이 32bit 인지 64bit 인지 확인할 수 있다.  이후 설명은 당신의 시스템이 32bit Linux 라고 가정한다.

압축 파일을 dir 로 복사하여 (예: cgen) 압축을 푼다. cgen/commonapi-generator 폴더와 cgen/commonapi_dbus_generator 폴더 안에 code generator 실행 파일이 있을 것이다.

드디어 아래와 같이 코드를 generate 할 단계이다. (CommonAPI code with the commonapi-generator and CommonAPI D-Bus code with the commonapi-dbus-generator; SOME/IP code generator 는 이 시점에서 필요 없다.  아래 명령은 cgen directory 가 project dir 에 있다고 가정한 것이다):
```
<.>/project$ ./cgen/commonapi-generator/commonapi-generator-linux-x86 -sk ./fidl/HelloWorld.fidl
<.>/project$ ./cgen/commonapi_dbus_generator/commonapi-dbus-generator-linux-x86 ./fidl/HelloWorld.fidl
```

문제 없이 정상 실횅되었다면 생성된 코드는 src-gen dir 에 생성될 것이다.옵션 _-sk_ 는 service 의 interface instance 를 default 구현을 generate 한다.

## Step 5: Write the client and the service application
이제 Hello World app 을 작성할 수 있다.  _src_ 와 _build_ 를 만들고 _src_ 로 이동한다.  여기서 4개의 파일을 작성한다: 1) client code _HelloWorldClient.cpp_; 2) service main function _HelloWorldService.cpp_;  3,4) generate 된 stub skeleton 의 구현 파일 (header and source, 여기서는 각각 _HelloWorldStubImpl.hpp_ 과 _HelloWorldStubImpl.cpp_ 로 이름을 정했다.)

client 부터 작성해 보자. _HelloWorldClient.cpp_ 를 다음과 같이 작성하자:
```
// HelloWorldClient.cpp
#include <iostream>
#include <string>
#include <unistd.h>
#include <CommonAPI/CommonAPI.hpp>
#include <v1/commonapi/HelloWorldProxy.hpp>

using namespace v1_0::commonapi;

int main() {
    std::shared_ptr < CommonAPI::Runtime > runtime = CommonAPI::Runtime::get();
    std::shared_ptr<HelloWorldProxy<>> myProxy =
    	runtime->buildProxy<HelloWorldProxy>("local", "test");

    std::cout << "Checking availability!" << std::endl;
    while (!myProxy->isAvailable())
        usleep(10);
    std::cout << "Available..." << std::endl;

    CommonAPI::CallStatus callStatus;
    std::string returnMessage;
    myProxy->sayHello("Bob", callStatus, returnMessage);
    std::cout << "Got message: '" << returnMessage << "'\n";
    return 0;
}
```

CommonAPI application 작성 시 generic runtime object 의 pointer 를 우선 얻어야 한다.  runtime 은 proxies and stubs 생성을 위해 필요하다.  client app 은 service app 내 interface instance 의 function 을 호출해야 한다.  이 호출을 위해 client 에서는 이 interface 를 위한 proxi 를 빌드해야 한다.  

interface name 은 _buildProxy()_ 함수의 템플릿 인자를 구성한다; 그리고 우리는 certain instance 를 위해 proxy 를 빌드하였다; instance name 은 _buildProxy()_ 의 두번째 param 이다.  원론적인 얘기를 하자면 1st param 을 이용하여 다른 domain 에 있는 instance 간 instance 를 구분할 수 있지만, 지금 논의하기엔 앞선 내용이고 우선은 domain 을 항상 "local" 로 두자.

proxi 는 _isAvailable()_ 함수를 제공한다; service 를 먼저 시작한다면 _isAvailable()_ 은 항상 true 를 리턴한다.  물론 service 가 available 할 때 불리는 callback 함수를 등록할 수도 있다.  여기서는 간단히 하기 위해 사용하진 않았다.

다음은 fidl 파일에 정의한 _sayHello()_ 을 호출해 보자.  이 함수는 1개의 in-parameter (string) 와 1개의 out-parameter (also string) 를 가진다.  _HelloWorldPorxy.hpp_ 를 리뷰해 보면 _sayHello()_ 를 어떻게 호출해야 하는지 알 수 있다.  여기서 중요한 점은 이 함수의 sync 버전(여기서 사용하는) 이나 async 버전(_sayHelloAsync()_, 약간 더 복잡) 을 사용할 수 있다는 것이다.

함수 호출 후에는 호출 결과의 성공/실패 여부에 대한 정보를 가지고 있는 CallStatus 라는 것을 리턴한다.  여기서는 예제를 간단히 하기 위해 결과 값을 체크하지 않고 항상 성공적이라고 가정한다.

이제 _HelloWorldService.cpp_ 의 파일 이름으로 service 를 작성해 보자:
```
// HelloWorldService.cpp
#include <iostream>
#include <thread>
#include <CommonAPI/CommonAPI.hpp>
#include "HelloWorldStubImpl.hpp"

using namespace std;

int main() {
    std::shared_ptr<CommonAPI::Runtime> runtime = CommonAPI::Runtime::get();
    std::shared_ptr<HelloWorldStubImpl> myService =
    	std::make_shared<HelloWorldStubImpl>();
    runtime->registerService("local", "test", myService);
    std::cout << "Successfully Registered Service!" << std::endl;

    while (true) {
        std::cout << "Waiting for calls... (Abort with CTRL+C)" << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(30));
    }
    return 0;
 }
```

main 함수는 client 보다도 간단한데, interface 함수의 구현부는 stub impl. 에 작성되어야 하기 때문이다.
client 와 마찬가지로, runtime env 에 대한 pointer 가 필요하다.  그리고 stub 의 구현을 인스턴스화한 뒤 이 interface instance 를 _registerService()_ 를 호출하여 등록한다.  service 는 kill 되지 않는 한 무한 루프로 동작하면서 function call 에 응답해야 한다; 즉 함수 마지막에 while loop 가 필요하다.

마지막으로 stub 의 구현이 필요하다.  이 작업은 stub-default impl. 을 상속하여 stub-impl. 을 구현하는 작업이다.  헤더 파일은 다음과 같다:
```
// HelloWorldStubImpl.hpp
#ifndef HELLOWORLDSTUBIMPL_H_
#define HELLOWORLDSTUBIMPL_H_
#include <CommonAPI/CommonAPI.hpp>
#include <v1/commonapi/HelloWorldStubDefault.hpp>

class HelloWorldStubImpl: public v1_0::commonapi::HelloWorldStubDefault {
public:
    HelloWorldStubImpl();
    virtual ~HelloWorldStubImpl();
    virtual void sayHello(const std::shared_ptr<CommonAPI::ClientId> _client,
    	std::string _name, sayHelloReply_t _return);
};
#endif /* HELLOWORLDSTUBIMPL_H_ */
```
cpp file 에 작성한 구현부는 다음과 같다:
```
// HelloWorldStubImpl.cpp
#include "HelloWorldStubImpl.hpp"

HelloWorldStubImpl::HelloWorldStubImpl() { }
HelloWorldStubImpl::~HelloWorldStubImpl() { }

void HelloWorldStubImpl::sayHello(const std::shared_ptr<CommonAPI::ClientId> _client,
	std::string _name, sayHelloReply_t _reply) {
	    std::stringstream messageStream;
	    messageStream << "Hello " << _name << "!";
	    std::cout << "sayHello('" << _name << "'): '" << messageStream.str() << "'\n";

    _reply(messageStream.str());
};
```

If the function sayHello is called it gets the name (which is supposed to be the name of the developer of this application) and returns it with an added "Hello" in front. The return parameter is not directly the string as it is defined in the fidl-file; it is a standard function object with the return parameters as in-parameters. The reason for this is to provide the possibility to answer not synchronously in the implementation of this function but to delegate the answer to a different thread.
