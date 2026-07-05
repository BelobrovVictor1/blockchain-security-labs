# Blockchain Security Labs

Public proof-of-work repository for my blockchain security, smart contract security, transaction analysis, and on-chain investigation learning.

This repo documents controlled lab work, exploit explanations, transaction reports, Web3 notes, and investigation-style writeups.

Current level: beginner / early junior.

Current focus:

```text
Solidity security basics
Ethernaut labs
Etherscan analysis
transaction reconstruction
storage analysis
internal call tracing
Web3 command understanding
smart contract risk notes
blockchain investigation reports
```

---

## Purpose

The purpose of this repo is to build practical, evidence-based security skill.

Each artifact should prove at least one of the following:

```text
I understand a vulnerability.
I can reproduce a controlled exploit in a lab.
I can explain a transaction clearly.
I can separate root cause from impact.
I can read contract state and storage.
I can explain Web3 concepts without vague language.
I can write reports with scope and limitations.
```

This repo is not meant to look perfect.

It is meant to show disciplined progress.

---

## Current Proof Kit Status

Minimum proof kit:

```text
12 / 12 complete
```

Completed categories:

```text
Ethernaut writeups
Transaction reports
Web3 concept notes
Mistakes log
Sample client-style report
```

---

## Repository Structure

```text
ethernaut/
  01-fallback/
    writeup.md
  04-telephone/
    writeup.md
  08-vault/
    writeup.md

forensics/
  transaction-report-01-vault.md
  transaction-report-02-telephone.md
  transaction-report-03-fallback.md

notes/
  web3-tx-origin-vs-msg-sender.md
  web3-external-vs-internal-calls.md
  web3-private-storage-is-not-secret.md
  mistakes-log.md

samples/
  sample-transaction-explanation-report.md
```

---

# Key Artifacts

## Ethernaut Writeups

### 01 — Fallback

File:

```text
ethernaut/01-fallback/writeup.md
```

Concepts:

```text
receive()
raw ETH transfer
unsafe ownership change
broken access control
root cause vs impact path
```

Core lesson:

```text
Receiving ETH should not silently grant privileged permissions.
```

---

### 04 — Telephone

File:

```text
ethernaut/04-telephone/writeup.md
```

Concepts:

```text
tx.origin
msg.sender
helper contracts
contract-to-contract calls
unsafe authorization logic
```

Core lesson:

```text
Do not use tx.origin for authorization.
```

---

### 08 — Vault

File:

```text
ethernaut/08-vault/writeup.md
```

Concepts:

```text
private storage
raw storage reads
getStorageAt
on-chain data visibility
insecure secret storage
```

Core lesson:

```text
private in Solidity does not mean secret on-chain.
```

---

## Transaction / Forensics Reports

### Vault Transaction Report

File:

```text
forensics/transaction-report-01-vault.md
```

Focus:

```text
storage slot reading
password extraction from storage
unlock transaction
state change from locked = true to locked = false
```

---

### Telephone Transaction Report

File:

```text
forensics/transaction-report-02-telephone.md
```

Focus:

```text
external transaction
helper contract
internal call
tx.origin vs msg.sender
ownership change
```

---

### Fallback Transaction Report

File:

```text
forensics/transaction-report-03-fallback.md
```

Focus:

```text
contribute()
raw ETH transfer
receive()
ownership takeover
withdraw()
root cause vs impact path
```

---

## Web3 Notes

### tx.origin vs msg.sender

File:

```text
notes/web3-tx-origin-vs-msg-sender.md
```

Focus:

```text
original transaction starter
direct function caller
call-location analysis
Telephone-style vulnerability
```

Useful mental model:

```text
At Contract.function(...):
tx.origin = ?
msg.sender = ?
argument = ?
state change = ?
```

---

### External Transactions vs Internal Calls

File:

```text
notes/web3-external-vs-internal-calls.md
```

Focus:

```text
external transactions
internal calls
Etherscan interpretation
contract-to-contract execution
exploit path reconstruction
```

Core lesson:

```text
The external transaction is not always the full security story.
```

---

### Private Storage Is Not Secret

File:

```text
notes/web3-private-storage-is-not-secret.md
```

Focus:

```text
Solidity private visibility
public blockchain storage
storage slots
Vault-style vulnerabilities
```

Core lesson:

```text
Do not store secrets directly on-chain.
```

---

### Mistakes Log

File:

```text
notes/mistakes-log.md
```

Focus:

```text
corrected mental models
beginner mistakes
security reasoning discipline
evidence vs interpretation
scope control
```

Core lesson:

```text
Security learning requires honest error correction.
```

---

## Sample Report

### Sample Transaction Explanation Report

File:

```text
samples/sample-transaction-explanation-report.md
```

Purpose:

```text
Example of a client-style report explaining a blockchain transaction sequence.
```

Covers:

```text
important notice
executive summary
addresses reviewed
timeline
state changes
root cause
impact
limitations
final conclusion
```

This sample is based on a controlled Ethernaut lab.

It is not a real client investigation.

---

# Current Technical Themes

## 1. Access Control

Examples:

```text
Telephone
Fallback
```

Questions being practiced:

```text
Who is allowed to call this function?
What condition controls privilege?
Can a non-owner become owner?
Is authorization based on msg.sender or tx.origin?
```

---

## 2. Storage Visibility

Examples:

```text
Vault
```

Questions being practiced:

```text
What is stored on-chain?
Which storage slot contains the value?
Is a private variable actually sensitive?
Can the value be read with getStorageAt?
```

---

## 3. Transaction Reconstruction

Examples:

```text
Telephone transaction report
Fallback transaction report
Vault transaction report
```

Questions being practiced:

```text
Who started the transaction?
Which contract was called first?
Were there internal calls?
Which function changed state?
What was the root cause?
What was the final impact?
```

---

## 4. Report Writing

Reports should separate:

```text
evidence
interpretation
root cause
impact path
limitations
conclusion
```

Weak report style:

```text
The contract got hacked.
```

Better report style:

```text
The contract allowed ownership takeover because receive() changed owner after a small contribution and raw ETH transfer.
```

---

# Current Skill Level

I am currently beginner / early junior.

Strengths:

```text
hands-on learning
transaction report writing
basic exploit explanation
early call-context analysis
storage visibility understanding
clear correction of mistakes
```

Current weaknesses:

```text
Solidity syntax still needs work
Web3 / JavaScript console commands still need work
Foundry not yet integrated
delegatecall still needs deeper study
not ready for full private audits
client pipeline not built yet
```

---

# Ethics and Boundaries

This repo is for defensive education, controlled lab work, and public blockchain analysis.

Allowed:

```text
controlled lab exploitation
defensive smart contract review
transaction analysis
wallet-flow analysis from public blockchain data
incident reconstruction from public data
security education
responsible disclosure
```

Not allowed:

```text
stealing funds
unauthorized exploitation
phishing
malware
evasion
helping attackers hide funds
bypassing access controls
laundering guidance
recovery hacking
private key or seed phrase handling
```

---

# What This Repo Does Not Claim

This repo does not claim:

```text
professional audit certification
complete smart contract audit capability
guaranteed vulnerability discovery
fund recovery capability
legal attribution
law enforcement-level investigation
```

The current goal is to build practical security skill and credible proof-of-work step by step.

---

# Next Technical Milestones

Near-term:

```text
1. Improve Solidity syntax
2. Learn Web3 / JavaScript console commands properly
3. Practice ABI and calldata decoding
4. Start Foundry basics
5. Reproduce simple exploits locally
6. Re-learn delegatecall carefully
7. Write more transaction reports from real public examples
```

Medium-term:

```text
1. Foundry-based exploit tests
2. More smart contract vulnerability classes
3. Token approvals and allowance analysis
4. Wallet-flow reports
5. Suspicious transaction timelines
6. Pre-audit preparation checklists
```

---

# Business Direction

Long-term goal:

```text
Build a high-trust boutique security, investigations, and forensics practice.
```

Current entry point:

```text
crypto / blockchain transaction analysis
smart contract risk notes
wallet-flow reports
on-chain investigation reports
founder security education
```

Current safe service direction:

```text
Transaction Explanation Reports
Wallet Flow Reports
Suspicious Transaction Timelines
Pre-Audit Preparation Checklists
Smart Contract Risk Notes
```

Not currently selling:

```text
full smart contract audits
certified security reviews
fund recovery
legal attribution
incident response retainers
```

---

# Current Status

Minimum public proof kit is complete.

Next phase:

```text
clean presentation
service offer
outreach list
first low-risk reports
market feedback
```

The repo now has enough proof to support beginner-safe outreach for tightly scoped blockchain transaction explanation services.
