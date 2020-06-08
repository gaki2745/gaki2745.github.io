---
layout: post
title: Observer Pattern
subtitle: ""
categories: ios
tags: designPattern
comments: false
---



## Observer Pattern(옵저버 패턴)



이번시간에는 Swift Design Pattern중에 많이 사용하고 있는 Observer Pattern에 대해 정리해보도록 하겠습니다.

<img width="472" alt="image" src="https://user-images.githubusercontent.com/33486820/78560928-d20d6a00-7851-11ea-8da4-ffcb184e18df.png">

뜻에서 짐작 할 수 있듯이 "관측하는 역할"이 존재한다는 것을 알 수 있습니다.

자 그럼 관측자의 개념이 왜 나오는 것일 까요?



#### 출판사 + 구독자 = 옵저버 패턴

신문을 출판하는 출판사를 생각해봅시다. 신문을 찍어내는것은 출판사의 역할이고 이를 구독할지 말지를 결정하는 것은 구독자입니다. 이를 옵저버 패턴에 그대로 가져와서 생각을 해보면 아래와 같습니다.

> "신문을 찍어내는 출판사를 주제(subject), 구독자를 옵저버(observer) "



<img width="831" alt="image" src="https://user-images.githubusercontent.com/33486820/78562499-6c6ead00-7854-11ea-969d-f87cdcdfacef.png">

- Subject 객체는 데이터를 관리하고 있습니다
- Subject 객체의 데이터가 변경이 되면 Observer한테 그 소식이 전달 됩니다.
- Observer 객체들은 Subject 객체를 구독(Subscribe)하고있으며 Subject 객체의 데이터가 바뀌면 갱신 내용을 전달 받습니다.



크게 두가지의 역할이 존재합니다. 전반적인 데이터를 관리하는 Subject 객체와 또 공통 관심사를 가지고 있는 Observer객체들이 존재합니다.



> 옵저버 패턴에서는 한 객체의 상태가 바뀌면 그 갹체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many)의존성을 정의합니다.



즉 Observer는 Subject에 의존하게 됩니다. 대부분의 경우 Subject와 Observer를 프로토콜로 선언을 하게 됩니다.

이는 나중에 의존성 이슈에관하여 조금더 다루도록 하겠습니다.



자 그럼 코드로 직접 구현하면서 어떻게 동작하는지 조금더 명확하게 알아보겠습니다.



### 구현

기상관측에 관한 상황을 가정해보겠습니다. 실시간으로 기상데이터를 관측하는 Satellite(인공위성) 이 있다고 하겠습니다. 이 인공위성은 기상을 관측하는 즉시 데이터가 들어오는 이를 전국에 있는 WeatherStation(기상관측소)가 항시 인공위성의 데이터를 구독하고 있다고 생각해보겠습니다. 

즉 Observer = WeatherStation(기상관측소), Subject = Satellite(인공위성) 이런 구조라고 생각하고 실습을 해보겠습니다.



- Observer 프로토콜 선언

```swift
protocol ObserverProtocol {
    var id : Int { get set }
    func update(_ value: Any?)
}
```

옵저버의 identity를 위해 Id 프로퍼티를 추가했습니다. 추후에 탐색과 삭제시 필요한 프로퍼티입니다. 추후 WeatherStation의 id로 사용하기 위함입니다.



- Subject 프로토콜 선언

```swift
protocol Subject {
    var observers : [Observer] { get set }
    func addObserver(_ observer: Observer)
    func removeObserver(_ observer: Observer)
    func notifyObservers(_ observers: [Observer])
}
```

Subject의 경우 observers의 Observer 타입의 시퀀스를 가지고 있습니다. 이는 하나의 Subject에 여러개의 Observer객체가 구독할 수 있기 때문입니다. 그리고 Observer들에게 값이 변경되면 **Notify** 를 할 수 있고  Observer를 추가및 삭제 할 수있게 프로토콜을 선언했습니다.



- Subect의 데이터 모델링

```swift
// 날씨 모델링
struct Weather {
    var temperatrue: Double             // 온도
    var humidity: Double                // 습도
    var pressure: Double                // 기압
    
    init(temp: Double, humidity: Double, pressure: Double) {
        self.temperatrue = temp
        self.humidity = humidity
        self.pressure = pressure
    }
}
// 옵저버들이 구독할 인공위성
class Satellite: Subject {
    
    internal var observers: [Observer] = [Observer]()
  	// 초기 날씨 데이터 0.0으로 임시 초기화
    private var value: Weather = Weather(temp: 0.0, humidity: 0.0, pressure: 0.0)
    var weather : Weather {
        set {
            value = newValue
            notifyObservers(self.observers)
        }
        get {
            return value
        }
    }
    
    func addObserver(_ observer: Observer) {
        observers.append(observer)
    }
    
    func removeObserver(_ observer: Observer) {
        guard let index = self.observers.firstIndex(where: { $0.id == observer.id }) else { return }
        self.observers.remove(at: index)
    }
    // 모든 Observer에게 알려줌과 동시에 값 업데이트
    func notifyObservers(_ observers: [Observer]) {
        observers.forEach({ $0.update(weather)})
    }
    
    deinit {
        observers.removeAll()
    }
}

```

`WeatherData` 클래스는 Subject 프로토콜을 채택하여 구현하고 있습니다. 쉽게 말해서 옵저버들이 관심있어할 Subject의 Data라고 생각하시면됩니다. Subject의 목적은 데이터를 제공하고 Observer의 관리, 그리고 즉각적으로 데이터가 변경 되면 Observer들에게 알리는 것입니다. 그리고 Observer의 목적은 Subject의 데이터 즉 `WeatherData` 의 실질적인 데이터 값 `Weather` 의 값이 변경되는 것을 관측하고 있다가 값이 변경되면 변경된 사항을 전달 받는 형태입니다.



- Satellite(인공위성) 과 WeahterStation(기상관측소) 생성

```swift
class WeatherStation: Observer {
    private var satellite: Satellite = Satellite()
    var id: Int
    
    init(_ id: Int, _ satellite: Satellite) {
        self.id = id
        self.satellite = satellite
        self.satellite.addObserver(self)
    }
    
    func update(_ value: Any?) {
        guard let weather = value as? Weather else { return }
        print("\(id)번 관측소","새로운 날씨 관측","온도:\(weather.temperatrue)", "습도:\(weather.humidity)", "기압:\(weather.pressure)")
    }
}
```



자 이제 구현한 것들을 사용해서 상황을 만들어 테스트 해보겠습니다.

```swift
// 날씨 데이터
let firstWeather = Weather(temp: 10, humidity: 20, pressure: 30)
let secondWeather = Weather(temp: 40, humidity: 50, pressure: 60)

// 인공위성 생성
let satellite = Satellite()
// 관측소 생성과 동시에 인공위성 관측시작!
let firstStation = WeatherStation(1, satellite)
let secondStation = WeatherStation(2, satellite)
// 인공위성이 첫번째 날씨데이터를 받아 업데이트 함
satellite.weather = firstWeather
/*
 출력
 1번 관측소 새로운 날씨 관측 온도:10.0 습도:20.0 기압:30.0
 2번 관측소 새로운 날씨 관측 온도:10.0 습도:20.0 기압:30.0
*/
// 인공위성이 두번째 날씨데이터를 받아 업데이트 함
satellite.weather = secondWeather
/*
 출력
 1번 관측소 새로운 날씨 관측 온도:40.0 습도:50.0 기압:60.0
 2번 관측소 새로운 날씨 관측 온도:40.0 습도:50.0 기압:60.0
*/

```

1. 인공위성 생성
2. 관측소 1, 2 생성과 동시에 인공위성 Subject 객체 구독
3. 인공위성이 날씨데이터를 업데이트 할 때마다 WeatherStation의 update 함수가 호출 되는 것을 볼 수 있다



코드에서 중요한 부분을 한번 더 짚어 보겠습니다.

```swift
class Satellite: Subject {
  ...
		var weather : Weather {
        set {
            value = newValue
            notifyObservers(self.observers)
        }
        get {
            return value
        }
    }
  ...
    // 모든 Observer에게 알려줌과 동시에 값 업데이트
    func notifyObservers(_ observers: [Observer]) {
        observers.forEach({ $0.update(weather)})
    }
}
```

인공위성의 weather가 값이 변경(Set)이 될경우 이를 모든 구독자에게 **notify** 하는 것입니다.

그 후 모든 Observer들의 update 함수를 호출 하게 됩니다.



### 마치며



지금까지 옵저버 패턴을 예제를 통해 어떻게 동작하는지 알아봤습니다. 사실 방법은 다양하고 스타일도 저마다 달라서 정답이라고 하는 것은 없습니다. 하지만 명심해야할 것은

**Subject는 Observer에 대해서 알 필요가 없다 단지 프로토콜을 채택해서 구현한다는 것 뿐** 이라고 생각합니다. 위의 프로토콜을 정의해서 Subject를 구현하고 마찬가지로 Observer도 구현하면 서로 아는 것은 단순하게 프로토콜에 명시된 동작뿐이게 됩니다. 세세한 내용은 알 지 못한다는 것입니다! 이를 느슨한 결합이라고 하고 이는 다양한 이점이 있다고합니다



> 느슨한 결합(Loose Coupiling)이란 ? 두 객체가 상호작용을 하긴 하지만 서로에 대해 서로 잘 모른다는 것을 의미한다.
>
> 이로 인해 변경사항이 있거나 추가할 사항이 있으면 유연하게 대처를 가능하게 하고 또한 옵저버 패턴에 있어서 Subject와 Observer에 대해 재사용성을 높인다. 그리고 서로 변경사항에 대해 영향을 미치지 않는다.





## Reference

- [Head First Design Patterns](https://book.naver.com/bookdb/book_detail.nhn?bid=1882446)
- [Design pattern in Swift - Observers](https://benoitpasquier.com/observer-design-pattern-swift/)

