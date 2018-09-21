---
layout: post
title: 'Solidity Study'
author: through.kim
date: 2018-09-17 13:58
tags: [blockchain, solidity, dapp]
image: '/files/covers/solidity.png'
comments: true
---

> 이더리움 dApp을 만들기 위해 필요한 스마트 컨트렉트를 작성하는 언어, 솔리디티(Solidity)에 대해 정리하며 공부해본다.

블록체인의 하나인 이더리움을 사용하는 애플리케이션인 dApp을 작성하기 위해서는 크게 두 부분을 작업하게 된다.

 - 스마트 컨트렉트: 보통 애플리케이션의 백엔드에 해당한다. 블록체인과 연결되는 부분
 - 프론트엔드: 보통 Web3.js 라이브러리를 사용해 작업하게 된다.
 
 이 포스팅에서는 블록체인과 직접 연결되어 사용되는 스마트 컨트렉트를 작성할 때 사용되는 언어인 솔리디티(Solidity)에 대해 공부해보고자 한다.
 다음 예시는 [이더리움 공식사이트](https://www.ethereum.org/)에서 제공하는 솔리디티 예문이다. 
 다음 코드를 통해 솔리디티로 작성된 컨트렉트의 전체적인 구조를 파악할 수 있다.
 
 ```
pragma solidity ^0.4.16;

contract TokenERC20 {
    // Public variables of the token
    string public name;
    string public symbol;
    uint8 public decimals = 18;
    // 18 decimals is the strongly suggested default, avoid changing it
    uint256 public totalSupply;

    // This creates an array with all balances
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;

    // This generates a public event on the blockchain that will notify clients
    event Transfer(address indexed from, address indexed to, uint256 value);
    
    // This generates a public event on the blockchain that will notify clients
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);

    // This notifies clients about the amount burnt
    event Burn(address indexed from, uint256 value);

    /**
     * Constructor function
     *
     * Initializes contract with initial supply tokens to the creator of the contract
     */
    function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
        balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
    }
    
    ...
    
    function buy() payable returns (uint amount){
        amount = msg.value / buyPrice;                    // calculates the amount
        _transfer(this, msg.sender, amount);
        return amount;
    }

    function sell(uint amount) returns (uint revenue){
        require(balanceOf[msg.sender] >= amount);         // checks if the sender has enough to sell
        balanceOf[this] += amount;                        // adds the amount to owner's balance
        balanceOf[msg.sender] -= amount;                  // subtracts the amount from seller's balance
        revenue = amount * sellPrice;
        msg.sender.transfer(revenue);                     // sends ether to the seller: it's important to do this last to prevent recursion attacks
        Transfer(msg.sender, this, amount);               // executes an event reflecting on the change
        return revenue;                                   // ends function and returns
    }
    
    ...
    
}
```

## Pragma
```
pragma solidity ^0.4.18;
```
첫 줄에 나와있는 pragma는 작성된 코드가 Solidity 0.4.18버전을 기반으로 작성되었으며, 그 이후 버전에서 정상 작동 할 수 있다는 것을 의미한다.

## State Variables
```
string public name;
uint8 public decimals = 18;
```
변수 선언부에 해당한다. 여타 언어와 크게 다르지 않다.

## mapping
```
mapping (address => uint256) public balanceOf;
mapping (address => mapping (address => uint256)) public allowance;
```
매핑은 키와 값의 쌍으로 이루어진 해시 테이블이다. 
예를들어 위 예문의 `balanceOf`는 address를 키로 가지고, uint256형의 값을 갖고 있는 것으로 해석할 수 있다.

## Functions
```
function TokenERC20(
    uint256 initialSupply,
    string tokenName,
    string tokenSymbol
) public {
    totalSupply = initialSupply * 10 ** uint256(decimals);  
    balanceOf[msg.sender] = totalSupply;                
    name = tokenName;                                   
    symbol = tokenSymbol;                               
}
...
    
function buy() payable returns (uint amount){
    amount = msg.value / buyPrice;                    // calculates the amount
    _transfer(this, msg.sender, amount);
    return amount;
}
```
함수 상단 예시의 `TokenERC20()`은 컨스트럭터(생성자)에 해당한다. 아래의 `buy()` 리턴값이 있는 함수의 모습이다.
Solidity의 함수는 public, private 등의 키워드를 사용해 함수의 가시성(Visibility)을 설정해줄 수 있다.

 - public: 컨트렉트 내,외부에 모두 공개되는 함수
 - private: 오직 컨트렉트에 계약된 계정에만 공개되는 함수, 함수명 앞에 언더바(_)를 붙이는 것이 관례이다.
 - external: 컨트렉트의 ABI에 포함되어 계약 외부에만 공개되는 함수 (register()와 같은 성격유형의 함수에 자주 사용됨)
 - internal: 해당 컨트렉트와, 컨트렉트를 상속한 하위 컨트렉트에 공개되는 함수
  
또 한 visibility 뒤에 `constant`, `view` 혹은 `pure`가 붙는 함수도 볼 수 있다. 키워드들의 의미는 다음과 같다.

 - constant: 해당 함수가 블록체인의 상태를 변경시키지 않을 것을 컴파일러에 알려주는 역할을 한다. 오직 컨트렉트의 상태변수를 조회하는 용도로만 사용된다는 것을 알려주는 것. Solidity 0.4.17버전 이후로 deprecate되었다.
 - view: 해당 함수가 블록체인의 상태를 변경시키지 않음을 의미한다. constant를 대체한다고 볼 수 있다. 
 - pure: 해당 함수가 블록체인의 상태를 변경시키지도 않고, 조회하지도 않음을 의미한다.

```
function buy() payable returns (uint amount){}
```
위 함수에서 payble은 이더리움을 전송하는 동작이 포함되어 있다면 반드시 붙어야하는 function modifier 이다.
solidity의 모든 function에서 이더리움 전송하는 동작이 포함되어있으면 Reject시키는 것이 기본 동작이며, 
이더리움을 전송하는 기능을 함수에 추가하고자 하면 반드시 payable을 붙여주어야 한다.

## Modifier
```
pragma solidity ^0.4.22;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(
            msg.sender == seller,
            "Only seller can call this."
        );
        _;
    }

    function abort() public view onlySeller { // Modifier usage
        // ...
    }
}
```
위의 예시 코드에서 Solidity의 modifier 형태를 확인할 수 있다. 
먼저 `onlySeller()`라는 modifier를 선언하고, 아래의 abort() 함수 선언과정에서 뒤에 붙여주고 있는 것을 확인할 수 있다.
abort() 함수가 호출되면, 함수를 실행하기 전에 modifier 에 정의된 조건으로 미리 검증을 한 뒤에 실행하는 것으로 이해할 수 있다.
 
## Events
```
pragma solidity ^0.4.21;

contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // Event

    function bid() public payable {
        // ...
        emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
```
상단 코드에서 Solidity Event의 유형을 볼 수 있다. 
Event는 컨트렉트 내부에서 상태변화가 일어났을 때 클라이언트에게 변화 내용을 전달 해주는 역할을 하는 것으로 이해할 수 있다.
위 코드에서는 어떤 사용자가 경매에서 입찰을 함으로써 최고입찰가가 변경되었을 경우 `HighestBidIncreased()` 이벤트를 호출한다.
이어 프론트엔드단에서 이벤트를 listen하여 최고입찰가와 입찰자가 변경되었음을 알려줄 수 있도록 한다. 