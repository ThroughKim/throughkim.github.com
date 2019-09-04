---
layout: post
title: '[Swift-Design Pattern] 싱글톤 패턴 (Singleton Pattern)'
author: through.kim
date: 2019-09-04 15:16
tags: [swift, study]
image: '/files/covers/ios_app.jpg'
comments: true
---

> Singleton Pattern은 객체를 하나만 생성하여, 생성된 객체를 어디서든 참조할 수 있도록 하는 패턴이다. Thread-safe 하게 작성하여 멀티스레드에서 사용해도 문제가 없어야 한다.

JAVA와는 다르게, Swift에서는 간결하게 구현이 가능한 듯 하다.

## 코드

### 클래스

```swift
class Printer {
    static let shared = Printer()
    private init() {}

    func printDoc(_ doc: String) {
        print(doc)
    }
}
```

static 변수에 Singleton 패턴을 적용할 클래스의 인스턴스를 할당해주면 된다.

### 클라이언트

```swift
Printer.shared.printDoc("Print Document")
```

클라이언트에서는 Printer 클래스의 shared 변수를 불러와 사용하면 된다.