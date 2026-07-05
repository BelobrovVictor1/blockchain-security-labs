# Mistakes Log — Blockchain Security Learning

## Purpose

This file tracks mistakes, confusions, and corrected mental models from my blockchain security learning process.

The goal is not to look perfect.

The goal is to show disciplined error correction.

Security work requires:

```text
clear thinking
evidence-based conclusions
precise language
honest uncertainty
repeatable investigation habits
```

A mistake is useful if it becomes a better rule, checklist, or report standard.

---

## Current Level

Beginner / early junior.

Current focus:

```text
Ethernaut labs
Solidity security basics
Web3 console commands
Etherscan analysis
transaction reports
wallet-flow thinking
exploit reproduction
smart contract risk notes
```

---

# Mistake 1 — Confusing `tx.origin` and `msg.sender`

## What I Initially Got Wrong

I initially treated `tx.origin` and `msg.sender` as if they were basically the same thing.

That is only true in simple direct wallet-to-contract calls.

It becomes wrong when one contract calls another contract.

---

## Correct Mental Model

```text
tx.origin  = the wallet that started the whole transaction
msg.sender = the direct caller of the current function
```

For this call chain:

```text
player wallet -> helper contract -> target contract
```

At the helper contract:

```text
tx.origin  = player wallet
msg.sender = player wallet
```

At the target contract:

```text
tx.origin  = player wallet
msg.sender = helper contract
```

---

## Better Question Format

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

Then:

```text
At Target.changeOwner(player):
tx.origin = ?
msg.sender = ?
argument = ?
state change = ?
```

The answer depends on the exact function location.

---

## Lesson

Always analyze call context at a specific function location.

Never answer `tx.origin` / `msg.sender` questions globally.

---

# Mistake 2 — Reading Only the External Transaction

## What I Initially Got Wrong

I looked at the external transaction and risked treating it as the whole story.

But the external transaction only shows the first layer.

In exploit analysis, the real vulnerable action can happen inside an internal call.

---

## Correct Mental Model

An external transaction is started by a wallet.

An internal call is made by a contract during that same transaction.

Example:

```text
player wallet -> helper contract -> target contract
```

External transaction:

```text
player wallet -> helper contract
```

Internal call:

```text
helper contract -> target contract
```

---

## Lesson

A transaction report should reconstruct the full path:

```text
wallet -> contract A -> contract B -> state change
```

Not just:

```text
wallet -> contract A
```

---

# Mistake 3 — Confusing Root Cause With Impact Path

## What I Initially Got Wrong

In Fallback, it is easy to think that `withdraw()` is the vulnerability because it drains the contract.

That is incomplete.

`withdraw()` is the final impact path.

The real vulnerability happens earlier.

---

## Correct Mental Model

In the Fallback level:

```text
root cause = unsafe ownership change inside receive()
impact path = withdraw()
```

The dangerous state change was:

```text
owner = msg.sender
```

inside `receive()`.

After the player became owner, `withdraw()` became possible.

---

## Lesson

In every report, separate:

```text
root cause
trigger
state change
impact path
final result
```

This makes the report more professional and less shallow.

---

# Mistake 4 — Thinking `private` Means Secret

## What I Initially Got Wrong

I saw a Solidity variable marked `private` and treated it like it was hidden.

That is wrong on a public blockchain.

---

## Correct Mental Model

```text
private != secret
```

Solidity `private` means other Solidity contracts cannot directly access the variable by name.

It does not mean the data is hidden from blockchain storage.

Raw storage is public.

Anyone can read it using storage inspection tools or RPC calls such as:

```text
getStorageAt(contractAddress, slot)
```

---

## Vault Example

The Vault contract had:

```solidity
bool public locked;
bytes32 private password;
```

Simple storage layout:

```text
slot 0 = locked
slot 1 = password
```

Reading slot 1 revealed the password.

Then the player called:

```text
unlock(password)
```

The contract changed:

```text
locked = false
```

---

## Lesson

Do not store secrets directly on-chain.

If a value must remain secret, it should not be stored in public contract storage in readable form.

---

# Mistake 5 — Treating Storage Reads Like Transactions

## What I Initially Got Wrong

I risked expecting every important action to have a transaction hash.

But reading storage does not require a state-changing transaction.

---

## Correct Mental Model

Storage reads can be done locally through an RPC call.

They do not change blockchain state.

So they may not appear as normal transactions.

Example:

```text
getStorageAt(instance, 1)
```

This can reveal a value without creating a transaction.

The visible transaction comes later when the value is used.

---

## Lesson

A transaction report may need to include off-chain preparation steps, such as:

```text
storage read
ABI decoding
function selector lookup
source code review
Etherscan inspection
```

Not every important investigation step has a transaction hash.

---

# Mistake 6 — Wanting to Rush Toward Harder Levels Too Early

## What I Initially Got Wrong

I wanted to move quickly to harder concepts like delegatecall before fully stabilizing the basics.

That can create fake progress.

---

## Correct Mental Model

Harder topics only help if the foundation is stable.

Before delegatecall, I need stronger understanding of:

```text
external calls
internal calls
msg.sender
tx.origin
storage
calldata
ABI
function selectors
contract state changes
```

---

## Lesson

Do not confuse difficulty with progress.

A simple concept explained precisely is more valuable than a hard concept explained vaguely.

---

# Mistake 7 — Underestimating Web3 / JavaScript Console Skills

## What I Initially Got Wrong

I focused on solving Ethernaut levels but did not fully understand the Web3 / JavaScript commands used during the solving process.

That is a problem.

If I can run a command but cannot explain it, the skill is incomplete.

---

## Correct Mental Model

Every command should become understandable.

For example, I should know:

```text
what object/function I am calling
what address is being queried
what network is being used
whether the call changes state
whether it costs gas
what result is returned
how to verify the result
```

---

## Lesson

No blind command execution.

Every useful Web3 command should eventually become a note, drill, or reusable checklist.

---

# Mistake 8 — Thinking GitHub Alone Creates Money

## What I Initially Got Wrong

I started worrying that adding GitHub files might not directly create income.

That concern is correct.

GitHub does not create money by itself.

---

## Correct Mental Model

GitHub is the trust engine.

Money comes from:

```text
clear service offer
sample reports
outreach
client conversations
strict scope
delivery quality
repeatable reports
testimonials
referrals
```

The portfolio supports sales, but it does not replace sales.

---

## Lesson

The correct system is:

```text
skill engine + proof engine + market engine
```

Not just:

```text
study and upload files
```

---

# Mistake 9 — Risk of Overclaiming Too Early

## What I Initially Got Wrong

The long-term goal includes smart contract audits and forensics, but I am not ready to sell full audits yet.

Selling too much too early would create technical, legal, ethical, and reputational risk.

---

## Correct Mental Model

Early services should be low-risk and tightly scoped.

Good early offers:

```text
Transaction Explanation Report
Wallet Flow Report
Suspicious Transaction Timeline
Founder Security Education Note
Pre-Audit Preparation Checklist
Smart Contract Risk Notes
```

Avoid for now:

```text
full audits
certified security
guaranteed recovery
legal attribution
recovery hacking
incident response retainers
```

---

## Lesson

Trust is built by accurate scope.

A small promise delivered well is better than a large promise delivered badly.

---

# Mistake 10 — Not Separating Evidence From Interpretation

## What I Initially Got Wrong

In security reporting, it is tempting to jump directly to conclusions.

But a good report separates what is visible from what is inferred.

---

## Correct Mental Model

Evidence:

```text
Transaction hash
From address
To address
Input data
Value
Token transfer
Internal call
State change
Contract source code
Storage slot value
```

Interpretation:

```text
This pattern appears suspicious
This likely indicates approval misuse
This may be a phishing-style flow
This suggests the caller gained ownership
```

Conclusion:

```text
Based on the visible on-chain evidence, the most likely explanation is...
```

---

## Lesson

Use careful language.

Do not say:

```text
This is definitely the criminal.
```

Say:

```text
This address received the funds on-chain. This report does not identify the real-world controller of the address.
```

---

# Current Personal Rules

## Rule 1

Every technical concept must become one of:

```text
GitHub file
bug card
transaction report
Web3 note
service asset
public post draft
```

---

## Rule 2

Every report must separate:

```text
root cause
trigger
state change
impact path
limitations
```

---

## Rule 3

Every transaction analysis must ask:

```text
At the relevant function:
tx.origin = ?
msg.sender = ?
arguments = ?
state change = ?
```

---

## Rule 4

Do not use vague labels when precise labels are possible.

Weak:

```text
The contract got hacked.
```

Better:

```text
The contract allowed ownership takeover through unsafe logic inside receive().
```

---

## Rule 5

No full audit claims until technical depth, tooling, and review process are much stronger.

---

## Rule 6

Do not hide inside study.

After the proof kit is complete, start the market engine:

```text
service offer
target list
outreach
sample reports
feedback
paid tests
```

---

# Current Next Improvements

## Technical

```text
1. Strengthen Solidity syntax
2. Learn Web3 / JavaScript console commands properly
3. Start Foundry after the minimum proof kit
4. Re-learn delegatecall slowly
5. Practice calldata and ABI decoding
6. Practice reading token approvals and transfers on Etherscan
```

## Business

```text
1. Finish service menu v2
2. Create outreach message
3. Build 20-target list
4. Send first 5 soft outreach messages
5. Try first free/testimonial or $25 report
```

---

# Final Summary

This mistakes log is a record of corrected thinking.

The most important corrections so far are:

```text
tx.origin is not msg.sender
external transaction is not the full execution path
private storage is not secret
withdraw() can be the impact path, not the root bug
storage reads may not create transactions
GitHub creates trust, not money by itself
early services must be scoped tightly
evidence must be separated from interpretation
```

The goal is not to avoid mistakes.

The goal is to catch them, correct them, and turn them into better security judgment.
