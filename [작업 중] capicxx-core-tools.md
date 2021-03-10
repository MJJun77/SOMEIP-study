(( site : https://genivi.github.io/capicxx-core-tools/ ))
* 사이트 반복 reading 효율을 위한 자가 번역...
* 내맘대로 약어 : desc == description, comm == communication, net == network, app == application, env == environment, def == definition, gen == generate, gentor == generator, func == function, repo == repository  ... and so on 

# Welcome to CommonAPI C++!
CommonAPI C++ 는 IPC 와 network comm 을 위한 C++ framework 이다. 
기본 목적은 IPC 나 network comm 을 위한 또다른 새로운 mechanism 을 만들지 않고 high-level C++ API 를 제공하는 것이다. C++ 개발자들은 comm framework 나 protocol 에 대한 깊은 지식 없이도 이것들을 사용할 수 있는 이점이 있다. (*주: CommonAPI C++ 도 충분히 복잡해 보이는데, 사실 범용적인 d-bus 같은 IPC 를 쓰는 게 만만한 작업이 아니다.)

app 이 실제 사용할 IPC mechanism 에 독립적이 되도록 하기 위해 CommonAPI C++ 는 core component 를 구성한다.  이 component 는 1) app API 를 제공하는 component와, 2) IPC mechanism specific 한 _binding_ 이라 명명한 IPC mechanism specific 한 component 를 제공한다.
이 두 기본 component 는 다시 표준 function(init, logging, runtime env 등) 을 제공하는 generic part 들과 app specific part 로 나누어진다.
app specific part 는 interface description language 인 Franca IDL 로 기술된 파일로부터 생성되어야 한다. (Franca: https://github.com/franca/franca[FrancaIDL]).
![CommonAPIOverview02.png image](pictures/CommonAPIOverview02.png "CommonAPIOverview02.png image")

첫번째 procedure 는 platform이나 program-language independent 한 Franca IDL 로 service interface 를 정의하는 것이다.
다음으로 CommonAPI code gentor(CommonAPI-Tools) 를 사용하여 proxy 와 stub 을 gen 한다.
그 다음엔 gen 된 skeleton function body 를 이용하여 stub function 을 구현하고 난 후, gen 된 proxy 를 사용하여 이 func 들을 호출하는 client 를 구현한다.
실행 파일(client & server) 생성을 위해 binding specific code gentor 와 함께 IPC frameware specific code (glue-code) 를 gen 해라.
(현재 D-Bus 를 위한 것과 SOME/IP 를 위한 게 있다.(*주: 하단 링크)  제공되는 documentation의 상세 내용을 확인해라.) 

At the end compile it together with the provided runtime libraries (for CommonAPI itself and again a binding specific library, for details see again the provided documentation).
![Workflow.png image](pictures/Workflow.png "Workflow.png image")

CommonAPI C++ 를 처음 사용해 본다면 tutorial (아래 download 섹션 링크) 링크 및 예제를 참조해라.
source code 는 CommonAPI-Tools repo 에 있다. 보다 쉬운 소개는 Wiki 에도 있다. (아래 링크도 확인)

# Basic Features
* Interface description with the easy to learn, human-readable FrancaIDL.
* Support of Franca features as inheritance, polymorphic structs, unions.
* Code generation for clear and not too complex C++ code.
* Actual support for D-Bus (http://www.freedesktop.org/wiki/Software/dbus/) and SOME/IP (see http://some-ip.com/).
* High-performance implementation by using C++ templates (runtime-performance).
* Highly configurable for the integration on different platforms (e.g. main-loop support, support of Franca deployment files).
* Multithreading support (thread-safe).

# Links
* CommonAPI C++ runtime: https://github.com/GENIVI/capicxx-core-runtime/
* Eclipse update-site: http://genivi.github.io/capicxx-core-tools/updatesite/
* Wiki page: https://github.com/GENIVI/capicxx-core-tools/wiki/
* Overview repositories: https://github.com/orgs/GENIVI/teams/someip/repositories/

# Bindings
* CommonAPI C++ D-Bus binding: http://genivi.github.io/capicxx-dbus-tools/
* CommonAPI C++ SOME/IP binding: http://genivi.github.io/capicxx-someip-tools/

# Downloads (*주: 링크는 안걸어 둠.  상단에 있는 original site 이용할 것)
* Download Core Code Generator 3.1.12.3 (Binary)
* Download Core Runtime 3.1.12.4 (TAR Ball)
* Download User Guide 3.1.12 (pdf)
