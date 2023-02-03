### TASK: Claim ownership of the Delegate Contract

```solidity
/ SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```
#### CODE EXPLOITS
The caller of the function "pwn()" will become the owner of the contract. This alone is not significant for us because we don’t need to gain ownership of this contract, but just keep it in mind for what’s coming next.

```solidity
function pwn() public {
    owner = msg.sender;
}
```


The Delegation contract has a fallback function that is automatically executed when no other function matches the called function signature. The fallback function uses the 'delegatecall' keyword to delegate the incoming message call to the Delegate contract. 
<br/>

A delegate call is a mechanism that allows a contract to call another contract's functions as if they were its own. The delegate call is performed using the 'delegatecall' keyword and it uses the context of the calling contract, meaning that the called contract operates under the caller's address and balance. When a delegate call is performed, the called contract's code is executed in the context of the calling contract, and any changes to the state or balance of the contract are made to the calling contract, not the called contract. This makes delegate calls a powerful tool for code reuse and modularity, but also requires careful consideration of security and access control.
<br/>

```solidity
  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
```

If the delegate call is successful, the function returns the Delegation contract itself, effectively preserving its state and allowing it to continue executing code. This mechanism allows the Delegation contract to act as a wrapper for the Delegate contract, forwarding calls to it and preserving its state.

<br/>
By sending a transaction to the Delegation contract with the "pwn" function as the data payload, an attacker could potentially change the value of the owner variable in the Delegate contract to their own address, effectively taking ownership of the Delegate contract.

#### RECOMMENDATION

To prevent this, it's important to carefully manage the delegate contract and ensure that it has appropriate access controls in place.



Link to exploit: https://github.com/Sayrarh/Ethernaut-Challenge-with-Foundry/blob/master/test/Delegation.t.sol