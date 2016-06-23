# 패키지 개발하기

Simply put: a package is a git repository with semantically versioned tags,
that contains Swift sources and a `Package.swift` manifest file at its root.

## 라이브러리 모듈을 외부 패키지로 만들기 

If you are building an app with several modules, at some point you may decide to
make that module into an external package. Doing this makes that code available
as a dependable library that others may use.

Doing so with the package manager is relatively simple:

 1. Create a new repository on GitHub
 2. In a terminal, step into the module directory
 3. `git init`
 4. `git remote add origin [github-URL]`
 5. `git add .`
 6. `git commit --message="…"`
 7. `git tag 1.0.0`
 8. `git push origin master --tags`

Now delete the subdirectory,
and amend your `Package.swift` so that its `package` declaration includes:

```swift
let package = Package(
    dependencies: [
        .Package(url: "…", versions: Version(1,0,0)..<Version(2,0,0)),
    ]
)
```

Now type `swift build`


## 앱과 패키지를 나란히 작업하기
If you are developing an app that consumes a package
and you need to work on that package simultaneously
then you have several options:

 1. **Edit the sources that the package manager clones**

    The sources are cloned visibly into `./Packages` to facilitate this.

 2. **Alter your `Package.swift` so it refers to a local clone of the package**

    This can be tedious however as you will need to force an update every time you make a change, including updating the version tag.

Both options are currently non-ideal since it is easy to commit code that will break for other
members of your team, for example, if you change the sources for `Foo` and then commit a change to
your app that uses those new changes but you have not committed those changes to `Foo` then you have
caused dependency hell for your co-workers.

It is our intention to provide tooling to prevent such situations,
but for now please be aware of the caveats.

## 패키징 레거시 코드 

You may be working with code that builds both as a package and not.  For example, you may be packaging a project that also builds with Xcode.

In these cases, you can use the build configuration `SWIFT_PACKAGE` to conditionally compile code for Swift packages.

```swift
#if SWIFT_PACKAGE
import Foundation
#endif
```

