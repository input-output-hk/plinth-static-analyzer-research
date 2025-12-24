# Plutarch Audit Summaries

## Overview

This document summarizes findings from **5 Plutarch audits** in the Cardano ecosystem.

**Findings Classification**:

- **Relevant findings**: 19
- **May be relevant findings**: 5
- **Not relevant findings**: 78
- **Total findings**: 102

### Common Patterns

Common issues in Plutarch smart contracts include:

1. **Incomplete Token Validation (2 occurrences) - [INCOMPLETE-TOKEN-VALIDATION]**: Validators that check currency symbol but not token name, or vice versa. This pattern allows attackers to use wrong tokens (with correct symbol but wrong name) or mint unauthorized tokens under the same policy, leading to vote manipulation and authentication bypass.

2. **Trash Tokens (2 occurrences) - [TRASH-TOKENS]**: Validators using subset checks instead of exact equality for value validation, allowing attackers to bloat UTxOs with arbitrary tokens.

3. **Operations Without State Changes (2 occurrences) - [UNCHANGED-STATE]**: Validators that allow operations to succeed without modifying any state, enabling denial-of-service attacks by repeatedly executing transactions that pass validation but accomplish nothing.

4. **Unvalidated Datum (2 occurrences) - [UNVALIDATED-DATUM]**: Validator fails to validate datum fields on continuing outputs, allowing datum tampering and collateral theft. .

5. **Missing Address Validation (1 occurrence) - [MISSING-ADDRESS-VALIDATION]**: Minting policy that validates token properties without verifying the destination address, allowing critical borrow tokens to be redirected and enabling market drainage attacks.

## Agora, Agora pro

**Auditor**: VacuumLabs

**Auditee**: Liqwid Labs

**Description**: Agora is a Cardano native on-chain governance module written in Plutarch. It provides the necessary building blocks for any on-chain decentralized application to adopt and extend itself into a DAO-governed application. It consists of an open-source module (Agora) and a proprietary extension (Agora-Pro)

### Findings

**Relevant**

- **AGO-104. Attacker can fail any voted-on/locked proposal**: Validator doesn't check transaction validity range length.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **AGO-105. Stake ST token name not checked**: Stake validator validates currency symbol but not token name of stake ST, allowing attacker to use tokens with wrong names and fake vote history.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AGO-203. Stakes can be frozen effectively forever by the multisig entity**: Validator doesn't check transaction validity range length.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **AGO-204. Governor can be DoSed by creating Proposals without passing min GT limit**: Function validates currency symbol but not token name of stake ST.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AGO-306: Proposal can be DoSed by using UnlockStake with no input stakes**: Proposal voting could be DoSed spending a proposal with the correct redeemer, including no stake inputs.<br><ins>Detectable pattern</ins>: operations that succeed without modifying state [UNCHANGED-STATE]
- **AGO-307. Proposal can be DoSed by using UnlockStake with relevant cosigners' stakes**: Validator allows operation that passes validation but doesn't change state, enabling DoS attacks.<br><ins>Detectable pattern</ins>: operations that succeed without modifying state [UNCHANGED-STATE]

**May be relevant**

- **AGO-401. Staking credential is undefined**: Protocol doesn't define staking credentials for script UTxOs, potentially missing staking rewards and complicating off-chain UTxO discovery.<br><ins>Detectable pattern</ins>: script addresses without defined staking credentials [UNVALIDATED-STAKING]

**Not relevant**

- **AGO-001. Stake state token can be taken away**: Stake validator doesn't enforce stake state token is burned when stake is destroyed, allowing attacker to reuse token in malicious stake UTxO with inflated voting power. Requires semantic understanding of protocol lifecycle and which tokens should be burned versus returned in specific operations
- **AGO-002. Acting on behalf of delegatee role + Unlocking delegated stakes**: Validator determines signature context globally across all inputs instead of per-input, allowing stake owner to act on others' stakes with same delegatee. Requires semantic understanding of authorization model
- **AGO-003. Fake proposal can be created and GAT minted without any voting happening**: Proposal minting policy checks governor ST presence but doesn't validate redeemer. Requires semantic understanding of which redeemers should permit which token minting operations
- **AGO-004. Multiple GATs can be minted into fake scripts**: Authority token minting policy doesn't validate the quantity of tokens minted per output. Requires semantic understanding of intended token distribution model
- **AGO-005. Stake lock can be removed without retracting votes**: Bug in function with inverted filter logic causes locks from unrelated proposals to be removed when retracting votes. Logic bug requiring semantic understanding of intended behavior
- **AGO-101. Delegatee can steal delegated inputs**: Function validates input-output correspondence but checks subset instead of equality, allowing outputs to be missing. Requires semantic understanding that inputs and outputs should correspond 1:1
- **AGO-102. Stake ST minting policy does not check lockedBy**: Minting policy doesn't validate datum field is empty when creating new stake. Requires semantic understanding of correct initial state for datum fields
- **AGO-103. pdestroy doesn't check outputs**: Destroy redeemer doesn't validate absence of stake outputs, allowing attacker to create new stake UTxO with a datum that doesn’t pass certain checks. Requires semantic understanding of which operations should be terminal versus those that produce continuing outputs
- **AGO-106. Stake can retract all votes in its cooldown period**: Cooldown validation uses filter instead of guard. Requires semantic understanding of whether code is selecting valid items to process versus validating all items must be valid
- **AGO-201. GATs are equal in the potential damage they can cause**: All governance authority tokens have equal power regardless of action criticality, allowing any passed proposal to fully compromise the system. Protocol governance issue
- **AGO-202. getStakeDatum is staking credential sensitive**: Function filters stake UTxOs by address including staking credential, causing each stake to see only a subset of stakes with matching credential. Requires semantic understanding of protocol's staking credential model and intended validation scope
- **AGO-301. Ambiguity in destroying multiple stakes**: Stake validator allows destroying multiple stakes per transaction but minting policy doesn't allow burning multiple stake STs at once. Protocol design inconsistency
- **AGO-302. Some strict inequalities should not be strict**: Threshold comparisons use strict inequalities where inclusive inequalities are expected. Requires understanding business logic
- **AGO-303. tooLate in code can also mean early**: Logic bug where proposal actions before startingTime are incorrectly treated as "too late" instead of "too early", allowing premature proposal finalization
- **AGO-304. Other minor inconsistencies**: Documentation and code quality issues
- **AGO-305. Other naming suggestions**: Code quality and documentation issues
- **AGO-308. Proposal can be DoSed by voting and unlocking repeatedly**: Attacker can repeatedly vote and unlock stake to DoS proposal by constantly changing the proposal UTxO, invalidating other users' transactions. Protocol design issue
- **AGO-309. Incorrect token references across the code**: Variable names and comments incorrectly reference which tokens are used
- **AGO-402. PUnlock**: Ambiguous proposal redeemer name: Redeemer named PUnlock suggests unlocking proposal but actually unlocks stakes. Naming issue
- **AGO-403. Delegatee cannot vote with delegated and own stakes in one transaction**: Validation requires all stakes to have the same owner or same delegatee, preventing mixed transactions. Protocol design issue

## Splash Protocol Dex

**Auditor**: AnastasiaLabs

**Auditee**: Spectrum Labs

**Description**: Unlike centralized exchanges that match buy and sell orders (aka CLOB exchanges), or constant product Automated Market Maker (AMM) exchanges, Splash uses different types of AMM liquidity pools, the Virtual Limit Order Book (VLOB), and combines them all. This allows different types of market makers to earn interest by providing liquidity as efficiently as they want, and traders to benefit from the best prices by tapping all liquidity in a single order.

### Findings

**Relevant**

- **ID-101. Other token name:** Pool NFT and liquidity token minting policies validate expected token name is minted but don't restrict minting of additional tokens with different names under the same currency symbol.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-106. Duplicates in DAO signers:** DAOPolicy signer list doesn't check for duplicates, allowing some key holders to have amplified voting power and increased risk if compromised.<br><ins>Detectable pattern</ins>: lists of identity/authorization types (PubKeyHash, Address, ValidatorHash, etc.) without uniqueness validation [LIST-UNIQUENESS]
- **ID-201. Pool creation:** Pool NFT and liquidity token minting policies don't validate destination address or initial pool datum state during minting, relying entirely on off-chain code for correct initialization.<br><ins>Detectable pattern</ins>: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION] [UNVALIDATED-DATUM]
- **ID-302 Destroy allows hijacking the pool:** Not checking the output datum and value when the pool is spent with the Destroy redeemer.<br><ins>Detectable pattern</ins>: output without datum validation [UNVALIDATED-DATUM]
- **ID-301. Zero spam:** Validator allows Swap, Deposit, and Redeem transactions with zero amounts, enabling DoS attacks through repeated spam. Requires semantic understanding of whether operations with zero amounts are intentional.<br><ins>Detectable pattern</ins>: operations that don't change state [UNCHANGED-STATE]
- **ID-401. DAO can change the pool unrestricted:** Minting policy uses input index from redeemer to identify pool UTxO but doesn't verify the presence of pool NFT at that index, allowing attackers to substitute fake UTxO at pool address with arbitrary datum.<br><ins>Detectable pattern</ins>: input selection by redeemer-provided index without validating presence of identifying NFT at that input [UNVALIDATED-INPUT-INDEX]

**May be relevant**

- **ID-108. Optimize output datum validation:** Validator compares output datum field-by-field instead of constructing expected datum and comparing as whole.<br><ins>Detectable pattern</ins>: multiple individual field comparisons between datums
- **ID-114. Unnecessary datum re-construction:** Function extracts fields from input datum, reconstructs new datum from those fields, then compares to output datum instead of direct equality check.<br><ins>Detectable pattern</ins>: datum fields extracted and immediately used to reconstruct identical datum structure
- **ID-202. Fee consistency checks:** Protocol validates individual bounds for feeNum and treasuryFee but doesn't enforce their relationship, allowing feeNum - treasuryFee to become negative and break all swap transactions.<br><ins>Detectable pattern</ins>: subtraction without relational constraint (eg. treasuryFee >= feeNum)
- **ID-303. Lack of checking of the purpose field of the staking validator:** Staking validator doesn't validate ScriptPurpose field, allowing unintended execution of delegate or deregister actions instead of reward withdrawal, with deregistration invalidating DAO policy checks.<br><ins>Detectable pattern</ins>: staking validator without purpose field validation in script context

**Not relevant**

- **ID-501 DAO can withdraw all user funds:** The minting policy (PFeeSwitch) doesn’t check that the output pool UTxO assets to exchange (treasuryX and treasuryY) are not negative in the validateTreasuryWithdraw function
- **ID-102 Potential of unsafe order types:** There is no restriction on the types of orders the pool can execute, anyone can implement their own “order” smart contract or interact directly with the pool. An issue may arise for incorrectly implemented order contracts that promise to execute a desired operation, but due to bug/malicious intent, the contract might not execute.
- **ID-103 Order types may be vulnerable to frontrunning:** There is no restriction on the types of orders the pool can execute, anyone can implement their own “order” smart contract or interact directly with the pool. This makes it possible for executors to front-run user swaps and other operations if the order type does not prevent this
- **ID-104 Confusing variable naming:** Variables feeNum and feeDen are named misleadingly - feeNum / feeDen represents the portion user receives after fees rather than the fee ratio itself, potentially causing developer confusion. Code clarity/naming issue
- **ID-105 Setting invalid/empty DAOPolicy:** DAOPolicy field in the datum is used to check the execution of dao actions on the pool. Setting the DAOPolicy value to a wrong value or setting it to an empty list will make accessing the treasury amounts in the pool impossible
- **ID-107 DAO signers:** The list of accepted signatories cannot be examined by looking at the onchain data and certain operations can change the DAOPolicy. There is no way to check the actual list of a DAOPolicy
- **ID-109. Treasury fee denominator must equal to fee denominator:** Code assumes feeDen == treasuryFeeDen in calculations but uses two separate constants with same value, risking calculation errors if one changes without the other. Requires semantic understanding that these values must remain equal for calculations to work correctly
- **ID-111. Unnecessary if in correctLpTokenDelta:** Conditional logic made redundant by subsequent constraints. Requires constraint solving and data flow analysis to detect unreachable branches
- **ID-110 Redundant rounding:** Redundant call to the rounding function
- **ID-112 Fee consistency checks:** Upper limit of the swap fees should be modified to a reasonable price
- **ID-113 Fixed Balance pool weights:** Implementation only supports two-token pools with fixed 20:80 ratio instead of customizable weights. Business logic issue

## Liqwid v1

**Auditor**: VacuumLabs

**Auditee**: Liqwid Labs

**Description**: An open source and non-custodial liquidity protocol for interest rate curves based on lender supply and borrower demand of the underlying Cardano native asset. Each Cardano native asset that is accepted by the protocol has its own liquidity pool, a.k.a market. Each of these markets has its own interest rate, and this is determined by the supply of the underlying asset along with the borrowing demand for the asset. If borrowing demand is low, and supplied liquidity is high, then interest rates are low; and vice versa.

### Findings

**Relevant**

- **LIQV1-001. Retrieving the collateral without repaying allows for draining a market**: Borrow token minting policy doesn't verify the output address where the new borrow token is sent, allowing attacker to redirect collateral to a malicious script instead of the proper loan validator.<br><ins>Detectable pattern</ins>: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **LIQV1-003. Loan collateral can be stolen by overwriting the loan datum**: Conditional logic skips datum validation on continuing output, allowing arbitrary datum modification.<br><ins>Detectable pattern</ins>: continuing output at same script address without any datum validation (neither full equality nor field-specific checks) [UNVALIDATED-CONTINUING-DATUM]

**Not relevant**

- **LIQV1-002. Supply can be stolen while batching**: Both branches of conditional check minAda instead of using different values for Ada vs non-Ada markets. Already caught by standard tooling
- **LIQV1-004. Minting additional borrow tokens allows draining markets**: Validator filters out UTxOs containing multiple tokens without validating them. Requires understanding protocol invariants about token quantities per UTxO
- **LIQV1-005. Division by zero—collateral can be locked forever**: Division operation in interest calculation can reach zero divisor when principal and interest are both zero, causing validation failure. Requires understanding reachable states
- **LIQV1-006. Centralization**: Privileged keys can bypass validation logic for critical operations. Protocol governance and trust model design issue
- **LIQV1-007. Market can be drained by tricking the checkQTokenRate**: Validation checks inequality based on one variable's sign without validating sign consistency between related variables, allowing invalid state transitions. Requires understanding reachable states
- **LIQV1-101. Incorrect reserve calculation can require liquidity deposit for batching**: Calculation uses intermediate variable values before updates rather than final computed values. Logic bug requiring semantic understanding of intended calculation
- **LIQV1-102. A quick loan is without interest—staking rewards can be stolen**: Interest calculation allows borrowing for short periods without charges. Protocol economic design issue
- **LIQV1-103. Staking rewards can be withdrawn while redeeming liquidity**: Conditional validation logic allows bypassing staking reward checks in certain branches. Logic bug and consequence of - \*\*LIQV1-007
- **LIQV1-201. Interest is accrued even after a loan is paid back**: Interest calculation continues accumulating rather than stopping at loan repayment. Logic bug requiring understanding business logic
- **LIQV1-202. Negative interest is outstanding after a loan is paid back**: Same value subtracted twice in related calculations, causing incorrect result. Logic bug
- **LIQV1-203. Oracle exchange rate is a simple number**: Price oracle design doesn't account for asset's liquidity. Protocol design issue
- **LIQV1-204. Reserve only decreases**: Function calculates value to increase reserve but never applies it, only implementing decrease logic. Incomplete implementation requiring understanding of business logic
- **LIQV1-205. Attacker can block Action UTxOs**: Multiple users competing for the same UTxO causes transaction conflicts requiring retries. Concurrency design issue
- **LIQV1-206. Loan interest rounded down leads to an accumulated interest**: Rounding individual values before aggregation causes sum to differ from expected aggregate. Requires understanding business logic to determine if pattern is problematic
- **LIQV1-207. Attacker can block the batching process – minBatchSize requirement is not enforced**: Validation checks cumulative actions batched across all transactions rather than requiring minimum per transaction. Logic bug
- **LIQV1-208. Staking delegation is controlled by a single key**: Single private key can redirect staking to any pool, potentially capturing all rewards. Protocol governance issue
- **LIQV1-209. Oracle is controlled by a single key**: Single private key controls price updates with no expiration mechanism. Protocol governance issue
- **LIQV1-301. The maxLoan threshold can be exceeded**: Validation includes loan amount in both numerator and denominator of ratio check. Requires understanding whether ratio should be checked against existing or new total
- **LIQV1-302. compoundingPeriod should be considered part of the interest model**: Parameter affecting interest calculations stored separately from interest model structure. Code organization issue
- **LIQV1-303. Attacker can slow down the batching process**: Single Batch UTxO creates congestion allowing minimal batching to slow the system. Protocol design issue
- **LIQV1-304. Attacker can hide liquidity during the batching process**: Batching high-liquidity UTxOs first locks significant liquidity for extended periods. Protocol design issue
- **LIQV1-305. Close factor below 100% causes never-ending liquidations**: Close factor can create infinite liquidation series with perpetual remainder. Protocol design issue
- **LIQV1-306. Batching requires batchers to pay min-ADA**: Batchers must provide min-ADA for output UTxOs, disincentivizing decentralized participation. Protocol economics issue
- **LIQV1-401. Variable name “supply” in some functions includes the reserve as well**: Variables named "supply" actually contain supply plus reserve. Naming issue
- **LIQV1-402. Functions checkBatchTime appear twice with different semantics**: Two functions named checkBatchTime perform different validation checks. Naming issue
- **LIQV1-403. Variable liquidationDiscount has a misleading name**: Parameter named liquidationDiscount represents 1 - discount rather than the discount itself. Naming issue
- **LIQV1-404. Variable maxBatchTime has confusing name**: Parameter maxBatchTime limits action time rather than batching time. Naming issue
- **LIQV1-405. Functions in Helpers.hs are not documented properly**: Helper functions lack clear documentation and meaningful names. Documentation issue
- **LIQV1-406. The use of a -Interest suffix leads to unclear variable names**: Suffix naming convention creates confusing names like interestInterest and misleading names like totalOwedInterest. Naming issue
- **LIQV1-407. The variable supplyDiff has multiple meanings**: Variable supplyDiff has different semantic meanings in Action UTxOs versus Batch UTxOs. Naming issue
- **LIQV1-408. Reserve field in Action datum may be imprecise**: Separate rounding of reserve datum field and actual value causes discrepancies. Logic bug requiring understanding business logic
- **LIQV1-409. Other code best practices**: Multiple code quality issues including misleading names, duplicate code, dead code, and outdated comments. Code quality and maintainability issues already caught by standard tooling

## WingRiders v2

**Auditor**: VacuumLabs

**Auditee**: WingRiders

**Description**: WingRiders v2 is an extension and rewrite of the WingRiders DEX supporting both constant-product and stableswap AMM pools. The protocol heavily relies on authority tokens, datums, and scripted accounting, which introduces multiple risks related to unchecked parameter updates, invariant enforcement, and output composition.

### Findings

**Relevant**

- **WR2-201. Anyone can block script fee beneficiary funds**: Validator doesn't validate datum attached to outputs sent to script addresses, allowing arbitrary datums that may prevent beneficiary script from spending.<br><ins>Detectable pattern</ins>: outputs to script addresses without datum validation [UNVALIDATED-DATUM]
- **WR2-303 Additional tokens may make compensation output unspendable**: Compensation outputs allowed arbitrary extra tokens, potentially breaking downstream scripts.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **WR2-405. Zap-in swapA is not sanitized**: Numeric redeemer parameter used directly in arithmetic operations without any validation.<br><ins>Detectable pattern</ins>: redeemer fields (untrusted user input) used without validation

**Not relevant**

- **WR2-001. Fee authority can manipulate treasury reserve fields**: Validator allows fee authority to update pool fees but doesn't check that treasury reserve datum fields remain unchanged, enabling manipulation of reserve values. Requires semantic understanding of which datum fields should be immutable
- **WR2-002. Requests can be unlocked by agents**: Function uses hardcoded constant as initial index value, allowing attacker to use redeemer indices that cause duplicate input selection. Logic error
- **WR2-003. Malicious fee change can block all liquidity**: Fee authorities can set fee values to extremely large numbers that cause subsequent transactions to exceed transaction size limits, locking all liquidity. Requires understanding the protocol's business logic
- **WR2-101. Emergency withdrawal in stableswap pool breaks invariant**: Update in invariant parameter in pool datum isn't validated. Requires understanding which datum fields should be updated for specific operations
- **WR2-102. Stableswap zap-in formulas are wrong about non-pool fees**: Formulas from previous protocol version not updated to reflect new fee structure, causing incorrect fee calculations. Protocol design issue
- **WR2-103. Adding staking rewards to stableswap pool breaks invariant**: Update in invariant parameter in pool datum isn't validated. Requires understanding which datum fields should be updated for specific operations
- **WR2-301. Fee auth token holders can withdraw accumulated project and reserve fees**: Fee authority can change beneficiary addresses and immediately withdraw all accumulated fees to new addresses. Protocol governance and trust model issue
- **WR2-302. pinitialStableswapPoolCorrect does not check agentFeeAda value**: Pool creation doesn't validate fee value, allowing arbitrary values including negative numbers. Requires understanding which datum fields should have bounds validation
- **WR2-401. Dead code**: Unused functions and types throughout the codebase. Already caught by standard tooling
- **WR2-402. Documentation**: Incorrect, outdated, and missing documentation throughout codebase
- **WR2-403. Naming**: Unclear, inconsistent, and misleading variable and function names throughout codebase
- **WR2-404. feeInBasis semantics and naming**: Constant semantics changed from previous version but naming and documentation not updated to reflect new meaning. Code quality and documentation issue

## MELD Token Migration

**Auditor**: VacuumLabs

**Auditee**: Liqwid Labs

**Description**: The protocol migrates old Meld tokens to new ones at a 1:1 ratio by requiring users to lock their old tokens in Locker UTxOs, after which they can mint the same amount of new tokens. Since the old token policy does not allow burning, the locked tokens are permanently frozen in Archive UTxOs. Locker UTxOs can later be batched into Cleanup transactions that move all old tokens into Archive UTxOs, returning the extra Ada to users while keeping the Locker fee as a reward for the batcher. Archive UTxOs can be public or controlled by a specific public key, and old tokens can only move between Archives, enabling secure consolidation and recovery of min-Ada when merging Archives with proper authorization.

### Findings

**Relevant**

- **MTOK-202. Filling up the UTxOs with arbitrary tokens**: Validator doesn't restrict which tokens can be included in Archive and Locker UTxOs, allowing attackers to bloat them with arbitrary tokens and cause transactions to hit size limits.<br><ins>Detectable pattern</ins>: subset value validation instead of exact equality check [TRASH-TOKENS]
- **MTOK-302. Denial of service of the Archive**: Archive UTxO can be included in arbitrary transactions without performing its intended function (adding tokens or processing Lockers), enabling DoS attacks.<br><ins>Detectable pattern</ins>: operations that succeed without modifying state [UNCHANGED-STATE]

**Worth Noting**

- **MTOK-402. Staking credential of an Archive disallows Archive merging**: Validator compares full addresses including staking credentials, preventing merging of Archives with different staking parts. Requires understanding whether staking credentials should be part of address validation. Although this appears to be a semantic issue, worth investigating as this may contradict an existing rule about address comparison

```haskell
plustan04 :: Inspection
plustan04 = mkAntiPatternInspection (Id "PLU-STAN-04") "Usage of eq instance of ScriptHash/PublicKeyHash/Credential"
    (FindAst pat)
    & descriptionL .~ "Usage of eq instance of script-hash / pubkeyhash / payment credential "
    & solutionL .~
        [ "Potential staking value theft might want to prefer eq comparison of address" ]
    & severityL .~ Warning
```

**Not relevant**

- **MTOK-201 Merging of Archive UTxOs leads to unwarranted gains**: Whoever performs the merging of Archive UTxOs can claim the min-ADA from the inputs for themselves. Economic/fairness issue
- **MTOK-301 High value of ldRefundAMount freezes min-Ada**: ldRefundAmount datum field determines the amount of Ada that is refunded during the Cleanup tx, but if the amount is set higher than the amount of Ada present, whoever does the Cleanup has to provide the extra Ada. Requires understanding what datum fields represent in business logic
- **MTOK-401 Locker fee can disincentivize the use of the standard workflow**: The standard workflow leads to a total cost equal to the Locker fee plus tx fees. Alternatively it could be just the min-Ada and tx fees. (Setting Locker Fees to high can disincentivize users to use the intended standard flow). Protocol economics issue
