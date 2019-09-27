# flatMap? comapctMap? 변경사항

Swift 4.1 에서 부터 이름이 변경 되었다. 하지만 `flatMap`의 사용이 중단 된 것은 아니다.

<br>

## flatMap -> compactMap 변경  

<br?

Swift의 array와 같은 sequence에 flatMap을 사용하면 nil이 매핑 되는 것을 필터링하는 작업을 해주느 메서드이다.

```swift
func flatMap<U>(_ transform: (Warapped) throws -> U?) rethrows -> U?
```

- nil 필터링

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



