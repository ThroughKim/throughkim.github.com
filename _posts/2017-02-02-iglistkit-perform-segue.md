---
layout: post
title: 'IGListKit segue 시행하기'
author: through.kim
date: 2017-02-02 15:46
tags: [ios, app, study]
image: '/files/covers/ios_app.jpg'
---

>사이드메뉴를 IGListKit으로 구현하였는데, 기존의 스토리보드 Segue방식으로 뷰컨트롤러를 푸시해줄 수 없었다.

기존에 구현했던 방식 대로 index값을 넘겨주고 값을 얻어오려고 하니 index가 항상 0으로 표시되었고, ViewController가 가지고 있는 Menu 리스트의 값 역시 0으로 표시되어 List out of index 에러 가 발생했다. 

IGListKit에서는 인덱스의 숫자 값이 아닌 객체 자체를 주고받는 방식이었기 때문이다. 다음과 같이 해결할 수 있었다.

```swift
## Section Controller

extension MenuSectionController: IGListSectionType {    
    
    ...
    
    func didSelectItem(at index: Int) {
        viewController?.performSegue(withIdentifier: "showViewController", sender: menu.type)
    }
}
```

선택된 메뉴의 타입값(enum)을 sender에 담아 보내고, ViewController에서 해당 segue identifier에 따른 동작을 정의해주는 방식으로 구현했다.

```swift
## View Controller

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        super.prepare(for: segue, sender: sender)

        ...
        
        guard let menuType = sender as? String else {
            fatalError("식별불가능한 값을 받았습니다. \(sender)")
        }
        guard let menuName = App.MenuType(rawValue: menuType)?.getMenuName() else {
            fatalError("정의되지 않은 메뉴 타입입니다.\(menuType)")
        }
        guard let segueDestination = segue.destination as? UINavigationController else {
            fatalError("예상치 못한 목표 뷰 컨트롤러: \(segue.destination)")
        }
        guard let targetViewController = segueDestination.topViewController as? MailListViewController else {
            fatalError("예상치 못한 목표 뷰 컨트롤러: \(segueDestination.topViewController)")
        }
        
        targetViewController.title = menuName
        targetViewController.type = menuType
    }

```

