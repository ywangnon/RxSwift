# Schedulers

## 🧭 Schedulers — RxSwift의 ‘실행 컨텍스트’ 관리

> **“subscribeOn은 시작점을, observeOn은 관찰 지점을 바꾼다.”**

`SchedulerType`은 Observable 연산이 **어느 스레드/큐에서 실행될지** 결정합니다. _동시성 코&#xB4DC;_&#xB3C4; _선언형 체&#xC778;_&#xC5D0; 녹여 가독성과 안전성을 높일 수 있습니다.

***

### 1️⃣ 주요 Scheduler 종류

| Scheduler                            | 기반                   | 용도 & 특징                                 |
| ------------------------------------ | -------------------- | --------------------------------------- |
| **MainScheduler**                    | 메인 쓰레드               | UI 업데이트 필수, `Drive`, `bind(to:)` 기본 값   |
| **CurrentThreadScheduler**           | 현재 호출 스택             | 디폴트(Cold Observable) — 재귀 시 스택 오버플로우 주의 |
| **SerialDispatchQueueScheduler**     | GCD Serial Queue     | 시리얼 작업, 데이터베이스 I/O, 파일 쓰기               |
| **ConcurrentDispatchQueueScheduler** | GCD Concurrent Queue | CPU‑bound 계산, 파싱, 이미지 필터                |
| **OperationQueueScheduler**          | `OperationQueue`     | 의존성·우선순위 제어 필요 시                        |
| **ImmediateSchedulerType**           | 즉시 실행                | 테스트·동기 작업 최적화                           |
| **TestScheduler**                    | 가상 시간                | 단위 테스트, Marble 테스트 구축                   |

***

### 2️⃣ subscribe(on:) vs observe(on:)

| 메서드              | 목적                               | 예시 비유                    |
| ---------------- | -------------------------------- | ------------------------ |
| `subscribe(on:)` | **생산 단계**(Observable 생성) 스케줄러 지정 | “대파 손질을 주방에서 시작”         |
| `observe(on:)`   | **소비 단계** 이후 체인 실행 스케줄러 변경       | “손질된 재료를 홀 서빙 테이블에서 마무리” |

```swift
URLSession.shared.rx.data(request: req)
    .subscribe(on: ConcurrentDispatchQueueScheduler(qos: .background)) // 네트워크 백그라운드
    .map(parseJSON)
    .observe(on: MainScheduler.instance) // 결과 UI 바인딩
    .bind(to: tableView.rx.items(cellIdentifier: "Cell")) { _, model, cell in
        cell.textLabel?.text = model.title
    }
    .disposed(by: bag)
```

***

### 3️⃣ 실전 패턴

#### A. 비동기 이미지 처리

```swift
imagePicker.rx.selectedImage
    .subscribe(on: ConcurrentDispatchQueueScheduler(qos: .userInitiated))
    .map(ImageProcessor.resize)
    .observe(on: MainScheduler.instance)
    .bind(to: imageView.rx.image)
    .disposed(by: bag)
```

#### B. CoreData 저장

```swift
saveTrigger
    .withLatestFrom(input)
    .observe(on: SerialDispatchQueueScheduler(qos: .utility))
    .flatMap(saveToCoreData)
    .observe(on: MainScheduler.instance)
    .subscribe(onNext: showToast)
    .disposed(by: bag)
```

***

### 4️⃣ TestScheduler — 가상 시간 단위 테스트

```swift
let scheduler = TestScheduler(initialClock: 0)
let hot = scheduler.createHotObservable([
    .next(100, 1), .next(200, 2), .completed(300)
])

let res = scheduler.start { hot.map { $0 * 2 } }
XCTAssertEqual(res.events, [
    .next(100, 2), .next(200, 4), .completed(300)
])
```

> 가상 시간으로 **동기 테스트** 작성 → 테스트 속도 ⬆️, 재현성 ⬆️.

***

### 5️⃣ Deadlock & Race Condition 방지

1. **메인 → 메인 재입력**: Observable 체인 중 다시 MainScheduler로 전환 시 중첩 DispatchQueue 호출 지양.
2. **Shared Mutable State**: SerialScheduler로 보호하거나 `Actor`(Swift Concurrency) 사용.
3. **subscribeOn 중복**: 가장 처음에 위치한 subscribeOn만 적용.

***

### 6️⃣ Best Practices Checklist ✅

* [ ] &#x20;Heavy 작업은 `subscribe(on:)` + ConcurrentScheduler, 결과 UI는 `observe(on: Main)`
* [ ] &#x20;DB/File I/O는 SerialScheduler로 순서 보장
* [ ] &#x20;라이브러리 Driver/Signal 사용 시 MainScheduler 보장 확인
* [ ] &#x20;Unit Test에서 `TestScheduler`로 시간·스레드 독립 확인

***

### 7️⃣ Mini Quiz

1. `observe(on:)`를 연속 두 번 호출하면 어떻게 동작할까?
2. `subscribe(on:)`으로 MainScheduler, `observe(on:)`으로 Background 지정 시 UI 작업을 하면 무슨 문제가?
3. `TestScheduler` 사용 시 Cold Observable과 Hot Observable의 차이는?

<details>

<summary>정답</summary>

1. **마지막 observe(on:)가 우선** — 체인 진행 중 가장 최근에 지정한 Scheduler가 이후 연산에 적용됩니다.
2. UI 업데이트가 **백그라운드 스레드**에서 실행돼 크래시(`UI API called on background thread`) 위험.
3. TestScheduler에서
   * **Cold**: `createColdObservable` → 구독 시 이벤트 스케줄링 시작.
   * **Hot**: `createHotObservable` → 테스트 시간 0부터 이벤트 흐름, 구독 타이밍 따라 수신 이벤트 달라짐.

</details>

***

> 🎉 Schedulers 섹션 완료! 이제 Sequences 카테고리로 넘어가 스트림 생을 마스터해 보세요. 🚀
