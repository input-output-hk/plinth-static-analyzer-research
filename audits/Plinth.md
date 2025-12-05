# Plinth Audit Summaries

## MuesliSwap

**Description**
From: MLabs To: MuesliSwapTeam

**Summary:** MuseliSwap's DEX is based on a decentralized version of the traditional order book style exchange used in stock markets. Users submit trades to a smart contract and decentralized matchmakers scan the blockchain to settle trades, charging a small fee to do so.

**Findings**

**Relevant**

- **MSW-B01. Dangerous Use of unstableMakeIsData:** Using unstableMakeIsData assigns constructor indices based on declaration order, which can change unpredictably and break deployed contracts. Detectable pattern: usage of unstableMakeIsData function

**May be relevant**

- **MSW-A01. Double Satisfaction Attack:** Validator aggregates payment outputs by checking only recipient's address without verifying uniqueness. Allows attackers to satisfy multiple orders with single payment by batching orders from the same creator. Detectable pattern: value aggregation filtering by address without datum uniqueness check [DOUBLE-SATISFACTION]

**Not relevant**

- **MSW-A02 & MSW-A03. Partial Order Matching Vulnerabilities:** Inverted logic and incorrect rounding direction in partial matching. Semantic logic errors requiring understanding of intended economic behavior
- **MSW-B02. Redundant Conditional Expressions:** Pattern like "if x then True else False" that can be simplified. Already caught by standard tooling (HLint)
- **MSW-B03. Reusable Functions in Local Scope:** Utility functions defined in local scope rather than top-level. Code organization issue requiring semantic judgment
- **MSW-B04. Misleading Eq Instance:** Eq instance that omits fields from equality comparison. Requires semantic understanding of which fields should participate in equality
- **MSW-B05. Confusing Implementation of Rounding Function:** Function handles impossible cases given domain constraints. Requires understanding input range constraints and business logic

## AADA

**Description**
From: VacuumLabs To: AADA Finance

**Summary:** A peer-to-peer lending protocol that allows users to borrow and lend crypto assets directly on the blockchain. Borrowers can request loans by posting collateral, and lenders can provide the requested loan amount. The protocol includes mechanisms for loan repayment, interest calculation, and collateral liquidation if borrowers default.

**Findings**

**May be relevant**

- **AADA-002 & AADA-003. Double Satisfaction Attack:** Validator aggregates payment outputs by checking only recipient's address without verifying uniqueness. A lender can batch multiple loan requests from the same borrower in a single transaction, providing only the maximum loan/collateral amount instead of the sum. The valuePaidTo function aggregates all outputs to an address without uniqueness checks, similar to MuesliSwap A01 but hidden behind helper function calls instead of direct list comprehensions
- **AADA-004. Resource DoS attack:** Validator doesn't restrict which tokens can be included in output UTxOs, allowing attackers to bloat outputs with arbitrary tokens. Detectable pattern: subset value validation instead of equality check [TRASH-TOKENS]
- **AADA-101 & AADA-102. Timestamp Manipulation for Interest Calculation:** Interest calculation uses user-provided timestamps from redeemers (interestPayDate, lendDate) without validating them against transaction's validity range. Allows borrower to set future date for zero interest or lender to set past date forcing immediate full interest payment
- **AADA-104, AADA-303 & AADA-403. Incomplete Token Tuple Validation:** Validators check for token presence in minting operations without verifying whether tokens are being minted (positive) versus burned (negative), or whether specific token names match expectations. Detectable pattern: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **AADA-202. Losing partial interest precision:** Unnecessary intermediary rounding in interest calculation causes lenders to lose money. Pattern: division before multiplication (elapsed / total) _ amount loses precision compared to (elapsed _ amount) / total.
- **AADA-302. Rounding of interest favors borrower:** Interest calculation uses floor division, favoring borrower. Static analyzer could detect rounding direction (div vs divCeiling) but determining correct direction requires business context.
- **AADA-304. Constant protocol staking key rewards capped:** Staking credential hardcoded in contract parameters rather than datum causes reward saturation once pool reaches cap. Detectable pattern: staking key in immutable parameters vs variable datum. [IMMUTABLE-CREDENTIAL]
- **AADA-305. Oracle policy not checking destination of all tokens:** Policy checks at least one oracle token goes to dest address using existential quantifier, but doesn't enforce all oracle tokens go there. Detectable pattern: any vs all for token destination validation. [MISSING-ADDRESS-VALIDATION]
- **AADA-402. Inverted percentage calculation:** Two inverted divisions in separate functions accidentally cancel out (a/b) used as divisor → multiply by b/a). Detectable pattern: reciprocal cancellation, but requires cross-function data flow analysis.

**Not relevant**

- **AADA-001. Borrower/Lender tokens do not have to be NFTs:** On-chain validation doesn't check currency symbol of Borrower/Lender NFTs, allowing use of fungible tokens instead
- **AADA-005. Lender can take collateral when providing loan:** Inverted comparison operator in collateral validation
- **AADA-006. Lender can take whole collateral in liquidation:** After deadline, lender can claim entire collateral without returning excess value
- **AADA-007. Inability to stop compromised oracle node:** Oracle policy has no mechanism to replace/remove compromised nodes
- **AADA-008. Oracle price consensus uncertainty:** No clear rules for how oracle nodes reach consensus on exchange rates during liquidation.
- **AADA-105. Structure of liquidation transaction not enforced:** Validator does not enforce proper structure of liquidation transactions, relying instead on off-chain closed-source code run by oracle nodes
- **AADA-201. Repaying loan near deadline can be DoS-ed:** Lender can spam the chain near deadline to prevent borrower's repayment transaction, then liquidate the whole collateral. Vulnerability exploits timing and transaction ordering rather than code patterns
- **AADA-203. Incomplete documentation:** Missing or outdated documentation for business requirements, staking rewards allocation, interest payment rules, and liquidation logic. Not a code issue
- **AADA-204. Lack of oracle decision transparency:** Oracle nodes make binary sign/no-sign decisions without clear on-chain verification of their reasoning. Design issue, not detectable code pattern
- **AADA-301. Loss of Min-ADA:** Borrower's min-ADA (~2 ADA) from Request UTxO can be claimed by Lender. Economic/fairness issue, not code pattern
- **AADA-401. Dead Code:** Unused functions and datum fields. Already caught by standard tooling (GHC, HLS, HLint)
- **AADA-404. Naming inconsistencies:** Misleading terminology (calling fungible tokens "NFTs"), inconsistent camel case, unclear function names. Code style issue, not security vulnerability.
- **AADA-405. Duplicated logic:** Identical scripts (Liquidation.hs/Interest.hs) and re-implemented utility functions.

## AADA Debt/Request

**Description**
From: VacuumLabs To: AADA Finance

**Summary:** Another VacuumLabs audit of AADA Finance's P2P lending protocol. In this version, the Lender creates a "debt request" specifying loan terms, and the Borrower fulfills it by locking collateral. Both parties receive bond NFTs (Lender NFT and Borrower NFT) that can be traded, transferring responsibilities with ownership.

**Findings**

**Not relevant**

- **AADADR-301. Lender sets collateral amount for borrower:** Design/business logic issue about who controls collateral specification
- **AADADR-401. Branch validation logic depending on redeemer:** Uses disjunctive validation (a1 ∧ a2 ∧ ... ∧ ak) ∨ (b1 ∧ ... ∧ bl) instead of redeemer-based branching. Code style/optimization issue, not security vulnerability
- **AADADR-402. borrowerGetsWhatHeWants is not used:** Unused function. Already caught by standard tooling
- **AADADR-403. Redundant function for checking transaction inputs:** Function txHasOneDebtRequestInputOnly checks subset of what txHasOneScInputOnly already checks. Code duplication issue

## Ardana dUSD

**Description**
From: VacuumLabs To: ArdanaLabs

**Summary:** A stablecoin protocol on Cardano that creates dUSD backed by ADA collateral. Core components include Vaults (hold collateral and manage loans), Buffer (stabilizes dUSD price via auctions), Price Module (aggregates price data), and Protocol Parameters Module. Users can create vaults, take loans by minting dUSD, and face liquidation if collateralization ratio drops below threshold.

**Findings**

**May be relevant**

- **ARD-005. Use of fixed admin keys:** Admin keys hardcoded in script addresses making rotation impossible if compromised. Detectable pattern: admin credentials in immutable script hash vs mutable datum (maybe from a UTxO of a reference script) [IMMUTABLE-CREDENTIAL]
- **ARD-101. Deposited ADA is not staked:** Vault collateral ADA not staked, users lose potential rewards. Detectable pattern: missing staking credential in script addresses
- **ARD-102. Limited storage of historical stability fees:** Historical fee changes stored in datum map, limited to 20 entries due to datum size constraints. Long-term vaults lose early fee history when >20 changes occur, enabling interest underpayment. A map with size limits is syntactically detectable, but determining this creates an economic exploit requires understanding what the map stores and how it's used in calculations
- **ARD-201. Timestamp is the lower bound of 12-hour validity windows:** Design uses lower bound of txInfoValidRange for timestamp, allowing users to manipulate timestamp within the 12-hour window for price speculation. Detectable pattern: lower vs upper bound usage, but assessing exploit might not be so easy [VALIDITY-RANGE-BOUND]
- **ARD-401. Price module contention:** Price module UTxO spent for reads, causing congestion. Detectable pattern: UTxO spent but recreated with identical datum/value (read-only access), should use reference input instead. [READ-ONLY-SPEND]

**Not relevant**

- **ARD-001.Unclear specification of Buffer and auctions:** Unclear specification of Buffer and auctions: Missing design details for Buffer mechanism, auction parameters, and stability fee calculations. Specification/documentation issue
- **ARD-002. Liquidation burns too little, duplicating value:** Protocol economic design issue about token supply management, not code pattern
- **ARD-003. Fixed exchange rate slows down liquidations:** Liquidation uses fixed 1:1 dUSD:USD rate instead of actual market price, making liquidations unprofitable when dUSD > 1 USD and too profitable when dUSD < 1 USD. Protocol economic design, not code pattern
- **ARD-004. Unclear specification of governance system:** Specification mentions three admin keys hard-wired into the system but doesn't clarify if any single key has full access or if multisig is required. Specification/documentation issue
- **ARD-202. Liquidations not batched, slowing process down:** Design disallows batching multiple liquidations in one transaction, potentially slowing liquidation response during market crashes. Design/efficiency issue about missing functionality, not code vulnerability
- **ARD-203. Unprofitable small liquidations can make dUSD undercollateralized:** Small vaults unprofitable to liquidate (3% discount < tx fees) can accumulate, causing systemic undercollateralization during price crashes. Protocol economics issue about missing minimum size constraints, not code pattern
- **ARD-301. Unfulfillable acceptance criterion for vault:** Specification claims vaults liquidated within 1 hour of becoming sick, but design requires 2+ hours (hourly price updates + 2 consecutive sick reports needed). Specification inconsistency, not code issue

## Cerra P2P Lending

**Description**
From: Tweag To: Cerra

**Summary:** A peer-to-peer lending protocol that empowers users to leverage or hedge their cryptocurrency holdings.

**Findings**

**Relevant**

- **2.2.1.1. All assets can be stolen from lending UTxOs:** Minting policy searches for continuing output by datum type without verifying destination address, allowing assets to be redirected to any address with matching datum. Detectable pattern: output filtered by datum properties without scriptAddress/validatorHash check [MISSING-ADDRESS-VALIDATION]

**May be relevant**

- **2.2.1.2. Loans can be liquidated before their deadline:** Deadline validation uses upper bound of validity range instead of lower bound, allowing lenders to liquidate early by setting upper bound after deadline while lower bound remains before deadline. Detectable pattern: lower vs upper bound usage for deadline checks, but determining correct bound requires understanding check semantics [VALIDITY-RANGE-BOUND]
- **2.2.1.3. Lenders can increase the amount of due interests:** When accepting a borrow offer, validation checks loanStartTime equals lower bound of validity range, allowing the lender to set arbitrarily early start date (and matching lower bound) while transaction executes later, inflating interest period. Detectable pattern: lower vs upper bound usage for timestamp validation, but determining correct bound requires understanding check semantics [VALIDITY-RANGE-BOUND]
- **2.2.2.1. The amount of Ada in the script is too strict:** Validators enforce exact ADA amounts (2 or 4 ADA) in script outputs using equality checks, but Plutus V2 minimum ADA depends on UTxO size. Detectable pattern: exact equality check on ADA amounts in output validation (potentially outdated Plutus V1 pattern or misunderstanding of minUTxO)
- **2.2.2.3. Borrower NFT could be referenced when retrieving loan assets:** Transaction spends and recreates UTxO containing borrower NFT to prove ownership instead of using reference input [READ-ONLY-SPEND]
- **2.2.2.6. Inputs other than the script ones should be allowed to have datums:** Validator searches for "the only input with any datum" instead of filtering by specific datum type and address, preventing transactions from including other inputs with datums.
- **2.2.4.1. Validation guards are not centralized:** Validation logic scattered across partial utility functions that throw errors internally, rather than centralized guards in the validator with total utility functions returning Maybe/Either. Detectable pattern: utility functions containing error/traceError/debugError calls, hiding validation logic away from main validator [PARTIAL-UTILITIES]

**Not relevant**

- **2.2.2.2. Redeemers should reflect each distinct action:** Single redeemer type used for multiple actions, with context used to discriminate between them instead of explicit redeemer variants for each action.
- **2.2.2.4. Most of the contract logics should appear in the lending script:** Validation logic concentrated in minting policies (least contextual knowledge) rather than spending script validator.
- **2.2.2.5. Loans using Ada as asset are treated in a convoluted way:** Conditional logic for ADA vs non-ADA assets hidden in helper functions rather than explicit branching in validation logic. Code organization issue requiring semantic judgment about proper abstraction levels
- **2.2.3.1. The factory NFT still requires offchain verifications:** Factory NFT identified by elimination rather than explicit currency symbol check, allowing protocol to operate with any token, moving trust from on-chain validation to off-chain filtering.
- **2.2.3.2. Oracle reference outputs are unspecified:** Code expects oracle UTxOs at hardcoded positions (0,1) in reference inputs list, which breaks if reference script is added. Documentation/specification issue about undocumented ordering constraint
- **2.2.3.3. Time units are underspecified and heterogeneous in the lending datum:** Time-related datum fields use inconsistent units (milliseconds, seconds)
- **2.2.4.2. Bang patterns and laziness are used in an erratic manner:** Inconsistent use of bang patterns and lazy evaluation of error-throwing assignments makes it unclear when validation checks execute based on control flow. Related to 2.2.4.1 - laziness compounds the hidden validation issue [PARTIAL-UTILITIES]
- **2.2.4.3. Function names are sometimes misleading:** Function names don't accurately describe their behavior
- **2.2.4.4. The overall code quality and use of Haskell can be improved:** Multiple anti-patterns like if a then True else False, fromJust (Just value), overuse of list comprehensions, and missing pattern matching opportunities. Code quality issues catchable by standard tooling (haskell-language-server, linters)

## Cerra AMM

**Description**
From: Tweag To: Cerra

**Findings**

**May be relevant**

- **2.2.0.2. Checking the batcher token could be done with a reference input:** UTxO containing batcher token spent and recreated to prove ownership instead of using reference input. Detectable pattern: UTxO spent but recreated with identical datum/value (read-only access), should use reference input instead [READ-ONLY-SPEND]
- **2.2.0.5. Usage of map for additional info in PoolDatum is inadapted:** Map with fixed required keys used instead of dedicated datum fields, reducing type safety and allowing misspellings/omissions. Detectable pattern: Map type with validation checking for specific hardcoded keys, indicating fixed structure better represented as record fields
- **2.2.0.7. Precision is a static value that takes up space in pool datums:** Datum fields validated to equal hardcoded constant (12), making datums unnecessarily large. Detectable pattern: datum fields with validation requiring exact match to hardcoded constant (static values bloating datum)
- **2.2.0.8. Implementation of equality test for PoolDatum is incomplete:** Eq instance for PoolDatum omits pdPrice field from equality check, comparing only subset of fields. Detectable pattern: custom Eq instance that doesn't compare all data type fields (syntactically identifiable but determining if omission is intentional design vs oversight requires semantic understanding of domain model)

**Not relevant**

- **2.2.0.1. No fairness is guaranteed nor encouraged when applying orders:** No on-chain constraints on order selection or execution sequence, allowing batcher to arbitrarily choose which orders to execute and in what order. Design issue about missing fairness mechanism, not a code pattern
- **2.2.0.3. The product lacks a detailed specification:** Missing specification document describing high-level transaction purposes and low-level on-chain actions. Documentation issue, not a code pattern
- **2.2.0.4. The pool NFT and factory token might be merged:** Two separate minting policies (factory token for initialization validation, pool NFT for unique ID) could potentially be merged into one. Design suggestion, not a code pattern
- **2.2.0.6. Remnants of the profit sharing feature take up space:** Dead code issue, not a code pattern

## Djed

**Description**
From: Tweag To: IOG

**Summary:** Cardano's native overcollateralized stablecoin, developed by IOG and powered by COTI. DJED is backed by ADA and uses SHEN as a reserve coin.

**Findings**

**May be relevant**

- **2.2.1.1. Additional tokens of the order token currency symbol can be minted:** Validation checks only the specific token name ("DjedOrderTicket") without restricting other token names under the same currency symbol, allowing arbitrary minting of tokens with different names. Detectable pattern: token validation filtering by specific token name, but detecting the missing check for other token names requires identifying absence of negative validation (proving no other tokens exist)

**Not relevant**

- **2.2.2.1. Non-Djed tokens are not guaranteed to go back to order submitters:** Validator doesn't require extra tokens to be returned to order submitters during processing, allowing operators to claim them. Determining which tokens are "protocol-related" vs "extra/unrelated" requires semantic understanding of protocol design
- **2.2.2.2. Documentation comments and in-code comments are lacking:** Many helper functions lack documentation comments and in-code comments, especially for complex logic. Documentation issue
- **2.2.2.3. There is no text in the specification for mintSC and mintRC:** Functions computing reserve state and order validity lack specification documentation. Documentation issue
- **2.2.2.4. Minting non-Djed tokens at the same time as an order token is impossible:** Order token minting policy fails when other tokens are minted in the same transaction, but this restriction is undocumented. Documentation/specification issue about undocumented constraint
- **2.2.2.5. Requirements about sorting orders when processing them are inconsistent:** Documentation claims input UTxOs must be sorted, but validator is indifferent to sorting and performs internal sorting. Documentation inconsistency with actual behavior
- **2.2.2.6. There are discrepancies regarding the positions of UTxOs in outputs:** Specification states "some output" must be produced, but implementation requires specific position (usually first). Documentation inconsistency with implementation
- **2.2.2.7. Specification and implementation disagree on absence of stablecoin in input:** Implementation checks no stablecoin in input during configManagerMinting, but specification doesn't mention this requirement. Documentation inconsistency with implementation
- **2.2.2.8. The documentation comment in the stablecoin contract is incorrect:** Comment claims only the operator can use stablecoin contract, but users can spend stablecoin UTxO when submitting complaints. Documentation error
- **2.2.2.9. The oracle and stablecoin cannot be terminated simultaneously:** Implementation requires stablecoin to be only input during termination, but specification only requires no request UTxO consumed, preventing simultaneous oracle/stablecoin termination. Specification/implementation discrepancy about intended behavior
- **2.2.2.10. Specification could be improved with visual representations:** Complex value flows and time-related variables would benefit from visual diagrams and timelines. Documentation improvement suggestion
- **2.2.3.1. The onchain sorting algorithm for orders is costly:** Uses insertion sort (O(n²)) for sorting orders instead of O(n log n) algorithms. Performance optimization consideration dependent on expected input sizes and usage patterns, not a detectable anti-pattern
- **2.2.3.2. The implementation of function processOrders lacks clarity:** Function implementation with recursive helpers lacks comments on different cases and unclear accumulator parameter positioning. Code clarity issue
- **2.2.3.3. Part of the datum in the stable coin UTxO is redundant with the value:** Datum fields for reserve amounts and coin circulation count duplicate information already in UTxO value. Design issue about data redundancy
- **2.2.3.4. Recursive operations are not implemented using a "fold" function:** Custom recursive functions used instead of fold family functions where applicable. Code clarity issue
- **2.2.3.5. Functions mintSC and mintRC have misleading names:** Function names suggest minting/burning but don't actually mint or burn tokens, and their actual role is unclear. Naming/documentation issue
- **2.2.3.6. Interval variable names are confusing:** Variables named "interval" actually represent interval length/duration, not time ranges. Naming issue
- **2.2.3.7. Function validMinAdaTransfer uses Either against common usage:** Returns Left for success and Right for failure, reversing common Either convention. Code style issue
- **2.2.3.8. There are inconsistent debug messages in reward fee contract:** Error messages inconsistently describe either expected behavior or incorrect behavior that occurred. Code style issue
- **2.2.3.9. Some documentation comments are not detailed enough:** Documentation comments too vague to provide useful information about requirements and constraints. Documentation issue

## Optim Bonds and Pools

**Description**
From: Tweag To: Optim

**Summary:** Users of the Optim protocol deposit their assets into pools called vaults. These pools of funds are invested by smart contracts that are coded with one or more investment strategies and designed to allocate and reallocate funds to maximize the assets yield potential. Each vault deals with a different element of yield generating Cardano DeFi.

**Findings**

**May be relevant**

- **2.3.1.1. uniqNFT can be redirected when posting a bond:** Minting policy doesn't verify minted uniqNFT goes to correct validator address, allowing redirection to arbitrary address. Detectable pattern: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **2.3.1.4. Pool tokens target is unchecked:** Minting policy doesn't verify where minted pool tokens are sent, allowing redirection to arbitrary addresses or minting without pool creation. Detectable pattern: minting policy missing destination address validation for minted tokens [MISSING-ADDRESS-VALIDATION]
- **2.3.1.8. Funds can be locked after datum tampering:** Validator doesn't verify continuing output datum during partial redemption, allowing attackers to modify datum fields and break subsequent operations. Detectable pattern: continuing output at same script address without any datum validation (neither full equality nor field-specific checks) [UNVALIDATED-CONTINUING-DATUM]

**Not relevant**

- **2.3.1.2. Open validator can contain unsound bonds:** User can redirect uniqNFT and pay it to Open validator with crafted datum, creating fake bond UTxO that accepts margin deposits. Consequence of 2.3.1.1 (if address validation is fixed in 2.3.1.1, this issue is prevented)
- **2.3.1.3. Funds can be locked with irrevocable staking rights using unsound bonds:** Margin added to fake bonds (from 2.3.1.2) becomes permanently locked with no redemption possible. Consequence of 2.3.1.1 and 2.3.1.2 (if address validation is fixed in 2.3.1.1, this issue is prevented)
- **2.3.1.5. Pool target can be compromised using another uniqNFT:** Attacker can substitute bonds by including different bond's uniqNFT in margin, causing pool to match wrong bond. Consequence of uniqNFT redirection from 2.3.1.1 and 2.3.2.1 (if address validation is fixed, this issue is prevented)
- **2.3.1.6. Pools can be emptied by matching a small bond:** Validator doesn't verify bond tokens equal pool tokens during matching, allowing attacker to match pool to bond with fewer tokens and steal remaining value. Requires understanding protocol invariant that bond and pool token quantities should be equal
- **2.3.1.7. Pools can be emptied by matching a bond with the cancel redeemer:** Validator assumes specific redeemer (WriteBond) when consuming BondWriter UTxO but doesn't verify it, allowing use of CancelBond with different validation logic. Requires semantic understanding of which redeemer should be used, and redeemer checks may be implicit in pattern matching at higher code levels
- **2.3.1.9. Bond tokens can be stolen after datum tampering:** Attacker uses datum tampering from 2.3.1.8 to perform complex multi-step attack stealing bond tokens. Consequence of 2.3.1.8 (if datum validation is added, this issue is prevented)
- **2.3.1.10. Bond tokens can be redeemed with different pool tokens:** Attacker combines stolen pool tokens (2.3.1.4) with datum tampering (2.3.1.8) to redeem bond tokens using wrong pool tokens. Consequence of 2.3.1.4 and 2.3.1.8 (if both issues are fixed, this attack is prevented)
- **2.3.2.1. uniqNFT can be redirected when cancelling a bond:** BondWriter validator doesn't verify uniqNFT is burned during bond cancellation, allowing redirection to arbitrary address. Requires semantic understanding of protocol lifecycle and which tokens should be burned in which operations
- **2.3.2.2. uniqNFT is sometimes required as fees:** Fee calculation includes uniqNFT in value being divided, requiring a unique token to be sent to multiple destinations when fee is 100%. Requires semantic understanding to identify token as NFT (not syntactically distinguishable from fungible tokens) and that this NFT must be present in continuing output, creating impossible constraint
- **2.3.3.1. Buffer boundary is open on the right:** Validator checks buffer ∈ [0, duration) instead of [0, duration], unclear why boundary case is excluded. Design decision requiring clarification, not a detectable code pattern
- **2.3.3.2. Initial margin is unchecked:** Validator enforces margin constraints when adding to existing bonds but not during bond creation, allowing arbitrary values including non-ADA tokens as initial margin. Requires semantic understanding that arbitrary initial margin values are unintended and should be restricted like margin additions
- **2.3.3.3. Bond tokens and pool tokens are handled differently:** Conceptually similar tokens are handled with different burning strategies (all-at-once vs incremental), creating inconsistency. Design pattern concern about consistency and maintainability

## Private Audit #1

**Description**
[No description provided]

**Summary:** [No summary provided]

**Findings**

**Relevant**

- **ID-01. Incomplete Currency Symbol Validation:** Function validates token name matches but doesn't properly check currency symbol, allowing tokens with same name but different policies to pollute UTxOs. Detectable pattern: incomplete validation of token tuple components (cs, tn, n) [INCOMPLETE-TOKEN-VALIDATION]
- **ID-02. Unvalidated Reference Script Field:** Protocol outputs don't validate or check reference script field, allowing arbitrary reference scripts to be attached and increase future transaction fees. Detectable pattern: output validation logic that ignores reference script field entirely [UNVALIDATED-REFERENCE-SCRIPT]
