---
layout: post
title: Swift OOP) ISP - 인터페이스 분리원칙(Interface Segregation Principle)
subtitle: ""
categories: swift
tags: oop
comments: false
---

## ISP - 인터페이스 분리원칙(Interface Segregation Principle)



ISP는 인터페이스에 관한 어떠한 분리원칙을 제시하고 있습니다. 어떻게 분리를 하며 또 분리하지 않을때는 어떤 문제가 있을지 공부해보겠습니다.

인터페이스 즉 Swift에서 protocol을 정의할 때 한 protocol에 너무 많은 메소드들을 포함하게 된다면 그것을 상속해서 구현하는 쪽(클라이언트)에서는 불필요한 메서드까지 구현해야하는 상황까지 발생한 적이 있습니다. 



> "클라이언트가 자신이 사용하지 않는 메소드에 의존하지 않아야 한다."



우선 클라이언트-서버 개념부터 알아보고 가겠습니다. 여기서 말하는 클라이언트는 **코드를 사용하는 쪽** 이 됩니다. 그리고 서버는 코드를 제공하는 쪽이 됩니다.

<img width="862" alt="image" src="https://user-images.githubusercontent.com/33486820/87947129-dba79e80-cadd-11ea-971c-38984eb21b95.png">

- 클라이언트: 객체를 사용하는 쪽
- 서버: 사용되는 쪽



ISP의 경우 인터페이스에 해당하는 경우가 두가지 존재하게 됩니다.

- 상속을 통해 사용하는 경우
- instance를  사용하는 경우



### 상속을 통해 사용

우선은 아래와 같이 클라이언트-서버를 A,B로 나누어 간단한 예제를 만들어 보았습니다.

```swift
protocol B {
    func 기능1()
    func 기능2()
    func 기능3()
}

class A: B {
    func 기능1() {
        // 기능 구현
    }
    // 사용하지 않는 메서드
    func 기능2() { }
    func 기능3() { }
}
```

B protocol 을 채택하여 구현하는 A(클라이언트) 쪽에서는 B에서 선언한 `기능1`,`기능2`,`기능3` 함수를 직접 구현하여 사용하고 있습니다.

하지만 실질적으로 사용하는 함수는 `기능1` 의 함수 뿐이며 나머지 두 함수는 결국 사용하지 않아 비워두는 수밖에 없습니다.

> 참고) Objective-C의 경우 optional로 채택한 인터페이스에 대해서 구현하지 않아도 되지만 Swift의 경우 불가능



앞에서 언급했듯이 클라이언트(A)는 사용하지 않는 메서드에 의존하지 않아야 한다고 말했습니다. 여기서 **의존이란 사용** 이란 말과 동일합니다. 그러므로 B 프로토콜을 채택하여 제공하는 메서드에 의존하게 되고 위의 예제에서 발생하는 문제는 **사용하지 않는 메소드가 존재** 한다는 것입니다.

결국 이것이 ISP를 위반한 경우에 생기는 문제점이되겠습니다. 

- 클라이언트가 사용하지 않는 메소드의 선언을 강제 -> SRP 위반
- 상속 받은 메소드를 퇴화시켜야함(LSP 위반)
- 덩치가 큰 protocol을 상속 받으면 불필요한 메소드를 선언해야할 가능성이 높음



채택을 통해 사용하지 않는 메소드는 따로 나누어 새로운 인터페이스로 분리하는 것 또한 하나의 방법이 될 수 있습니다.



> 서버는 클라이언트가 필요로 하는 최소한의 인터페이스만 제공하여 둘 간의 의존도를 낮추어야 한다.



### 인스턴스를 사용하는 경우

- 직접적으로 메소드를 사용하지 않으면 크게 영향을 주지는 않음
- 컴파일시 의존관계에 의해 불필요한 컴파일 필요
- 불필요한 모듈의 업데이트 유발



> 큰 덩어리의 인터페이스들을 구체적이고 작은 단위들로 분리시켜 클라이언트들이 꼭 필요한 메소드만 사용할 수 있게 해야 한다.



### String-Hashable의 예

swift에서 Hashable이라는 protocol이 존재합니다. String 또한 Hashable 이기에 `hashValue` 를 사용할 수 있습니다. 

```swift
func myFunction(key: String) {
    let hashValue = key.hashValue
  	...
}
```

위와 같이 String 타입으로 매개변수를 받아 `hashValue` 를 사용하는 예제가 있습니다. 하지만 String 내부의 정의를 살펴보면 Hashable을 채택하는 것을 확인할 수 있습니다. 

<img width="612" alt="image" src="https://user-images.githubusercontent.com/33486820/87949709-57efb100-cae1-11ea-81dd-bd1cbb181598.png">

그렇기 때문에 `hashValue` 만 사용한다면 좁은 범주의 Hashable로 매개변수 타입을 변경하는 것이 클라이언트와 String의 **의존관계를 끊을 수 있게 해줍니다.**

```swift
func myFunction(key: Hashable) {
    let hashValue = key.hashValue
  	...
}
```



### iOS에서의 ISP 예

iOS에서 TableView작업을 할 때 두개의 protocol을 통해 테이블뷰에 대한 작업을 하였습니다.

- `UITableViewDataSource`
- `UITableViewDelegate`

이 두개를 분리 한 것 또한 필요한 protocol만 채택하여 작업을 하기위해 분리를 해놓은 것이라 할 수 있습니다. TableView에 담길 데이터와 관련된 작업은 DataSource, TableView 의 동작에 관련된 작업을 명시해놓은 함수들이 있기 때문에 원하는 것을 채택하여 구현하면 됩니다.

Objective-C보다 Swift로 넘어오면서 많은 것들이 Hashable과 같이 분리가 되어 제공되고 있는 것을 알 수 있습니다. `Hashable`, `Equatable`, `Comparable` 필요한 메서드들을 한두개만 제공함으로써 위예제에서 봤던 큰 규모의 서버와의 의존관계를 낮추는 것이 가능합니다.









