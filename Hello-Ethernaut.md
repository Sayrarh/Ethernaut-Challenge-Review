### TASK
The level walks one through the basics of how to play the game. <br/>

Follow the instructions carefully and explore using your browser console. 
It might get tricky when you "await contract.infoNum()", you might not know what next to do and get confused. <br/>
Just simply check the contract's ABI by typing "contract" to see the functions included in the contract so you can get better information on what to do next.  <br/>
I then called the  "info42" and followed the instruction afterwards. <br/>

For you to submit the challenge you have to know the password to authenticate. Type the "contract" in youur console again, check the ABI, you will see the password function, now call the password function in your console to get the password and complete the process. <br/>

Below is the code snippet showed after cracking the 'Hello Ethernaut' challenge. <br/>


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {

  string public password;
  uint8 public infoNum = 42;
  string public theMethodName = 'The method name is method7123949.';
  bool private cleared = false;

  // constructor
  constructor(string memory _password) {
    password = _password;
  }

  function info() public pure returns (string memory) {
    return 'You will find what you need in info1().';
  }

  function info1() public pure returns (string memory) {
    return 'Try info2(), but with "hello" as a parameter.';
  }

  function info2(string memory param) public pure returns (string memory) {
    if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
      return 'The property infoNum holds the number of the next info method to call.';
    }
    return 'Wrong parameter.';
  }

  function info42() public pure returns (string memory) {
    return 'theMethodName is the name of the next method.';
  }

  function method7123949() public pure returns (string memory) {
    return 'If you know the password, submit it to authenticate().';
  }

  function authenticate(string memory passkey) public {
    if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
      cleared = true;
    }
  }

  function getCleared() public view returns (bool) {
    return cleared;
  }
}

```

#### Informational Exploit
The password variable is public and can be accessed by anyone. The authenticate function compares the passed argument to the password variable using a Keccak256 hash, and sets the "cleared" variable to true if they match. 
It is not advisable to make the password public if it's meant to be private because it can access by anyone. 