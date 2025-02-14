# **Mento Audit Competition on Hats.finance** 


## Introduction to Hats.finance


Hats.finance builds autonomous security infrastructure for integration with major DeFi protocols to secure users' assets. 
It aims to be the decentralized choice for Web3 security, offering proactive security mechanisms like decentralized audit competitions and bug bounties. 
The protocol facilitates audit competitions to quickly secure smart contracts by having auditors compete, thereby reducing auditing costs and accelerating submissions. 
This aligns with their mission of fostering a robust, secure, and scalable Web3 ecosystem through decentralized security solutions​.

## About Hats Audit Competition


Hats Audit Competitions offer a unique and decentralized approach to enhancing the security of web3 projects. Leveraging the large collective expertise of hundreds of skilled auditors, these competitions foster a proactive bug hunting environment to fortify projects before their launch. Unlike traditional security assessments, Hats Audit Competitions operate on a time-based and results-driven model, ensuring that only successful auditors are rewarded for their contributions. This pay-for-results ethos not only allocates budgets more efficiently by paying exclusively for identified vulnerabilities but also retains funds if no issues are discovered. With a streamlined evaluation process, Hats prioritizes quality over quantity by rewarding the first submitter of a vulnerability, thus eliminating duplicate efforts and attracting top talent in web3 auditing. The process embodies Hats Finance's commitment to reducing fees, maintaining project control, and promoting high-quality security assessments, setting a new standard for decentralized security in the web3 space​​.

## Mento Overview

Mento powers stable digital currencies enabling global financial access on Celo.

## Competition Details


- Type: A public audit competition hosted by Mento
- Duration: 2 weeks
- Maximum Reward: $15,999.18
- Submissions: 45
- Total Payout: $1,559.92 distributed among 3 participants.

## Scope of Audit

## Project overview
The Mento protocol is a smart contract platform built on the Celo blockchain that enables the creation and exchange of stable-value digital assets. Stable assets created with Mento can be classified as 'hybrid stable assets' as they are algorithmic, transparent, and backed by an over-collateralized, diversified portfolio of exogenous crypto assets.

## Audit competition scope
This audit focuses on Mento's [Locking Contracts](https://github.com/mento-protocol/mento-core/tree/develop/contracts/governance/locking). These contracts are a fork of [Rarible's Locking Contracts](https://github.com/rarible/locking-contracts), which are inspired by [Curve's veToken model](https://resources.curve.fi/crv-token/overview/). Users can lock their MENTO in exchange for veMENTO, which grants them the ability to create and vote on governance proposals.

The first version of the Locking Contracts was previously audited by [0xMacro](https://0xmacro.com/library/audits/mento-2) and [Sherlock](https://audits.sherlock.xyz/contests/187?filter=results). With Celo's transition to an L2, the block time will be reduced from 5 seconds to 1 second. To accommodate this update, we have introduced changes to the Locking contracts that:
- Ensure the governance voting period remains consistent at approximately one week
- Protect users' existing Locks from being affected
- Provide MentoLabs with additional flexibility during the L2 transition

While all [Locking contracts](https://github.com/mento-protocol/mento-core/tree/develop/contracts/governance/locking) are in scope and provide context for understanding the locking mechanism, we request that participants focus particularly on [the recent changes](https://github.com/mento-protocol/mento-core/pull/542/files), as these have not been audited before.

The key changes introduced include:
- Renaming of `roundTimestamp()` to `getWeekNumber()` and updating it to support both L1 (5s) and L2 (1s) block times
- Modification of `_getEpochShift()` to work with two epoch shifts
- Addition of L2-related variables in `LockingBase.sol`
- Setter functions for L2-related variables
- Pause functionality for locking and governance functions
- A new temporary role for MentoLabs Multisig with permissions to call:
    - `setL2TransitionBlock()`
    - `setL2EpochShift()`
    - `setL2StartingPointWeek()`
    - `setPaused()`

The files in scope:
```
|-- mento-core/
     |-- contracts/
          |-- governance/
               |-- locking/
                    |-- libs/
                        |-- LibBrokenLine.sol
                        |-- LibIntMapping.sol
                    |-- Locking.sol
                    |-- LockingBase.sol
                    |-- LockingRelock.sol
                    |-- LockingVotes.sol
```

## Migration Plan

To provide clarity on the L2 upgrade and how the updated contracts will support this process, here is our migration plan:

- Following the Hats audit contest and resolution of valid issues, we will deploy the new implementation of Locking contracts and update the existing Locking proxy to point to this new implementation. As this is an upgrade, we request that contest participants verify the correct usage of storage slots in the upgraded contracts.
- Post-upgrade, a governance proposal will be initiated to set the Mento Labs multisig address via Mento Governance calling `setMentoLabsMultisig()`. Participants should proceed with the assumption that this proposal will pass and the multisig address will be properly configured.
- Once Celo announces the L2 transition block number, we will compute the appropriate values for `l2StartingPointWeek` and `l2EpochShift`.
- Approximately 10 days before the L2 transition, the Mento Labs multisig will execute `setL2TransitionBlock`with the designated L2 transition block number, which will pause related Locking and Governance functions. This pause is aimed to ensure a Governance and Locking freeze during the L2 transition. Any active proposals during this freeze period may fail to queue or execute, which is expected. In case of any unexpected proposals, Mento Watchdogs will have the authority to veto proposals.
- Following Celo's successful transition to L2, we will:
    - Revalidate the calculated values for `l2StartingPointWeek` and `l2EpochShift`
    - Have the Mento Labs Multisig call `setL2StartingPointWeek` and `setL2EpochShift`

> Participants should assume these functions will be called with correct values. Issues related to calculation errors will be considered invalid unless participants can demonstrate that correct value implementation is impossible (e.g., proving that the correctly calculated value would cause an overflow/underflow).

- After setting the L2-related variables and confirming proper lock accounting for both L2 and L1 block numbers, the Mento Labs multisig will unpause locking and governance functions.
- Following a period of observation to verify that all locks (both pre-transition and post-transition) are functioning as intended, we will remove the temporary Mento Labs role.
- To address any unexpected scenarios, we have established a new Proxy Admin and transferred upgradability rights to that proxy through this proposal: https://governance.mento.org/proposals/19794090652830713445593922349268260764133965786127689022060988255391916327351





## Low severity issues


- **Add a mechanism to prevent frontrunning of initialization in Locking.sol**

  After deploying the `Locking.sol` contract, there is a risk of the `__Locking_init` function being called by an attacker before legitimate initialization, allowing them to input malicious parameters and compromise the contract's functionality. Adding a `disableInitialize` mechanism in the constructor is recommended to prevent such an attack.


  **Link**: [Issue #43](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/43)


- **LockingBase.sol has incorrect storage slot gap impacting future upgrades**

  The `LockingBase.sol` contract miscalculated storage slot gaps, which could affect future upgrades by leading to storage collisions. An upgrade added three storage slots but didn't correctly adjust the gap, leading to inconsistency with the project's 50-slot standard. Adjusting the gap now could risk storage layout shifting. The issue highlights an improvement area, though the proposed solution could inadvertently cause problems.


  **Link**: [Issue #12](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/12)


- **Mento's relock mechanism missing minimum amount check found in Locking.sol**

  The Mento locking mechanism, based on Rarible contracts, has a flaw in its relocking process. Unlike the initial lock, relocking doesn't enforce a minimum amount, allowing users to bypass the validation by providing a new amount greater than the residual value, even if the residual is zero. A proposed resolution includes ensuring the total of residue and new amount meets the minimum required.


  **Link**: [Issue #10](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/10)


- **Add disableInitialize mechanism to prevent frontrunning in Locking.sol constructor**

  The `Locking.sol` contract can be vulnerable to a frontrunning attack during initialization because it lacks a protective mechanism like `disableInitialize` in its constructor. An attacker could exploit this by preemptively invoking the `__Locking_init` function, initializing the contract with malicious parameters, and therefore impacting its intended operation. Adding such a mechanism is recommended to prevent this risk.


  **Link**: [Issue #43](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/43)


- **Incorrect Storage Slot Gap Calculation in LockingBase.sol Contract**

  The `LockingBase.sol` contract has an incorrect storage slot gap calculation that could affect future upgrades. It attempts to reserve 50 storage slots, but with recent upgrades and previously used slots, it actually uses 59 slots. This inconsistency risks storage collisions during future updates, but fixing it now could cause additional problems.


  **Link**: [Issue #12](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/12)


- **Mento Locking Mechanism Allows Relocking Without Minimum Amount Due to Bypass**

  The Mento locking mechanism, derived from Rarible contracts, has a flaw in relocking that allows bypassing minimum lock funds because the relock function lacks the necessary checks. Adding a verification step to ensure the sum of residue and new amounts meet the minimum requirement can remedy this inconsistency.


  **Link**: [Issue #10](https://github.com/hats-finance/Mento-0x2a1b9b1f6fa7c2e73815a7dff0e1688767382694/issues/10)



## Conclusion

The Hats.finance audit competition provided a decentralized approach for enhancing the security of the Mento protocol, which supports stable digital currencies on the Celo blockchain. Over a two-week period, the competition received 45 submissions, resulting in a total payout of $1,559.92. The audit focused specifically on Mento's Locking Contracts, a fork of Rarible's Locking Contracts, which facilitate governance through token locking. Recent updates to accommodate Celo's transition to L2 blockchain were a priority, including changes to maintain governance voting periods and user protections during the transition. Essential alterations included renaming functions, adding L2-related variables, and introducing pause functionalities for governance. The audit identified low-severity issues such as minor vulnerabilities in the relock mechanism and storage slot miscalculations. Overall, the competition helped mitigate risks, and by implementing recommended improvements, Mento is poised for a smooth transition to L2, bolstering its decentralized governance capabilities and security.

## Disclaimer


This report does not assert that the audited contracts are completely secure. Continuous review and comprehensive testing are advised before deploying critical smart contracts.


The Mento audit competition illustrates the collaborative effort in identifying and rectifying potential vulnerabilities, enhancing the overall security and functionality of the platform.


Hats.finance does not provide any guarantee or warranty regarding the security of this project. Smart contract software should be used at the sole risk and responsibility of users.

