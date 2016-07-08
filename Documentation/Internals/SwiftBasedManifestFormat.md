# 스위프트-기반 매니페스트 포맷

## 목적

우리는 부가적인 패키지 메타데이터를 기술할 수 있는 어떤 도구를, 소스 파일들 내부의 콘텐트 바깥영역에 가질 필요가 있다.이 문서는 이 매니페스트 데이터를 위한 스위프트-기반 포맷 사용에 대한 제안을 기술한다.


## 동기부여

패키지 매니저는 프로젝트 파일들과 소스 코드로부터 얻어지는 "컨벤션 기반의" 프로젝트 구조를 지원하기 위해 노력한다. 이러한 접근 방식은 사용자들이 우선 실제 소프트웨어 저작에 초점을 맞추도록 하고 툴들은 다음의 알맞은 default에 의해 그것을 프러덕트로 조립하는 것으로 예상하게 한다.

하지만, 패키지들은 프로젝트 구조로부터 자연스럽게 추론할 수 없는 정보도 가지고 있다. 이러한 경우, 우리는 부가적인 프로젝트 정보를 가지고 있는  어떤 매니페스트를 지원할 필요가 있다.


하이 레벨에서, 이 매니페스트의 주된 목적은 다음과 같다 :

* 컨벤션 기반의 시스템을 보완한다. 
 
  이 메니페스트는 프로젝트 메타데이터를 추가할 수 있는 유일하고 확실한 위치가 됨으로서 컨벤션 기반의 시스템을 보완한다. 이게 없으면 프로젝트는 커스텀 설정을 사용해야 한다. 목표는 80% 이상의 프로젝트가 메니페스트와 컨벤션 기반의 레이아웃만을 사용해도 되도록 하는 것이다.

  매니페스트가 컨벤션 기반 시스템의 디테일들 중 주의깊게 뽑은 몇 개의 키를 확장하고 오버라이드 하도록 허락함으로서, 더 많은 프로젝트들이 복잡한 컨벤션들을 정의할 필요 없이 이 시스템을 사용할 수 있도록 한다.

* 패키지 정보를 표준 포맷으로 제공한다.

  모든 프로젝트들이 포함하기를 희망하는, 또는 그것을 포함한 모든 프로젝트들이 표준 방식으로 하길 원하는 몇 가지 매우 중요하고 매우 일반적인 정보 조각들이 있다. 예를 들어, 프로젝트의 라이센스 선언문은 매우 표준화된 정의를 따라야 한다.

* 패키지 매니저 프로젝트의 인디케이터로서의 역할을 한다.

 매우 단순하지만 패키지의 루트에 알려진 이름의 매니페스트를 가진다는 것은 개발자와 그런 타입의 프로젝트의 툴들에게, 그것들이 어떻게 상호작용할 것인지를 예상할 수 있게 해 주는 인디케이터로서의 역할을 한다.


* 프로그램적인 분석과 프로젝트 구조 편집에 대한 지원을 제공한다. 

 매니페스트는 기계가 읽고 쓸 수 있는 포맷이어야 한다. 패키지의 내용을 검사하고 프로젝트 구조를 자동으로 편집하길  원할 것 같은 다양한 툴(예를 들면, 인덱스를 위한 빌드 정보)들을 구상하고 있다. 예를 들어, import 구문을 추가해서 새로운 라이브러리 의존성을 도입하려 할 때, 툴이 할 수 있다면, 사용자에게 알린 뒤, 자동으로 매니페스트를 업데이트해서 새로운 의존성을 지정하도록 하고 싶다.


## 제안

우리는 매니페스트를 작성하는데 스위프트 언어 그 자체를 사용하는 것을 제안한다. 제안하는 매니페스트에 대한 몇 개의 라이브러리를 사용하는 조그만 크로스-플랫폼 프로젝트 예제는 다음과 같다 :

```swift 
// 패키지를 선언하기 위한 API를 임포트 한다.
import PackageDescription
    
// 패키지를 선언한다.
let package = Package(
    // 패키지의 이름 (기본값은 소스 루트 디렉토리의 이름).
    name: "Foo",

    // 패키지의 타겟 리스트.
    targets: [
        // 메인 애플리케이션 선언.
        Target(
            name: "Foo",
            // 애플리케이션의 타입을 선언.
            type: .Tool,
            // 이 타겟은 패키지의 배포되는 프러덕트임을 선언
            // (내부 라이브러리나 툴이 아님).
            published: true),
        
        // 지원 라이브러리인 "CoreFoo" 에 대한 정보를 더한다.(CoreFoo/**/*.swift 에 있는 컨벤션 기반의 시스템에 의해 찾을 수 있는 ).
        Target(
            name: "CoreFoo",
            depends: [
                // 이 라이브러리는 "Utils" 타겟에 항상 의존한다.
                "Utils",
                
                // 이 라이브러리는 Linux 환경에서는 "AccessibilityUtils" 에 의존한다.
                .Conditional(name: "AccessibilityUtils", platforms: [.Linux])
            ]),
    
        // 노트: 컨벤션 기반의 시스템에 의해, "Utils" 타겟이 있다는 것을 추론한다.
        // 하지만, 기본 값들은 괜찮기 때문에 이것을 전혀 수정할 필요가 없다.
    
        // Linux 인 경우에만 "AccessibilityUtils" 타겟을 선언한다.
        Target(name: "AccessibilityUtils", platforms: [.Linux])
	])
```

*NOTE: 이 예제는 설명을 위한 것이므로, 정확한 API들은 변경될 것이다.*

매니페스트를 스위프트로 작성함에 따라, 소스코드를 작성하는 것 뿐만 아니라, 프로젝트 메타데이터를 만드는 데까지 일관된 개발 경험을 보장한다. 이것은 개발자가 모든 개발 편의 사항을 가지고 있는 일관된 환경을 가지게 됨을 의미한다 : 문법 컬러링, 코드 완성, API 문서, 그리고 포매팅 도구까지. 이것은 또한 스위프트에 낯선 개발자들이 스위프트 언어와 그 툴들을 배우는 데 집중하고 다른 커스텀 패키지 디스크립션 포맷을 신경쓰지 않도록 해 준다.

패키지 기술 그 자체는 컨벤션 기반 시스템에 *추가적인* 정보에 대한 선언적인 정의이다. 프로젝트에 사용될 실제 패키지 선언은 컨벤션 기반의 패키지 정의와 기본 비헤비어들을 오버라이드 하거나 커스터마이즈 하기 위한 패키지 디스크립션으로 이루어져 있다.

예를 들어, 아래 타켓 설명은

```swift
Target(name: "AccessibilityUtils", platforms: [.Linux])
```

새로운 타켓을 더하지 *않는다*. 대신, 이것은 기존의 `AccessibilityUtils`타겟이 어느 플랫폼에서 사용가능한지를 지정하기 위해 수정한다.


## 커스터마이즈하기

우리는 패키지 정의 선언이 컨벤션 기반의 시스템을 수정하는 80% 이상의 사용 케이스를 커버하기 위해 만들었다. 그럼에도 불구하고, 순수한 선언 모델에서는 인코딩하기 어렵거나 귀찮은 어떤 종류의 적절한 프로젝트 구조가 있다. 예를 들어, 사용자가 자신의 소스코드를 나눠주길 원하는 방식을 만족시키는  일반적인 목적의 매커니즘을 디자인 하는 것은 어렵다.

대신, 우리는 사용자들에게 `Package` 오브젝트와 네이티브 Swift API를 이용해 인터렉트할 수 있도록 했다. 파일내의 패키지 디스크립션에는 패키지를 설정하는 자연스러운 명령형 스위프트 API 코드를 추가할 수 있다. 예를 들어, 아래는 어떤 파일이 체크되지 않은 최적화로  빌드될지를 선택하는 커스텀 컨벤션을 사용하는 프로젝트이다.

```swift
import PackageDescription
    
let package = Package(name: "FTW")
    
// MARK: 커스텀 설정
    
// 릴리즈 모드에서 모든 *_unchecked.swift 파일들을 "-Ounchecked"를 사용해서 빌드한다.
for target in package.targets {
    for source in target.sources {
        if source.path.hasSuffix("_unchecked.swift") {
            source.customFlags += [.Conditional("-Ounchecked", mode: .Release)
        }
    }
}
```

기억해야 할 중요한 것은, 이 기능을 사용한다하더라도 패키지 매니페스트는 여전히 선언적**이어야 한다.** 이것은 매니페스트의 유일한 출력물은 패키지의 완전한 디스크립션이며, 패키지 매니저와 빌드 툴들에 의해 작동되어야 한다. 예를 들어, 매니페스트는 빌드 결과물과 어떤 직접적인 인터렉션을 하려고 시도하지 **않아야 한다.** 그런 모든 인터렉션들은 패키지 매니저 라이브러리에 의해 문서화되고 공개된  API를 거쳐야 하며, 패키지 매니저 툴에 의해 표면화 되어야 한다. 


## 편집기 지원

The package definition format being written in Swift is problematic for tools that wish to perform automatic updates to the file (for example, in response to a user action, or to bind to a user interface), or for situations where dealing with executable code is problematic.

To that end, the declarative package specification portion of the file is "Swift" in the same sense that "JSON is Javascript". The syntax itself is valid, executable, Swift but the tools that process it will only accept a restricted, declarative, subset of Swift which can be statically evaluated, and which can be unambiguously, automatically rewritten by editing tools. We do not intend to define a "standard" syntax for "Swift Object Notation", but we intend to accept a natural restriction of the language which only accepts literal expressions. We do intend to allow the restricted subset to take full advantage of Swift's rich type inference and literal convertable design to allow for a succinct, readable, and yet expressive syntax.

The customization section above will *not* be written in this syntax. Instead, the customization section will be clearly demarcated in the file. The leading file section up to the first '// MARK:' will be processed as part of the restricted declarative specification. All subsequent code **must be** honored by tools which only need to consume the output of the specification, and **should be** displayed by tools which present an editor view of the manifest, but **should not** be automatically modified. The semantics of the APIs will be specifically designed to accommodate the expected use case of editor support for the primary data with custom project-specific logic for special cases.

All tools which process the package manifest **must** validate that the declaration portion of the specification fits into the restricted language subset, to ensure a consistent user experience.


## 구현

We need to have efficient, programmatic access to the data from the manifest for use in the package manager and associated tools. Additionally, we may wish to use this data in contexts where the executable-code nature of the manifest format is problematic. On the other hand, we also want the file format to properly match the Swift language.

To satisfy these two goals, we intend to extract the package metadata from the manifest file by using the Swift parser **and** type checker to parse the leading package declaration portion of the file (not including any customizations made subsequent to the package definition). Once type checked, we will then validate the AST produced for the package description using custom logic which validates that the AST can (a) be parsed into validate model objects without needing to execute code, and (b) is written following the strict format such that it can be automatically modified by editing tools.

Tools that do not need to be as strict with the manifest format will be able to load it by using Swift directly to execute the file and then interact with the package definition API to extract the constructed model.


## 논의

We decided to use a Swift-based format for the manifest because we believe it gives developers the best experience for working with and describing their project. The primary alternative we considered was to use a declarative format encoded in a common data format like JSON. Although that would simplify implementation of the tooling around the manifest, it has the downside that users must then learn this additional language, and the development of high quality tools for that (documentation, syntax coloring, parsing diagnostics) isn't aligned with our goal of building great tools for Swift. In contrast, using the Swift language means that we can leverage all of the work on Swift to make those tools great.

The decision to use a restricted subset of Swift for the primary package definition is because we believe it is important that common tasks which require the manifest be able to be automated or surfaced via a user interface.

We decided to allow additional customization of the package via imperative code because we do not anticipate that the convention based system will be able to cover all possible uses cases. When users need to accommodate special cases, we want them to be able to do so using the most natural and expression medium, by writing Swift code. By explicitly designing in a customization system, we believe we will be able to deliver a higher quality set of core conventions -- there is an escape hatch for the special cases that allows us to focus on only delivering conventions (and core APIs) for the things that truly merit it.

A common problem with systems that permit arbitrary customization (especially via a programmatic interface) is that they become difficult to maintain and evolve, since it is hard to predict how developers have taken advantage of the interface. We deal with this by requiring that the manifest only interact with the package and tools through a strict, well defined API. That is, even though we allow developers to write arbitrary code to construct their package, we do not allow arbitrary interactions with the build process. Viewed a different way, the output of *all* manifests **must be** able to be treated as a single declaration specification -- even if part of that specification was programmatically generated.
