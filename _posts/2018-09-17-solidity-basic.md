---
layout: post
title: 'Solidity Study (작성중)'
author: through.kim
date: 2018-09-17 13:58
tags: [blockchain, solidity, dapp]
image: '/files/covers/solidity.png'
comments: true
---

> 이더리움 dApp을 만들기 위해 필요한 스마트 컨트렉트를 작성하는 언어, 솔리디티(Solidity)에 대해 정리하며 공부해본다.

블록체인의 하나인 이더리움을 사용하는 애플리케이션, dApp을 작성하기 위해서는 크게 두 부분을 작업하게 된다.

 - 스마트 컨트렉트: 보통 애플리케이션의 백엔드에 해당한다. 블록체인과 연결되는 부분
 - 프론트엔드: 보통 Web3.js 라이브러리를 사용해 작업하게 된다.
 
 블록체인과 직접 연결되어 사용되는 스마트 컨트렉트를 작성할 때 사용되는 언어인 솔리디티(Solidity)에 대해 공부해보고자 한다.
 
 다음 예시는 [이더리움 공식사이트](https://www.ethereum.org/)에서 제공하는 솔리디티 예문이다. 다음 코드를 통해 전체적인 구조를 파악하고자 한다.
 
 ```solidity
 
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
}
```

## Pragma
```solidity
pragma solidity ^0.4.18;
```
첫 줄에 나와있는 pragma는 작성된 코드가 Solidity 0.4.18버전을 기반으로 작성되었으며, 그 이후 버전에서 정상 작동 할 수 있다는 것을 의미한다.

## State Variables
```solidity
string public name;
uint8 public decimals = 18;
```
변수 선언부에 해당한다.

## mapping
```solidity
mapping (address => uint256) public balanceOf;
mapping (address => mapping (address => uint256)) public allowance;
```
매핑은 키와 값의 쌍으로 이루어진 해시 테이블이다.

## Functions
```solidity
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
```
함수, public, private등의 가시성(Visibility)을 설정해줄 수 있다.

  - public: 컨트렉트 내,외부에 모두 공개되는 함수
  - private: 오직 컨트렉트에 계약된 계정에만 공개되는 함수
  - external: 컨트렉트의 ABI에 포함되어 계약 외부에만 공개되는 함수 (register()와 같은 성격유형의 함수에 자주 사용됨)
  - internal: 해당 컨트렉트와, 컨트렉트를 상속한 하위 컨트렉트에 공개되는 함수
 