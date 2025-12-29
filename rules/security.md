# Incomplete token validation

## Description
Any validator or minting policy that relies on token based authorization must fully validate the token tuple (currency symbol, token name, quantity) and must restrict minting/burning and additional tokens under the same policy where relevant. This rule is intended to prevent authorization bypass, invariant breaks and protocol state corruption. The rule should detect situations where the on-chain logic has an incomplete validation of token tuple components (cs, tn, n), for example:
- Accepts any token under a policy
- Accepts any token under certain token name
- Accepts any quantity
- Does not restrict minting/burning to the intended policy

## Examples

### Valid case

### Invalid case

## Estimation
