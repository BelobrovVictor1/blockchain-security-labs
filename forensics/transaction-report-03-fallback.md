# Transaction Report 03 — Ethernaut Fallback

## Case

Ethernaut 01 — Fallback ownership takeover and withdrawal

## Status

Solved

## Network

Sepolia

## Report Type

Transaction reconstruction and exploit-path analysis

---

## Addresses

### Player

```text
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

### Fallback Instance

```text
0xF8b84eC252BF76d1370943FcB04186Cce8e16daa
```

### Initial Owner

```text
0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
```

---

## Executive Summary

The player took ownership of the Fallback contract by first making a small contribution and then sending ETH directly to the contract.

The direct ETH transfer triggered the contract’s `receive()` function.

Because the player had already contributed and sent ETH greater than zero, the contract changed ownership to the player.

After becoming owner, the player called `withdraw()` and reduced the contract balance to zero.

The root issue was unsafe ownership-changing logic inside `receive()`.

---

## Transaction Hashes

### Transaction 1 — Contribution

```text
TODO: add contribute() transaction hash
```

### Transaction 2 — Raw ETH Transfer

```text
TODO: add raw ETH transfer transaction hash
```

### Transaction 3 — Withdraw

```text
TODO: add withdraw() transaction hash
```

These hashes must be added later from Etherscan or the wallet activity history.

Do not guess them.

---

## Initial State

Before the exploit path, the player had no contribution recorded.

```text
contributions[player] = 0
```

The contract balance was also zero in the checked state.

```text
contract balance = 0
```

The owner was:

```text
owner = 0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
```

The player was not the owner.

---

## Step 1 — Contribution Transaction

The player called `contribute()` with a tiny ETH amount.

### Transaction Direction

```text
from = player
to   = Fallback instance
call = contribute()
```

### State Before

```text
contributions[player] = 0
```

### State After

```text
contributions[player] = 100000000000000 wei
```

### Meaning

This transaction did not directly change ownership.

Its purpose was to satisfy one of the conditions inside `receive()`:

```text
contributions[msg.sender] > 0
```

This prepared the account for the ownership takeover.

---

## Step 2 — Raw ETH Transfer

The player then sent ETH directly to the Fallback contract.

This was not a normal named function call.

It was a raw ETH transfer with empty calldata.

### Transaction Direction

```text
from  = player
to    = Fallback instance
data  = empty
value = greater than 0
```

Because calldata was empty and ETH was sent, the contract executed `receive()`.

---

## `receive()` Condition

The relevant condition was:

```text
msg.value > 0 && contributions[msg.sender] > 0
```

For the player, both checks passed.

### At `Fallback.receive()`

```text
msg.sender = 0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
msg.value  > 0
contributions[msg.sender] = 100000000000000 wei
```

Therefore, the contract executed:

```text
owner = msg.sender
```

---

## Ownership Change

### Before Raw ETH Transfer

```text
owner = 0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
```

### After Raw ETH Transfer

```text
owner = 0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

The player became the owner.

This was the critical state change.

---

## Step 3 — Withdraw Transaction

After becoming owner, the player called `withdraw()`.

### Transaction Direction

```text
from = player
to   = Fallback instance
call = withdraw()
```

### Authorization Context

At this point:

```text
msg.sender = player
owner      = player
```

The owner-only check passed.

The contract allowed the withdrawal.

---

## Final State

After `withdraw()`:

```text
contract balance = 0
```

The exploit path was complete.

---

## Exploit Timeline

```text
1. Player contribution is 0
2. Player calls contribute() with tiny ETH amount
3. contributions[player] becomes 100000000000000 wei
4. Player sends raw ETH directly to the contract
5. Contract executes receive()
6. receive() checks msg.value > 0
7. receive() checks contributions[msg.sender] > 0
8. Both checks pass
9. Contract sets owner = player
10. Player calls withdraw()
11. Contract balance becomes 0
```

---

## Root Cause

The root cause was broken authorization logic.

The contract allowed ownership to change inside `receive()` based only on:

```text
msg.value > 0
```

and:

```text
contributions[msg.sender] > 0
```

This is not a safe ownership-control mechanism.

A user who makes a tiny contribution should not gain the ability to become contract owner by sending ETH directly to the contract.

---

## Impact

The impact was full ownership takeover.

Once the player became owner, the player could call privileged functionality.

In this level, the privileged function was:

```text
withdraw()
```

The practical result was:

```text
contract balance = 0
```

---

## Why `withdraw()` Was Not the Root Bug

`withdraw()` was the final impact path.

It was not the root vulnerability.

The real vulnerability happened earlier, when `receive()` changed the owner.

A correct investigation should separate:

```text
root cause = unsafe ownership change in receive()
impact path = withdraw()
```

This distinction matters in real reports because fixing only the withdrawal function would not address the broken ownership-transfer logic.

---

## Security Classification

### Vulnerability Category

```text
Broken access control
Unsafe privilege assignment
Unsafe receive/fallback logic
```

### Severity in This Level

```text
High
```

### Reason

The bug allowed a non-owner to become owner and withdraw contract funds.

---

## Recommended Fix

Remove ownership-changing logic from `receive()`.

The receive function should not grant privileged roles.

A safer receive function would only accept ETH or emit an event.

Example safer behavior:

```text
receive() external payable {
    // accept ETH only
}
```

Ownership transfer should use explicit authorization.

Example safer ownership model:

```text
onlyOwner transferOwnership(newOwner)
```

or a two-step process:

```text
1. current owner nominates new owner
2. nominated owner accepts ownership
```

---

## Evidence Completeness

Current evidence is enough to explain the exploit path and state changes.

However, this report is not complete as a transaction report until the three transaction hashes are added:

```text
1. contribute() transaction hash
2. raw ETH transfer transaction hash
3. withdraw() transaction hash
```

Those hashes should be retrieved from Sepolia Etherscan or wallet transaction history.

---

## Final Conclusion

The Fallback level was solved by abusing unsafe logic inside `receive()`.

The player first made a small contribution, which created a non-zero contribution record.

Then the player sent ETH directly to the contract.

That raw ETH transfer triggered `receive()`.

Because `msg.value > 0` and `contributions[msg.sender] > 0`, the contract set:

```text
owner = msg.sender
```

Since `msg.sender` was the player, the player became owner.

After ownership changed, the player called `withdraw()` and reduced the contract balance to zero.

The main lesson is that receiving ETH must not silently grant privileged permissions.
