---
layout: post
title: '[Swift-Design Pattern] 데코레이터 패턴 (Decorator pattern)'
author: through.kim
date: 2019-09-09 11:17
tags: [study, swift]
image: '/files/covers/ios_app.jpg'
comments: true
---

> 데코레이터 패턴이란 객체간 결합을 통해 기능을 유연하게 확장할 수 있도록 돕는 디자인 패턴이다.

# 예시

네비게이션의 도로 표시 방식을 통해 데코레이터 패턴을 학습해 본다.

## AS-IS

### 도로 표시 클래스

```swift
class RoadDisplay {
    func draw() {
        print("기본 도로 표시")
    }
}
```

draw()를 이용해 기본적인 도로만 표시하는 클래스이다.

### 도로 + 차선 표시 클래스

```swift
class RoadDisplayWithLane: RoadDisplay {
    override func draw() {
        super.draw()
        self.drawLane()
    }

    private func drawLane() {
        print("+ 차선 표시")
    }
}
```

draw()를 호출하면 차선도 함께 표시해주는 클래스이다.

### 클라이언트

```swift
let road = RoadDisplay()
road.draw()                 //기본 도로 표시

let roadWithLane = RoadDisplayWithLane()
roadWithLane.draw()         //기본 도로 표시
                            //+ 차선 표시
```

클라이언트에서 위 두 클래스를 호출해보면 잘 표시되는 것을 확인할 수 있다.

### 문제점

교통량 표시 기능을 추가할 경우 여러 조합이 생긴다.

- 도로 표시
- 도로 + 차선 표시
- 도로 + 교통량 표시
- 도로 + 차선 + 교통량 표시

각각 조합에 대한 클래스를 모두 생성해주어야 한다는 문제점이 발생한다.

##TO-BE

기능을 추상화하는 Display 프로토콜을 생성하고, 이 프로토콜을 따르는 객체를 만든 뒤 클라이언트에서 조합해 사용하는 방법을 사용하면 된다.

### Display 프로토콜과 기본 도로 표시 클래스

```swift
protocol Display {
    func draw()
}

class RoadDisplay: Display {
    func draw() {
        print("기본 도로 표시")
    }
}
```

모든 Display 객체가 따를 기본 프로토콜인 Display와 기본 도로 표시 클래스이다.

### DisplayDecorator 클래스

```swift
class DisplayDecorator: Display {
    var decoratedDisplay: Display

    init(display: Display) {
        self.decoratedDisplay = display
    }

    func draw() {
        self.decoratedDisplay.draw()
    }
}
```

추가 기능(Decorator)들이 따르게 될 공통된 클래스이다. 추가기능들은 이 클래스를 상속받아 사용하게 된다.

### Decorator 클래스들

```swift
class LaneDecorator: DisplayDecorator {
    override func draw() {
        super.draw()
        self.drawLane()
    }

    private func drawLane() {
        print("+ 차선 표시")
    }

}

class TrafficDecorator: DisplayDecorator {
    override func draw() {
        super.draw()
        self.drawTraffic()
    }

    private func drawTraffic() {
        print("+ 교통량 표시")
    }

}
```

DisplayDecorator 클래스를 상속받아 각각 추가 기능을 구현해준다.

### 클라이언트

```swift
let road = RoadDisplay()
road.draw()                     //기본 도로 표시

let roadWithLane = LaneDecorator(display: road)
roadWithLane.draw()             //기본 도로 표시
                                //+ 차선 표시

let roadWithTraffic = TrafficDecorator(display: road)
roadWithTraffic.draw()          //기본 도로 표시
                                //+ 교통량 표시

let roadWithTrafficAndLane = LaneDecorator(display: roadWithTraffic)
roadWithTrafficAndLane.draw()   //기본 도로 표시
                                //+ 차선 표시
                                //+ 교통량 표시
```

클라이언트단에서 데코레이터들을 조합해서 사용하게 된다.
