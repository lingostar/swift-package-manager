# Swift Package Manager

스위프트 패키지 매니저는 소스코드의 배포를 위한 툴이며, 당신의 코드를 공유하고 다른 사람들의 코드를 재활용하기 쉽게 하는 것을 목표로 한다. 이 툴은 스위프트 패키지들을 컴파일하고 링크하고, 의존성과 버전을 관리하고, 유연한 배포와 협업 모델을 지원한다.

GitHub 같은 서비스상에서 패키지들을 공유하기에 매우 쉽게 시스템을 디자인 했지만, 패키지들은 개인 개발이나 팀 내에서 코드를 공유하거나, 또는 다른 어떤 상황에서도 아주 유용하다.

* * *

## 작업 진행 중

스위프트 패키지 매니저는 초기단계의 디자인과 개발 중이다. 안정적인 버전을 Swift 3에서 사용할 수 있도록 하는 목표를 가지고 있지만 현재 모든 자세한 사항들은 변경되기 쉬우며 많은 중요한 기능들은 아직 구현되지 않았다.

추가적으로, 스위프트 문법이 안정화되어 있지 않으므로 만들어진 패키지는 스위프트의 발전에 따라 깨질 수도 있다는 것을 알아두길 바란다.

## 설치

패키지 매니저는  [**트렁크 개발** 스냅샷은 swift.org에서 볼 수 있음](https://swift.org/download/)와 함께 번들로 묶여 있다. 패키지 매니저를 설치하려면 커맨드 라인에서  아래의 인스톨 중 하나를 해야 한다:

* Xcode 7.3:

        export TOOLCHAINS=swift

* Xcode 7.2:

        export PATH=/Library/Toolchains/swift-latest.xctoolchain/usr/bin:$PATH

* Linux:

        export PATH=path/to/toolchain/usr/bin:$PATH

터미널에  `swift build --version`을 타이핑 해 보면 설치를 확인할 수 있다:

```sh
$ swift build --version
Apple Swift Package Manager
```

아래는 스냅샷을 성공적으로 설치하지 못했다는 것을 나타낸다:

    <unknown>:0: error: no such file or directory: 'build'

### 스위프트 환경 관리

OS X의 `TOOLCHAINS` 환경 변수가 어떤 `swift`가 인스턴스화 되었는지를 제어하는 데 사용될 수 있다:

```sh
$ xcrun --find swift
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swift
$ swift --version
Apple Swift version 2.2
$ export TOOLCHAINS=swift
$ xcrun --find swift
/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin/swift
$ swift --version
Swift version 3.0-dev
```

OS X에서 `/usr/bin/swift` 은 활성 툴체인으로 invocation을 포워드 해 주는 stub일 뿐이다. 그러므로 `swift build`를 호출하면 `TOOLCHAINS`환경 변수에 정의된 스위프트를 이용할 것이다.

특정 툴체인을 사용하기 위해서 `.xctoolchain`의 Info.plist에서 `TOOLCHAINS`를 `CFBundleIdentifier`로 설정할 수 있다.

이 기능은 Xcode 7.3이 필요하다


## 개발

패키지 매니저는 그 자체로 스위프트 패키지이므로 그 자신을 빌드하기 위해 사용될 수 있다. 하지만 우리는 다음 세 가지 옵션들 중 하나를 권한다:

1. [스위프트 프로젝트 `build-script`](https://github.com/apple/swift/blob/master/README.md) 사용하기:

        swift/utils/build-script --swiftpm --llbuild

2. 부트스트랩 스크립트로 독립적으로 하기 :
  1. [스위프트 스냅샷을 다운로드하고 설치하기](https://swift.org/download)
  2. 그 `usr/bin` 디렉토리로 가서
  3. 부트스트랩 스크립트를 실행하기:

            swiftpm/Utilities/bootstrap --swiftc path/to/snapshot/usr/bin/swiftc --sbt path/to/snapshot/usr/bin/swift-build-tool

   `swiftc` 와 `swift-build-tool` 는 모두 다운로드 가능한 스위프트 스냅샷에서 제공하는 실행파일(익스큐터블)이며, _이 들은 리포지터리의 소스로부터는 빌드되지 **않는다**_.

3. [Support](Support)의 Xcode 프로젝트를 이용하기. 이 옵션은 다음을 필요로 한다:
   * Xcode 7.3 (beta)
   * [llbuild](https://github.com/apple/swift-llbuild)SwiftPM 복제본에 대응해서 복제됨
  * 아마도, [보다 최신의 스위프트 스냅샷](https://swift.org/download)

###스위프트 버전 선택하기

`SWIFT_EXEC` 환경 변수는`swift build`에 의해 사용되는 `swiftc`실행파일의 경로를 지정한다. 설정되어 있지 않으면, SPM이 위치를 찾으려 할 것이다:


1. `swift-build`의 상위 디렉토리 안.
2. (OS X의 경우) `xcrun --find swiftc`를 호출해서 확인
3. PATH 안


개발 지향의 문서를 더 보고 싶으면[Documentation/Internals](Documentation/Internals)을 보라.


## 시스템 요구사항

패키지 매니저의 시스템 요구사항은 [Swift의 요구사항](https://github.com/apple/swift#system-requirements)과 동일하며 패키지 매니저가 빌드-타임 뿐만 아니라 런타임에도 Git를 필요로 한다는 조건을 가진다.

## 기여하기

스위프트 프로젝트에 대한 기여를 운영하는 정책과 우수 사례에 대해 알고 싶으면, [기여자 가이드](https://swift.org/contributing/)를 보라.

기여에 관심이 있다면, 현재 구현을 결정한 것에 대한 제반사항과 앞으로의 기능 개발을 위한 방향을 제시하는 [커뮤니티 제안](Documentation/PackageManagerCommunityProposal.md)을 읽기 바란다.

이 프로젝트의 개발과 발전를 위해 테스트는 매우 중요한 부분이며, 새로운 기여는 모든 기능 변화에 대해 테스트를 포함해야 한다. 테스트 구종을 위해, `bootstrap` 스크립트에 `test`명령을 넘겨라.

    ./Utilities/bootstrap test

> 장기적으로, 우리는 테스팅이 패키지 매니저 그 자체에 통합된 부분이 되길 원하며, 이 부분의 커스텀은 지원하지 않은 것이다.

스위프트 패키지 매니저는 [llbuild](https://github.com/apple/swift-llbuild)를 아래쪽에서 소스 파일을 컴파일하기 위한 빌드 시스템으로 사용한다. 이 또한 오픈소스이며 스위프트 프로젝트의 일부이다.

## 도움 받기

패키지 매니저에 문제가 있으면, 도움을 받을 수 있다. 다음 방법을 추천한다:

* The [스위프트-사용자 메일링 리스트](mailto:swift-users@swift.org)
* Our [버그 추적](http://bugs.swift.org)

당신의 질문을 리스트에서 공유하는 것이 불편하면,[CODE_OWNERS.txt](CODE_OWNERS.txt)에서 찾을 수 있는 코드 소유권자에게 코드에 대해 자세한 연락을 취할 수 있다; 하지만, 메일링 리스트가 도움을 청할 수 있는 가장 좋은 장소이다.

* * *

## 기술적 오버뷰

스위프트와 패키지 매니저에 대한 전체적인 가이드는 [swift.org 에서](https://swift.org/package-manager/) 볼 수 있다. 아래는 기술적인 문서로서, 스위프트 패키지 매니저의 기능에 대한 동기부여를 하는 기본적인 개념들을 서술한다.

### 모듈들

스위프트는 코드를 _모듈별_ 로 관리한다. 각각의 모듈은 네임스페이스를 가지며 코드의 어떤 부분이 모듈의 바깥에서 사용가능한지에 대한 접근 권한을 강제한다.

프로그램은 자신의 모든 코드를 하나의 모듈에 가지고 있어야 하며, 다른 모듈들을 _의존성_ 을 가지고 임포트 할 수도 있다.
OS X의 Darwin이나 Linux의 GLibc과 같이 시스템이 제공하는 편리한 모듈들을 제외하고, 대부분의 의존하는 모듈들은 사용하기 위해 코드를 다운로드 하고 빌드되어야 한다.

> 특정 문제를 해결하는 코드를 추출해서 분리된 모듈에 넣는 것은 그 코드가 다른 상황에서도 사용될 수 있도록 하는 것이다.
> 예를들어, 네트워크 요청을 만드는 기능을 제공하는 모듈은 사진 공유 앱과 날씨 예보를 보여주는 프로그램간에 공유될 수 있다.
> 그리고 만약 더 나은 작업을 하는 모듈이 나온다면, 최소한의 변경으로 손쉽게 교체될 수 있다.
> 모듈화를 받아들임으로서, 작업 도중에 만나는 문제들에 발목잡히지 않고, 당장 해결해야 하는 문제의 즐거운 부분에 집중할 수 있다.

제일 중요한 원칙으로 : 모듈이 많이 만드는 게 적게 만드는 것 보다 나을 것이다. 패키지 매니저는 가능한 쉽게 여러개의 모듈들을 이용해 패키지와 앱을 만들도록 디자인 되었다.

### 스위프트 모듈 빌드하기

스위프트 패키지 매니저와 그 빌드 시스템은 당신의 소스코드를 컴파일 하는 법을 이해해야 한다. 이를 위해, 파일 시스템 상의 소스코드 구성을 통해 의미하는 바를 결정하는 컨벤션-기반의 접근을 사용한다. 하지만 그 자세한 사항들은 커스터마이즈 할 수 있으며 규칙을 덮어쓸 수 있다. 간단히 예를 들어보면 아래와 같을 것이다:

    foo/Package.swift
    foo/Sources/main.swift

> `Package.swift` 는 패키지의 메타데이터를 포함하는 마니페스트 파일이다. 간단한 프로젝트에서는 비어있는 파일도 괜찮지만, 파일이 존재해야 한다. `Package.swift`는 다음 섹션에서 설명하겠다.

그런  다음 아래 명령을 `foo` 디렉토리에서 실행하면:


```sh
swift build
```


스위프트는 `foo`라는 이름의 단일 실행파일을 빌드할 것이다.

패키지 매니저에게 있어, 모든 것은 패키지 이므로 `Package.swift`이다. 하지만 이것이 당신의 소프트웨어를 세상에 릴리즈 해야 한다는 것을 의미하지는 않는다: 앱을 개발해서 다른 사람이 보거나 사용할 수 있는 곳에 퍼블리싱하지 않아도 된다. 반면에, 만약 언젠가 당신의 프로젝트가 보다 많은 사람들에게 사용가능 _해야 한다면_ 당신의 소스들은 퍼블리시를 준비한 상태여야 한다. 패키지 매니저는 특정 형태의 배포방식으로부터 독립적이다. 그러므로 당신의 개인 프로젝트들 내에서, 워크그룹 내에서, 팀 또는 회사 내에서 또는 세상 모두와 함께 코드를 공유하기 위해서 사용할 수 있다. 

물론, 패키지 매니저는 그 자신을 빌드하기 위해 사용되므로, 그 자신의 소스파일들은 아래 링크의 컨벤션을 따라 레이아웃 되어 있다.

> [읽을 거리: 소스 레이아웃](Documentation/SourceLayouts.md)

현재는 우리가 스태틱한 라이브러리들만 빌드할 수 있다는 것을 기억하기 바란다. 일반적으로 이것은 장점을 가지고 있지만, 다이나믹 라이브러리에 대한 수요가 있다는 것을 이해하고 있으며 이것을 지원하는 것이 계획된 코스에 추가될 것이다.

### 패키지 & 의존성 관리

현대의 개발은 기하급수적인 외부 의존성들에 의해 탄력을 받고 있다(나은 방향이든 나쁜 방향이든). 이는 당신이 보다 적은 시간에 많은 일을 할 수 있도록 하는 멋진 것이지만, 프로젝트에 의존성을 더하는 것은 연관된 조정 비용을 발생시킨다.

의존성을 위해 소스코드를 다운로드하고 빌드하는 것 외에, 의존 라이브러리의 의존성들 역시 다운로드되고 빌드되어야 한다. 이 작업을 모든 의존성 그래프가 만족될 때까지 해야 한다.
더 복잡한 면은, 의존성은 특정 버전을 요구하는 경우가 있으며, 다른 모듈들과의 버전 요구사항들을 하나의 동일한 의존성 안에서 중재해야 한다.

패키지 매니저의 역할은 프로젝트의 모든 의존성에 대해서 다운로드하고 빌드하는 과정들을 자동화하고 코드 재사용을 통해 중재 비용을 최소화하는 데 있다.

의존성은 `Package.swift` 매니페스트 파일에 지정되어 있다.

> [읽을 거리: Package.swift — 매니페스트 파일](Documentation/Package.swift.md)

> [읽을 거리: 패키지 개발하기](Documentation/DevelopingPackages.md)

### 시스템 라이브러리 사용하기

당신의 플랫폼은 시스템 패키지 매니저에 의해 설치된 풍부하고 강력한 C 라이브러리들을 가지고 있다. 스위프트 코드들은 그것을 사용할 수 있다.

> [읽을 거리: 시스템 모듈들](Documentation/SystemModules.md)

## License

Copyright 2015 - 2016 Apple Inc. and the Swift project authors.
Licensed under Apache License v2.0 with Runtime Library Exception.

See https://swift.org/LICENSE.txt for license information.

See https://swift.org/CONTRIBUTORS.txt for Swift project authors.
