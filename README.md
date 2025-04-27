# 개요

> **하루 10분, 리액티브 사고방식을 익혀보세요!**

***

## 📌 빠른 링크

|                     | 링크                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------ |
| 🌐 공식 홈페이지          | [ReactiveX.io](http://reactivex.io/)                                                       |
| 🐙 RxSwift GitHub   | [github.com/ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift)                       |
| 🤝 RxSwiftCommunity | [github.com/RxSwiftCommunity](https://github.com/RxSwiftCommunity)                         |
| 📖 GitBook          | [gitbook/RxSwift](https://ywangnon123.gitbook.io/rxsw)                                     |
| 🏗️ 연습용 플레이그라운드     | [RxSwift\_GitBook\_Playground](https://github.com/ywangnon/RxSwift_GitBook_Playground.git) |

***

## ✨ 왜 RxSwift인가?

> "비동기의 복잡함을 **선언형 코드** 한 줄로 다스린다!"

1. **코드 간결화**
   * map · filter · zip 등 풍부한 오퍼레이터로 반복되는 보일러플레이트 제거
2. **명확한 데이터 흐름**
   * 💧 _Observable → Operator → Observer_ 흐름이 명확해 디버깅‧유지보수 용이
3. **자연스러운 비동기 처리**
   * GCD, `completion` 지옥에서 탈출
   * 스케줄러로 스레드 전환을 한눈에 파악
4. **강력한 콤포지션**
   * 여러 비동기 스트림을 하나로 결합(combineLatest, merge, concat…)해 복잡한 UI 상태를 단순화
5. **테스트 용이성**
   * `TestScheduler`로 가상 시간 기반 단위 테스트 가능

> 📖 _Rx 마블 다이어그&#xB7A8;_&#xC73C;로 오퍼레이터 작동 방식을 시각적으로 학습해 보세요!

***

## ⚙️ 설치하기

### Swift Package Manager (추천)

```swift
// swift-tools-version: 5.9

import PackageDescription

let package = Package(
    name: "RxPractice",
    platforms: [ .iOS(.v15) ],
    dependencies: [
        .package(url: "https://github.com/ReactiveX/RxSwift.git", from: "6.6.0")
    ],
    targets: [
        .target(
            name: "RxPractice",
            dependencies: ["RxSwift", "RxCocoa"])
    ]
)
```

> 👉 **Xcode**: `File` ▸ `Add Packages...` 에서 URL 입력 후 `Up to Next Major` 선택

***

## 🏃‍♂️ 빠른 시작

1.  **Playground 클론**

    ```bash
    git clone https://github.com/ywangnon/RxSwift_GitBook_Playground.git
    open RxSwift_GitBook_Playground/RxSwiftPractice.xcodeproj
    ```
2. **샘플 시나리오**
   * 📰 _Networking_ : URLSession을 Rx 확장으로 감싸기
   * 🔍 _SearchBar_ : `UITextField`를 `rx.text`로 바인딩
   * 🎵 _Music Player_ : `Observable.combineLatest`로 UI 상태 동기화
3. **미션**
   * `Throttle`, `Debounce` 차이를 실습해보기
   * `flatMapLatest` vs `switchMap` 비교

> 💡 _Hint_: `RxTimeInterval.milliseconds(300)`로 UI 이벤트 디바운싱

***

## 📁 프로젝트 구조

```
RxSwift_GitBook_Playground
├── Sources
│   ├── NetworkService.swift
│   ├── SearchViewModel.swift
│   └── MusicPlayerViewModel.swift
└── Tests
    ├── SearchViewModelTests.swift
    └── MusicPlayerViewModelTests.swift
```

***

## 🔖 더 읽어보기

* [RxSwift 공식 문서](https://github.com/ReactiveX/RxSwift/tree/main/Documentation)
* [RxMarbles(인터랙티브 오퍼레이터 시뮬레이터)](http://rxmarbles.com/)

***

## 🗒️ 기여 가이드

1. **Fork → PR**: 예제 추가나 오타 수정 환영합니다!
2. 커밋 메시지 컨벤션: `[Add]`, `[Fix]`, `[Docs]`, `[Refactor]`
3. Issue 템플릿에 따라 재현 코드/스크린샷 포함 부탁드립니다.

***

> "**ReactiveX**는 언어가 아니라 _패러다&#xC784;_&#xC785;니다. 오늘부터 선언형 스트림으로 앱의 복잡성을 줄여보세요!"

***

© 2025 HS Yoo · Built with ❤️ & RxSwift
