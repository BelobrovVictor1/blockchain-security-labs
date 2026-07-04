# Transaction Report 02 — Ethernaut Telephone Helper Exploit

## Summary

This report analyzes the transaction used to exploit the Ethernaut Telephone level.

The exploit used a helper contract to create a multi-contract call chain:

```text
player wallet -> helper contract -> Telephone contract
```

The player wallet signed a transaction to the helper contract. During execution, the helper contract made an internal call to the Telephone contract and called `changeOwner(player)`. This changed the Telephone contract owner to the player wallet.

## Transaction Overview

```text
Network: Sepolia Testnet

Transaction Hash:
0xc74808161c9b0b3ea36e201e8adc5a01476180b79a68019cbc75561a285e5a22

Status:
Success

External From:
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa

External To:
0x1fb2892468418b03638e006855178856fd2C2e95

Value:
0 ETH

External Function:
attack(address _winner)

External Argument:
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

## External Transaction

The external transaction was the transaction signed by the player wallet.

```text
player wallet -> helper.attack(player)
```

The transaction had `0 ETH` value, meaning no ETH was transferred. However, the transaction included calldata for the `attack(address)` function.

This is important because a transaction with `0 ETH` can still change contract state if it calls a state-changing function.

## Internal Call

During the external transaction, the helper contract made an internal contract call to the Telephone instance.

```text
helper contract -> Telephone.changeOwner(player)
```

Internal call details:

```text
Internal From:
0x1fb2892468418b03638e006855178856fd2C2e95

Internal To:
0x284de3cb...ae5664c6d

Internal Function:
changeOwner(address _account)

Internal Argument:
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

The internal call is the critical action because it is where the helper contract called the vulnerable Telephone contract.

## Caller Identity During Exploit

At the external transaction level:

```text
player wallet -> helper.attack(player)
```

Inside `Helper.attack(player)`:

```text
tx.origin = player
msg.sender = player
newOwner = player
```

During the internal call:

```text
helper -> Telephone.changeOwner(player)
```

Inside `Telephone.changeOwner(player)`:

```text
tx.origin = player
msg.sender = helper
_owner = player
```

Because `tx.origin` and `msg.sender` were different inside the Telephone contract, the vulnerable condition passed.

## State Change

The state-diff view showed that the Telephone owner storage changed.

```text
Before:
0x0000000000000000000000002c2307bb8824a0abbf2cc7d76d8e63374d2f8446

After:
0x000000000000000000000000926d6ea686964324290fd4987141b5ed8e59b7fa
```

Interpretation:

```text
Before owner:
0x2c2307bb8824a0abbf2cc7d76d8e63374d2f8446

After owner:
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa
```

This proves that the Telephone contract owner changed to the player wallet.

## Evidence Chain

```text
1. Player wallet signed a successful transaction to the helper contract.
2. The external transaction called attack(player).
3. The helper contract made an internal call to Telephone.changeOwner(player).
4. The Telephone contract updated its owner storage.
5. The final owner became the player wallet.
```

## Key Lesson

The external transaction only shows the first hop:

```text
player -> helper
```

The actual exploit effect happened through the internal call:

```text
helper -> Telephone
```

For investigation work, it is not enough to inspect only the external `From` and `To` fields. Internal calls and state changes must also be reviewed.

## Conclusion

The transaction successfully exploited the Telephone level by using a helper contract to create a call chain where `tx.origin` remained the player wallet while `msg.sender` inside the Telephone contract became the helper contract.

The internal call to `changeOwner(player)` and the state-diff owner change prove successful ownership takeover.
