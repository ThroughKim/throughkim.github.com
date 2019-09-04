---
layout: post
title: '[Swift-Design Pattern] 스트래티지 패턴 (Strategy pattern)'
author: through.kim
date: 2019-09-04 13:30
tags: [study, swift]
image: '/files/covers/ios_app.jpg'
comments: true
---

> Strategy Pattern은 클래스의 행위를 캡슐화 하여 동적으로 행위를 자유롭게 바꿀 수 있도록 돕는 패턴이다.

# 예시

로봇 프로토콜과 그것을 따르는 태권V, 아톰 객체를 생성하는 예시를 통해 스트래티지 패턴을 학습해본다.

## AS-IS

### Robot 클래스

```swift
protocol Robot {
    var name: String { get }
    init(name: String)
    func attack()
    func move()
}

class TaekwonV: Robot {
    var name: String

    required init(name: String) {
        self.name = name
    }

    func attack() {
        print("I launch Missle")
    }

    func move() {
        print("I can only Walk")
    }

}

class Atom: Robot {
    var name: String

    required init(name: String) {
        self.name = name
    }

    func attack() {
        print("I have Punch")
    }

    func move() {
        print("I can Fly")
    }

}
```

### Client

```swift
let taekwonV = TaekwonV(name: "TaekwonV")
print("My name is \(taekwonV.name)")    // My name is TaekwonV
taekwonV.attack()                       // I launch Missle
taekwonV.move()                         // I can only Walk

let atom = Atom(name: "Atom")
print("My name is \(atom.name)")        // My name is Atom
atom.attack()                           // I have Punch
atom.move()                             // I can Fly
```

### 문제점

로봇 클래스를 캡슐화 하여 생성할 수 있도록 하는 기본적인 코드이다. 현재로서는 큰 문제가 없지만 기존 클래스에 수정을 가하거나, 새로운 클래스를 추가하고자 할 때 다음과 같은 문제가 발생하게 된다.

- 기존 행위를 수정 (ex. 펀치 공격의 코드를 수정하고 싶은 경우)
    - 펀치 공격을 사용하는 모든 클래스의 개별 코드를 수정해야 한다.
- 새로운 로봇 클래스를 추가 (ex. 미사일 공격이 가능하고, 날 수 있는 로봇)
    - 태권V의 미사일 공격 코드와 아톰의 비행 코드가 중복된다.


## TO-BE

이 문제를 해결하기 위해 공통적인 행위를 캡슐화 하여 사용한다. 이 예시에서는 공격 행위와 이동 행위를 캡슐화 할 수 있다.

### Strategy 클래스

```swift
protocol AttackStrategy {
    func attack()
}

class MissleStrategy: AttackStrategy {
    func attack() {
        print("I launch Missle")
    }
}

class PunchStrategy: AttackStrategy {
    func attack() {
        print("I have Punch")
    }
}

protocol MoveStrategy {
    func move()
}

class FlyStrategy: MoveStrategy {
    func move() {
        print("I can Fly")
    }
}

class WalkStrategy: MoveStrategy {
    func move() {
        print("I can only Walk")
    }
}
```


공통된 행위를 Abstract 하여 작성해주었다. 개별 행위에 대한 수정 사항이 있을 경우 이 부분을 수정해주면 되고, 새로운 행위의 추가도 이 부분에서만 작업해주면 된다.

### Robot 클래스

```swift
protocol Robot {
    var name: String { get }
    var attackStrategy: AttackStrategy { get set }
    var moveStrategy: MoveStrategy { get set }
    init(name: String, attackStrategy: AttackStrategy, moveStrategy: MoveStrategy)
}

extension Robot {
    func attack() {
        self.attackStrategy.attack()
    }

    func move() {
        self.moveStrategy.move()
    }

    mutating func setAttack(attackStrategy: AttackStrategy) {
        self.attackStrategy = attackStrategy
    }

    mutating func setMove(moveStrategy: MoveStrategy) {
        self.moveStrategy = moveStrategy
    }

}
```

Protocol로 기본적인 Abstract 요소를 정의해주고, 기본 행위를 지정해주기 위해 Extension을 활용한다. Initializer에서 행위에 대한 Strategy의 기본값을 파라미터로 받아 객체를 생성하고, 별도의 Setter 메소드를 두어 행위의 변환이 자유롭도록 만들어주었다.

```swift
class TaekwonV: Robot {
    var name: String
    var attackStrategy: AttackStrategy
    var moveStrategy: MoveStrategy

    required init(
        name: String,
        attackStrategy: AttackStrategy = MissleStrategy(),
        moveStrategy: MoveStrategy = WalkStrategy()
    ) {
        self.name = name
        self.attackStrategy = attackStrategy
        self.moveStrategy = moveStrategy
    }
}

class Atom: Robot {
    var name: String
    var attackStrategy: AttackStrategy
    var moveStrategy: MoveStrategy

    required init(
        name: String,
        attackStrategy: AttackStrategy = PunchStrategy(),
        moveStrategy: MoveStrategy = FlyStrategy()
    ) {
        self.name = name
        self.attackStrategy = attackStrategy
        self.moveStrategy = moveStrategy
    }
}
```

개별 로봇을 생성할 때 인자로 넘겨주는 Strategy의 기본값에 차별을 둔다. 생성 이후에도 Setter 메소드를 통해 행위의 Strategy는 변경 가능하다.

### 클라이언트

```swift
let taekwonV = TaekwonV(name: "TaekwonV")
print("My name is \(taekwonV.name)")    // My name is TaekwonV
taekwonV.attack()                       // I launch Missle
taekwonV.move()                         // I can only Walk

let atom = Atom(name: "Atom")
print("My name is \(atom.name)")        // My name is Atom
atom.attack()                           // I have Punch
atom.move()                             // I can Fly
```

클라이언트의 코드와 살행 결과는 AS-IS와 동일하다.