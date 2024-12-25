---
title: "iOS스터디 week2, 컬렉션(collection)"
layout: single
Typora-root-url: ../
categories: iOS
tag: [TIL, Collection]
use_math: true
---

## 스위프트 컬렉션

데이터를 효율적으로 관리하기 위한 자료형 타입이다. 
- Array : 데이터를 순서대로 저장하는 컬렉션
- Dictionary : key: value 하나의 쌍으로 관리 (X순서)
- Set : X순서, X중복

## Array 배열
파이썬에서의 `리스트` 와 많이 유사한 형태이다

```swift
// 정식문법
let strArray1: Array<Int> = [1,2,3,4,5]


// 단축문법
let strArray2: [String] = [1,2,3,4,5]
```

 - 배열의 문법 약속
 - [] 대괄호로 묶는다. 배열의 인덱스의 시작은 0부터 (모든 프로그래밍 언어 공통 사항)
 - 1개의 배열에는 동일한 타입의 데이터만 담을 수 있다.
 - (순서가 있기 때문에) 값은 중복 가능

### 배열의 기본기능

```swift
numsArray.count   //영어단어 그대로...
numsArray.isEmpty


numsArray.contains(1)  //파라미터로 값을 전달
numsArray.contains(8)


numsArray.randomElement()


numsArray.swapAt(0, 1)

```
### 배열의 각 요소(element)에 대한 접근

서브스크립트를 이용해서 각 요소에 효율적으로 접근할 수 있다. 이 점 또한 파이썬과 매우 유사한 형태
```swift
stringArray.first   // 리턴값 String?  ====> 빈 배열이라면 nil 리턴
stringArray.last


// 배열의 시작 인덱스
stringArray.startIndex

stringArray.endIndex
//stringArray.endIndex.advanced(by: -1)


stringArray[stringArray.startIndex]

stringArray[stringArray.endIndex - 1]

stringArray.firstIndex(of: "iOS")     // 앞에서 부터 찾았을때 "iOS"는 배열의 (앞에서부터) 몇번째

stringArray.lastIndex(of: "iOS")     // 뒤에서 부터 찾았을때 "iOS"는 배열의 (앞에서부터) 몇번째



if let index = stringArray.firstIndex(of: "iOS") {
    print(index)
}

```
스위프트를 배우면서 제일 많이 느끼는건, 에러처리에 굉장히 쫄아있는(?) 느낌이다.

파이썬에 비하면 정말 과할 정도로 많은 함수를 지원하는데, 공통적으로 모든함수에서, '너가 찾으려는 게 없을 수도 있어~'를 대비하여 옵셔널 처리를 하여 결과값을 보여준다. 물론, 해당값을 생으로 쓰려면 옵셔널 바인딩과 같은 옵셔널을 벗기는 과정을 한번 더 거쳐야하지만, 에러가 나는 것 대신 nil이 나오는 건 나쁘지 않은 것 같다.

### 배열의 삽입, 교체, 추가 삭제 및 기타기능

이것도 타 언어들과 매우 비슷한 기능인 것 같아서 간단하게만.
```swift
var alphabet = ["A", "B", "C", "D", "E", "F", "G"]

alphabet.insert("c", at: 4)
alphabet.insert(contentsOf: ["a","b","c"], at: 0)
alphabet.insert(contentsOf: ["a","b","c"], at: alphabet.endIndex)


// 요소 교체하기
alphabet[0] = "a"
alphabet[0...2] = ["x", "y", "z"]
alphabet[0...1] = []

//교체하기 함수 문법
alphabet.replaceSubrange(0...2, with: ["a", "b", "c"])

//추가하기 
alphabet += ["H"]

alphabet.append("H")   // 맨 마지막에 추가하는 것
alphabet.append(contentsOf: ["H", "I"])
alphabet.append(7)   // 에러 ===> 동일한 자료형만 저장가능함

//삭제(제거하기)
alphabet[0...2] = []   //빈배열 전달하면 해당 범위가 삭제


// 요소 한개 삭제
alphabet.remove(at: 2)  // 삭제하고, 삭제된 요소 리턴
alphabet.remove(at: 8)    // 잘못된 인덱스 전달 ====> 에러발생


// 요소 범위 삭제
alphabet.removeSubrange(0...2)
alphabet.removeFirst()   // 맨 앞에 요소 삭제하고 삭제된 요소 리턴 (리턴형 String)
alphabet.removeFirst(2)   // 앞의 두개의 요소 삭제 ===> 리턴은 안함
alphabet.removeLast()   // 맨 뒤에 요소 삭제하고 삭제된 요소 리턴 (리턴형 String)
alphabet.removeLast(2)

//배열이 비어있는지도 잘 확인해보고 삭제(제거)해야함 =======> 에러
alphabet.removeFirst()    // 리턴형 String
alphabet.removeLast()     // 리턴형 String

// 배열의 요소 모두 삭제(제거)
alphabet.removeAll()
alphabet.removeAll(keepingCapacity: true)  // 저장공간을 일단은 보관해 둠. (안의 데이터만 일단 날림)
```
메모리와 관련된 배열함수도 있는게 신기하다.

1. 배열을 직접정렬하는 메서드  sort  (동사)
2. 정렬된 새로운 배열을 리턴  sorted (동사ing/동사ed)

```swift
nums.sort()   // 배열을 직접 정렬 (오름차순) (배열리턴 하지 않음)
nums.sorted()    //오름차순으로 정렬


nums.reverse()   // 요소를 역순으로 정렬  (배열리턴 하지 않음)
nums.reversed()

nums.sorted().reversed()
// 새로운 배열은 생성하지 않고, 배열의 메모리는 공유하면서 역순으로 열거할 수 있는 형식을 리턴

nums.suffle()
nums.suffled()
```

배열끼리의 비교도 가능하다
```swift
let a = ["A", "B", "C"]
let b = ["a", "b", "c"]

// 두 배열을 비교해보기

a == b   // false
a != b   // true

// 개별요소 비교, 저장순서도 비교 하기 때문에
```
배열에서는 `순서` 가 가장 중요한 특징이기 때문에, 이 부분도 비교할 요소로 들어간다는 점 알아두어야 할 꺼 같다.

```swift
for i in nums {
    print(i)
}
//Python이랑 형태가 매우 동일..

//  enumerate 
// (offset: 0, element: 10)


for tuple in nums.enumerated() {
    //print(tuple)
    print("\(tuple.0) - \(tuple.1)")
}


for tuple in nums.enumerated() {      // 바로 뽑아내기
    print("\(tuple.offset) - \(tuple.element)")
}


for (index, value) in nums.enumerated().reversed() {      // 거꾸로 뒤에서 부터
    print("\(index) - \(value)")
}
```
파이썬도 enumerated 함수가 있는데, 특이한 건 튜플에 key/value가 offset, element라는 이름으로 붙여져서 리턴되고, 이를 직접 해당명명법으로 호출까지 되는 점이 신기한거 같다.
(생각보다 세련된 언어라는 게 느껴지는 부분)

<br/>

## Dictionary 딕셔너리

 - 딕셔너리의 문법 약속
 - [] 대괄호로 묶는다. (쌍을 콜론으로 처리)
 - 키값은 유일해야함 / 중복 불가능(구분하는 요소이기 때문에) 밸류값은 중복 가능
 - 1개의 딕셔너리에는 동일한 자료형 쌍의 데이터만 담을 수 있음
 - 키값은 Hashble 해야함

```swift
// 단축문법
var words: [String: String] = [:]

// 정식문법
let words1: Dictionary<Int, String>


var dic = ["A": "Apple", "B": "Banana", "C": "City"]   // 딕셔너리 리터럴로 생성해서 저장
let dic1 = [1: "Apple", 2: "Banana", 3: "City"]    // 내부적으로 순서가 존재하지 않음
```

#### Hashable?
해시테이블 알고리즘은, 앞에서부터 탐색하는 완전탐색, 양쪽방향에서 찾는 이진탐색보다 훨씬 빠른 알고리즘이다. 
> 도서관의 책들이 잘 정리되어 있으면 우리는 우리가 원하는 책을 쉽게 열람할 수 있다.(= 해시테이블) 하지만 오래된 책방 뒤죽박죽 되어 있는 책들 중에 우리가 원하는 책을 찾으려면 꽤 시간이 걸릴 것이다. (완전탐색/이진탐색)

근데 이게 천하무적일까? 모든 알고리즘에는 장점이 있다면 단점이 있는 법. `Hash Collision` 해시충돌이 이 알고리즘의 단점이라고 볼 수 있다. 

- Hash라는 알고리즘은 고유의 key값을 생성해서 단 하나의 자료를 찾는데 쓰인다고 하지만, `고유`의 원칙은 참 어렵다고 한다. 즉, 무한히 생성될 수 있는 Key값에 유한한 Hash값을 매칭한다면, 분명 겹치는 hash가 생길 수 밖에 없다는게 이 알고리즘의 단점이다.

- Hash의 저장 단계의 시간복잡도는 0(1)이다. key는 고유하며 해시함수의 결과로 나온 해시에 매칭되는 value를 찾으면 되기 때문이다. 하지만, 최악의 경우 0(n)이 될 수 있다. 해시 충돌로 인해 모든 bucket의 value들을 찾아봐야 하는 경우를 고려해야하기 때문.

### 빈 딕셔너리 생성

```swift
let emptyDic1: Dictionary<Int, String> = [:]
let emptyDic2 = Dictionary<Int, String>()
let emptyDic3 = [Int: String]()

var dictFromLiteral = [:]    // 타입 정보가 없으면 유추할 수가 없다.
```
여기서는 타입추론이 안되나봄...

### 딕셔너리의 여러가지 기능
```swift
dic = ["A": "Apple", "B": "Banana", "C": "City"]

dic.count
dic.isEmpty

dic.randomElement()      // Named Tuple 형태로 리턴
// optional(key: "C", value: "City")
```

```swift
//딕셔너리는 기본적으로 서브스크립트[ ]를 이용한 문법을 주로 사용
// 딕셔너리
dic = ["A": "Apple", "B": "Banana", "C": "City"]



dic["A"] + "김동찬"      // nil의 가능성 =====> String?(옵셔널)
//Value of optional type 'String?' must be unwrapped to a value of type 'String'



if let a = dic["A"] {    // 옵셔널 바인딩
    print(a)
} else {
    print("Not found")
}

dic["S", default: "Empty"]       // nil이 발생할 확률이 없음
// 자료가 없으면 기본값을 리턴하는 문법  ===> 리턴 String


dic.keys
dic.values

dic.keys.sorted()
dic.values.sorted()


for key in dic.keys.sorted() {     // 오름차순 정렬  ===> "return 배열" 배열이 됨
    print(key)
}
```

### 딕셔너리 업데이트(update) - 삽입하기/교체하기/추가하기

딕셔너리는 append 함수를 제공하지 않는다. 
- append는 순서가 있는 컬렉션의 끝에 추가하는 개념
- 딕셔너리는 순서가 없기 때문에, update를 통해서 추가

```swift
words = [:]

words["A"] = "Apple"   // 애플로 다시 바꾸기

words["B"] = "Banana"  // 동일한 키가 없으면 ===> 추가하기
words["B"] = "Blue"    // 동일한 키가 있으면 ===> 기존 밸류 덮어쓰기

words.updateValue("City", forKey: "C") // ==> nil


// (정식 기능) 함수 문법 (update + insert = upsert)

words.updateValue("City", forKey: "C")   // 새로운 요소가 추가되면 ==> 리턴 nil

words = [:]      // 빈 딕셔너리로 만들기
words = ["A": "A"]   // 전체 교체하기(바꾸기)
```


### 딕셔너리 삭제하기

```swift
dic = ["A": "Apple", "B": "Banana", "C": "City"]


// 요소 삭제해 보기
dic["B"] = nil    // 해당요소 삭제
dic["E"] = nil   // 존재하지 않는 키/값을 삭제 ======> 아무일이 일어나지 않음(에러아님)
dic


// 함수로 삭제해보기
dic.removeValue(forKey: "A")   // 삭제후, 삭제된 값 리턴
dic.removeValue(forKey: "A")   // 다시 삭제해보기 ===> nil리턴


// 전체 삭제하기
dic.removeAll()
dic.removeAll(keepingCapacity: true) //여기도 있넨..
```


### 딕셔너리 기타등등

```swift
let a = ["A": "Apple", "B": "Banana", "C": "City"]
let b = ["A": "Apple", "C": "City", "B": "Banana"]


// 비교 연산자
a == b   // true
// 딕셔너리는 원래 순서가 없기 때문에, 순서는 상관없음
// (순서 상관없이 무조건 true나옴 - Hashable하기 때문에, 순서 상관없이 비교가능)

a != b   // false
```

```swift
// 딕셔너리의 중첩 사용

var dict1 = [String: [String]]()


dict1["arr1"] = ["A", "B", "C"]
dict1["arr2"] = ["D", "E", "F"]

//["arr1": ["A", "B", "C"], "arr2": ["D", "E", "F"]]

dict1["arr"] = "A"  // ======> Cannot assign value of type 'String' to subscript of type '[String]'



var dict2 = [String: [String: Int]]()

dict2["dic1"] = ["name": 1, "age": 2]
dict2["dic2"] = ["name": 2, "age": 4]
// ["dic2": ["age": 4, "name": 2], "dic1": ["age": 2, "name": 1]]
```
얘도 마찬가지로 타입이 정해졌으면 섞이지 말고, 하나로 통일해야 한다.

```swift
let dict = ["A": "Apple", "B": "Banana", "C": "City"]

for (key, value) in dict {
    print("\(key): \(value)")
}


for (key, _) in dict {
    print("Key :", key)
}


for (_, value) in dict {
    print("Value :", value)
}
```
딕셔너리는 열거하지 않아도, Named 튜플 형태로 하나식 꺼내서 전달, 순서가 없으므로, 실행마다 순서가 달라짐!!!
- 데이터 바구니이기 때문에, 차례대로 하나씩 꺼내서 사용하는 경우가 많을 수 있어서 활용 가능

## Set 세트

수학에서의 집합과 비슷한 연산을 제공하는 컬렉션
 - Set의 문법 약속
 - 생김새는 배열과 같음(따라서, 생성시 타입을 선언 해야함)
 - 수학에서의 집합과 동일하기 때문에 요소는 유일해야함(순서가 존재하지 않음)

**Set의 값과 Dictionary의 키값은 Hashable**
1. 정렬순서보다 검색속도가 중요한 경우에 사용
2. 검색에 내부적으로 Hashing 알고리즘 사용<br/>(hashing ===> 특정값을 고정된 길이의 값으로 변환하는 기법으로 인덱싱과 암호화에서 자주 사용됨)
3. 집합의 수학적인 개념(합집합/교집합/차집합/대칭차집합)을 이용할 필요가 있을 때 <br/>(집합을 계산하기 간편한 함수를 내장)


```swift
// 단축문법
let set1: Set = [1, 2, 3]

// 정식문법
let set2: Set<Int> = [1, 2, 3]

// 빈 세트 생성
let emptySet: Set<Int> = []
let emptySet1 = Set<Int>()
```

### There is no append func.
Set는 append 함수를 제공하지 않음 => append는 순서가 있는 컬렉션의 끝에 추가하는 개념
- Set은 순서가 없기 때문에, update를 통해서 추가
- 당연하게도 서브스크립트 관련 문법 없음!!

```swift
set.update(with: 1)     // Int?
set.update(with: 7)     // 새로운 요소가 추가되면 ====> 리턴 nil




var stringSet: Set<String> = ["Apple", "Banana", "City", "Swift"]


// 요소 삭제해 보기
stringSet.remove("Swift")     // "Swift" 삭제한 요소를 리턴
stringSet                     // ["Hello", "Apple"]

// 존재하지 않는 요소를 삭제해보기
stringSet.remove("Steve")       // nil    (에러는 발생하지 않음)

// 전체요소 삭제
stringSet.removeAll()
stringSet.removeAll(keepingCapacity: true)
```
여기 또 있네 메모리;;


### 어찌보면 set를 사용하는 근본적인 이유
**부분집합 / 상위집합 / 서로소**
```swift
a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
b = [1, 3, 5, 7, 9]     // 홀수 모음
c = [2, 4, 6, 8, 10]    // 짝수 모음
d = [1, 7, 5, 9, 3]     // 홀수 모음

// 부분집합 여부를 판단
b.isSubset(of: a)   // true 부분집합 여부
b.isStrictSubset(of: a)   // false 진부분집합 여부


// 상위집합
a.isSuperset(of: b)    // true 상위집합 여부
a.isStrictSuperset(of: b)   // false  진상위집합 여부

// 서로소 여부
d.isDisjoint(with: c)

```
<br/>

- 합집합
```swift
var unionSet =  b.union(c)
b.formUnion(c)      // 원본변경
```
<br/>

- 교집합
```swift
var interSet = a.intersection(b)
a.formIntersection(b)      // 원본변경
```
<br/>

- 차집합
```swift
var subSet = a.subtracting(b)
a.subtract(b)       // 원본변경

```
<br/>

- 대칭차집합
```swift
var symmetricSet = a.symmetricDifference(b)
a.formSymmetricDifference(b)       // 원본변경
```
<br/>

### 특이한 세트의 특징(?)
Set을 정렬하면, 배열로 리턴함 (정렬은 순서가 필요하기 때문)
```swift
var newSet: Set = [1, 2, 3, 4, 5]
var newArray: Array = newSet.sorted() //array 타입으로 반환됨
```
<br/>

## KeyValuPairs
넌 뭐냐<br/>.<br/>.<br/>.

딕셔너리와 유사한 형태이지만, 배열처럼 순서가 있는 컬렉션..?
- 스위프트 5.2 버전에 등장
- 딕셔너리와 비슷한 형태지만, "순서"가 있는 컬렉션
- key값이 해셔블(hashable)일 필요없음 (검색 알고리즘상 빠르지 않음)
- key값이 동일한 것도 가능

*약간 필요할때만 써라~ 이런 느낌이다. 임시적인 타입 같아보임.*

- 배열처럼, 인덱스로 접근 가능
- 요소에서는 튜플방식으로 접근
- append / remove 같은 기능이 없음


```Swift
let introduce: KeyValuePairs = ["first": "Hello", "second": "My Name", "third":"is"]

introduce.count
introduce.isEmpty

introduce[0]

print("\(introduce[0].key)는 \(introduce[0].value) 입니다.")
print("\(introduce[1].key)는 \(introduce[1].value) 입니다.")
print("\(introduce[2].key)는 \(introduce[2].value) 입니다.")
```

딕셔너리이지만, 저장된 순서가 중요할 경우, 또는 데이터가 반복될 경우만 임시적/제한적으로 사용
 

## Copy-On-Write 최적화
코드상에서 값을 복사해서 담는다 하더라도, 실제 값이 바뀌기 전까지는 그냥 하나의 메모리 값을 공유해서 사용한다는 이론.
(메모리를 적게 차지하기 위해서 스위프트 언어가 알아서 내부에서 처리하는 매커니즘)

```swift
var array = [1,2,3,4,5,6,7]

var subArray = array[0...2]
// 여기서 메모리는 따로 저장되지 않고, Array의 일부 메모리를 참조하는 방식으로 메모리를 사용한다 (추가적인 메모리X)
```


<br/>

```toc

```
