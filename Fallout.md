### TASK : Claim ownership of the contract code below

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}

```

### CODE VUNERABILITIES

In this contract, an address can claim ownership of the contract by being the first one to send ether to it during its creation. The visibility of the variable "owner" is "public payable", meaning that it is publicly readable and writable by a payable address. To claim ownership, the msg.sender needs to be equal to the value of the variable owner.

<br/>
<b>NOTE:<b>
Another way to define a constructor for a contract was to use the same name for the constructor function as the contract itself. This could lead to issues if the developer made a typo while writing the constructor's name, as the function would not be recognized as a constructor and the contract would not be initialized at the time of creation. This could result in unexpected behavior or errors in the contract's functionality. 

```solidity
 /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```

In the Fallout contract, notice that the contract’s name is 'Fallout' but the constructor function is called 'Fal1out'. The developer used number '1' instead of letter 'l'. 


As a result of this typo, the owner variable is never initialized and will be set to the default value of address(0) at deployment time. Run this command to see the current owner;

```
 await contract.owner()

```

This means that the owner variable is not set to the address that deploys the contract and thus, the onlyOwner modifier will always fail, allowing anyone to call the collectAllocations() function and drain the contract's balance. <br/>
Also, the allocations mapping will also not be initialized, as it is also done in the constructor function. This vulnerability allows any user to claim ownership of the contract and drain its balance by calling the collectAllocations() function. 

Call function Fal1out or run this command:

```
  await contract.Fal1out()
```

Now check the current contract owner.