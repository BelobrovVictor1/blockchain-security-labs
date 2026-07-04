# Ethernaut 01 — Fallback

## Level

Ethernaut Level 01: Fallback

## Status

Solved

## Network

Sepolia

## Instance

`0xF8b84eC252BF76d1370943FcB04186Cce8e16daa`

## Player

`0x926D6eA686964324290Fd4987141b5ED8e59b7Fa`

---

## One-Sentence Summary

The contract lets a user become owner by first making a small contribution and then sending raw ETH directly to the contract, which triggers `receive()` and changes ownership.

---

## Core Lesson

A fallback or receive function must not contain sensitive state-changing logic unless it is extremely carefully designed.

In this level, the dangerous behavior is not only that the contract accepts ETH.

The real issue is that receiving ETH can change the contract owner.

---

## Vulnerability Type

Unsafe ownership change through `receive()`.

---

## Relevant Contract Behavior

The contract has three important paths:

1. `contribute()`
2. `receive()`
3. `withdraw()`

### 1. `contribute()`

The player must first call `contribute()` with a small ETH amount.

This updates:

```text
contributions[player] > 0
```

In my solved instance:

```text
contribution before = 0
contribution after  = 100000000000000 wei
```

This contribution does not directly win the level.

It only prepares the condition needed for the next step.

---

### 2. Raw ETH transfer triggers `receive()`

After contributing, the player sends raw ETH directly to the contract.

This means the transaction does not call a named function like `contribute()` or `withdraw()`.

Instead, the contract receives ETH and executes `receive()`.

The dangerous condition is:

```text
msg.value > 0
AND
contributions[msg.sender] > 0
```

If both are true, the contract changes owner:

```text
owner = msg.sender
```

In my solved instance:

```text
owner before = 0x3c34A342b2aF5e885FcAA3800dB5B205fFfa3FfB
owner after  = 0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

This is the actual ownership takeover.

---

### 3. `withdraw()`

After becoming owner, the player can call `withdraw()`.

This drains the ETH balance from the contract.

In my solved instance:

```text
contract balance after withdraw = 0
```

Important distinction:

`withdraw()` is the impact path.

It is not the root bug.

The root bug is the unsafe ownership change inside `receive()`.

---

## Exploit Path

The exploit requires three steps:

```text
1. Call contribute() with a tiny amount of ETH
2. Send raw ETH directly to the contract
3. Call withdraw()
```

Expanded:

```text
player -> Fallback.contribute{value: tiny_amount}()
player -> raw ETH transfer -> Fallback.receive()
Fallback.receive() -> owner = player
player -> Fallback.withdraw()
```

---

## Call Context Breakdown

### Step 1 — `contribute()`

At `Fallback.contribute()`:

```text
msg.sender = player
msg.value  = tiny contribution amount
state change = contributions[player] increases
```

Result:

```text
contributions[player] > 0
```

---

### Step 2 — raw ETH transfer

At the external transaction:

```text
from  = player
to    = Fallback instance
data  = empty
value = ETH amount greater than 0
```

Because the transaction has empty calldata and ETH value, `receive()` is triggered.

At `Fallback.receive()`:

```text
msg.sender = player
msg.value  > 0
contributions[msg.sender] > 0
state change = owner becomes player
```

Result:

```text
owner = player
```

---

### Step 3 — `withdraw()`

At `Fallback.withdraw()`:

```text
msg.sender = player
owner      = player
state change = contract balance is withdrawn
```

Result:

```text
contract balance = 0
```

---

## Why This Works

The contract treats a previous small contribution as enough trust to allow ownership transfer through `receive()`.

That is unsafe because sending ETH to a contract is a very common action.

A receive function should not silently grant privileged roles.

The attacker does not need to break cryptography, steal a private key, or bypass access control directly.

The attacker only needs to satisfy the contract’s weak ownership-change condition.

---

## Root Cause

The root cause is broken authorization logic.

The contract allows ownership to change based on:

```text
msg.value > 0
AND
contributions[msg.sender] > 0
```

This is not a valid ownership-control mechanism.

Ownership should not be granted just because an address contributed ETH and later sent ETH directly.

---

## Impact

After ownership changes, the attacker can call owner-only functionality.

In this level, the impact is:

```text
attacker becomes owner
attacker calls withdraw()
contract balance becomes 0
```

In a real contract, the same pattern could lead to:

- unauthorized withdrawals
- admin takeover
- configuration changes
- loss of funds
- loss of protocol control

---

## Fix

The ownership change should be removed from `receive()`.

A safer design would use explicit ownership transfer logic, such as:

```text
onlyOwner transferOwnership(newOwner)
```

or a two-step ownership transfer flow:

```text
current owner nominates new owner
new owner accepts ownership
```

The receive function should only accept ETH or emit an event.

It should not assign privileged roles.

---

## Security Takeaways

1. `receive()` can execute logic when ETH is sent with empty calldata.
2. Raw ETH transfers can trigger important contract behavior.
3. Small prior interaction with a contract should not imply admin trust.
4. Ownership transfer must use explicit authorization.
5. `withdraw()` was only the final impact; the real bug was ownership takeover inside `receive()`.

---

## Final Explanation

The exploit works because the player first becomes a recorded contributor.

Then the player sends ETH directly to the contract, triggering `receive()`.

Since the player has a contribution greater than zero and sends ETH greater than zero, the contract sets:

```text
owner = msg.sender
```

Because `msg.sender` is the player during the raw ETH transfer, the player becomes owner.

After that, the player can call `withdraw()` and empty the contract balance.

The critical mistake is putting ownership-changing logic inside `receive()`.
