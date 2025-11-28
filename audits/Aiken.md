# Aiken Audit Summaries

## Splash Protocol Dex

**Description**
From: AnastasiaLabs To: Spectrum Labs

**Summary:** Unlike centralized exchanges that match buy and sell orders (aka CLOB exchanges), or constant product Automated Market Maker (AMM) exchanges, Splash uses different types of AMM liquidity pools, the Virtual Limit Order Book (VLOB), and combines them all. This allows different types of market makers to earn interest by providing liquidity as efficiently as they want, and traders to benefit from the best prices by tapping all liquidity in a single order.

**Findings**



**May be relevant**

- **ID-401. DAO can change the pool unrestricted:** Minting policy uses input index from redeemer to identify pool UTxO but doesn't verify the presence of pool NFT at that index, allowing attackers to substitute fake UTxO at pool address with arbitrary datum. Detectable pattern: input selection by redeemer-provided index without validating presence of identifying NFT at that input
- **ID-301 Zero spam:** Ensure that a minimal amount is actually transacted to avoid the possibility of endless deposit and redeem the same value. It is recommended to add some minimal fee to the redeem action to make it more expensive (also check for a minimal amount of tokens to be deposited instead of allowing 0). Note: could potentially be detected as operations that don't change state [UNCHANGED-STATE]
- **ID-302 Destroy allows hijacking the pool:** Not checking the output datum and value when the pool is spent with the Destroy redeemer. Detectable pattern: lack of check of the output datum and value. [MISSING-DATUM-VALIDATION]
- **ID-303. Lack of checking of the purpose field of the staking validator:** Staking validator doesn't validate ScriptPurpose field, allowing unintended execution of delegate or deregister actions instead of reward withdrawal, with deregistration invalidating DAO policy checks. Detectable pattern: staking validator without purpose field validation in script context
- **ID-201 Pool creation:** Pool NFT and liquidity token minting policies don't validate destination address or initial pool datum state during minting, relying entirely on off-chain code for correct initialization. Detectable pattern: minting policy missing destination address validation for minted tokens and checks on the correctness of the datum missing. [MISSING-ADDRESS-VALIDATION]
- **ID-202. Fee consistency checks:** Protocol validates individual bounds for feeNum and treasuryFee but doesn't enforce their relationship, allowing feeNum - treasuryFee to become negative and break all swap transactions. Detectable pattern: subtraction without relational constraint (eg. treasuryFee >= feeNum)
- **ID-101 Other token name:** Pool NFT and pool liquidity minting policies allow minting any number of tokens that are named differently than the configured ones. Detectable pattern: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-106. Duplicates in DAO signers:** DAOPolicy signer list doesn't check for duplicates, allowing some key holders to have amplified voting power and increased risk if compromised. Detectable pattern: lists of identity/authorization types (PubKeyHash, Address, ValidatorHash, etc.) without uniqueness validation
- **ID-108 Optimize output datum validation:** Validation on the continuing datum of the pool is done comparing individual fields of the input datum with the fields of the output datum. Instead, constructing the expected datum and compare it against the actual datum in the continuing pool output would be more efficient. Detectable pattern: multiple individual field comparisons between datums
- **ID-109 Treasury fee denominator must be equal to fee denominator:** Two constants with the same value that are used for the same purpose, could lead to misunderstandings in the future. Detectable pattern: Two constats
- **ID-111. Unnecessary if in correctLpTokenDelta:** Detectable pattern: conditional logic made redundant by subsequent constraints. Requires constraint solving and data flow analysis to detect unreachable branches
- **ID-114 Unnecessary datum re-construction:** Function extracts fields from input datum, reconstructs new datum from those fields, then compares to output datum instead of direct equality check. Detectable pattern: datum fields extracted and immediately used to reconstruct identical datum structure

**Not relevant**

- **ID-501 DAO can withdraw all user funds:** The minting policy (PFeeSwitch) doesn’t check that the output pool UTxO assets to exchange (treasuryX and treasuryY) are not negative in the validateTreasuryWithdraw function
- **ID-102 Potential of unsafe order types:** There is no restriction on the types of orders the pool can execute, anyone can implement their own “order” smart contract or interact directly with the pool. An issue may arise for incorrectly implemented order contracts that promise to execute a desired operation, but due to bug/malicious intent, the contract might not execute.
- **ID-103 Order types may be vulnerable to frontrunning:** There is no restriction on the types of orders the pool can execute, anyone can implement their own “order” smart contract or interact directly with the pool. This makes it possible for executors to front-run user swaps and other operations if the order type does not prevent this
- **ID-104 Confusing variable naming:** Variables feeNum and feeDen are named misleadingly - feeNum / feeDen represents the portion user receives after fees rather than the fee ratio itself, potentially causing developer confusion. Code clarity/naming issue
- **ID-105 Setting invalid/empty DAOPolicy:** DAOPolicy field in the datum is used to check the execution of dao actions on the pool. Setting the DAOPolicy value to a wrong value or setting it to an empty list will make accessing the treasury amounts in the pool impossible
- **ID-107 DAO signers:** The list of accepted signatories cannot be examined by looking at the onchain data and certain operations can change the DAOPolicy. There is no way to check the actual list of a DAOPolicy
- **ID-110 Redundant rounding:** Redundant call to the rounding function
- **ID-112 Fee consistency checks:** Upper limit of the swap fees should be modified to a reasonable price
- **ID-113 Fixed Balance pool weights:** Implementation only supports two-token pools with fixed 20:80 ratio instead of customizable weights. Business logic issue



## BookToken

**Description**
From: Tweag To: Book.io

**Summary:**The BOOK token platform and exchange is the first of its kind by using reading to “mine” for tokens. In order to obtain $BOOK tokens, users must purchase NFT books – and then read them. Readers are ONLY rewarded with native $BOOK tokens based on how much they read. The more you read, the more $BOOK you can earn.
These Cardano-native tokens are redeemable on the platform to purchase new eBooks and Audiobooks – and also transferrable from the BOOK Token platform to other compatible wallets and crypto exchanges.


**Findings**

**May be relevant**

- **2.2.4.3. Some helper functions impact compiled script size:** Helper functions are trivial wrappers that could be inlined. Detectable pattern: functions that only pattern match or call another function with fixed arguments

**Not relevant**

- **2.2.1.1. The reference token is not an NFT:** Reference token is burned and reminted on updates instead of using one-shot NFT, adding transaction costs and relying on admin to avoid multiple tokens. Requires semantic understanding of intended token lifecycle
- **2.2.1.2. Obsolete tokens are not burned:** Obsolete tokens sent to script address instead of being burned via minting policy. Requires semantic understanding of intended token lifecycle and destination script purpose
- **2.2.1.3. Custom failure scripts have no upside over the canonical failure script:** Parametrized scripts used instead of canonical failure script for locking obsolete tokens. Requires semantic understanding of script purpose and whether parametrization provides value
- **2.2.2.1. New tokens can be stolen by abusing an undesired rounding-up operation:** Integer division with negative dividend rounds toward negative infinity, causing amounts to round up in absolute value. Requires semantic understanding of correct rounding direction for the business logic
- **2.2.3.1. Unlock transaction is underspecified:** Validator logic for unlock transactions not explicitly documented. Documentation issue
- **2.2.3.2. Swap transaction is underspecified:** Validator logic for swap transactions not explicitly documented. Documentation issue
- **2.2.4.1. Conditionals could be avoided using short-circuiting in boolean conjunctions:** Suggests rewriting conditionals using boolean conjunction for readability. Code style issue already caught by standard tooling
- **2.2.4.2. Unnecessary named intermediates:** Excessive use of intermediate variables may reduce code clarity. Code style issue
- **2.2.4.4. Comments are lacking in the code:** Source code would benefit from more description comments. Documentation issue


## MinSwap Dex

**Description**
From: Tweag To: MinSwap Labs

**Summary:** Minswap is a Cardano DEX where LPs can see the potential APY of a pool before providing liquidity, and then make an informed choice about which pools they chose to provide liquidity to. MinSwap DEX also employs an automated yield farm strategy, which helps LPs to rebalance the liquidity they provided into the most efficient pools.


**Findings**

**May be relevant**

- **2.2.1.1. Unauthorized Redeeming of Open Orders:** Validator checks for presence of other script inputs without verifying their redeemers, allowing authorization bypass. Detectable pattern: validation depends on other script inputs without checking txInfoRedeemers, though determining whether specific redeemers are required for validation requires semantic understanding [UNCHECKED-REDEEMER]
- **2.2.1.2. LP Tokens Can Be Duplicated:** Minting policy checks for pool output with NFT but doesn't verify which redeemer was used on pool input, allowing unauthorized minting. Detectable pattern: minting policy depends on other script inputs/outputs without checking txInfoRedeemers, though determining whether specific redeemers are required for validation requires semantic understanding [UNCHECKED-REDEEMER]
- **2.2.1.3. Unauthorized Hijacking of Pools Funds:** Validator selects continuing output by NFT presence without verifying destination address, allowing pool funds to be redirected to attacker's script. Detectable pattern: continuing output selected by token presence without address validation (should use getContinuingOutputs or verify scriptAddress)  [MISSING-ADDRESS-VALIDATION]
- **2.2.4.3. Large Refactoring Opportunities:** Heavy code duplication across order validation functions could be refactored for better maintainability

**Not relevant**

- **2.2.2.1. Batchers Can Choose Batching Order:** On-chain code doesn't enforce chronological order processing. Requires semantic understanding of whether order matters for protocol fairness
- **2.2.2.2. Batcher Is Not Allowed to Apply Their Own Orders:** Code filters out orders from specific addresses when processing. Requires semantic understanding of whether address-based filtering is intentional or erroneous
- **2.2.3.1. Batchers Can Choose Pools:** Batchers can execute orders against any pool, including custom pools. Requires semantic understanding of intended protocol constraints
- **2.2.3.2. Batchers Licenses Cannot be Revoked:** Batcher licenses are irrevocable until expiration. Protocol governance issue
- **2.2.3.3. Assumptions on Batcher's Licenses Distribution:** Specification doesn't document how batcher licenses are distributed and renewed. Documentation issue
- **2.2.3.4. Pools Cannot Be Closed:** Pools can be created but not closed, permanently locking creator's minAda. Requires semantic understanding of intended pool lifecycle
- **2.2.4.1. Reliance on Indexes Into ScriptContexts' txInputs and txOutputs:** Validators use redeemer-provided indices to select inputs, relying on transaction ordering. Requires semantic understanding of whether input ordering constraints are necessary
- **2.2.4.2. Duplication of ScriptContext Definition:** Custom ScriptContext definition omits getContinuingOutputs function, contributing to vulnerability 2.2.1.3. Code organization and maintenance issue
- **2.2.4.4. Protocol Specification Lacking:** Documentation lacks formal specification defining correct behavior and operations

## Minswap AMM Dex v2

**Description**
From: Certik To: MinSwap Labs

**Summary:** Minswap AMM V2 uses a constant-product formula (x * y = k) to price trades, ensuring that larger trades incur progressively worse rates to protect liquidity. To handle concurrency on Cardano, it uses a batching system where user actions become orders that are collected and executed together in a liquidity pool. Only whitelisted wallets, called batchers, are allowed to trigger and process these batch transactions.


**Findings**

**May be relevant**

- **MIN-01 Logical issue in fee settings:** Fee setting functions currently do not safeguard against the possibility of setting both the numerator and denominator of fees to zero. Detectable pattern: No check on denominator and numerator value being zero.
- **ORD-02 Missing check on io_ratio_denominator:** Redundancy in the validation check and lack of validation due to typo mistake. Detectable pattern: duplicate checks.


**Not relevant**

- **FAC-01 Creation of pools with invalid parameters:** Needs checks to ensure that reserve values and total liquidity fall within practical ranges. Requires understanding business logic to determine which are practical ranges.
- **VAL-01 Centralization related risks:** Centralized privileges or roles in the protocol could be improved via decentralized mechanisms
- **FAC-02 Pool creation allows complete asset withdrawal:** Should allow for the replenishment of pool liquidity. Requires understanding business logic.
- **AUT-01 Incorrect comment:** Code quality issue.
- **GLOBAL-01 Unit test documentation:** Documentation and code quality issues.
- **MIN-02 TODO comments:** Code quality issues.
- **ORD-01 Missing formulas for WithdrawImbalance and PartialSwap:** Code quality issues.
- **ORD-02 Typos:** Code quality issues.
- **MAT-01 Potential optimization in math.calculate_withdraw_imbalance():** Code quality issues.

## MinSwap Dex v2 reaudit

**Description**
From: Certik To: MinSwap Labs

**Summary:** Minswap AMM V2 uses a constant-product formula (x * y = k) to price trades, ensuring that larger trades incur progressively worse rates to protect liquidity. To handle concurrency on Cardano, it uses a batching system where user actions become orders that are collected and executed together in a liquidity pool. Only whitelisted wallets, called batchers, are allowed to trigger and process these batch transactions.


**Findings**

**May be relevant**
- **ORD-02 Unoptimized check:** Using empty strings to verify if assets are ADA. It is recommended to use functions that do the verification. Detectable pattern: use of empty string to compare an asset name.

**Not relevant**

- **TYP-01 Potential for multiple roles per address:** There are no constraints to prevent a single address from being assigned multiple or even all roles.
- **MIN-01 Centralization related risks**
- **ORD-01 Missing check for batcher fee in donation orders:** Uncertainty about whether the batcher fee is correctly paid in all scenarios. Requires understanging of the business logic.

## Nuvola

**Description**
From: TxPipe To: Nuvola

**Summary:** Nuvola is a DePIN aggregator, a first for Cardano.  DePIN is short for Decentralized Physical Infrastructure Network, which brings cutting-edge Web 3.0 solutions to a broad number of real-world applications. Being decentralized at heart, DePIN companies pay their network supporters, such as node operators, rewards as incentive for maintaining the network.

**Findings**

**May be relevant**

- **NUV-001. Reward Token can be stolen on Process Reward operation:** Validator uses subset value check on user output (not script address) allowing authorization token to escape burning and be reused. Detectable pattern: subset value validation without exact equality check [TRASH-TOKENS]
- **NUV-002. Rewards Claim UTxOs can be deleted without owner consent and fees stolen:** Minting policy validates token presence in outputs without checking mint amount is positive vs negative, allowing burning to consume UTxOs without authorization. Detectable pattern: minting validation without checking token quantity sign [INCOMPLETE-TOKEN-VALIDATION]
- **NUV-003. Stake Tokens can be arbitrarily minted during the payback loan operation:** Minting policy validates operation type without restricting quantity, combined with subset value check on outputs allowing tokens to go anywhere. Detectable pattern: minting without quantity validation and subset value validation [INCOMPLETE-TOKEN-VALIDATION] [TRASH-TOKENS]
- **NUV-102. Genesis stake ref can be forged on LoanDatum:** Datum field copied from input to output without validation that values match, allowing arbitrary data. Requires understanding which datum fields must be preserved across operations [UNVALIDATED-CONTINUING-DATUM]
- **NUV-204. Prevent inclusion of reference scripts:** Validator doesn't validate reference script field of outputs, allowing arbitrary reference scripts to be attached and increase future transaction fees. Detectable pattern: output validation without reference script field checks [UNVALIDATED-REFERENCE-SCRIPT]
- **NUV-301. CreateLend spend redeemer can be used to remove a lend UTxO:** Wildcard pattern matching on redeemer allows unintended redeemer types to trigger logic. Detectable pattern: wildcard pattern matching on redeemer types
- **NUV-303. Claim reward tokens can be minted with arbitrary token name:** Minting policy validates expected token name but doesn't restrict minting other token names under the same currency symbol. Detectable pattern: token name validation without restricting other names [INCOMPLETE-TOKEN-VALIDATION]
- **NUV-305. Trash tokens can be added to multiple UTxOs:** Validator checks required tokens are present but doesn't restrict additional tokens, allowing arbitrary tokens to bloat UTxOs and increase costs. Detectable pattern: subset value validation without exact equality check [TRASH-TOKENS]

**Not relevant**

- **NUV-101. It is possible to create loans that have expired:** Validation uses inequality check instead of equality for expiration date, allowing past dates. Requires understanding business logic to determine correct comparison operator
- **NUV-202. Missing checks for some LendingDatum fields:** Datum fields not validated during creation, allowing unreasonable values. Requires understanding which fields need validation
- **NUV-203. Staking credentials not being checked in two operations:** Script UTxO creation doesn't validate staking credential matches user's expected credential. Requires understanding protocol design and user expectations
- **NUV-302. Spendable loan UTxO without a loan token:** Spend validation requires token burning, making UTxOs without tokens permanently unspendable. Requires understanding business logic
- **NUV-304. Optimization for the Apply Loan operation:** Multiple opportunities to reduce computation costs through better data handling and removing redundant checks. Requires understanding performance implications and protocol-specific code structure

