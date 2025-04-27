# Setup

## ⚙️ Setup — RxSwift 6.9 환경 구축

> **목표:** 5분 만에 RxSwift를 프로젝트·Playground에 통합해 첫 스트림을 실행합니다.

***

### 1️⃣ 종속성 관리 도구별 설치

#### 🚀 Swift Package Manager (권장)

**A. Xcode GUI 경로**

1. **File ▸ Add Packages…** 선택
2. URL 입력 👉 `https://github.com/ReactiveX/RxSwift.git`
3. Version Rule: _Up to Next Major_ (`6.9.0` 이상)

> 🔄 저장 후 `⌘B` 빌드하여 패키지를 불러옵니다.

***

#### 📦 CocoaPods (레거시 프로젝트 호환)

1. 터미널에서 프로젝트 루트 이동
2. `pod init` 후 _Podfile_ 열기
3.  아래 줄 추가

    ```ruby
    pod 'RxSwift', '~> 6.9'
    pod 'RxCocoa', '~> 6.9'
    ```
4. `pod install --repo-update`
5. `.xcworkspace` 열어 확인

> ✅ Apple Silicon(M1‑M4)에서도 Rosetta가 _필요 없도록_ CocoaPods 2.0부터 네이티브 arm64 gem을 제공합니다.

***

### 2️⃣ Playground 세팅 (실습용)

1. `File ▸ New ▸ Playground… ▸ Blank` 선택 → **Platform**: iOS
2. 상단 메뉴 **File ▸ Add Packages…** 로 RxSwift 6.9 추가
3.  Playground 코드 예시

    ```swift
    import RxSwift
    import RxCocoa

    let disposeBag = DisposeBag()

    Observable.just("Hello Rx 6.9 ✨")
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    ```
4. `⌥⌘P`(Run)로 출력 확인 — "Hello Rx 6.9 ✨"

💡 _Tip_: 반복 빌드 시간을 줄이려면 Playground _Build Active Scheme_ 옵션 OFF.

***

### 3️⃣ 흔한 빌드 오류 & 해결

| 에러 메시지                                                                        | 원인                 | 해결 방법                                                     |
| ----------------------------------------------------------------------------- | ------------------ | --------------------------------------------------------- |
| `Missing package product 'RxCocoa'`                                           | 종속성 선택 누락          | _Package Dependencies_ 탭에서 재체크 후 `⌘B`                     |
| `RxSwift/Observable.swift: Module compiled with Swift 5.x cannot be imported` | Xcode·Swift 버전 불일치 | Xcode 16로 업그레이드 또는 패키지 재빌드                                |
| `dyld: Library not loaded: RxSwift`                                           | CocoaPods Embed 누락 | _Frameworks, Libraries & Embedded Content_ > Embed & Sign |

***

### 4️⃣ 다음 단계

👉 **Core Concepts** 챕터로 넘어가 `Observable`·`Observer` 첫 실습을 진행해 보세요.
