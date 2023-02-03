### TASK
The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```
### Gas Optimization

```solidity
 uint public totalSupply;

 constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }
```
The total supply could have been declared as 'immutable' because is only initialized in the contract, and it is never updated.

### CODE VUNERABILITIES

1. Integer Overflow/Underflow: In the transfer function there is no protection against integer overflow/underflow issues when subtracting '_value' from 'balances[msg.sender]' and adding it to balances[_to].

```solidity
function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
```

2. Reentrancy: This contract is vulnerable to reentrancy attacks, which can allow an attacker to repeatedly call the transfer function before the first call is finished, potentially draining the sender's balance.


#### Recommendations

1. Integer Overflow/Underflow: To prevent integer overflow/underflow, you can use the SafeMath library.


2. Reentrancy: To prevent reentrancy attacks, one should modify the code to use a mutex mechanism, such as the one provided by the ReentrancyGuard library. Additionally, one can add a mutex to prevent reentrancy by checking if the transfer function is already executing in the same callstack before updating the balance.

Link to exploit : https://github.com/Sayrarh/Ethernaut-Challenge-with-Foundry/blob/master/src/Token.sol