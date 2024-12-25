---
title: "iOS 스터디 week4. 구조체의 확장과 생성자 관련 세부사항"
layout: single
Typora-root-url: ../
categories: iOS
tag: [TIL, extenstion]
use_math: true
---

## 구조체의 확장과 생성자 관련 세부사항
확장에서도 생성자를 구현할 수 있지만, 모든 생성자를 구현할 수는 없다.

잘생각해보자<br/>
class에서 편의생성자는 delegate across로 같은 스코프? 계층에 있는 생성자를 호출
호출된 지정생성자는 메모리를 찍어내는 역할을 함.
- 확장에서도, 말 의미 그대로 지정생성자를 추가하기보다는, 편의 생성자만 생성가능 (지정생성자 추가 불가/소멸자 추가 불가)
- 다만 클래스가 아닌 경우에, 본체에 지정생성자를 호출하는 방법만 가능(convinence는 아니지만, 비슷한 방식)  
- `[예외]` 값타입(구조체)의 경우 저장속성에 기본값/생성자 정의안한 경우, 생성자 구현 가능

<br/>

### 클래스에서의 확장예시
```swift
// UIColor 기본 생성자
var color = UIColor(red: 0.3, green: 0.5, blue: 0.4, alpha: 1)
    


extension UIColor {      // 익스텐션 + 편의생성자 조합
    
    convenience init(color: CGFloat) {   // Float   / Double
        self.init(red: color/255, green: color/255, blue: color/255, alpha: 1)
    }

}


// 아주 쉽게 객체 생성하는 방법을 제공 가능해짐

UIColor(color: 1)
//UIColor(red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat)

```

### 구조체에서의 확장 예시

```swift
//일단 확장 개념 뺴고!!

//클래스였다면
class Point {
    var x = 0.0, y = 0.0

    convenience init(){
        self.init(x:0.0, y:0.0)
    }

    init(x: Double, y:Double){
        self.x = x
        self.y = y
    }
}

//구조체라서 클래스랑 비슷한 방식이지만 convenience 키워드 생략 가능하다.
struct Point {
    var x = 0.0, y = 0.0
    
    init(){
        self.init(x:0.0, y:0.0)
    }

    init(x: Double, y:Double){
        self.x = x
        self.y = y
    }
}
```
구조체는 (원래) 편의 생성자가 존재하지 않고, 상속과 관련이 없기 때문에 지정생성자의 형태로도 자유롭게 생성자 구현 가능

<br/>

```swift
struct Point {
    var x = 0.0, y = 0.0
}

extension Point{
    init(){
        self.init(x:0.0, y:0.0)
    }
}
```
`에러가 나는 이유?` -> 본체에 멤버와이즈 이니셜라이저랑 기본생성자가 자동 구현이 됨. 따라서 확장에서 같은 기본 생성자를 한번 더 구현하게 되면 에러가 난다!

<br/>

```swift
struct Point {
    var x = 0.0, y = 0.0
}

extension Point{
    init(num:Double){
        self.init(x:num, y:num) // 본체의 생성자 호출 부분→
    }
}
```
이런식으로 다른 생성자를 호출하면 에러가 나지 않음. → 약간 편의 생성자 느낌이긴 하다.

<br/>

### 구조체에서 직접 생성자 구현
직접 생성자 구현하면, 기본 생성자 init() → 멤버와이즈 생성자 제공 안되는 것이 원칙
```swift
struct Point {
    var x = 0.0, y = 0.0

    init(x: Double, y:Double){
        self.x = x
        self.y = y
    }
}

extension Point{
    init(){
        init(x: 0.0, y:0.0)
    }

}
```

### 예외! 기본값 + 생성자 정의 안한 경우
 모든 저장속성에 기본값제공 + 본체에 직접 생성자를 구현하지 않았다면, 멤버와이즈 생성자 자동생성 + 확장에서 생성자 구현해도 괜찮음
 ```swift
 struct Point {
    var x = 0.0, y = 0.0
}

extension Point{
    init(num:Double){
        self.x = num
        self.y = num
    }
}
 
 ```

 ## Swift 공식문서의 예시

 ```swift
 struct Point {
    var x = 0.0, y = 0.0
    
    //init(x: Double, y: Double)
    //init()
}


struct Size {
    var width = 0.0, height = 0.0
}


// Rect구조체
struct Rect {     //기본 생성자 / 멤버와이즈 생성자가 자동 제공 중
    var origin = Point()
    var size = Size()
}



let defaultRect = Rect()    // 기본생성자

Rect(origin: Point(x: Double, y: Double), size: Size(width: Double, height: Double))

let memberwiseRect = Rect(origin: Point(x: 2.0, y: 2.0),
                          size: Size(width: 5.0, height: 5.0))    
// 멤버와이즈 생성자



extension Rect {
    // 센터값으로 Rect 생성하는 생성자 만들기 
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        
        // (1) 본체의 멤버와이즈 생성자 호출 방식으로 구현 가능
        self.init(origin: Point(x: originX, y: originY), size: size)
        
        // (2) 예외적인 경우, 직접 값을 설정하는 방식으로도 구현 가능
        self.origin = Point(x: originX, y: originY)
        self.size = size
    }
}


// 새로 추가한 생성자로 인스턴스 생성해보기

let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
                      size: Size(width: 3.0, height: 3.0))

 
 ```
 예외적인 경우 → 저장속성에 기본값 + 본체에 생성자 구현 안한 경우 여전히 기본생성자/멤버와이즈 생성자 제공

 ## 생성자 회고
 ```swift
 class Dog{
    var name: String?
 }

 Dog()      //가능 -> Nil
 ```
 사실 모든 저장속성에 기본값이 있다면 init을 자동적으로 swift에서 만들어주고 있는 셈

 ```swift
 struct Dog{
    var name: String
    var weght: Int
    var height: Int
 }


 Dog(name: String, weight: Int, height: Int)       
 //기본생성자 + 멤버와이즈 이니셜라이저 제공
 
 ```

 <br/>

그렇다면 이런 경우엔?
```swift
 struct Dog{
    var name: String = "초코"
    var weght: Int = 0
    var height: Int
 }


 Dog(height: Int)    
 Dog(name: String, weight: Int, height: Int)       
```
멤버와이즈 이니셜라이저는 기본으로 생성되어지고, 추가로 우리는 height값만 있다면 인스턴스를 찍어낼 수 있기 때문에 height만 파라미터로 받는 생성자도 추가적으로 제공되는 것이다.

 <br/>


```swift
 struct Dog{
    var name: String = "초코"
    var weght: Int = 0
    var height: Int = 0
 }

 extension Dog{
    init(name: String){
        self.name = name
    }
 }

 Dog()
 Dog(name: String, weight: Int, height:Int)
 Dog(name: String)
 //셋 모두 제공
```
원칙적으로는 init을 개발자가 추가했으니, 본체의 멤버와이즈는 작동되지 않아야하는 것이 원칙이나, 구조체에 한해서, 예외적으로 모두 다 제공이 된다.

## 멤버의 확장(서브스크립트)
- 메서드이기 뭐 당연한 소리겠다.

```swift
extension Int {
    subscript(num: Int) -> Int {
        
        var decimalBase = 1
        
        //자릿수 선정(10의 자리, 100의 자리..)
        for _ in 0..<num {  
            decimalBase *= 10
        }
        
        return (self / decimalBase) % 10
        
    }
}
```

<br/>

## 중첩타입
클래스, 구조체 및 열거형에 새 중첩 유형을 추가 가능한 속성이다.
다만, class 안에 class... 어디서 많이 본 듯하다. `오버라이딩` 가능한 타입속성에서 `static` 키워드 대신 `class` 키워드를 사용했었다. 헷갈리지 말자!
> 오버라이드? 슈퍼클래스로부터 상속받은 메서드, 프로퍼티들을 재정의하는 키워드

```swift
class Dongchan {
   class func sayBye() {
        print("Bye")
    }
}
 
class DongchanAlex: Dongchan{
    override class func sayBye() {     
    }
}
```

<br/>
그럼 이제 중첩이 어떤 식으로 이루어지는 지 알아보자

```swift
// Int(정수형 타입)에 종류(Kind) ====> 중첩 열거형 추가해 보기

extension Int {
    
    enum Kind {       // 음수인지, 0인지, 양수인지
        case negative, zero, positive
    }
    
    var kind: Kind {    // 계산 속성으로 구현
        switch self {
        case 0:                   // 0인 경우
            return Kind.zero
        case let x where x > 0:   // 0보다 큰경우
            return Kind.positive
        default:                  // 나머지 (0보다 작은 경우)
            return Kind.negative
        }
    }
}
```
<br/>
????그럼 rawValue 사용할 수 있나?

```swift
Int.Kind.positive.rawValue
Int.Kind.zero
Int.Kind.negative
```
`에러발생` → 열거형 다시 복습해야겠다
열거형 자체에 Int형으로 되어 있지 않기 때문에 아무리 rawvalue로 접근하고 싶어도 그럴 수 없다.
만약 열거형을 Int형으로 바꿔서 정의해준다면 가능하다!
```swift
extension Int {
    
    enum Kind: Int {       // 열거형 Int형으로 교체
        case negative, zero, positive
    }
    
    var kind: Kind {    
        switch self {
        case 0:                   
            return Kind.zero
        case let x where x > 0:   
            return Kind.positive
        default:                 
            return Kind.negative
        }
    }
}

Int.Kind.positive.rawValue      //2
```



```toc
```