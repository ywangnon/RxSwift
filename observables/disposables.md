# Disposables

* 옵저버블이 전달하는 이벤트는 아님.
* 옵저버블과 관련된 리소스가 해제되는 시점에서 자동으로 실행
* 따로 실행하고 싶은 코드가 있으면 onDisposed에 작성해야 함. 작성하지 않아도 자동으로 실행되어 메모리 해제하고 있음
* 공식적으로는 직접 입력해서 메모리 정리하는 게 좋음
* DisposeBag에 담아서 처리. disposebag 해제되는 시점에서 해제 됨. arc의 auto releasepool과 같음.
* dispose 되는 즉시 메모리에서 해제됨. complete 작동 안함.
