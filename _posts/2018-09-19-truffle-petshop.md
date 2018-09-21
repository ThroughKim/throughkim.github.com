---
layout: post
title: '트러플 펫샵 예제를 통해 이더리움 dApp 공부(작성중)'
author: through.kim
date: 2018-09-19 15:16
tags: [blockchain, solidity, dapp]
image: '/files/covers/solidity.png'
comments: true
---

> 트러플(Truffle suite)이란 이더리움 스마트 컨트렉트를 사용하는 dApp을 쉽고 편하게 개발할 수 있도록 돕는 툴이다.
이 툴을 사용해 펫샵을 만들어보는 튜토리얼을 따라해보며 공부해본다.

출처 : [Etherium Pet Show -- Your first dApp](https://truffleframework.com/tutorials/pet-shop)

이 튜토리얼을 통해 dApp을 만드는 프로세스를 경험해볼 수 있다. 만들어 볼 dApp은 펫샵을 위한 분양 트래킹 시스템이다.

이 튜토리얼은 이더리움과 스마트 컨트렉트에 대한 기본적인 지식과 HTML 및 JavaScript에 대한 지식이 있지만,
dApp에 익숙하지 않은 개발자를 대상으로 하고있다.
 
이 튜토리얼에서 다음 내용을 다룬다.

1. 개발 환경 구축
2. Truffle Box를 이용해 Truffle 프로젝트 제작
3. 스마트 컨트렉트 작성
4. 스마트 컨트렉트 컴파일, 마이그레이션
5. 스마트 컨트렉트 테스트
6. 스마트 컨트렉트와 상호작용하는 UI 제작
7. 브라우저에서 dApp 사용
 
## 배경

Pete's Pet Shop의 Pete Scandlon은 애완동물 입양을 처리하는 효율적인 방법으로 이더리움을 사용하는데 관심이 있었다.
상점에는 16마리의 애완동물을 위한 공간이 있으며, 애완동물의 데이터베이스를 가지고 있다. 
[Proof of Concept](https://ko.wikipedia.org/wiki/개념_증명)로 
Pete는 이더리움 주소와 입양된 애완동물을 연결해주는 dApp을 보고싶어 한다.

웹사이트의 구조와 CSS는 제공될 것이며, 우리가 해야 할 작업은 스마트 컨트렉와 그것을 사용할 프론트엔드 로직 작성이다. 

## 개발환경 구축

시작하기에 앞서 다음 두 요구사항을 설치해야 한다.
 
- [Node.js & npm](Node.js & npm)
- [Git](https://git-scm.com/)
 
설치 한 뒤에 Truffle을 설치하기 위해서는 한 줄의 명령어만 입력해주면 된다.

```bash
npm install -g truffle
```

Truffle이 정상적으로 설치되었는지 확인하려면 터미널에 `truffle version`을 타이핑해보면 된다.
만약 오류가 보인다면, npm모듈이 path에 제대로 추가되었는지 확인해봐야 한다.

또한 우리는 스마트 컨트렉트 배포, 앱 개발 및 테스트 실행에 활용할 개인 이더리움 블록체인인 [Ganache](https://truffleframework.com/ganache)를 사용할 것이다.
다음 주소 - [https://truffleframework.com/ganache](https://truffleframework.com/ganache) 에서 다운로드 받을 수 있다.

## Truffle Box를 이용해 Truffle 프로젝트 제작

1. Truflle은 현재 디렉토리에서 초기화되므로 우선 개발을 진행할 폴더에 새로운 디렉토리를 만들고 그 폴더 안으로 이동해야 한다.  

~~~
mkdir pet-shop-tutorial

cd pet-shop-tutorial
~~~
  
2. Truffle에서는 `pet-shop`이라 불리는 이 튜토리얼을 위해 특별한 [Truffle Box](https://truffleframework.com/boxes)를 만들었다.
이 Truffle Box에는 기본 프로젝트 구조는 물론이고 유저 인터페이스를 위한 코드도 마련되어 있다. `truffle unbox` 명령어를 사용해 이 Truffle Box를 열어볼 수 있다.

~~~
truffle unbox pet-shop
~~~
  
> Note: Truffle은 몇몇 다른 방법으로도 초기화될 수 있다. 또 하나의 유용한 초기화 명령어는 `truffle init`이며, 
예제 컨트렉트의 내용이 포함되지 않은 빈 프로젝트를 만들어 준다. 더 많은 정보를 원하면 [Creating a project](https://truffleframework.com/docs/truffle/getting-started/creating-a-project)
문서를 참조하면 된다.

### 디렉토리 구조

기본적인 Truffle 디렉토리 구조는 다음과 같다.

- `contracts/`: 스마트 컨트렉트를 위한 Solidity 소스 파일이 저장되어있는 폴더이다. 
`Migrations.sol`이라는 중요한 컨트렉트가 여기 안에 들어있다. 이 컨트렉트는 추후에 설명할 것이다.

- `migrations/`: Truffle은 스마트 컨트렉트 배포를 처리하기 위해 마이그레이션 시스템을 사용한다.
마이그레이션은 배포된 스마트 컨트렉트의 변화를 추적하는데 사용되는 부가적인 스마트 컨트렉트이다.

- `test/`: 스마트 컨트렉트를 테스트하기 위한 자바스크립트, Solidity 테스트 파일이 저장될 폴더이다.

- `truffle.js`: Truffle 설정 파일
 
 `pet-shop` Truffle Box는 이외에도 몇 개의 폴더가 더 있지만 지금 신경 쓸 내용은 아니다.
 
## 스마트 컨트렉트 작성

이제 백엔드 로직과 저장소 역할을 할 스마트 컨트렉트를 작성하면서 dApp 제작을 시작해본다.

1. `contracts/`폴더에 `Adoption.sol`파일을 만든다.
2. 다음과 같이 내용을 만들어준다.

```
pragma solidity ^0.4.17;

contract Adoption {

}
```
알아야 할 내용:

- 요구되는 Solidity의 최소 버전은 스마트 컨트렉트 최상단에 `pragma solidity ^0.4.17;`로 명시되어있다. 
`pragma`는 컴파일러에게 알려주는 추가적인 정보이다. 꺽쇠(^)는 '표시된 버전 혹은 그 이상'을 의미한다.
- 자바스크립트나 PHP처럼 세미콜론으로 라인을 끝낸다.
 
### 변수 세팅

Solidity는 정적타이핑 언어이기 때문에 string, integer, 배열 과 같은 데이터 타입을 미리 정의해주어야 한다.
Solidity는 address라고 불리는 특이한 데이터 타입을 가지고 있다.
address는 20바이트의 데이터로 이루어진 이더리움 주소 타입이다.
이더리움 블록체인 위의 모든 계정과 스마트 컨트렌트는 주소를 가지고 있고, 이 주소를 통해 이더리움을 보내고 받을 수 있다.

1. `contract Adoption {` 라인 다음에 다음 변수를 추가한다.

```
address[16] public acopters;
```
알아야 할 내용:

- 우리는 `adopters`라는 하나의 변수를 정의했다. 이 변수는 이더리움 주소로 이루어진 배열이다.
배열은 하나의 타입만 저장 가능하며, 저장할 변수의 갯수는 고정할 수도 아닐 수도 있다.
지금의 예시에서는 배열에 저장될 타입은 `address`타입이며, 배열의 길이는 16으로 고정했다.
- `adopters`변수는 public으로 선언되었다. Public 변수는 자동으로 getter 메소드를 가진다.
그러나 배열의 경우는 getter 메소드에서 키값이 요구되며, 하나의 값만 반환하도록 되어있다.
추후에 작성할 UI에 배열 전체가 필요하기 때문에, 배열 전체를 반환하는 메소드를 작성할 예정이다.
 
### 첫 함수: 애완동물 입양

이제 유저가 입양 요청을 할 수 있도록 만든다.

1. 방금 선언했던 변수 아래에 다음 내용의 함수를 작성해준다.

```
// Adopting a pet
function adopt(uint petId) public returns (uint) {
  require(petId >= 0 && petId <= 15);

  adopters[petId] = msg.sender;

  return petId;
}
```
알아야 할 내용:  

- Solidity에서는 함수의 파라미터와 리던 타입이 반드시 명시되어야 한다. 
이 예시에서는 `pedId`(int 타입)을 받고 int 타입을 리턴한다고 명시되었다.  
- 우선 `pedId`가 adopters 배열의 범위 안에 드는지 검사한다.
Solidity의 배열 인덱스는 0부터 시작하므로 ID값은 0~15 사이에 있어야 한다.
`require()`을 이용해 ID가 해당 범위 내에 있는지 체크한다.  
- ID가 범위 내에 있다면, 요청을 보낸 주소를 `adopters`배열에 집어넣는다.
이 함수를 호출한 개인 혹은 스마트 컨트렉트의 주소는 `msg.sender`로 불러올 수 있다.  
- 마지막으로, 요청과 함께 넘어온 `petId`를 확인의 의미로 반환한다.
 
### 두번째 함수: 입양자 검색

위에서 이야기 하였듯이 배열의 getter 함수는 주어진 키에 해당하는 하나의 값만 반환하도록 되어있다. 
우리의 UI는 모든 애완동물에 대한 분양 상태를 업데이트 해야하는데, 16번의 API 요청을 보내 것은 바람직하지 않다.
따라서 다음 해야할 일은 전체 배열을 반환하는 함수를 작성하는 것이다.

1. 스마트 컨트렉트의 adopt() 함수 다음에 getAdopters() 함수를 아래 내용과 같이 추가해준다.

```
// Retrieving the adopters
function getAdopters() public view returns (address[16]) {
  return adopters;
}
```
`adopters`는 이미 선언되어 있기 때문에, 단순히 그것을 반환하기만 하면 된다.
반환 타입을 `address[16]`으로 설정해주어야 하는 것을 명심해야 한다.(`adopters`를 선언했을 때 지정해준 타입을 반환하면 된다.)


## 스마트 컨트렉트 컴파일, 마이그레이션

지금까지 스마트 컨트렉트를 작성했으니 이제 컴파일 하고 마이그레이션을 할 차례이다.

Truffle은 Truffle develop이라고 불리는 개발자 콘솔이 내장되어있다.
그것을 이용해 컨트렉트 배포 테스트를 위한 개발용 블록체인을 생성할 수 있다.
또, Truffle 명령어를 콘솔에서 바로 실행할 수 있는 기능도 있다.
우리는 컨트렉트에 필요한 작업 대부분을 이 Truffle Develop을 이용해서 할 것이다.

### 컴파일

Solidity는 컴파일 언어이다. 
따라서 우리가 작성한 Solidity코드를 이더리움 가상 머신(EVM)에서 실행시키기 위해서는 컴파일하여 바이트코드로 만들어야 하낟.
사람이 읽을 수 있는 Solidity코드를 EVM이 이해할 수 있는 무언가로 번역한다고 생각하면 된다.

1. 터미널에서 dApp의 root 폴더로 이동하여 다음 내용을 입력한다.

```
truffle compile
```

아마 다음과 비슷한 내용을 볼 수 있을것이다.

```
Compiling ./contracts/Migrations.sol...
Compiling ./contracts/Adoption.sol...
Writing artifacts to ./build/contracts
```

### 마이그레이션

마이그레이션이란, 컨트렉트를 배포하고 변경하는 일련의 과정을 기술한 배포 스크립트이다.

우리의 컨트렉트를 성공적으로 컴파일했으니 이제 블록체인으로 마이그레이션 할 때가 되었다.
아마 `migrations/`폴더에 `1_initial_migrations.js`라는 자바스크립트 파일이 하나 들어있는 것을 확인할 수 있을 것이다.
이 파일은 `Migrations.sol` 컨트렉트의 배포를 관장하며, 차후에 스마트 컨트렉트가 변경되지 않았을 때 이중 배포되는 것을 방지하는 역할을 한다.

이제 우리가 작성한 `Adoption.sol`파일의 마이그레이션 스크립트를 작성해보자

1. `migrations/`폴더 내에 `2_deploy_contracts.js`라는 이름의 새로운 파일을 생성한다.
2. `2_deploy_contracts.js`파일을 열어 다음 내용을 입력한다.

        var Adoption = artifacts.require("Adoption");
        
        module.exports = function(deployer) {
          deployer.deploy(Adoption);
        };

3. 작성한 컨트렉트를 블록체인으로 배포하기 전에, 우리는 블록체인을 실행시켜야 한다. 
이번 튜토리얼에서는 이더리움 개발을 위한 개인 블록체인인 Ganache를 사용할 것이다.
Ganache에서는 컨트렉트 배포, 앱 개발, 테스트 등을 할 수 있다.
Ganache를 아직 다운로드 받지 않았다면, [다운로드](https://truffleframework.com/ganache)받고,
아이콘을 더블클릭하면 로컬의 7545번 포트에서 구동되는 블록체인이 생성된다.

    ![Ganache 실행화면](/files/blockchain_images/ganache-initial.png)

4. 다시 터미널로 돌아와 컨트렉트를 블록체인에 마이그레이션 해준다.

        truffle migrate

    아마 다음과 비슷한 결과물이 나올 것이다.

        Using network 'development'.
        
        Running migration: 1_initial_migration.js
          Deploying Migrations...
          ... 0xcc1a5aea7c0a8257ba3ae366b83af2d257d73a5772e84393b0576065bf24aedf
          Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0
        Saving successful migration to network...
          ... 0xd7bc86d31bee32fa3988f1c1eabce403a1b5d570340a3a9cdba53a472ee8c956
        Saving artifacts...
        Running migration: 2_deploy_contracts.js
          Deploying Adoption...
          ... 0x43b6a6888c90c38568d4f9ea494b9e2a22f55e506a8197938fb1bb6e5eaa5d34
          Adoption: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
        Saving successful migration to network...
          ... 0xf36163615f41ef7ed8f4a8f192149a0bf633fe1a2398ce001bf44c43dc7bdda0
        Saving artifacts...

    마이그레이션이 순서대로 실행된 후 배포된 컨트렉트의 주소도 확인할 수 있다. (터미널에 나타나는 주소는 다를 것이다.)

5. Ganache는 블록체인의 상태가 변경되면 알려준다. 블록체인의 현재 블록이 최초에 0이었던 것이 현재는 4로 바뀌었다.
그리고 첫 번째 계정의 이더리움이 100이었는데 현재는 조금 줄어든 것을 볼 수 있을 것이다.
마이그레이션 과정에서 트랜젝션 비용을 지불했기 때문이다.
트랜젝션 비용에 대해서는 나중에 이야기 할 것이다.
 
    ![마이그레이션 이후의 Ganache 화면](/files/blockchain_images/ganache-migrated.png)
 
당신은 이제 첫 스마트 컨트렉트를 작성하고 로컬에서 작동중인 블록체인에 배포까지 했다.
이제 스마트 컨트렉트가 우리가 원하는 방식대로 잘 작동하고 있는지 확인할 차례다.
 
## 스마트 컨트렉트 테스트
 
Truffle은 자바스크립트와 Solidity 모두 테스트 작성에 사용할 수 있기 때문에, 스마트 컨트렉트 테스트에 있어서 유연함을 갖췄다고 볼 수 있다.
이 튜토리얼에서는 Solidity를 사용해 테스트를 작성해보도록 하자.

1. `test/`폴더 안에 `TestAdoption.sol`파일을 생성한다.

2. `TestAdoption.sol`파일의 내용을 다음과 같이 수정한다.

        pragma solidity ^0.4.17;
        
        import "truffle/Assert.sol";
        import "truffle/DeployedAddresses.sol";
        import "../contracts/Adoption.sol";
        
        contract TestAdoption {
        Adoption adoption = Adoption(DeployedAddresses.Adoption());
        
        }

테스트는 3개 파일을 임포트하는 것으로 시작된다.
  
- `Assert.sol`: 테스트에 필요한 다양한 assertion을 제공한다.
테스트에서 assertion은 테스트의 통과/실패를 결정할 수 있도록 특정 값의 일치, 불일치 혹은 빈 값 등을 검사한다.
[Truffle에서 제공하는 전체 assertion은 여기에서 확인할 수 있다.](https://github.com/trufflesuite/truffle-core/blob/master/lib/testing/Assert.sol)
- `DeployedAddresses.sol`: 테스트가 진행될 때, Truffle은 블록체인에 테스트용 컨트렉트의 인스턴스를 배포할 것이다.
이 스마트 컨트렉트는 배포 된 컨트렉트의 주소를 받아오는 역할을 한다.
- `Adoption.sol`: 우리가 테스트할 스마트 컨트렉트
  
그리고 우리는 테스트 할 스마트 컨트렉트를 전역 변수(contract-wide variable)로 정의한다. 
스마트 컨트렉트의 주소를 받아오기 위해 `DeployedAddresses`를 호출한다.
  
### adopt() 메소드 테스트
  
`adopt()` 함수가 성공적으로 실행되었다면 주어진 `petId`를 다시 반환한다는 사실을 기억해야한다.
우리는 우리가 `adopt()`함수에 넘겨준 id와 반환될 id가 같아야 함을 확인하면 된다.

1. `TestAdoption.sol`파일의 `Adoption`선언부 다음에 다음 메소드를 입력한다.
 
        // Testing the adopt() function
        function testUserCanAdoptPet() public {
            uint returnedId = adoption.adopt(8);
            uint expected = 8;
            Assert.equal(returnedId, expected, "Adoption of pet ID 8 should be recorded.");
        }
        
    알아야 할 내용:
    - 우리가 선언했던 스마트 컨트렉트의 adopt 메소드를 파리미터 8과 함께 호출한다.
    - 우리가 받기를 기대하는 값 역시 8이다.
    - 마지막으로 우리가 실제로 받은값, 받기를 원하는 값, 실패시의 메세지를 `Assert.equal()`메소드에 인자로 넘겨준다.
    (실패시 표시될 메시지는 테스트에 실패했을 경우 콘솔창에 프린트된다.)
    
### 개별 애완동물 소유자 검색 테스트

앞서 public 변수는 자동으로 getter 메소드를 가진다고 설명했었다. 
그리고 앞서 했던 adoption 테스트 때문에 우리의 주소도 가져올 수 있게 되었다.
저장된 데이터는 우리가 테스트하는동안은 유지될 것이기 때문에 우리가 분양받았던 8번 애완동물의 adopter 주소는 우리의 주소가 될 것이다.

1. `TestAdoption.sol`에서 앞서 작성했던 테스트 아래에 다음 테스트를 작성한다.

        // Testing retrieval of a single pet's owner
        function testGetAdopterAddressByPetId() public {
          // Expected owner is this contract
          address expected = this;
          address adopter = adoption.adopters(8);
          Assert.equal(adopter, expected, "Owner of pet ID 8 should be recorded.");
        }
        
    우리가 작성중인 TestAdoption 컨트렉트가 트랜젝션을 전송할 것이기 때문에 `expected`에 들어갈 주소값은 `this`로 설정한다.
    `this`는 현재 컨트렉트의 주소를 반환하는 전역 변수이다.
    그리고 앞서 했던 테스트처럼 원하는 값이 반환되었는지 assert 해보면 된다. 
    
### 전체 애완동물 소유자 검색 테스트

배열의 getter 메소드가 단일값만을 반환하기 때문에, 전체 배열을 반환하는 메소드를 만들었었다.

1. `TestAdoption.sol`파일에서 앞서 작성했던 테스트 아래에 다음 메소드를 작성한다.

        // Testing retrieval of all pet owners
        function testGetAdopterAddressByPetIdInArray() public {
          // Expected owner is this contract
          address expected = this;
        
          // Store adopters in memory rather than contract's storage
          address[16] memory adopters = adoption.getAdopters();
        
          Assert.equal(adopters[8], expected, "Owner of pet ID 8 should be recorded.");
        }
        
    `adopters`에 메모리 속성이 있는 것이 보인다. 
    메모리 속성은 solidity에게 값을 컨트렉트 저장소가 아닌 메모리에 임시 저장하도록 지시하는 역할을 한다.
    `adopters`가 배열이고, 첫 adoption 테스트에서 우리가 8번 애완동물을 분양받았었기 때문에
    우리는 받아온 `adopters` 배열의 8번째 값이 우리의 주소와 일치하는지 확인해보면 된다. 
    

