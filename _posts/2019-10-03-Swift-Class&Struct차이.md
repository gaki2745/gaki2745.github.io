---
published: false
---



# Swift struct vs class  


Swift에서 구조체를 사용해야할지 아니면 새로운 클래스를 사용해야할지 순간 망설인 적이 몇번 있습니다. 그렇기에 두개의 차이점을 확실히 알고 일말의 망설임 없이 사용할 수 있도록 하고자 합니다!  


<br>

## 공통점  

- 값을 저장하기 위해 Property의 정의가 가능하다.  
- 기능 실행을 위해 메소드 정의가 가능하다.  
- 서브스크립트 문법을 통해 구조체 또는 클래스가 갖는 프로퍼티에 접근하도록 서브스크립트를 정의할 수 있다.
- `initializer` 정의가 가능하다.
- `extension` 정의가 가능하다.
- `protocol`을 준수 할 수 있다.  

<br>

## 차이점  

##### struct(구조체)

- 구조체는 **value type**이므로 매개변수 전달시 value copy가 일어나는 **Call By Value**이다.
- stack memory영역에 할당 한다.
	- 이는 속도가 매우 빠르다
    - scope based lifetime: 컴파일 타임에 컴파일러가 언제 메모리를 할당 해제 할지 정확히 알고 있다.
    - data locality: CPU 캐시 히트율이 높다
- 구조체는 상속이 불가능하다.
- `Cadable` 프로토콜을 이용하여 보다 편리하게 JSON과 struct를 변환 가능하다.(swift4이상)
- 항상 새로운 변수로 value copy가 일어나기 때문에 멀티스레드 환경에서 공유 변수로 인해 문제를 일으킬 확률이 적다.
- 구조체에서는 **AnyObject** 로 타입캐스팅이 불가능하고 오직 클래스에서만 가능하다.
- 구조체는 생성자를 구현하지 않을 때 기본 `initializer`를 사용할 수 있다.  

<br>

##### class(클래스)  

- 클래스는 **reference type**이므로  할당 또는 매개변수 전달시 객체를 가리키고 있는 메모리 주소 값만 복사되는 **Call By Reference**이다.
- heap memory영역에 할당 한다.
	- 이는 속도가 느리다.
    - 런타임에 직접 alloc(할당) 하며 Reference Counting을 통해 delloc이 필요하다.
    - Memory Fragmentation(메모리 단편화) 등의 Overhead가 존재한다.
- 상속이 가능하다.
- `NSData` 직렬화(Serialize)가 가능하다.
- `Codable` 프로토콜을 사용 불가능하다.
- 런타임에서 타입 캐스팅을 통해서 클래스 인스턴스에 따라 여러 동작이 가능하다(참고로 런타임에 타입 캐스팅이 이루어 지는 것은 as?,as! 를 통한 타운캐스팅이 해당된다)
- `deinitializer`가 존재한다.


























