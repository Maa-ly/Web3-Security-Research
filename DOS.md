### Denial Of Service (DOS)

**Decription**
DDos  exploit is where an attacker disrupts the normal operation of a decentralized application (DApp), smart contract, or blockchain network. The goal of a DoS attack is to make the service unavailable to users or to prevent the smart contract from executing certain functions.

**Types of DDos attacks**
At a broader classification, types of DDoS attacks can be categorized as:

1. Application Layer Attacks
These target vulnerabilities in the application logic or functionality to disrupt service.

- Gas Limit Exploitation:
Exploits application design flaws, such as unbounded loops, to cause transactions to exceed gas limits, preventing execution.
- Griefing Attack:
Manipulates application functionality to make operations more costly or unusable for legitimate users, e.g., spamming invalid inputs.

<details>
<summary> Example Vulnerability </summary>
A smart contract iterates through a dynamically growing list of users, which could exceed the gas limit if too many entries are added.

```javascript
// Vulnerable Contract
pragma solidity ^0.8.0;

contract GasLimitExploit {
    address[] public users;

    function addUser() public {
        users.push(msg.sender);
    }

    function payout() public {
        for (uint i = 0; i < users.length; i++) {
            payable(users[i]).transfer(1 ether);
        }
    }

    // Receive Ether
    receive() external payable {}
}

```
</details>

### Attack 
The attacker adds thousands of entries to the users array, causing the payout function to fail due to excessive gas usage.

<details>
<summary>Mitigation</summary>
Avoid unbounded loops and introduce a pull payment mechanism to distribute funds.

```javascript
// Mitigated Contract
pragma solidity ^0.8.0;

contract MitigatedGasLimit {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        uint amount = balances[msg.sender];
        balances[msg.sender] = 0; // Update state before external call
        payable(msg.sender).transfer(amount);
    }
}

```
</details>




2. Protocol Attacks
These exploit weaknesses in the underlying protocol or blockchain mechanisms.

- Storage Manipulation:
Overloads or manipulates the smart contract's state storage to cause expensive operations, out-of-gas errors, or unresponsiveness.

- Lockout Attack:
Prevents the use of specific smart contract functionalities by exploiting protocol-level behavior, e.g., sending invalid data or locking critical operations.


### Protocol Attacks , Storage Manipulation
Vulnerable Code
The contract charges gas based on the number of storage slots accessed. An attacker can inflate the array to manipulate gas usage.

<details>
<summary>Vulnerable Code</summary>

```javascript
// Vulnerable Contract
pragma solidity ^0.8.0;

contract StorageExploit {
    uint[] public data;

    function addData(uint _value) public {
        data.push(_value);
    }

    function expensiveOperation() public {
        for (uint i = 0; i < data.length; i++) {
            data[i] = data[i] + 1; // Expensive operation on large storage
        }
    }
}

```
</details>


</details>

### Attack 

The attacker fills the data array with large amounts of data, making the expensiveOperation function consume excessive gas or fail.

<details>
<summary>Mitigation</summary>
Use batch processing to handle large arrays incrementally.

```javascript
// Mitigated Contract
pragma solidity ^0.8.0;

contract MitigatedStorage {
    uint[] public data;

    function addData(uint _value) public {
        data.push(_value);
    }

    function processBatch(uint start, uint end) public {
        require(end <= data.length, "Invalid range");
        for (uint i = start; i < end; i++) {
            data[i] = data[i] + 1;
        }
    }
}

```
</details>


1. Volumetric Attacks
These involve overwhelming the network or blockchain with a high volume of transactions or data to degrade performance or accessibility.

- Block Filling:
Floods the blockchain with high-gas transactions to fill block space, delaying or preventing other users' transactions.

</details>

### Volumetric Attacks, Block filling
The attacker floods the blockchain with transactions targeting a function that accepts arbitrary data, consuming block gas limits.

<details>
<summary>Vulnerable Code</summary>


```javascript
// Vulnerable Contract
pragma solidity ^0.8.0;

contract BlockFillingExploit {
    function spam(uint _value) public {
        for (uint i = 0; i < _value; i++) {
            // No meaningful operation
        }
    }
}

```
</details>

</details>

### Attack 
An attacker repeatedly calls spam with high _value, filling up blocks and delaying other transactions.

<details>
<summary>Mitigation</summary>

Introduce rate limiting or restrict the size of user-provided inputs.

```javascript
// Mitigated Contract
pragma solidity ^0.8.0;

contract MitigatedBlockFilling {
    uint public lastCallTime;

    modifier rateLimit() {
        require(block.timestamp >= lastCallTime + 1 minutes, "Rate limit exceeded");
        _;
        lastCallTime = block.timestamp;
    }

    function safeSpam(uint _value) public rateLimit {
        require(_value <= 100, "Input exceeds limit");
        for (uint i = 0; i < _value; i++) {
            // Meaningful operation
        }
    }
}

```
</details>

**Resources to Read more**
https://medium.com/@kavib/ddos-distributed-denial-of-service-types-signs-of-attack-consequences-mitigation-c3b8a7bf81d5

https://medium.com/@kavib/ddos-distributed-denial-of-service-types-signs-of-attack-consequences-mitigation-c3b8a7bf81d5

https://www.isecurdata.com/ddos-mitigation-strategies-infographic/
