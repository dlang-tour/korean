# D 언어 개발 환경 구성하기(Install D locally)

D 언어 공식 웹사이트인 [dlang.org](https://dlang.org) 에서 **DMD** (Digital Mars D의 줄임말)라는 표준 D 컴파일러를 다운로드할 수 있습니다.

다운로드 및 설치 방법을 확인하려면 [다운로드](http://dlang.org/download.html) 페이지를 방문하십시오.

### 마이크로소프트 윈도우 환경(Windows)에서 설치

* 모든 설정을 자동으로 해주는 [인스톨러](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd-{{latest-release}}.exe) 를 사용
* [압축파일](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.windows.7z) 형태로 받아서 사용
* 패키지 관리자인 [chocolatey](https://chocolatey.org/packages/dmd) 설치 후 `choco install dmd` 를 실행해 설치

### 애플 맥 OS 환경(macOS) 에서 설치

* `.dmg` 패키지를 직접 [다운로드](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.dmg) 하여 설치
* [압축파일](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.osx.tar.xz) 형태로 받아서 사용
* 패키지 관리자인 [Homebrew](http://brew.sh) 설치 후 `brew install dmd` 를 실행해 설치

### 리눅스(Linux) / FreeBSD / macOS 에서 설치

시스템 전역에서 필요한 게 아니라, 단지 개인적으로 사용하고자 한다면 설치하려는 홈 디렉토리에서 `curl -fsS https://dlang.org/install.sh | bash -s dmd` 명령어로 간단히 설치 가능합니다.

하지만 대부분의 경우 패키지 관리자를 사용해 설치하는 것이 업데이트나 버전 관리에 유리합니다. 주요 리눅스 배포판 별 D 언어 설치 링크를 정리해두었습니다.

* [아치리눅스](https://wiki.archlinux.org/index.php/D_(programming_language))
* [데비안/우분투](http://d-apt.sourceforge.net).
* [페도라/센트OS](http://dlang.org/download.html#dmd)
* [젠투](https://wiki.gentoo.org/wiki/Dlang)
* [오픈수세](http://dlang.org/download.html#dmd)

## 다른 컴파일러 구현들(Other compilers)

DMD는 D 언어 표준 컴파일러로, 내부에 독자적인 컴파일러 구현을 포함하고 있습니다.

만약 다른 컴파일러가 제공하는 기반 기술이나 기능이 필요하다면, 두가지 정도 선택지가 더 남아있습니다.

* [**GDC**](http://gdcproject.org/downloads) 는 GCC 컴파일러 인프라스트럭처를 활용합니다
* [**LDC**](https://github.com/ldc-developers/ldc#installation) 는 LLVM 컴파일러 인프라스트럭처를 활용합니다

GDC와 LDC 는 표준 컴파일러에 비해 최신 사양에 대응하는 속도가 더딜 수 있습니다.

하지만, ARM CPU 등에서 표준 컴파일러보다 더 나은 성능을 발휘하는 결과물을 얻을 수 있습니다.

컴파일러별 특징에 대해 더 알고 싶다면 D 언어 위키의 [컴파일러](https://wiki.dlang.org/Compilers) 글을 참고하십시오.

## 에디터 설정하기(Configure your editor)

D 언어는 개발시 상용구 코드(boilerplate code)가 거의 드물기 때문에 고급 IDE(통합개발환경) 프로그램이 필요하지 않습니다.

그러나 자신에게 맞는 좋은 에디터를 준비함으로서, 더욱 프로그램 작성과 테스트가 즐거워지고 편해질 겁니다.

D 언어 코드 작성을 도와줄 플러그인/애드온이 존재하는 에디터들을 아래에 정리해두었습니다.

- [Atom](https://github.com/Pure-D/atomize-d)
- [이클립스](http://ddt-ide.github.io)
- [이맥스](https://github.com/Emacs-D-Mode-Maintainers/Emacs-D-Mode)
- [인텔리J](https://github.com/intellij-dlanguage/intellij-dlanguage)
- [서브라임 텍스트](https://github.com/yazd/DKit)
- [Vim](https://wiki.dlang.org/D_in_Vim)
- [VS Code](https://marketplace.visualstudio.com/items/webfreak.code-d)
- [비주얼 스튜디오](http://rainers.github.io/visuald/visuald/StartPage.html)

IDE를 사용하는 게 잘못된 것은 아닙니다. 만약 D 언어 코드 작성에 최적화된 IDE를 찾으신다면, 아래의 선택지가 존재합니다.

- [Coedit](https://github.com/BBasile/Coedit)
- [Dlang IDE](https://github.com/buggins/dlangide)

여기에 소개되지 않은, 각 에디터와 IDE에 대한 자세한 특징 비교는 D 언어 위키의 [에디터](https://wiki.dlang.org/Editors) and [IDE](https://wiki.dlang.org/IDEs) 페이지에서 확인할 수 있습니다.
