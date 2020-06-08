---
layout: post
title: Combine(2) - Subject
subtitle: ""
categories: swiftUI
tags: combine
comments: false
---

## Combine Subject


지난 시간에 이어 이번에는 Subject에 대해서 공부해볼까 합니다.



### Subject란?

우선 Subject는 Publisher의 일종입니다. 공식문서에서 정의 된걸 보겠습니다.

![image](https://user-images.githubusercontent.com/33486820/80270925-3146f780-86f7-11ea-88bb-f5fef2c7de0f.png)

왜 Publisher의 일종이라 했는지 감이 잡히시나요? Subject의 경우 Publisher 프로토콜을 채택하고 있습니다. 특별한 점은 비동기 이벤트 처리를 하은 파이프라인 외부에서도 파이프라인 안으로 데이터를 보낼 수 있습니다. 

조금 더 자세히 살펴 보겠습니다. Combine에는 미리 만들어진 두가지 빌트인 타입의 Subject가 존재합니다.



- CurrentValueSubject
- PassthroughSubject



### CurrentValueSubject

우선 CurrentValueSubject부터 살펴보겠습니다.

>  single value를 래핑하고 값이 변경될 때마다 새로운 element를 publishing 하는 subject

PassThroughSubject와 달리 CurrentValueSubject는 **가장 최근에 publish 된  element의 버퍼를 유지** 하게 됩니다.

밑의 예제에서도 볼 수 있듯이 CurrentValueSubject에서 send를 호출하게 되면, 현재의 값도 업데이트 되므로 , 값을 직접 업데이트 하는 것과 같다고 할 수 있습니다. 상태값을 가지는 Subject이기 때문에 주로 UI 상태값에 따라 데이터를 발행할때 사용하기 유용합니다.

- 예제

```swift
let currentValueSubject = CurrentValueSubject<String, Never>("A")
let subscriber = currentValueSubject.sink(receiveValue: {
    print($0)
})

currentValueSubject.value = "B"
currentValueSubject.send("C")

/*
A
B
C
*/

let currentStatus = CurrentValueSubject<Bool, Error>(true)

currentStatus.sink(receiveCompletion: { completion in
    switch completion {
    case .failure(let error):
        print("error 발생:",error.localizedDescription)
    case .finished:
        print("데이터 publishing 종료")
    }
}, receiveValue: { value in
    print("발행된 데이터:",value)
})

currentStatus.value = false
currentStatus.send(true)

/*
발행된 데이터: true
발행된 데이터: false
발행된 데이터: true
*/

```

초기값을 줘야한다는 점에서 Rx의 BehaviorSubject와 유사한것을 알 수 있습니다. 초기값을 넣지 않게 되면 오류가 발생하므로 반드시 설정을 해주어야 합니다.



### PassthroughSubject

> downstream subscribers 에게 element를 braodcasts하는 subject

CurrentValueSubject와 달리, PassthroughSubject에는 가장 최근에 publish된 element의 초기값 또는 버퍼가 없습니다. PassthroughSubject 는 subscribers가 없거나 현재 demand가 0 이면 value를 삭제(drop) 하게 됩니다.

즉 상태값을 가지지 않는 Subject라고 할 수 있습니다. Subject의 특성상 외부에서 안으로 데이터를 주입시킬 수 있기 때문에 기존 Combine 프로토콜이 아닌 콜백 클로저로 구현되어있더라도 쉽게 리팩터링이 가능합니다.

- 예제

```swift
let passthroughSubject = PassthroughSubject<String, Error>()

passthroughSubject.sink(receiveCompletion: { completion in
    switch completion {
    case .failure(let error):
        print("error 발생:",error.localizedDescription)
    case .finished:
        print("데이터 publishing 종료")
    }
}, receiveValue: { value in
    print("발행된 데이터:",value)
})


passthroughSubject.send("Gaki")
passthroughSubject.send("Hello")
// 데이터의 발행을 종료
passthroughSubject.send(completion: .finished)
// 이미 종료(구독이)가 되었기 때문에 호출이 안됨
passthroughSubject.send("Say again!")

/*
발행된 데이터: Gaki
발행된 데이터: Hello
데이터 publishing 종료
*/
```

CurrentValueSubject와 다르게 초기값도 필요하지않습니다. 



그리고 두 Subject의 공통점을 하나 발견할 수 있습니다. 바로 `send(_:)` 라는 메소드를 통해 값을 Stream에 직접 **주입** 할 수 있는 Publisher입니다. send메소드는 3개지의 종류가있습니다.

<img width="457" alt="image" src="https://user-images.githubusercontent.com/33486820/80279074-252e5a80-8736-11ea-9f54-eb510fadb1f6.png">

위의 예제에서도 `send(input:)` 뿐만 아니라  `send(completion:)` 메서드를 사용해서 직접 subscriber에게 completion signal을 보내기도 했습니다.

```swift
passthroughSubject.send("Hello")
// 직접 subscriber에게 completion signal을 보내 발행 종료
passthroughSubject.send(completion: .finished)
// 이미 종료(구독이)가 되었기 때문에 호출이 안됨
passthroughSubject.send("Say again!")
```

그리고 마지막 `send(subscription:)` 메서드의 경우 매개변수로 `Subscription` 타입을 넘겨 받고 있습니다.

Publisher를 배울때 Publisher의 establish demand(발행 수요) 를 설정할 수 있다고 했습니다. 그렇기 때문에 이 메서드를 통해 업스트림의 발행 수요를 설정할 수 있습니다.



오늘은 Subject에 대해서 공부를 해보았습니다. Combine 시리즈를 모두 정리 했을때는 최종적으로 하고자 하는 네트워킹 코드를 어떻게 연결하는지 그리고 UI와는 어떻게 연동을 하는지에 대해서 최종적으로 목표로 두고 계속해서 정리를 하겠습니다!



### Reference

- [ZeddiOS-Combine](https://zeddios.tistory.com/965?category=842493)
- [해리의유목코딩-Combine입문하기2](https://medium.com/harrythegreat/swift-combine-입문가이드2-publisher-subscribe-operator-723ed5d17e70)
- [SwiftUI + Combine = ❤️ | Peter Friese](https://peterfriese.dev/swift-combine-love/)

