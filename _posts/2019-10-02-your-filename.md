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







