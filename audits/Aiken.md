# Aiken Audit Summaries

## Overview

This document summarizes findings from **36 Aiken audits** in the Cardano ecosystem.

**Findings Classification**:

- **Relevant findings**: 98
- **May be relevant findings**: 56
- **Not relevant findings**: 273
- **Total findings**: 429

### Common Patterns

Common issues in Aiken smart contracts include:

1. **Unvalidated Datum Fields (27 occurrences) - [UNVALIDATED-DATUM], [PARTIAL-UNVALIDATED-DATUM]:** Validators that create or update outputs without properly validating datum contents, allowing arbitrary or malicious data that can break subsequent operations or enable attacks.

2. **Trash Tokens / Subset Value Validation (20 occurrences) - [TRASH-TOKENS]:** Validators using subset checks instead of exact equality for value validation, allowing attackers to bloat UTxOs with arbitrary tokens, increasing costs and enabling potential exploits.

3. **Missing Address Validation (15 occurrences) - [MISSING-ADDRESS-VALIDATION]:** Minting policies and validators that fail to verify the destination address of minted tokens or continuing outputs, allowing attackers to redirect assets to arbitrary addresses.

4. **Incomplete Token Validation (17 occurrences) - [INCOMPLETE-TOKEN-VALIDATION]:** Validators that check only some components of token tuples (currency symbol, token name, or quantity) while leaving others unchecked, allowing attackers to mint unauthorized tokens with the same name but different policy or bypass burning requirements.

5. **Unvalidated Reference Script Field (13 occurrences) - [UNVALIDATED-REFERENCE-SCRIPT]:** Outputs that don't validate the reference script field, allowing arbitrary reference scripts to be attached, which can significantly increase future transaction fees.

## BookToken

**Auditor**: Tweag

**Auditee**: Book.io

**Description**:The BOOK token platform and exchange is the first of its kind by using reading to “mine” for tokens. In order to obtain `$BOOK` tokens, users must purchase NFT books – and then read them. Readers are ONLY rewarded with native `$BOOK` tokens based on how much they read. The more you read, the more `$BOOK` you can earn.
These Cardano-native tokens are redeemable on the platform to purchase new eBooks and Audiobooks – and also transferrable from the `$BOOK` Token platform to other compatible wallets and crypto exchanges.

### Findings

**May be relevant**

- **2.2.4.3. Some helper functions impact compiled script size**: Helper functions are trivial wrappers that could be inlined.<br><ins>Detectable pattern</ins>: functions that only pattern match or call another function with fixed arguments. Should check if there is some Plinth tool that covers this issue or not

**Not relevant**

- **2.2.1.1. The reference token is not an NFT**: Reference token is burned and reminted on updates instead of using one-shot NFT, adding transaction costs and relying on admin to avoid multiple tokens. Requires semantic understanding of intended token lifecycle
- **2.2.1.2. Obsolete tokens are not burned**: Obsolete tokens sent to script address instead of being burned via minting policy. Requires semantic understanding of intended token lifecycle and destination script purpose
- **2.2.1.3. Custom failure scripts have no upside over the canonical failure script**: Parametrized scripts used instead of canonical failure script for locking obsolete tokens. Requires semantic understanding of script purpose and whether parametrization provides value
- **2.2.2.1. New tokens can be stolen by abusing an undesired rounding-up operation**: Integer division with negative dividend rounds toward negative infinity, causing amounts to round up in absolute value. Requires semantic understanding of correct rounding direction for the business logic
- **2.2.3.1. Unlock transaction is underspecified**: Validator logic for unlock transactions not explicitly documented. Documentation issue
- **2.2.3.2. Swap transaction is underspecified**: Validator logic for swap transactions not explicitly documented. Documentation issue
- **2.2.4.1. Conditionals could be avoided using short-circuiting in boolean conjunctions**: Suggests rewriting conditionals using boolean conjunction for readability. Code style issue already caught by standard tooling
- **2.2.4.2. Unnecessary named intermediates**: Excessive use of intermediate variables may reduce code clarity. Code style issue
- **2.2.4.4. Comments are lacking in the code**: Source code would benefit from more description comments. Documentation issue

## MinSwap Dex

**Auditor**: Tweag

**Auditee**: MinSwap Labs

**Description**: Minswap is a Cardano DEX where LPs can see the potential APY of a pool before providing liquidity, and then make an informed choice about which pools they chose to provide liquidity to. MinSwap DEX also employs an automated yield farm strategy, which helps LPs to rebalance the liquidity they provided into the most efficient pools.

### Findings

**Relevant**

- **2.2.1.3. Unauthorized Hijacking of Pools Funds**: Validator selects continuing output by NFT presence without verifying destination address, allowing pool funds to be redirected to attacker's script.<br><ins>Detectable pattern</ins>: continuing output selected by token presence without address validation [MISSING-ADDRESS-VALIDATION]

**May be relevant**

- **2.2.1.1. Unauthorized Redeeming of Open Orders**: Validator checks for presence of other script inputs without verifying their redeemers, allowing authorization bypass.<br><ins>Detectable pattern</ins>: validation depends on other script inputs without checking txInfoRedeemers, though determining whether specific redeemers are required for validation requires semantic understanding [UNCHECKED-REDEEMER]
- **2.2.1.2. LP Tokens Can Be Duplicated**: Minting policy checks for pool output with NFT but doesn't verify which redeemer was used on pool input, allowing unauthorized minting.<br><ins>Detectable pattern</ins>: minting policy depends on other script inputs/outputs without checking txInfoRedeemers, though determining whether specific redeemers are required for validation requires semantic understanding [UNCHECKED-REDEEMER]

**Not relevant**

- **2.2.2.1. Batchers Can Choose Batching Order**: On-chain code doesn't enforce chronological order processing. Requires semantic understanding of whether order matters for protocol fairness
- **2.2.2.2. Batcher Is Not Allowed to Apply Their Own Orders**: Code filters out orders from specific addresses when processing. Requires semantic understanding of whether address-based filtering is intentional or erroneous
- **2.2.3.1. Batchers Can Choose Pools**: Batchers can execute orders against any pool, including custom pools. Requires semantic understanding of intended protocol constraints
- **2.2.3.2. Batchers Licenses Cannot be Revoked**: Batcher licenses are irrevocable until expiration. Protocol governance issue
- **2.2.3.3. Assumptions on Batcher's Licenses Distribution**: Specification doesn't document how batcher licenses are distributed and renewed. Documentation issue
- **2.2.3.4. Pools Cannot Be Closed**: Pools can be created but not closed, permanently locking creator's minAda. Requires semantic understanding of intended pool lifecycle
- **2.2.4.1. Reliance on Indexes Into ScriptContexts' txInputs and txOutputs**: Validators use redeemer-provided indices to select inputs, relying on transaction ordering. Requires semantic understanding of whether input ordering constraints are necessary
- **2.2.4.2. Duplication of ScriptContext Definition**: Custom ScriptContext definition omits getContinuingOutputs function, contributing to vulnerability 2.2.1.3. Code organization and maintenance issue
- **2.2.4.3. Large Refactoring Opportunities**: Heavy code duplication across order validation functions could be refactored for better maintainability
- **2.2.4.4. Protocol Specification Lacking**: Documentation lacks formal specification defining correct behavior and operations

## Minswap AMM Dex v2

**Auditor**: Certik

**Auditee**: MinSwap Labs

**Description**: Minswap AMM V2 uses a constant-product formula (x \* y = k) to price trades, ensuring that larger trades incur progressively worse rates to protect liquidity. To handle concurrency on Cardano, it uses a batching system where user actions become orders that are collected and executed together in a liquidity pool. Only whitelisted wallets, called batchers, are allowed to trigger and process these batch transactions.

### Findings

**May be relevant**

- **MIN-01 Logical issue in fee settings**: Fee setting functions currently do not safeguard against the possibility of setting both the numerator and denominator of fees to zero.<br><ins>Detectable pattern</ins>: No check on denominator and numerator value being zero.

**Not relevant**

- **FAC-01 Creation of pools with invalid parameters**: Needs checks to ensure that reserve values and total liquidity fall within practical ranges. Requires understanding business logic to determine which are practical ranges.
- **VAL-01 Centralization related risks**: Centralized privileges or roles in the protocol could be improved via decentralized mechanisms
- **FAC-02 Pool creation allows complete asset withdrawal**: Should allow for the replenishment of pool liquidity. Requires understanding business logic.
- **AUT-01 Incorrect comment**: Code quality issue.
- **GLOBAL-01 Unit test documentation**: Documentation and code quality issues.
- **MIN-02 TODO comments**: Code quality issues.
- **ORD-01 Missing formulas for WithdrawImbalance and PartialSwap**: Code quality issues.
- **ORD-02 Missing check on io_ratio_denominator**: Validation checks io_ratio_numerator twice instead of checking both numerator and denominator, leaving denominator unchecked. Copy-paste error already caught by standard tooling
- **MAT-01 Potential optimization in math.calculate_withdraw_imbalance()**: Code quality issues.

## MinSwap Dex v2 reaudit

**Auditor**: Certik

**Auditee**: MinSwap Labs

**Description**: Minswap AMM V2 uses a constant-product formula (x \* y = k) to price trades, ensuring that larger trades incur progressively worse rates to protect liquidity. To handle concurrency on Cardano, it uses a batching system where user actions become orders that are collected and executed together in a liquidity pool. Only whitelisted wallets, called batchers, are allowed to trigger and process these batch transactions.

### Findings

**May be relevant**

- **ORD-02 Unoptimized check**: Using empty strings to verify if assets are ADA. It is recommended to use functions that do the verification.<br><ins>Detectable pattern</ins>: use of empty string to compare an asset name. Needs confirmation if there is an equivalent `utils.is_ada_asset() ` function in Plinth.

**Not relevant**

- **TYP-01 Potential for multiple roles per address**: There are no constraints to prevent a single address from being assigned multiple or even all roles.
- **MIN-01 Centralization related risks**
- **ORD-01 Missing check for batcher fee in donation orders**: Uncertainty about whether the batcher fee is correctly paid in all scenarios. Requires understanging of the business logic.

## Nuvola

**Auditor**: TxPipe

**Auditee**: Nuvola

**Description**: Nuvola is a DePIN aggregator, a first for Cardano. DePIN is short for Decentralized Physical Infrastructure Network, which brings cutting-edge Web 3.0 solutions to a broad number of real-world applications. Being decentralized at heart, DePIN companies pay their network supporters, such as node operators, rewards as incentive for maintaining the network.

### Findings

**Relevant**:

- **NUV-001. Reward Token can be stolen on Process Reward operation**: Validator uses subset value check on user output (not script address) allowing authorization token to escape burning and be reused.<br><ins>Detectable pattern</ins>: subset value validation without exact equality check [TRASH-TOKENS]
- **NUV-002. Rewards Claim UTxOs can be deleted without owner consent and fees stolen**: Minting policy validates token presence in outputs without checking mint amount, allowing burning to consume UTxOs without authorization.<br><ins>Detectable pattern</ins>: minting validation without checking token quantity sign [INCOMPLETE-TOKEN-VALIDATION]
- **NUV-003. Stake Tokens can be arbitrarily minted during the payback loan operation**: The payback loan validator for lender UTxO doesn’t restrict extra stake tokens, so arbitrary stake tokens can be minted and stored.<br><ins>Detectable pattern</ins>: minting without quantity validation and subset value validation [INCOMPLETE-TOKEN-VALIDATION] [TRASH-TOKENS]
- **NUV-201 Reward Claim UTxOs can be created with no or more than one token**: ClaimReward creation does not enforce that exactly one reward token is present. <br><ins>Detectable pattern</ins>: validation without exact equality check on values. [INCOMPLETE-TOKEN-VALIDATION]
- **NUV-204. Prevent inclusion of reference scripts**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **NUV-303. Claim reward tokens can be minted with arbitrary token name**: Minting policy validates expected token name but doesn't restrict minting other token names under the same currency symbol.<br><ins>Detectable pattern</ins>: token name validation without restricting other names [INCOMPLETE-TOKEN-VALIDATION]
- **NUV-305. Trash tokens can be added to multiple UTxOs**: Validator checks required tokens are present but doesn't restrict additional tokens, allowing arbitrary tokens to bloat UTxOs and increase costs.<br><ins>Detectable pattern</ins>: subset value validation without exact equality check [TRASH-TOKENS]

**May be relevant**

- **NUV-202. Missing checks for some LendingDatum fields**: Datum fields not validated during creation, allowing unreasonable values.<br><ins>Detectable pattern</ins>: outputs with only partial datum validation [PARTIAL-UNVALIDATED-DATUM]
- **NUV-301. CreateLend spend redeemer can be used to remove a lend UTxO**: Wildcard pattern matching on redeemer allows unintended redeemer types to trigger logic.<br><ins>Detectable pattern</ins>: wildcard pattern matching on redeemer types

**Not relevant**

- **NUV-101. It is possible to create loans that have expired**: Validation uses inequality check instead of equality for expiration date, allowing past dates. Requires understanding business logic to determine correct comparison operator
- **NUV-102. Genesis stake ref can be forged on LoanDatum**: Datum field copied from input to output without validation that values match, allowing arbitrary data. Requires understanding intended use of datum and which datum fields must be preserved across operations
- **NUV-203. Staking credentials not being checked in two operations**: Script UTxO creation doesn't validate staking credential matches user's expected credential. Requires understanding protocol design and user expectations
- **NUV-302. Spendable loan UTxO without a loan token**: Spend validation requires token burning, making UTxOs without tokens permanently unspendable. Requires understanding business logic
- **NUV-304. Optimization for the Apply Loan operation**: Multiple opportunities to reduce computation costs through better data handling and removing redundant checks. Requires understanding performance implications and protocol-specific code structure

## Private Audit #02

### Findings

**May be relevant**

- **ID-01. Unvalidated Datum on Creation and Update**: UTxO datum is not checked on creation and update transactions. May require understanding if all datum fields actually need validation.<br><ins>Detectable pattern</ins>: outputs to script addresses without datum validation or with only partial datum validation [PARTIAL-UNVALIDATED-DATUM]

## Private Audit #03

### Findings

**Relevant**

- **ID-004. Missing checks in operation**: Multiple missing validations during creation operation: outputs can be created without required validation, identifier fields in datum not validated against token names, token destination not validated, datum fields trusted blindly.<br><ins>Detectable pattern</ins>: output creation with incomplete validation (multiple missing checks for address, datum fields, and token consistency) [MISSING-ADDRESS-VALIDATION] [PARTIAL-UNVALIDATED-DATUM]
- **ID-201. Prevent inclusion of reference scripts**: UTxOs can carry large reference scripts, inflating future transaction fees.<br><ins>Detectable pattern</ins>: output validation logic that ignores reference script field entirely [UNVALIDATED-REFERENCE-SCRIPT]
- **ID-202. Continuing output datum and reference scripts can change**: Validation checks output address and value don't change but doesn't validate datum and reference script fields remain unchanged.<br><ins>Detectable pattern</ins>: output validation without datum and reference script field checks [UNVALIDATED-DATUM] [UNVALIDATED-REFERENCE-SCRIPT]
- **ID-204. Missing checks on minting and updating operations**: Creation operation validates some datum fields but doesn't check list fields for duplicates or validate some fields have reasonable values. Update operation also missing these checks.<br><ins>Detectable pattern</ins>: lists without uniqueness validation and datum fields without bounds validation [LIST-UNIQUENESS] [PARTIAL-UNVALIDATED-DATUM]
- **ID-206. Arbitrary tokens can be added to values**: Validators check inclusion instead of equality, allowing token dust and UTxO bloat which might increase transaction size and fees.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **ID-203. Multiple tokens can be paid to the same UTxO**: Minting allows multiple tokens in one UTxO, breaking assumptions for subsequent operations.<br><ins>Detectable pattern</ins>: token quantity not validated when minting [INCOMPLETE-TOKEN-VALIDATION]

**May be relevant**

- **ID-001. Protocol tokens can be stolen**: Validator validates input contains required token but doesn't validate token is included in continuing output, allowing token to be leaked.<br><ins>Detectable pattern</ins>: singleton token validated in input but not in continuing output (warning)
- **ID-003. Consistency of certain fields not validated in operation**: Critical datum fields are not checked for consistency, allowing silent corruption of state.<br><ins>Detectable pattern</ins>: datum field read but not checked [PARTIAL-UNVALIDATED-DATUM]
- **ID-105. Missing validation in multiple operations**: Multiple fields not validated in continuing outputs, address filtering uses complete address instead of payment credential, value comparisons use >= instead of exact equality.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]
- **ID-302. Optimize expected datum verification**: Datum equality checks cast output data to specific type then compare, could be optimized by casting expected datum to Data and comparing directly.<br><ins>Detectable pattern</ins>: datum equality checks using upcast instead of downcast. Check if there is similar mechanism in Plinth.

## Private Audit #04

### Findings

**May be relevant**

- **ID-01. Sub-optimal cost for transactions**: Validator repeatedly searches input list for own script hash when processing multiple items.<br><ins>Detectable pattern</ins>: loop-invariant own-script-hash lookup repeated multiple times instead of caching result

## Private Audit #05

### Findings

**Relevant**

- **ID-01. Output value not validated**: Protocol operations don't validate output values, allowing arbitrary tokens to be added.<br><ins>Detectable pattern</ins>: output value validation without token restriction checks [TRASH-TOKENS]
- **ID-02. Unvalidated Reference Script Field**: Validator doesn't validate reference script fields in outputs, allowing arbitrary reference scripts to be attached and increasing future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

## Private Audit #06

### Findings

**Relevant**

- **ID-01. Multiple tokens can be minted**: Minting policy validates token presence without enforcing exactly one token is minted, allowing extra tokens to be created.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-02. Unvalidated Reference Script Field**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **ID-03. Trash Tokens Allowed**: Validator checks required tokens are present but doesn't restrict additional tokens, allowing arbitrary tokens to bloat UTxOs and increase costs.<br><ins>Detectable pattern</ins>: subset value validation without exact equality check [TRASH-TOKENS]

**May be relevant**

- **ID-04. Missing datum field validation**: Protocol outputs may store incorrect data in datum fields, making them unprocessable since certain fields have no validation.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

## Private Audit #07

### Findings

**Relevant**

- **ID-01. Incomplete Token Validation**: Validator checks token currency symbol without verifying token name, allowing wrong tokens from same policy to be used.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-02. Arbitrary Token Minting**: Minting policy allows minting whenever an authorization token appears in any input, even when coming from a script instead of authorized wallet. Combined with protocol threading allows arbitrary token minting.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]

## Private Audit #08

### Findings

**Relevant**

- **ID-01. Missing Datum Validation**: Validator creates output at script address without validating datum, potentially creating unspendable UTxO.<br><ins>Detectable pattern</ins>: output creation without datum validation [UNVALIDATED-DATUM]
- **ID-02. Missing Address and Datum Validation**: Validator creates output without validating address or datum, allowing tokens to be misdirected or creating unspendable outputs.<br><ins>Detectable pattern</ins>: output creation without address and datum validation [MISSING-ADDRESS-VALIDATION] [UNVALIDATED-DATUM]
- **ID-03. Unbounded Validity Range Exploitation**: Validator uses transaction's validity bound for calculations without restricting validity range length, allowing artificially inflated values and exploitation.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **ID-04. Operations Allow Invalid Initial State**: Protocol allows creating outputs with arbitrarily low start time without restricting validity interval length, enabling exploitation through accumulated values.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **ID-05. Token Quantity Not Validated**: Validator checks token presence without validating quantity > 0, allowing zero-quantity tokens to satisfy checks.<br><ins>Detectable pattern</ins>: token presence check without quantity validation [INCOMPLETE-TOKEN-VALIDATION]
- **ID-06. Protocol Outputs Can Include Trash Tokens**: Output creation lacks validation allowing arbitrary tokens that can be merged into valid UTxOs.<br><ins>Detectable pattern</ins>: output value validation without token restriction checks [TRASH-TOKENS]
- **ID-07. Output Contains Wrong Tokens**: Protocol operation does not validate output value.<br><ins>Detectable pattern</ins>: output value validation without token restriction checks [TRASH-TOKENS]
- **ID-08. Identifier Token Name Not Validated**: Validator does not validate the token name of the minted identifier token.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]

## Private Audit #09

### Findings

**Relevant**

- **ID-01. Incomplete Value Validation**: Validator checked only that the UTxO contained some non-zero value, but not what that value consisted of. An attacker could insert a useless dummy token to satisfy the validator, while providing less than the protocol-required minimum ADA.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-02. Unvalidated Destination Address**: Minting policy doesn't validate that minted tokens are sent to the correct validator address.<br><ins>Detectable pattern</ins>: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **ID-03. Trash Tokens on Update**: When updating a script UTxO, the validator allows extra native tokens to be added accidentally. This could permanently lock the UTxO and force redeployment.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check on continuing outputs [TRASH-TOKENS]

**May be relevant**

- **ID-04. Multiple Satisfaction Issue**: Validator filters inputs by token presence without validating count of matching UTxOs.<br><ins>Detectable pattern</ins>: absence of quantity validation after filtering for specific tokens could be flagged as warning [DOUBLE-SATISFACTION]

## Private Audit #10

### Findings

**Relevant**

- **ID-01. Exact Value Equality on Updates**: Pool value check uses exact value equality preventing ADA addition when datum size increases.<br><ins>Detectable pattern</ins>: exact equality on continuing output values without accounting for variable minUTxO requirements [STRICT-VALUE-EQUALITY]
- **ID-02. Unvalidated Reference Script Field**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

**May be relevant**

- **ID-03. Datum Fields Not Checked at Creation**: There are no checks on certain datum fields when creating outputs. As a result, it is possible to create outputs that do not satisfy protocol invariants.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

## Private Audit #11

### Findings

**Relevant**

- **ID-01. Unvalidated Output Datum**: Validator doesn't validate datum field of outputs to script addresses, allowing datum hashes which require preimages to spend.<br><ins>Detectable pattern</ins>: outputs to script addresses without datum validation [UNVALIDATED-DATUM]
- **ID-02. Unvalidated Reference Script Field**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **ID-03. Trash Tokens Allowed**: Validator checks required tokens are present but doesn't restrict additional tokens, allowing arbitrary tokens to bloat UTxOs and increase costs.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]

## Private Audit #12

### Findings

**Relevant**

- **ID-01. Exact Value Equality on Updates**: Pool value check uses exact value equality preventing ADA addition when datum size increases.<br><ins>Detectable pattern</ins>: exact equality on continuing output values without accounting for variable minUTxO requirements [STRICT-VALUE-EQUALITY]
- **ID-02. Unvalidated Reference Script Field**: Validator doesn't validate reference script field in script outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

## Private Audit #13

### Findings

**Relevant**

- **ID-01. Unvalidated Reference Script Field**: Validator doesn't validate reference script field in outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

## Private Audit #14

### Findings

**Relevant**

- **ID-01. Unvalidated Reference Script Field**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

## Splash Protocol Stableswap (Splash Dex - draft)

**Auditor**: AnastasiaLabs

**Auditee**: Spectrum Labs

**Description**: Splash is a decentralized open-source protocol for efficient market-making and trading on Cardano.

### Findings

**Relevant**

- **ID-201. Restricted token dust attack**: Validator checks input and output values that have same policy IDs but doesn't verify same token names within those policies, allowing attackers to add arbitrary tokens under the same policy.<br><ins>Detectable pattern</ins>: value comparison using only `policies()` equality without validating complete asset list (policy + token name pairs) [INCOMPLETE-TOKEN-VALIDATION] [TRASH-TOKENS]
- **ID-401. DAO can change the pool unrestricted**: Minting policy uses input index from redeemer to identify pool UTxO but doesn't verify the presence of pool NFT at that index, allowing attackers to substitute fake UTxO at pool address with arbitrary datum.<br><ins>Detectable pattern</ins>: input selection by redeemer-provided index without validating presence of identifying NFT at that input [UNVALIDATED-INPUT-INDEX]

**May be relevant**

- **ID-501. Wrong usage of DAO action validator script**: Pool validator expects DAO script hash to be used as staking credential, but DAO is implemented as spending validator (spend keyword) instead of staking validator (withdraw keyword), causing type mismatch that bypasses validation when executed in staking context.<br><ins>Detectable pattern</ins>: script hash used in staking context but corresponding validator declared with wrong validator type keyword
- **ID-301. Zero spam**: Validator allows Swap, Deposit, and Redeem transactions with zero amounts, enabling DoS attacks through repeated spam. It should ensure a minimal amount is actually transacted. Note: could potentially be detected as operations that don't change state [UNCHANGED-STATE]

**Not relevant**

- **ID-302. Pool invariant can be violated at dao change**: DAO action allows changing amplification parameter an2n without validating the stableswap invariant relationship with reserve amounts, causing inconsistent swap behavior. Requires understanding protocol’s business logic
- **ID-202. Inconsistent protocol fees**: Protocol validates individual bounds for lp_fee_num and protocol_fee_num but doesn't enforce their relationship, allowing lp_fee_num + protocol_fee_num > denom and breaking all swap transactions. Requires understanding business logic to determine which fields should have relational constraints
- **ID-101. Order types may be vulnerable to frontrunning**: Pool doesn't restrict which order contracts can interact with it. Design decision about permissionless interoperability
- **ID-102. Incorrect assumption in code**: Default branch in swap validation assumes specific delta conditions without checking them, allowing unexpected cases to execute. Logic bug requiring semantic understanding of valid swap cases and business logic

## AMM Dex v2

**Auditor**: Certik

**Auditee**: MinSwap Labs

**Description**: Minswap is a Decentralized Exchange (DEX). The purpose of a DEX is to enable permissionless trading of token pairs. For each swap, a fee is taken, which goes to the Liquidity Providers (LPs). Anyone can provide Liquidity as well, hence profits are decentralized.

### Findings

**Not relevant**

- **FAC-01. Creation of pools with invalid parameters**: Validator checks datum fields match calculated values but doesn't validate reserves and liquidity are positive, allowing creation of non-functional pools with zero reserves. Requires understanding business logic to determine which datum fields need bounds validation
- **MIN-01. Logical issue in fee settings**: Validator checks fee percentage ranges but doesn't prevent setting both numerator and denominator to zero, causing division by zero errors in fee calculations. Requires understanding business logic
- **VAL-01. Centralization related risks**: Privileged admin and batcher tokens grant special permissions that could be abused if compromised. Protocol governance and trust model design issue, not a code pattern
- **FAC-02. Pool creation allows complete asset withdrawal**: Pool mints LP tokens equal to sqrt(reserve_a \* reserve_b), allowing LPs to fully withdraw reserves and make the pool non-functional. Protocol economic design issue that requires understanding the business logic
- **ORD-02. Missing check on io_ratio_denominator**: Validation checks io_ratio_numerator twice instead of checking both numerator and denominator, leaving denominator unchecked. Copy-paste error already caught by standard tooling
- **AUT-01. Incorrect comment**: Comment placed in front of wrong redeemer type. Documentation issue
  GLOBAL-01. Unit test documentation: Project lacks comprehensive unit tests and test coverage documentation. Testing and QA practice issue, not a code pattern
- **MIN-02. TODO comments**: Codebase contains TODO comments indicating unfinished tasks. Code maintenance issue
- **ORD-01. Missing formulas for WithdrawImbalance and PartialSwap**: Documentation lacks formulas for certain order types, marked with TODO placeholders. Documentation issue
- **ORD-03. Typos**: Recurring typographical errors in comments and messages throughout codebase. Code style issue already caught by spell-checkers
- **MAT-01. Potential optimization in math.calculate_withdraw_imbalance()**: Function could skip calculations when ratios are close enough that integer rounding would yield zero adjustment anyway. Requires understanding of acceptable tolerance thresholds

## Forwards

**Auditor**: TxPipe

**Auditee**: Strike Finance

**Description**: The Strike Forwards protocol is a Cardano on-chain system that enables two users to create a decentralized forward contract to exchange assets at a future date for a pre-agreed price. One user creates a forward by locking collateral and parameters, another accepts it by matching collateral, and later both parties deposit their respective assets to settle. If deadlines pass without proper settlement, liquidation mechanisms redistribute collateral and burn control tokens. The protocol relies on multiple validators (Forwards, Collateral, Agreement, Liquidate) and strict token minting/burning rules to enforce correct execution.

### Findings

**Relevant**

- **STF-001. UTxO address not validated in Create Forward operation**: Minting policy validates datum and value of output UTxO where token is minted but doesn't verify destination address, allowing tokens to be redirected to arbitrary addresses instead of the Forwards validator.<br><ins>Detectable pattern</ins>: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **STF-005. Double counting of tokens in values**: Using subset for value validation instead of exact equality allows trash tokens to bloat UTxOs and enables double counting when the same token serves multiple roles, allowing attackers to satisfy multiple checks with overlapping tokens.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **STF-201. Prevent inclusion of reference scripts**: Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **STF-301. Do Datum comparisons in Data**: Datum comparisons upcast output datum from Data to type rather than downcasting expected datum to Data, which is more expensive.<br><ins>Detectable pattern</ins>: datum equality checks using upcast instead of downcast

**May be relevant**

- **STF-003. Double satisfaction in operations that require token burning**: Validator validates exact number of tokens burned in transaction without ensuring only one validator input is spent, allowing attacker to batch multiple UTxOs and burn fewer tokens than required.<br><ins>Detectable pattern</ins>: aggregate mint/burn validation without input uniqueness check [DOUBLE-SATISFACTION]
- **STF-004. Missing validations in Accept Forward operation**: Validator doesn't verify token minting occurs, doesn't validate collateral UTxO contains required token quantity, and allows accepting forwards with exercise dates in the past.<br><ins>Detectable pattern</ins>: Minting policy that doesn't do any validation at all regarding the tokens being minted.
- **STF-006. One Side Deposit can be performed multiple times**: Validator validates output datum boolean fields without checking input datum state, allowing operation to be repeated and reset deposit flags, enabling asset theft.<br><ins>Detectable pattern</ins>: output datum validation without corresponding input datum validation [PARTIAL-UNVALIDATED-DATUM]
- **STF-007 Missing datum fields validation in Create Forward**: Critical datum fields (asset equality, negative amounts, timestamps) are not validated at forward creation. <br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **STF-002. Potential loss of collateral if neither party deposits the asset**: After the exercise date, if neither party has deposited assets, the Collateral UTxO becomes unspendable because no operation covers this state, permanently locking collaterals. Requires understanding protocol's business logic
- **STF-101. Users could deposit assets after the exercise date has passed**: Validator uses lower bound of validity range to check deadline not passed, allowing users to set past lower bound while transaction executes after deadline.<br><ins>Detectable pattern</ins>: lower vs upper bound usage for deadline checks, but determining correct bound requires understanding check semantics
- **STF-202. One Side Deposit can be bypassed**: Both Sides Deposit operation doesn't verify that one party has already deposited, allowing users to run it when neither party has deposited and gain control of both collaterals. Requires understanding protocol's business logic
- **STF-203. Party identity can be forged**: Redeemer specifies which party performs deposit operation without validator verifying that party's signature, allowing anyone to submit transactions claiming to be either party. Requires understanding protocol's authorization model
- **STF-302. Clean up output lookup in Both Sides Deposit**: Multiple filter operations used to identify specific outputs instead of pattern matching on filtered list. Code organization suggestion
- **STF-303. Standardize the output lookups**: Output lookups use inconsistent filtering approaches (script hash, full address, or token presence) across different operations. Code organization suggestion
- **STF-304. Cleanup output lookup in Accept Forwards**: Complex find implementation used instead of simpler pattern matching to locate single output. Code organization suggestion
- **STF-305. Various recommendations for the Types module**: Multiple type design improvements including using ADTs instead of Int for roles, using stdlib types, refactoring datum structure, removing unused fields, and renaming for clarity. Code quality and design suggestions

## Sundae Swap V3

**Auditor**: TxPipe

**Auditee**: Sundae Labs

**Description**: SundaeSwap is a decentralized exchange built for the Cardano blockchain. It allows participants of the blockchain to provide liquidity and create a market for others to exchange their native tokens. In return, swappers pay a small fee and liquidity providers earn a return on their deposit.

### Findings

**Relevant**

- **SSW-001. Create pool doesn't validate the pool output address**: Minting policy mints pool NFT without verifying destination address, allowing attacker to mint NFT to their wallet and impersonate pools to steal order funds.<br><ins>Detectable pattern</ins>: minting policy missing destination address validation [MISSING-ADDRESS-VALIDATION]
- **SSW-002. Pool output address is not correctly checked in scoop operation**: PoolScoop redeemer doesn't validate payment credential of continuing pool output, allowing scooper to redirect pool funds to arbitrary address.<br><ins>Detectable pattern</ins>: continuing output without payment credential validation [MISSING-ADDRESS-VALIDATION]
- **SSW-101. Settings datum size is limited forever by the initially locked ADA**: Settings validator enforces exact value equality preventing adding ADA when datum grows and requires higher minUTxO.<br><ins>Detectable pattern</ins>: exact ADA/lovelace equality on continuing output values without accounting for variable minUTxO requirements [STRICT-VALUE-EQUALITY]
- **SSW-202. Metadata output datum not checked in pool create**: Pool creation doesn't validate metadata output has datum, potentially creating unspendable UTxO if metadata address is a script.<br><ins>Detectable pattern</ins>: outputs to addresses without datum validation [UNVALIDATED-DATUM]
- **SSW-308. No checks on settings UTxO when it is created**: Settings NFT minting policy validates minting but not destination address or initial UTxO state.<br><ins>Detectable pattern</ins>: minting without validating initial conditions, but determining correct initial state requires protocol understanding [MISSING-ADDRESS-VALIDATION]
- **SSW-313. UpdatePoolFees doesn't require the settings UTxO as reference input**: Validator requires settings UTxO as reference input for UpdatePoolFees but doesn't use it, creating unnecessary off-chain requirement.<br><ins>Detectable pattern</ins>: reference input required but never accessed in validation logic

**May be relevant**

- **SSW-203 Create pool doesn't validate `fees_per_10_thousand` in pool output datum**: `fees_per_10_thousand` is not checked. <br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation. [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **SSW-102. Order Scoop redeemer enforces one and only one withdrawal**: Order validator enforces exactly one withdrawal but Strategy Orders with Script authorization require additional withdrawal, causing transaction failures. Requires understanding protocol multi-validator coordination and business logic
- **SSW-201. Create pool doesn't validate if ADA is not in the pair**: Pool creation checks for at most 3 assets but fails when ADA is not in trading pair because output has 4 assets (Pool NFT + A + B + minUTxO ADA). Requires understanding protocol business logic
- **SSW-203. Create pool doesn't validate fees_per_10_thousand in pool output datum**: Pool creation doesn't validate fee values are in valid range [0, 10000], allowing invalid percentage values. Requires understanding protocol business logic
- **SSW-204. No way to modify the list of authorized staking keys in the protocol settings**: Settings validator prevents modifying authorized_staking_keys field in all update paths, making key rotation impossible. Requires understanding protocol governance and business logic
- **SSW-205. Pool fees update lacks validation of fees percentages**: UpdatePoolFees allows setting fees outside valid range [0, 10000] that CreatePool enforces, allowing fee manager to bypass initial validation. Requires understanding protocol multi-operation coordination and business logic
- **SSW-206. Pool NFT cannot be burned**: Pool minting policy has no redeemer allowing pool NFT burning, making pool destruction impossible when liquidity is depleted. Requires understanding protocol lifecycle and multi-validator coordination
- **SSW-301. Redundant parameters in process_order**: outputs = output + rest_outputs: Function receives both a list and its head/tail components as separate parameters, creating redundancy. Code organization and style issue
- **SSW-302. Redundant check for pool output stake credential in pool scoop validator**: Staking credential checked separately after full address already validated, creating redundant validation. Requires understanding type structure and validation semantics
- **SSW-303. Optimizable power of two (do_2_exp)**: Function uses linear recursion instead of more efficient algorithms available in standard library. Requires understanding function semantics and algorithmic analysis
- **SSW-304. Redundant datum parameter in process_order**: Function receives datum parameter but only uses fields already available as separate parameters, creating redundancy. Requires data flow analysis to determine parameter redundancy
- **SSW-305. Total fee computed recursively can be calculated in single expression**: Recursive fee accumulation through iteration can be replaced with direct formula using order type counts. Requires understanding mathematical equivalence between iterative and closed-form solutions
- **SSW-306. Optimizable check for initial LP minting in create pool**: Validator computes sqrt for verification when expected value is already known, which can be verified more efficiently by squaring. Requires understanding computation semantics and mathematical equivalence
- **SSW-307. Optimizable check for LP minting in scoop**: Validator searches inputs for pool NFT which can be replaced by checking first output and verifying NFT not minted. Requires understanding token conservation logic and equivalent validation approaches
- **SSW-309. Optimizable manipulation of values in do_donation**: Uses value.merge and value.negate instead of more efficient value.add with negative amounts. Aiken-specific library optimization, not applicable to Plinth/Haskell
- **SSW-310. Formula simplifications in do_deposit**: Deposit amount calculations use intermediate "change" variables that can be eliminated through algebraic simplification. Requires mathematical reasoning and understanding formula equivalence
- **SSW-311. Asymmetry of deposit operation**: Deposit operation is asymmetric in corner cases due to integer rounding, deviating from theoretical AMM symmetry. Requires understanding protocol’s AMM specific business logic
- **SSW-312. Optimizable manipulation of output value in has_expected_pool_value**: Multiple function calls traverse same value structure separately instead of single combined traversal. Requires understanding function behavior and data structure traversal patterns
- **SSW-314. PoolState not used anymore**: Type definition no longer used after refactoring remains in the codebase. Dead code issue already caught by standard tooling

## Payment Service

**Auditor**: TxPipe

**Auditee**: Masumi

**Description**: A single-UTxO payment escrow protocol where funds are locked, services are delivered off-chain, and buyers, sellers, or a multisig admin can resolve withdrawals, refunds, or disputes based on timing and signed actions.

### Findings

**Relevant**

- **MAS-101 - Possible to bloat Payment UTxO with trash assets**: Validator checks output value >= input value without enforcing equality, allowing arbitrary tokens to be added to Payment UTxO.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **MAS-201 - Prevent inclusion of reference scripts**: Validator doesn't validate reference script fields in outputs, allowing arbitrary reference scripts to be attached and increasing future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]

**May be relevant**

- **MAS-308 - Validate Payment UTxO initial state**: Payment UTxO can be created with invalid datum field values without validation of initial state.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **MAS-301 - Remove refund_denied field**: Datum field redundant with another field (always derives same value). Requires understanding field semantics and their relationship to determine redundancy
- **MAS-302 - Rename CancelDenyRefund**: Redeemer name misleading as it performs additional actions beyond what name suggests. Code quality issue
- **MAS-303 - Add state field to Payment UTxO**: Payment state must be derived from multiple datum fields rather than having explicit state fields. Code design suggestion about state representation
- **MAS-304 - Redundant validation in WithdrawRefund**: Validation contains redundant checks where the second condition implies the first condition. Code simplification issue requiring understanding of logical equivalence between conditions
- **MAS-305 - SetRefundRequest has double purpose**: Single redeemer performs two distinct operations (requesting refund and adding funds). Requires understanding redeemer semantics to determine if combining actions is doable
- **MAS-306 - Redundant check for state in SubmitResult**: Check validates all possible state values making it always true. Requires understanding protocol's state transitions
- **MAS-307 - Payment UTxO refund_requested field is not needed**: Datum field redundant with state field after refactoring. Requires understanding field semantics to determine redundancy
- **MAS-309 - Unnecessary and suboptimal function output_value_is_preserved**: Custom function reimplements standard library functionality less efficiently. Requires understanding function semantics and library equivalence to determine redundancy

## Treasury Contracts

**Auditor**: MLabs

**Auditee**: SundaeLabs

**Description**: The system manages funds withdrawn from the Cardano treasury, ensuring they can’t be delegated, used for governance, or lost. It uses two scripts: one for treasury-held funds and one for vendor-specific, milestone-released funds. A permissions system controls who can perform operational and administrative actions, such as moving funds, managing vesting milestones, sweeping expired UTxOs, or reorganizing treasury assets.

### Findings

**Relevant**

- **3.10 Vendor datum can desynchronize with value during modify**: Vendor’s output datum has to be properly constrained. Right now, there is no constraint whatsoever.<br><ins>Detectable pattern</ins>: output without datum validation [UNVALIDATED-DATUM]
- **3.11. Datum is not checked during fund action**: Fund operation doesn't validate vendor output datum fields.<br><ins>Detectable pattern</ins>: output without datum validation [UNVALIDATED-DATUM]
- **3.12. Modify endpoint does not enforce invariants on the vendor datum**: Modify operation doesn't validate vendor output datum fields.<br><ins>Detectable pattern</ins>: continuing output without datum validation [UNVALIDATED-DATUM]
- **3.13. Committee can pause matured payouts during adjudicate**: Adjudicate checks maturation using validity range without restricting range length, allowing committee to set arbitrarily early lower bound and bypass maturation checks.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **3.15. Staking credential can be attached to unswept value in vendor contract**: Vendor sweep doesn't validate staking credential on vendor output, allowing arbitrary credentials to be attached.<br><ins>Detectable pattern</ins>: output validation without staking credential checks [MISSING-STAKE-VALIDATION]
- **3.18. Fund should check redeemer is only denominated in the accepted currencies**: Fund and modify operations don't validate which tokens are used in payout values, allowing arbitrary tokens instead of only accepted currencies.<br><ins>Detectable pattern</ins>: value validation without token restriction checks [TRASH-TOKENS]
- **4.7. Limit oneshot minting policy to mint a single registry token**: Oneshot minting policy doesn't enforce exactly one token is minted, allowing multiple tokens with same policy but different names.<br><ins>Detectable pattern</ins>: minting validation without exact quantity check [INCOMPLETE-TOKEN-VALIDATION]

**May be relevant**

- **3.4. Double satisfaction between 2 TRSC instances when sweeping both**: Multiple Treasury UTxOs validate against same aggregate donation outputs, allowing single donation to satisfy multiple validators.<br><ins>Detectable pattern</ins>: aggregate value validation without uniqueness check [DOUBLE-SATISFACTION]
- **3.6. Steal withdraw rewards from treasury contract with double satisfaction**: Treasury withdrawal validates outputs without preventing treasury inputs, allowing combination with other operations to satisfy checks with single payment.<br><ins>Detectable pattern</ins>: aggregate output validation without input restrictions [DOUBLE-SATISFACTION]
- **3.17. DoSing treasury sweep UTxOs**: Treasury sweep allows arbitrarily small donations enabling DoS through repeated minimal sweeps.<br><ins>Detectable pattern</ins>: operations allowing insignificant state changes [UNCHANGED-STATE]

**Not relevant**

- **3.5. Steal funds when sweeping treasury**: Sweep validator assumes treasury inputs exceed outputs but combining with operations that add funds makes subtraction negative, allowing arbitrarily small donations. Requires understanding business logic of multiple operations and their interactions
- **3.7. Uncapped VendorDatum size can halt vendor script**: VendorDatum payouts list can grow until transactions exceed on-chain limits, locking funds. Requires understanding protocol usage patterns
- **3.8. Modify active matured payouts in datum is possible**: Modify operation doesn't validate that already-matured payouts remain unchanged, allowing maturation dates to be moved to future and making claimable payouts unclaimable. Requires understanding the business logic and state transitions of the specific protocol
- **3.9. Modification forces withdrawal of matured non-ada payouts**: Validator uses exact equality for non-ADA assets but inequality for ADA, forcing matured non-ADA payouts to be withdrawn during modification while ADA payouts can remain. Requires understanding intended payout withdrawal design
- **3.14. Committee can steal malformed vendor inputs while re-organizing treasury inputs**: Reorganize validates treasury inputs ≤ outputs but allows treasury inputs, enabling combination with operations that add treasury outputs to satisfy checks while stealing funds. Requires understanding how specific operations interact
- **3.16. minAda is sweepable from vendor utxo with unmatured payments**: Sweep operation incorrectly accounts for minimum ADA requirements when creating continuing outputs, forcing sweeper to provide additional ADA. Requires understanding protocol's minUTxO handling
- **4.1. payout_upperbound from TreasuryConfiguration is unused**: Configuration field not used in validation logic. Dead code issue already caught by standard tooling
- **4.2. Fund action isn't enforced to be witnessed by the vendor**: Fund operation doesn't explicitly validate vendor signature, relying on permission configuration that could be incorrectly initialized. Requires understanding protocol's authorization model
- **4.3. Modify doesn't allow for an increase of payouts**: Modify operation disallows treasury inputs, preventing increase in total payout values. Protocol design decision about allowed operations
- **4.4. Unnecessary redeemer parameter for adjudicate, fund and disburse**: Redeemer parameters duplicate information already available in transaction outputs and datums. Requires data flow analysis to determine parameter redundancy and semantic understanding of what redeemer fields represent
- **4.5. Disallow spending treasury UTxOs during vendor sweep**: Sweep operation allows treasury inputs without clear validation requirements, increasing complexity. Protocol design decision about operation scope
- **4.6. Redundant checks in treasury sweep**: Sweep validation uses unnecessarily complex checks that could be simplified. Simplification suggestion that requires understand the protocol’s business logic
- **4.9. Cap the number of script inputs in the fund action**: Fund action prevents unwanted script inputs using indirect validation rather than explicit input counting. Requires understanding that different validation approaches are functionally equivalent
- **4.10. Constrain redeemers used during fund and reorganize action**: Validator doesn't enforce that all treasury inputs use the same redeemer type. Requires understanding which redeemer constraints are necessary
- **4.11. Modify does not enforce enough constraints**: Modify operation allows multiple different actions without enforcing specific constraints for each. Determining appropriate validation logic for combined operations requires understanding protocol's business logic

## Pro-rata Sale

**Auditor**: SundaeLabs

**Auditee**: Butane

**Description**: A pro-rata token sale protocol where users lock ADA to buy Butane tokens at a fixed price, with excess ADA rebated proportionally if oversubscribed.

### Findings

**Relevant**

- **BTN-301 - Issues with minUTXO protections**: Validation checks use exact equality on ADA amounts instead of allowing surplus, preventing users from including extra ADA for minUTxO.<br><ins>Detectable pattern</ins>: exact equality on ADA amounts when inequality would be more appropriate [STRICT-VALUE-EQUALITY]

**May be relevant**

- **BTN-200 - Funds could get deadlocked if the recipient is a script**: Validator enforces NoDatum on outputs and expects VerificationKeyCredential, permanently locking funds if directed to script addresses.<br><ins>Detectable pattern</ins>: outputs to script addresses without datum validation
- **BTN-201 - Depositors must sacrifice their staking rewards**: Validator requires stake credential to equal payment credential, causing loss of staking rewards.<br><ins>Detectable pattern</ins>: staking credential validation using payment credential comparison

**Not relevant**

- **BTN-000 - No commitment to price or allocation by Admin**: Admin can arbitrarily change sale price, token allocation, and token policy without commitment, allowing significant deviation from expected terms. Protocol governance and trust model issue
- **BTN-001 - Admin can change terms during partial claim or deadlock unclaimed funds**: Admin can spend Admin UTXO after sale closes but before all claims complete, changing terms mid-claim or removing datum to permanently lock user funds. Protocol governance and trust model issue
- **BTN-100 - Not commitment to sale schedule**: Admin controls sale timing without constraints, able to extend under-subscribed sales indefinitely or close prematurely, while participants are locked in. Protocol governance and trust model issue
- **BTN-101 - Incomplete solutions to previous findings**: Earlier fixes introduced a flawed state machine that allows over-minting or permanent deadlock if burned. Requires understanding of the state machine compliant to the protocol
- **BTN-202 - Will not use state machine**: Admin UTxO intentionally moved back to a wallet instead of locking it in a script, placing additional trust in admin role. Protocol governance and trust model issue
- **BTN-300 - Attach staking addresses to deposits**: Recommendation to attach stake addresses to deposits so users earn staking rewards during sale. Requires understanding protocol's business logic about who should receive staking rewards and whether stake credential enforcement is appropriate

## Treasury Contracts

**Auditor**: TxPipe

**Auditee**: SundaeLabs

**Description**: The Cardano Treasury contracts manage Cardano treasury funds safely by preventing delegation and governance use. They structure funds into Treasury, Vendor, and Registry UTxOs, allowing controlled allocation to projects. Actions are governed by a multisignature permission system encoded in the validators, ensuring only authorized operations can move or modify treasury allocations.

### Findings

**Relevant**

- **TRS-102. Matured payouts can be passed as not matured by modifying the validity range**: Maturity check uses lower bound of validity range without restricting range length, allowing users to set past lower bound to incorrectly classify mature payouts as immature.<br><ins>Detectable pattern</ins>: temporal checks without validity range length constraints [VALIDITY-RANGE-BOUND]
- **TRS-103. Vendor Adjudicate: vulnerability allows bypassing payout status checks**: Using list.zip with redeemer-provided statuses list without length validation allows attackers to provide fewer statuses than payouts, causing validation to skip unchecked payouts.<br><ins>Detectable pattern</ins>: list.zip with redeemer data without length equality check
- **TRS-204. Prevent inclusion of reference scripts**: Validator doesn't validate reference script field of Treasury outputs in unpermissioned operations, allowing attackers to attach large reference scripts and increase future transaction fees.<br><ins>Detectable pattern</ins>: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **TRS-202 Vendor Sweep: vendor output stake credential not checked**: SweepVendor redeemer does not check that the vendor output stake credential is None if there are still not withdrawn funds.<br><ins>Detectable pattern</ins>: output validation without stake credential checks [MISSING-STAKE-VALIDATION]

**May be relevant**

- **TRS-001. Treasury Sweep and Vendor Malformed Double Satisfaction**: Multiple Treasury UTxOs validate against the same aggregate donation in transaction outputs, allowing single donation to satisfy multiple validators and steal funds.<br><ins>Detectable pattern</ins>: aggregate value validation without uniqueness check [DOUBLE-SATISFACTION]
- **TRS-104. Treasury Fund DS attack vector**: Multiple Treasury UTxOs from different scripts referencing the same Vendor credential can be batched, with both validators satisfied by same Vendor outputs, allowing fund theft.<br><ins>Detectable pattern</ins>: aggregate output validation without uniqueness check [DOUBLE-SATISFACTION]
- **TRS-203. Treasury Sweep DDOS**: Treasury Sweep allows donating minimal amounts (even 1 lovelace) enabling DoS attacks through repeated spam transactions that technically pass validation but don't meaningfully reduce treasury.<br><ins>Detectable pattern</ins>: operations allowing states to be unchanged [UNCHANGED-STATE]

**Not relevant**

- **TRS-002. Treasury script withdrawal Double Satisfaction**: Treasury staking script withdrawal can produce outputs that satisfy Vendor validator checks for Treasury payments, allowing attackers to steal funds by using withdrawals instead of genuine payments. Requires understanding protocol business logic
- **TRS-003. False publish purpose renders scripts unusable**: Treasury staking validator rejects all publish purposes, preventing required credential registration and dRep delegation as mandated by Cardano constitution. Requires understanding governance requirements and protocol business logic
- **TRS-101. Treasury Fund: Vendor UTxOs can be created with insufficient funds**: Validator checks aggregate payout values match total but doesn't validate each individual Vendor UTxO has sufficient value to cover its datum's payouts, creating unspendable UTxOs. Requires understanding protocol business logic and datum field semantics
- **TRS-105. Funds from malformed vendor UTxOs can be stolen**: Vendor Malformed validator checks funds sent to Treasury without forbidding Treasury inputs, allowing combination with Treasury operations where same outputs satisfy both validators. Requires understanding multi-validator coordination and business logic
- **TRS-201. Vendor Sweep: incompatible with Treasury Reorganize**: SweepVendor allows spending Treasury UTxOs but is only valid after expiration while Reorganize is only valid before expiration, creating an impossible operation combination. Requires understanding protocol temporal constraints and business logic

## Splash DAO

**Auditor**: TxPipe

**Auditee**: Splash

**Description**: The Splash DAO protocol enables token-locked governance via Voting Escrows (VEs). Users lock assets to mint governance power and participate in two voting systems: Weighting Polls (inflation distribution) and Governance Proposals (protocol parameter changes). The system is composed of multiple factories, minting policies, and validators coordinating VE lifecycle, voting, reward distribution, and protocol updates.

### Findings

**Relevant**

- **SPH-001 - Permission Manager UTxO authenticity is not validated**: Validator doesn't check that referenced PM UTxO contains required authentication token, allowing any UTxO with correct datum format to act as permission manager.<br><ins>Detectable pattern</ins>: reference input selection by datum without validating presence of identifying token [UNVALIDATED-INPUT-INDEX]
- **SPH-101 - Charge operation allows trash tokens to be added to Smart Farms**: Charge validates value only increases without restricting which tokens can be added, allowing arbitrary tokens to bloat UTxO and prevent future operations.<br><ins>Detectable pattern</ins>: subset value validation without token restriction checks [TRASH-TOKENS] [INCOMPLETE-TOKEN-VALIDATION]
- **SPH-102 - Governance proposals can contain duplicated options**: Proposal creation validates minimum option count without checking option hash uniqueness, allowing duplicate hashes.<br><ins>Detectable pattern</ins>: lists without uniqueness validation [LIST-UNIQUENESS]
- **SPH-203 - Weighting Poll Factory allows for duplicate IDs in the active farms list**: Active farms list doesn't validate uniqueness, allowing duplicate entries.<br><ins>Detectable pattern</ins>: lists without uniqueness validation [LIST-UNIQUENESS]

**May be relevant**

- **SPH-201 - Smart Farms can't be destroyed**: Smart Farm validator doesn't allow destruction operation, permanently locking minADA.<br><ins>Detectable pattern</ins>: validator without terminal operation allowing UTxO destruction

**Not relevant**

- **SPH-002 - Governance Power and Voting Escrow Composition tokens can be stolen out of the Voting Escrow validator**: Factory validates VE output without checking identifier token is minted, allowing reuse of existing VE identifier from Extend operation to satisfy both validators and steal minted tokens. Requires understanding multi-validator coordination
- **SPH-003 - Voting Escrow Redeem allows tokens to be leaked**: VE Redeem delegates validation to factory by checking factory input presence without verifying which redeemer is used, allowing factory to use different redeemer and leak tokens. Requires understanding which redeemers should be used for specific operations
- **SPH-004 - Double satisfaction during Voting Escrow Redeem**: Multiple VE UTxOs can redeem simultaneously while factory only validates token handling for one, allowing others to leak tokens. Requires understanding multi-validator coordination
- **SPH-005 - Weighting Poll can't be created with SPLASH tokens**: Over-restrictive token validation prevents SPLASH tokens in Weighting Poll (WP) output when they should be allowed. Requires understanding protocol's business logic
- **SPH-006 - Voting Escrow extend operation forces users to steal composition tokens**: VE extend logic requires composition token equality, making it impossible to lock all newly minted tokens into the Voting Escrow. In the worst case scenario, with the extend operation, tokens locked in the VE could be duplicated and all newly minted composition tokens could be stolen. Requires understanding multi-validator coordination
- **SPH-007 - Inflation UTxO can be faked, locking remaining SPLASH forever**: WP creation doesn't validate Inflation UTxO authenticity, allowing fake UTxO to desync factory datum and permanently lock real Inflation UTxO. Requires understanding multi-validator coordination
- **SPH-103 - Weighting Poll Destroy operation can't be executed in certain scenarios**: Destroy operation checks exact SPLASH amount remaining instead of distribution completion status, failing when rounding leaves micro-tokens. Requires understanding rounding behavior and appropriate validation approach
- **SPH-104 - Weighting Poll Factory can be updated together with other factories' proposals**: WP Factory uses redeemer-provided index while other factories use hardcoded index, allowing simultaneous modification with other proposals. Requires understanding multi-validator coordination
- **SPH-105 - Issues and missing validations for the VE Identifier token**: Identifier NFTs can be minted freely and duplicated, permanently locking VEs, due to Identity NFTs not being bounded to a validator. Requires understanding token lifecycle and protocol invariants
- **SPH-106 - Governance power tokens and weighting power tokens not needed**: Voting tokens add complexity and attack surface without providing essential guarantees. Design simplification suggestion that requires understanding the protocol's business logic
- **SPH-107 - Possible DoS attack to Smart Farm and Voting Escrow factories**: Factories can be consumed by anyone enabling DoS through UTxO contention. Requires understanding protocol’s access control requirements
- **SPH-202 - Users can vote multiple times per epoch/proposal**: Users can redeem VE, recreate it, and vote again within the same period if lock expires mid-voting. Requires understanding voting power and the specific business logic of the protocol
- **SPH-204 Users can vote on Governance Proposals up to 12 hours after deadline**: Deadline check allows users to vote up to 12 hours after the deadline. Requires understanding the purpose of the validity interval
- **SPH-301 Proposal creation: governance_power_policy not validated**: Proposal datum field has no check on the correctness of the expected value, so it may reference an arbitrary governance power policy. It requires understanding purpose of datum field

## AADA v1.1

**Auditor**: VacuumLabs

**Auditee**: Aada Finance

**Description**: A peer-to-peer decentralized lending protocol on Cardano where borrowers create collateralized loan requests and lenders fulfill them. Borrower and Lender positions are represented by transferable NFTs, enabling repayment, liquidation, or secondary-market transfers. The protocol supports early repayment with time-proportional interest, oracle-based liquidations, and enforces a minimum interest payment.

### Findings

**Relevant**

- **AADAIK-001. Attacker can spoof AADA NFTs and impersonate their rightful holders**: Function validate_token_mint validates specific token tuple is minted but doesn't check exclusivity, allowing attacker to mint additional tokens with different names in same transaction.<br><ins>Detectable pattern</ins>: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AADAIK-203. ADA locked by the protocol can not be staked**: Contract addresses constructed with None as staking credential and validator enforces no staking credential on inputs, preventing staking of locked ADA.<br><ins>Detectable pattern</ins>: missing staking credential in script addresses [UNVALIDATED-STAKING]

**May be relevant**

- **AADAIK-102. Double satisfaction among different scripts**: Validators prevent double satisfaction only among same protocol's script inputs, allowing attacker to batch with other protocol's scripts expecting payment to same party.<br><ins>Detectable pattern</ins>: double satisfaction prevention checking only same-script inputs without restricting other script inputs [DOUBLE-SATISFACTION]
- **AADAIK-202. Interest calculation is imprecise**: Interest calculation uses division before multiplication (elapsed / total) _ amount, losing precision compared to (elapsed _ amount) / total.<br><ins>Detectable pattern</ins>: division before multiplication losing precision [PRECISION-LOSS]

**Not relevant**

- **AADAIK-002 Lender can liquidate and take the whole collateral as soon as he lends**: Liquidation logic compares if the loan deadline is after the lower bound of the transaction validity range, which can be set in the past, allowing lenders to liquidate collateral immediately after lending. Requires understanding the usage of validity range
- **AADAIK-003 Borrower is not able to repay the loan and loses the collateral**: Functions use asset_name instead of policy_id parameter in from_asset function call, causing validation failures. Logic bug requiring understanding of function parameter semantics
- **AADAIK-101 Borrower can pay only min interest for any loan**: Interest calculation uses a user controlled lower validity bound, allowing borrowers to fake early repayment times and pay only the minimum interest. Requires understanding the business logic and use of validity range
- **AADAIK-103 Lender can collect interest immediately**: The lender can set the loan start time in the past, forcing borrowers to pay full interest even if little or no time has elapsed. Requires understanding the meaning of datum fields
- **AADAIK-201 Expiration setting of borrow request is not enforced**: Expiration checks rely on the lower validity bound, which can be arbitrarily old, allowing expired loan requests to be accepted. Requires understanding usage of validity range
- **AADAIK-301 The collateral amount is set by the lender for the borrower**: The lender defines the collateral amount in the datum, which may not reflect the actual deposited collateral, potentially triggering premature liquidation. Requires understanding the protocol's business logic
- **AADAIK-302 Other code suggestions**: Naming inconsistencies, unused code, misleading variable names, and scattered constants. Code quality issues already caught by standard tooling

## Iagon Delegation v3

**Auditor**: VacuumLabs

**Auditee**: Iagon

**Description**: Iagon Delegation v3 is a centralized Cardano staking and delegation system for the IAG token, where most security critical logic relies on a mutable reference UTxO and a powerful operator hot key. Nodes, delegations, and secondary market sales of delegated stake are enforced on-chain, but operator and reference keepers have sweeping control, making governance, correctness, and key compromise the primary risk surface.

### Findings

**Relevant**

- **ID3-101 Reference keepers might block all scripts and value**: Reference keepers can move the genesis token to an arbitrary address, breaking all scripts and locking all protocol funds. <br><ins>Detectable pattern</ins>: continuing output without address validation [MISSING-ADDRESS-VALIDATION]
- **ID3-303. Operator can mint any node, order and delegation tokens**: Minting policy validates a mint occurs but doesn't restrict to specific token name in redeemer, allowing operator to mint arbitrary additional tokens with different names. <br><ins>Detectable pattern</ins>: minting validation without token name exclusivity [INCOMPLETE-TOKEN-VALIDATION]
- **ID3-306. Node can be moved to any address during an update**: Node update doesn't validate new node UTxO address remains unchanged, allowing redirection to arbitrary address. <br><ins>Detectable pattern</ins>: continuing output without address validation [MISSING-ADDRESS-VALIDATION]
- **ID3-308. Delegation can be moved to any address during an update**: Delegation update doesn't validate new delegation UTxO address remains unchanged, allowing redirection to arbitrary address. <br><ins>Detectable pattern</ins>: continuing output without address validation [MISSING-ADDRESS-VALIDATION]
- **ID3-309. Split delegation UTxOs are mostly unchecked**: Split delegation creates new delegation UTxOs with arbitrary addresses and unchecked datums, only validating value contains delegation token and IAG stake. <br><ins>Detectable pattern</ins>: output creation with incomplete validation (address and datum unchecked) [MISSING-ADDRESS-VALIDATION] [UNVALIDATED-DATUM]
- **ID3-311. The payment output could contain dust tokens**: Payment output validation doesn't restrict tokens to Ada only, allowing arbitrary tokens that increase minAda and reduce usable value to the seller. <br><ins>Detectable pattern</ins>: subset value validation without token restriction checks [TRASH-TOKENS]
- **ID3-312. The payment output can be staked to any credential**: Payment output staking credential unchecked, allowing operator to choose arbitrary credential. <br><ins>Detectable pattern</ins>: output validation without staking credential checks [UNVALIDATED-STAKING]

**May be relevant**

- **ID3-103. Reference keepers might block all scripts and value II**: Reference keepers can make keeper list arbitrarily long, causing reference UTxO to exceed transaction size or execution limits in dependent scripts. <br><ins>Detectable pattern</ins>: unbounded list length without size validation
- **ID3-302 Order tokens can be freely minted**: Burn redeemer also allows minting arbitrary order tokens, bypassing mint validation. <br><ins>Detectable pattern</ins>: No exclusivity enforced on burning
- **ID3-408. Some fields are not used nor verified on-chain**: Datum fields not validated on-chain, relying on operator to set correctly. <br><ins>Detectable pattern</ins>: datum fields without validation [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **ID3-102. Node and delegation can not withdraw without the operator**: All withdrawals require the operator’s signature, meaning funds are locked when the operator is unavailable. Protocol governance and trust model issue
- **ID3-104. Reference keepers can steal Ada contained in orders**: Reference keepers can update delegation contract hash in reference datum to malicious contract, allowing validation delegation to dummy validator and theft of order value. Requires understanding protocol's business logic and multi-validator coordination
- **ID3-301. Operator as the sole guarantor of unique id computation**: Token uniqueness is not enforced on-chain and relies entirely on operator not making mistakes. Protocol design decision about token name generation
- **ID3-304. Delayed order signature is not necessary**: Order processing validates delayed signature even though order keeper already signed full transaction with complete datum. Code complexity and redundancy requiring understanding of authorization model
- **ID3-305. Reference keepers might lock themselves out of the protocol**: Reference keepers can update their own list without validating new list can achieve an unusable configuration. Requires understanding protocol governance and authorization model
- **ID3-307. Delegation seller may receive slightly smaller compensation**: Price calculation uses floor division causing seller to lose up to 1 lovelace due to rounding. Requires understanding correct rounding direction for business logic
- **ID3-313. Orders do not protect against delegation sale price change**: Orders remain valid after delegation price changes, allowing operator to keep price difference if price drops. Protocol design decision about price change handling
- **ID3-401. plutus.json does not correspond to the code**: Compiled artifacts outdated compared to source code. Code maintenance and build process issue
- **ID3-402. Genesis token name is unnecessarily complex**: Genesis token name computation captures uniqueness already reflected in policy id, making name computation redundant. Code complexity and design decision about token naming
- **ID3-403. Grammar issues, typos, semantics**: Various grammar errors and typos throughout documentation and comments. Documentation issues
- **ID3-404. Copy paste errors**: Incorrect terminology derived from reused code. Documentation issue
- **ID3-405. Excessive comments**: Almost every line has comments that repeat what code already says instead of explaining non-trivial logic. Documentation and code style issue
- **ID3-406. Naming**: Ambiguous naming obscures semantics of fields and variables. Code style and naming issues
- **ID3-407. Delegation can be delegated to a non-existent or inactive node**: Node validity is not enforced in delegation update and split flows, unlike in creation flow. Requires understanding protocol's business logic and validation consistency requirements
- **ID3-409. Multiple order trades not possible in a single transaction**: Comment claims parallelization possible but minting policy allows exactly one burn per transaction. Documentation inconsistency with implementation

## Cardano Casino v1.0

**Auditor**: VacuumLabs

**Auditee**: [Not provided]

**Description**: On-chain settlement layer of a Cardano casino game. Smart contracts manage bets, payouts, fees, and bank funds, while all game logic, randomness, and outcome determination happen off-chain and are enforced via a trusted backend key. User bets are locked on-chain, resolved by the backend, and winnings are paid from shared Bank UTxOs under admin control.

### Findings

**Relevant**

- **CCA-003. Value checks are not satisfiable**: Validator uses value.policies function and asserts result equals specific list of policies, but list doesn't include Ada policy (#"") which is present in every UTxO, making all checks unsatisfiable and preventing protocol operation.<br><ins>Detectable pattern</ins>: value.policies equality check without accounting for Ada policy
- **CCA-301. Staking credentials are not handled**: Validator doesn't check staking credentials on user deposits and Bank UTxOs, allowing transaction authors to allocate future staking rewards to themselves.<br><ins>Detectable pattern</ins>: output validation without staking credential checks [UNVALIDATED-STAKING]
- **CCA-303. Strict Ada amount comparison**: Validator requires exact Ada amount in output UTxOs without accounting for minAda requirements, making UTxO creation infeasible when computed amount is below minAda threshold.<br><ins>Detectable pattern</ins>: exact equality on ADA amounts in output validation [STRICT-VALUE-EQUALITY]
- **CCA-304. UTxOs can be polluted by dust tokens**: Validator checks only Ada value without restricting other tokens, allowing arbitrary tokens to be added to UTxOs and causing transaction loading issues.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **CCA-306 Owner’s staking credential can be set arbitrarily**: Wins are paid to a pubkey hash without staking credentials, allowing backend manipulation.<br><ins>Detectable pattern</ins>: output validation without staking credential checks [UNVALIDATED-STAKING]

**May be relevant**

- **CCA-201 Empty transactions**: Transactions with no valid updates can reshuffle UTxOs, blocking legitimate backend operations.<br><ins>Detectable pattern</ins>: Allowing transactions without enforcing meaningful actions [UNCHANGED-STATE]
- **CCA-204. Double satisfaction on Bet owner's compensation**: Validator checks only own script inputs when validating compensation payment to Bet owner's address, allowing multiple scripts expecting payment to the same address to be satisfied with single payment.<br><ins>Detectable pattern</ins>: value aggregation filtering by address without script input uniqueness check [DOUBLE-SATISFACTION]

**Not relevant**

- **CCA-001. User deposits can be stolen**: Deposits can be spent in update transactions with zero valid requests and without enforcing backend or user signatures, allowing arbitrary theft. Requires understanding protocol workflow and which operations require signatures
- **CCA-002. Missing incentives for users to sign a transaction**: UpdateRequests require both Casino and user signatures, but users have no incentive to sign transactions causing them losses since they can inspect game results before signing. Protocol economic design
- **CCA-101. UpdateRequests can be invalidated**: Validator filters UpdateRequests by signature validity but doesn't reject transactions containing incorrectly signed requests, allowing attackers to remove valid requests from the chain and steal their Ada. Requires understanding protocol's request validation model
- **CCA-102. The backend key can take over the protocol**: All UTxO types share the same validator, allowing backend to use FulfilBet redeemer to be missused e.g. on Admin UTxO and modify protocol ownership or halt protocol. Requires understanding which redeemers should be valid for which UTxO types
- **CCA-103. Unaccounted Bet funds**: Balance calculation doesn't account for fees locked in Bet UTxO, allowing unaccounted funds to be stolen when the user wins. Requires understanding protocol's accounting model and intended fund flows
- **CCA-202. No key redundancy and rotation possibility**: Hardcoded keys with no redundancy or rotation mechanism make the protocol fragile to loss or compromise (key loss halts protocol, key compromise enables fund theft). Protocol governance and key management design issue
- **CCA-203. Non-transparent off-chain computation**: Critical payout values are computed off-chain, passed as redeemers and blindly trusted on-chain, risking bank drainage. Requires understanding which calculations should be on-chain versus delegated to off-chain
- **CCA-302. Single key can withdraw Bank funds**: Backend key alone can withdraw all Bank funds by creating arbitrary winning UpdateRequests/Bets, while Bank withdrawal requires two keys. Protocol governance and trust model issue
- **CCA-305 UTxOs with a defined staking credential are unprotected**: Script input discovery fails when staking credentials are present, leaving UTxOs unrestricted. Requires understanding usage of addresses
- **CCA-401. Every UTxO validates the whole transaction**: Validator runs identical validation for each script UTxO in transaction instead of validating once, increasing fees and limiting batch size. Requires understanding protocol's transaction structure and which validations can be safely deduplicated
- **CCA-402. Bank inputs and outputs not used in all code branches**: Variables computed before validation but only used in specific redeemer branches, wasting computation in other branches. Code organization issue requiring understanding which branches use which variables
- **CCA-403. Fee can be set in whole percents only**: Fee protocol parameter denominated in whole percents without finer granularity (e.g., 2.5% impossible). Protocol design decision about parameter precision and representation
- **CCA-404. Duplicated code**: Code performs the same computations in multiple places using duplicated lines instead of common functions. Code duplication issue already caught by standard tooling
- **CCA-405. Unnecessary redeemer in UpdateAdminData**: UpdateAdminData redeemer contains data that mirrors what's already in the signed transaction, making it redundant since admin signature already authorizes the change. Requires understanding authorization model and what data is redundant given transaction structure
- **CCA-406. Suggest upgrading Aiken and stdlib**: Project uses outdated Aiken version, newer versions include useful features like compiler version in compiled artifacts. Dependency and maintenance issue specific to AIken
- **CCA-407. Unnecessarily strict checks on reference inputs**: Function get_admin_data unnecessarily restricts reference inputs to inline datums only, limiting transaction flexibility. Requires understanding which reference inputs should be validated versus ignored
- **CCA-408. Fee increase could disable active Bets**: Admin fee could change retroactively invalidating existing bets. Protocol design issue requiring off-chain coordination between fee changes and bet resolution

## Cardano Casino v1.1

**Auditor**: VacuumLabs

**Auditee**: [Not provided]

**Description**: The most important change from the v1.0 is the change of the multiplier type which was changed from an integer to a rational number. That allows for a more granular game setup.

### Findings

**Not relevant**

- **CCA2-401 Artefacts do not contain script version**: The compiled plutus.json artefacts do not reflect the updated script version specified in aiken.toml. Build process and maintenance issue specific to Aiken
- **CCA2-402 Not formatted code**: Codebase was not formatted, resulting in inconsistent styling. Code style issue already caught by standard tooling

## Pondora v2 (v1.0)

**Auditor**: Invariant0

**Auditee**: Pond Labs

**Description**: Pondora v2 is a smart account framework for Cardano that lets users lock funds into a programmable vault (“Pond”) and authorize intents describing how and when funds may be used. Intents are authorized on-chain or off-chain (via Merkle roots) and executed by third parties or batchers. Security relies heavily on correct intent authorization, ownership binding through labels, and cross-module accounting, making it highly sensitive to validation gaps, redeemer trust, and cross-script interactions.

### Findings

**Relevant**

- **POND2-305. Near-full and small spends of Ada UTxOs can be troublesome**: Strict equality checks on Ada amounts prevent UTxO creation when change is below minUTxO requirements.<br><ins>Detectable pattern</ins>: exact equality on ADA amounts when inequality would be more appropriate [STRICT-VALUE-EQUALITY]

**May be relevant**

- **POND2-002. Nonce helper validator self-reference**: Validator requires its own script hash as argument, creating circular dependency since arguments affect the compiled hash.<br><ins>Detectable pattern</ins>: validator arguments containing self-referencing script hash/credential
- **POND2-301. Nonce helper overwrites a valid match if it almost finds another**: Inner loop in nonce helper sets running total to 0 instead of inheriting previous total, resetting counter when subsequent redeemer meets criteria but doesn't use nonce.<br><ins>Detectable pattern</ins>: loop initialization using constant instead of accumulator variable
- **POND2-306 Quantity is unchecked for LabelOutRef label**: Quantity field ignored when the label type is LabelOutRef, allowing manipulation.<br><ins>Detectable pattern</ins>: No validation on some redeemer's fields. [UNCHECKED-REDEEMER]
- **POND2-307. Unlock module vulnerable to cross-script double satisfaction**: Module expects payment output that could satisfy multiple scripts at the same address with the same value/datum.<br><ins>Detectable pattern</ins>: value aggregation filtering by address without datum uniqueness check [DOUBLE-SATISFACTION]

**Not relevant**

- **POND2-001. Nonce helper script may be bypassed**: Nonce helper uses stake_registration certificate which doesn't require script witness, allowing nonce validation to be skipped. Requires understanding certificate types and their witness requirements
- **POND2-003. Account helper does not prevent unauthorized drain of funds**: Helper iterates over redeemers matching schema without verifying inputs are from the correct script address. Requires understanding which addresses should be validated
- **POND2-004. Address-based accounting lets an attacker steal funds**: Helper uses (address, label) as owner unique identifier without checking with datum's owner field. Requires understanding datum field semantics and business logic
- **POND2-005. All funds can be stolen by pretending to be an owner of any UTxO**: Owner identity is trusted from the redeemer and not checked against staking credentials. Requires understanding protocol's ownership model
- **POND2-006. Value can be stolen as anyone can sign intent authorization**: Signature verification doesn't check signing key belongs to UTxO's owner. Requires understanding authorization model
- **POND2-007. Pond native validator skips all validation after an unknown redeemer**: Validator terminates iteration when encountering non-matching redeemer instead of continuing. Logic bug requiring understanding intended iteration behavior
- **POND2-008 Service fee module stops processing after the first match**: Validator processes first matching input/redeemer pair but returns immediately without continuing iteration through remaining redeemers, allowing fee theft without execution. Logic bug requiring understanding intended iteration behavior
- **POND2-009. Service fee module observes intents from non-pond scripts**: Intents from arbitrary scripts are trusted, enabling service fees theft without executing anything. Requires understanding which script addresses are valid
- **POND2-101. Arbitrary label injection when a label is present in the datum**: Validator skips checking datum label equals redeemer label when label is present. Logic bug requiring understanding validation semantics
- **POND2-102. Attacker can repeatedly DoS intent authorizations and keep Ada**: Intent UTxOs can be burned by anyone, destroying authorization proofs. Requires understanding protocol's authorization model
- **POND2-103. Modules do not support indirect calls s.a. the and module's**: Modules search only for direct redemptions and don't recognize indirect calls through and module, preventing module argument passing. Protocol design issue about module composition
- **POND2-201. Signed intents can not be revoked once issued**: Off-chain signatures have no on-chain revocation mechanism, allowing indefinite reuse. Protocol design decision about signature lifecycle
- **POND2-202. Defragmentation of native token balances unlocks min Ada within**: Defragmenting another party's tokens allows claiming leftover min Ada from merged UTxOs. Protocol economic design issue
- **POND2-302. Multiple users can not redeem the same intent in one transaction**: Helper uses (label, intent) as uniqueness key without including owner, causing second redemption to be ignored. Requires understanding which fields should constitute unique key
- **POND2-303. Nonce helper prevents batch unlocks by a single intent**: Nonce helper enforces exactly one redemption can mention it, preventing batch unlocks. Protocol design decision about nonce usage scope
- **POND2-304. The account functions do not prevent excessive fragmentation**: Module checks amount spent but not how many UTxOs change is split into. Protocol design issue about missing fragmentation constraints
- **POND2-401. Staking credential registration validation will be required soon**: Scripts rely on withdrawal path without registration checks, but future era will require script witness for registration. Protocol design decision about future-proofing
- **POND2-402. Documentation imprecise about allowing only 1 intent token mint**: Documentation states single intent token per transaction but validation checks one per output UTxO. Documentation issue
- **POND2-403. Pond unlock ByOwner contains additional fields**: Redeemer contains unused fields kept for schema consistency across validator variants. Code maintenance issue
- **POND2-404. Nonce helper can check that it is registered just once**: Registration timing relies on orchestration of valid_until dates instead of explicit time window validation. Code improvement suggestion requiring understanding of timestamp semantics
- **POND2-405. Typos and incorrect documentation**: Multiple typos and misplaced/imprecise comments throughout the codebase. Documentation and code style issues
- **POND2-406. Naming and dead code**: Misleading function parameter names and unused function. Code style and naming issues already caught by standard tooling
- **POND2-407. Code style suggestions**: Multiple code style improvements including using standard library functions, avoiding variable shadowing, and extracting duplicated logic. Code quality and maintainability issues
- **POND2-408. Suggest documenting negative quantity impact**: Negative quantities discarded silently but not documented. Documentation issue
- **POND2-409. Intents using nonce could be blocked if nonce value is found**: Anyone registering a nonce credential can block intent execution. Protocol design tradeoff about nonce credential validation
- **POND2-410. Pond native validator potentially checks all pond versions**: Withdrawal validator validates all inputs with PondRedeemer regardless of pond version, preventing mixed-version transactions. Protocol design decision about version handling

## Pondora v2 (v1.1)

**Auditor**: Invariant0

**Auditee**: Pond Labs

**Description**: Revision of Pondora v2 that focuses on incremental but high-impact changes to the smart-account system: introducing temporary owners, child intents (intent dependency chains), refined unlock-module Ada control, and updated off-chain authorization signing.

### Findings

**Relevant**

- **POND2i-202. Burning of child intent tokens is not controlled enough**: Validator doesn't validate minted quantity or actual burning of child intent tokens, allowing multiple mints with single burn or spending without burning.<br><ins>Detectable pattern</ins>: burn validation without checking mint quantity or restricting minting operations [INCOMPLETE-TOKEN-VALIDATION]

**Not relevant**

- **POND2i-001. Intents can be authorized on behalf of any other user**: Function all_outputs_paid_to_script defaults to True instead of False when intent token owner doesn't match, allowing attacker to replace any user's authorized root hash. Logic bug requiring understanding of validation logic semantics
- **POND2i-101. Unlock module's start validity timestamp makes it inexecutable**: Validator checks transaction validity interval falls within (valid_until, valid_until) instead of (valid_from, valid_until), making execution practically impossible. Requires understanding intended interval bounds
- **POND2i-102. Child intent tokens can not be minted as intended**: Function all_outputs_paid_to_script only allows child intent token in UTxO authorizing same child_ref_root hash instead of new child.root_hash, preventing new child intent authorizations from ever being created. Requires understanding protocol's business logic
- **POND2i-201. Authorized child intent can be used to unauthorize authorizing hash**: Child intent authorization can be abused to revoke its own parent authorization, since it targets authorizing child_ref_root instead of child.root_hash. Requires understanding protocol's business logic
- **POND2i-301. Temporary owner can authorize intents not bound by his validity**: Temporary owner can create permanent on-chain intent authorizations without validity constraints, allowing execution after authority expires. Protocol design decision about authorization scope
- **POND2i-401. Code style and documentation**: Naming ambiguity and outdated documentation around generalized auth logic. Code style and documentation issues

## FluidTokens Lending v3

**Auditor**: VacuumLabs

**Auditee**: FluidTokens

**Description**: Flexible Cardano lending protocol supporting both peer-to-peer loans and single-lender pools, with static and oracle driven dynamic loans. It introduces advanced features such as partial liquidations, Dutch auctions, perpetual loans, recasting, bond-token–based position ownership, and deep integration with CIP-113 programmable tokens.

### Findings

**Relevant**

- **FTL3-002. Lender pool funds can be stolen**: Validator doesn't verify continuing pool output address, allowing borrower to redirect remaining pool funds to controlled address instead of recreating pool at correct script address.<br><ins>Detectable pattern</ins>: continuing output without address validation [MISSING-ADDRESS-VALIDATION]
- **FTL3-006. Blocking funds and gaining unfair advantage by adding a programmable token**: Malicious programmable tokens can be injected to block liquidations, repayments, or pools.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **FTL3-205. Too small Ada equity makes liquidation impossible**: Validator requires exact equity amount in borrower output instead of minimum amount, preventing UTxO creation when computed equity is below minUTxO requirements.<br><ins>Detectable pattern</ins>: exact equality on ADA amounts when inequality would be more appropriate [STRICT-VALUE-EQUALITY]
- **FTL3-206. It might be impossible to add collateral to a non-specific asset collateral loan**: Validator uses strict Ada equality preventing increases, but adding collateral tokens increases UTxO size requiring higher minAda.<br><ins>Detectable pattern</ins>: exact equality on ADA amounts when inequality would be more appropriate [STRICT-VALUE-EQUALITY]
- **FTL3-302. Oracle's valid_from is unchecked**: Validator checks oracle validity range length and transaction validity against valid_to, but doesn't validate valid_from, allowing malformed ranges to pass validation.<br><ins>Detectable pattern</ins>: temporal validation checking only one bound of validity range without validating the other bound [VALIDITY-RANGE-BOUND]

**May be relevant**

- **FTL3-001. Repayments can not be withdrawn**: Repayment tokens must be burned to withdraw funds, but the minting policy forbids burning, making repayments permanently locked.<br><ins>Detectable pattern</ins>: No burning logic on the validator [NO-BURNING-LOGIC]
- **FTL3-003. Collateral can not be withdrawn**: Loan tokens required to unlock collateral cannot be burned due to minting policy restrictions.<br><ins>Detectable pattern</ins>: No burning logic on the validator [NO-BURNING-LOGIC]
- **FTL3-005. Lender can claim the whole collateral in an auction by malicious address**: Auction creation doesn't validate owner address format in datum, allowing lender to set invalid-length credentials or pointer credentials that break payment validation.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]
- **FTL3-030. Datums can not be parsed**: Code uses builtin.un_list_data on datums with constructor types, causing parsing to always fail and making protocol unfeasible. Aiken-specific issue; requires verification if equivalent issue exists in Haskell/Plinth.
- **FTL3-102. Ada in expired requests is vulnerable to double satisfaction**: Multiple expired requests can reference the same borrower compensation output via redeemer-provided index, allowing single payment to satisfy multiple requests and stealing excess Ada.<br><ins>Detectable pattern</ins>: redeemer-provided output index without uniqueness validation [DOUBLE-SATISFACTION]
- **FTL3-104. Cross-script double satisfaction**: Validators check party receives payment without preventing other script inputs, allowing single payment to satisfy multiple validators expecting payment to the same party.<br><ins>Detectable pattern</ins>: value aggregation filtering by address without script input uniqueness check [DOUBLE-SATISFACTION]
- **FTL3-029. Bond addresses trusted without bond presence verification**: Reference inputs used without checking NFT presence allow value redirection.<br><ins>Detectable pattern</ins>: Usage of reference inputs without identifying NFT verification
- **FTL3-208. User stake credentials to authorize programmable token transfers**: Using staking credentials for ownership complicates transaction construction and signing.<br><ins>Detectable pattern</ins>: Usage of staking credential instead of payment credential for spend authorization

**Not relevant**

- **FTL3-004. Lender can claim the whole collateral in a dutch auction before start**: Cancel mechanism allows the lender to cancel the auction before the start date without collateral validation, enabling theft of collateral which typically exceeds debt value. Requires understanding protocol's business logic
- **FTL3-007. Ada collateral is not protected in requests**: Validator checks collateral preservation but explicitly skips Ada validation, allowing the lender to claim the entire Ada collateral while only providing principal to the borrower. Requires understanding which assets should be validated as collateral
- **FTL3-008. Lender can disable repaying and liquidate**: Function hash_output_ref errors on transaction indices > 255, allowing lender to create loan at index 256+ by adding dummy outputs, making loan unrepayable and forcing liquidation. Requires understanding function semantics
- **FTL3-009. Loan token can not be minted for programmable token loans**: Minting policy searches for inputs using wrong staking credential (loan validator instead of request/pool validator), preventing token minting for programmable token requests/pools. Requires understanding multi-validator coordination
- **FTL3-010. Repayment token can not be minted for programmable token loans**: Minting policy searches for loan inputs using wrong staking credential (repayment validator instead of loan validator), preventing repayment token minting and making repayment impossible. Requires understanding multi-validator coordination
- **FTL3-011. Protocol is unfeasible due to usage of get_outputs_to_smart_credential**: Function searches for outputs using incorrect staking credentials in multiple operations, making protocol unfeasible when programmable tokens are involved. Requires understanding multi-validator coordination
- **FTL3-012. Dutch auction's borrower compensation goes to the lender**: Function parameter receives wrong variable, causing excess auction proceeds to be sent to lender instead of borrower. Logic bug requiring understanding of variable semantics and parameter passing
- **FTL3-013. AMM formulas are incorrect**: Oracle type using constant-product AMM formulas has incorrect mathematical calculations for price estimation including fees and slippage. Requires understanding AMM mathematics and correct formula derivation
- **FTL3-014. Wrong arguments in conversion from Ada to token**: Function receives swapped token A/B arguments (lovelace supplied as token A when it should be token B according to oracle type), causing invalid conversion calculations. Logic bug requiring understanding of function parameter semantics and oracle data structure
- **FTL3-015. Healthy loans can be liquidated**: Function uses inverted comparison logic, allowing liquidation only for healthy loans. Logic bug requiring understanding the business logic
- **FTL3-016. Amortization formula is wrong**: Amortization calculation uses incorrect formula with exponent applied to entire numerator instead of just (1 + i) term, causing massive overpayment. Requires understanding correct mathematical formula
- **FTL3-017. Recasting does not work well with the amortization formula**: Recast logic subtracts recast amount from initial principal without accounting for already-repaid principal portions or updating remaining term, causing incorrect installment calculations. Requires understanding the protocol's business logic
- **FTL3-018. Recasting on due loan installments avoids interest and penalties**: Recast allowed without validating due installments are repaid first, enabling borrowers to avoid interest by recasting instead of making scheduled payments. Requires understanding protocol's business logic
- **FTL3-019. Inconsistent perpetual loan interest computation**: Two functions use different interest rate calculations, causing mismatched interest values. Logic bug requiring understanding of intended interest calculation semantics
- **FTL3-020. Remaining debt on perpetual loans does not assume previous payments**: Function calculates total debt from initial lend date without accounting for already-paid interest installments, causing double-charging of interest. Requires understanding protocol's business logic
- **FTL3-021. Dutch auction payments can break due to checking bond addresses**: Payment validation uses bond holder addresses but fails with certain staking credential types, allows adding unspendable programmable tokens, and may not handle programmable credential addresses correctly. Requires understanding multi-validator coordination
- **FTL3-022. Perpetual loan recasting logic is incorrect**: Recasting reduces principal without ensuring accrued interest is paid first, causing interest on original principal to be lost in debt calculation. Requires understanding protocol's business logic
- **FTL3-023. Loan inputs with programmable assets bypass action validator checks**: Action validators search for loan inputs using wrong credential parameter, causing loans with programmable collateral to be missed and spendable without validation. Requires understanding multi-validator coordination
- **FTL3-024. Wrong action credential allows borrowers to unlock programmable collateral**: Action validators search for continuing loan outputs using wrong credential parameter, allowing borrowers to redirect outputs to addresses they control and bypass validation. Requires understanding multi-validator coordination
- **FTL3-025. Wrong receipt condition allows blocking funds with programmable assets**: Condition logic returns True when receipt is not required regardless of receipt token presence, allowing arbitrary tokens to be added. Requires understanding business logic to determine if conditional logic is redundant
- **FTL3-026. Programmable collateral sent to uncontrollable auction credential is lost**: Function forces programmable assets to use spend credential as stake credential, but auction script can't control programmable token transfers, causing permanent fund loss. Requires understanding multi-validator coordination
- **FTL3-027. Zero liquidation penalty incorrectly skips equity return to borrower**: Condition changed from < 0 to <= 0 withholds equity when penalty is zero, even though borrower should receive positive equity. Logic bug requiring understanding business logic
- **FTL3-028. Malicious parties can block transactions by holding bonds without stake credentials**: Validators expect inline stake credentials on bond addresses, allowing DoS attacks by using addresses without inline credentials. Requires understanding protocol's authorization model and acceptable address types
- **FTL3-031. Wrong config index extracts incorrect loan policy id**: Validators use wrong index (2 instead of 6) to extract loan policy id from config, retrieving pool policy id instead and preventing loan input discovery for programmable tokens. Logic bug requiring understanding of data structure indexing
- **FTL3-032. LTV is calculated based on the initial principal**: Function calculates LTV using only initial principal versus collateral value without accounting for accumulated interest or repaid principal, causing incorrect liquidation eligibility. Requires understanding protocol's business logic
- **FTL3-033. Repayment increments wrong field causing eventual collateral loss**: Function increments wrong datum field due to outdated positional index after field reordering, making loans appear unpaid. Logic bug requiring understanding of data structure field positions
- **FTL3-101. DEX oracle computation uses hardcoded fees**: Oracle uses hardcoded 0.3% fee for constant product formula, preventing use with different fee structures or non-constant product pools. Protocol design decision about parameter flexibility
- **FTL3-103. Too big a loan can liquidate the borrower**: Validator doesn't enforce maximum loan amount, allowing the lender to lend an excessive amount that pushes LTV over liquidation threshold immediately. Requires understanding protocol's business logic
- **FTL3-105. Permissioned conditions not enforced for programmable tokens**: Permission validators search for inputs using wrong staking credentials, preventing permission checks for programmable token UTxOs. Requires understanding multi-validator coordination
- **FTL3-106. Time unit change error disables recasts**: Function divides milliseconds by hours after installmentPeriod unit change from milliseconds to hours, causing incorrect due installment calculation and disabling recasts. Logic bug requiring understanding of time unit semantics
- **FTL3-201. Minting multiple repayment tokens is nearly unfeasible**: Policy compares lexicographically sorted minted tokens against unsorted expected tokens derived from hash ordering, making multi-token minting unfeasible when hash order differs from lexicographic order. Requires understanding list ordering semantics
- **FTL3-202. Request can not be cancelled after expiration by a different party**: Validation expects burned request token to be in borrower compensation output, making cancellation unfeasible. Logic bug requiring understanding token lifecycle
- **FTL3-203. It is possible to lend to an expired request**: Validator doesn't check request expiration date, allowing lending to expired requests that should only be cancellable. Requires understanding protocol's business logic
- **FTL3-204. Pool might be blocked until recreated**: Attacker can borrow from pool and recreate it at index > 255, blocking further borrows due to hash_output_ref function limitations. Requires understanding function semantics
- **FTL3-207. No liquidation discount**: Partial liquidation returns exact equity to the borrower without discount, leaving the lender unable to cover exchange fees and price volatility. Protocol economic design issue
- **FTL3-301. Permissioned lending party is chosen by an index out of context**: Index used to select signing party from whitelist represents input position order rather than being an independent parameter, making multi-party transactions difficult to construct. Requires understanding index semantics and redeemer structure
- **FTL3-303. Dutch auction can be bought before it starts**: Validator doesn't check if a transaction occurs after auction start date, allowing purchases before start at proportionally higher price. Requires understanding protocol's business logic
- **FTL3-304. Indexing repayments in repayment minting policy is troublesome**: Policy uses loan input indices to validate repayment outputs but doesn't skip indices for loans not creating repayments, requiring dummy outputs. Requires understanding protocol's business logic
- **FTL3-305. Burning and minting request and pool tokens is inconvenient**: Policy iterates over sorted mint records including burns but uses index to locate outputs, requiring dummy outputs when burns are interspersed with mints. Requires understanding protocol's business logic
- **FTL3-306. The principalLTV variable is overused**: Single variable holds different semantics, creating confusion. Code clarity and naming issue
- **FTL3-307. Ada oracle use is inconsistent**: Protocol handles Ada oracle differently for principal versus collateral, creating inconsistency. Code organization and consistency issue
- **FTL3-308. AMM formulas are based on a rational number that is then rounded**: Functions truncate rational numbers without considering whether lower or upper bound is needed. Requires understanding correct rounding direction for business logic
- **FTL3-309. Oracle safe-guards suggestion**: Oracle feeds may be unsuitable for all loan sizes, requiring size-based feed restrictions. Protocol design decision about oracle usage constraints
- **FTL3-310. Equity computation charges conversion fees to the lender**: Equity calculation for partial liquidation uses a conversion method that charges fees to the lender instead of borrower, preventing lender from recovering full remaining debt. Requires understanding protocol's business logic
- **FTL3-311. Pool KYC token signature can be reused to borrow more**: Signature includes borrow amount but isn't bound to specific instance, allowing reuse across multiple transactions to borrow more than signed amount. Protocol design decision
- **FTL3-312. Unlimited recasts do not work**: Documentation claims negative max_possible_recasts enables unlimited recasts, but validation uses simple < comparison that fails for negative values. Documentation issue
- **FTL3-313. Borrowers can avoid late repayment penalty**: Validator uses validFrom timestamp for penalty calculation without restricting validity range length, allowing borrowers to set past validFrom to avoid late penalties. Requires semantic context to know if lower bound or upper bound should be used
- **FTL3-314. Installment amounts might not add up to the total principal and interest**: Individual installments rounded up separately may total more than single repayment of principal plus interest due to accumulated rounding errors. Requires understanding rounding accumulation and business logic for correct approach
- **FTL3-315. Total installments field for perpetual loans**: Field is mandatory for all loans but doesn't make sense for perpetual loans and can be misused to allow collateral claim without principal repayment. Requires understanding which fields apply to which loan types
- **FTL3-316. Hash function mismatch in oracle key verification**: Validator uses blake2b_256 to hash oracle keys but compares against VerificationKeyHash type which uses blake2b_224, causing comparisons to always fail. Logic bug requiring understanding hash function semantics and type requirements
- **FTL3-317. Native tokens can be sent to programmable credential**: Protocol doesn't restrict native tokens from being sent to programmable credential, causing user inconvenience requiring specific wallets to retrieve. Protocol design decision
- **FTL3-318. Big bond reference inputs can cause DoS via transaction limits**: Bloated reference inputs can exceed transaction limits. It is a design issue
- **FTL3-401. Dropping a byte of a hash result is discouraged**: Token name generation drops byte from sha2_256 hash result instead of using full result from shorter hash function, which doesn't preserve studied pseudo-random properties. Code quality issue about cryptographic best practices
- **FTL3-402. Request id and pool id might be identical**: Request and pool tokens minted in the same transaction can receive identical asset names, causing conflicts in originAssetName field used off-chain. Requires understanding token name generation
- **FTL3-403. Equity payment is in the principal asset**: Equity returned to the borrower in principal asset instead of collateral asset, which is counterintuitive (e.g., receiving stablecoin instead of ADA when ADA collateral is liquidated). Protocol design decision
- **FTL3-404. Code quality, naming, and documentation issues**: Multiple issues including unused functions, misleading parameter names, shadowed type names, suboptimal validation patterns, incorrect comments, and outdated documentation. Code quality and maintenance issues already caught by standard tooling

## FluidTokens p2p Loans v3 v1.0

**Auditor**: TxPipe

**Auditee**: FluidTokens

**Description**: This project implements a peer-to-peer NFT-collateralized lending protocol on Cardano. Loans can be initiated by either borrowers or lenders, are repaid in installments, and are secured by NFTs. Access control and rights are managed via transferable borrower and lender bond NFTs, which grant the ability to repay, claim repayments, or seize collateral upon default.

### Findings

**Relevant**

- **FTA2-002. Borrower can claim his collateral prematurely**: Validator doesn't verify active loan output address matches own script hash, allowing borrower to redirect to controlled script and claim collateral without repaying.<br><ins>Detectable pattern</ins>: continuing output without address validation [MISSING-ADDRESS-VALIDATION]
- **FTA2-003. Borrower can steal the whole content of a collection offer pool**: Validator verifies only staking credential of ongoing collection offer output without checking payment credential, allowing attacker to redirect to controlled script.<br><ins>Detectable pattern</ins>: output validation checking only staking credential without payment credential validation [MISSING-ADDRESS-VALIDATION]
- **FTA2-304. Undefined repayments' staking credential**: Validator doesn't validate staking credential in repayment UTxOs, allowing borrowers to set arbitrary credentials.<br><ins>Detectable pattern</ins>: output validation without staking credential checks [UNVALIDATED-STAKING]

**May be relevant**

- **FTA2-001. The same bond NFT can be minted multiple times**: Token name uses bytearray.push with UTxO index without validating index < 256, causing wraparound where indices 1 and 257 produce identical token names and allowing duplicate NFT minting. Aiken-specific issue (bytearray.push wraps at 256); requires verification if equivalent issue exists in Haskell/Plinth.
- **FTA2-201. Double satisfaction in the loan amount payment**: Lender pays loan amount directly to borrower's address without unique identifier, allowing malicious lender to batch with another protocol's operation to satisfy both with single payment.<br><ins>Detectable pattern</ins>: value aggregation filtering by address without datum uniqueness check [DOUBLE-SATISFACTION]
- **FTA2-405. Undocumented assumptions and unchecked fields**: Several datum fields are assumed to be honest but not validated, allowing malformed UTxOs.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **FTA2-101. Lender and borrower bonds use the same policy**: Minting policies for lender and borrower bonds are identical because bondType variable is unused, preventing simultaneous minting required by validator. Logic bug requiring understanding the protocol's business logic
- **FTA2-102. Repayments are locked when active loan is claimed**: Claiming active loan burns lender NFT, preventing withdrawal of already-repaid installments. Protocol design issue that requires understanding the business logic
- **FTA2-301 Script hashes in the code are placeholders**: Hardcoded script hashes are invalid placeholders, making the protocol undeployable. Build and deployment issue
- **FTA2-302. Duplications of type declarations**: Type declarations duplicated across multiple files with different names (e.g., Datum vs RepaymentDatum vs ActiveDatum), requiring manual synchronization of changes. Code organization issue that requires understanding the semantics of each type
- **FTA2-303. Min Ada is not handled by the smart contract**: Protocol doesn't explicitly handle minAda costs, causing borrowers to pay additional minAda per installment UTxO to lenders. Protocol economic design issue requiring understanding intended minAda allocation
- **FTA2-401. Aiken warnings**: Codebase produces 122 compiler warnings for unused imports, types, constructors, and code style issues. Code quality issues already caught by standard tooling
- **FTA2-402. Helper functions are declared multiple times**: Helper functions duplicated across multiple files, sometimes unused. Code duplication issue already caught by standard tooling
- **FTA2-403. Graveyard design improvement**: Validator requires lender bond tokens sent to graveyard when withdrawing final repayment, forcing users to send tokens to graveyard then retrieve them when claiming repayments in arbitrary order. Protocol design decision about token lifecycle
- **FTA2-404. Incorrect documentation of the loan request's redeemer**: Documentation incorrectly describes lenderAddress field as borrower's bond NFT destination instead of lender's. Documentation issue
- **FTA2-406. Naming and shadowing:** Variable names are unclear, generic, or misleading. Code style and naming issues

## Perpetuals

**Auditor**: [Not provided]

**Auditee**: Strike Finance

**Description**: [No summary provided]

### Findings

**Relevant**

- **ID-11. Token Dust Attack on Pool Output:** Validator ensures NFT and underlying asset are present in pool output but doesn't restrict additional tokens, allowing attackers to bloat UTXO with arbitrary tokens causing size limit issues.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **ID-13. Missing Token Validation in Output Value:** Validators don't perform explicit validation of output token composition, allowing unexpected tokens to be added or token quantities altered without detection.<br><ins>Detectable pattern</ins>: subset value validation instead of equality check [TRASH-TOKENS]
- **ID-2. Missing Validation and Unbounded Fields in Position Datum:** Position datum fields not validated during minting, allowing unrealistic leverage, prices, or timestamps. Validity range is not restricted, allowing long ranges that distort time-sensitive calculations.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation combined with unrestricted validity range [PARTIAL-UNVALIDATED-DATUM] [VALIDITY-RANGE-BOUND]

**May be relevant**

- **ID-7. Missing Validation for position_asset_amount When Opening a Position:** Contract reads position_asset_amount when opening position but doesn't validate value, allowing zero, negative, or unrealistically large values inconsistent with collateral/leverage.<br><ins>Detectable pattern</ins>: output creation with incomplete datum field validation [PARTIAL-UNVALIDATED-DATUM]

**Not relevant**

- **ID-1. Use of Lower Bound for Current Time:** Interest fee calculation uses lower bound of validity range for current_time, allowing manipulation of interest fees. Requires understanding whether lower or upper bound is semantically correct for the specific calculation
- **ID-3. Lack of Supply Check May Cause Invalid Borrowing or Division by Zero:** Contract assumes underlying assets available in pool without checking, risking division by zero when 100% utilization reached or negative pool state when lended_amount exceeds available supply. Requires understanding business logic
- **ID-4. Unsafe Asset Comparison Allows Over-Lending:** Asset comparison using match(…, >=) can pass even when pool balance goes negative after borrowing, allowing lending more than available. Requires understanding correct validation approach for asset conservation
- **ID-5. Misuse of match Function for Multi-Asset Value Comparison:** match(…, >=) checks Lovelace with >= while assuming other assets unchanged, but fails when underlying assets are lent out causing token quantities to decrease. Requires understanding multi-asset value comparison semantics and when asset quantities should change
- **ID-6. Missing Update to total_lended_amount in Pool Datum:** Contract deducts lended_amount from pool value but doesn't update total_lended_amount datum field, desynchronizing actual balance from accounting metadata. Requires understanding which datum fields must be updated for specific operations
- **ID-8. Missing Validation of current_usd_price in Close Position Flow:** Protocol uses current_usd_price to calculate repayment without validating price is within reasonable range relative to lent amount and collateral, allowing manipulation of debt repayment. Requires understanding business logic
- **ID-9. Missing Validation of Lent Amount Returned to Pool in TraderClose:** Protocol calculates send_asset_amount returned to pool but doesn't validate it matches or exceeds original lent amount plus fees, allowing users to return less than borrowed. Requires understanding business logic
- **ID-10. Incorrect Liquidation Condition Due to Improper Loss Calculation:** Calculation subtracts total_value_loss from collateral_value, but when total_value_loss is negative the subtraction becomes addition, incorrectly increasing collateral value and preventing warranted liquidations. Requires understanding business logic
- **ID-12. Missing Token Burn in liquidate_position Flow:** Liquidation doesn't burn position token unlike close_position and cancel_position flows, leaving orphaned tokens on-chain. Requires understanding token lifecycle
