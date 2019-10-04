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

<br>

## 값 타입과 참조 타입  

구조체의 경우 `call by value`가 일어나 value copy가 발생하고 , 클래스는 `call by reference`가 일어나 실제로 그 객체의 주소가 전달 되어 접근이 가능합니다. 일단 값 타입과 참조 타입의 가장 큰 차이점은 **무엇이 전달 되느냐**입니다. 참조라는 개념은 C언어 기반의 C++,Objective-C등의 언어에서 사용되는 Pointer(포인터)의 개념과 유사한 개념입니다. 우선 이해를 돕기 위해 C언어에서 `call by value` 와 `call by reference`의 차이점을 간략하게 설명 드리겠습니다.  

<br>

##### call by value vs call by reference  



```c
// 편의를 위해 call by value : cbv, call by reference : cbr 이라고 하겠습니다.

// call by value 
int cbvSum(int a, int b) {
  
  int sum = a + b;
  
  return sum;
}

int main() {
  int a = 10;
  int b = 20;
  int sum;
  
  // 1. call by value로 단순하게 a,b의 값만 복사하여 cbvSum 함수에 매개변수러 전달 한다.
  // 값에 대한 복사만 일어났기 때문에 a, b에는 아무런 영향이 없다 
  sum = cbvSum(a, b);
  // 종료 시점에도 a = 10, b = 20 유지
   
}
```

<br>

##### call by reference  


```c
// call by reference
void cbrDoubleSum(int*pA, int*pB, int*pSum) {
  // 3. 함수 내부에선 포인터로 a, b의 값을 접근하여 a,b의 값을 직접 두배로 올리고
  *pA *= 2;
  *pB *= 2;
  
  // 매개변수로 받은 pSum( main에서는 sum) 에 집접 값을 넣고 함수를 종료한다.
  *pSum = *pA + *pB;
}

int main()
{
  int a = 10;
  int b = 20;
  int sum;
  
  // 1. call by reference로 a,b의 주소값(&)을 함수 매개변수로 넘겨주고 실제 그 주소값의 변수에 접근하기 위해 
  // 2. cbrDoubleSum함수에서는 매개변수를 대응하기위해 포인터(*)를 통해 실제 값에 접근한다.
  cbrDoubleSum(&a, &b, &sum);
  // 함수가 종료되면 a = 20, b = 40, sum = 60으로 실제 값에 변화가 발생한 것을 알 수 있다.
 
}

```  

위에서 두개의 개념을 간략하게나마 소개를 드렸습니다. 하지만 Swift에서는 참조라는 것을 표현해주기 위해 애스터리스크(`*`) 를 사용하지는 않습니다.

<br>

## Swift의 값 타입과 참조타입  

<br>

##### struct: 값 타입 

```swift
struct BasicInformation {
    let name: String
    var age: Int
}

var gakiInfo: BasicInformation = BasicInformation(name: "Gaki", age: 25)
gakiInfo.age = 100

var gakiClone: BasicInformation = gakiInfo
gakiClone.age = 25

print(gakiInfo.age)     // 100
print(gakiClone.age)    // 25
```

gakiClone의 age 프로퍼티를 변경 시켜도 gaki의 age에는 아무런 영향이 없는 것을 확인 할 수 있습니다. 이는 값만 복사가 되었기 때문에 전혀 서로에게 형향을 주지 않는 다는 것을 알수 있습니다.  

<br>

##### class: 참조 타입  

```swift
class Developer {
    // var로 프로퍼티를 선언하고 초기화를 하지 않으면 initialize를 해야한다고 경고가 뜬다.
    var stack: String = "iOS Developer"
    var career: Int = 1
}

var gaki: Developer = Developer()
var clone: Developer = gaki

print("gaki는 \(gaki.stack) 입니다.")
print("clone은 \(clone.stack) 입니다.")

// gaki는 iOS Developer 입니다.
// clone은 iOS Developer 입니다.

clone.stack = "Android Developer"

print("gaki는 \(gaki.stack) 입니다.")
// gaki는 Android Developer 입니다.
```
