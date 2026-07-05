# Web3 Note — Private Storage Is Not Secret

## Purpose

This note explains why `private` variables in Solidity are not actually secret.

This concept is critical for:

- smart contract security
- storage analysis
- Etherscan investigation
- exploit reproduction
- reading contract state
- understanding Ethernaut Vault-style vulnerabilities

---

## Simple Explanation

In Solidity, `private` does not mean hidden from the blockchain.

It only means other Solidity contracts cannot directly access the variable by name.

The raw storage data is still stored on-chain.

Anyone can read it if they know where to look.

---

## Core Rule

```text
private != secret
```

A better mental model:

```text
private = not directly accessible from other Solidity contracts
secret = not publicly readable
```

On a public blockchain, contract storage is public.

So if sensitive data is stored directly in contract storage, it should be treated as exposed.

---

## Solidity `private`

Example:

```solidity
bytes32 private password;
```

This prevents another Solidity contract from doing something like:

```solidity
target.password()
```

But it does not prevent people from reading the raw storage slot.

The data still exists on-chain.

---

## Blockchain Storage Is Public

Ethereum contracts store data in storage slots.

Each slot is 32 bytes.

The storage can be read with tools such as:

```text
getStorageAt(contractAddress, slotNumber)
```

For example:

```text
getStorageAt(instance, 0)
getStorageAt(instance, 1)
```

If a password is stored in slot 1, reading slot 1 may reveal it.

---

## Ethernaut Vault Example

The Vault contract had two important variables:

```solidity
bool public locked;
bytes32 private password;
```

In this simple layout:

```text
slot 0 = locked
slot 1 = password
```

The password was marked `private`.

But it was still stored in raw contract storage.

Reading slot 1 revealed the password.

---

## Vault Storage Layout

The contract storage looked like this:

```text
slot 0 = locked
slot 1 = password
```

Before unlocking:

```text
locked = true
```

Slot 1 contained:

```text
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

This value was the stored password.

The player used it as the argument for:

```text
unlock(bytes32 _password)
```

---

## Vault Exploit Path

The exploit path was:

```text
1. Read storage slot 1
2. Extract the password
3. Call unlock(password)
4. Contract compares provided password with stored password
5. Match succeeds
6. locked becomes false
```

Expanded:

```text
player -> read Vault storage slot 1
player -> Vault.unlock(password)
Vault -> checks password
Vault -> locked = false
```

---

## Important Distinction

The exploit did not break encryption.

The exploit did not guess the password.

The exploit did not bypass the function directly.

The exploit used the fact that the password was stored publicly on-chain.

The mistake was assuming `private` meant secret.

---

## Call Context

At `Vault.unlock(password)`:

```text
msg.sender = player
argument   = password read from storage slot 1
state change = locked becomes false
```

Before the transaction:

```text
locked = true
```

After the transaction:

```text
locked = false
```

---

## Why This Works

The contract trusted a password stored on-chain.

But public blockchain storage can be inspected by anyone.

If the secret is stored on-chain, the attacker does not need special access.

The attacker only needs to read the correct storage slot.

---

## Root Cause

The root cause is insecure secret storage.

The contract stored sensitive authentication data directly in public contract storage.

This is unsafe because anyone can read the raw storage.

---

## Impact

The attacker could unlock the contract.

In this level, the impact was:

```text
locked changed from true to false
```

In real contracts, the same mistake could expose:

```text
passwords
private keys
commitment secrets
admin secrets
hidden configuration
future reveal data
game randomness
whitelist secrets
```

If the value controls access, funds, randomness, or privileged behavior, exposure can become serious.

---

## Common Beginner Mistake

A beginner may see:

```solidity
private
```

and think:

```text
Only the contract can see this.
```

That is wrong.

The correct interpretation is:

```text
Other Solidity contracts cannot directly access this variable by name, but the raw value is still publicly readable from blockchain storage.
```

---

## Public vs Private vs Secret

### Public

```solidity
uint256 public value;
```

Solidity automatically creates a getter function.

Other users and contracts can easily read it.

---

### Private

```solidity
uint256 private value;
```

Solidity does not create a public getter.

Other Solidity contracts cannot directly access it by name.

But the value still exists in public storage.

---

### Secret

A true secret should not be stored directly on-chain in readable form.

If a value must remain secret, it usually needs to stay off-chain until reveal time, or be handled through a safer cryptographic design.

---

## Safer Design Patterns

Do not store raw secrets on-chain.

Better options depend on the use case.

Possible safer patterns include:

```text
commit-reveal schemes
hashed commitments
off-chain secrets
threshold signatures
zero-knowledge proofs
secure randomness sources
careful delayed reveal mechanisms
```

But even these require careful design.

A simple hash may not be enough if the secret is weak or guessable.

---

## Example: Weak Hash Problem

A developer may think:

```text
I will store hash(password) instead of password.
```

That is better than storing the raw password.

But if the password is weak, attackers may brute-force it off-chain.

Example weak secrets:

```text
1234
password
admin
secret
small numbers
common words
```

So the safer rule is:

```text
Do not rely on simple on-chain password checks for serious security.
```

---

## Investigation Checklist

When reviewing a contract, ask:

```text
1. What variables are stored?
2. Are any variables marked private?
3. Are those variables actually sensitive?
4. Which storage slots contain important values?
5. Can those values be read with getStorageAt?
6. Does the contract use stored values for authorization?
7. Does the contract use stored values for randomness?
8. Would exposure of the value break the system?
```

---

## Forensics Use

Storage reading is useful in investigations.

It can help answer:

```text
What was the contract state before a transaction?
What changed after a transaction?
What value was used as an argument?
Was a hidden-looking variable actually readable?
Did the attacker read public storage before calling a function?
```

In the Vault level, the transaction only shows the unlock call.

The important pre-step was reading storage.

That storage read does not appear as a normal transaction because it can be done as a local call.

---

## Why Storage Reads May Not Appear As Transactions

Reading storage does not require changing blockchain state.

It can be done locally through an RPC call.

That means there may be no transaction hash for:

```text
getStorageAt(instance, slot)
```

The on-chain transaction appears later when the attacker uses the discovered value.

In Vault, the visible transaction was:

```text
unlock(bytes32 _password)
```

But the hidden preparation step was:

```text
read storage slot 1
```

---

## Practical Drill 1

Contract:

```solidity
bool public locked;
bytes32 private password;
```

Likely simple storage layout:

```text
slot 0 = locked
slot 1 = password
```

Question:

```text
Is password secret?
```

Answer:

```text
No. It is private in Solidity visibility, but still readable from raw storage.
```

---

## Practical Drill 2

If storage slot 1 contains:

```text
0x412076657279207374726f6e67207365637265742070617373776f7264203a29
```

And the contract has:

```solidity
function unlock(bytes32 _password) public {
    if (password == _password) {
        locked = false;
    }
}
```

Then the attacker can call:

```text
unlock(0x412076657279207374726f6e67207365637265742070617373776f7264203a29)
```

Expected state change:

```text
locked = false
```

---

## Practical Drill 3

Question:

```text
Why is this not a normal password system?
```

Answer:

```text
Because the password is stored on a public blockchain. Anyone can read the storage value and reuse it.
```

---

## Common Mistakes

### Mistake 1

Thinking `private` means encrypted.

Correction:

`private` does not encrypt the data.

---

### Mistake 2

Thinking Etherscan not showing a variable means it is hidden.

Correction:

Even if Etherscan does not show a nice getter, raw storage can still be inspected.

---

### Mistake 3

Thinking storage reads always create transactions.

Correction:

Storage reads can be done locally through RPC calls and do not require a state-changing transaction.

---

### Mistake 4

Thinking this only matters for passwords.

Correction:

It matters for any sensitive value stored on-chain.

Examples:

```text
random seeds
commitment secrets
game answers
admin secrets
private sale allowlists
future reveal metadata
```

---

## Final Summary

Solidity `private` only controls access from Solidity code.

It does not hide the value from blockchain observers.

Raw contract storage is publicly readable.

In Ethernaut Vault, the password was marked private, but it was stored in slot 1.

Reading slot 1 revealed the password.

The player then called:

```text
unlock(password)
```

The contract accepted the value and changed:

```text
locked = false
```

The main security lesson is:

```text
Do not store secrets directly on-chain.
```
