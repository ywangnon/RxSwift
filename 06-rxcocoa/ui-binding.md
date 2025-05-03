# UI Binding

## 🎛️ RxCocoa UI Binding — From Events to Declarative Updates

> **"**“타깃-액션은 안녕, 이제는 **`Rx`**&#xC640; 함께!”

RxCocoa는 UIKit 컴포넌트에 **Reactive extensions**를 제공해 터치·텍스트·스크롤 등 이벤트 스트림을 쉽게 구독하고, 속성을 바인딩할 수 있습니다. 이 문서에서는 가장 많이 쓰이는 `UIButton.rx.tap`, `UITextField.rx.text`, `UITableView.rx.items` 바인딩 패턴을 정리합니다.

***

### 1️⃣ UIButton — Reactive Tap

```swift
loginButton.rx.tap
    .throttle(.milliseconds(500), scheduler: MainScheduler.instance)
    .bind(to: viewModel.loginTap)
    .disposed(by: bag)
```

* `rx.tap` = `Observable<Void>`
* Tap debounce/throttle로 중복 클릭 방지
*   **withUnretained(self)** 활용 예:

    ```swift
    button.rx.tap
        .withUnretained(self)
        .subscribe(onNext: { vc, _ in vc.dismiss(animated:true) })
    ```

***

### 2️⃣ UITextField & UILabel — Two‑way Binding

```swift
// View → ViewModel
emailField.rx.text.orEmpty
    .bind(to: viewModel.email)
    .disposed(by: bag)

// ViewModel → View
viewModel.email
    .bind(to: emailField.rx.text)
    .disposed(by: bag)
```

* `.orEmpty` : Optional String → String 변환
* RxCocoa에는 내장된 양방향 바인딩 도우미가 없으므로, 복잡한 경우에는 `BindRelay` 또는 사용자 정의 연산자를 사용하는 것을 고려하세요.

***

### 3️⃣ UITableView — items(cellIdentifier:)

#### A. Simple Static Cell

```swift
dataObservable
    .bind(to: tableView.rx.items(cellIdentifier: "Cell")) { index, model, cell in
        cell.textLabel?.text = model.title
    }
    .disposed(by: bag)
```

#### B. Sectioned Table (RxDataSources)

```swift
let dataSource = RxTableViewSectionedReloadDataSource<SectionModel<String, Item>>(configureCell: ...)

sectionsObservable
    .bind(to: tableView.rx.items(dataSource: dataSource))
    .disposed(by: bag)
```

* `RxDataSources`는 변경 사항을 인식하는 `animated` 처리 드라이버를 추가합니다.

***

### 4️⃣ CollectionView Quick Note

```swift
items.bind(to: collectionView.rx.items(cellIdentifier: "Cell", cellType: MyCell.self)) { row, item, cell in
    cell.configure(item)
}.disposed(by: bag)
```

***

### 5️⃣ ControlProperty & Driver

| Feature   | `ControlEvent` | `ControlProperty` | `Driver`       |
| --------- | -------------- | ----------------- | -------------- |
| Source    | UI 이벤트 (tap)   | UI 속성 (text)      | Any Observable |
| Error     | never          | never             | never          |
| Scheduler | Main           | Main              | Main           |

> UI Binding Output은 **Driver** 권장 (Main thread, shareReplay(1))

***

### 6️⃣ Memory & Thread Safety

* RxCocoa는 자동으로 **MainScheduler**에서 구독합니다.
* 바인딩 확장 함수(`bind(to:)`)는 대상 객체가 해제될 때 자동으로 dispose됩니다.

***

### 7️⃣ Mini Quiz

1. `rx.tap`을 `Driver`로 변환하려면?
2. `tableView.rx.modelSelected`의 메모리 주의점?
3. `rx.text`가 Optional인 이유는?

<details>

<summary>Answers</summary>

1. `button.rx.tap.asDriver()` — 에러가 발생하지 않기 때문에 `onErrorJustReturn(())`은 불필요합니다.
2. 선택 흐름(select stream)은 셀을 유지(retain)할 수 있으므로, 사용 후 `.bind(onDisposed:)` 또는 `withUnretained`를 사용해 순환 참조를 방지하세요.
3. `UITextField`의 텍스트는 placeholder가 표시될 때 `nil`일 수 있으므로, `.orEmpty`를 사용해 비옵셔널 문자열로 변환하세요.

</details>

***

> 다음 ▶️ **gesture\_keyboard.md** 로 이동해 Gesture Recognizer & Keyboard reactive 패턴을 배워봅시다. 🚀
