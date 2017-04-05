---
layout: post
title: 'Swift 기초 - 특징, 변수, 상수, 타입'
author: through.kim
date: 2017-04-05 13:18
tags: [ios, app, study, swift]
image: '/files/covers/ios_app.jpg'
---

출처 : [SWIFTER 블로그](https://swifter.kr/)

> Swift에 대한 기본 개념을 정리하며 공부해본다.

* TOC  
{:toc}

# 스위프트의 특징

Swift는 정적인 프로그래밍 언어이다. 정적 프로그래밍 언어란, 컴파일 등 프로그래밍 언어가 실행되기 이전 단계에서 변수/상수의 타입을 결정하는 언어를 의미한다. 정적인 언어는 동적 언어와 비교해 실행이 되기 전에 프로그램의 타당성이 검증되기 때문에 안정성이 높으며, 대규모 프로그램 개발에 적합하다는 특징을 갖는다.

Swift는 nil 허용을 제어가능하다. nil이란 값이 존재하지 않는 것을 의마하는데, 변수/상수가 초기화되지 않은 상태나 참조가 존재하지 않는 상태를 의미한다. Swift는 기본적으로 변수/상수에 nil을 대입할 수 없다. 대신 Optional<Wrapped>타입을 가지며, 이 타입을 이용해 nil에 대한 허용을 제어할 수 있다.

```swift
let a: Optional<Int>
let b: Int
a = nil // no error
b = nil // complie error
```

Swift는 타입 추론구조가 있어서, 변수/상수에 대입된 값에 따라 컴파일러가 타입을 유추하는 것이 가능하다. 타입 선언을 생략할 수 있어 코드가 보다 간결해지는 효과를 얻을 수 있다. 

Swift는 범용적인 제네릭을 갖는다. 제네릭은 특정 타입에 국한되지 않는 범용적 프로그램을 만들기 위한 기능이다. 일반적인 함수는 인수에 대해 String, Int와 같이 구체적인 타입을 명시해야 되므로 다양한 유형의 인수를 전달받을 수 없다. 제네릭을 사용하면 추상적인 인수 타입을 사용하기 때문에 다양한 타입의 인수를 넘겨받는 것이 가능하다.

```swift
func max<T: Comparable>(_ x: T, - y: T) -> T
```

예시의 제네릭의 경우 T는 타입 인수이며 넘겨받을 인수 x, y에 따라 구체적인 타입이 설정되게 된다. <T: Comparable>은 T가 Comparable 프로토콜을 준수해야 함을 의미하는 구문이다. 또한 x와 y 모두 타입이 T로 설정되어 있기 때문에 같은 타입을 전달해주어야 한다. 예사의 제네릭을 일반 함수로 나타내면 다음과 같다.

```swift
func max(_ x: Int, _ y: Int) -> Int
func max(_ x: String, _ y: String) -> String
```

---

# 변수, 상수, 타입

## 변수와 상수

변수는 var, 상수는 let이라는 키워드를 사용하여 아래와 같이 선언한다.

```swift
var 변수명: type
let 상수명: type
```

또한 선언과 동시에 값을 할당할 수 있다.

```swift
var a: Int = 100
```

100은 Type annotation 없이도 Int형임을 추론할 수 있다. 이런 경우 Type annotation을 생략 가능하다.

```swift
var a = 100
```

Type annotation과 타입 추론에 의해 결정된 타입을 확인하려면 type(of:) 함수를 이용한다.

```swift
let a = "hello"
type(of: a) // String.Type
```

Swift에서는 변수 및 상수 값이 존재하지 않는 경우 nil이라는 리터럴로 표현한다. 초기화 되지 않고 nil 리터럴을 사용하면 컴파일 에러가 발생하게 된다.

```swift
var a: Int
print(a) // Complie error
```

변수와 상수의 범위는 전역(Global)과 지역(Local)로 나뉜다. 동일한 범위 내에서는 변수, 상수, 함수 등 종류가 다르더라도 고유한 이름을 사용해야 한다. 또한 전역 변수를 지역에서 사용하는 것은 가능하지만, 그 반대는 불가능하다.

## Bool 타입

참 거짓을 나타내는 true와 false가 Bool 리터럴이다. true와 false를 대입할 때 Type annotation을 해주지 않으면 Bool 타입으로 입력된다.

```swift
let a = true
let b = false
let c = !a  // false
```

## Numeric 타입

대표적으로 Int 타입이 있다. Swift에서는 Int 이외에도 다양한 수치 자료형이 존재한다. 수치를 나타내는 리터럴을 숫자 리터럴이라고 하며, 정수 리터럴과 부동소수점 리터럴로 나뉜다. 정수 리터럴의 경우 Int 외에도 Int8, Int16, Int32, Int64에 대입이 가능하며, 부동소수점 리터럴의 경우 Type annotation이 되지 않았다면 Double 타입으로 인식된다.

Swift는 같은 정수리터럴 사이 혹은 같은 부동소수점 리터럴 사이라도 타입이 다르면 할당할 수 없고, 이니셜라이저를 통해 형변환을 해주어야 한다.

```swift
let a: Int = 100
let b: Int64 = Int64(a)

let x: Double = 1.4
let y: Float = Float(x)
```

## String 타입

Swift는 값을 큰따옴표(")로 둘러 싸면 문자열 리터럴로 인식한다. \\(값)을 이용하면 원하는 값을 문자열 리터럴 내에 포함시킬 수 있다.

```swift
let result = 3 + 7
let s: String = "결과는 \(result) 입니다."
```

"a"와 같은 단일문자는 Type추론을 통해 String으로 인식되기 때문에 Character 타입을 원한다면 Type annotation을 해주어야 한다.

```swift
let a1 = "a"                // String.Type
let a2: Character = "a"     // Character.type

let b1 = "hello"            // String.Type
let b2 = b1.characters      // String.CharacterView
```

String.CharacterView 타입은 Character타입의 컬렉션을 나타낸다.

숫자 리터럴과 상호 변환하기 위해서는 이니셜라이저를 사용하면 된다. String > Int 타입 변환에서 Int 타입으로 나타낼 수 없는 경우 nil을 반환한다.

```swift
let a1 = 100
let a2 = String(a1)     // "100"

let b1 = "hello"
let b2 = Int(b1)        // nil
```

## Array\<Element\> 타입

배열은 순서를 가진 컬렉션으로 Element에 구체적인 타입을 선언하여 Array(String) 이나 Array(Int) 타입 등으로 구현할 수 있다. 

Array\<Element\>에는 [1,2,3,4,5]와 같은 배열 리터럴을 대입할 수 있다. Array\<Element\>에 아무런 값도 대입하지 않으면 컴파일러는 PlaceHolder로 Element 타입을 추론한다. 값이 대입 되면 해당 값을 바탕으로 타입을 추론하게 된다. 빈 배열리터럴일 경우 Type annotaiton을 해주어 타입을 명시해주는 것이 좋다. 요소 내에 다른 타입이 섞여있으면 컴파일 에러가 발생한다.

```swift
let i = [1,2,3,4,5]     // [Int] Type
let s = ["a", "b", "c"] // [String] Type
let ia = [[1,2], [3,4]] // [[Int]] Type
let x = [1, 2, "a"]     // Complie Error
```

Array\<Element\> 조작은 다음과 같이 할 수 있다.

```swift
var list = [1,2,3]
list[1]                 // 2
list[1] = 4
list                    // [1,4,3]
list.append(2)          // [1,4,3,2]
list.insert(0, at: 0)   // [0,1,4,3,2]
list.remove(at: 2)      // [0,1,3,2]
list.removeAll          // []
```

## Dictionary\<Key, Value\> 타입

딕셔너리 타입은 키와 값의 쌍을 갖는 컬렉션이며, 키를 값에 접근하기 위한 용도로 사용한다. 키-값 쌍사이에 순서는 상관없다. 키는 유니크하며, 값은 중복이어도 상관 없다. Array\<Element\>와 마찬가지로 Dictionary\<Key, Value\>도 Key와 Value라는 PlaceHolder를 갖는다.

Dictionary\<Key, Value\>에는 ["key1": "value1", "key2": "value2"]와 같은 사전 리터럴을 대입할 수 있다. 키가 String, 값이 Int 타입인 사전 리터럴은 [String: Int]타입으로 추론된다. 

```swift
let a = [1: "a", 2: "b", 3: "c"]    // [Int: String]
```

Dictionary\<Key, Value\>는  Dictionary\<Key: Hashable, Value\>타입이며, Key는 Hashable 프로토콜을 지켜야한다는 의미이다. Int 타입이나 String 타입은 Hashable 프로토콜을 지키기 때문에 HashValue를 갖지만 Array\<Element\>타입은 HashValue를 갖지 않기 때문에 Key로 사용할 수 없다.

Dictionary\<Key, Value\>는 다음과 같이 조작할 수 있다. 

```swift
let a = ["a": 1]
let b = a["a"]  // 1 (Optional<Int> 타입이다.)

let dic = ["key1": "val1"]
let v1 = dic["key1"] != nil   // true
let v2 = dic["key2"] != nil   // false

// 변경
var a1 = ["key": 1]
a1["key"] = 3
a1 // ["key": 3]
 
// 추가
var a2 = ["key1": 1]
a2["key2"] = 2
a2 // ["key1": 1, "key2": 2]
 
// 삭제
var a3 = ["key": 1]
a3["key"] = nil
a3 // [:]
```

## Optional\<Wrapped\> 타입

Swift 언어의 변수와 상수는 기본적으로는 nil을 허용하지 않지만, nil 허용이 필요한 경우 Optional\<Wrapped\>타입을 사용한다. 이 때 Wrapped형은 Optional\<Int\>와 같이 구체적인 타입을 지정해 사용하게 된다.

Optional\<Wrapped\>타입은 Wrapped 타입의 값의 여부에 따라 두가지로 나타난다.

```swift
enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}
```

.none과 .some 두가지 케이스를 정의하고 있는데, .none은 값이 없는, nil 인 경우이고, .some은 값이 존재하는 경우이다.

```swift
let n = Optional<Int>.none
print(".none: \(n)")        // nil

let s = Optional<Int>.some(5)
print(".some: \(s)")        // Optional(5)
```

enum 케이스보다 더 간단하게 Optional\<Wrapped\>를 생성하려면 다음과 같이 할 수 있다.

```swift
var a: Int?

a = nil             // nil 리터럴 대입에 의한 .none 생성
a = Optional(5)     // 이니셜라이저에 의한 .some 생성
a = 3               // 대입에 의한 .some 생성
```

단 nil리터럴은 기본 타입이 정해지지 않았기 때문에 Type annotation을 우선적으로 해주어야 한다.

```swift
let a: Int? = nil
let b = nil         // 타입이 결정되지 않았기 때문에 컴파일 에러 발생
```

또한 .some값을 인수로 받아와 Optional(_:)을 사용하여 생성이 가능하며, 이니셜라이저로 전달된 인수를 기준으로 타입 추론이 이루어진다.

```swift
let a = Optional(10)        // Optional<Int>
let b = Optional("abc")     // Optional<String>
```

Optional\<Wrapped\>타입은 값을 갖고 있지 않은 상태도 고려해야 하며, Optional\<Wrapped\>에서 \<Wrapped\>값을 추출해 내는 데 다음과 같은 방법이 존재한다

- Optional Binding
- ?? 연산자
- 강제로 풀기(Forced Unwrap)

Optoinal Binding은 조건 분기문이나 반복문 조건을 이용해 Optional\<Wrapped\> 타입 값의 유무에 따라 처리를 전환하는 방법이다. Optional Binding은 if-let문을 이용해 처리한다.

```swift
let a = Optional("한")       //Optional<String> 타입

if let r = a {
    // do something         // a값이 존재하는(nil아 아닌)경우에 실행
}
```

?? 연산자는 Optional\<Wrapped\>에 값이 없을경우의 기본값을 지정해주는 것이다.

```swift
let a: Int? = 10
let b = a ?? 3      // 10
```

강제로 풀기는 ! 연산자에 의해 Optional\<Wrapped\> 타입에서 Wrapped타입의 값을 강제 추출하는 방법이다.

```swift
let a: Int? = 10
let b: Int? = 3

a! + b!     // 13
```

값의 존재가 명확한 부분이 아니면 강제로 풀기는 권장하지 않는 방법이다. 값이 존재하지 않으면 프로그램이 종료되기 때문이다.


Optional Chain은 Unwrap을 하지 않고 Wrapped 타입의 속성이나 함수에 접근하는 방식이다. Optional\<Wrapped\>타입에서 Wrapped타입의 속성에 접근하려면 일단 Optional Binding으로 접근해야 한다.

```swift
let a = Optional(10.0)
leb b: Bool?

if let x = a {
    b = x.isInfinite
} else {
    b = nil
}
print(b)    // Optional(false)
```

Wrapping하지 않고 Wrapped 타입의 속성이나 함수에 접근할 수 있는 방법도 있다.

```swift
let a= Optiona(10.0)
let b = a?.isInfinite

print(b)    // Optional(false)
```

## Any 타입

Any타입은 모든 타입을 대입할 수 있다. 

```swift
let s: Any = "한글"
let i: Any = 123123
```

Any타입에 리터럴을 대입하면 원래 타입의 정보가 손실되어 기존 타입에서 가능했던 작업을 동일하게 진행하는 것은 불가능하다.

```swift
let a: Any = 1
let b: Any = 2

a + b   // Complie error
```

## 튜플 타입

여러가지 타입을 하나의 타입으로 모아서 취급하는 것이 튜플 타입이다. 튜플의 요소가 되는 타입을 ()으로 구분하여 열거하면 되며, 타입이나 갯수의 제한이 없다.

```swift
var t: (Int, String)
t = (1, "a")

var u: (Int, String, Double)
u = (1, "a", 12.3)
```

튜플의 요소에 접근하려면 튜플을 정의할 때 요소이름을 지정하면 가능하다.

```swift
let t = (int: 1, string: "a")
let i = t.int       // 1
let s = t.string    // "a"
```

## 타입 캐스팅

- 타입판정  
값의 타입을 확인하려면 is 연산자를 사용하면 된다.

```swift
let a: Any = 12
let b = a is Int // true
```

- 상위캐스팅  
상위 캐스팅은 연산자 as를 사용한다.

```swift
let a = "abc" as Any    // String > Any 상위 캐스팅
```

- 하위캐스팅  
하위 캐스팅은 연산자 as?를 사용한다.

```swift
let a = 12 as Any
let b = a as? Int       // Optional(12)
let x = a as? String    // nil
```