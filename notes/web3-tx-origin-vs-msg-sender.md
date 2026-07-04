# Web3 Note — `tx.origin` vs `msg.sender`

## Purpose

This note explains the difference between `tx.origin` and `msg.sender` in Solidity.

This concept is critical for:

- smart contract security
- exploit reproduction
- transaction analysis
- internal call tracing
- understanding phishing-style contract attacks
- reading Ethernaut Telephone-style vulnerabilities

---

## Simple Explanation

`tx.origin` is the wallet that started the whole transaction.

`msg.sender` is the direct caller of the function currently executing.

They are sometimes the same.

They are not always the same.

The difference becomes important when one contract calls another contract.

---

## Core Definitions

### `tx.origin`

`tx.origin` is the original external account that started the transaction.

It stays the same through the full call chain.

Example:

```text
Wallet -> Contract A -> Contract B
```

Inside both contracts:

```text
tx.origin = Wallet
```

---

### `msg.sender`

`msg.sender` is the immediate caller of the current function.

It can change at every step of the call chain.

Example:

```text
Wallet -> Contract A -> Contract B
```

Inside Contract A:

```text
msg.sender = Wallet
```

Inside Contract B:

```text
msg.sender = Contract A
```

---

## Key Difference

```text
tx.origin  = who started the whole transaction
msg.sender = who directly called the current function
```

This is the shortest useful version.

But for real exploit analysis, we need to track the exact call location.

---

## Example 1 — Direct Call

Call chain:

```text
Player wallet -> Target.changeOwner(player)
```

At `Target.changeOwner(player)`:

```text
tx.origin  = player wallet
msg.sender = player wallet
argument   = player
```

Here, `tx.origin` and `msg.sender` are the same.

There is no helper contract.

---

## Example 2 — Contract-Mediated Call

Call chain:

```text
Player wallet -> Helper.attack(player) -> Target.changeOwner(player)
```

At `Helper.attack(player)`:

```text
tx.origin  = player wallet
msg.sender = player wallet
argument   = player
```

At `Target.changeOwner(player)`:

```text
tx.origin  = player wallet
msg.sender = Helper contract
argument   = player
```

This is the dangerous case.

The original wallet is still the player.

But the direct caller is now the helper contract.

---

## Why This Matters

Some vulnerable contracts check:

```solidity
require(tx.origin != msg.sender);
```

or use `tx.origin` for authorization.

This is dangerous because a malicious or helper contract can sit between the user and the target contract.

The target contract may think:

```text
tx.origin = real wallet
msg.sender = contract
```

That difference can be abused if the contract logic depends on it incorrectly.

---

## Telephone Example

Ethernaut Telephone used this kind of logic:

```solidity
if (tx.origin != msg.sender) {
    owner = _owner;
}
```

The exploit used a helper contract.

Call chain:

```text
player wallet -> helper contract -> Telephone contract
```

At `Helper.attack(player)`:

```text
tx.origin  = player
msg.sender = player
_owner     = player
```

At `Telephone.changeOwner(player)`:

```text
tx.origin  = player
msg.sender = helper contract
_owner     = player
```

The condition passed:

```text
tx.origin != msg.sender
```

Because:

```text
player != helper contract
```

Then the contract executed:

```text
owner = player
```

Result:

```text
Telephone owner became player
```

---

## Important Mental Model

Always ask the question at the exact function location.

Bad question:

```text
Who calls attack?
```

Better question:

```text
At Helper.attack(player):
tx.origin = ?
msg.sender = ?
argument = ?
state change = ?
```

And then:

```text
At Telephone.changeOwner(player):
tx.origin = ?
msg.sender = ?
argument = ?
state change = ?
```

This avoids confusion.

The correct answer depends on where in the call chain we are standing.

---

## Call Chain Table

For this call chain:

```text
player wallet -> helper contract -> target contract
```

| Location | `tx.origin` | `msg.sender` |
|---|---|---|
| Helper contract | player wallet | player wallet |
| Target contract | player wallet | helper contract |

The original wallet stays the same.

The direct caller changes.

---

## Forensics Use

When reading transactions, the external transaction usually shows:

```text
From = wallet that started the transaction
To   = first contract called
```

The external `From` is usually the same account as `tx.origin`.

But internal calls can have different direct callers.

Example:

```text
External transaction:
From = player
To   = helper contract
```

Internal call:

```text
From = helper contract
To   = target contract
```

At the target contract:

```text
tx.origin  = player
msg.sender = helper contract
```

This is why internal transaction traces matter.

A normal Etherscan transaction page may show the external transaction, but the important security behavior often happens inside the internal call.

---

## Security Rule

Do not use `tx.origin` for authorization.

Unsafe pattern:

```solidity
require(tx.origin == owner);
```

Safer pattern:

```solidity
require(msg.sender == owner);
```

Even better, use a well-tested access control pattern such as OpenZeppelin `Ownable` or `AccessControl`.

---

## Why `tx.origin` Authorization Is Dangerous

Imagine a victim owns a contract.

The vulnerable contract checks:

```solidity
require(tx.origin == owner);
```

A malicious contract can trick the owner into calling it.

The call chain becomes:

```text
owner wallet -> malicious contract -> vulnerable contract
```

At the vulnerable contract:

```text
tx.origin  = owner wallet
msg.sender = malicious contract
```

If the vulnerable contract authorizes by `tx.origin`, the check may pass even though the direct caller is the malicious contract.

This is why `tx.origin` can create phishing-style attack surfaces.

---

## Correct Authorization Thinking

A privileged function should normally ask:

```text
Who is directly calling me?
```

That means checking:

```text
msg.sender
```

Not:

```text
tx.origin
```

The contract should care about the immediate caller because that is the account or contract actually interacting with it.

---

## Practical Drill 1

Call chain:

```text
Alice wallet -> Vault.deposit()
```

At `Vault.deposit()`:

```text
tx.origin  = Alice
msg.sender = Alice
```

Reason:

Alice directly called the Vault.

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

Reason:

Alice started the transaction, but Router directly called the Pool.

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
```

State change:

```text
owner = Player
```

Reason:

The vulnerable contract checks that the original wallet and direct caller are different.

---

## Common Beginner Mistakes

### Mistake 1

Thinking `msg.sender` always means the wallet.

Correction:

`msg.sender` means the direct caller.

The direct caller can be a wallet or a contract.

---

### Mistake 2

Thinking `tx.origin` changes during internal calls.

Correction:

`tx.origin` stays the same for the full transaction.

---

### Mistake 3

Reading only the external transaction.

Correction:

The external transaction is not enough when contracts call other contracts.

You must inspect internal calls, traces, emitted events, and state changes.

---

### Mistake 4

Thinking `tx.origin != msg.sender` is always bad by itself.

Correction:

The comparison is not automatically the whole bug.

The bug depends on what the contract does after the check.

In Telephone, the dangerous action was:

```text
owner = _owner
```

The comparison enabled an unsafe ownership change.

---

## Final Summary

`tx.origin` is the original wallet that started the whole transaction.

`msg.sender` is the direct caller of the current function.

In direct wallet-to-contract calls, they are usually the same.

In contract-to-contract calls, they are usually different.

For security analysis, always ask the question at a precise function location:

```text
At Contract.function(...):
tx.origin = ?
msg.sender = ?
argument = ?
state change = ?
```

The main security lesson is simple:

```text
Do not use tx.origin for authorization.
Use msg.sender-based access control instead.
```
