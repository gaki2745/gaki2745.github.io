---
published: false
---
# Swift Type Casting

다른 객체지향 언어에서 제공하는 Type Casting이 Swift에서는 어떻게 다루어지는지 확실하게 정리하고 넘어가겠습니다. `is, as, as?, as!` 키워드에 관해서 정의와 차이점등을 중점적으로 정리하도록 하겠습니다.

<br>

## Type Casting  

TypeCasting의 정의는 **인스턴스의 타입을 확인하거나, 인스턴스의 타입을 Super Class또는 Sub Calss 타입처럼 다루기 위해 사용된다.  

> Swfit에서 타입 캐스팅은 `is` 와 `as` 연산자로 구현할 수 있고, 이 두 연산자의 값의 타입을 확인하거나, 값을 다른 타입으로 변환하는 간단하고 Swift만의 표현적인 방법을 제공한다.  

<br>

## 인스턴스 타입 Checking : `is`  

`is` 키워드를 사용하면 인스턴스의 Sub Class 인지 아닌지 판단 할 수 있게 제공한다. 만약 해당 Sub Class의 인스턴스면 `true`를 반환하고 아니면 `false`를 반환한다. 

<br>

##### SuperClass : **Person**

```swift
class Person {

	var name: String
   	
    init(name: String) {
    	self.name = name
    }
}    	
```  

- `is`를 사용하여 타입 체크  

```swift
var gaki = Person(name: "Gaki")

if gaki is Person {
    print(true)		// true
}    
```  

위와 같이 `is`의 경우 `gaki`라는 인스턴스가 `Person`의 인스턴스인가를 판단할 때 쓸 수 있다. 마찬가지로 Sub Class의 인스턴스에도 적용이 되는지 확인해보겠습니다.  

<br>

##### SuperClass : **MediaItem**  

```swift
class MediaItem {

    var name: String
    
    init(name: String) {
        self.name = name
    }
}
```  

##### SubClass : **Movie**, **Song** 

```swift
class Movie: MediaItem {

    var director: String
    
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {

    var artist: String
    
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}
```

<img width="742" alt="image" src="https://user-images.githubusercontent.com/33486820/66009839-3e9c5a80-e4f7-11e9-98d3-eb0c25aeaf20.png">

위와 같이 구조로 이루어진 상태에서 `is`를 사용하여 하위 클래스에 대한 동작이 어떻게 이루어지는 확인해보겠습니다.

<br>
##### SuperClass Type Array: **library**  

```swift
let library = [

    Movie(name: "어벤져스: 엔드게임", director: "안소니 루소"),
    
    Song(name: "Viva La Vida", artist: "콜드플레이"),
    
    Movie(name: "스파이더맨: 파프롬홈", director: "존 왓츠"),
    
    Movie(name: "기생충", director: "봉준호"),
    
]
var movieCount = 0

var songCount = 0

for item in library {
    if item is Movie {
        movieCount += 1

    } else if item is Song {
        songCount += 1
    }
}
print("Media library: \(movieCount)개의 영화와  \(songCount)개의 노래가 존재")

```  

library배열에서 `is`를 통해 `Movie`와 `Song` 타입의 인스턴스르 발견할 시 갯수를 늘려보는 예제이다. 여기서 궁금한 점은 **직접 item을 Movie나 Song 타입의 인스턴스로 접근 할 수는 없는가**이다. 여기서 Movie 클래스의 `director`를 접근해보면 아래와 같은 에러가 발생한다.  


<img width="824" alt="image" src="https://user-images.githubusercontent.com/33486820/66010356-9340d500-e4f9-11e9-857c-699c9983f83f.png">  

위와 같이 직접 SubClass의 인스턴스를 참조하여 프로퍼티나 접근하고 싶을때는 `as`를 통한 **Downcasting**이 필요하다.  

<br>

## DownCasting  
<br>

특정 클래스 타입의 상수나 변수는 하위 클래스의 인스턴스를 참조할 수 있다. 위와 같이 library 배열에서 sub class의`Movie`나 `Song` 클래스 인스턴스에 직접 접근을 할 수 있게 해준다. 이 경우 타입 캐스트 연산자 **as? 또는 as!** 를 사용하여 다운캐스팅을 진행할 수 있다.  

<br>

## DownCasting 연산자의 두가지 형태  

다운캐스팅은 실패할 가능성이 있기때문에, 타입 캐스트 연산자는 아래와 같이 두가지 형태로 제공한다.

- `as?` : 조건부 형식으로 다운캐스팅 하려는 타입의 Optional 값을 반환한다.
- `as!` : 강제 형식으로 강제 언래핑을 통해 값을 반환한다.  
<br>

> 다운캐스팅이 정말 명확할때(ex tableView나 collectionView의 cell에 대하여 dequeuereusablecell을 사용하여 Cell값을 가져올 때 보통 강제 언래핑을 통하여 가져온다. 하지만 이 방법도 코드가 길어지고 많은 view나 controller가 존재하게되면 좋지 않게 되므로 가급적이면 as?를 통해 처리해주는 것을 지향해야한다.  

##### DownCasting을 통해 SubClass 인스턴스 접근  

```swift
for item in library {

    if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
   }
}

/*
Movie: 어벤져스: 엔드게임, dir. 안소니 루소
Song: Viva La Vida, by 콜드플레이
Movie: 스파이더맨: 파프롬홈, dir. 존 왓츠
Movie: 기생충, dir. 봉준호
 */
```  

> 주의: as? 의 경우 Optional값의 인스턴스를 반환하기 때문에 반드시 guard-let 이나 if-let 구문을 사용하여 옵셔널 바인딩을 통해 값을 추출 해야한다.  

위와 같이 `if-let`구문에서 `as?`를 통한 조건부 형식으로 다운 캐스팅을 통해 Movie 타입으로 캐스팅이 되면 Optional 값으로 `movie`상수는 Movie 클래스의 인스턴스로써 사용할 수 있다. `Song`도 마찬가지다.  


## Type Casting for Any and AnyObject  

`Any`와 `AnyObect` 타입에서는 타입캐스팅이 어떻게 이루어지는지 다루어 보겠습니다. 우선 Any와 AnyObject에 관해서 간략하게 설명을 하겠습니다.  

Swift에서는 **특정하지 않은 타입**에 대해 동작하도록 타입을 두가지 제공합니다.


<img width="940" alt="image" src="https://user-images.githubusercontent.com/33486820/66011693-17e22200-e4ff-11e9-997e-207d0e6eb31e.png">  

위의 다이어그램에서 확인할 수 있듯이 Any가 조금 더 큰 범주에 속하고 그 속에 AnyObject가 있는 것을 확인 할 수 있습니다.  

<br>

##### Any: 함수타입을 포함하여 모든 타입의 인스턴스를 나타낸다.

- String과 Int 형이 공존하는 배열을 Any로 선언

```swift  
var anyArr: [Any] = [1, 2, "3", "4"]
// or
var anyArr : Array<Any> = [1, 2, "3", "4"]
```  

위와 같이 Any 타입으로 선언한 배열은 String 도 들어올 수 있고 Int형도 들어 올 수 있습니다. 뿐만 아니라 아래와 같이 Bool값, Double 형 도 들어 올 수 있습니다.  

```swift
var anyArr : [Any] = [1, "two", true, 4.0] 
```  

> 가능한 이유?: Any의 경우 모든 타입의 인스턴스를 포괄할 수 있습니다. 정확하게는 **구조체**로 구현된 값 타입은 모두 들어 올 수있고 String, Int, Double, Float, Bool 등 모두 구조체로 구현이 되어있기 때문에 가능합니다.  

- 구조체로 선언 되어있는 String, Int(그 외 다른 기본자료형도 마찬가지) 

<img width="409" alt="image" src="https://user-images.githubusercontent.com/33486820/66012054-7f4ca180-e500-11e9-962e-a90b82a838c6.png">

<img width="491" alt="image" src="https://user-images.githubusercontent.com/33486820/66012070-91c6db00-e500-11e9-8d16-f0aded123349.png">  

<br>


##### AnyObject: 모든 클래스 타입의 인스턴스를 나타낸다.

![image](https://user-images.githubusercontent.com/33486820/66012161-07cb4200-e501-11e9-8ad0-2adb4508ac68.png)

공식문서에 언급 되어있는데 AnyObject는 **프로토콜** 입니다. 이 AnyObject 프로토콜은 **모든 클래스가 암시적으로 conform(준수) 하는 프로토콜이다**라고 정의가 되어있습니다.  

> 결론 : AnyObject는 클래스, 클래스 타입 또는 클래스 전용 프로토콜(protocol : class) 의 인스턴스에 대한 구체적인 타입으로 사용될 수 있습니다. 즉 클래스 타입이 아니면 사용할 수 없습니다.  

```swift
class Aclass{ }

class Bclass{ }

var anyObjectArr : [AnyObject] = [Aclass(), Bclass()]
```

위의 예제 처럼 `Aclass` 와 `Bclass` 타입을 갖는 `anyObjectArr`가 생성이 됩니다. 아래와 같이 AnyObject형의 배열에는 구조체형의 Int, Double, String.. 등의 기본 자료구조형을 넣을 수 가 없는 이유가 바로 이와같이 클래스 타입만을 가지기 때문입니다.  

```swift
var anyObjectArr : [AnyObject] = [1, "two", true, 4.0]

// error 발생
```

> error: value of type 'Int' does not conform to expected element type 'AnyObject'

위와 같이 AnyOject에 준수하지 않는다고 에러가 발생하고 아래와 같이 수정을 해주어야 비로소 에러가 사라지게 됩니다.


```swift
var anyObjectArr : [AnyObject] = [1 as AnyObject, "two" as AnyObject, true as AnyObject, 4.0 as AnyObject]
```  
<br>

다시 본론으로 돌아와서 Any타입이 경우에는 함수(클로저 포함)뿐만 아니라 모든 타입의 인스턴스를 넣을 수 있기 때문에 아래와 같이 기본 자료형, Movie라는 클래스타입, 클로저도 함수이기 때문에 함수도 넣을 수 있습니다.  


```swift
var groups = [Any]()
groups.append(1.0)
groups.append(1)
groups.append("string")
groups.append((3.0, 5.0))
groups.append(Movie(name: "조커", director: "토드 필립스"))
groups.append({ (name: String) -> String in "hello my name is \(name)"})

for item in groups {
    switch item {
    case let anInt as Int:
        print("\(anInt) is an int")
    case let aDouble as Double:
        print("\(aDouble) is a double")
    case let aString as String:
        print("\(aString) is a string")
    case let (x, y) as (Double, Double) :
        print("x좌표 : \(x), y좌표: \(y)")
    case let movie as Movie:
        print("영화정보 입력> 제목 : \(movie.name), 감독 : \(movie.director)")
    case let introduceClosure as (String) -> String :
        print(introduceClosure("Gaki"))
    default:
        print("so on")
    }
}

/* 출력
1.0 is a double

1 is an int

string is a string

x좌표 : 3.0, y좌표: 5.0

영화정보 입력> 제목 : 조커, 감독 : 토드 필립스

hello my name is Gaki
*/
```

위에서 as를 패턴매칭(switch)에서만 사용한 것을 볼 수 있습니다. 궁금증이 드는데 위에서 다운캐스팅 때에는`as?`와 `as!`를 사용했지만 switch 구문에서나 Upcasting 에서는 as 를 사용한 것을 볼 수 있습니다. 이는 `as`와 `as?`, `as!`가 실행 시간에서 차이가 있습니다.  

<br>

## as와 as! as? 의 차이점  

우선 본론부터 말씀드리면 **as는 컴파일 시간**에 실행이 되고 **as?, as!는 런타임 시간**에 실행이 됩니다.   

- 컴파일 타임 : 소스코드(swift파일)을 작성하고 compile이라는 과정을 통해 기계어코드로 변환 되어 **실행 가능한 프로그램이 되게 만드는 편집 과정**

- 런타임 : 컴파일 과정을 마친 프로그램이 사용자에 의해 실행되어 지고 응용프로그램이 동작될때의 시간

실행 순서로 판단하면 컴파일 다음 런타임이 진행 된다는 것을 확인 할 수 있습니다.
앞서 말씀드렸듯이 `as`의 경우 컴파일 시간에 실행이 되어 캐스팅이 되는지 안되는지 판단을 우선 합니다. 아래와 같이 컴파일러가 항상 성공할 것이라는것을 알게되면 다운캐스팅에서도 as를 사용할 수 있습니다.

- 다운캐스팅에서 as 사용  

```swift
let favoritMovie: Movie = marvelMovie as Movie
```

컴파일러는 항상 성공한다는 것을 알게 되고 즉 캐스팅 하려는 타입이 같은 타입이거나 부모 클래스 타입이라는 것을 알때 위와 같이 다운 캐스팅에서도 as 를 사용하여도 문제가 되지 않습니다.  

> 참고 : 타입캐스팅의 의미  
	캐스팅은 실제로 인스턴스를 수정하거나 값을 변경하는 작업이 아니어서 그 인스턴스는 메모리에 똑같이 남아있다. 다만 컴퓨터에게 "내가 이 인스턴스를 사용할 때는 앞으로 이런 타입으로 접근할꺼야"라고 힌트를 주는 것 뿐이다.     - 야곰 Swift프로그래밍 - 
    
    
<br>

<hr>

## Reference  

- [Zedd iOS - TypeCasting](https://zeddios.tistory.com/265)
- [Type casting in swift : difference between is, as, as?, as!](https://medium.com/@abhimuralidharan/typecastinginswift-1bafacd39c99)
- [야곰의 Swift Programming 개정판](http://www.hanbit.co.kr/store/books/look.php?p_code=B2206901403)
- [Apple Developer](https://developer.apple.com/documentation)












