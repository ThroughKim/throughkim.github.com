---
layout: post
title: 'IGListKit 튜토리얼'
author: through.kim
date: 2017-01-12 11:05
tags: [ios, app, study]
image: '/files/covers/ios_app.jpg'
---
출처: [IGListKit Tutorial: Better UICollectionViews](https://www.raywenderlich.com/147162/iglistkit-tutorial-better-uicollectionviews)

> iOS의 UICollectionView를 개선한 IGListKit의 튜토리얼을 번역하며 따라해본다.

* TOC  
{:toc}

대부분의 어플리케이션의 시작은 비슷비슷하다. 몇개의 스크린, 몇개의 버튼 그리고 한두개의 리스트. 하지만 시간이 지나고 어플리케이션의 규모가 커지면 기능들이 넘쳐나게된다(feature-creep). 당신의 깔끔했던 데이터 소스들은 데드라인과 PM의 압박에 시달리며 먼지가 되어간다. 잠시후, 당신은 거대한 뷰 컨트롤러(MVC, Massive View Controller)를 유지보수하기 위해 야근을 하게 된다. IGListKit을 적용하면 이같은 압박에서 해방될 수 있을것이다.  

[IGListKit](https://github.com/Instagram/IGListKit)은 feature-creep하며 거대한 ViewController를 대체하기 위해 만들어졌다. IGListKit을 이용해 리스트를 만들면 어플리케이션의 컴포넌트를 각각 분리할 수 있고, 업데이트 속도를 개선할 수 있으며, 어떤 타입의 데이터라도 지원할 수 있게 된다.  

이 튜토리얼에서 UICollectionViews를 IGListKit을 활용해 리팩토링할 것이다.

# 시작하기

당신은 NASA의 최고 소프트웨어 기술자이며 최신 화성 유인탐사 미션의 스태프이다. 팀은 Marslink라는 앱의 첫번째 버전을 이미 작성했으며, 이곳에서 [다운로드](https://koenig-media.raywenderlich.com/uploads/2016/12/Marslink_Starter.zip)받을 수 있다. 이 앱을 다운로드 받은 뒤 _'Marslink.xcworkspace'_ 를 열고 앱을 실행시켜라.

![MarsLink 첫 실행](/files/iglistkit/1.png)

아직까지 어플리케이션은 우주인의 일지(Journal)만을 보여주고 있다. 당신이 해야 할 일은 크루가 어떠한 새로운 기능을 원하든지 추가해주는 것이다. _'Marslink/ViewControllers/ClassicFeedViewController.swift'_ 파일을 훑어보며 어떤 프로젝트인지 살펴보자. 만약 당신이 _'UICollectionView'_ 를 사용해 본 적이 없다면, 저 파일이 거의 표준적인 모습이라고 생각하면 된다.  

- _'ClassicFeedViewController'_ 는 _'UICollectionViewDataSource'_ 를 상속받는 _'UIViewController'_ 의 서브클래스이다.
- _'viewDidLoad()'_ 는 _'UICollectionView'_ 를 생성하고, 셀을 등록하며, 데이터소스를 설정하고, 뷰에 컬렉션뷰를 추가해준다.
- _'loader.entries'_ 배열은 개별 섹션이 각각 날짜를 나타내는 셀과, 텍스트(일지)를 나타내는 두 개의 셀을 포함하도록 설정해준다.
- 날짜 셀은 [Sol date](https://en.wikipedia.org/wiki/Timekeeping_on_Mars)를 가지고 있고, 텍스트 셀은 일지의 텍스트를 가지고 있다.
- _'collectionView(\_:layout:sizeForItemAt:)'_ 은 날짜 셀의 고정된 사이즈와 텍스트 양에 맞춰 계산된 텍스트 셀의 사이즈를 반환한다.

모두 다 제대로 작동하는 것 같다. 하지만 미션의 관리자가 긴급한 업데이트를 요청해왔다.

> 우주인이 화성에 고립되었다. 우리는 48시간 이내에 날씨 모듈과 실시간 채팅 기능을 업데이트 해야한다.

![추가해야 할 기능들](/files/iglistkit/2.png)

[JPL](https://en.wikipedia.org/wiki/Jet_Propulsion_Laboratory)의 엔지니어들은 이미 작동이 가능한 이러한 종류의 시스템을 구현해놓았다. 당신이 해야할 일은 이 시스템을 어플리케이션 속에 집어넣는 일이다. 

# IGListKit 소개

_'UICollectionView'_ 는 강력한 도구이지만, 강력한 힘에는 강력한 의무가 뒤따른다. 데이터 소스와 뷰를 동기화 상태로 유지하는 것이 가장 중요하며, 충돌(Crash)은 일반적으로 여기에서 발생하는 Disconnection으로 인해 발생한다.  
IGListKit은 [Instagram](https://engineering.instagram.com/open-sourcing-iglistkit-3d66f1e4e9aa#.iafykt5q3)팀이 만든 데이터 중심(data-driven) UICollectionView 프레임워크이다. 이 프레임워크를 사용하여 _'UICollectionView'_ 에 출력할 객체의 배열을 제공할 수 있다. 각 유형의 객체에 대해 IGListKit의 어댑터는 셀을 작성하기위한 모든 세부 사항을 갖는 _'Section Controller'_ 라는 것을 참조한다. 아래의 이미지는 IGListKit의 구조이다.

![IGListkit의 Flow Chart](/files/iglistkit/3.png)

IGListKit은 객체에 대해 자동으로 Diff를 시행하고, 변경사항에 대해 _'UICollectionView'_ 상에 애니메이션을 포함한 일괄적 업데이트(batch update)를 시행한다. 이 방법을 이용하면 batch update를 직접 작성할 필요가 없어지고, [이런](https://www.raizlabs.com/dev/2014/02/animating-items-in-a-uicollectionview/) 문제들을 예방할 수 있다.

# UICollectionView를 IGListKit으로 대체하기

IGListKit은 컬렉션 데이터의 변경 사항을 식별하고, 적절한 행에 애니메이션과 함께 추가하는 역할을 수행한다. 또한 다른 타입의 데이터와 다른 구조의 UI를 가진 복수의 섹션들을 쉽게 핸들링 할 수 있도록 구성되어 있다. 이러한 점을 미루어 봤을 때, IGListKit은 새로운 요구사항의 배치를 위한 최상의 해결책이다. 이제 implement를 시작해보자.

우선 _'Marslink.xcworkspace'_ 에서 계속 작업을 한다. _'ViewContollers'_ 그룹에서 우클릭하여 _'New File...'_ 을 선택한다. 이어서 _'Cocoa Touch Class'_ 를 선택하고, _'UIViewController'_ 를 상속하는 _'FeedViewController'_ 를 생성해준다.

_'AppDelegate.swift'_ 를 열어 _'application(:didFinishLaunchWithOptions:)'_ 를 찾아준다. 메소드 내에서 _'ClassicFeedViewController()'_ 를 _'Navigation Controller'_ 로 밀어주는(push) 라인을 찾아 다음의 내용으로 대체해준다.

```swift
nav.pushViewController(FeedViewController(), animated: false)
```

위 라인을 설정해줌으로서 _'FeedViewController'_ 가 이제 루트 뷰 컨트롤러가 되었다. ClassicFeedViewController.swift는 참고용으로 남겨두어도 좋다. 그러나 _'FeedViewController'_ 가 IGListKit으로 굴러가는 컬렉션 뷰를 생성할 곳임을 기억하자.

빌드후 실행을 하여서, 아래와 같이 빈 뷰컨트롤러가 화면에 나타나는지 확인한다.

![빈 뷰 컨트롤러](/files/iglistkit/4.png)

## 일지(Journal) Loader 추가하기
_'FeedViewController.swift'_ 파일을 열고, 다음 속성을 최상단에 추가해준다.

```swift
let loader = JournalEntryLoader()
```

_'JournalEntryLoader'_ 는 하드코딩 된 일지를 entries 배열에 로드해주는 클래스이다.

다음의 내용을 viewDidLoad()메소드의 제일 밑에 입력해준다.

```swift
loader.loadLatest()
```

_'loadLatest()'_ 는 최신 상태의 일지를 로드해주는 _'JournalEntryLoader'_ 의 메소드이다.

## 컬렌션 뷰 추가하기

이제 IGListKit만의 컨트롤을 뷰 컨트롤러에 추가해줄 시간이다. 그에 앞서, IGListKit Framework를 뷰컨트롤러에 임포트해주어야 한다. _'FeedViewController'_ 의 상단에 새로운 임포트를 추가해준다.

```swift
import IGListKit
```

> NOTE: 이 튜토리얼의 프로젝트는 [CocoaPods](https://cocoapods.org/)를 사용해 의존성을 관리한다. IGListKit은 Objectrive-C로 쓰여졌기 때문에, 수동으로 프로젝트에 추가하기를 원한다면 [bridging header](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html#//apple_ref/doc/uid/TP40014216-CH10-ID156)에 #import를 해주어야 한다.

_'FeedViewController'_ 의 상단에 intialize된 collectionView를 추가해준다.

```swift
//1
let collectionView: IGListCollectionView = {
	//2
	let view = IGListCollectionView(frame: CGRect.zero, collectionViewLayout: UICollectionViewFlowLayout())
	//3
	view.backgroundColor = UIColor.black
	return view
}()
```

1. IGListKit은 _'IGListCollectionView'_를 사용하는데, 이것은 UICollectionView의 서브클래스 이며, 기존의 UICollectionView에 비해 몇몇 기능성들이 패치되었고, [다른 것들을 예방](https://github.com/Instagram/IGListKit/blob/master/Source/IGListCollectionView.h)할 수 있도록 구성되어 있다.
2. 아직 뷰를 생성하지 않았기 때문에 사이즈0의 사각형에서 시작한다. 레이아웃은 _'ClassicFeedViewController'_ 가 그랬던 것 처럼 _'UICollctionViewFlowLayout'_ 을 사용하게 된다.
3. 배경의 색깔은 NASA가 승인한 검정색을 사용한다.

다음 내용을 _'viewDidLoad()'_ 의 하단에 추가하여, 컨트롤러의 뷰에 방금 생성한 collectionView를 부착해준다.

```swift
view.addSubview(collectionView)
```

이어서, _'viewDidLoad()'_ 메소드 바로 아래에 다음의 메소드를 추가해준다.

```swift
override func viewDidLayoutSubviews() {
	super.viewDidLayoutSubviews()
	collectionView.frame = view.bounds
}
```

_'viewDidLayoutSubviews()'_ 을 오버라이딩 하며 우리가 사용할 _'collectionView'_ 의 프레임을 View의 경계와 일치하도록 세팅해준다.

## IGListAdapter와 데이터소스

UICollectionView를 사용하기 위해서는 _'UICollectionViewDataSource'_ 를 적용한 데이터 소스들이 필요했다. 섹션과 행의 갯수 그리고 개별 셀을 반환하기 위해 필요한 작업이었다.  
IGListKit에서는 컬렉션 뷰를 컨트롤하기 위해 _'IGListAdapter'_ 라고 불리는 것을 사용한다. 또한 _'IGListAdapterDataSource'_ 프로토콜을 충족시키는 데이터소스가 필요하다. 이것은 기존의 _'UICollectionViewDataSource'_ 와는 다르게 개별 셀과 셀의 갯수를 반환하는 대신 객체 배열과 섹션 컨트롤러(Section Controller, 잠시 후에 더 자세히 다룬다.) 를 반환한다.

_'FeedViewController.swift'_ 파일의 상단에 다음을 추가해준다.

```swift
lazy var adapter: IGListAdapter = {
	return IGListAdapter(updater: IGListAdapterUpdater(), viewController: self, workingRangeSize: 0)
}()
```

이것은 lazily-initialized된 변수인 IGListAdapter를 생성한다. 생성자는 세개의 파라미터를 필요로 한다.

1. updater는 행과 섹션의 업데이트를 처리하여 IGListUpdatingDelegate를 충족시키기 위한 객체이다. 기본적으로 IGListAdapterUpdater를 사용하면 된다.
2. viewController는 어댑터의 집(Home)에 해당하는 UIViewController를 가리킨다. 해당 뷰 컨트롤러는 이후에 다른 뷰컨트롤러로 Navigating하기 위해 사용된다.
3. workingRangeSize는 [working range](https://instagram.github.io/IGListKit/getting-started.html#working-range)의 사이즈를 의미한다. working range는 사용자가 현재 보고 있는 프레임 밖에서 컨텐츠를 로딩할 수 있도록 돕는다.

> NOTE: Working Range는 더 어려운 개념이라서 이 튜토리얼에서는 다루지 않는다. 그러나 도큐멘테이션과 예시 어플리케이션이 [IGListKit Repo](https://instagram.github.io/IGListKit/)에 있으니 사용하고 싶다면 참고하면 된다. 

다음 내용을 _'viewDidLoad()'_ 의 하단에 추가해준다.

```swift
adapter.collectionView = collectionView
adapter.dataSource = self
```

컬렉션 뷰를 어댑터로 연결시켜주는 내용이다. 또한 self를 어댑터의 데이터소스로 설정해준다. (아마 컴파일 에러가 발생할 것이다. 아직 IGListAdapterDataSource 프로토콜을 적용시키지 않았기 때문이다.)

_'FeedViewController'_ 에 _'IGListAdapterDataSource'_ 를 적용하면 오류를 고칠 수 있다. 파일 최하단에 다음 내용을 추가해주면 된다.

```swift
extension FeedViewController: IGListAdapterDataSource {
	//1
	func objects(for listAdapter: IGListAdapter) -> [IGListDiffable] {
		return loader.entries
	}

	//2
	func listAdapter(_ listAdapter: IGListAdapter, sectionControllerFor object: Any) -> IGListSectionController {
		return IGListSectionController()
	}

	//3
	func emptyView(for listAdapter: IGListAdapter) -> UIView? {return nil}
}
```

이제 _'FeedViewController'_ 에 IGListAdapterDataSource를 부착했고, 세개의 요구되는 메소드를 상속한다. 

- _'objects(for:)'_ 는 컬렉션 뷰에 나타날 데이터 객체의 배열을 반환한다. 여기서는 Journal Entry들을 갖고 있는 _'loader.entries'_를 넘겨준다.
- _'listAdapter(_:sectionControllerFor:)'_ 메소드는 각각의 데이터 객체 타입에 상응하는 _'Section Controller'_ 의 인스턴스를 반환해준다. 아직은 컴파일 에러를 막기 위해 임시 방편으로 순수한 _'IGListSectionController'_ 를 반환해주고 있다. 나중에 이것을 수정하여 커스텀 Journal Section을 반환하도록 만들어주어야 한다.
- _'emptyView(for:)'_ 는 출력될 리스트가 비어있을 때 보여줄 뷰를 리턴한다. NASA는 시간이 부족하기 때문에 아직 이 기능에 예산을 책정하지 않았다.

## 첫 Section Controller 만들기

_'Section Controller'_ 는 주어진 데이터 객체를 추상화하고, 컬렉션 뷰의 섹션 내의 셀을 컨트롤하고 구성(configure)해주는 역할을 한다. 이 컨셉은 뷰를 구성하는 view-model 방식과 유사하다. 데이터 객체는 View-model이고, 셀은 View와 같다. section controller는 둘 사이의 접착제 역할을 한다.

IGListKit을 사용하여, 다른 타입의 데이터와 행동(Behavior)을 위한 새로운 Section Controller를 만들어 보자. JPL 기술자들은 이미 _'JournalEntry'_ 모델을 만들었다. 따라서 그것을 다룰 수 있는 Section Controller를 제작할 필요가 있다.

네비게이터에서 _'Section Controller'_ 그룹을 우클릭 하여 새 파일을 생성해준다. Cocoa Touch Class 를 선택해주고, IGListSectionController의 서브클래스인 JournalSectionController를 생성해준다.

![JournalSectionController 생성창](/files/iglistkit/5.png)

Xcode는 서드파티 프레임워크를 자동으로 임포트해주지 않기 때문에 _'JournalSectionController.swift'_ 파일의 최상단에 다음을 추가해주어야 한다.

```swift
import IGListKit
```

다음 속성들을 _'JournalSectionController:'_ 의 상단에 추가해준다.

```swift
var entry: JournalEntry!
let solFormatter = SolFormatter()
```

_'JournalEntry'_ 는 데이터 소스를 받아올 떄 사용할 모델 클래스이다. _'SolFormatter'_ 클래스는 날짜를 Sol format으로 변환시켜주는 메소드를 제공한다.

또한 _'JournalSectionController'_ 내부에 _'init()'_ 메소드를 다음과 같이 추가하여 오버라이드 해준다.

```swift
override init(){
	super.init()
	inset = UIEdgeInsets(top:0, left:0, botton: 15, right: 0)
}
```

이 내용이 없으면, 섹션들이 다닥다닥 붙어서 나타날 것이다. 이것을 이용해 15포인트의 padding을 JournalSectionController 객체의 바닥에 추가해준다.

새로 만든 Section Controller는 IGListKit에서 사용되기 위해 IGListSectionType의 프로토콜을 충족해야한다. 다음 extension을 파일 하단에 추가해주자.

```swift
extension JournalSectionController: IGListSectionType {
	func numberOfItems() -> Int {
		return 2
	}

	func sizeForItem(at index: Int) -> CGSize {
		return .zero
	}

	func cellForItem(at index: Int) -> UICollectionViewCell {
		return UICollectionViewCell()
	}

	func didUpdate(to object: Any) {
	}

	func didSelectItem(at index: Int) {}
}
```

_'IGListSectionType'_ 프로토콜을 충족하기 위해 4개의 요구되는 메소드를 상속했다. _'numberOfItems()'_ 메소드를 제외한 모든 메소드들은 스텁값들이다.

_'didUpdate(:to)'_ 에 다음 내용을 추가해주자.

```swift
entry = object as? JournalEntry
```

_'didUpdate(:to)'_ 는 객체를 Section Controller로 전달하기 위해 사용된다. 이 메소드는 항상 셀 프로토콜 메소드보다 먼저 호출된다. 여기서는 전달된 객체를 _'entry'_ 에 저장한다.

> NOTE: 객체는 Section Controller의 생애주기동안 수차례 변경될 수 있다. 그런 일은 Custom Model Diffing과 같은 더 고급 기능을 사용할 때 일어날 것이다. 이번 튜토리얼에서는 걱정하지 않아도 된다.

이제 데이터를 가졌으니, 셀을 구성해줄 차례이다. _'cellForItem(at:)'_ 의 _'return UICollectionViewCell()'_ 를 다음의 내용으로 대체해준다.

```swift
// 1
let cellClass: AnyClass = index == 0 ? JournalEntryDateCell.self : JournalEntryCell.self
// 2
let cell = collectionContext!.dequeueReusableCell(of: cellClass, for: self, at: index)
// 3
if let cell = cell as? JournalEntryDateCell {
	cell.label.text = "SOL \(solFormatter.sols(fromDate: entry.date))"
} else if let cell = cell as? JournalEntryCell {
	cell.label.text = entry.text
}
return cell
```

_'cellForItem(at:)'_ 은 섹션내의 주어진 인덱스에서 셀이 요구될 때 호출된다. 내부에서는 다음과 같은 일들이 일어난다.

1. 만약에 첫번째 인덱스일 경우, _'JournalEntryDateCell'_ 을 사용한다, 아니면 _'JournalEntryCell'_ 을 사용한다. 따라서 JournalEntry 섹션은 항상 날짜를 나타내는 셀과 그 뒤를 따르는 텍스트를 나타내는 셀로 구성될 것이다.
2. 셀 클래스, 섹션 컨트롤러와 인덱스를 사용해 재사용 풀(reuse pool)에서 셀을 찾아낸다. 
3. 셀 타입에 따라 앞에 _'didUpdate(:to object)'_ 에서 설정했던 JournalEntry를 사용하도록 설정해준다.

이어서 _'sizeForItem(:at)'_ 의 _'return UICollectionViewCell()'_를 다음 내용으로 대체해준다.

```swift
// 1
guard let context = collectionContext, let entry = entry else { return .zero }
// 2
let width = context.containerSize.width
// 3
if index == 0 {
	return CGSize(width: width, height: 30)
} else {
	return JournalEntryCell.cellSize(width: width, text: entry.text)
}
```

1. _'collectionContext'_ 는 weak한 변수이며, null이 가능해야 하지만 nil이어서는 안된다. Swift의 guard를 사용하면 편리하게 구현할 수 있다.
2. _'IGListCollectionContext'_ 는 Section Controller에서 사용되는 어댑터, 컬렉션 그리고 뷰 컨트롤러에 대한 정보를 갖고있는 context 객체이다. 여기서는 컨테이너의 너비에 대한 정보를 얻기위해 필요로 한다.
3. 첫 번째 인덱스(날짜 셀)은 컨테이너만큼의 너비와 30포인트의 높이를 반환하고, 그렇지 않으면 셀 헬퍼 메소드를 사용해 텍스트 셀의 사이즈를 동적으로 계산한다.

마지막으로 살펴볼메소드는 누군가가 셀을 탭했을 때 호출되는 _'didSelectItem(at:)'_ 이다. 기본적으로 요구되는 메소드이기 때문에 반드시 추가해주어야 한다. 하지만 이 섹션에 대해서는탭에 따른 상호작용이 필요하지 않기 때문에 비워둔다.

이전에 _'UICollectionView'_ 를 사용해본 적이 있다면 서로다른 타입의 셀을 Dequeuing하고, 구성하고, 사이즈를 반환하는 이러한 패턴이 익숙할 것이다. _'ClassicFeedViewController'_ 로 돌아가보면 많은 코드들이 비슷한 것을 확인할 수 있다.

이제 JournalEntry 객체를 받아와서 두 셀의 사이즈를 반환하는 Section Controller를 만들었기 때문에 다 함께 묶을 시간이다.

_'FeedViewController.swift'_ 로 돌아가서 _'listAdapter(_:sectionControllerFor:)'_ 의 내용을 다음으로 대체해준다.

```swift
return JournalSectionController()
```

이제 _'listAdapter(_:sectionControllerFor:)'_ 메소드가 호출되면 방금 만들었던 Journal Seciton Controller를 반환한다. 앱을 빌드하고 실행해보면 글이 나타나는 것을 확인할 수 있다.

![Journal Entry](/files/iglistkit/6.png)

# 메세지 기능 추가하기

JPL 엔지니어링은 당신의 빠른 리팩토링에 기뻐했다. 하지만 그들은 고립된 우주인과의 교신기능이 절실히 필요하다. 그들은 최대한 빨리 메세징 기능을 만들어줄 것을 요구해왔다.

뷰를 추가하기에 앞서 당신은 데이터가 필요하다.

_'FeedViewController.swift'_ 파일을 열어서 상단에 새로운 속성을 추가해준다.

```swift
let pathfinder = Pathfinder()
```

_'pathfinder'_ 는 메세징 시스템으로 작동하며, 우주인이 화성에서 파낸 실제 패스파인더 로버를 의미한다.(영화 마션 참조)

_'IGListAdapterDataSource'_  extension 안에서 _'objects(for:)'_ 를 찾아 아래와 같이 내용을 수정해주어라.

```swift
var items: [IGListDiffable] = pathfinder.messages
items += loader.entries as [IGListDiffable]
return items
```

이 메소드가 IGListAdapter에게 데이터 소스 객체를 전달해준다는 사실을 다시 기억해보자. 이 수정을 통해 
pathfinder.messages를 아이템에 새로이 추가하여 새 섹션 컨트롤러에 대한 메시지를 제공한다.

Section Controllers그룹을 우클릭하여 IGListSectionController의 서브클래스인 MessageSectionController를 생성한 후, 상단에 IGListKit을 임포트 해준다.

```swift
import IGListKit
```

컴파일 에러를 피하기 위해, 우선은 그대로 놔둔 뒤 _'FeedViewController'_ 로 돌아간다. _'IGListAdapterDataSource'_ extension 안에 _'listAdapter(\_:sectionControllerFor:)'_ 를 다음과 같이 업데이트 해준다.

```swift
if object is Message {
	return MessageSectionController()
} else {
	return JournalSectionController()
}
```

수정을 통해 데이터 객체에 따른 분기처리를 해준다. 만약에 객체가 Message라면 새로 만들게 될 Message Section Controller를 반환할 것이다.  
JPL팀은 _'MessageSectionController'_ 에 다음과 같은 사항을 요구해왔다.

- Message 수신
- Bottom inset 15pt
- MessageCell.cellSize(width:text:)기능을 이용한 개별 셀 사이징
- Message 객체의 텍스트와 user.name의 값을 이용해 라벨을 생성하고 MessageCell을 구성함 

요구사항을 바탕으로 직접 구현해보자. 솔루션은 아래와 같다.

<details>
  <summary>솔루션</summary>
  <div class="language-swift highlighter-rouge"><pre class="highlight"><code>
<span class="cp"># MessageSectionController.swift</span>

<span class="kd">import</span> <span class="kt">IGListKit</span>

<span class="kd">class</span> <span class="kt">MessageSectionController</span><span class="p">:</span> <span class="kt">IGListSectionController</span> <span class="p">{</span>

	<span class="k">var</span> <span class="nv">message</span><span class="p">:</span> <span class="kt">Message</span><span class="o">!</span>

	<span class="k">override</span> <span class="nf">init</span><span class="p">()</span> <span class="p">{</span>
		<span class="k">super</span><span class="o">.</span><span class="nf">init</span><span class="p">()</span>
		<span class="n">inset</span> <span class="o">=</span> <span class="kt">UIEdgeInsets</span><span class="p">(</span><span class="nv">top</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="nv">left</span><span class="p">:</span> <span class="mi">0</span><span class="p">,</span> <span class="nv">bottom</span><span class="p">:</span> <span class="mi">15</span><span class="p">,</span> <span class="nv">right</span><span class="p">:</span> <span class="mi">0</span><span class="p">)</span>
	<span class="p">}</span>
<span class="p">}</span>

<span class="kd">extension</span> <span class="kt">MessageSectionController</span><span class="p">:</span> <span class="kt">IGListSectionType</span> <span class="p">{</span>
	<span class="kd">func</span> <span class="nf">numberOfItems</span><span class="p">()</span> <span class="o">-&gt;</span> <span class="kt">Int</span> <span class="p">{</span>
		<span class="k">return</span> <span class="mi">1</span>
	<span class="p">}</span>

	<span class="kd">func</span> <span class="nf">sizeForItem</span><span class="p">(</span><span class="n">at</span> <span class="nv">index</span><span class="p">:</span> <span class="kt">Int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">CGSize</span> <span class="p">{</span>
		<span class="k">guard</span> <span class="k">let</span> <span class="nv">context</span> <span class="o">=</span> <span class="n">collectionContext</span> <span class="k">else</span> <span class="p">{</span> <span class="k">return</span> <span class="o">.</span><span class="n">zero</span> <span class="p">}</span>
		<span class="k">return</span> <span class="kt">MessageCell</span><span class="o">.</span><span class="nf">cellSize</span><span class="p">(</span><span class="nv">width</span><span class="p">:</span> <span class="n">context</span><span class="o">.</span><span class="n">containerSize</span><span class="o">.</span><span class="n">width</span><span class="p">,</span> <span class="nv">text</span><span class="p">:</span> <span class="n">message</span><span class="o">.</span><span class="n">text</span><span class="p">)</span>
	<span class="p">}</span>

	<span class="kd">func</span> <span class="nf">cellForItem</span><span class="p">(</span><span class="n">at</span> <span class="nv">index</span><span class="p">:</span> <span class="kt">Int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">UICollectionViewCell</span> <span class="p">{</span>
		<span class="k">let</span> <span class="nv">cell</span> <span class="o">=</span> <span class="n">collectionContext</span><span class="p">?</span><span class="o">.</span><span class="nf">dequeueReusableCell</span><span class="p">(</span><span class="nv">of</span><span class="p">:</span> <span class="kt">MessageCell</span><span class="o">.</span><span class="k">self</span><span class="p">,</span> <span class="nv">for</span><span class="p">:</span> <span class="k">self</span><span class="p">,</span> <span class="nv">at</span><span class="p">:</span> <span class="n">index</span><span class="p">)</span> <span class="k">as!</span> <span class="kt">MessageCell</span>
		<span class="n">cell</span><span class="o">.</span><span class="n">messageLabel</span><span class="o">.</span><span class="n">text</span> <span class="o">=</span> <span class="n">message</span><span class="o">.</span><span class="n">text</span>
		<span class="n">cell</span><span class="o">.</span><span class="n">titleLabel</span><span class="o">.</span><span class="n">text</span> <span class="o">=</span> <span class="n">message</span><span class="o">.</span><span class="n">user</span><span class="o">.</span><span class="n">name</span><span class="o">.</span><span class="nf">uppercased</span><span class="p">()</span>
		<span class="k">return</span> <span class="n">cell</span>
	<span class="p">}</span>

	<span class="kd">func</span> <span class="nf">didUpdate</span><span class="p">(</span><span class="n">to</span> <span class="nv">object</span><span class="p">:</span> <span class="kt">Any</span><span class="p">)</span> <span class="p">{</span>
		<span class="n">message</span> <span class="o">=</span> <span class="n">object</span> <span class="k">as?</span> <span class="kt">Message</span>
	<span class="p">}</span>

	<span class="kd">func</span> <span class="nf">didSelectItem</span><span class="p">(</span><span class="n">at</span> <span class="nv">index</span><span class="p">:</span> <span class="kt">Int</span><span class="p">)</span> <span class="p">{}</span>
<span class="p">}</span>

</code></pre>
</div>
</details>

빌드후 실행시켜보면 메세지 기능이 피드에 잘 합쳐져있는 것을 확인할 수 있다.

![추가된 메세지 기능](/files/iglistkit/7.png)

# 화성의 날씨

우리의 우주인은 모래폭풍과 같은 것들 피하기 위해 날씨 정보가 필요하다. JPL은 현재 날씨를 표시해주는 새로운 모듈을 만들어냈다. 정보가 많기 때문에, 탭을 했을 때만 확장되어 표시되도록 요구하고 있다.

![요구받은 날씨 기능](/files/iglistkit/8.gif)

_'WeatherSectionController'_ 라는 마지막 section controller를 만들어준다. 생성자와 몇개의 변수를 선언하는 것으로 시작해보자.

```swift
import IGListKit

class WeatherSectionController: IGListSectionController {
	// 1
	var weather: Weather!
	// 2
	var expanded = false

	override init() {
		super.init()
		// 3
		inset = UIEdgeInsets(top: 0, left: 0, bottom: 15, right: 0)
	}
}
```

1. 이 section controller는 _'didUpdate(:to)'_ 를 통해 Wheather객체를 받을 것이다.
2. _'expanded'_ 는 이 날씨 섹션이 확장되었는지 아닌지를 알아내기위한 불린이다. 기본적으로 false로 생성된다.
3. 다른 섹션들과 마찬가지로 bottom inset은 15pt이다.

이제 IGListSectionType을 충족하기 위해 extension과 요구되는 메소드들을 추가해준다.

```swift
extension WeatherSectionController: IGListSectionType {
	// 1
	func didUpdate(to object: Any) {
		weather = object as? Weather
	}

	// 2
	func numberOfItems() -> Int {
		return expanded ? 5 : 1
	}

	// 3
	func sizeForItem(at index: Int) -> CGSize {
		guard let context = collectionContext else { return .zero }
		let width = context.containerSize.width
		if index == 0 {
			return CGSize(width: width, height: 70)
		} else {
			return CGSize(width: width, height: 40)
		}
	}
}
```

1. _'didUpdate(:to)'_ 에서 넘겨받은 Wheather객체를 저장한다.
2. 만약 날씨 섹션이 확장되었다면, _'numberOfItems()'_ 는 각각 다른 날씨 정보를 담고 있는 5개의 셀을 반환할 것이다. 확장되지 않았다면, placeholder를 표시해주어야 한다.
3. 첫 번째 셀은 헤더를 표시하기 때문에 다른 셀들보다 약간 더 커야한다.

다음은 날씨 셀을 구성하기 위해 _'cellForItem(at:)'_ 을 추가해주어야 한다. 다음과 같은 세부 요구사항이 있다.

- 첫 번째 셀은 _'WheatherSummaryCell'_ 타입이어야 한다. 나머지는 _'WeatherDetailCell'_ 타입이다.
- 날씨 요약 셀을 _'cell.setExpanded(_:)'_ 와 함께 구성한다.
- 네 개의 다른 세부날씨 셀을 구성한다. 각각의 셀은 다음과 같은 세부 라벨을 가진다.
    1. 'weathre.sunrise' 를 가지는 "Sunrise"
    2. 'weather.sunset' 을 가지는 "Sunset"
    3. '\(weather.high) C' 를 가지는 "High"
    4. '\(weather.low) C' 를 가지는 "Low"

요구사항을 바탕으로 직접 구현해보자. 솔루션은 아래와 같다.
<details>
  <summary>솔루션</summary>

  <div class="language-swift highlighter-rouge"><pre class="highlight"><code><span class="kd">func</span> <span class="nf">cellForItem</span><span class="p">(</span><span class="n">at</span> <span class="nv">index</span><span class="p">:</span> <span class="kt">Int</span><span class="p">)</span> <span class="o">-&gt;</span> <span class="kt">UICollectionViewCell</span> <span class="p">{</span>
	<span class="k">let</span> <span class="nv">cellClass</span><span class="p">:</span> <span class="kt">AnyClass</span> <span class="o">=</span> <span class="n">index</span> <span class="o">==</span> <span class="mi">0</span> <span class="p">?</span> <span class="kt">WeatherSummaryCell</span><span class="o">.</span><span class="nv">self</span> <span class="p">:</span> <span class="kt">WeatherDetailCell</span><span class="o">.</span><span class="k">self</span>
	<span class="k">let</span> <span class="nv">cell</span> <span class="o">=</span> <span class="n">collectionContext</span><span class="o">!.</span><span class="nf">dequeueReusableCell</span><span class="p">(</span><span class="nv">of</span><span class="p">:</span> <span class="n">cellClass</span><span class="p">,</span> <span class="nv">for</span><span class="p">:</span> <span class="k">self</span><span class="p">,</span> <span class="nv">at</span><span class="p">:</span> <span class="n">index</span><span class="p">)</span>
	<span class="k">if</span> <span class="k">let</span> <span class="nv">cell</span> <span class="o">=</span> <span class="n">cell</span> <span class="k">as?</span> <span class="kt">WeatherSummaryCell</span> <span class="p">{</span>
		<span class="n">cell</span><span class="o">.</span><span class="nf">setExpanded</span><span class="p">(</span><span class="n">expanded</span><span class="p">)</span>
	<span class="p">}</span> <span class="k">else</span> <span class="k">if</span> <span class="k">let</span> <span class="nv">cell</span> <span class="o">=</span> <span class="n">cell</span> <span class="k">as?</span> <span class="kt">WeatherDetailCell</span> <span class="p">{</span>
		<span class="k">let</span> <span class="nv">title</span><span class="p">:</span> <span class="kt">String</span><span class="p">,</span> <span class="nv">detail</span><span class="p">:</span> <span class="kt">String</span>
		<span class="k">switch</span> <span class="n">index</span> <span class="p">{</span>
			<span class="k">case</span> <span class="mi">1</span><span class="p">:</span>
			<span class="n">title</span> <span class="o">=</span> <span class="s">"SUNRISE"</span>
			<span class="n">detail</span> <span class="o">=</span> <span class="n">weather</span><span class="o">.</span><span class="n">sunrise</span>
			<span class="k">case</span> <span class="mi">2</span><span class="p">:</span>
			<span class="n">title</span> <span class="o">=</span> <span class="s">"SUNSET"</span>
			<span class="n">detail</span> <span class="o">=</span> <span class="n">weather</span><span class="o">.</span><span class="n">sunset</span>
			<span class="k">case</span> <span class="mi">3</span><span class="p">:</span>
			<span class="n">title</span> <span class="o">=</span> <span class="s">"HIGH"</span>
			<span class="n">detail</span> <span class="o">=</span> <span class="s">"</span><span class="se">\(</span><span class="n">weather</span><span class="o">.</span><span class="n">high</span><span class="se">)</span><span class="s"> C"</span>
			<span class="k">case</span> <span class="mi">4</span><span class="p">:</span>
			<span class="n">title</span> <span class="o">=</span> <span class="s">"LOW"</span>
			<span class="n">detail</span> <span class="o">=</span> <span class="s">"</span><span class="se">\(</span><span class="n">weather</span><span class="o">.</span><span class="n">low</span><span class="se">)</span><span class="s"> C"</span>
			<span class="k">default</span><span class="p">:</span>
			<span class="n">title</span> <span class="o">=</span> <span class="s">"n/a"</span>
			<span class="n">detail</span> <span class="o">=</span> <span class="s">"n/a"</span>
		<span class="p">}</span>
		<span class="n">cell</span><span class="o">.</span><span class="n">titleLabel</span><span class="o">.</span><span class="n">text</span> <span class="o">=</span> <span class="n">title</span>
		<span class="n">cell</span><span class="o">.</span><span class="n">detailLabel</span><span class="o">.</span><span class="n">text</span> <span class="o">=</span> <span class="n">detail</span>
	<span class="p">}</span>
	<span class="k">return</span> <span class="n">cell</span>
<span class="p">}</span>
</code></pre>
</div>

 </details>

마지막으로 해야 할 일은 셀이 탭되었을 때 섹션을 확장하고, 셀을 업데이트 하는 것이다. 마지막 메소드는 다음과 같다.

```swift
func didSelectItem(at index: Int) {
	expanded = !expanded
	collectionContext?.reload(self)
}
```

_'reload()'_ 는 전체 섹션을 새로고침한다. 섹션 컨트롤러의 셀의 숫자나 내용이 바뀌었을 때 언제라도 이것을 사용할 수 있다. _'numberOfItems()'_ 를 이용해 확장을 구현 했기 때문에, 이것은 _'expanded'_ 플래그에 따라 셀을 더하거나 없애거나 할 것이다.

이제 Section Controller에 객체를 넘겨주는 부분을 구현할 시간이다. 다시 _'FeedViewController.swift'_ 로 돌아와서  _'FeedViewController'_ 의 상단 근처에 다음 내용을 작성해준다.

```swift
let wxScanner = WxScanner()
```

_'WxScanner'_ 는 기상 정보를 위한 모델 객체이다.

이어서, _'IGListAdapterDataSource'_  extension안에  _'objects(:for)'_ 를 다음과 같이 수정해준다.

```swift
// 1
var items: [IGListDiffable] = [wxScanner.currentWeather]
items += loader.entries as [IGListDiffable]
items += pathfinder.messages as [IGListDiffable]
// 2
return items.sorted(by: { (left: Any, right: Any) -> Bool in
	if let left = left as? DateSortable, let right = right as? DateSortable {
		return left.date > right.date
	}
	return false
})
```

데이터 소스가 _'currentWeather'_ 를 포함하도록 업데이트 한 것이다.

1. _'currentWeather'_ 를 항목들 배열에 추가하기
2. 모든 데이터는 DataSortable 프로토콜을 준수하므로, 이를 사용하여 정렬한다. 이렇게하면 데이터가 시간순으로 표시된다.

마지막으로 _'listAdapter(\_:sectionControllerFor:)'_ 를 다음과 같이 바꾸어준다.

```swift
if object is Message {
	return MessageSectionController()
} else if object is Weather {
	return WeatherSectionController()
} else {
	return JournalSectionController()
}
```

이제 만약에 _'Weather'_ 객체가 나타나면 _'WeatherSectionController'_를 반환하게 된다.

빌드하고 실행해보자. 최상단에 새로운 날씨가 나타나는 것을 확인할 수 있다. 섹션을 탭하면 확장되고 축소시킬 수 있다.

![날씨가 추가된 모습](/files/iglistkit/9.png)

# 업데이트 수행하기

JPL은 당신이 만들어낸 발전에 대해 매우 기뻐하고 있다. 당신이 일하는 동안 NASA의 책임자는 우주인의 구조 작업 스케쥴을 조정하여 지구에서 다른 우주선을 발사하고, 우주인도 화성에서 발사를 하여 지구에서 발사한 우주선으로 옮겨타는 방식으로 구조를 할 계획을 세우고 있다. 복잡한 발사가 될 것이며, 따라서 우주인도 정확한 시간에 이륙해야할 것이다.

JPL 엔지니어링은 메세징 모듈을 확장하여, 실시간 채팅을 추가해주기를 원한다.

_'FeedViewController.swift'_ 를 열고 _'viewDidLoad()'_ 의 마지막 줄에 다음의 내용을 추가해준다.

```swift
pathfinder.delegate = self
pathfinder.connect()
```

_'pathfinder'_ 모듈은 이미 실시간 기능을 지원하기 위해 패치가 되어있다. 당신이 해야할 일은 유닛을 연결하고, 약속 된 이벤트에 대해 반응하게 하는 것이다.

다음 extension을 파일 하단에 추가해준다.

```swift
extension FeedViewController: PathfinderDelegate {
	func pathfinderDidUpdateMessages(pathfinder: Pathfinder) {
		adapter.performUpdates(animated: true)
	}
}
```

_'FeedViewController'_ 는 이제 PathfinderDelegate를 충족한다. performUpdates(animated: completion:)은 _'IGListAdapter'_ 에게 자신의 데이터 소스를 새로운 객체로 UI에 추가해줄 것을 요청한다. 이같은 방식으로 객체의 삭제, 수정, 이동, 삽입을 처리할 수 있다. 

빌드후 실행을 시켜보면 캡틴의 메세지가 업데이트 되고 있는 것을 확인할 수 있다. 

![실시간 업데이트](/files/iglistkit/10.png)

# 마치며

완성된 프로젝트는 [이곳](https://koenig-media.raywenderlich.com/uploads/2016/12/MarsLink_Final.zip)에서 다운로드 받을 수 있다.

고립되었던 우주인을 집으로 데려오는 과정에서, 당신은 IGListKit의 기본적인 기능들을 많이 배울 수 있었다. Supplementary view와 디스플레이 이벤트와 같은 중요한 기능또한 살펴보았다. 

[Realm](https://realm.io/news/tryswift-ryan-nystrom-refactoring-at-scale-lessons-learned-rewriting-instagram-feed/)에서 발표했던 강연에서 Instagram의 IGListKit의 출처에 대해 더 많이 읽고 볼 수 있다. 이 강연에서는 앱이 커질 때 볼 수있는 일반적인 UICollectionView 문제에 대해 다루고, IGListKit이 어떻게 그것을 해결했는지 볼 수 있을 것이다. 


IGListKit에 기여하는 데 관심이 있다면, 깃허브에 설정된 [starter-task](https://github.com/Instagram/IGListKit/issues?q=is%3Aissue+is%3Aopen+label%3Astarter-task) 태그를 이용해 쉽게 시작할 수 있을 것이다.
