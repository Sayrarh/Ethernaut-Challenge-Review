### TASK

The goal of this level is to make the balance of the contract greater than zero.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.0;

contract Force {
  /*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/
}
```

### CODE EXPLOITS

Ways to send ether into a contract;

1. Sending Ether along with a function call to a payable function is the most common method. e.g having a "deposit()" function in a contract that accept user's funds.
2. A contract that implements a "receive()" function.
3. Payable fallback function are also used.
4. Using the selfdestruct function to send Ether to a contract, though this method has recently been depreciated. Look up [EIP-6049](https://eips.ethereum.org/EIPS/eip-6049) for more details.

The selfdestruct method is the only solution to the challenge presented in the contract.
Before diving into the solution, it's crucial to understand what the selfdestruct opcode in the Ethereum Virtual Machine (EVM) does. According to the Solidity Docs section on Deactivation and Self-destruction, the selfdestruct opcode allows for the removal of a contract's code from the blockchain and sends any remaining Ether stored at the contract address to a designated target recipient. Additionally, the storage and code of the contract are also removed from the state.
<br/>
However, it's important to keep in mind two important notes:

1. Even if a contract is deleted through selfdestruct, it remains a part of the blockchain's history and is likely retained by most Ethereum nodes. This means that using selfdestruct is not the same as deleting data from a hard disk.

   <br/>

But the only solution for the challenge is to use selfdestruct because no function exists in the Force contract.

### Solution

The solution requires creating a custom contract with the sole purpose of receiving Ether, executing selfdestruct, and sending the Ether to the Force contract address.

```solidity
contract ForceHack {
  constructor(address payable forceAddress) payable {
    selfdestruct(forceAddress);
  }
}
```

#### [Link to exploit](https://github.com/Sayrarh/Ethernaut-Challenge-with-Foundry/blob/master/test/Force.t.sol)
