# Gesture & Keyboard

## 👆 제스처 & 키보드 RxCocoa 패턴 가이드

> **“UIKit 이벤트를 선언형으로 다루자.”**

이 문서에서는 \*\*제스처 인식기(Gesture Recognizer)\*\*와 **키보드 노티피케이션**을 RxCocoa로 우아하게 처리하는 방법을 한국어로 정리합니다.

***

### 1️⃣ UITapGestureRecognizer 바인딩

```swift
let tap = UITapGestureRecognizer()
view.addGestureRecognizer(tap)

tap.rx.event
    .withUnretained(self)
    .subscribe(onNext: { vc, _ in
        vc.view.endEditing(true) // 탭 시 키보드 내리기
    })
    .disposed(by: bag)
```

* `rx.event`는 `Observable<UITapGestureRecognizer>`
* `withUnretained`로 메모리 누수 방지
* 동일 패턴으로 **UISwipeGestureRecognizer, UIPanGestureRecognizer** 활용 가능

#### 한 번만 동작시키기

```swift
tap.rx.event
    .take(1)
    .bind(to: viewModel.firstTap)
    .disposed(by: bag)
```

***

### 2️⃣ RxGesture 라이브러리 (선택)

* [RxGesture](https://github.com/RxSwiftCommunity/RxGesture)는 `view.rx.tapGesture()` 등 간단 문법 제공
*   예시:

    ```swift
    view.rx.tapGesture()
        .when(.recognized)
        .subscribe(onNext: { _ in print("Tapped!") })
        .disposed(by: bag)
    ```

***

### 3️⃣ 키보드 높이 스트림화

```swift
extension Reactive where Base: UIView {
    var keyboardHeight: Observable<CGFloat> {
        let willShow = NotificationCenter.default.rx.notification(UIResponder.keyboardWillShowNotification)
            .map { ($0.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect)?.height ?? 0 }

        let willHide = NotificationCenter.default.rx.notification(UIResponder.keyboardWillHideNotification)
            .map { _ in CGFloat(0) }

        return Observable.merge(willShow, willHide)
            .distinctUntilChanged()
    }
}
```

#### 사용 예

```swift
view.rx.keyboardHeight
    .observe(on: MainScheduler.instance)
    .subscribe(onNext: { [weak self] height in
        self?.bottomConstraint.constant = height
        self?.view.layoutIfNeeded()
    })
    .disposed(by: bag)
```

***

### 4️⃣ 키보드 안전영역 애니메이션

```swift
view.rx.keyboardHeight
    .withLatestFrom(view.rx.layoutIfNeeded(), resultSelector: { h, _ in h })
    .bind(to: scrollView.rx.keyboardAnimatedInsets)
    .disposed(by: bag)
```

> `keyboardAnimatedInsets`는 커스텀 Extension으로, 애니메이션 곡선을 노티에서 추출해 사용

***

### 5️⃣ 메모리 관리 & 주의사항

| 항목        | 주의점                                         |
| --------- | ------------------------------------------- |
| 제스처 인식기   | View deinit 시 `gesture.view` retain 확인      |
| 키보드 노티    | `.takeUntil(self.rx.deallocated)`로 자동 해제 가능 |
| RxGesture | `when(.recognized)` 필터링 안 하면 중복 이벤트 발생      |

***

### 6️⃣ Mini Quiz

1. 두 손가락 탭 제스처를 Rx로 만들고, 첫 인식 후 자동 해제하려면?
2. 키보드 높이 스트림에서 높이가 동일해도 중복 이벤트가 발생한다면 어떻게 해결할까?
3. RxGesture 대신 기본 `rx.event`를 써야 하는 상황 한 가지는?

<details>

<summary>Answers</summary>

1.

    ```swift
    let tap2 = UITapGestureRecognizer()
    tap2.numberOfTouchesRequired = 2
    view.addGestureRecognizer(tap2)

    tap2.rx.event
        .take(1)
        .subscribe(onNext: { _ in print("double finger tap") })
        .disposed(by: bag)
    ```
2. `distinctUntilChanged()` 오퍼레이터를 추가하여 동일 값 필터링
3. RxGesture 미사용 프로젝트(의존성 최소화) 또는 커스텀 상태(`.began`, `.changed`) 등 세밀한 제어 필요할 때

</details>

***

> 🎉 Operators 챕터 끝! 이제 테스트 케이스 작성으로 학습을 강화해 보세요. 🚀
