(( site : https://github.com/GENIVI/capicxx-someip-tools/wiki/CommonAPI-C---SomeIP-in-10-minutes ))
내맘대로 약어 : desc == description, comm == communication, net == network, app == application, env == environment, def == definition, gen == generate, gentor == generator, func == function, repo == repository ... and so on

# CommonAPI C++ in 10 minutes (with SOME/IP)

# Table of contents
* Step 1 & 2: Preparation / Prerequisites
* Step 3: Build the CommonAPI SOME/IP Runtime Library
* Step 4: Write the Franca file and generate code
* Step 5: Write the client and the service application
* Step 6: Build and run

## Step 1 & 2: Preparation / Prerequisites
본 설명은 CommonAPI 3.1.10 로 테스트하였고 standard Linux distribution(글쓴이 환경: Xubuntu 14.04) 을 가정하여 설명하였다.  code gentor 는 java 8 runtime env 가 필요하다. 
"CommonAPI C++ D-Bus in 10 minutes" 를 읽어 보지 않았다면, CommonAPI runtime lib 빌드를 위해 해당 페이지의 step 2를 참조하라. (*주: CommonAPI runtime lib 빌드 과정이 공통이라는 얘기일 뿐이지, CommonAPI for SOME/IP 를 위해 CommonAPI for D-Bus 전체를 읽어볼 필요는 없다.)

## Step 3: Build the CommonAPI SOME/IP Runtime Library**
Start again with cloning the source code of CommonAPI-SomeIP:
```
$ git clone https://github.com/GENIVI/capicxx-someip-runtime.git
```
_CommonAPI C++ D-Bus_ 에 D-Bus library(libdbus) 가 필요한 것처럼, SOME/IP binding build 는 SOME/IP core library(vsomeip) 가 필요하다.
vsomeip lib 의 source code는 github 에서 받을 수 있다. repo 에서 clone 하면 된다:
```
$ git clone https://github.com/GENIVI/vsomeip.git
```
계속 하기 전에, 사전 준비 사항을 확인해야 한다.  vsomeip 는 some core network func 를 위해 standard cross-platform C++ library Boost.Asio 를 사용한다.
vsomeip 를 컴파일하기 전에 Boost (1.55 - 1.65) 가 설치되었는지 확인해야 한다.

http://www.boost.org/doc/libs/1_59_0/more/getting_started/unix-variants.html 의 설치 가이드를 확인해도 되고, ubuntu 기반이라면 package manager APT 를 이용하면 된다. (README on https://github.com/GENIVI/vsomeip).
Boost 가 성공적으로 설치되었다면 어려움 없이 vsomeip 를 빌드할 수 있다:
```
$ cd vsomeip
<.>/vsomeip$ mkdir build
<.>/vsomeip$ cd build
<.>/vsomeip/build$ cmake ..
<.>/vsomeip/build$ make
```

내 환경에서는 CMake 가 아래와 같은 Boost 관련 로그를 출력한다:
```
-- Boost version: 1.55.0
-- Found the following Boost libraries:
--   system
--   thread
--   log
```

이렇게 해도 동작하지만, 향후 있을 수도 있는 special problem 을 회피하기 위해 다음과 같은 parameter 를 추가하여 CMake 를 빌드하기를 추천한다:
```
<.>/vsomeip/build$ cmake -DENABLE_SIGNAL_HANDLING=1 -DDIAGNOSIS_ADDRESS=0x10 ..
```

`ENABLE_SIGNAL_HANDLING=1`로 vsomeip 의 signal handling(SIGINT/SIGTERM) 이 enable 된다; ctrl-c로 vsomeip app 을 abort 시킬 수 있다.
두번째로 쓰인 DIAGNOSIS_ADDRESS는 SOME/IP client ID의 첫번째 바이트 값(지금 시점에서 뭔지 잘 모르겠다면 그냥 무시해도 된다)을 지정한다.
일단은 client ID 는 해당 시스템에서 unique 한 값이며, SOME/IP message header 에서 사용된다는 정도만 알면 된다. 
만약 local로 SOME/IP 통신을 하기 위한 용도라면(*주: 즉 외부 통신 없이 한 device 내에서만의 통신) 이 옵션은 굳이 필요가 없다.
 If you only intend to communicate locally via SOME/IP this is not necessary.

이제 "CommonAPI C++ SOME/IP runtime library" 가 성공적으로 빌드되어야 한다:
```
$ cd capicxx-someip-runtime
<.>/capicxx-someip-runtime$ mkdir build
<.>/capicxx-someip-runtime$ cd build
<.>/capicxx-someip-runtime/build$ cmake -DUSE_INSTALLED_COMMONAPI=OFF ..
<.>/capicxx-someip-runtime/build$ make
```

D-Bus 예에서처럼, CommonAPI C++ SOME/IP library 또한 설치하지 않을 것이다.  libCommonAPI-SomeIP.so 는 build dir 에 생성되어 있을 것이다.

## Step 4: Write the Franca file and generate code
다시 한 번 "CommonAPI C++ D-Bus in 10 minutes"의 Step 4 를 참조하자:
* subdir 생성 후 fidl 파일을 생성한다.
* 이 subdir 로 이동한다.
* D-Bus tutorial 에서 설명한 것처럼 똑같이 _HelloWorld.fidl_ 를 생성한다.

D-BUS 의 예와 한가지 다른 점은 SOME/IP Spec 은 추가적으로 service 와 method ID 의 definition 이 필요하다는 점이다.

Franca IDL 은, Franca deployment files라 불리는, IPC framework specific parameter 를 추가할 수 있는 방법을 제공한다.
이 deployment file 은 .fdepl (deployment param file) 파일이고, Franca-like syntax를 가진다.
fdepl 파일이 제공해야 하는 정확한 내용이 fdepl file 에 기술되어 있어야 한다.
CommonAPI C++ SOME/IP spec 을 사용하여 작성된 deployment spec file 을 SOME/IP code generator project 에서 찾아볼 수 있다.
complete tools project 를 확인해 보고 싶지 않다면, 이 사이트를 참조해 보자: https://github.com/GENIVI/capicxx-someip-tools; 
`CommonAPI-SOMEIP_deployment_spec.fdepl` 파일을 _org.genivi.commonapi.someip/deployment_ 폴더에서 확인할 수 있다.

이 deployment spec 에 기반하여 deployment parameter file 을 작성할 수 있다.
```
import "platform:/plugin/org.genivi.commonapi.someip/deployment/CommonAPI-SOMEIP_deployment_spec.fdepl"
import "HelloWorld.fidl"

define org.genivi.commonapi.someip.deployment for interface commonapi.HelloWorld {
	SomeIpServiceID = 4660

	method sayHello {
		SomeIpMethodID = 123
	}
}

define org.genivi.commonapi.someip.deployment for provider MyService {
	instance commonapi.HelloWorld {
		InstanceId = "test"
		SomeIpInstanceID = 22136
	}
}
```
SOME/IP deployment spec은 필수 param 이 있다; binding 독립적인 CommonAPI deployment spec 을 impart 해야 한다. 그러면 instance deployment 에서 InstanceID param 을 찾을 수 있다.
다음으로 SOME/IP code gentor 가 필요하다.  다른code gentor 와 마찬가지로 github에서 다운받을 수 있다.
CommonAPI C++ D-Bus code gentor 와 마찬가지로, SOME/IP gentor 를 실행하면 된다 (D-Bus 예와 마찬가지로 cgen subfolder 에 있는 것을 가정하였다):
```
<.>/project$ ./cgen/commonapi-generator/commonapi-generator-linux-x86 -sk ./fidl/HelloWorld.fidl
<.>/project$ ./cgen/commonapi_someip_generator/commonapi-someip-generator-linux-x86 ./fidl/HelloWorld.fdepl
```

:exclamation: some SOME/IP param 은 필수 항목인 것을 기억하자.  즉 code gentor input file 은 fdepl이다; fidl 은 code gentor 의 입력 파일이 아니다.

gen 된 sources 가 있는 dir 을 확인해 보자.
```
$ cd src-gen/v1/commonapi
<.>/src-gen/v1/commonapi$ ls
HelloWorld.hpp           HelloWorldSomeIPDeployment.cpp  HelloWorldSomeIPProxy.hpp        HelloWorldStubDefault.cpp
HelloWorldProxyBase.hpp  HelloWorldSomeIPDeployment.hpp  HelloWorldSomeIPStubAdapter.cpp  HelloWorldStubDefault.hpp
HelloWorldProxy.hpp      HelloWorldSomeIPProxy.cpp       HelloWorldSomeIPStubAdapter.hpp  HelloWorldStub.hpp
```

## Step 5: Write the client and the service application
CommonAPI C++ 부분이다.  "CommonAPI C++ D-Bus in 10 minutes" 의 step 5 와 동일하다.

## Step 6: Build and run
우리가 원하는 app 을 빌드하는 가장 쉬운 방법은 simple CMake file 을 만드는 것이다.
project dir 에 CMakeLists.txt 를 만들자:
```
cmake_minimum_required(VERSION 2.8)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -std=c++0x")
include_directories(
    src-gen
    $ENV{RUNTIME_PATH}/capicxx-core-runtime/include
    $ENV{RUNTIME_PATH}/capicxx-someip-runtime/include
    $ENV{RUNTIME_PATH}/vsomeip/interface
)
link_directories(
    $ENV{RUNTIME_PATH}/capicxx-core-runtime/build
    $ENV{RUNTIME_PATH}/capicxx-someip-runtime/build
    $ENV{RUNTIME_PATH}/vsomeip/build
)
add_executable(HelloWorldClient
	src/HelloWorldClient.cpp
	src-gen/v1/commonapi/HelloWorldSomeIPProxy.cpp
	src-gen/v1/commonapi/HelloWorldSomeIPDeployment.cpp
)
target_link_libraries(HelloWorldClient CommonAPI CommonAPI-SomeIP vsomeip)
add_executable(HelloWorldService
	src/HelloWorldService.cpp
	src/HelloWorldStubImpl.cpp
	src-gen/v1/commonapi/HelloWorldSomeIPStubAdapter.cpp
	src-gen/v1/commonapi/HelloWorldStubDefault.cpp
	src-gen/v1/commonapi/HelloWorldSomeIPDeployment.cpp
)
target_link_libraries(HelloWorldService CommonAPI CommonAPI-SomeIP vsomeip)
```
CMake 와 make 를 호출하자 (이전 step 에서 이미 build 폴더를 만들어 두었었다):
```
<.>/build$ cmake ..
<.>/build$ make
```
이제 작성한 service 와 client app 을 시작할 준비가 거의 다 되었다.
이전 버전의 vsomeip 에서는 vsomeip config file 이 꼭 필요했는데, 외부 통신이 없는 simple 한 HelloWorld app 은 필요가 없다. 따라서 추가 작업 없이 실행 가능하다.
```
<.>/build$ ./HelloWorldService &
2016-12-12 08:44:33.260469 [info] Default configuration module loaded.
2016-12-12 08:44:33.260517 [info] Initializing vsomeip application "service-sample".
2016-12-12 08:44:33.260637 [info] SOME/IP client identifier configured. Using 1001 (was: 0000)
2016-12-12 08:44:33.260690 [info] No routing manager configured. Using auto-configuration.
2016-12-12 08:44:33.260724 [info] Instantiating routing manager [Host].
2016-12-12 08:44:33.260841 [info] init_routing_endpoint Routing endpoint at /tmp/vsomeip-0
2016-12-12 08:44:33.260950 [info] Client [1001] is connecting to [0] at /tmp/vsomeip-0
2016-12-12 08:44:33.261012 [info] Service Discovery enabled. Trying to load module.
2016-12-12 08:44:33.261231 [info] Service Discovery module loaded.
2016-12-12 08:44:33.261331 [info] Application(service-sample, 1001) is initialized (11, 100).
2016-12-12 08:44:33.261895 [info] OFFER(1001): [1234.5678:1.0]
Successfully Registered Service!
Waiting for calls... (Abort with CTRL+C)
2016-12-12 08:44:33.262143 [info] Starting vsomeip application "service-sample" using 2 threads
2016-12-12 08:44:33.263693 [info] Watchdog is disabled!
2016-12-12 08:44:33.263989 [info] vSomeIP 2.5.2
2016-12-12 08:44:33.264096 [info] Network interface "lo" is up and running.
2016-12-12 08:44:33.466823 [info] Port configuration missing for [1234.5678]. Service is internal.
<.>/build$ ./HelloWorldClient
2016-12-12 08:45:40.470731 [info] Parsed vsomeip configuration in 0ms
2016-12-12 08:45:40.471458 [info] Default configuration module loaded.
2016-12-12 08:45:40.471644 [info] Initializing vsomeip application "client-sample".
2016-12-12 08:45:40.472069 [info] SOME/IP client identifier configured. Using 1002 (was: 0000)
2016-12-12 08:45:40.472246 [info] No routing manager configured. Using auto-configuration.
2016-12-12 08:45:40.472378 [info] Instantiating routing manager [Proxy].
2016-12-12 08:45:40.472808 [info] Client [1002] is connecting to [0] at /tmp/vsomeip-0
2016-12-12 08:45:40.473233 [info] Listening at /tmp/vsomeip-1002
2016-12-12 08:45:40.473411 [info] Application(client-sample, 1002) is initialized (11, 100).
Checking availability!
2016-12-12 08:45:40.476118 [info] Starting vsomeip application "client-sample" using 2 threads
2016-12-12 08:45:40.497001 [info] Client 1002 successfully connected to routing  ~> registering..
2016-12-12 08:45:40.497741 [info] Application/Client 1002 is registering.
2016-12-12 08:45:40.497881 [info] Client [1001] is connecting to [1002] at /tmp/vsomeip-1002
2016-12-12 08:45:40.498225 [info] Application/Client 1002 is registered.
2016-12-12 08:45:40.498408 [info] REGISTERED_ACK(1002)
2016-12-12 08:45:40.498539 [info] REQUEST(1002): [1234.5678:1.4294967295]
Available...
2016-12-12 08:45:40.502828 [info] Client [1002] is connecting to [1001] at /tmp/vsomeip-1001
sayHello('Bob'): 'Hello Bob!'
Got message: 'Hello Bob!'
Waiting abort... (Abort with CTRL+C)
<.>/build$
```
