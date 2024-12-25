---
title: "inout 입출력 파라미터, gaurd문, 옵셔널 바인딩"
layout: single
Typora-root-url: ../
categories: iOS
tag: TIL
use_math: true
---

기존 파이썬이나 자바스크립트 문법을 알고 있기 때문에, 비슷한 방식의 문법구조와 관련된 건 스킵하기로 했다. 다만 Swift에서 꼭 알아야만 하는 문법과 이 언어의 특징과 같은 문법들은 꼼꼼하게 되짚어보자.<br> *(사실 이게 더 어려움;;)*

## 1. inout 입출력 파라미터

함수를 통해, 변수를 직접 수정하고 싶은 경우, 함수내의 파라미터는 기본적으로 복사되어 전달되는 값타입(str, string, Int...)이며, 임시상수이기 때문에 변경 불가가 원칙임. 

```swift
var num1 = 123
var num2 = 456

func swap(a: Int, b: Int){
  var c = a   //123

  a = b   // a = 456
  b = c   // b = 123
}
```

뭐 파이썬처럼 생각해보면 맞지 않을까? 라는 의문이 들 수 있는게 당연하다 (일단 나부터 개추;;)

실제로는 이 코드는 에러블록을 띄우는데, <strong>a</strong>와 <strong>b</strong>는 파라미터이기 때문에, 전역변수 scope에 있던 변수들이 복사되어 전달된다 (직접 쓰인다 정도로 이해하면 될 듯) 따라서, 원본이 전달되기 때문에 전역변수의 값이 변경되어서는 안되는 변경 불가가 원칙이다.

```swift
var num1 = 123
var num2 = 456

func swap(a: inout Int, b: inout Int){  //직접이 아닌, 주소값이 전달됨(참조)
  var temp = a

  a = b
  b = temp
}

swap(a: &num1, b:&num2) //함수를 실행하고자 할 때는 & 기호를 붙여서 생성해야함
```
이 예시를 보면 파라미터 앞에 inout 키워드를 쓰게되면, 이는 직접 전달 방식보단, 주소값 참조의 의미가 된다. 

내부적으로는, copy-in copy-out 매커니즘인데, 실제 원본이 전달된다고 생각하면 됨. 값을 복사해서 바디 내부로 전달하고, 함수가 종료될 때, 아규먼트로 전달한 변수에 복사됨 (함수 바디 내부에서 외부로 복사되어 전달)   
<br/>

**inout파라미터 사용시 주의해야할 점**
- 상수 let이나, 리터럴을 전달하는 것은 불가능
- 파라미터의 기본값 선언을 허용하지 않음
- 가변 파라미터나 여러개의 파라미터를 선언하여 사용하는 것도 불가능
<br/>
<br/>

## 2. guard문

If 문을 사용할 때 불편한 점? 바로 조건이 너무 많게 될 경우 코드 들여쓰기가 계속 될 수 있다는 것이다.<br/> → 가독성 매우 떨어지게 됨.

guard문을 사용하여 불만족하는 조건을 사전에 걸러낸다.
- else문을 먼저 배치, 먼저 조건을 판별하여 early-exit 가능하게 함.
- 조건을 만족하는 경우 코드가 다음 줄로 넘어가서 계속 실행됨.
- 가드문에서 선언된 변수는 아래문장에서 사용가능하다 (func 자체로는 같은 scope임) => 옵셔널 바인딩에서 더 알아보자.

```swift
  if true {   //서울에 거주하는 경우...
    if true {   //축구를 좋아하는 경우...
      if true {   //남자인 경우....
        if false{    //알파메일일 경우....
          .... 너무 난잡해짐
        }
      }
    }
  }
```
여러가지의 조건을 만족하는 유저를 구한다는 가정을 해보면, 코드가 상당히 들여써질 수 밖에 없다. else도 마찬가지일꺼고. 가드문은 이런 코드가독성을 굉장히 상승시켜줄만한 문법이라고 볼 수 있다.

```swift
func checkNumebrOf(password: String) -> Bool{
  guard password.count >=6 else {return false}

  guard 조건 else {return false}

  guard 조건 else {return false}
  ...
  return true
}
```
여전히 뭔 개소리인지 모르겠다면 밑에 예제를 보고 이해해보자

![github-blog-5.png]({{site.url}}/images/2024-07-10-inout-guard-optional/dog.png)
*그냥 갑자기 한국축구가 생각나서 붙여봤다.*

```swift
//if문
func usingIf() {
  var id: String? = nil
  if let str = id {
    if str.count > = 6 {
      print(str)
    }
  }
}
// 조건 2개인데 벌써 토나옴
```

```swift
//guard문
func usingGuard(){
  var id: String? = nil

  guard let str = id else {return}
  guard str.count >= 6 else {return}
}
//조건 2개인데 이 정도차이라면, 실무에선 어떻겠...
```
중요한건! 무조건 Return키워드를 통해서 무조건 scope를 탈출해야한다!
<br>
<br>

## 3. 옵셔널 바인딩

옵셔널 타입에 대해서는....(그냥 알아들었으면 끄덕여)

음..그래서 옵셔널 타입(임시타입)을 추출하는 방법에는 4가지가 있다.

1. **강제추출** : nil이 아닌 값이 있다는 것을 확신해서 강제로 값을 추출하는 방법

```swift
num!
```
2. **nil이 아닌지 확인 후, 강제추출** : If문을 통해 nil이 아님을 먼저 확인 후, 강제로 추출한다. nil이면 상수에 안담김. nil이 아니면, 상수에 담김 
```swift
if num != nil {
  print(num!)
}
```

3. **옵셔널 바인딩** : 바인딩(상수나 변수에 대입)이 되는 경우만 특정 작업을 하겠다는 의미. 바인딩이 된다면, 특정 작업을 하겠다는 뜻.

```swift
//if let
if let name = optionalName{ //name이라는 상수에 바인딩이 되면~
  print(name)
}
```

```swift
//guard let
func doSomething(name: String?){
    guard let n = name else {return}
    print(n)
}
// 누가봐도 if문 보다 편함. 같은 Scope내에서 가드문 변수는 사용가능하기 때문
```

> 만약 옵셔널 값이 nil값이라면 어떻게 반응할까?
> <br/>이때는 메모리에 담기질 않기 때문에 그냥 실행되질 않음. guard문의 경우 exit.

4. **Nil-Coalescing** : 옵셔널 표현식 뒤에 기본값을 제시하여 옵셔널의 가능성을 없앰.

```swift
optionalName ?? "홍길동"    //앞의 옵셔널표현식이 nil이라면, 기본값을 제시한다.
// = optionalName !=nil ? optionalName!: "홍길동"


let str : String? = "안녕하세요" //optional타입
var hellp = "인사하겠습니다" + (str ?? "say hi")

// 인사하겠습니다 안녕하세요
// 인사하겠습니다 say hi
```

옵셔널 타입에 대하여, 기본 값을 제시할 수 있을 때 사용 <br> 직접 값을 벗겨서 사용하는 것은 아니고, 디폴트 값 제시를 통해 옵셔널 가능성을 없애는 방법이다.

<br>

**옵셔널 타입의 파라미터 사용**
<br/>일반적으로는 String? = nil 이런식으로 사용하여 함수를 유연하게 사용하게 함. 

```swift
func doSomething(with label: String, name: String? = nil){  
    print("\(label): \(name)")
}

doSomething(with: "레이블", name: "스티브잡스")
doSomething(with: "레이블", name: nil) 
//이런식으로 계속 빈 레이블에 Nil을 넣어줘야하는 번거로움을 덜어낼 수 있음

```