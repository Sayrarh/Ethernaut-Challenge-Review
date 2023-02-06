### TASK: Unlock the vault to pass the level!
The goal for this challenge is to be able to guess the Vault secret password and unlock it.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

All data is stored publicly and permanently on the blockchain. Everything can be seen even if you declare a variable as 'private' or 'internal'. 

<br/>

#### Declaring a contract as 'private' or 'internal'

In Solidity, the 'private' and 'internal' state visibility determine the access restrictions for state variables and functions within a contract.
<br/>
1. Private state variables and function can only be accessed within the contract itself and are not accessible from external contracts.

2. Internal state variables and functions have the same visibility restrictions as private state variables and functions, but they are also accessible from contracts that inherit from the current contract.
<br/>

Note that while these visibility modifiers provide some level of access control within a contract and its inheritances, they do not provide any security guarantees for the data stored on the blockchain. As all data on the blockchain is publicly accessible, it is still possible for malicious actors to access the data.
<br/>

So it is important to be mindful of the security implications when storing sensitive information on the blockchain. Instead of storing sensitive information directly on the blockchain, it is recommended to store only encrypted or hashed values, or to store sensitive information off-chain and store only references to it on the blockchain.


### CODE VUNERABLITIES

1. The contract contains a security vulnerability because the password is stored as a private state variable, which can be read by malicious contract callers by reading the slot where the value is stored.
The password should be hashed and encrypted before being stored in the contract to ensure that it cannot be read by unauthorized parties, or stored off-chain if it's so confidential.

```solidity
 bytes32 private password;
```

2. The unlock function does not have any measures to prevent an attacker from repeatedly guessing the password, which could potentially lead to a denial-of-service attack on the contract.

```solidity
function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
```

### UNLOCK THE VAULT
What you should know about storage slot: 
- Each storage slot will use 32 bytes (word size). For each variable, a size in bytes is determined according to its type.

- The first item in a storage slot is stored lower-order aligned.

- Value types use only as many bytes as are necessary to store them.

- If a value type does not fit the remaining part of a storage slot, it is stored in the next storage slot.

- Structs and array data always start a new slot and their items are packed tightly according to these rules.

- Items following struct or array data always start a new storage slot.

 
##### Constant and Immutable Variables

The compiler does not reserve a storage slot for both 'constant' and 'immutable' variables in Solidity as are known as "compile-time constants" and are replaced with their values at compile-time(for constant variable) and the value for an immutable variable is passed at construction time, before the contract is deployed to the blockchain.
<br/>
As a result, these variables do not require a storage slot and are not stored on the blockchain. This can result in a lower gas cost for contract execution, as there is no need to allocate storage for these variables.
<br/>
 
bool 'locked'  will be at slot 0. The bool type takes 1 byte of storage, but because the next state variable takes the whole 32 bytes slot, we cannot pack locked with password.
<br/>
bytes32 password will take slot1. The bytes32 take the whole 32 bytes word.

##### Solution
On the console go through the following steps:


```javascript
//Get the contract address
await contract.address
// Read the storage at the contract's address, in slot 1 (password)
password = web3.eth.getStorageAt(vaultContractAddress, storageslot, function (err, result){password= result})
console.log(`The password is ${password}`);
// Display the password as text
// "A very strong secret password :)"
console.log(`In text: ${web3.toAscii(password)}`);
//unlock the vault with the password
await contract.unlock(password)
//check the locked status
await contract.locked();//false
```