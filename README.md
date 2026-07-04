# Blockchain Security Labs

Junior blockchain security portfolio focused on smart contract exploit reproduction, Web3 command understanding, and basic blockchain transaction analysis.

## Focus Areas

- Solidity security fundamentals
- Ethernaut exploit writeups
- Web3 console command decoding
- Etherscan transaction analysis
- Smart contract bug pattern library
- Beginner blockchain forensics notes

## Current Status

I am building practical security skill through hands-on labs.  
The goal is not only to solve challenges, but to understand the vulnerable logic, reproduce exploits, collect evidence, and write clear findings.

## Repository Structure

```text
ethernaut/
  01-fallback/
  02-fallout/
  03-coinflip/
  04-telephone/
  05-token/
  06-delegation/
  07-force/
  08-vault/
  09-king/

web3-survival-kit/
  commands.md
  address-roles.md
  calldata-and-abi.md
  mistakes-log.md

forensics/
  transaction-report-01-vault.md
  transaction-report-02-telephone.md

pattern-library/
  access-control.md
  tx-origin.md
  storage-privacy.md
  delegatecall.md
  accounting-bugs.md
```
## Completed Writeups

| Lab | Pattern | Status |
|---|---|---|
| Ethernaut Vault | Private storage is not secret | Drafted |
| Ethernaut Fallback | Unsafe receive() ownership takeover | Drafted |
| Ethernaut Telephone | tx.origin caller confusion | Drafted |

## Method

Each lab follows this structure:

1. Vulnerable assumption
2. Root cause
3. Attack path
4. Evidence
5. Recommended fix
6. Real-world relevance

## Disclaimer

This repository is for ethical security learning, exploit reproduction in controlled environments, and defensive research.
