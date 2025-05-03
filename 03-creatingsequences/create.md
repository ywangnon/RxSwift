# create

## 🏗️ `Observable.create` — 커스텀 시퀀스를 직접 설계하기

> **“Observable을 DIY 하자!”** 수많은 팩토리 메서드로도 부족할 때, `create`를 사용해 **임의의 방식으로 onNext/onCompleted/onError**를 제어할 수 있습니다.

***

### 1️⃣ 시그니처 & 기본 구조

```swift
Observable<Element>.create { observer -> Disposable in
    // 1. 이벤트 방출
    observer.onNext(...)
    observer.onCompleted() // 또는 onError

    // 2. 정리(해제) 로직 반환
    return Disposables.create {
        // cancel task, invalidate timer, deallocate resource
    }
}
```

* **observer**: `AnyObserver<Element>` — `.onNext`, `.onError`, `.onCompleted` 호출 가능.
* **Disposable**: 구독이 dispose 될 때 호출될 정리 클로저.

***

### 2️⃣ 예제 — Alamofire 없는 간단 HTTP GET

```swift
func getRequest(url: URL) -> Observable<Data> {
    return Observable.create { observer in
        let task = URLSession.shared.dataTask(with: url) { data, _, error in
            if let err = error {
                observer.onError(err)
            } else if let d = data {
                observer.onNext(d)
                observer.onCompleted()
            }
        }
        task.resume()

        return Disposables.create {
            task.cancel() // dispose 시 network cancel
        }
    }
}
```

* 구독자에게는 **Cold Observable**(매 구독마다 새 요청)로 동작.

***

### 3️⃣ 주의사항 체크리스트

| 체크                  | 설명                                     |
| ------------------- | -------------------------------------- |
| ✅ `observer` 이벤트 순서 | `onNext*` → `onCompleted/onError` 단 1회 |
| ✅ 스레드               | 호출 스레드를 명시하거나 `subscribeOn` 활용         |
| ✅ Dispose 처리        | 비동기 작업 cancel / 타이머 invalidate         |
| ❌ retain cycle      | `observer` 클로저 내부 `[weak self]` 필요 시   |

***

### 4️⃣ `Single.create` · `Maybe.create`

* **Single**: 성공/실패 둘 중 하나만 방출 (`SingleEvent.success`, `.failure`).
* **Maybe**: 성공(값)/완료(값 없음)/실패 세 가지.

```swift
Single<String>.create { single in
    fetch(id: 1) { result in
        switch result {
        case .success(let str):
            single(.success(str))
        case .failure(let err):
            single(.failure(err))
        }
    }
    return Disposables.create()
}
```

***

### 5️⃣ Mini Quiz

1. `Observable.create` 내부에서 `.onNext`를 두 번 연속 호출 후 `.onCompleted()` 없이 함수를 반환하면 무슨 일이 일어날까?
2. 뷰컨트롤러 deinit 전에 네트워크 요청을 취소하려면 반환 Disposable에 어떤 코드를 넣어야 할까?
3. `Single.create`와 `AsyncSubject`를 비교했을 때 차이점은?

<details>

<summary>Answers</summary>

1. 스트림이 **완료되지 않아** 구독자가 `.completed` 이벤트를 수신하지 못하고, 작업이 영구히 열려 있을 수 있다. (Dispose 호출 시까지 유출)
2. `task.cancel()` 과 같이 URLSessionTask를 취소하거나, alamofireRequest.cancel() 호출 등을 넣어 Dispose 시 네트워크 종료.
3. `Single`은 **Cold**이고 subscribe 시 생성·방출·완료를 스스로 처리하지만, `AsyncSubject`는 **Hot**으로 외부에서 `onNext`/`onCompleted` 제어한다.

</details>

***

> 다음 ▶️ **just, of, from** 로 이동해 손쉬운 시퀀스 팩토리 메서드 `just`, `of`, `from`을 학습합니다. 🚀
