---
layout: post
title: Swift의 Queue & Stack 구현
subtitle: ""
categories: algorithm
tags: swiftAlgorithm
comments: false
---



Swift의 경우 C 처럼 STL을 제공하 않기 때문에 직접 자료구조를 직접 작성해주어야 한다. 기본적으로 Queue와 Stack은 배열 기반으로 해당 기능을 구혀 해주는데 Swift에서 기본적으로 제공하는 array 메서드를 사용하여 기능을 구현하면된다. 자료구조형의 데이터 타입은 제네릭을 사용하야 다양한 기본타입, 객체 타입에 대처 할 수 있게 한다.     

<br>

```swift
public struct Queue<T> {
    fileprivate var array = [T]() //제네릭 타입의 배열 생성
    
    public var front: T? { return array.first } // Queue의 맨 앞 부분은 first , 반환 타입은 Queue의 element기 때문에 T
    public var count: Int { return array.count }
    public var isEmpty: Bool { return array.isEmpty }
    // 실제 구조체의 프로퍼티에 접근해 값을 바꾸기 떄문에 mutating 키워드를 써주어야 한다.
    public mutating func enqueue(_ element: T) { array.append(element) }
    public mutating func dequeue() -> T? { return isEmpty ? nil : array.removeFirst() }
    public mutating func clear() { array.removeAll() }
    
}


public struct Stack<T> {
    fileprivate var array = [T]()
    
    public var peek: T? { return array.last }
    public var count: Int { return array.count }
    public var isEmpty: Bool { return array.isEmpty }
    
    public mutating func push(_ element: T) { array.append(element) }
    public mutating func pop() -> T? { return array.popLast() }
    public mutating func clear() { array.removeAll() }
}
```
