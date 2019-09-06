---
layout: post
title: '[Swift-Design Pattern] 커맨드 패턴 (Command pattern)'
author: through.kim
date: 2019-09-06 11:21
tags: [study, swift]
image: '/files/covers/ios_app.jpg'
comments: true
---

> Command Pattern은 실행될 기능을 추상화, 캡슐화 하여 한 클래스에서 여러 기능을 실행할 수 있도록 하는 패턴이다.

# 예시

여러 기능을 수행할 수 있는 버튼 클래스를 만들며 커맨드 패턴을 익혀본다.

## AS-IS

### 버튼 클래스와 램프 클래스

```swift
class Lamp {
    func turnOn() {
        print("Lamp On")
    }
}

class Button {
    private let lamp: Lamp

    init(lamp: Lamp) {
        self.lamp = lamp
    }

    func pressed() {
        self.lamp.turnOn()
    }
}
```

Button을 생성할 때 lamp를 인자로 전달받도록 작성되었다.

### 클라이언트

```swift
let lamp = Lamp()
let lampButton = Button(lamp: lamp)
lampButton.pressed()                // Lamp On
```

실행해보면 버튼의 pressed()가 호출 되었을 때 Lamp On 이 출력됨을 확인해볼 수 있다.

### 문제점

현재로서는 큰 문제가 없지만, 추후에 새로운 기능을 추가하거나, 한 버튼에서 여러 역할을 수행할 수 있도록 수정하려면 문제가 발생하게 된다.

- 버튼이 다른 기능을 수행하려면? (ex. 버튼을 누르면 알람이 켜지도록)
    - Alarm 클래스를 만들고, Button의 생성자에서 Alarm의 인스턴스를 인자로 받아 버튼 인스턴스를 생성하여야 함
    - 기존 버튼 클래스를 수정해야함
- 한 버튼에서 순차적으로 다른 기능을 수행하려면?
    - 필요한 기능이 추가될 때 마다 버튼 클래스 코드를 수정해야함

## TO-BE

버튼이 실행할 기능을 커맨드 프로토콜을 이용해 캡슐화 하여 구현할 수 있다.

### 커맨드 프로토콜

```swift
protocol Command {
    func execute()
}
```

커맨드 프로토콜과 준수해야할 메소드인 execute()를 선언해준다.

### 버튼 클래스

```swift
class Button {
    private var command: Command

    init(command: Command) {
        self.command = command
    }

    func setCommand(command: Command) {
        self.command = command
    }

    func pressed() {
        self.command.execute()
    }

}
```

버튼 클래스의 생성자에서 기능을 수행할 대상을 인자로 받아오는 것이 아닌 커맨드 프로토콜을 인자로 받아오는 것으로 변경했다. 또 pressed()가 호출되었을 때 커맨드 프로토콜의 execute()를 호출해주는 것으로 수정했다.

### 램프 클래스와 램프 커맨드 클래스

```swift
class Lamp {
    func turnOn() {
        print("Lamp On")
    }
}

class LampOnCommand: Command {
    private var lamp: Lamp

    init(lamp: Lamp) {
        self.lamp = lamp
    }

    func execute() {
        self.lamp.turnOn()
    }

}
```

램프 클래스는 기존과 같으며, 새로이 추가된 램프 커맨드에서는 램프를 생성자의 인자로 받아온다.
램프 커맨드 클래스에서는 커맨드 프로토콜을 준수하기 위해 execute() 메소드를 선언한 뒤 그것을 램프 클래스의 turnOn() 메소드와 연결해준다.

### 알람 클래스와 알람 커맨드 클래스

```swift
class Alarm {
    func start() {
        print("Alarm Start")
    }
}

class AlarmStartCommand: Command {
    private var alarm: Alarm

    init(alarm: Alarm) {
        self.alarm = alarm
    }

    func execute() {
        self.alarm.start()
    }
}
```

버튼이 동작할 수 있는 알람 클래스를 추가해준다. 새로운 클래스를 추가하더라도, 커맨드 프로토콜만 준수한다면 버튼 클래스를 수정하지 않고 얼마든지 추가할 수 있다.

### 클라이언트

```swift
let lamp = Lamp()
let lampOnCommand = LampOnCommand(lamp: lamp)
let alarm = Alarm()
let alarmStartCommand = AlarmStartCommand(alarm: alarm)

// 램프를 켜는 버튼
let button1 = Button(command: lampOnCommand)
button1.pressed()                               // Lamp On

// 한 번 누르면 알람이 켜지고 두 번 누르면 램프가 켜지는 버튼
let button2 = Button(command: alarmStartCommand)
button2.pressed()                               // Alarm Start
button2.setCommand(command: lampOnCommand)
button2.pressed()                               // Lamp On

```

실행을 해보면 각 기능들이 잘 동작함을 알 수 있다. 커맨드 패턴을 통해 버튼의 새로운 기능을 추가할 때 마다 버튼 클래스를 수정해주어야 하는 번거로움을 없앨 수 있다.