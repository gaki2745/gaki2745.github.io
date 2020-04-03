---
layout: post
current: post
navigation: True
title: Swift, RxSwift Result Type 사용법
date: 2020-04-03 11:50:00
tags: Swift RxSwift
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


## Swift5 Result Type, RxSwift에서의 Result Type 



오늘은 Swift5에 새롭게 추가된 Result Type에 대해서 정리해볼까 합니다.

비동기 API 코드를 작성하여 네트워킹을 할 경우 Result Type을 많이 쓰는데 기존과 어떤 차이점이 있고 어떤 이점이 있는지 공부해보도록 하겠습니다.



우선은 네이버 검색 API를 요총하는 코드로 예제를 작성해보겠습니다.



### 기존의 방식



1. 옵셔널로 제어한 completion

```swift
func searchPlaces<T: Decodable>(_ input: Input, completion: @escaping (T?) -> Void) {
    // URLComponent 생성
    guard let url = createComponent(input).url else {
        return completion(nil)
    }
    // URLSession을 통한 데이터 요청
    URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            return completion(nil)
        }
        // JSON Decoding
        guard let places = try? JSONDecoder().decode(T.self, fron: data) else {
            return completion(nil)
        }
        completion(places)
    }.resume()
}

```

위의 코드를 보면 `searchPlaces` 라는 메서드는 input을 넘겨받고 urlComponent 생성 -> URLSession요청 -> Decoding 순으로 진행하고 있습니다.

그 순서 속에 guard let 을 통한 바인딩으로 옵셔널을 체크하고 있습니다. 그리고 결국 성공을 하면 decoding한 객체를 반환 `completion(places)` 를 통해 성공한 객체를 escaping 하는 방식입니다.



그 후 결과를 처리하는 부분에서 아래와 같이 요청할 수 있습니다.

```swift
searchPlaces { (place: Place?) in
    if let plcae = model {
        // API요청 성공
    }
}

```

최종적으로 요청한 Place  JSON 객체가 디코딩이 잘되어 nil 이 아닌지 체크를 하면 끝입니다.



이 방법도 잘되었다고는 할 수 없는 방법입니다. 우리는 **urlComponent를 생성하는데 실패했는지? 아니면 URLSession 요청에서? Decoding실패?** 즉, 어디서 에러가 나서 nil이 반환 되었는지 모르기 때문에 error를 명시해주는게 좀더 보편적인 방법입니다.



2. 에러를 명확히 구분해서 정의

```swift
enum NetworkError: Error {
    case urlError             // URL생성에러
    case sessionError         // session에러
    case decodingError        // jsonDecoding에러
}

func searchPlaces<T: Decodable>(_ input: Input, completion: @escaping (T?, NetworkError) -> Void) {
    // URLComponent 생성
    guard let url = createComponent(input).url else {
        return completion(nil, .urlError)
    }
    // URLSession을 통한 데이터 요청
    URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            return completion(nil, .sessionError)
        }
        // JSON Decoding
        guard let places = try? JSONDecoder().decode(T.self, fron: data) else {
            return completion(nil, .decodingError)
        }
        completion(places, nil)
    }.resume()
}

```



차이를 아시겠나요? 에러를 조금더 명확하게 해줌으로써 어디서 에러가 어디서 발생 되었는지 확인 할 수 있습니다.

```swift
searchPlaces { (place: Place?, error) in
    if let error = error {
        print(error)
    }
    if let plcae = model {
        // API요청 성공
    }
}

```



이 처럼 에러를 미리 구분을 하여 특정에러의 위치를 확인 할 수도 있었습니다.

사실 Swift5의 Result Type이 나오기 이전에도  API요청의 경우**실패 or 성공** 이 명확하기 때문에 

**Success, Failiure** 의 구조를 enum 형태로 만들어서 사용하기도 했습니다.



3. enum 타입 Success, Failure

```swift
enum APIResult<T> {
    case success(T)
    case failure(NetworkError)
}

enum NetworkError: Error {
    case urlError             // URL생성에러
    case sessionError         // session에러
    case decodingError        // jsonDecoding에러
}

func searchPlaces<T: Decodable>(_ input: Input, completion: @escaping (APIResult<T>) -> Void) {
    // URLComponent 생성
    guard let url = createComponent(input).url else {
        return completion(.failure(.urlError))
    }
    // URLSession을 통한 데이터 요청
    URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            return completion(.failure(.sessionError))
        }
        // JSON Decoding
        guard let places = try? JSONDecoder().decode(T.self, fron: data) else {
            return completion(.failure(.decodingError))
        }
        completion(.success(places))
    }.resume()
}

```



```swift
searchPlaces { (result: APIResult<Place>) in
    switch result {
    case .success(let place):
    // 성공
    case .failure(let error):
    // 실패
    }
}

```



위 같은 경우 지금의 Result Type의 형태와 매우 유사합니다.

이런 enum타입의 APIResult 따로 만들지 않아도 사용할 수 있게 애플은 Swift5부터 stdlib에 Result Type을 추가하여 제공하고 있습니다

<img width="588" alt="image" src="https://user-images.githubusercontent.com/33486820/78318045-cb0b0100-759e-11ea-945e-65836df11ed6.png">





### Result Type 사용



```swift
enum NetworkError: Error {
    case urlError             // URL생성에러
    case sessionError         // session에러
    case decodingError        // jsonDecoding에러
}

func searchPlaces<T: Decodable>(_ input: Input, completion: @escaping (Result<T, NetworkError>) -> Void) {
    // URLComponent 생성
    guard let url = createComponent(input).url else {
        return completion(.failure(.urlError))
    }
    // URLSession을 통한 데이터 요청
    URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            return completion(.failure(.sessionError))
        }
        // JSON Decoding
        do {
            let places = try? JSONDecoder().decode(T.self, from: data)
            completion(.success(places))
        } catch {
            completion(.failure(.decodingError))
        }
    }.resume()
}

```



```swift
searchPlaces { (result: Result<Place, NetworkError>) in
    switch result {
    case .success(let place):
    // 성공
    case .failure(let error):
    // 실패
    }
}

```



크게 달라진건 없다고 생각합니다 그냥 "더이상 ResultType enum은 안만들어도 된다?" 정도라고 생각하시면 됩니다. 이제 우리는 Error만 명시해줘서 에러 핸들링을 명확하게 구분하고 또 실패 성공의 데이터흐름을 조금 더 명확하게 할 수 있게 되었습니다.





### RxSwift의 API요청 Result Type



추가적으로 RxSwift에서는 어떻게 Result Type을 사용할까요? 크게 달라지는건 없습니다. 

반환 타입이 escaping을 통한 객체탈출이 아니라 Observable이고 Observable에 맞게 completion이 `just` 으로 바뀌었다! 정도로만 이해하시면 됩니다.



```swift
func searchPlaces<T: Decodable>(input: PlaceInput) -> Observable<Result<[T], NetworkError>> {
    guard let url = createSearchComponents(input: input).url else {
        return .just(.failure(urlError))
    }
    var request = URLRequest(url: url)
    return session.rx.data(request: request)
        .map { data in
            do {
                let response = try JSONDecoder().decode(T.self, from: data)
                return .success(response.places ?? [])
            } catch {
                return .failure(.decodingError)
            }
    }
}

```



```swift
let initValue = initialResult
    .map { result -> [Place] in
        guard case .success(let value) = result else { return [] }
        return value
}
.asDriver(onErrorJustReturn: [])

let initError = initialResult
    .map { result -> String? in
        guard case .failure(let error) = result else { return nil }
        return error.message
}
    .filterNil()

```



## Reference

- https://swift.org/blog/swift-5-released/
- https://developer.apple.com/documentation/xcode_release_notes/xcode_10_2_release_notes/swift_5_release_notes_for_xcode_10_2
- https://github.com/fimuxd/RxSwift
