# 패키지 매니저

스위프트 패키지 매니저는 소스코드의 배포를 위한 툴이다. 스위프트 빌드 시스템과 통합되어 다운로드하고, 컴파일하고, 의존성들을 링크라는 것을 자동화 한다.

패키지 매니저는 스위프트 3과 함께 배포될 예정이며 스위프트 3 개발 스냅샷과 함께 사용가능하다.

## 개념 오버뷰

이 섹션은 스위프트 패키지 매니저의 기능을  사용하기 위한 기본 개념을 설명한다.

###모듈
스위프트는 코드를  *모듈* 단위로 관리한다. 모듈들은 각자의 네임 스페이스를 정의하며 모듈의 코드 중 어떤 부분이 외부에서 접근가능할지에 대한 접근 권한을 설정한다.

프로그램은 모든 코드를 하나의 모듈에 가질 수도 있으며, 의존성에 의해 다른 모듈을 임포트 할 수도 있다. macOS의 다윈이나 Linux의 Glibc처럼 시스템이 제공하는 편리한 모듈들을 제외한 대부분의 의존성들은 사용하기 위해 코드를 다운로드하고 빌드하는 게 필요하다.

어떤 문제를 풀기 위해 별도의 모듈에 코드를 작성한다면, 그 코드는 다른 상황에서도 사용할 수 있다.  예를들어, 네트워크 요청을 만드는 기능을 제공하는 모듈은 사진 공유 앱과 날씨 예보를 보여주는 프로그램간에 공유될 수 있다. 모듈을 사용하는 건 동일한 기능을 직접 구현하기보다 다른 개발자의 코드 위에서 빌드하도록 해 준다.

###패키지
*패키지*는 스위프트 소스파일과 매니페스트 파일로 구성된다. 매니페스트 파일은, `Package.swift` 이름을 가지며, 패키지의 이름과 그 콘텐트를 `PackageDescription` 모듈을 이용해 정의한다.

패키지는 하나 이상의 타겟을 가진다. 각각의 타겟은 제품을 정의하고 하나 이상의 의존성을 설정할 수 있다.

###프러덕트
타겟은 그 프러덕트로 라이브러리 또는 실행파일을 빌드할 수 있다. *라이브러리*는 다른 스위프트 코드에 의해 임포트될 수 있는 모듈을 포함한다. *실행파일*은 OS에 의해 실행될 수 있는 프로그램이다.

###의존성
타겟의 의존성은 패키지 안의 코드에 의해 요구되는 모듈들이다. 의존성은 패키지에 대한 절대/상대 경로와 사용할 수 있는 패키지의 버전에 대한 요구사항의 집합이다. 패키지 매니저의 역할은 프로젝트를 위한 의존성들을 모두 다운로드하고 빌드하는 과정을 자동화 함으로서 그런 조정을 위한 비용을 줄이는 것이다. 이것은 재귀적인 프로세스이다 : 하나의 의존성은 그 자신이 의존성들을 가질 수 있으며, 그 각각은 또한 의존성들을 가지면서 의존성 그래프가 만들어질 수 있다. 패키지 매니저는 전체 의존성 그래프를 만족시키기 위해 필요한 모든 것들을 다운로드하고 빌드한다.

```
아래 섹션은 스위프트 작업 경험이 있다는 것을 전제한다. 
스위프트 언어에 처음이라면, 개론 리소스들 중 하나를 먼저 살펴봐야 할 것이다. 
'스위프트 프로그래밍 언어' 문서의  '스위프트 투어' 를 권한다.

만약 코드 예제를 따라해 보길 원한다면, 정상적으로 동작하는 스위프트가 설치되어 있어야 한다. 
스위프트 설치 방법은 '시작하기'에서 볼 수 있다.
```


## 예제 사용
'시작하기' 에서, 간단한 "Hello, world!" 프로그램이 스위프트 패키지 매니저로 빌드되었다.

스위프트 패키지 매니저가 할 수 있는 것을 보다 완전하게 보여주기 위해서, 아래의 예제는 4개의 상호의존적인 패키지로 구성되어 있다.

* PlayingCard - `PlayingCard` , `Suit` 그리고  `Rank` 타입을 정의
* FisherYates - `shuffle()`과 `shuffleInPlace()` 메소드를 구현하는 확장을 정의
* DeckOfPlayingCard - `PlayingCard` 값의 배열을 섞고 제공하는 `Deck` 타입을 정의.
* Dealer - `DeckOfPlayingCards`를 만들고, 섞고, 첫 10장의 카드를 제공하는 실행파일을 정의.

> GitHub에서 Dealer프로젝트의 소스코드를 다운로드 받은 뒤 아래 명령으로 실행하면 전체 예제를 빌드하고 실행할 수 있다.

```
$ cd example-package-dealer
$ swift build
$ .build/debug/Dealer
```

###라이브러리 패키지 만들기
우리는 표준 52장 카드 덱에 플레이하는 카드를 나타내는 모듈을 만드는 것으로 시작할 것이다. `PlayingCard` 모듈은 `PlayingCard` 타입을 정의하며, 이것은 `Suit` 이뉴머레이션 값 (클럽, 다이아몬드, 하트, 스페이드)과 `Rank` 이뉴머레이션 값 (에이스, 2, 3, ... , 잭, 퀸, 킹)으로 이루어진다.

```
public enum Rank : Int {
    case Ace = 1
    case Two, Three, Four, Five, Six, Seven, Eight, Nine, Ten
    case Jack, Queen, King
}

public enum Suit: String {
    case Spades, Hearts, Diamonds, Clubs
}

public struct PlayingCard {
    let rank: Rank
    let suit: Suit
}
```

컨벤션에 의해, 패키지가 가지는 모든 소스파일들은 `Sources/` 디렉토리안에 위치해야 한다.

```
example-package-playingcard
├── Sources
│   ├── PlayingCard.swift
│   ├── Rank.swift
│   └── Suit.swift
└── Package.swift
```

`PlayingCard` 모듈은 실행파일을 만들지 않기 때문에, *라이브러리*라고 부를 수 있다. 라이브러리는 다른 패키지들에 의해 임포트 되는 모듈을 빌드하는 패키지이다. 기본적으로, 라이브러리 모듈은 모든  `Sources/` 디렉토리안의 소스코드안에 선언되어 있는 `public` 타입과 메소드들을 노출시킨다.


`swift build`를 실행하여 스위프트 빌드 프로세스를 시작해 보자. 모든 것이 정상적으로 동작한다면, `.build/debug` 디렉토리 안에 `PlayingCard.a` 스태틱 라이브러리를 만들어 낸다.

```
example-package-playingcard
└── .build
    └── debug
        ├── PlayingCard.a
        ├── PlayingCard.o
        ├── PlayingCard.swiftdoc
        └── PlayingCard.swiftmodule
```

> `PlayingCard`패키지의 전체 코드는 [https://github.com/apple/example-package-playingcard](https://github.com/apple/example-package-playingcard) 에서 찾을 수 있다.


###빌드 설정문 사용하기
다음에 빌드할 모듈은 `FisherYates`이다. `PlayingCard`와 달리, 이 모듈은 새로운 타입을 전혀 정의 하지 않는다. 대신에, 존재하는 타입인 `CollectionType`과 `MutableCollectionType` 프로토콜을 확장해서 `shuffle()` 메소드와 그 mutating 쌍인 `shuffleInPlace()`를 추가한다.

`shuffleInPlace()` 구현은 Fisher-Yates 알고리듬을 사용해 컬렉션 내의 요소들을 랜덤하게 치환한다. 스위프트 표준 라이브러리가 랜덤 숫자 제너레이터를 제공하지 않으므로, 이 메소드는 시스템 모듈로부터 임포트한 함수를 호출해야 한다. 이 함수가 macOS와  Linux 양쪽에서 호환되도록, 코드는 빌드 설정문을 사용한다.

macOS에서, 시스템 모듈은 `Darwin`이며, `arc4random_uniform(_:) 함수를 제공한다. Linux에서, 시스템 모듈은 Glibc이며, `random()` 함수를 제공한다:

```
#if os(Linux)
import Glibc
#else
import Darwin.C
#endif

public extension MutableCollectionType where Index == Int {
    mutating func shuffleInPlace() {
        if count <= 1 { return }

        for i in 0..<count - 1 {
#if os(Linux)
            let j = Int(random() % (count - i)))) + i
#else
            let j = Int(arc4random_uniform(UInt32(count - i))) + i
#endif
            if i == j { continue }
            swap(&self[i], &self[j])
        }
    }
}
```

> `FisherYates`패키지의 전체 코드는 [https://github.com/apple/example-package-fisheryates](https://github.com/apple/example-package-fisheryates) 에서 찾을 수 있다.

### 의존성 불러오기

`DeckOfPlayingCards` 패키지는 이전 두개의 패키지들을 함께 가져온다 : 이것은 `FisherYates`의 `shuffle()`메소드를 `PlayingCards` 값들의 배열에 사용하는 `Deck` 타입을 정의한다.


`FisherYates`와 `PlayingCards` 모듈을 사용하기 위해서, `DeckOfPlayingCards` 패키지는 `Package.swift` 매니페스트파일에 그 패키지들을 의존성으로 선언해야 한다.

```
import PackageDescription

let package = Package(
    name: "DeckOfPlayingCards",
    targets: [],
    dependencies: [
        .Package(url: "https://github.com/apple/example-package-fisheryates.git",
                 majorVersion: 1),
        .Package(url: "https://github.com/apple/example-package-playingcard.git",
                 majorVersion: 1),
    ]
)
```
각각의 의존성은 소스 URL과 버전 요구사항을 지정한다. 소스 URL은 현재 사용자가 접근할 수 있는 Git 리포지터리 URL이다. 버전 요구사항은 Semantic Versioning(SemVer) 컨벤션을 따르며, 체크아웃 할 Git 태그를 결정해서 의존성을 빌드하는 데 사용한다. `FisherYates` 와 `PlayingCard` 양쪽 모두의 의존선에 있어서, 메이저 버전 중 가장 최신 버전인 `1`(예를 들어 `1.0.0`)이 사용될 것이다.

`swift build` 명령이 동작 하면, 패키지 매니저는 모든 의존성들을 다운로드하고, 정적 라이브러리로 컴파일한 뒤, 패키지 모듈에 링크한다. 이것은 `DeckOfPlayingCards`가 import 구문으로 의존하는 모듈들의 퍼블릭 멤버들에 접근할 수 있도록 한다.

프로젝트 루트 디렉토리 안의  `Packages` 디렉토리에서 다운로드받은 소스들을 볼 수 있으며, 중간 빌드 프로덕트들은 프로젝트 루트 디렉토리 안의 `.build` 디렉토리에서 찾아볼 수 있다.


```
example-package-deckofplayingcards
├── .build
│   └── debug
│       ├── DeckOfPlayingCards.a
│       ├── DeckOfPlayingCards.o
│       ├── DeckOfPlayingCards.swiftdoc
│       ├── DeckOfPlayingCards.swiftmodule
│       ├── FisherYates.a
│       ├── FisherYates.o
│       ├── FisherYates.swiftdoc
│       ├── FisherYates.swiftmodule
│       ├── PlayingCard.a
│       ├── PlayingCard.o
│       ├── PlayingCard.swiftdoc
│       └── PlayingCard.swiftmodule
└── Packages
    └── example-package-fisheryates-1.0.1
    │   ├── Package.swift
    │   ├── README.md
    │   └── Sources
    └── example-package-playingcard-1.0.0
        ├── Package.swift
        ├── README.md
        └── Sources
```

`Packages` 디렉토리는 패키지의 의존성에 대한 복제된 리포지터리를 포함하고 있다. 이를 통해 당신은 각각의 패키지에 대해 탑-레벨의 클론 없이도 소스코드를 변경한 경우 그 변경사항들을 각자의 오리진에 바로 푸시할 수 있다.

> `DeckOfPlayingCards`패키지의 전체 코드는 [https://github.com/apple/example-package-deckofplayingcards](https://github.com/apple/example-package-deckofplayingcards) 에서 찾을 수 있다.



###서브의존성 해결하기

다른 모든 것들은 준비되었으므로, 이제 `Dealer` 모듈을 빌드할 수 있다. `Dealer`모듈은 `DeckOfPlayingCards` 패키지에 의존하고 있기 때문에, `PlayingCards`와 `FisherYates` 패키지에도 의존하고 있다. 하지만 스위프트 패키지 매니저는 자동으로 서브의존성을 해결해 주기 때문에, `DeckOfPlayingCards` 패키지에 대해서만 의존성을 선언해도 된다.

```
import PackageDescription

let package = Package(
    name: "Dealer",
    targets: [],
    dependencies: [
        .Package(url: "https://github.com/apple/example-package-deckofplayingcards.git",
                 majorVersion: 1),
    ]
)
```
스위프트는 코드에서 참조되는 모든 타입에 대해 소스 파일이 모듈을 임포트 하길 요구한다. `Dealer` 모듈의 `main.swift`파일의 경우, `DeckOfPlayingCards`의 `Deck`타입과 `PlayingCard`의 `PlayingCard`타입이 참조된다. `Deck` 타입의 `shuffle()` 메소드가 내부적으로 `FisherYates` 모듈을 사용하고 있지만, 그 모듈은 `main.swift`에 임포트 되지 않는다.

```
import PlayingCard
import DeckOfPlayingCards

let numberOfCards = 10

var deck = Deck.standard52CardDeck()
deck.shuffle()

for _ in 1...numberOfCards {
    guard let card = deck.deal() else {
        print("No More Cards!")
        break
    }

    print(card)
}
```

컨벤션에 따르면, 루트 디렉토리에 `main.swift` 이름이 붙은 파일을 포함하는 패키지는 실행 파일을 만들어 낸다.

`swift build` 명령을 실행하면 스위프트 빌드 시스템이 `Dealer` 실행파일을 만들도록 하며, 그 실행 파일은 `.build/debug` 디렉토리에서 실행할 수 있다.

```
$ swift build
$ ./.build/debug/Dealer
♠︎6
♢K
♢2
♡8
♠︎7
♣︎10
♣︎5
♢A
♡Q
♡7
```


> `Dealer`패키지의 전체 코드는 [https://github.com/apple/example-package-dealer](https://github.com/apple/example-package-dealer) 에서 찾을 수 있다.



## 커뮤니티 제안
스위프트 패키지 매니저의 초기 배포판은 시작점일 뿐이며, 가장 좋은 툴을 만드는 데 당신이 함께 하면 좋겠다.

프로젝트를 시작하는 것을 돕기 위해, 다음의 커뮤니티 제안능 준비했다. 이것은 현재 구현에 대한 결정을 만드는 컨텍스트들을 제공하며, 미래 기능들의 개발에 대한 방향을 제공한다. 함께 하는 것에 관심이 있다면, 이 문서를 가장 먼저 읽어야 할 것이다.



