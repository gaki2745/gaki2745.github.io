---
layout: post
current: post
navigation: True
title: Swift 고차함수 Map, Filter, Reduce
date: 2019-09-27 10:18:00
tags: Swift
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


<br>

# 고차함수(Higer-order function)란?

<br>


Swift의 경우 함수를 일급 객체로 취급한다. 그렇기 때문에 객체의 특성을 가지고 함수를 다른 함수의 전달인자로 사용할 수 있다. 

> 매개변수로 함수를 갖는 함수를 고차함수라고 한다.  

Swift의 대표적인 고차함수로 **Map, Filter, Reduce**가 있다.   

<br>


## Map  

<br>

map은 자신을 호출할때 **매개변수로 전달된 함수를 실행**하여 그 결과를 다시 반환해주는 함수다. 배열, 딕셔너리, 세트, 옵셔널 등에서 사용할 수 있다. 즉 Swift의 **Sequence, Collection** 프로토콜을 따르는 타입과 오셔널은 모두 map을 사용할 수 있다.  

<img width="844" alt="image" src="https://user-images.githubusercontent.com/33486820/65736156-1d152a80-e115-11e9-9ed2-ffd028f75e80.png">  


map을 사용 할 경우 기존의 대상이 되는 컨테이너 값은 변경 되지 않고 매개변수로 전달된 함수를 실행하여 값이 변경이된 **새로운 컨테이너**가 생성되어 반환된다.

<br>

사실 map의 사용방법은 기존의 `for-in`구문과 차이가 없다.  

- for-in구문을 사용한 새로운 배열 반환  

```swift  
let numbers: [Int] = [0, 1, 2, 3, 4]

var doubledNumber: [Int] = [Int]()
var stringNumber: [String] = [String]()

for number in numbers {
    doubledNumber.append(number * 2)
    stringNumber.append(String(number))
}

//[0, 2, 4, 6, 8]
//["1", "2", "3", "4"]
```  

- map을 사용  

```
doubledNumber = numbers.map({ (number: Int) -> Int in
    return number * 2
})

stringNumber = numbers.map({ (number: Int) -> String in
    return String(number)
})


// 클로저를 이용하여 코드를 간결화
doubledNumber = numbers.map{ $0 * 2 }

stringNumber = numbers.map{ return String($0)}

```  

<br>

## Filter  

<br>

filter는 말 그대로 컨테이너 내부의 값을 걸러서 추출하는 역할을 하는 고차함수다. 맵과 동작원리는 동일하며 새로운 컨테이너를 반환한다. 하지만 기존 컨텐츠를 변형하는 것이 아니라 **특정 조건**에 맞게 걸러내는 역할을 할 수 있다.   
<br>
filter 함수의 매개변수로 전달되는 함수의 반환 타입은 Bool 이다. 해당 컨텐츠의 값을 갖고 새로운 컨테이너에 포함될 항목이라고 판단 하면 true, 그렇지 않으면 false를 반환한다.  

- for-in 구문을 사용  

```swift

let numbers: [Int] = [0, 1, 2, 3, 4, 5]
var evenNumbers: [Int] = [Int]()

// for 구문 사용
for number in numbers {
  if number % 2 != 0 { continue }
  evenNumbers.append(number)
}

print(evenNumbers) // [0, 2, 4]

```  

- filter 사용  

```swift
let mappedNumbers: [Int] = numbers.map{ $0 + 3 }

let evenNumbers: [Int] = mappedNumbers.filter { (number: Int) -> Bool in
    return number % 2 == 0
}

print(evenNumbers) // [4, 6, 8, 10]
```  

- 클로저를 사용  

```swift
let evenNumbers = mappedNumbers.filter{ $0 % 2 == 0 }
let oddNumbers = mappedNumbers.filter{ $0 % 2 != 0 }
```  

- map과 filter를 동시에 사용할 수 있다.

```swift
let evenNumer = numbers.map{ $0 + 3 }.filter{ $0 % 2 == 0 }
```  

<br>

## Reduce  

reduce의 기능은 combination(결합)의 기능이다. 컨테이너 내부의 컨텐츠를 하나로 합하는 기능을 실행하는 고차함수이다. 그 대상이 배열이면 모든 값을 전달인자로 전달받은 클로저의 연산 결과로 합해준다.  


<br>

reduce의 경우 두가지 형태로 구현이 되어있다. 첫 번째 reduce는 클로저가 각 요소를 전달 받아 연산한 후 값을 다음 클로저 실행을 위해 반환하며 컨테이너를 순환하는 형태이다.  

```swift
public static reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) rethrows -> Result) throws -> Result
```  

`initialResult`라는 이름의 매개변수로 전달되는 값을 통해 초깃값을 지정해 줄수 있고 `nextPartialResult`라는 이름의 매개변수로 클로저를 전달받는다.  

<br>

두번째 reduce 메서드는 컨테이너를 순환하며 클로저가 실행되지만 클러조가 따로 결괏값을 반환하지 않는 형태이다. `inout` 키워드를 통해 초깃값에 직접 연산을 실행하게 된다.  

```swift
public func reduce<Result>(into initialResult: Reslt, _ updateAccumulatingResult: (inout Result, Element) throws -> ()) rethrows -> Result
```

`updateAccumualtingResult` 매개변수로 전달받은 클로저의 매개변수 중 첫 번째 매개변수를 inout 매개변수로 사용한다. `updateAccumualtingResult` 클로저의 첫 번째 매개변수는 reduce 메서드의 `initialResult` 매개변수를 이용해 전달받은 초깃값 또는 이전에 실행된 클로저 때문에 변경 되어있는 결괏값이다. 

```swift

let numbers: [Int] = [1, 2, 3]


var sum: Int = numbers.reduce(0, { (result: Int , element: Int) -> Int in
    print("\(result) + \(element)")
    return result + element
})
//    0 + 1
//    1 + 2
//    3 + 3

print(sum) // 6
```

위의 클로저에서 초깃값은 0으로 설정했고 그 옆에 클로저를 통해 result는 현재 계산이 된 값, element는 대상이 되는 컨텐츠가 되어 아래와 같이 모든 배열의 합이 결과로 출력이된다.

<br>

- 클로저를 활용해 조금더 간략하게 표현  

```swift 
var sum: Int = numbers.reduce(0) {
    print("\($0) + \($1)")
    return $0 + $1
}
// 위의 클로저의 결과값 매개변수가 첫번째 즉 $0 이다.
```  
<br>



- 문자열 배열을 메서드를 사용하여 연결  

```swift
let company: [String] = ["NAVER", "Samsung" ,"Kakao", "LINE"]

let redueceCompany: String = company.reduce("Korea Company: ") {
    return $0 + " " + $1
}

print(redueceCompany) // "Korea Company:  NAVER Samsung Kakao LINE"
```
<br>

### `reduce(into:)` 사용법  

<br>

첫번째 reduce 메서드와 다르게 `reduce(into:)`의 경우는 **다른 컨테이너에 값을 변경하여 넣는 것이 가능**하다.



```swift
var doubledNubmers: [Int] = numbers.reduce(into: [1, 2]) { (result: inout [Int], element: Int) in
    print("result : \(result), element: \(element)")
    
    guard element % 2 == 0 else {
        return
    }
    
    print("\(result) append \(element)")
    
    result.append(element * 2)
}
//result : [1, 2], element: 1
//result : [1, 2], element: 2
//[1, 2] append 2
//result : [1, 2, 4], element: 3

print(doubledNubmers) //[1, 2, 4]
```












