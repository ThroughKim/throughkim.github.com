---
layout: post
title: '스위프트로 블록체인 구현하기'
author: through.kim
date: 2018-01-15 13:18
tags: [swift, blockchain]
image: '/files/covers/blockchain.png'
---

> 요즘 암호 화폐가 이슈이다. 암호 화폐 기술의 근간은 블록체인이며, 이 블록체인은 4차 산업혁명을 이끌어 나갈 기술이라고 평가된다.
 그러나 많은 사람이 블록체인이 무엇인지, 어떻게 구동되는지 알지 못한다.
 어떤 개념을 이해하는데 가장 확실한 방법은 직접 만들어보고 체험해보는 것이라고 생각한다.
 이 포스팅을 통해 Swift를 사용하여 블록체인을 직접 구축하며 블록체인의 원리를 이해해보고자 한다.


블록체인은 요즘 이슈가 되는 암호화폐인 비트코인의 근간이 되는 기술이다.
블록체인의 핵심 가치는 중앙에 의해 제어되지 않는 분산된 원장(Ledger)을 제공하는 것이다.
이 포스팅에서는 iOS/MacOS를 바탕으로 Swift 언어를 사용해 기본적인 블록체인을 구축해본다.

 __알림: 이 포스팅에서는 노드(node)/피어(peer), 검증(validation), 보상(reward) 등에 대해 다루지 않으며 기초적인 블록체인의 구조만을 구현한다.__

가장 빠르고 쉽게 시작하는 방법은 Swift 플레이그라운드를 사용하는 방법이다.
MacOS 플레이그라운드에는 블록체인을 구현하는데 필요한 SHA1 해시를 생성하는데 필요한 기능들이 있다.
따라서, __반드시__ MacOS 플레이그라운드로 프로젝트를 생성해야한다.

![MacOS Playground Only!](/files/blockchain_images/blockchain_project.png)


# Block 객체 구현하기

첫 번쨰 단계는 블록체인에서 단일 블록을 나타내는 Block 객체를 구현하는것이다. 구현된 내용은 다음과 같다.

```swift
class Block {

    var index :Int = 0
    var dateCreated :String
    var previousHash :String!
    var hash :String!
    var nonce :Int
    var data :String

    var key :String {
        get {
            return String(self.index)
                + self.dateCreated
                + self.previousHash
                + self.data
                + String(self.nonce)
        }
    }

    init(data :String) {
        self.dateCreated = Date().toString()
        self.nonce = 0
        self.data = data
    }

}

// Date -> String 변환시켜주기 위한 Extension
extension Date {
    func toString() -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        return formatter.string(from: self)
    }
}
```

Block 객체의 속성들은 다음과 같다.

 - index: 블록체인에서 블록의 위치. 인덱스가 0일 경우 블록체인의 첫 번째 블록을 의미하며, 보통 제네시스 블록이라고 불리운다. 인덱스 1일경우 블록체인 내에서 두 번째 블록임을 의미한다.
 - dateCreated: 블록이 생성된 날짜
 - previousHash: 이전 블록의 해시값
 - hash: 현재 블록의 hash값
 - nonce: 자동으로 1씩 증가하는 숫자로, 해시값을 얻을때까지 수행했던 계산의 횟수를 의미한다.
 - data: 가치있는 자산. 돈, 의료 정보, 부동산 정보등이 들어갈 수 있다.
 - key: 해시함수로 전달되는 계산된 속성값.


# 블록체인 객체 구현하기

블록체인 클래스의 구현내용은 다음과 같다. 동일 플레이그라운드 파일의 하단에 다음과 같이 붙여서 작성한다.

```swift
class Blockchain {

    private (set) var blocks :[Block] = [Block]()

    init(_ genesisBlock :Block) {

        addBlock(genesisBlock)
    }

    func addBlock(_ block :Block) {

        if self.blocks.isEmpty {
            // 제네시스 블록 추가하기
            // 첫 블록앞에는 블럭이 없다.
            block.previousHash = "0"
            block.hash = generateHash(for: block)
        } else {
            let previousBlock = getPreviousBlock()
            block.previousHash = previousBlock.hash
            block.index = self.blocks.count
            block.hash = generateHash(for: block)
        }

        self.blocks.append(block)
        displayBlock(block)
    }

    private func getPreviousBlock() -> Block {
        return self.blocks[self.blocks.count - 1]
    }

    private func displayBlock(_ block :Block) {
        print("------ Block \(block.index) ---------")
        print("Date Created : \(block.dateCreated) ")
        print("Data : \(block.data) ")
        print("Nonce : \(block.nonce) ")
        print("Previous Hash : \(block.previousHash!) ")
        print("Hash : \(block.hash!) ")

    }

    private func generateHash(for block: Block) -> String {

        var hash = block.key.sha1Hash()

        // 작업증명 세팅
        // "00" 으로 시작하는 것이 좋다. "0000" 으로 시작하면 너무 오래걸려서 플레이그라운드 크래시가 발생한다.
        while(!hash.hasPrefix("00")) {
            block.nonce += 1
            hash = block.key.sha1Hash()
        }

        return hash
    }

}

// sha1Hash 구현
extension String {

    func sha1Hash() -> String {

        let task = Process()
        task.launchPath = "/usr/bin/shasum"
        task.arguments = []

        let inputPipe = Pipe()

        inputPipe.fileHandleForWriting.write(self.data(using: String.Encoding.utf8)!)

        inputPipe.fileHandleForWriting.closeFile()

        let outputPipe = Pipe()
        task.standardOutput = outputPipe
        task.standardInput = inputPipe
        task.launch()

        let data = outputPipe.fileHandleForReading.readDataToEndOfFile()
        let hash = String(data: data, encoding: String.Encoding.utf8)!
        return hash.replacingOccurrences(of: "  -\n", with: "")
    }
}
```
블록체인 객체는 자신을 초기화하기 위해서 블록의 인스턴스가 필요한데, 이 블록을 블록체인의 첫 번째 블록이며 제네시스 블록이라고 한다.

__addBlock()__ 함수는 블록체인에 블록을 추가한다. 다음블록은 이전 블록의 해시에 기반하는데, 그 해시값은 블록의 key 속성을 이용해 생성된다.

__generateHash()__ 함수는 블록에 할당될 유니크한 해시값을 생성하는 역할을 맡는다. 랜덤 해시를 사용하는 대신에 우리는 __"00"__ 으로 시작하는 특별한 해시를 찾아낼 것이다.
이러한 컨셉은 _"작업증명 시스템"_ 이라고 알려져있다. 실제 작업증명 시스템은 더 풀기에 더 복잡하기 때문에, 그 해시값을 찾아낸 사용자에게 보상이 지급된다.(예를 들어 비트코인)

예제 코드에서는 addBlock() 함수에서 GenerateHash()를 호출하여, 설정된 Hash값을 발견한 경우 블록체인에 해당 블록을 추가하고, 블록의 정보를 디스플레이하도록 만들어졌다.


# 블록체인 구동

이제 블록체인을 실제로 동작시켜보자. 다음의 코드를 플레이그라운드 최하단에 작성하면 블록체인이 구동된다.

```swift
let genesisBlock = Block(data: "genesis")
let blockChain = Blockchain(genesisBlock)

for index in 1...5 {
    let block = Block(data: "\(index)")
    blockChain.addBlock(block)
}
```

코드의 내용을 다음과 같다.
genesisBlock 이라는 임의의 블록을 초기화한다. 여기서 데이터는 어떤 값이 들어가도 무관하다.
이어서 블록체인에 제네시스블록을 넣어 초기화해준다. 마지막으로 5개의 블록체인이 추가될 때 까지 블록체인에 블록을 추가하도록 한다.
실행하면 다음과 같은 정보가 프린트될 것이다.

```swift
------ Block 0 ---------
Date Created : 2018-01-15 15:30:23
Data : genesis
Nonce : 814
Previous Hash : 0
Hash : 00d3fff768b2b34493e1d8dd9492ba5b72e7e4fc
------ Block 1 ---------
Date Created : 2018-01-15 15:30:50
Data : 1
Nonce : 36
Previous Hash : 00d3fff768b2b34493e1d8dd9492ba5b72e7e4fc
Hash : 00439626e99c4910f5f6a0b0235b3393616c8984
------ Block 2 ---------
Date Created : 2018-01-15 15:30:51
Data : 2
Nonce : 812
Previous Hash : 00439626e99c4910f5f6a0b0235b3393616c8984
Hash : 00a402d2dee753952ca461c79a3fc5bbbfe7838a
...
```

<iframe width="750" height="530" src="https://www.youtube.com/embed/0GNurVtlaKk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

생성일과 데이터, Nonce값 그리고 이전 블록의 해시값과 현재 블록의 해시값이 출력됨을 볼 수 있다.
이때까지 작성한 전체 코드는 다음과 같다. 그대로 플레이그라운드에 복사하여 붙여넣기하면 동일한 내용이 실행될 것이다.

```swift
import Foundation

class Block {

    var index :Int = 0
    var dateCreated :String
    var previousHash :String!
    var hash :String!
    var nonce :Int
    var data :String

    var key :String {
        get {
            return String(self.index)
                + self.dateCreated
                + self.previousHash
                + self.data
                + String(self.nonce)
        }
    }

    init(data :String) {
        self.dateCreated = Date().toString()
        self.nonce = 0
        self.data = data
    }

}

// Date -> String 변환시켜주기 위한 Extension
extension Date {
    func toString() -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        return formatter.string(from: self)
    }
}


class Blockchain {

    private (set) var blocks :[Block] = [Block]()

    init(_ genesisBlock :Block) {

        addBlock(genesisBlock)
    }

    func addBlock(_ block :Block) {

        if self.blocks.isEmpty {
            // 제네시스 블록 추가하기
            // 첫 블록앞에는 블럭이 없다.
            block.previousHash = "0"
            block.hash = generateHash(for: block)
        } else {
            let previousBlock = getPreviousBlock()
            block.previousHash = previousBlock.hash
            block.index = self.blocks.count
            block.hash = generateHash(for: block)
        }

        self.blocks.append(block)
        displayBlock(block)
    }

    private func getPreviousBlock() -> Block {
        return self.blocks[self.blocks.count - 1]
    }

    private func displayBlock(_ block :Block) {
        print("------ Block \(block.index) ---------")
        print("Date Created : \(block.dateCreated) ")
        print("Data : \(block.data) ")
        print("Nonce : \(block.nonce) ")
        print("Previous Hash : \(block.previousHash!) ")
        print("Hash : \(block.hash!) ")

    }

    private func generateHash(for block: Block) -> String {

        var hash = block.key.sha1Hash()

        // 작업증명 세팅
        // "00" 으로 시작하는 것이 좋다. "0000" 으로 시작하면 너무 오래걸려서 플레이그라운드 크래시가 발생한다.
        while(!hash.hasPrefix("00")) {
            block.nonce += 1
            hash = block.key.sha1Hash()
//            print(hash)
        }

        return hash
    }

}

// sha1Hash 구현
extension String {

    func sha1Hash() -> String {

        let task = Process()
        task.launchPath = "/usr/bin/shasum"
        task.arguments = []

        let inputPipe = Pipe()

        inputPipe.fileHandleForWriting.write(self.data(using: String.Encoding.utf8)!)

        inputPipe.fileHandleForWriting.closeFile()

        let outputPipe = Pipe()
        task.standardOutput = outputPipe
        task.standardInput = inputPipe
        task.launch()

        let data = outputPipe.fileHandleForReading.readDataToEndOfFile()
        let hash = String(data: data, encoding: String.Encoding.utf8)!
        return hash.replacingOccurrences(of: "  -\n", with: "")
    }
}

let genesisBlock = Block(data: "genesis")
let blockChain = Blockchain(genesisBlock)

for index in 1...5 {
    let block = Block(data: "\(index)")
    blockChain.addBlock(block)
}

```

이 글의 원작자는 iOS 블록체인 프로그래밍의 강의를 운영하고 있다. 혹시 관심있다면 다음 [링크](https://gumroad.com/l/sFOao/blockchainfree)를 통해 접속할 수 있다.
원작자의 블로그는 다음의 [링크](http://www.azamsharp.com/courses/)를 통해 접속할 수 있다.

원본 : [Blockchain Programming in iOS - Mohammad Azam](https://hackernoon.com/blockchain-programming-in-ios-ffaff9b328cc)