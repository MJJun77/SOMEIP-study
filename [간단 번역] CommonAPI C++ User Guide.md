(( 2021.Mar : Study 용 번역 중 from https://docs.projects.genivi.org/ipc.common-api-tools/3.1.3/html/CommonAPICppUserGuide.html ))
이름은 CommonAPI 이지만, 어쨌든 _genivi.org_ 이니 목적은 분명하다...
약자: intf == interface, desc == description, gen == generation, env == environment, cli== command-line, gentor == generator  ...   and so on...

# CommonAPI C++ User Guide
29. Jul 2015

## Introduction
### 문서 목적
이 user guide 는 다음 내용에 대해 설명한다:
* CommonAPI 설치 방법 (code generator(CommonAPI Tools) 포함)
* Hello World program 작성 가이드
* Franca IDL 을 사용하여 CommonAPI 를사용하는 좀 더 깊이있는 예제
* CommonAPI specification 에 포함되지 않은 CommonAPI 의 추가 정보

### CommonAPI C++
__CommonAPI C++ 는, Middleware 를 통해 IPC 통신하는 분산 app 개발을 위한 표준화된 C++ API 이다.__
(*주: 원문은 CommonAPI C++ is a standardized C++ API specification for the development of distributed applications which communicate via a middleware for interprocess communication.)
주 목적은 system dependent 한 IPC stack 과 독립적인 C++ interface 를 만드는 것이다. 기본 동작 원리는 아래 그림과 같다.
![CommonAPIOverview01.png image](pictures/CommonAPIOverview01.png "CommonAPIOverview01.png image")

* CommonAPI C++ 는 middleware-independent part(CommonAPI Core) 와 middleware-specific part(CommonAPI Binding) 로 나누어져 있다.
* CommonAPI 는 logical intf spec 기술을 위해 intf desc language 인 FrancaIDL 를 사용한다.  FrancaIDL 로부터 code gen 하는 것은 CommonAPI 의 일부분이다.
* CommonAPI C++ binding 을 위한 code gen 은 middleware-specific parameters(deployment parameters) 가 필요하다. 이 파라미터들은 Franca deployment files (*.fdepl) 에 정의된다.

:warning: CommonAPI C++ Core 는 deployment parameters 에 대한 책임이 없다.  하지만 CommonAPI C++ Core 자체에 대한 additional deployment parameter 를 추가하는 것이 타당하다.

CommonAPI user API 는 두 파트로 나누어진다. (아래 그림 참조):
* FrancaIDL 가 gen 한, FrancaIDL 파일에 정의된 attributes, types, method 관련된 API function (*주: 연두색 부분)
* runtime environment 를 로딩하거나 proxy 를 create 하는 등의 "common" part (Runtime API) (*주: 파란색 부분)
![CommonAPIOverview02.png image](pictures/CommonAPIOverview02.png "CommonAPIOverview02.png image")

이 그림은 CommonAPI C++ 이 어떻게 상호 작용하는지 보여준다.  아래 사항을 확인하자:
* 주된 내용은 CommonAPI 의 gen 된 user API 이다.
* CommonAPI Core 와 IPC stack 간 직접적인 연관이 없다.
* gen 된 CommonAPI Binding(*주: 빨간색) 은 모든 다른 CommonAPI 와 interface 하고 있다.

app 개발자의 workflow 는 다음과 같다:
* FnancaIDL 파일에 method, attributes 를 가지는 interface spec 을 작성한다.
* CommonAPI code gentor 를 사용하여 client 와 service 코드를 gen 한다.
* gen 된 service skeleton 에 service 구현부를 작성한다.
* proxy 를 사용하여 client 를 구현한다.

### Link
CommonAPI 는 GENIVI project 이고, source code 와 latest news 는 여기에 있다:http://projects.genivi.org/commonapi/
Source code 의 Git repository는: http://git.projects.genivi.org/
documentation 은 GENIVI document page 를 참조하라: http://docs.projects.genivi.org/ 

:warning:
http://git.projects.genivi.org/ 에는 code gentor 소스밖에 없을 것이다. code gentor 를 직접 소스 빌드하는 것은 귀찮은 작업이다.  편의를 위해 실행파일을 올려두었다: http://docs.projects.genivi.org/yamaica-update-site/.
Closely related to CommonAPI is the yamaica project which provides a full integration of all Franca IDL and CommonAPI plugins and some more enhanced features like the import and export of Franca files to Enterprise Architect. The yamaica project provides eclipse update-sites for CommonAPI and yamaica (see http://docs.projects.genivi.org/yamaica-update-site/) ready for installation.
현재 official FrancaIDL 는: https://code.google.com/a/eclipselabs.org/p/franca/

## Integration Guide for CommonAPI users
이후 설명은 Linux Platform 을 가정한다.
하지만CommonAPI 는 Windows 도 지원하는데, Windows 관련 알아야 할 점은 이 Integration Guide 하단에 별도 분리된 Windows 문단을 참조하라.

### Requirements
CommonAPI 는 GENIVI 를 위해 개발되었고 따라서 mostly Linux platform 에서 동작한다. 추가적으로 테스트나 개발 목적으로 Windows 에서 동작할 수도 있다. 알아야 할 점은:
* CommonAPI 는 C++11 feature(variadic templates, std::bind, std::function 등)이 필요하므로 컴파일러가 이를 지원해야 한다. (e.g. gcc 4.8).
* CommonAPI build system 은 CMake 이다.  host system 에 설치되어 있어야 한다.(CommonAPI requires a CMake version > 2.8.12.)
* 이 user guide 는 "common" CommonAPI part 에 대해서만 설명한다; binding specific 관련 설명은 없다.  추가 정보가 필요한 경우 binding specific user guide 를 참조하라.
* Eclipse Luna 이전 버전을 사용하지 마라; 동작할 수도 있는데, 보장할 수 없다.
* code gen 을 위한 tool chain 은 Maven 이다. 적어도 Maven 3 버전 이상이어야 한다.  eclipse 를 사용한다면 maven plug-in 이 설치되었는지 확인해라.

### Dependencies
![CommonAPI-Dependencies.png image](pictures/CommonAPI-Dependencies.png "CommonAPI-Dependencies.png image")

### Compile Runtime
#### Command-line
cli 에서 CommonAPI 빌드하기 위해 Git 에서 Common API runtime 를 다운로드받은 후 원하는 tag (e.g. 3.0.0) 로 변경한 뒤 아래와 같이 컴파일하자:
```
$ git clone git://git.projects.genivi.org/ipc/common-api-runtime.git
$ cd common-api-runtime
$ git checkout tags/3.0.0
$ mkdir build
$ cd build
$ cmake ..
$ make
```
이건 일반적인 표준 방법이므로, 성공적으로 shared CommonAPI runtime library libCommonAPI.so 가 build/src/CommonAPI 애 생성되어야 한다.  CMake 가 doxygen 과 asciidoc 이 설치되었는지 확인하는데, 이건documentation 을 위해서만 필요하다.  CommonAPI unit test 는 Google C++ Testing Framework 를 사용하여 작성되었다.  만일 unit test 를 돌리고 싶다면 환경 변수 GTEST_ROOT 가 올바른 dir 을 가리켜야 한다. (아래의 contributor’s guide 참조).

:warning: 만일 tar file 로부터 CommonAPI 설치를 원하면 여기서 다운받을 수 있다: http://docs.projects.genivi.org/yamaica-update-site/CommonAPI/runtime/

CMake / make 를 위한 여러 옵션이 있다 (default 는 shared lib).  아래와 같이 static lib 생성이 가능하고, 경로는 build/src/CommonAPI 이다.
```
$ cmake -DBUILD_SHARED_LIBS=OFF ..
```

Release version 생성을 위해서는 아래와 같이 하면 된다. (default is debug).
```
$ cmake -DCMAKE_BUILD_TYPE=Release ..
```

추가 설정이 없다면 `make install` 은 default 로 CommonAPI libraries 와 header files 을 /usr/local 로 복사한다. the installation prefix 를 변경하여 dest dir 을 변경할 수 있다. (e.g. to test).
```
$ cmake -DCMAKE_INSTALL_PREFIX=/test  ..
```

아래는 추가적인 cmake parameter 이다:
| CMake Parameter | Value | Description |
|:---             |:---  |:---         |
| -DUSE_INSTALLED_COMMONAPI | OFF, ON                              | use uninstalled,  installed CommonAPI core library |
| -DMAX_LOG_LEVEL           | ERROR, WARNING, INFO, DEBUG, VERBOSE | log messages with lower log level are ignored      |


아래는 make 의 target list 이다:
| Make Target | Description | 
|:---             |:---  |
| make all | Same as make. Will compile and link CommonAPI. |
| make clean | Deletes binaries, but not the files which has been generated by CMake. |
| make maintainer-clean | Deletes everything in the build directory. |
| make install | Copies libraries to /user/local/lib/commonapiX.X.X and header files to /user/local/include/commonapiX.X.X/CommonAPI. |
| make DESTDIR=< install_dir > install | The destination directory for the installation can be influenced by DESTDIR. |
추가적인 make target 들은 아래 contributer's guide 에 설명되어 있다.

#### Eclipse
(*주: Eclipse 는 안쓸 거니 생략..)

### Compile tools
CommonAPI code gentor 의 cli 실행파일 버전은 Linux(32 bit) 용이 있다. eclipse 용 code gentor 의 update site 도 있다. (아래 참조)  아래 가이드는 code gentor 를 직접 빌드하는 경우에 대한 설명이다.

#### Command-line
cli 에서 maven 을 call 하여 code gentor 를 빌드할 수 있다.  console 에서 CommonAPI_Tools 경로인 _org.genivi.commonapi.core.releng_ 로 이동하여 아래를 호출한다:
```
mvn -Dtarget.id=org.genivi.commonapi.core.target clean verify
```
성공적으로 빌드가 끝나면 _org.genivi.commonapi.core.cli.product/target/products/commonapi-generator.zip_ 와 the update-sites in _org.genivi.commonapi.core.updatesite/target_ 를 찾을 수 있을 것이다.
#### Eclipse
(*주: 생략)

### Write Application
CommonAPI 는 실행 app 생성을 위해 다음과 같은 basic workflow 가 필요하다.
![CommonAPIWorkflow.png image](pictures/CommonAPIWorkflow.png "CommonAPIWorkflow.png image")

#### Generating Code
어떤 개발 환경이든, app API 는 CommonAPI code gentor (cli 버전이든 eclipse update-site 이든) 를 사용하여 생성된다.
CommonAPI Tools 를 사용하는 가장 간단한 방법은 Eclipse 에 GENIVI project server 를 update-site 에 추가하는 것이다.  Eclipse 에서 Help -> Install New Software -> Add 를 해서 Update Site 를 추가하자.
URL 인 http://docs.projects.genivi.org/yamaica-update-site/CommonAPI/updatesite/ 을 추가하고 confirm 을 클릭하자.  그리고 site selection dropdown box 에서 새롭게 추가된 site 를 선택하고 Software selection window 에서 "GENIVI Common API" Tree 의 모든 메뉴를 선택한다.
이제 Eclipse 에 Software 가 설치된 다음에는 아무 .fidl 이나 .fdepl 파일을 우클릭하여 CommonAPI -> Generate Common API Code 옵션을 선택하여 CommonAPI 를 위한 C++ Code 를 gen 할 수 있다.
만일 자신만의 update-site 를 빌드했다면 이 사이트를 추가할 수 있다; 그냥 archive 로 더해라.
CommonAPI code gen 의 cli 용 실행파일은 .zip 으로 only Linux(32bit) 용으로만 있다: http://docs.projects.genivi.org/yamaica-update-site/CommonAPI/generator.  다운받아서 압축을 풀면 된다.
아래와 같이 사용하면 된다:
```
commonapi_generator [options] file [file...]
```

사용 가능한 옵션은:
| Option | Description  |
|:---|:---|
| -dest < path/to/output/folder > | gen 되는 파일 경로 지정 |
|-pref < path/to/header/file > | gen 되는 파일의 head comment 부분에 추가되는 text (예: license 설명 등). | 

:warning:
만일 당신의 CommonAPI binding 이 deployment file 이 필요하다면 code gentor 는 적절한 France File 을 import 하는 적절한 deployment file 이 필요하다. 
특정 bindings (예: D-Bus)과 파일 배포는 의무가 아니다: 이 경우 fidl 파일을 입력으로 code gen 을 시작하는 것도 가능하다.
(*주: 정확히 이해 못했다.  원문: If your CommonAPI binding requires deployment files the input for the code generator is the appropriate deployment file which imports the Franca file.
For some bindings (e.g. D-Bus) and deployment file is not obligatory; in this case it is also possible to start the code generator with fidl-files as input.)

#### Build Applications
작성한 app 은 오직 gen된 CommonAPI 와 CommonAPI runtime library 와만 링크되어 컴파일되어야 한다.  빠른 setup 을 위해 제공되는 CommonAPI-Tools/CommonAPI-Examples 사용을 고려해 봐라.

### Project Setup
#### Structuring CommonAPI project libraries
CommonAPI 실행파일은 크게 6부분으로 구성된다:
1. developer 가 manual 로 작성한 app code
1. gen 된 CommonAPI(binding independent) code.  client는 app 에서 불리는 proxy functions 을 포함; servic는 개발자가 직접 구현해야 하는 gen functions 포함 (optionally it is possible to generate default implementations).
3. CommonAPI runtime library.
4. gen 된 binding specific code (_glue code_ 라 불리는 것).
5. binding runtime library
6. 사용하는 middleware 의 일반적인 libraries (예: libdbus).

이 6개 파트를 target platform 을 위해 shared or static lib 로 분할/통합하는 게 가능하다 할지라도, CommonAPI app 통합을 위해 의도된 표준 방법이 있다 (아래 그림 참조)
![CommonAPIStructuringLibraries.png image](pictures/CommonAPIStructuringLibraries.png "CommonAPIStructuringLibraries.png image")















