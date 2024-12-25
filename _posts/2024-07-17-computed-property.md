---
title: "iOS스터디 week3, 계산속성과 저장속성"
layout: single
Typora-root-url: ../
categories: iOS
tag: TIL
use_math: true
---

## 구조체(struct) and 클래스(class)
이 둘은 모두 객체지향 → 공통적으로 속성값(properties)은 같다.
> 저장속성, 지연(Lazy)저장속성, 계산속성(Computed), 타입속성, 속성 감시자(observer)

공통적인 속성값 중, 계산속성과 타입속성에 대해 알아보자.

<br/>

### 계산속성(Computd Properties)
값이 일반적으로 저장되는 일반적인 속성(변수)를 저장 속성이라고 한다.

```swift
#여태 배웠던 방식

class Person {
    var name: String = "사람"
    var height: Double = 160.0
    var weight: Double = 60.0
    
    func calculateBMI() -> Double {
        let bmi = weight / (height * height) * 10000
        return bmi
    }
}
```
지금까지는 변수선언 부 밑에 매서드를 작성하여 간단하게 리턴하는 방식을 사용하였다. 이 방식은 단순이 계산 속성이라기 보다는, 저장속성으로 저장 value들을 이용하여 계산 문법을 실행한 결과값을 리턴했다고 보는게 맞겠다.

```Swift
#계산속성 적용하기

class Person1 {
    var name: String = "사람"
    var height: Double = 160.0
    var weight: Double = 60.0
    
    var bmi: Double {
        get {                                               
            let result = weight / (height * height) * 10000
            return result
        }
    }
}
```
밖에서 해당 인스턴스에 접근해서 "get", 값을 얻는다는 의미이다. 어떻게 보면 저장속성이라고 볼 수 있지만, 리눅스의 Read속성과 비슷해보인다. 단순히 값을 읽어오는 것도 컴퓨터로서는 계산으로 판단하게 되는 것 같아보임.

- 항상 다른 저장 속성에 의한 결과로 계산해 나오는 그런 방식의 메서드인 경우
- 아예 속성처럼 만들 수 있다. (= 계산 속성)

```swift
class Person2 {
    var name: String = "사람"
    var height: Double = 160.0
    var weight: Double = 60.0
    
    var bmi: Double {
        get {        //getter
            let bmi = weight / (height * height) * 10000
            return bmi
        }
        set(bmi) {   //setter
            weight = bmi * height * height / 10000
        }
    }
}

let p2 = Person2()
p2.height = 165 
p2.weight = 65  


p2.bmi      //23.875
p2.bmi = 25    //weight 속성 자동 변경(set속성)

p2.weight       //68.0625 변경되어 있음

```
밖에서 해당 인스턴스에 접근해서 "set", 값을 세팅(설정)한다는 의미

```swift
class Person3 {
    var name: String = "사람"
    var height: Double = 160.0
    var weight: Double = 60.0
    
    var bmi: Double {
        let bmi = weight / (height * height) * 10000
        return bmi
    }
}
```
get블록만 있다면, 혹은 set을 쓸 필요가 없다면, 굳이 get{}으로 한번 더 감쌀 필요가 없다.

```swift
class Person4 {
    var name: String = "사람"
    var height: Double = 160.0
    var weight: Double = 60.0
    
    var bmi: Double {
        get {
            let bmi = weight / (height * height) * 10000
            return bmi
        }
        set {
            weight = newValue * height * height / 10000
        }
    }
}
```
set블록의 파라미터를 생략하고 'newValue'로 대체가능하지만, 가독성에 좋은지는 잘 모르겠다. (기왕이면 메서드답게 코드 작성하는게.... 예외를 두면 더 헷갈림;;;)

<br/>

### 계산속성의 특징

메서드가 아닌, 속성방식으로 구현?
-  외부에서 보기에 속성이름으로 설정가능하므로 보다 명확해 보임
-  계산 속성은 실제 메모리 공간을 가지지 않고, 해당 속성에 접근했을때 다른 속성에 접근해서 계산한후, 그 계산 결과를 리턴하거나 세팅하는 메서드

### 계산속성의 주의점
 - 항상 변하는 값이므로, var로 선언해야함 (let로 선언불가)
 - 자료형 선언을 해야함(형식추론 형태 안됨) (메서드이기 때문에 파라미터, 리턴형이 필요한 개념)
 - get은 반드시 선언 해야함(값을 얻는 것은 필수, 값을 set하는 것은 선택)

<br/>


### 타입속성 (Type Properties)
static이라는 키워드를 붙여서 사용하는 저장속성이다.
- 일반 저장속성은 인스턴스를 생성할때, 생성자에서 모든 속성을 초기화를 완료.
- 저장 타입(형식) 속성은 생성자가 따로 없기때문에, 항상 기본값이 필요. 생략할 수 없음.
- 반드시 타입(형식)의 이름으로 접근해야한다.

### 저장 속성, 계산 속성 모두 타입속성 가능

1. 타입-저장 속성
let과 var모두 선언 가능하다.
```swift
class Circle {
    
    // 저장타입 속성 (값이 항상 있어야 함)
    static let pi: Double = 3.14
    static var count: Int = 0   // 인스턴스를 몇개를 찍어내는지 확인
    
    // 저장 속성
    var radius: Double     // 반지름
    
    // 계산 속성
    var diameter: Double {     // 지름
        get {
            return radius * 2
        }
        set {
            radius = newValue / 2
        }
    }
    
    init(radius: Double) {
        self.radius = radius
        Circle.count += 1
    }
}

var circle1 = Circle(radius: 2)
Circle.count                      //1

var circle2 = Circle(radius: 3)
Circle.count                      //2

let a = Circle.pi
circle1.pi                      //안됨
```
이와 비슷한게 애플에서 만들어둔 여러가지 타입들이 해당된다.
- Int.max/Int.min 이런 것들이 다 할당되어 있지만,
- 4.min 이런 방식은 존재하지도, 말도 안되기 때문


2. 타입-계산 속성 (class키워드에서만)
```swift
class Circle1 {
    // 저장 타입 속성
    static let pi: Double = 3.14
    static var count: Int = 0
    
    // (계산) 타입 속성(read-only), Get생략
    static var multiPi: Double {
        return pi * 2
    }
    
    // 저장 속성
    var radius: Double     // 반지름
    
    
    // 생성자
    init(radius: Double) {
        self.radius = radius
    }
    
}
```
- 메모리 공간이 할당되어 있지 않음! (메서드 그 자체)
- 계속 바뀌는 값이기 때문에 Var만 선언 가능
<br/>

**Lazy(지연) 속성과 매우 유사한 특징을 갖는다**
- 지연속성 : 인스턴스가 해당 저장값(value)를 호출할때 사용됨 (only var)
- 타입속성 : 특정 인스턴스에 속한 속성이 아니기 때문에 불러올 순 없으나, 타입 자체에서 사용됨. (let, var모두 사용 가능)
- 인스턴스 내에서도 접근하려면 타입이름 + 속성으로 써야 접근 가능함!!








```toc

```
