# Web3 Note — External Transactions vs Internal Calls

## Purpose

This note explains the difference between external transactions and internal calls.

This concept is critical for:

- reading Etherscan correctly
- understanding exploit paths
- reconstructing contract-to-contract calls
- explaining `tx.origin` vs `msg.sender`
- writing transaction reports
- avoiding wrong conclusions from incomplete transaction views

---

## Simple Explanation

An external transaction is the transaction started by a wallet.

An internal call is a call made from one contract to another contract during that same transaction.

A transaction can look simple from the outside but contain multiple internal contract calls.

This matters because the important security behavior often happens inside the internal calls.

---

## Core Definitions

### External Transaction

An external transaction starts from an externally owned account.

An externally owned account is usually a normal wallet controlled by a private key.

Example:

```text
player wallet -> helper contract
```

On Etherscan, the main transaction view usually shows:

```text
From = player wallet
To   = helper contract
```

This is the external transaction.

---

### Internal Call

An internal call happens when a contract calls another contract during the execution of the same transaction.

Example:

```text
helper contract -> target contract
```

This call is not a separate external wallet transaction.

It happens inside the original transaction.

---

## Basic Call Chain

Example:

```text
player wallet -> helper contract -> target contract
```

There is one external transaction:

```text
player wallet -> helper contract
```

There is also one internal call:

```text
helper contract -> target contract
```

Both are part of the same transaction execution.

---

## Why This Matters

A beginner may look only at the external transaction and think:

```text
The player only called the helper contract.
```

That is incomplete.

The full exploit path may be:

```text
player wallet -> helper contract -> vulnerable target contract
```

The vulnerable state change may happen inside the target contract, not inside the helper contract.

So the external transaction alone does not always show the full security story.

---

## External Transaction View

An external transaction usually tells you:

```text
who started the transaction
which contract was called first
how much ETH was sent
what input data was sent
whether the transaction succeeded or failed
```

Example:

```text
From = player
To   = helper contract
Value = 0 ETH
Status = Success
```

This is useful, but incomplete.

It does not automatically explain every contract action that happened inside the transaction.

---

## Internal Call View

Internal calls help reveal:

```text
which contracts were called during execution
which contract called which contract
whether ETH moved between contracts
which deeper contract function was reached
```

Example:

```text
Internal call:
From = helper contract
To   = Telephone contract
Call = changeOwner(player)
```

This shows the real vulnerable call.

---

## Telephone Example

In Ethernaut Telephone, the external transaction was:

```text
player wallet -> helper contract
```

The internal call was:

```text
helper contract -> Telephone.changeOwner(player)
```

At the external transaction level:

```text
From = player
To   = helper contract
```

At the internal call level:

```text
From = helper contract
To   = Telephone contract
```

At `Telephone.changeOwner(player)`:

```text
tx.origin  = player
msg.sender = helper contract
argument   = player
```

This is why the vulnerability worked.

The vulnerable contract saw that:

```text
tx.origin != msg.sender
```

Because:

```text
player != helper contract
```

Then it changed ownership.

---

## Fallback Example

In Ethernaut Fallback, the important transaction was a raw ETH transfer.

The external transaction was:

```text
player wallet -> Fallback contract
```

The calldata was empty.

The contract received ETH and executed:

```text
receive()
```

At `Fallback.receive()`:

```text
msg.sender = player
msg.value  > 0
contributions[msg.sender] > 0
state change = owner becomes player
```

This example is different from Telephone.

There does not need to be a helper contract.

The key issue is that an external raw ETH transfer triggered internal contract logic through `receive()`.

---

## Vault Example

In Ethernaut Vault, the important action was a direct function call:

```text
player wallet -> Vault.unlock(password)
```

The password was read from storage first.

Then the player called:

```text
unlock(bytes32 _password)
```

At `Vault.unlock(password)`:

```text
msg.sender = player
argument   = password read from storage slot 1
state change = locked becomes false
```

This example shows that not every exploit needs internal calls.

Some exploits are direct calls after reading public blockchain state.

---

## Common Transaction Patterns

### Pattern 1 — Direct Wallet-to-Contract Call

```text
wallet -> target contract
```

At the target contract:

```text
tx.origin  = wallet
msg.sender = wallet
```

Example:

```text
player -> Vault.unlock(password)
```

---

### Pattern 2 — Wallet-to-Helper-to-Target

```text
wallet -> helper contract -> target contract
```

At the helper contract:

```text
tx.origin  = wallet
msg.sender = wallet
```

At the target contract:

```text
tx.origin  = wallet
msg.sender = helper contract
```

Example:

```text
player -> Helper.attack(player) -> Telephone.changeOwner(player)
```

---

### Pattern 3 — Wallet-to-Router-to-Pool

```text
wallet -> router -> pool
```

At the router:

```text
tx.origin  = wallet
msg.sender = wallet
```

At the pool:

```text
tx.origin  = wallet
msg.sender = router
```

This pattern is common in DeFi.

The user does not call the pool directly.

The user calls a router, and the router calls the pool.

---

### Pattern 4 — Raw ETH Transfer to Contract

```text
wallet -> contract
```

With:

```text
data  = empty
value > 0
```

This can trigger:

```text
receive()
```

Example:

```text
player -> Fallback.receive()
```

This can be dangerous if `receive()` changes important state.

---

## Why Etherscan Can Be Misleading

The main Etherscan transaction page may make the transaction look simple.

Example:

```text
From = player
To   = helper contract
```

But the security-relevant action may be hidden deeper:

```text
helper contract -> target contract
target contract changes owner
```

So a proper investigation should not stop at the external transaction.

You should inspect:

```text
external transaction
input data
internal calls
logs/events
state changes
balance changes
contract source code
function arguments
```

---

## Investigation Checklist

When reading a transaction, ask:

```text
1. Who started the transaction?
2. What was the first contract called?
3. Was ETH sent?
4. Was calldata empty or non-empty?
5. Which function was called externally?
6. Were there internal contract calls?
7. Which contract made each internal call?
8. Which function changed state?
9. What changed before vs after?
10. Was the final impact caused by the first call or a deeper call?
```

This prevents shallow analysis.

---

## State Change Focus

A transaction report should focus on state changes, not only function names.

Weak analysis:

```text
The player called withdraw().
```

Better analysis:

```text
The player became owner during receive(), then called withdraw().
```

Even better:

```text
Root cause = unsafe ownership change in receive()
Impact path = owner-authorized withdraw()
Final state = contract balance became 0
```

This is the standard you need for public reports.

---

## How This Connects to `msg.sender`

Internal calls change `msg.sender`.

Example:

```text
player wallet -> helper contract -> target contract
```

At the target contract:

```text
msg.sender = helper contract
```

Not the player.

This is why internal calls matter for access control analysis.

A contract that checks `msg.sender` checks the direct caller.

A contract that checks `tx.origin` checks the original wallet.

Those are not the same in contract-to-contract call chains.

---

## How This Connects to Forensics

Forensics is not only about addresses.

It is about reconstructing what happened.

A useful transaction report should explain:

```text
who initiated the transaction
which contracts were involved
which calls happened internally
which state variables changed
which balances changed
which function caused the impact
what the root cause was
```

This turns raw blockchain data into a readable investigation.

---

## Practical Drill 1

Call chain:

```text
Alice wallet -> Token.transfer(Bob, 100)
```

At `Token.transfer()`:

```text
tx.origin  = Alice
msg.sender = Alice
argument 1 = Bob
argument 2 = 100
```

Transaction type:

```text
direct external transaction
```

---

## Practical Drill 2

Call chain:

```text
Alice wallet -> Router.swap() -> Pool.executeSwap()
```

At `Router.swap()`:

```text
tx.origin  = Alice
msg.sender = Alice
```

At `Pool.executeSwap()`:

```text
tx.origin  = Alice
msg.sender = Router
```

Transaction type:

```text
external transaction with internal call
```

---

## Practical Drill 3

Call chain:

```text
Player wallet -> Helper.attack(player) -> Telephone.changeOwner(player)
```

At `Helper.attack(player)`:

```text
tx.origin  = Player
msg.sender = Player
argument   = Player
```

At `Telephone.changeOwner(player)`:

```text
tx.origin  = Player
msg.sender = Helper
argument   = Player
state change = owner becomes Player
```

Transaction type:

```text
external transaction with internal exploit call
```

---

## Practical Drill 4

Call chain:

```text
Player wallet -> Fallback contract
```

With:

```text
data  = empty
value > 0
```

At `Fallback.receive()`:

```text
tx.origin  = Player
msg.sender = Player
msg.value  > 0
state change = owner becomes Player
```

Transaction type:

```text
raw ETH transfer triggering receive()
```

---

## Common Beginner Mistakes

### Mistake 1

Thinking the external transaction shows the whole story.

Correction:

The external transaction only shows the first layer.

Internal calls may contain the real exploit behavior.

---

### Mistake 2

Ignoring internal calls.

Correction:

Always check whether one contract called another contract.

This is essential for exploit reconstruction.

---

### Mistake 3

Thinking every state change appears clearly on the main transaction page.

Correction:

Some state changes require reading contract storage, events, traces, or before/after contract state.

---

### Mistake 4

Confusing internal calls with separate transactions.

Correction:

An internal call is not a separate wallet transaction.

It happens inside the original transaction execution.

---

### Mistake 5

Focusing only on the final impact.

Correction:

The final impact may be different from the root bug.

Example:

```text
withdraw() = impact path
receive() ownership change = root bug
```

---

## Final Summary

An external transaction is the wallet-started transaction.

An internal call is a contract-to-contract call that happens during that transaction.

The external transaction tells you the first layer.

The internal calls tell you what happened deeper.

For security analysis, always reconstruct the full path:

```text
wallet -> contract A -> contract B -> state change
```

The main investigation question is not only:

```text
What function was called?
```

The better question is:

```text
At the vulnerable function:
tx.origin = ?
msg.sender = ?
arguments = ?
state change = ?
impact = ?
```
