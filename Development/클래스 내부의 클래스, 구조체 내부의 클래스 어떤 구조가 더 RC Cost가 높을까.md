## 클래스 내부의 클래스 vs 구조체 내부의 클래스

- WWDC에서 구조체가 클래스보다 RC Cost가 적다고 했다. 당연.
- 근데 구조체 내부에 클래스가 많으면 RC Cost가 클래스 내부에 클래스가 있는 것보다 높다고 했다. 왜?

### 클래스 내부에 클래스(Reference type)가 있는 구조
```swift
class Label {
    var text: String
    var font: UIFont
    func draw() { ... }
}

let label1 = Label(text: "Kyo", font: font)
let label2 = label1
```
![스크린샷 2022-11-05 오전 1 02 06](https://user-images.githubusercontent.com/59204352/200021680-28219451-aa22-45d7-bc31-cf652eac68f1.png)



### 구조체 내부에 클래스(Reference type)가 있는 구조
```swift
struct Label {
    var text: String
    var font: UIFont
    func draw() { ... }
}

let label1 = Label(text: "Kyo", font: font)
let label2 = label1
```
![스크린샷 2022-11-05 오전 1 04 29](https://user-images.githubusercontent.com/59204352/200022241-2fb1bd5d-5134-4ff3-9aea-280b77592b1e.png)


### 결론

- 구조체 내부에 클래스가 있는 구조에서는 RC가 2인 객체가 2개가 생성되지만
- 클래스 내부에 클래스가 있는 구조에서는 RC가 2인 객체가 1개만 생성이 된다.
- 따라서 메모리 할당해제를 위한 Cost가 구조체 내부에 클래스가 있는 구조가 더 높다고 말한 것 같다.
