# Sample Transaction Explanation Report

## Report Type

Transaction Explanation Report

## Case

Ethernaut Fallback — Ownership Takeover Through `receive()`

## Network

Sepolia

## Status

Educational lab example

---

## Important Notice

This is a sample report based on a controlled security lab.

It is not a recovery service, legal opinion, full smart contract audit, or guarantee of fund recovery.

The purpose of this report is to explain what happened in a transaction sequence, identify the relevant contract behavior, and describe the security issue in clear language.

---

## Executive Summary

The contract ownership was taken over through a three-step sequence.

First, the player made a small contribution to the contract.

Second, the player sent ETH directly to the contract. This triggered the contract’s `receive()` function.

Because the player had already contributed and sent ETH greater than zero, the contract changed the owner to the player.

Third, the player called `withdraw()` after becoming owner.

The main issue was unsafe ownership-changing logic inside the `receive()` function.

---

## Addresses Reviewed

### Player Address

```text
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

### Contract Address

```text
0xF8b84eC252BF76d1370943FcB04186Cce8e16daa
```

### Original Owner

```text
0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
```

---

## High-Level Timeline

```text
1. Player had no contribution recorded.
2. Player called contribute() with a small ETH amount.
3. Player contribution became greater than zero.
4. Player sent ETH directly to the contract.
5. Contract receive() function executed.
6. receive() changed owner to the player.
7. Player called withdraw().
8. Contract balance became zero.
```

---

## What Happened

The contract had a condition inside `receive()`.

The condition checked whether:

```text
msg.value > 0
```

and:

```text
contributions[msg.sender] > 0
```

After the player made a contribution, the second condition became true.

Then, when the player sent ETH directly to the contract, the first condition was also true.

As a result, the contract executed:

```text
owner = msg.sender
```

Because `msg.sender` was the player during the raw ETH transfer, the player became the contract owner.

---

## Key State Changes

### Before Contribution

```text
contributions[player] = 0
```

### After Contribution

```text
contributions[player] = 100000000000000 wei
```

### Before Raw ETH Transfer

```text
owner = 0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
```

### After Raw ETH Transfer

```text
owner = 0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

### After Withdraw

```text
contract balance = 0
```

---

## Root Cause

The root cause was broken access control.

The contract allowed ownership to change based on a weak condition:

```text
msg.value > 0 && contributions[msg.sender] > 0
```

This is not a safe way to control contract ownership.

A user who sends ETH and has a small contribution should not automatically receive owner privileges.

---

## Impact

The player became the contract owner.

After becoming owner, the player could access owner-only functionality.

In this case, the player called:

```text
withdraw()
```

The result was:

```text
contract balance = 0
```

---

## Root Cause vs Impact Path

It is important to separate the root cause from the impact.

### Root Cause

```text
Unsafe ownership change inside receive()
```

### Impact Path

```text
Owner-authorized withdraw()
```

`withdraw()` was not the original vulnerability.

`withdraw()` became dangerous only after ownership was incorrectly changed.

---

## Technical Explanation

The exploit path was:

```text
player -> contribute()
player -> raw ETH transfer
contract -> receive()
receive() -> owner = player
player -> withdraw()
```

At `receive()`:

```text
msg.sender = player
msg.value  > 0
contributions[msg.sender] > 0
```

Because both required conditions passed, the contract changed ownership.

This shows why receiving ETH should not silently assign privileged permissions.

---

## Risk Classification

### Vulnerability Category

```text
Broken access control
Unsafe privilege assignment
Unsafe receive/fallback logic
```

### Severity

```text
High
```

### Reason

The issue allowed a non-owner to become owner and withdraw contract funds.

---

## Recommended Fix

The ownership-changing logic should be removed from `receive()`.

A safer `receive()` function should only accept ETH or emit an event.

Example safer behavior:

```solidity
receive() external payable {
    // Accept ETH only
}
```

Ownership transfer should use explicit owner-only logic.

Example:

```solidity
function transferOwnership(address newOwner) external onlyOwner {
    owner = newOwner;
}
```

A safer production pattern would use a tested access control library such as OpenZeppelin `Ownable`.

---

## What This Report Does Not Claim

This report does not claim:

```text
- full smart contract audit coverage
- guaranteed security
- fund recovery
- legal responsibility assessment
- identification of a real-world attacker
- off-chain attribution
```

This report only explains the on-chain behavior visible from the transaction path and contract logic.

---

## Final Conclusion

The contract was vulnerable because its `receive()` function could change ownership.

The player first made a small contribution.

Then the player sent ETH directly to the contract.

That direct ETH transfer triggered `receive()`.

Because the player had a non-zero contribution and sent ETH greater than zero, the contract assigned ownership to the player.

After becoming owner, the player called `withdraw()` and emptied the contract balance.

The main lesson is that privileged state changes must not be hidden inside ETH-receiving logic.
