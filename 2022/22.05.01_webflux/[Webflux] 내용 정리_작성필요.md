# [Webflux] 내용 정리



Flux<T>는 Flux 시퀀스를 생성, 변환, 조정하는 데 사용할 수 있는 많은 연산자로 보강된 Reactive Streams Publisher입니다.

0에서 n개의 <T> 요소(`onNext` 이벤트)를 내보낸 다음 완료되거나 오류(`onComplete` 및 `onError` 터미널 이벤트)를 발생시킬 수 있습니다. 터미널 이벤트가 트리거되지 않으면 Flux는 무한합니다.





## 참고 자료

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Flux

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html