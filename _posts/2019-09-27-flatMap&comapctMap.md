---
layout: post
current: post
navigation: True
title: Swift flatMap과 compactMap 사용법 
date: 2019-09-27 05:18:00
tags: Swift 
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


<br>

# flatMap? comapctMap? 변경사항

Swift 4.1 에서 부터 이름이 변경 되었다. 하지만 `flatMap`의 사용이 중단 된 것은 아니다.

<br>

## flatMap -> compactMap 변경  

<br>

Swift의 array와 같은 sequence에 flatMap을 사용하면 nil이 매핑 되는 것을 필터링하는 작업을 해주느 메서드이다.


### compactMap으로 변경 : nil 필터링

```swift
let company: [String?] = ["NAVER", nil, "Samsung", "Kakao", nil, nil, "google"]

let valid = company.flatMap{ $0 }

print(valid) // ["NAVER", "Samsung", "Kakao", "google"]
```

<br>

위의 flatMap은 이제 compactMap으로 변경이 되어 아래와 같이 적용이 된다.  

```swift
let company: [String?] = ["NAVER", nil, "Samsung", "Kakao", nil, nil, "google"]

let valid = company.compactMap{ $0 }

print(valid) // ["NAVER", "Samsung", "Kakao", "google"]
```

하지만 여전히 flat을 사용해야하는 부분이 존재한다.

## flat을 여전히 사용하는 부분

<br>

### 1. Sequence의 컨텐츠를 일렬로 flat하게 정렬할때 사용 
 
```swift
let scores = [[3,6,7], [5,8], [9,1,13]] 

let allScores = scores.flatMap { $0 } 
//[3, 6, 7, 5, 8, 9, 1, 13] 

let passMarks = scores.flatMap { $0.filter { $0 > 7} } 
// [8, 9, 13]
```

### 2. Optional에서 flatMap사용  


```swift
func flatMap<U>(_ transform: (Warapped) throws -> U?) rethrows -> U?
```

위 처럼 input 값으로 Optional 값을 받게 되면 Optional에서의 flatMap을 호출하기 때문에 Optional Case를 다룰때 flatMap을 사용한다.


```swift
let input: Int? = Int("8")
let numbers: Int? = input.flatMap { $0 > 5 ? $0 : nil }

print(numbers) // Optional(8)
```


<br>

<hr>

## Reference  

- https://zeddios.tistory.com/448

