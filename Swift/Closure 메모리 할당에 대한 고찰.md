# 클로저의 Capture, Capturelist에 대한 고찰
## 목차
1. [공부하면서 든 생각](#공부하면서-든-생각)
2. [클로저란?](#클로저란)
3. [클로저가 캡처를 하는 이유가 뭘까?](#클로저가-캡처를-하는-이유가-뭘까)
4. [클로저의 캡처 리스트란?](#클로저의-캡처-리스트란)


## 공부하면서 든 생각
- 아래 내용을 공부 한 후, `escaping`키워드도 공부를 해보았고 case마다 메모리에 할당되는 방식이 다르다는 것을 알게되었다. 이러한 case들을 외우지 않고 개념적으로 파악하기 위해 고민을 해보았다.

- 일단 함수가 언제 Heap에 저장이 되나/ 언제는 Stack에서 실행이 되나에 대해서는 case별로 생각하는게 아니라
중첩함수던, 그냥 함수던 그 `함수 자체를 오랫동안 보존을 할거냐 안 할거냐`를 보는게 맞는 것 같다.

ex → 중첩함수의 리턴값을 변수에, 함수를 변수에, 

- 함수나 클로저를 오랫동안 유지를 시킬거면 외부 변수에 할당을 해야할 것이고, 외부변수에 할당을 한다면 Heap에 존재하게 되므로 오래 유지가 가능해진다.

(클로저는 기본적으로 힙에 저장되기는 한다.)

- 또한, 함수가 종료되면 내부의 클로저도 힙에서 사라지기 때문에 오랫동안 그 클로저를 유지하기 위해 `escaping`시키는 것이라고 생각이 든다.


## 클로저란?

- 일단 Closure란 내부 함수와 내부 함수에 영향을 미치는 주변 환경을 모두 포함한 객체이다.
- 클로저는 객체안에 속해있어도 별도로 힙에 분리되어 할당된다.

- 함수를 변수에 담거나, 중첩함수의 리턴값을 다른 변수에 담는등 함수를 오래 사용하려고 변수에 담게 되면 힙에 저장이되게된다.

## 클로저의 캡처 현상이란??

- 클로저가 외부의 변수를 사용할때, 저장할 필요가 있는 값의 1)값을 저장하거나 2)참조(주소)를 저장한다.
    - Closure은 외부의 값이 Value & Reference Type이던 모두 참조 캡처(주소값을 저장)를 한다.
    
     → 지속적으로 참조할수 있도록 저장한다. 
    

## 클로저가 캡처를 하는 이유가 뭘까?

- 클로저가 캡처를 함으로서 주변에 정의된 상수나 변수가 더 이상 메모리에 존재하지 않더라도 해당 상수나 변수의 값을 자신 내부에서 참조하거나 수정할 수 있다.

예시

- 비동기 콜백을 작성하는 경우 현재 상태를 미리 획득해두지 않으면, 실제로 클로저의 기능을 실행하는 순간에는 주변의 상수나 변수가 이미 메모리에 존재하지 않는 경우가 발생한다.

### 값 타입을 캡처할 경우 → 참조 Capture을 해서 참조를 한다.

- 값타입을 캡처 할 경우, 지속적으로 참조가 되기 때문에 값이 계속 변경 될 수 있다.
```swift
// 값 타입 Capture 예시
func doSomething() {
    var num = 10
    
    let closure = {
        print(num)    // 위의 num이 Reference캡처된다.
    }
    
    num = 20         // 지속적으로 캡처된다.
    closure()
}

doSomething()       // 20이 출력
```


### 참조 타입을 캡처할 경우 → 강한 참조 사이클 발생

- 참조타입을 캡처 할 경우, 참조하고있는 객체의 RC를 올리기 때문에 강한 참조 사이클이 발생할 수 있다.
- 때문에 166번 Line처럼 nil을 할당해도 내부적으로 RC를 높혔기때문에 할당해제가 안된다.
```swift
// 참조 타입 Capture 예시
class Person {
    var name: String = "Kyo"
    
    lazy var closure: () -> String = {
        return self.name     // Person RC + 1
    }
    
    deinit {
        print("\(self.name) is deinit!!")
    }
}

var kyo: Person? = Person()  // Person RC + 1
kyo?.name = "Kyo PAK"
print(kyo!.closure())   // "Kyo PAK" 출력

kyo = nil   // deinit 호출이 안된다.
```
![스크린샷 2022-10-11 오후 11 40 14](https://user-images.githubusercontent.com/59204352/195549733-8182917b-f7a7-415a-aab9-9b6232d9d6cf.png)


## 클로저의 캡처 리스트란??

- 캡처리스트를 사용함으로써,
- 값타입 캡처시 일어나는 지속적인 참조문제를 해결 가능.
- 참조 타입 캡처시 일어날 수 있는 RC증가 문제를 해결가능 (`weak`, `unowned` 필요)

### 값 타입을 캡처리스트화 하면? → 값을 Const Value Type으로 캡처한다.

- 참조하지 않고 클로저가 선언 되었을 때의 그 값을 딱 상수로 복사해온다. → 지속적인 변경이 안이루어진다.
```swift
// 값타입 캡처에 캡처리스트를 사용한다면?
func doSomethingCaptureList() {
    var num = 10
    
    let closure = { [num] in
        //num = 2     // constant라서 변경 불가
        print(num)    // 135번 line num이 상수로 복사된다.
    }
    
    num = 20        // 값이 바뀌어도 캡처되지 않는다.
    closure()
}

doSomethingCaptureList()    // 10이 출력
```
### 참조 타입을 캡처리스트화 하면?

- 여전히 참조 타입이기 때문에 강한 참조가 발생할 수 있지만, `weak`, `unowned` 키워드를 사용할 수 있게되고 사용함으로서 약한참조를 할 수 있다.

- 아래와 같이 캡처리스트만 추가한다고 해서 무조건 rc가 안 올라가지 않는다. 여전히 할당해제가 안된다

```swift
class Person {
    var name: String = "Kyo"
    
    lazy var closure: () -> String = { [self] in
        return self.name     // Person RC + 1
    }
    
    deinit {
        print("\(self.name) is deinit!!")
    }
}
```

- 그렇다면 만약 아래와 같이 weak, unowned키워드를 사용한다면? RC안올라가서 할당해제 된다.
```swift
// 참조 타입 CaptureList 추가 예시
class Person {
    var name: String = "Kyo"
    
    lazy var closure: () -> String? = { [weak self] in
        return self?.name     // Person RC 그대로
    }
    
    deinit {
        print("\(self.name) is deinit!!")
    }
}

var kyo: Person? = Person()  // Person RC + 1
kyo?.name = "Kyo PAK"
print(kyo!.closure())   // "Kyo PAK" 출력

kyo = nil   // deinit 호출이 안된다.
```

<img width="659" alt="스크린샷 2022-10-13 오후 5 54 44" src="https://user-images.githubusercontent.com/59204352/195550925-6bf2d397-9f3c-4d96-9f31-eb29c1b2923a.png">

- 아래와 같이 `unowned` 키워드를 사용할 수 도 있다.
```swift
class Person {
    var name: String = "Kyo"
    
    lazy var closure: () -> String? = { [unowned self] in
        return self.name     // Person RC 그대로
    }
    
    deinit {
        print("\(self.name) is deinit!!")
    }
}
```


