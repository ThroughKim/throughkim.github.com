---
layout: post
title: 'IGListKit 셀 선택시 배경 변경하기'
author: through.kim
date: 2017-02-07 16:06
tags: [ios, app, study, iglistkit]
image: '/files/covers/ios_app.jpg'
comments: true
---

> IGListKit을 이용해 SideMenu를 구성하는 과정 중, 메뉴에서 해당하는 셀을 선택하면 아래의 이미지와 같이 활성화 된 셀의 배경색이 변경되게 구현하고자 한다.

![사이드 메뉴 구현](/files/iglistkit/sideMenuImg.png)

처음에 기본적 형태만 구현한 상태에서는 탭을 해도 색상이 변하지 않고 해당 셀에 지정된 action만 행할 뿐이었다. 구현하기 원하는 기능은 탭을 하면 선택된 셀의 배경색이 변하고, 선택되지 않은 셀의 배경들은 투명 혹은 배경과 같은 색깔을 띄는 것이었다. IGListKit의 예제를 보고 응용해보면, 한 섹션의 색은 잘 변하지만, 다른 섹션의 셀 색깔은 개별 섹션컨트롤러에서 변경하는 것이 불가능했다. 결국 Delegate를 만들고, 메뉴의 객체값에 isSelected 값을 넣어 구현할 수 있었다.  
  
구글링을 하면서 비슷한 [질문글](https://github.com/Instagram/IGListKit/issues/184)을 찾아볼 수 있었다. 이 글에 따르면 두 가지 방법이 존재한다.

 - At the VC level, iterate all of your section controllers. Check for your selectable SC type, then check if the SC is marked as selected (again using your custom property)  
 
 - Create a delegate on the section controller that forwards select/deselect events up to the VC  
    - IMO this is more tedious

이 내용에서 힌트를 얻어 내가 구현한 방법은 아래의 순서와 같다.

 1. 메뉴 섹션컨트롤러의 didSelected 에서 트리거
 2. 섹션컨트롤러 delegate로 자신(선택된) 섹션컨트롤러 객체 VC로 전달 
 3. VC의 어댑터에서 갖고있는 섹션컨트롤러 리스트 불러옴 
 4. 선택된 섹션컨트롤러로 전달되는 메뉴 객체에 isSelected = true 
 5. 선택되지 않은 섹션컨트롤러로 전달되는 메뉴 객체에 isSelected = false
 6. 어댑터에 새로운 데이터(isSelected값이 바뀐)를 넘겨 재로딩

아직 스위프트와 IGListKit에 대한 이해가 부족해서, 이 방법이 생각해낸 최선의 방법이었다.  
(후에 리팩토링 한다면 수정할 것)

IGListKit으로 메뉴를 구성해놓은 계층은 다음과 같다.

<table>
<tr>
    <td rowspan="4">MenuViewController</td>
    <td>>menuSectionController</td>
    <td>>MenuCell</td>
</tr>
<tr>
    <td>>menuSectionController</td>
    <td>>MenuCell</td>
</tr>
<tr>
    <td>>menuSectionController</td>
    <td>>MenuCell</td>
</tr>
<tr>
    <td>>menuSectionController</td>
    <td>>MenuCell</td>
</tr>
</table>

전체메일함, 받은메일함, 보낸메일함, 메일보관함 네개의 메뉴 객체를 각 섹션컨트롤러로 전달해주고, 섹션컨트롤러는 단일 셀을 갖는 구조로 되어있다.

우선 섹션컨트롤러로부터 Delegate를 만들어 ViewController로 전달해주는 부분을 만들어야 한다.

```swift

import UIKit
import IGListKit

protocol MenuSectionDelegate: class {
    func menuSelected(selectedMenuSection: MenuSectionController)
}

class MenuSectionController: IGListSectionController {
    var menu: Menu!
    weak var delegate: MenuSectionDelegate?
    
    override init() {
        super.init()
    }
}

extension MenuSectionController: IGListSectionType {
    func numberOfItems() -> Int {
        return 1
    }
    
    func sizeForItem(at index: Int) -> CGSize {
        let width = collectionContext!.containerSize.width
        return CGSize(width: width, height: 40)
    }
    
    func cellForItem(at index: Int) -> UICollectionViewCell {
        let cell = collectionContext?.dequeueReusableCell(of: MenuCollectionViewCell.self, for: self, at: index) as! MenuCollectionViewCell
        cell.iconImageView.image = menu.icon
        cell.menuLabel.text = menu.name
        if menu.isSelected {
            cell.backgroundColor = .blue
        } else if !menu.isSelected {
            cell.backgroundColor = .clear
        }
        return cell
    }
    
    func didUpdate(to object: Any) {
        menu = object as? Menu
    }
    
    func didSelectItem(at index: Int) {
        viewController?.performSegue(withIdentifier: "showMailList", sender: menu.type)
        self.delegate?.menuSelected(selectedMenuSection: self)
    }
}

```

클래스 상단에 Delegate 프로토콜과 메소드를 작성해주고, didSelectItem에서 delgate 메소드를 호출해준다. 해당 셀이 선택되면 셀이 포함된 섹션컨트롤러를 넘겨주게 된다.

이제 뷰컨트롤러를 수정해 Delegate메소드를 수신할 수 있도록 해야한다.

```swift

extension MenuViewController: IGListAdapterDataSource {
    func objects(for listAdapter: IGListAdapter) -> [IGListDiffable] {
        var items: [IGListDiffable] = profileLoader.user
            items += menuTypeLoader.menuTypeList as [IGListDiffable]
            items += menuLoader.menuList as [IGListDiffable]
        
        return items
    }
    
    func listAdapter(_ listAdapter: IGListAdapter, sectionControllerFor object: Any) -> IGListSectionController {
        switch object{
        case is User:       return ProfileSectionController()
        case is [MenuType]: return MenuTypeListSectionController()
        default:
            let menuSC = MenuSectionController()
            menuSC.delegate = self
            return menuSC
        }
    }
    
    func emptyView(for listAdapter: IGListAdapter) -> UIView? {
        return nil
    }
}

```

우선 IGListAdapterDataSource 부분에서 Delegate를 설정해주어야 한다. 메뉴객체를 다루는 MenuSectionController만 delegate해주어야 한다. 이어서 하단에 MenuSectionDelegate를 충족시켜주는 작업을 진행해주어야 한다.


```swift

extension MenuViewController: MenuSectionDelegate {
    func menuSelected(selectedMenuSection: MenuSectionController) {
    let selecteMenuIndex = adapter.section(for: selectedMenuSection) - 2    // minus number of sections before menuSection
        for menuIndex in 0..<menuList.count {
            if menuIndex != selecteMenuIndex {
                menuList[menuIndex].isSelected = false
            } else {
                menuList[menuIndex].isSelected = true
            }
        }
        adapter.reloadData(completion: nil)
    }
}
}

```

받아온 섹션컨트롤러의 값을 이용해 전달해줄 메뉴 객체의 isSelected값을 적절히 변경시켜주는 부분이다. 
메뉴 객체를 가져와서 해당 객체의 인덱스번호와 뷰컨트롤러가 갖고 있는 메뉴 객체의 인덱스번호를 매칭시켜 준 뒤 해당 객체의 isSelected값을 변경시켜준다.

  
(아마 선택된 셀과 상호작용 하는 기능은 업데이트 될 듯 하다. [https://github.com/Instagram/IGListKit/pull/449](https://github.com/Instagram/IGListKit/pull/449))

이렇게 수정을 해주면 이제 탭(활성화)된 셀은 배경이 변하고, 다른 셀들의 배경은 투명하게 변할 것이다.


