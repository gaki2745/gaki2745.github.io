---
layout: post
current: post
navigation: True
title: Factory Pattern
date: 2020-04-12 23:30:00
tags: Swift DESIGNPATTERN
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


## Factory Pattern



객체지향 디자인에서 **느슨한 결합(Loose Coupling) ** 의 개념은 중요하고 여겨지고 있습니다. 

예를 들어 객체의 인스턴스르 만드는 작업이 항상 공개되어있으면 안되고 오히려 공개되어있을 경우 결합에 관련된 문제가 발생할 수 있다는 것 입니다.

이를 팩토리 패턴을 통해 해결할 수 있다고 합니다. 오늘은 거기에 대해서 한번 자세하게 공부해보겠습니다.



자 우리는 Swift 이전에 Java에서는 인스턴스를 생성할때 어떻게 했는지 생각해보고 가겠습니다.

```java
Person person = new GoodPerson();
```



우리는 `new` 를 사용하여 구상 클래스의 인스턴스를 생성합니다.

`Person` 이라는 인터페이스 아래 `GoodPerson` 이라는 구상 클래스가 있고 이 클래스의 인스턴스를 생성하는 코드였습니다.

하지만 구상클래스가 여러개가 존재한다면 어떻게 될까요?

```java
Person person;

if (isKind) {
		person = new GoodPerson();
} else if (isBully) {
		person = new BadPerson();
} else {
  	person = new NormalPerson();
}
```

위와 같이 구상클래스가 3개 혹은 그 이상이 존재할 수 있고 만들어지는 인스턴스의 형식은 실행시에 조건에 따라 결정됩니다.

이런코드는 무언가를 변경하거나 확장해야 할 떄 코드를 다시 확인하고 추가 또는 제거해야한다는 것을 뜻합니다. 

그렇게 되면 코드의 관리 및 갱신이 어려워지고 그에따라 오류도 자주 발생할 수 있게됩니다.

사실 `new` 자체에 문제가 있는건 아니지만 이런 **변화** 가 발생할 때 문제가 되는것입니다.



이런 변화를 감당할려면 우리는 인터페이스(프로토콜)를 사용하면 변화를 이겨낼 수 있습니다. 인터페이스는 다형성 덕분에 어떤 클래스든 특정 인터페이스만 구현하면 사용할 수 있기 때문입니다.



> "변화에 닫힌 코드를 피하기 위해 인터페이스를 사용하자"
>
> "바뀔수 있는 부분을 찾아내서 바뀌지 않는 부분하고 분리시켜야한다" 



### 팩토리 패턴의 세가지



1. 팩토리 패턴: 사용자에게 인스턴스 생성 로직을 노출하지 않은채 객체를 생성한다.
2. 팩토리 메소드 패턴: 객체 생성을 위한 인터페이스를 정의하지만 이를 구현한 객체가 어떤 클래스를 생설할지 결정한다. 구상 클래스에게 객체의 생성을 위힘한다.
3. 추상 팩토리 패턴: 연관된 혹은 의존성이 있는 객체의 그룹을 구체적인 클래스를 지정하지 않고 생성하기 위해 인터페이스를 제공한다.



### 팩토리 패턴과 팩토리 메서드 패턴



팩토리 패턴의 경우 생성로직을 숨긴다에 의미를 크게 두고 있습니다. 예를 들어 피자공장의 로직을 구현해야한다고 가정해보겠습니다.

개발자인 저는 피자만들기의 자동화를 위해 아래와 같이 코드를 구현했습니다.

```swift
func orderPizza(_ type: String) -> Pizza {
    var pizza = Pizza()
    
    switch type {
    case "cheese":
        pizza = CheesePizza()
    case "Peperoni":
        pizza = PepperoniPizza()
    case "Clam":
        pizza = ClamPizza()
    case "Veggie":
        pizza = VeggiePizza()
    }
    
    pizza.prepare()
    pizza.bake()
    pizza.cut()
    pizza.box()
    return pizza
}
```

피자의 종류를 type 매개변수로 받아와 요청에 맞는 피자를 만들어야 하기 때문에 구상클래스가 총 4개가 존재합니다.

하지만여기서 메뉴를 변경한다고 또는 추가가 된다면 `orderPizza` 메서드를 전체적으로 갈아엎어야하는 상황이 발생합니다.



여기서 우리는 피자의 구상객체 즉 상황에 맞는 객체를 생성해서 반환해주는 팩터리 메서드를 추가해보도록 합시다!

```swift
public class SimplePizzaFactory {
    public func createPizza(_ type: String) -> Pizza? {
        var pizza: Pizza?
        switch type {
        case "cheese":
            pizza = CheesePizza()
        case "Peperoni":
            pizza = PepperoniPizza()
        case "Clam":
            pizza = ClamPizza()
        case "Veggie":
            pizza = VeggiePizza()
        default:
            pizza = nil
        }
        return pizza
    }
}
```

`SimplePizzaFactory` 는 말그대로 상황에 맞는 피자를 생성해주는 팩토리 클래스라고 생각하면 됩니다.

이럴경우 어떤 이점이 있을까요? 

"상황에 맞는 피자객체를 반환한다"의 경우가 과연 피자를 주문할때만 사용될까요? 온라인 피자주문 이라던지 앱으로 피자를 찜할 수 있는 메서드 `dibsOnPizza` 에서 사용할 수도있기때문에 따로 클래스에 캡슐화 시켜 놓으면 구현을 변경해야하는 경우 여기저기에서 고칠 필요가 없다는 장점이 있습니다.



이제 피자를 생성해서 반환해줄 팩터리가 있으니 `PizzaStore` 에서 어떻게 사용하는지 보겠습니다.

```swift
protocol

public class PizzaStore {
    var factory: SimplePizzaFactory
    
    init(factory: SimplePizzaFactory) {
        self.factory = factory
    }
    
    func orderPizza(_ type: String) -> Pizza? {
        
        // 팩토리 객체에 있는 create 메서드를 사용하여 객체 생성
        guard let pizza = factory.createPizza(type) else { return nil }
        
        pizza.prepare()
        pizza.bake()
        pizza.cut()
        pizza.box()
        return pizza
    }
}
```

인스턴스 생성 자체를 팩토리 객체를 통해 완벽하게 숨긴것을 볼 수 있습니다.



위의 `if-else` / `switch` 문 과 같은 조건문을 사용해서 분기로 객체를 생성하는 것은 피할 수가 없습니다. 모델링한 객체의 수가 늘어나고 구상 클래스가 추가됨에 따라 이런 조건문은 늘어나기 마련입니다. 그리고 함수 내부에 조건문이 존재한다는 것은 하나의 함수가 여러가지 일을 할 수 있다는 것을 의미하게 됩니다. **이는 SRP(Single Responsibliity Principle)를 위반하게 됩니다.** 또는 조건 혹은 케이삭 추가될때 코드를 재차 수정해야 하기 때문에 **OCP(Open Closed Principle)의 원칙도 위반하게 됩니다.** 

그렇기 때문에 같은 조건문이 반복된다면 이는 팩토리 메소드 패턴으로 조건문을 숨기고 조건문을 한곳에만 위치시키는 것이 가능합니다.





### 추상 팩토리 메서드



다시예제로 돌아와서 위의 PizzaStore로 예를 들어보겠습니다. 

우리는 피자종류가 Newyork 과 Chicago 두가지 스타일이 더 있다고 가정하겠습니다(이것보다 더 많지만..)

피자의 종류는 다르지만 기본적으로 아래 3개의 메서드는 겹쳐진다고 할 수 있겠습니다.

- `createPizza()` : 피자를 만들고
- `orderPizza()` : 피자를 주문받고
- `deliveryPizza()` : 피자를 배달하고



피자를 만드는 방식 그리고 주문방식(?) 배달방식등 지역마다 다 다르게 존재합니다. 그럼 이같은 중복되는 기능에 대해 역시 추상화를 진행할 수 있습니다.

기본적으로 피자를 만들고 주문받고 배달하는 기능이야 동일하게 이루어 지니깐요



```swift
protocol PizzaStore {
    func createPizza() -> Pizza?
    func orderPizza()
    func deliveryPizza()
}

class NewyorkPizzaStore: PizzaStore {
    
    func createPizza(_ type: String) -> Pizza? {
        var pizza: Pizza
        switch type {
        case "cheese":
            pizza = NewyorkCheesePizza()
        case "Peperoni":
            pizza = NewyorkPepperoniPizza()
        case "Clam":
            pizza = NewyorkClamPizza()
        case "Veggie":
            pizza = NewyorkVeggiePizza()
        default:
            pizza = nil
        }
        return pizza
    }
    
    func orderPizza() { }
    func deliveryPizza() { }
}

class ChicagoPizzaStore: PizzaStore {
    
    func createPizza(_ type: String) -> Pizza? {
        var pizza: Pizza
        switch type {
        case "cheese":
            pizza = ChicagoCheesePizza()
        case "Peperoni":
            pizza = ChicagoPepperoniPizza()
        case "Clam":
            pizza = ChicagoClamPizza()
        case "Veggie":
            pizza = ChicagoVeggiePizza()
        default:
            pizza = nil
        }
        return pizza
    }
    
    func orderPizza() { }
    func deliveryPizza() { }
}

```



공통적으로 `createPizza` 를 통해 객체를 생성하는 부분은 뉴욕이나 시카고 두 구상 클래스가 수행하는 작업입니다. 하지만 두개의 클래스가 반환 즉 생성하는 객체는 다르므로 이를 공통의 프로토콜 `PizzaStore` 를 통해 `createPizza()` 를 추상화 해서 사용하는 것입니다.



어디까지나 뉴욕피자와 시카고 피자는 공통의 생산품(객체) 가 `Pizza` 임에 가능한 것입니다. 

![image](https://user-images.githubusercontent.com/33486820/79069649-d5e63400-7d0a-11ea-95ad-734445470c01.png)



더 나아가 뉴욕과 시카고 피자가게가 생성하는 제품(피자 객체)를 추상화 해서 아래와 같이 모델링을 추가할 수도 있습니다.



```swift
protocol Pizza {
    var name: String { get }
    var dough: Daugh { get }
    var sause: Sause { get }
    var veggies: [Veggies]? { get }
    var cheese: Chesse? { get }
    var pepproni: Pepperoni? { get }
    var clam: Clam? { get }
    
    func bake()
    func cut()
    func box()
}
```





### 한번더 예제를 통해 이해시켜보자



추상 팩토리 패턴에 대해서 조금 더 명확한 예제가 있어서 정리해볼려고 합니다.

두 회사 Google과 Apple이 Button과 Label을 만들어서 제공합니다.



기본적으로 아래와 같이 프로토콜을 통해 두가지를 정의할 수 있습니다.

Button과 Label에 각각 제목을 지정할 수 있고 보여주는 기능까지 있다고 가정하겠습니다.

```swift
protocol Button {
    func setTitle(_ title: String) -> Void
    func show() -> Void
}

protocol Label {
    func setTitle(_ title: String) -> Void
    func show() -> Void
}
```



자 이제 Google와 Apple이 각각 Button과 Label 생성할 수 있도록 프로토콜을 상속하여 구현하게 됩니다.

```swift
class AppleButton: Button {
    var title: String?
    
    func setTitle(_ title: String) -> Void {
        self.title = title
    }
    
    func show() -> Void {
        print("Apple 스타일 버튼 보여주기: [Title: \(self.title!)]")
    }
}

class AppleLabel : Label {
    var title: String?
    
    func setTitle(_ title: String) -> Void {
        self.title = title
    }
    
    func show() -> Void {
        print("Apple 스타일 레이블 보여주기: [Title: \(self.title!)]")
    }
}

class GoogleButton : Button {
    var title: String?
    
    func setTitle(_ title: String) -> Void {
        self.title = title
    }
    
    func show() -> Void {
        print("Google 스타일 버튼 보여주기: [Title: \(self.title!)]")
    }
}

class GoogleLabel : Label {
    var title: String?
    
    func setTitle(_ title: String) -> Void {
        self.title = title
    }

    func show() -> Void {
        print("Google 스타일 레이블 보여주기: [Title: \(self.title!)]")
    }
}
```

그렇게 되면 아래와 같이 Button 과 Label에 대한 클래스를 각각 만들어 주어야하고 추후에 버튼, Label 뿐만 아니라 더 추가가 되게 된다면 클래스가 더 늘어 난다고 볼 수 있습니다.



이것을 이제 추상 팩토리 패턴으로 변경해보겠습니다.



#### 추상팩토리 프로토콜

```swift
protocol AbstractGUIFactory {
    func createButton() -> Button
    func createLabel() -> Label
}

class AppleFactory : AbstractGUIFactory {
    func createButton() -> Button {
        return AppleButton()
    }
    
    func createLabel() -> Label {
        return AppleLabel()
    }
}

class GoogleFactory : AbstractGUIFactory {
    func createButton() -> Button {
        return GoogleButton()
    }
    
    func createLabel() -> Label {
        return GoogleLabel()
    }
}
```



 Apple과 Google의 공통 프로토콜인 `AbstractGUIFactory` 를 선언해서 겹치는 메서드를 정의했고 각 구상클래스가 이를 채택하여 구체화 하였습니다.

차이가 보이시나요? 위의 코드에서는 각 클래스를 Button,Label마다 하나씩 만들어 주었는데 보다 명확하게 클래스의 수가 줄었습니다. 가령 `View` 객체를 생성해야하는 경우가 생길경우 아래와 같이 추상 팩토리 프로토콜에 클래스만 추가 해주면 됩니다.

```swift
protocol AbstractGUIFactory {
    func createButton() -> Button
    func createLabel() -> Label
		func createView() -> View
}
```





#### Client

 자 이제 Client에서 이를 활용해야겠죠?

`GUIBuilder` 라는 클래스를 통해 우리가 원하는 두가지 플랫폼 Apple 또는 Google 에 해당하는 객체를 생성하고 또 내부 함수까지 사용할 수 있게 하도록 작성해보겠습니다.

```swift
class GUIBuilder {
    private var platform: String
    private var guiFactory: AbstractGUIFactory?
    
    init(platform: String) {
        self.platform = platform
    }
    
    func initGuiFactory() -> Void {
        if guiFactory != nil { return }
        switch platform {
        case "apple":
            guiFactory = AppleFactory()
        case "google":
            guiFactory = GoogleFactory()
        default:
            guiFactory = nil
        }
    }
    
    func buildButton() -> Button {
        initGuiFactory()
        if let button = guiFactory.createButton() {
            return button
        } else {
            return Button()
        }
    }
    
    func buildLabel() -> Label {
        initGuiFactory()
        if let label = guiFactory.createLabel() {
            return Label
        } else {
            return Label()
        }
    }
}
```



#### GUIBuilder를 사용해보자

```swift
let guiBuilder: GUIBuilder = GUIBuilder(platform: "Apple")

let Label: Label = guiBuilder.buildLabel()
label.setTitle("Gaki")
label.show()

let button: Button = guiBuilder.buildButton()
button.setTitle("Connect")
button.show()
```



마지막으로 정리를 한번 하고 끝내도록 하겠습니다.

- 추상팩토리 패턴 - 서로 연관된, 또는 의존적인 객체들로 이루어진 제품군을 생성하기 위한 인터페이스(프로토콜)을 제공. 구상 클래스는 서브클래스에 의해 만들어진다
- 팩토리 메소드 패턴 - 객체를 생성하기 위한 인터페이스를 만든다. 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정하도록한다. 팩토리 메소드를 이용하면 인스턴스를 ㅁ나드는 일을 서브클래스로 미룰 수 있다.



이 처럼 객체생성을 캡슐화 해주고 구상 클래스에 대한 의존성을 줄여 줌으로서 궁극적으로 추구하고 하는 느슨한 결합을 도와주도록 하게됩니다.



## Reference

- [Abstract Factory Pattern in Swift 4](https://medium.com/@iamcrypticcoder/abstract-factory-pattern-in-swift-4-83abb273c977)
- [Swift World: Design Patterns — Abstract Factory](https://medium.com/swiftworld/swift-world-design-patterns-abstract-factory-86f39dc7b63f)
- [[Design Pattern] 추상 팩토리 패턴이란](https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html)
- [JAVA 객체지향 디자인 패턴, 한빛미디어](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788968480911&orderClick=JAj)
- [Head First Design Pattern](https://m.search.naver.com/search.naver?sm=mtp_hty.top&where=m&query=헤드+퍼스트+디자인+패턴#api=%3F_lp_type%3Dcm%26col_prs%3Dcsa%26format%3Dtext%26nqx_theme%3D%257B%2B%2522theme%2522%253A%257B%2522main%2522%253A%257B%2522name%2522%253A%2522book_info%2522%252C%2522os%2522%253A1882446%252C%2522pkid%2522%253A20000%257D%257D%2B%257D%26query%3D%2525ED%252597%2525A4%2525EB%252593%25259C%252B%2525ED%25258D%2525BC%2525EC%25258A%2525A4%2525ED%25258A%2525B8%252B%2525EB%252594%252594%2525EC%25259E%252590%2525EC%25259D%2525B8%252B%2525ED%25258C%2525A8%2525ED%252584%2525B4%26sm%3Digr_brg%26tab%3Dinfo%26tab_prs%3Dcsa%26where%3Dbridge&_lp_type=cm)



