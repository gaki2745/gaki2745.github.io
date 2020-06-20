---
layout: post
title: RxSwift) Subject/Relay/Driver/Variable 빠른요약
subtitle: ""
categories: swift
tags: rxSwift
comments: false
---


## 3분만에 보는 빠른 요약

### Subject 

- PublishSubject
  - default(초기값)이 없다
  - 구독된 순간 새로운 이벤트 수신을 알리고 싶을 때 사용
- BehaviorSubject
  - **default(초기값)을 넣어주어야 한다.**
  - PublishSubject와 동일하나 구독하는 시점 이전의 `.next` 이벤트를 반복한다.
  - 중간에 error가 발생하면 subscribe하고 있는 모든 옵져버들한테 에러를 보낸다.
- ReplaySubject
  - default(초기값)이 없다.
  - 버퍼사이즈를 지정하여 지정한 사이즈만큼 이벤트를 쌓아뒀다가 새롭게 구독한 subscriber에게 방출한다.
- AsyncSubject
  - default(초기값)이 없다.
  - Subject가 `.complete` 되면 가장 마지막 이벤트를 방출한다.(구독의 시점은 상관 없음, **Complete** 가 되는 시점에 방출)



### Relay

- `PublishRelay` 와 `BehaviorRelay`가 있다(각각 `PublishSubject`,`BehaviorSubject` 를 래핑하고있다).
- RxSwift의 Subject의 경우 `onNexct` 로 이벤트를 처리하는데 Relay는 `accept` 로 처리한다.

- error와 complete를 내보낼 수 없다. 즉 오직 `accept` 만 내보낼 수 있다. 이는 도중에 에러가 발생하거나, 완료가 되면 무시를한다.
- 에러로 인한 종료, 완료로 인한 종료가 되지 않는다 이는 스트림이 죽지않는 다는 것을 의미하고 UI와 같은 경우 에러가 나거나 완료가 났다고 종료가 되면 안되기 때문에 **UI 전용으로 사용** 하는 것이 주 목적이다.



### Subject와 Relay의 차이점

- Subject는 `.completed`, `.error`의 이벤트가 발생하면 subscribe가 종료가됩니다.
- Relay는 `.completed`, `.error`를 발생하지 않고 Dispose되기 전까지 계속 작동하기 때문에 UI Event에서 사용하기 적절합니다.


Subject와 Relay의 자세한 정리에 관하여는 아래의 블로그 포스팅을 참고하시길 바랍니다.

[Gaki-RxSwift Subject, RxCocoa Relay](https://gaki2745.github.io/swift/2019/12/23/Swift-RxSwift-03/)


### Driver

- Observable의 에러를 무시하고 싶으면 `asObservable` 이 아닌 `asDriver` 로 바꿔야 한다.
- 즉 UI 이벤트에서 사용하기 위해 만들어진 놈이다.
- onError가 없다
- UI 전용이 듯이 Main Scheduler에서 작동한다. -> `.observeOn(MainScheduler.instance)`추가할 필요가 없다.



### Variable

> "`Variable` is planned for future deprecation. Please consider `BehaviorRelay` as a replacement"

Variable의 경우 Deprecated되었습니다. 우선 그 이유는 Variable 또한 BehaviorSubject를 래핑하고 있습니다. BehaviorRelay(get-only)와는 다르게 get,set(읽기쓰기)가 가능하고 결국 Variable에 값을 = 를 통해 대입할 수 있습니다. 이는 명령형프로그래밍 스타일이기 때문에 Reactive에서는 부적합하다고 판단해서 deprecated 되었다고 합니다. BehaviorRelay의 별칭정도로 사용된다고 합니다.





