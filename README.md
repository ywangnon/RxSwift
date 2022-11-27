# 개요

## 관련 사이트

[HomePage](http://reactivex.io/)

[Github](https://github.com/ReactiveX/RxSwift)

[RxSwiftCommunity Github](https://github.com/RxSwiftCommunity)



## Rx를 사용하는 이유

Rx를 사용했을 때, 좋은 이유를 찾아보면 여러가지 이유가 나옵니다. 그 중에서 내가 가장 와 닿은 이유를 적어보겠습니다.

1. **코드의 간결화**
   * 보통 Rx를 사용하려는 코드는 복잡하게 데이터를 주고 받는 코드들일 것입니다. 그런 코드들을 데이터의 흐름을 생각하여 작성할 수 있게 되어 코드가 보기 쉬워집니다.
2. **비동기 코드의 명확**
   * Rx 자체적으로 데이터를 제어해주어서 데이터의 흐름을 명확하게 볼 수 있게 됩니다. DispatchQueue나 OperationQueue를 사용하여 데이터가 어떻게 흘러갈지 복잡하게 제어하지 않아도 됩니다.
3. **콜백처리**
   * 콜백처리로 결과를 이곳저곳에 넘겨주며 그 흐름을 추적하면서 작업하지 않게 됩니다. 자연스럽게 결과를 가공하여 처리할 수 있게 됩니다.

