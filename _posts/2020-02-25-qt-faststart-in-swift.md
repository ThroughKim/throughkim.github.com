---
layout: post
title: 'qtfaststart in Swift'
author: through.kim
date: 2020-02-25 10:15
tags: [study, swift, ios]
image: '/files/covers/ios_app.jpg'
comments: true
---

> mp4 파일중 스트리밍이 지원되지 않는 파일이 존재하였고, 이를 최적화 시켜주는 C라이브러리를 Swift로 컨버젼했음

# Mp4 atoms
mp4 파일 중 스트리밍이 되지 않는 파일들이 존재하는데, 이는 mp4 파일을 구성하는 요소인 atom 배치에 따른 것이었다.
데이터를 담고있는 mdat atom과 데이터에 대한 정보를 담고 있는 moov atom의 순서에 따라 스트리밍 가능 / 불가능 여부가 결정되는데,
스트리밍이 가능한 mp4는 moov atom이 mdat atom보다 앞에 배치되어 있었고, 스트리밍이 불가한 mp4는 moov atom이 파일의 가장 뒷부분에 붙어있었다.
이를 수정하여 mp4를 스트리밍해줄 수 있는 방법을 찾아보았다.

## AVAssetExportSession
애플에서 제공하는 기능으로, 세션에 shouldOptimizeForNetworkUse 값을 true로 주면 moov atom의 위치를 옮겨서 스트리밍이 가능해지도록 만들 수 있다.
하지만 input과 output이 모두 파일이며, 파일을 write하는데 시간이 걸리기 때문에 요구하는 성능을 만족시킬 수 없었다.

## qtfaststart.c
C 라이브러리로 byte 단위의 연산을 통해 moov atom을 찾고 이동시켜주는 기능을 제공해준다. 나는 이 라이브러리를 swift로 컨버전하여 사용했다.  

[qtfaststart.c](https://github.com/FFmpeg/FFmpeg/blob/master/tools/qt-faststart.c)

# 코드

## QTFastStart.swift
```swift
import Foundation

/// Optimize mp4 data for audio streaming.
/// return optimized data if success.
/// return original data if failure.

class QTFastStart {

    private var FREE_ATOM: Int { fourCcToInt("free") }
    private var JUNK_ATOM: Int { fourCcToInt("junk") }
    private var MDAT_ATOM: Int { fourCcToInt("mdat") }
    private var MOOV_ATOM: Int { fourCcToInt("moov") }
    private var PNOT_ATOM: Int { fourCcToInt("pnot") }
    private var SKIP_ATOM: Int { fourCcToInt("skip") }
    private var WIDE_ATOM: Int { fourCcToInt("wide") }
    private var PICT_ATOM: Int { fourCcToInt("pict") }
    private var FTYP_ATOM: Int { fourCcToInt("ftyp") }
    private var UUID_ATOM: Int { fourCcToInt("uuid") }
    
    private var CMOV_ATOM: Int { fourCcToInt("cmov") }
    private var STCO_ATOM: Int { fourCcToInt("stco") }
    private var CO64_ATOM: Int { fourCcToInt("c064") }
    
    private let ATOM_PREAMBLE_SIZE: Int = 8
    
    private func fourCcToInt(_ fourCc: String) -> Int {
        let data = fourCc.data(using: .ascii)!
        return Int(bigEndian: Int(data: data))
    }
    
    func readAndFill(_ data: inout ByteBuffer, _ buffer: inout ByteBuffer) -> Bool {
        buffer.clear()
        do {
            let slicedData: Data = try data.readBytes(buffer.length)
            buffer.writeBytes(slicedData)
            buffer.rewind()
            return true
        } catch {
            return false
        }
        
    }
    
    func process(_ data: Data) -> Data {
        var dataBytes = ByteBuffer(data: data)
        var atomBytes = ByteBuffer(size: ATOM_PREAMBLE_SIZE)
        var atomType: Int = 0
        var atomSize: Int = 0
        var startOffset: Int = 0
        let lastOffset: Int
        var ftypAtom: ByteBuffer? = nil
        var moovAtom: ByteBuffer
        let moovAtomSize: Int
        
        // mp4의 atom을 조회하여 moov atom이 최하단에 있는지 체크
        do {
            while readAndFill(&dataBytes, &atomBytes) {
                atomSize = Int(try atomBytes.getInteger() as UInt32)
                atomType = Int(try atomBytes.getInteger() as UInt32)
                
                if atomType == FTYP_ATOM {
                    let ftypAtomSize = atomSize
                    ftypAtom = ByteBuffer(size: ftypAtomSize)
                    atomBytes.rewind()
                    ftypAtom!.writeBytes(try atomBytes.readBytes(atomBytes.length))
                    
                    let ftypData = try dataBytes.readBytes(ftypAtom!.bytesAvailable)
                    ftypAtom!.writeBytes(ftypData)
                    ftypAtom!.rewind()
                    
                    startOffset = dataBytes.position

                } else {
                    dataBytes.position = dataBytes.position + atomSize - ATOM_PREAMBLE_SIZE
                }
                
                if atomType != FREE_ATOM
                    && atomType != JUNK_ATOM
                    && atomType != MDAT_ATOM
                    && atomType != MOOV_ATOM
                    && atomType != PNOT_ATOM
                    && atomType != SKIP_ATOM
                    && atomType != WIDE_ATOM
                    && atomType != PICT_ATOM
                    && atomType != UUID_ATOM
                    && atomType != FTYP_ATOM {
                    print("Encountered non-QT top-level atom")
                    break
                }
                
                if (atomSize < 8) {
                    break
                }
                
            }
            
            if atomType != MOOV_ATOM {
                print("last atom in file was not a moov atom")
                return data
            }
            
            // moov atom이 최하단에 있음을 확인 후 moovAtom 전체를 로드함
            moovAtomSize = atomSize
            lastOffset = dataBytes.length - moovAtomSize // moov 뒤에 더이상 데이터가 없다는 가정 (qt-faststart.c)
            dataBytes.position = lastOffset
            moovAtom = ByteBuffer(size: moovAtomSize)
            
            if !readAndFill(&dataBytes, &moovAtom) {
                print("moov parse error")
                return data
            }
            
            if try Int(moovAtom.getInteger(12) as UInt32) == CMOV_ATOM {
                print("Compressed moov atom not supported")
                return data
            }
            
            // stco 또는 co64 atom을 찾기 위해 moov atom을 검사
            while(moovAtom.bytesAvailable >= 8) {
                let atomHead = moovAtom.position
                atomType = Int(try moovAtom.getInteger(atomHead + 4) as UInt32)
                
                if !(atomType == STCO_ATOM || atomType == CO64_ATOM) {
                    moovAtom.position = moovAtom.position + 1
                    continue
                }
                
                atomSize = Int(try moovAtom.getInteger(atomHead) as UInt32)
                if atomSize > moovAtom.bytesAvailable {
                    print("bad atom size")
                    return data
                }
                
                moovAtom.position = atomHead + 12 // skip size (4 bytes), type (4 bytes), version (1 byte) and flags (3 bytes)
                if moovAtom.bytesAvailable < 4 {
                    print("malformed atom")
                    return data
                }
                
                let offsetCount = Int(try moovAtom.getInteger() as UInt32)
                if atomType == STCO_ATOM {
                    print("patching stco atom")
                    if moovAtom.bytesAvailable < offsetCount * 4 {
                        print("bad atom size/element count")
                        return data
                    }
                    
                    for _ in 0..<offsetCount {
                        let currentOffset = Int( try moovAtom.getInteger(moovAtom.position) as UInt32)
                        
                        let newOffset = currentOffset + moovAtomSize
                        
                        if currentOffset < 0 && newOffset >= 0 {
                            print("Unsupported file exception")
                            return data
                        }
                        moovAtom.put(UInt32(newOffset).bigEndian)

                    }
                } else if atomType == CO64_ATOM {
                    print("patching co64 atom")
                    if moovAtom.bytesAvailable < offsetCount * 8 {
                        print("bad atom size/element count")
                        return data
                    }
                    for _ in 0..<offsetCount {
                        let currentOffset = try moovAtom.getInteger(moovAtom.position) as Int
                        moovAtom.put(currentOffset + moovAtomSize)
                    }
                    
                 }
            }
            
            dataBytes.position = startOffset // ftyp atom 뒷부분으로 이동
            
            var outData = Data()
            
            guard let ftypAtom = ftypAtom else { return data }
            print("writing ftyp atom")
            outData.append(ftypAtom.data)
            
            print("writing new moov atom")
            outData.append(moovAtom.data)
            
            print("write rest of data")
            let restOfData = try dataBytes.readBytes(dataBytes.length - startOffset)
            outData.append(restOfData)
            
            return outData
            
        } catch {
            print(error)
            return data
        }
        
    }

}
```

## ByteBuffer
Data에서 Byte단위의 연산을 편하게 하기 위해 ByteBuffer도 구현했다.

```swift
import Foundation

class ByteBuffer {
    
    enum Error: Swift.Error {
        case eof
        case parse
    }
    
    private(set) var data = Data()

    open var position: Int = 0

    open var bytesAvailable: Int {
        data.count - position
    }
    
    open var length: Int {
        get {
            data.count
        }
        set {
            switch true {
            case (data.count < newValue):
                data.append(Data(count: newValue - data.count))
            case (newValue < data.count):
                data = data.subdata(in: 0..<newValue)
            default:
                break
            }
        }
    }

    open subscript(i: Int) -> UInt8 {
        get {
            data[i]
        }
        set {
            data[i] = newValue
        }
    }
    
    init(data: Data) {
        self.data = data
        position = 0
    }

    init(size: Int) {
        data = Data(repeating: 0x00, count: size)
        position = 0
    }
    
    @discardableResult
    open func clear() -> Self {
        position = 0
        return self
    }
    
    open func rewind() {
        position = 0
    }
    
    open func readBytes(_ length: Int) throws -> Data {
        guard length <= bytesAvailable else {
            throw ByteBuffer.Error.eof
        }
        position += length
        return data.subdata(in: position - length..<position)
    }
    
    @discardableResult
    open func writeBytes(_ value: Data) -> Self {
        if position == data.count {
            data.append(value)
            position = data.count
            return self
        }
        let length: Int = min(data.count, value.count)
        data[position..<position + length] = value[0..<length]
        if length == data.count {
            data.append(value[length..<value.count])
        }
        position += value.count
        return self
    }
    
    @discardableResult
    open func put<T: FixedWidthInteger>(_ value: T) -> ByteBuffer {
        writeBytes(value.data)
    }

    @discardableResult
    open func put(_ value: Float) -> ByteBuffer {
        writeBytes(Data(value.data.reversed()))
    }

    @discardableResult
    open func put(_ value: Double) -> ByteBuffer {
        writeBytes(Data(value.data.reversed()))
    }
    
    open func getInteger<T: FixedWidthInteger>() throws -> T {
        let sizeOfInteger = MemoryLayout<T>.size
        guard sizeOfInteger <= bytesAvailable else {
            throw ByteBuffer.Error.eof
        }
        position += sizeOfInteger
        return T(data: data[position - sizeOfInteger..<position]).bigEndian
    }
    
    open func getInteger<T: FixedWidthInteger>(_ index: Int) throws -> T {
        let sizeOfInteger = MemoryLayout<T>.size
        guard sizeOfInteger + index <= length else {
            throw ByteBuffer.Error.eof
        }
        return T(data: data[index..<index+sizeOfInteger]).bigEndian
    }

    open func getFloat() throws -> Float {
        let sizeOfFloat = MemoryLayout<UInt32>.size
        guard sizeOfFloat <= bytesAvailable else {
            throw ByteBuffer.Error.eof
        }
        position += sizeOfFloat
        return Float(data: Data(data.subdata(in: position - sizeOfFloat..<position).reversed()))
    }

    open func getDouble() throws -> Double {
        let sizeOfDouble = MemoryLayout<UInt64>.size
        guard sizeOfDouble <= bytesAvailable else {
            throw ByteBuffer.Error.eof
        }
        position += sizeOfDouble
        return Double(data: Data(data.subdata(in: position - sizeOfDouble..<position).reversed()))
    }
    
}

extension ExpressibleByIntegerLiteral {
    var data: Data {
        var value: Self = self
        return Data(bytes: &value, count: MemoryLayout<Self>.size)
    }

    init(data: Data) {
        let diff: Int = MemoryLayout<Self>.size - data.count
        if 0 < diff {
            var buffer = Data(repeating: 0, count: diff)
            buffer.append(data)
            self = buffer.withUnsafeBytes { $0.baseAddress!.assumingMemoryBound(to: Self.self).pointee }
            return
        }
        self = data.withUnsafeBytes { $0.baseAddress!.assumingMemoryBound(to: Self.self).pointee }
    }

    init(data: Slice<Data>) {
        self.init(data: Data(data))
    }
}

```

## Usage
process가 에러 없이 동작했다면 moov atom이 수정된 data를 리턴하고, 그렇지 않을 경우 원본 데이터를 리턴함
에러 핸들링 수정할 예정

```swift
let data = 문제가 있는 mp4의 데이터
let processedData = QTFastStart().process(data)
```
