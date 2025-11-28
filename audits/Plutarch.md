# Plutarch Audit Summaries

## Agora, Agora pro

**Description**
From: VacuumLabs To: Liqwid Labs

**Summary:** Agora is a Cardano native on-chain governance module written in Plutarch. It provides the necessary building blocks for any on-chain decentralized application to adopt and extend itself into a DAO-governed application. It consists of an open-source module (Agora) and a proprietary extension (Agora-Pro)

**Findings**

**May be relevant**

- **AGO-105. Stake ST token name not checked:** Stake validator validates currency symbol but not token name of stake ST, allowing attacker to use tokens with wrong names and fake vote history. Detectable pattern: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AGO-104. Attacker can fail any voted-on/locked proposal:** Validator doesn't check transaction validity range length. Requires understanding acceptable validity range lengths
- **AGO-203. Stakes can be frozen effectively forever by the multisig entity:** Validator doesn't check transaction validity range length. Requires understanding acceptable validity range lengths
- **AGO-204. Governor can be DoSed by creating Proposals without passing min GT limit:** Function validates currency symbol but not token name of stake ST. Detectable pattern: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AGO-307. Proposal can be DoSed by using UnlockStake with relevant cosigners' stakes:** Validator allows operation that passes validation but doesn't change state, enabling DoS attacks. Detectable pattern: operations that succeed without modifying state [UNCHANGED-STATE]
- **AGO-401. Staking credential is undefined:** Protocol doesn't define staking credentials for script UTxOs, potentially missing staking rewards and complicating off-chain UTxO discovery. Detectable pattern: script addresses without defined staking credentials

**Not relevant**

- **AGO-001. Stake state token can be taken away:** Stake validator doesn't enforce stake state token is burned when stake is destroyed, allowing attacker to reuse token in malicious stake UTxO with inflated voting power. Requires semantic understanding of protocol lifecycle and which tokens should be burned versus returned in specific operations
- **AGO-002. Acting on behalf of delegatee role + Unlocking delegated stakes:** Validator determines signature context globally across all inputs instead of per-input, allowing stake owner to act on others' stakes with same delegatee. Requires semantic understanding of authorization model
- **AGO-003. Fake proposal can be created and GAT minted without any voting happening:** Proposal minting policy checks governor ST presence but doesn't validate redeemer. Requires semantic understanding of which redeemers should permit which token minting operations
- **AGO-004. Multiple GATs can be minted into fake scripts:** Authority token minting policy doesn't validate the quantity of tokens minted per output. Requires semantic understanding of intended token distribution model
- **AGO-005. Stake lock can be removed without retracting votes:** Bug in function with inverted filter logic causes locks from unrelated proposals to be removed when retracting votes. Logic bug requiring semantic understanding of intended behavior
- **AGO-101. Delegatee can steal delegated inputs:** Function validates input-output correspondence but checks subset instead of equality, allowing outputs to be missing. Requires semantic understanding that inputs and outputs should correspond 1:1
- **AGO-102. Stake ST minting policy does not check lockedBy:** Minting policy doesn't validate datum field is empty when creating new stake. Requires semantic understanding of correct initial state for datum fields
- **AGO-103. pdestroy doesn't check outputs:** Destroy redeemer doesn't validate absence of stake outputs, allowing attacker to create new stake UTxO with a datum that doesn’t pass certain checks. Requires semantic understanding of which operations should be terminal versus those that produce continuing outputs
- **AGO-106. Stake can retract all votes in its cooldown period:** Cooldown validation uses filter instead of guard. Requires semantic understanding of whether code is selecting valid items to process versus validating all items must be valid
- **AGO-201. GATs are equal in the potential damage they can cause:** All governance authority tokens have equal power regardless of action criticality, allowing any passed proposal to fully compromise the system. Protocol governance issue
- **AGO-202. getStakeDatum is staking credential sensitive:** Function filters stake UTxOs by address including staking credential, causing each stake to see only a subset of stakes with matching credential. Requires semantic understanding of protocol's staking credential model and intended validation scope
- **AGO-301. Ambiguity in destroying multiple stakes:** Stake validator allows destroying multiple stakes per transaction but minting policy doesn't allow burning multiple stake STs at once. Protocol design inconsistency
- **AGO-302. Some strict inequalities should not be strict:** Threshold comparisons use strict inequalities where inclusive inequalities are expected. Requires understanding business logic
- **AGO-303. tooLate in code can also mean early:** Logic bug where proposal actions before startingTime are incorrectly treated as "too late" instead of "too early", allowing premature proposal finalization
- **AGO-304. Other minor inconsistencies:** Documentation and code quality issues
- **AGO-305. Other naming suggestions:** Code quality and documentation issues
- **AGO-306. Proposal can be DoSed by using UnlockStake with no input stakes:** Redeemer allows operation without validating required stake inputs are present. Requires semantic understanding of which redeemers require which inputs
- **AGO-308. Proposal can be DoSed by voting and unlocking repeatedly:** Attacker can repeatedly vote and unlock stake to DoS proposal by constantly changing the proposal UTxO, invalidating other users' transactions. Protocol design issue
- **AGO-309. Incorrect token references across the code:** Variable names and comments incorrectly reference which tokens are used
- **AGO-402. PUnlock:** Ambiguous proposal redeemer name: Redeemer named PUnlock suggests unlocking proposal but actually unlocks stakes. Naming issue
- **AGO-403. Delegatee cannot vote with delegated and own stakes in one transaction:** Validation requires all stakes to have the same owner or same delegatee, preventing mixed transactions. Protocol design issue

## Liqwid v1

**Description**
From: VacuumLabs To: Liqwid Labs

**Summary:** An open source and non-custodial liquidity protocol for interest rate curves based on lender supply and borrower demand of the underlying Cardano native asset. Each Cardano native asset that is accepted by the protocol has its own liquidity pool, a.k.a market. Each of these markets has its own interest rate, and this is determined by the supply of the underlying asset along with the borrowing demand for the asset. If borrowing demand is low, and supplied liquidity is high, then interest rates are low; and vice versa.

**Findings**

**May be relevant**
- **LIQV1-001. Retrieving the collateral without repaying allows for draining a market:** Borrow token minting policy doesn't verify the output address where the new borrow token is sent, allowing attacker to redirect collateral to a malicious script instead of the proper loan validator. Detectable pattern: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **LIQV1-003. Loan collateral can be stolen by overwriting the loan datum:** Conditional logic skips datum validation on continuing output, allowing arbitrary datum modification. Detectable pattern: continuing output at same script address without any datum validation (neither full equality nor field-specific checks) [UNVALIDATED-CONTINUING-DATUM]

**Not relevant**

- **LIQV1-002. Supply can be stolen while batching:** Both branches of conditional check minAda instead of using different values for Ada vs non-Ada markets. Already caught by standard tooling
- **LIQV1-004. Minting additional borrow tokens allows draining markets:** Validator filters out UTxOs containing multiple tokens without validating them. Requires understanding protocol invariants about token quantities per UTxO
- **LIQV1-005. Division by zero—collateral can be locked forever:** Division operation in interest calculation can reach zero divisor when principal and interest are both zero, causing validation failure. Requires understanding reachable states
- **LIQV1-006. Centralization:** Privileged keys can bypass validation logic for critical operations. Protocol governance and trust model design issue
- **LIQV1-007. Market can be drained by tricking the checkQTokenRate:** Validation checks inequality based on one variable's sign without validating sign consistency between related variables, allowing invalid state transitions. Requires understanding reachable states
- **LIQV1-101. Incorrect reserve calculation can require liquidity deposit for batching:** Calculation uses intermediate variable values before updates rather than final computed values. Logic bug requiring semantic understanding of intended calculation
- **LIQV1-102. A quick loan is without interest—staking rewards can be stolen:** Interest calculation allows borrowing for short periods without charges. Protocol economic design issue
- **LIQV1-103. Staking rewards can be withdrawn while redeeming liquidity:** Conditional validation logic allows bypassing staking reward checks in certain branches. Logic bug and consequence of - **LIQV1-007
- **LIQV1-201. Interest is accrued even after a loan is paid back:** Interest calculation continues accumulating rather than stopping at loan repayment. Logic bug requiring understanding business logic
- **LIQV1-202. Negative interest is outstanding after a loan is paid back:** Same value subtracted twice in related calculations, causing incorrect result. Logic bug
- **LIQV1-203. Oracle exchange rate is a simple number:** Price oracle design doesn't account for asset's liquidity. Protocol design issue
- **LIQV1-204. Reserve only decreases:** Function calculates value to increase reserve but never applies it, only implementing decrease logic. Incomplete implementation requiring understanding of business logic
- **LIQV1-205. Attacker can block Action UTxOs:** Multiple users competing for the same UTxO causes transaction conflicts requiring retries. Concurrency design issue
- **LIQV1-206. Loan interest rounded down leads to an accumulated interest:** Rounding individual values before aggregation causes sum to differ from expected aggregate. Requires understanding business logic to determine if pattern is problematic
- **LIQV1-207. Attacker can block the batching process – minBatchSize requirement is not enforced:** Validation checks cumulative actions batched across all transactions rather than requiring minimum per transaction. Logic bug
- **LIQV1-208. Staking delegation is controlled by a single key:** Single private key can redirect staking to any pool, potentially capturing all rewards. Protocol governance issue
- **LIQV1-209. Oracle is controlled by a single key:** Single private key controls price updates with no expiration mechanism. Protocol governance issue
- **LIQV1-301. The maxLoan threshold can be exceeded:** Validation includes loan amount in both numerator and denominator of ratio check. Requires understanding whether ratio should be checked against existing or new total
- **LIQV1-302. compoundingPeriod should be considered part of the interest model:** Parameter affecting interest calculations stored separately from interest model structure. Code organization issue
- **LIQV1-303. Attacker can slow down the batching process:** Single Batch UTxO creates congestion allowing minimal batching to slow the system. Protocol design issue
- **LIQV1-304. Attacker can hide liquidity during the batching process:** Batching high-liquidity UTxOs first locks significant liquidity for extended periods. Protocol design issue
- **LIQV1-305. Close factor below 100% causes never-ending liquidations:** Close factor can create infinite liquidation series with perpetual remainder. Protocol design issue
- **LIQV1-306. Batching requires batchers to pay min-ADA:** Batchers must provide min-ADA for output UTxOs, disincentivizing decentralized participation. Protocol economics issue
- **LIQV1-401. Variable name “supply” in some functions includes the reserve as well:** Variables named "supply" actually contain supply plus reserve. Naming issue
- **LIQV1-402. Functions checkBatchTime appear twice with different semantics:** Two functions named checkBatchTime perform different validation checks. Naming issue
- **LIQV1-403. Variable liquidationDiscount has a misleading name:** Parameter named liquidationDiscount represents 1 - discount rather than the discount itself. Naming issue
- **LIQV1-404. Variable maxBatchTime has confusing name:** Parameter maxBatchTime limits action time rather than batching time. Naming issue
- **LIQV1-405. Functions in Helpers.hs are not documented properly:** Helper functions lack clear documentation and meaningful names. Documentation issue
- **LIQV1-406. The use of a -Interest suffix leads to unclear variable names:** Suffix naming convention creates confusing names like interestInterest and misleading names like totalOwedInterest. Naming issue
- **LIQV1-407. The variable supplyDiff has multiple meanings:** Variable supplyDiff has different semantic meanings in Action UTxOs versus Batch UTxOs. Naming issue
- **LIQV1-408. Reserve field in Action datum may be imprecise:** Separate rounding of reserve datum field and actual value causes discrepancies. Logic bug requiring understanding business logic
- **LIQV1-409. Other code best practices:** Multiple code quality issues including misleading names, duplicate code, dead code, and outdated comments. Code quality and maintainability issues already caught by standard tooling
