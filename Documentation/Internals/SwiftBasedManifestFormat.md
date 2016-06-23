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

  The manifest should be machine readable and writeable format. We envision a variety of tools that may want to inspect the contents of packages (for example, to build information for an index) or make automatic edits to the project structure. For example, when introducing a new library dependency via adding an import statement, we would like it if a tool could, after a user prompt, automatically update the manifest to specify the new dependency.


## 제안

We propose to use the Swift language itself to write the manifest. An example of a proposed manifest for a small cross-platform project with several libraries might look something like this:

```swift 
// This imports the API for declaring packages.
import PackageDescription
    
// This declares the package.
let package = Package(
    // The name of the package (defaults to source root directory name).
    name: "Foo",

    // The list of targets in the package.
    targets: [
        // Declares the main application.
        Target(
            name: "Foo",
            // Declare the type of application.
            type: .Tool,
            // Declare that this target is a published product of the package
            // (as opposed to an internal library or tool).
            published: true),
        
        // Add information on a support library "CoreFoo" (as found by the
        // convention based system in CoreFoo/**/*.swift).
        Target(
            name: "CoreFoo",
            depends: [
                // The library always depends on the "Utils" target.
                "Utils",
                
                // This library depends on "AccessibilityUtils" on Linux.
                .Conditional(name: "AccessibilityUtils", platforms: [.Linux])
            ]),
    
        // NOTE: There is a "Utils" target inferred by the convention based
        // system, but we don't need to modify it at all because the defaults
        // were fine.
    
        // Declare that the "AccessibilityUtils" target is Linux-specific.
        Target(name: "AccessibilityUtils", platforms: [.Linux])
	])
```

*NOTE: this example is for expository purposes, the exact APIs are subject to change.*

By writing the manifest in Swift, we ensure a consistent development experience across not only authoring their source code, but also their project metadata. This means developers will have a consistent environment with all of the development conveniences they expect: syntax coloring, code completion, API documentation, and formatting tools. This also ensures that new developers to Swift can focus on learning the language and its tools, not another custom package description format.

The package description itself is a declarative definition of information which *augments* the convention based system. The actual package definition that will be used for a project consists of the convention based package definition with the package description applied to override or customize default behaviors. For example, this target description:

```swift
Target(name: "AccessibilityUtils", platforms: [.Linux])
```

*does not* add a new target. Rather, it modifies the existing target `AccessibilityUtils` to specify what platforms it is available for.


## 커스터마이즈하기

We intend for the declaration package definition to cover 80%+ of the use cases for modifying the convention based system. Nevertheless, there are some kinds of legitimate project structures which are difficult or cumbersome to encode in a purely declarative model. For example, designing a general purpose mechanism to cover all the ways in which users may wish to divide their source code is difficult.

Instead, we allow users to interact with the `Package` object using its native Swift APIs. The package declaration in a file may be followed by additional code which configures the package using a natural, imperative, Swifty API. For example, this is an example of a project which uses a custom convention for selecting which files build with unchecked optimizations:

```swift
import PackageDescription
    
let package = Package(name: "FTW")
    
// MARK: Custom Configuration
    
// Build all *_unchecked.swift files using "-Ounchecked" for Release mode.
for target in package.targets {
    for source in target.sources {
        if source.path.hasSuffix("_unchecked.swift") {
            source.customFlags += [.Conditional("-Ounchecked", mode: .Release)
        }
    }
}
```

It is important to note that even when using this feature, package manifest still **must be** declarative. That is, the only output of a manifest is a complete description of the package, which is then operated on by the package manager and build tools. For example, a manifest **must not** attempt to do anything to directly interact with the build output. All such interactions must go through a documented, public API vended by the package manager libraries and surfaced via the package manager tools.


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
