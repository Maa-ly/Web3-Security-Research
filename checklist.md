# Web3 Security Review Checklist

## 1. **Access Control**

- [ ] **Ensure that sensitive functions are protected by proper access control mechanisms.**  
    - **Explanation:** Functions that alter the state of the contract, such as withdrawing tokens, setting parameters, or minting tokens, should only be accessible by authorized addresses (e.g., the contract owner or specific roles). Use modifiers like `onlyOwner`, `onlyAdmin`, or role-based access control (e.g., OpenZeppelin’s `AccessControl`) to enforce this.
    - **Example:** In the `BunniHookOracle.setParams()` function, only the owner should be able to modify parameters like `multiplier`, `secs_`, and `ago_`. This ensures that unauthorized users cannot tamper with important contract parameters.
      ```solidity
      function setParams(uint16 multiplier_, uint32 secs_, uint32 ago_, uint128 minPrice_) external onlyOwner {
          multiplier = multiplier_;
          secs = secs_;
          ago = ago_;
          minPrice = minPrice_;
          emit SetParams(multiplier_, secs_, ago_, minPrice_);
      }
      ```

## 2. **Reentrancy Protection**

- [ ] **Use the `nonReentrant` modifier to prevent reentrancy attacks.**  
    - **Explanation:** Reentrancy attacks occur when an external contract makes a call back into the calling contract, potentially allowing malicious behavior, such as draining the contract’s funds. The `nonReentrant` modifier from OpenZeppelin prevents this by locking the contract during an external call.
    - **Example:** In the `withdrawIncentive()` function in `MasterBunni`, the `nonReentrant` modifier should be applied to prevent a malicious contract from repeatedly calling the function to drain funds during withdrawal.
      ```solidity
      function withdrawIncentive(RushIncentiveParams[] calldata params, address incentiveToken, address recipient)
          external
          nonReentrant
          returns (uint256 totalWithdrawnAmount)
      ```

## 3. **Slippage Protection**

- [ ] **Ensure slippage protection is implemented when swapping tokens.**  
    - **Explanation:** Slippage occurs when the price of a token changes between the time a transaction is initiated and when it is executed. If slippage is not controlled, users might receive fewer tokens than expected. Implementing slippage protection helps mitigate this risk.
    - **Example:** In the `TokenMigrator.migrate()` function, the contract owner can change the `newTokenPerOldToken` rate right before the user interacts, potentially causing users to receive fewer tokens than they expect. To avoid this, the rate should be fixed for each transaction or locked during the migration process.
      ```solidity
      uint256 currentRate = newTokenPerOldToken;  // Capture the rate before user migration
      ```

## 4. **Gas Optimization**

- [ ] **Optimize gas usage and ensure efficient contract execution.**  
    - **Explanation:** Gas costs can significantly affect the usability and cost of interacting with a contract. Review the contract for expensive operations (e.g., loops over large data sets or multiple external calls). Optimize these operations to reduce gas consumption.
    - **Example:** In the `depositIncentive()` function, if the number of incentive tokens grows large, ensure that loops do not run indefinitely or require excessive gas. Consider using batch processing or limiting the scope of the operation.
      ```solidity
      // Avoid processing a large number of tokens in one go; batch processing can help
      ```

## 5. **Token Handling & Approvals**

- [ ] **Check that token allowances are managed properly using `safeApprove`.**  
    - **Explanation:** Incorrect token approvals can lead to issues like reverts or approval mismatches. Always set the approval to 0 before changing it to avoid reverts when dealing with tokens like USDT, which can revert on approval if it's not reset.
    - **Example:** In the `BunniExecutor` contract, when approving swap output tokens to the reactor, ensure that the approval is first reset to 0 before being set to a new value, especially for tokens like USDT that can revert on multiple approvals.
      ```solidity
      token.safeApprove(address(reactor), 0);  // Reset allowance to prevent reverts
      token.safeApprove(address(reactor), type(uint256).max);  // Approve new amount
      ```

## 6. **Oracle and External Data Handling**

- [ ] **Validate external data sources (oracles) and inputs.**  
    - **Explanation:** If your contract depends on external data (such as price feeds or random number generation), ensure that these inputs are correctly validated to prevent manipulation or errors. Always use reliable and tamper-resistant oracles.
    - **Example:** In the `BunniHookOracle._queryTwap()` function, ensure that rounding is handled correctly when dividing cumulative tick values. For example, negative values should be rounded down to ensure consistent and predictable results.
      ```solidity
      arithmeticMeanTick = int24(tickCumulativesDelta / int56(uint56(windowSize)));
      if (tickCumulativesDelta < 0 && (tickCumulativesDelta % windowSize != 0)) arithmeticMeanTick--;
      ```

## 7. **Underflow/Overflow Checks**

- [ ] **Ensure safe casting and prevent overflows/underflows.**  
    - **Explanation:** Ensure that numeric values do not exceed the maximum allowed by the type (e.g., casting `uint256` to `uint64` can result in overflow if not checked). Add validation to prevent overflows in time-based calculations.
    - **Example:** In `MasterBunni.incentivizeRecurPool()`, the sum of `block.timestamp + key.duration` could overflow when casting to `uint64` if it exceeds `type(uint64).max`. Add a check to ensure this doesn’t happen.
      ```solidity
      require(block.timestamp + key.duration <= type(uint64).max, "Duration exceeds uint64 max");
      state.periodFinish = uint64(block.timestamp + key.duration);
      ```

## 8. **Domain Separator and Permit2 Integration**

- [ ] **Ensure that domain separators are dynamically generated and accurate.**  
    - **Explanation:** In contracts that use EIP-712 signatures (e.g., `permit` functions), the domain separator must be unique and correct, especially in scenarios where the chain ID changes or during a hard fork.
    - **Example:** In the `BunniZapIn` contract, the domain separator should be queried dynamically, not cached, to ensure it is correct and consistent with the current chain.
      ```solidity
      permit2DomainSeparator = IPermit2(SafeTransferLib.PERMIT2).DOMAIN_SEPARATOR();  // Avoid caching
      ```

## 9. **Merkle Tree & Hashing**

- [ ] **Use double-hashing for Merkle tree leaves to avoid second preimage attacks.**  
    - **Explanation:** Merkle trees should use double-hashing to mitigate second preimage attacks, which could allow an attacker to find another message that hashes to the same value.
    - **Example:** In `VeAirdrop.claim()`, use double hashing for Merkle tree leaves to ensure that attackers cannot exploit single-hashing.
      ```solidity
      bytes32 leaf = keccak256(bytes.concat(keccak256(abi.encode(addr, amount))));
      ```

## 10. **Token Decimals and Compatibility**

- [ ] **Check for compatibility with token decimals.**  
    - **Explanation:** If your contract assumes specific token decimals (e.g., 18 decimals), you should validate that the token being interacted with matches the expected decimal value. Failure to do so may result in incorrect balances or unexpected behavior.
    - **Example:** The `OptionsToken` contract assumes 18 decimals for the token. Add a validation check to ensure that the tokens interacting with the contract have the correct number of decimals.
      ```solidity
      require(token.decimals() == 18, "Token must have 18 decimals");
      ```

## 11. **Parameter Validation**

- [ ] **Validate input parameters for reasonable ranges.**  
    - **Explanation:** Ensure that input parameters passed to functions (e.g., multipliers, durations, amounts) are within valid ranges to prevent potential misuse or unintended behavior.
    - **Example:** In the `BunniHookOracle` contract, validate that the `multiplier_` is less than `MULTIPLIER_DENOM` and that `secs_` and `ago_` are within reasonable limits to prevent unexpected contract behavior.
      ```solidity
      require(multiplier_ < MULTIPLIER_DENOM, "Multiplier too large");
      ```

## 12. **Address Validation**

- [ ] **Ensure that zero addresses are not used for critical variables.**  
    - **Explanation:** Using the zero address (`0x0`) as a recipient or token address can lock assets in the contract permanently. Always validate addresses to ensure they are not zero.
    - **Example:** In `MasterBunni.depositIncentive()`, ensure that the `recipient` address is not zero before processing a deposit to avoid locking tokens in the contract.
      ```solidity
      require(recipient != address(0), "Invalid recipient address");
      ```

## 13. **Fallback & Error Handling**

- [ ] **Ensure that contracts have appropriate error messages and revert conditions.**  
    - **Explanation:** Always provide clear and informative revert messages when a transaction fails. This helps developers and users understand why an action was rejected.
    - **Example:** In `depositIncentive()`, ensure that errors like invalid pool keys or incorrect incentive amounts are reported with meaningful messages.
      ```solidity
      require(isValidRushPoolKey(params[i].key), "Invalid rush pool key");
      ```

## 14. **Reward Distribution & Incentives**

- [ ] **Ensure that reward and incentive calculations account for fee-on-transfer tokens.**  
    - **Explanation:** Some tokens (e.g., tokens with fee-on-transfer functionality) deduct a fee when transferred, leading to discrepancies between the expected amount and the actual received amount. Track the actual amount received after fees are deducted.
    - **Example:** In `MasterBunni.depositIncentive()`, calculate the difference in token balance before and after the transfer to properly account for fee-on-transfer tokens.
      ```solidity
      uint256 receivedAmount = incentiveToken.balanceOf(address(this)) - previousBalance;
      rushPoolIncentiveDeposits[id][incentiveToken][recipient] += receivedAmount;
      ```

## 15. **Event Emission**

- [ ] **Ensure that state-changing functions emit events for transparency.**  
    - **Explanation:** Emitting events allows external applications and users to track significant changes in contract state, such as token transfers, parameter changes, or incentive deposits. Events also help in debugging and auditing.
    - **Example:** In `setParams()`, emit an event every time the parameters are changed, so the changes can be tracked on the blockchain.
      ```solidity
      emit SetParams(multiplier_, secs_, ago_, minPrice_);
      ```

## 16. **Contract Upgradeability**

- [ ] **Ensure proper contract upgradeability mechanisms are in place.**  
    - **Explanation:** When using proxy patterns for contract upgradeability (e.g., through OpenZeppelin’s `UUPS` or `TransparentUpgradeableProxy`), ensure that only authorized addresses can upgrade the contract. Check that the storage layout is properly maintained.
    - **Example:** Ensure that the upgrade path follows secure proxy patterns to avoid vulnerabilities like uninitialized storage slots or incorrect state after an upgrade.

## 17. **Timelocks and Delays**

- [ ] **Implement timelocks for critical functions.**  
    - **Explanation:** Critical changes (e.g., changing incentive rates, token migration rates) should not be immediate. Introduce a delay or timelock mechanism to give stakeholders time to react or audit changes before they are applied.
    - **Example:** If `setParams()` changes important contract parameters, include a timelock before applying the change.

## 18. **General Best Practices**

- [ ] **Follow industry standards for token implementation (e.g., ERC20, ERC721).**  
    - **Explanation:** Standard token implementations reduce the risk of incompatibilities with other contracts and platforms. Make sure your contract adheres to established token standards for interoperability.
    - **Example:** Use OpenZeppelin’s `ERC20` or `ERC721` implementations to standardize token behavior across platforms.

## 19. **Testing and Auditing**

- [ ] **Ensure the contract has undergone rigorous testing and third-party audits.**  
    - **Explanation:** Thorough testing, including unit tests, integration tests, and simulation of edge cases, is essential. Additionally, contracts should be audited by reputable security firms to identify potential vulnerabilities.
    - **Example:** Consider using frameworks like Truffle or Hardhat for automated testing.

## 20. **User Interface (UI) and Experience**

- [ ] **Ensure the front-end interacts securely and effectively with the smart contract.**  
    - **Explanation:** The front-end should validate user inputs (addresses, amounts, etc.) to prevent user errors before interacting with the contract. Provide clear feedback and notifications to the user about transaction success or failure.
    - **Example:** The UI should alert users when they are interacting with a contract that is about to make a risky state change or when slippage exceeds acceptable limits.

---
