---
layout: post
title: '[Swift-Design Pattern] 옵저버 패턴 (Observer pattern)'
author: through.kim
date: 2019-09-05 14:48
tags: [study, swift]
image: '/files/covers/ios_app.jpg'
comments: true
---

> Observer Pattern은 한 객체의 상태 변화에 따라 다른 객체의 상태도 연동 되도록 1 대 N 객체간 의존 관계를 구성하는 디자인 패턴이다.

# 예시

점수 저장소인 ScoreRecord 클래스와 점수를 출력하는 View 클래스를 이용해 옵저버 패턴을 이해해 본다.

## AS-IS

### 점수 저장 클래스

```swift
class ScoreRecord {
    private var scores = [Int]()
    private var dataSheetView: DataSheetView?

    func setDataSheetView(dataSheetView: DataSheetView) {
        self.dataSheetView = dataSheetView
    }

    func addScore(_ score: Int) {
        self.scores.append(score)
        self.dataSheetView?.update()
    }

    func getScoreRecord() -> [Int] {
        return self.scores
    }

}
```

### 점수 출력 클래스

```swift
class DataSheetView {
    private var scoreRecord: ScoreRecord
    private var viewCount: Int

    init(scoreRecord: ScoreRecord, viewCount: Int) {
        self.scoreRecord = scoreRecord
        self.viewCount = viewCount
    }

    func update() {
        let record = self.scoreRecord.getScoreRecord()
        displayScores(record: record, viewCount: self.viewCount)
    }

    private func displayScores(record: [Int], viewCount: Int) {
        print("List of \(min(viewCount, record.count)) entries:")
        for i in 0..<min(viewCount, record.count) {
            print(record[i])
        }

    }

}
```

### 클라이언트

```swift
let scoreRecord = ScoreRecord()
let dataSheetView = DataSheetView(scoreRecord: scoreRecord, viewCount: 3)
scoreRecord.setDataSheetView(dataSheetView: dataSheetView)

for i in 0...5 {
    let score = i * 10
    print("Adding \(score)")
    scoreRecord.addScore(score)
}
```

실행시켜보면 scoreRecord에 데이터가 추가될 때 마다 DataSheetView가 update 되어 변경 된 데이터를 출력해주는 것을 확인해볼 수 있다.

### 문제점

저장소 클래스인 ScoreRecord 클래스는 출력 클래스인 DataSheetView 클래스를 변수로 갖고, 출력 클래스는 저장소 클래스를 생성자에서 인수로 받아 생성한다. 상호 의존성이 존재하므로 추후 확장성을 고려할 때 다음과 같은 문제가 발생할 수 있다.

- 목록으로만 출력하지 않고, 점수의 평균값, 혹은 최대값 최소값 등을 표기하는 View 클래스를 추가하여 출력하기를 원하는 경우
    - ScoreRecord 클래스에 새로운 변수와, 그 변수에 해당하는 Setter Method들을 추가해주어야 함
    - addScore() 메소드에 모든 View클래스의 update() 메소드 호출을 추가해주어야 함

## TO-BE

이 문제를 해결하기 위해 공통 기능을 추출하여 프로토콜으로 지정하여 일반화 하여야 한다.
Observer Pattern을 이용해 ScoreRecord 클래스를 Subject(관찰대상)으로, View 클래스들을 Observer(관찰자)로 만들어서 해결할 수 있다.

### Observer 프로토콜

```swift
protocol Observer {
    func update()
}
```

Subject로 부터 변경을 통보받을 메소드인 update()를 추상메소드로 지정해준다.

### Subject 프로토콜

```swift
protocol Subject {
    var observers: [Observer] { get set }
}

extension Subject {
    mutating func attach(observer: Observer) {
        self.observers.append(observer)
    }

    func notifyObservers() {
        self.observers.forEach {
            $0.update()
        }
    }

}
```

관찰대상(Subject)에 공통적으로 필요한 메소드들을 프로토콜과 extension을 통해 일반화 해주었다.
관찰자들을 저장할 observer 리스트와, 관찰자 추가 및 관찰자에게 변동 통보 하는 메소드들이 공통적으로 필요하다.

### 점수 저장 클래스

```swift
class ScoreRecord: Subject {
    var observers: [Observer] = []
    private var scores = [Int]()

    func addScore(_ score: Int) {
        self.scores.append(score)
        self.notifyObservers()
    }

    func getScoreRecord() -> [Int] {
        return self.scores
    }

}
```

Subject 프로토콜을 따르기위해 observers 리스트를 변수로 추가해주었다.
또한 addScore()가 호출되었을 때 개별 View 클래스의 update를 호출하는 것이 아닌, notifyObservers()를 호출하여 attach 되어있는 모든 observer에게 변동을 통보할 수 있도록 바꾸어 준다.

### 점수 출력 클래스들

```swift
class DataSheetView: Observer {
    private var scoreRecord: ScoreRecord
    private var viewCount: Int

    init(scoreRecord: ScoreRecord, viewCount: Int) {
        self.scoreRecord = scoreRecord
        self.viewCount = viewCount
    }

    func update() {
        let record = self.scoreRecord.getScoreRecord()
        displayScores(record: record, viewCount: self.viewCount)
    }

    private func displayScores(record: [Int], viewCount: Int) {
        print("List of \(min(viewCount, record.count)) entries:")
        for i in 0..<min(viewCount, record.count) {
            print(record[i])
        }

    }
}
```

AS-IS에 있던 DataSheetView이다. Observer 프로토콜을 따르도록 지정해준 것 외에는 변동 사항이 없다.

```swift
class MinMaxView: Observer {
    private var scoreRecord: ScoreRecord

    init(scoreRecord: ScoreRecord) {
        self.scoreRecord = scoreRecord
    }

    func update() {
        let record = self.scoreRecord.getScoreRecord()
        displayScores(record: record)
    }

    private func displayScores(record: [Int]) {
        print("Min: \(record.min() ?? 0) Max: \(record.max() ?? 0)")
    }

}
```

새로 추가할 View 클래스이다. 점수 리스트 변경을 통보 받으면 점수 리스트의 최고점과 최저점을 출력해주는 기능을 한다.

### 클라이언트

```swift
var scoreRecord = ScoreRecord()
let dataSheetView = DataSheetView(scoreRecord: scoreRecord, viewCount: 3)
let minMaxView = MinMaxView(scoreRecord: scoreRecord)

scoreRecord.attach(observer: dataSheetView)
scoreRecord.attach(observer: minMaxView)

for i in 0...5 {
    let score = i * 10
    print("Adding \(score)")
    scoreRecord.addScore(score)
}
```

기존에 존재하던 DataSheetView와 새로 추가된 View 클래스인 MinMaxView를 ScoreRecord 클래스의 객체에 Attach 해주면 ScoreRecord가 가지고 있는 점수 리스트에 변동이 생길때마다 통보받게 된다.
실행을 해보면 데이터가 추가될 때 마다 두 View 클래스에서 변경 내용에 맞는 출력이 나옴을 확인할 수 있다.