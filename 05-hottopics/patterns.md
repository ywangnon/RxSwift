# Patterns

## 🏛️ Architectural Patterns with RxSwift — MVVM, Coordinator, DI

> **“Rx는 패턴의 접착제.”** Reactive 스트림은 View‑Model 바인딩, 화면 전환, 의존성 주입을 깨끗하게 연결해 줍니다. 여기서는 iOS에서 자주 쓰는 세 가지 아키텍처 레이어와 Rx 결합 패턴을 살펴봅니다.

***

### 1️⃣ MVVM × Rx — 정석 패턴

#### 구조

```
View (UIKit / SwiftUI) ↔︎ ViewModel ↔︎ Model / Service
           ▲               ▲
           │ RxCocoa Bind  │ Observable / Single
```

#### ViewModel Skeleton

```swift
final class LoginViewModel {
    struct Input {
        let email: Observable<String>
        let password: Observable<String>
        let loginTap: Observable<Void>
    }
    struct Output {
        let isLoginEnabled: Driver<Bool>
        let loginResult: Signal<Result<User, Error>>
    }

    func transform(input: Input) -> Output {
        let creds = Observable.combineLatest(input.email, input.password)
        let enabled = creds.map { !$0.0.isEmpty && $0.1.count > 5 }
            .asDriver(onErrorJustReturn: false)

        let result = input.loginTap
            .withLatestFrom(creds)
            .flatMapLatest(api.login)
            .materialize()
            .asSignal()

        return Output(isLoginEnabled: enabled, loginResult: result)
    }
}
```

**포인트**

* Input은 **Observable/Signal** 로, Output은 **Driver/Signal** 로 노출해 메인 스레드·에러 무시 보장.
* Model 로직(네트워크) → Service 계층에서 `Single` · `Completable` 활용.

***

### 2️⃣ Coordinator + Rx — 화면 흐름 & 의존성 제거

#### Push 스타일 예제

```swift
final class AppCoordinator: Coordinator {
    let window: UIWindow
    private let bag = DisposeBag()

    func start() {
        showLogin()
    }

    private func showLogin() {
        let vm = LoginViewModel()
        let vc = LoginViewController(vm)

        vc.output.loggedIn
            .subscribe(onNext: { [weak self] user in
                self?.showHome(user)
            })
            .disposed(by: bag)

        rootNav.setViewControllers([vc], animated: false)
    }
}
```

* **VC→Coordinator** 방향으로 이벤트를 Subject/Signal 로 전달.
* Coordinator는 **viewController를 소유**하므로 DisposeBag 스코프 관리.

***

### 3️⃣ Dependency Injection (DI) with Rx

```swift
protocol AuthService {
    func login(email:String, pwd:String) -> Single<User>
}

final class ProdAuthService: AuthService { ... }
final class StubAuthService: AuthService { ... }

// Swinject ▶︎
container.register(AuthService.self) { _ in ProdAuthService() }

// ViewModel에 주입
class LoginViewModel {
    private let auth: AuthService
    init(auth: AuthService) { self.auth = auth }
}
```

> 테스트에서 `StubAuthService` + `TestScheduler` 사용으로 **단위 테스트** 용이.

***

### 4️⃣ Best‑Practice Checklist ✅

* [ ] &#x20;ViewController는 **입력·출력 Struct만** 다루고, 비즈니스 로직 X.
* [ ] &#x20;ViewModel → Output은 `Driver` / `Signal`, CollectionView는 `SectionModel`.
* [ ] &#x20;Coordinator에서 **화면 이벤트**(ex. loggedIn)만 받아, 내부 흐름 제어.
* [ ] &#x20;Service 계층 함수는 `Single` 또는 `Completable`로 신뢰성 확보.

***

### 5️⃣ Common Pitfalls

| 문제                          | 증상                      | 해결                                      |
| --------------------------- | ----------------------- | --------------------------------------- |
| Output을 `Observable` 그대로 노출 | UI 스레드 오류, 에러 전파        | `asDriver()`로 래핑                        |
| Coordinator retain cycle    | 화면 닫혀도 Coordinator 안 해제 | `[weak self]` or `childCoordinators` 관리 |
| 테스트에서 Timer·Interval        | 시간이 실 실행보다 느림           | `TestScheduler`로 가상 시간                  |

***

### 6️⃣ Mini Quiz

1. MVVM에서 **두 개 ViewModel** 간 데이터 공유가 필요할 때 Rx로 연결하는 방법?
2. Coordinator 패턴에서 화면 전환 결과를 ViewModel이 알 필요가 있을 때 의존성 역전 방법은?
3. `Single` vs `Completable` 사용 시점 구분?

<details>

<summary>Answers</summary>

1. `PublishRelay` 또는 `BehaviorRelay`를 **공용 DI Container**에 주입하거나, 상위 Coordinator에서 Signal 파이프 두 ViewModel에 주입.
2. Coordinator가 ViewModel에 **Subject(ObserverType) 주입** → ViewModel이 next 이벤트로 의사전달; 혹은 `CoordinatorOutput` Protocol + DelegateRx.
3. **Single** : _성공 값 1개_ + Completion, **Completable** : _값 없음_ — 예) 파일 다운로드(결과 파일) → Single, Database Migration 완료 플래그 → Completable.

</details>

***

> HotTopics 섹션 완결! 필요 시 각 패턴의 상세 예제나 테스트 코드 작성도 도와드릴 수 있습니다. 🚀
