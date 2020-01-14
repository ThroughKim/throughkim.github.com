---
layout: post
title: 'Arrange Multi-line UILabel in UIStackView with SnapKit'
author: through.kim
date: 2020-01-13 16:27
tags: [study, swift, ios]
image: '/files/covers/ios_app.jpg'
comments: true
---

> Multi Line UILabel을 UIStackView 안에서 정렬하기 위한 방법 정리

## 방법
UILabel을 UIView안에 넣고, UIView를 UIStackView에 넣으면 된다.

## 코드

```swift
class ExampleView: UIView {

    private lazy var titleLabel = UILabel()
    
    private lazy var contentLabel: UILabel = {
        let label = UILabel()
        label.numerOfLines = 0
        
        return label
    }()
    
    private lazy var contentView: UIView = {
        let view = UIView()
        view.addSubview(self.contentLabel)
        
        return view
    }()
    
    private lazy var spacer: UIView = {
        let view = UIView()
        view.setContentHuggingPriority(.defaultLow, for: .horizontal)
        
        return view
    }()
    
    private lazy var stackView: UIStackView = {
        let view = UIStackView(arrangedSubviews: [self.titleLabel, self.contentView, self.spacer])
        view.axis = .horizontal
        view.alignment = .firstBaseline
        view.distribution = .fill
        
        return view
    }()
    
    init(frame: CGRect) {
        super.init(frame: frame)
        
        self.addSubview(stackView)
        stackView.snp.makeConstraints{ make in
            make.size.equalToSuperview()
        }
        contentLabel.snp.makeConstraints { make in
            make.size.equalToSuperview()
        }
        
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

}
```
