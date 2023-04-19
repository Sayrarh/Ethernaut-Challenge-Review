# TASK: The goal of this level is for you to steal all the funds from the contract.

Things that might help:

- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if (balances[msg.sender] >= _amount) {
      (bool result, ) = msg.sender.call{ value: _amount }("");
      if (result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```

## What is Reentrancy?

A reentrancy attack is a type of vulnerability in a smart contract that can allow an attacker to repeatedly call a function within the contract before the previous invocation of the function has completed. This can lead to unexpected behavior, such as the attacker being able to withdraw funds from the contract multiple times potentially draining the account.

## How to migitigate reentrancy

To mitigate reentrancy vulnerability, it is important to follow best practices when designing and implementing smart contracts. One common technique to prevent reentrancy attacks is to use the "checks-effects-interactions" pattern, which involves;

- performing all checks,
- updating contract state before
- interacting with other contracts or external functions.

## Reentrance Code Exploit

The Reentrance contract above is vulnerable to a reentrancy attack. This can occur in the withdraw function where the msg.sender is called before the state of the contract is updated.

```solidity
function withdraw(uint _amount) public {
  if (balances[msg.sender] >= _amount) {
    (bool result, ) = msg.sender.call{ value: _amount }("");
    if (result) {
      _amount;
    }
    balances[msg.sender] -= _amount;
  }
}
```

An attacker can create a separate contract that repeatedly calls the withdraw function before the first invocation completes, allowing the attacker to repeatedly drain funds from the contract.

### Solution to Hack the Contract

## Recommendation

In this modified withdraw function, it first checks that no withdrawal is currently on for the user by using the locked mapping. It then performs all necessary state updates before interacting with the external function msg.sender.call. Finally, it checks that the external function call was successful before completing the withdrawal and then update the locked mapping back to false.

```solidity
function withdraw(uint _amount) public {
  require(!locked[msg.sender], "Withdrawal already in progress");
  require(balances[msg.sender] >= _amount, "Insufficient balance");

  locked[msg.sender] = true;
  balances[msg.sender] -= _amount;
  (bool result, ) = msg.sender.call{ value: _amount }("");
  require(result, "Withdrawal failed");
  locked[msg.sender] = false;
}
```
