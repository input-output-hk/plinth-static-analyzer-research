In this file, we list on-chain code audits conducted within the Cardano ecosystem.
For each audit report, we include the following information:

- Project name
- Audit provider
- Audited party
- Language used (Plinth, Aiken or Plutarch)
- Link to the full audit report (where available). If not public, “private” will be written.
- Link to the source code. If not public, “private” will also be used.
- Number of findings, including a breakdown by category (according to the categorization used by the audit provider)
- Audit date

Any other information that is not available will be marked as “not available.”

| Audited project       | Audit provider | Audited party   | Language | Code to Report                                                                                             | Code                                                                                                                                                    | Findings                                                         | Date       |
| --------------------- | -------------- | --------------- | -------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------- |
| Treasury Contracts    | MLabs          | SundaeSwap      | Aiken    | [Report](https://github.com/SundaeSwap-finance/treasury-contracts/blob/main/audits/mlabs.pdf)              | [Code](https://github.com/SundaeSwap-finance/treasury-contracts/)                                                                                       | 1 Critical<br>7 High<br>6 Medium<br>1 Low<br>11&nbsp;Suggestions | 07/06/2025 |
| MuesliSwap            | MLabs          | MuesliSwap Team | Plinth   | [Report](https://github.com/mlabs-haskell/muesliswap-audit-public/blob/master/MuesliSwap-audit-report.pdf) | [Code](https://github.com/MuesliSwapTeam/muesliswap-cardano-contracts/blob/main/order-validator/src/Cardano/MuesliSwapOrderValidator/OrderValidator.hs) | 2 Catastrophic<br>1 Critical<br>1 Non-critical<br>4 Code quality | 22/02/2022 |
| BookToken             | Tweag          | Book.io         | Aiken    | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/Booktoken-2023-06.pdf)                     |                                                                                                                                                         | 1 Critical<br>0 High<br>3 Medium<br>2 Low<br>4 Lowest            | 28/06/2023 |
| MinSwap               | Tweag          | MinSwap         | Aiken    | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/Minswap-2022-01.pdf)                       | [Code](https://github.com/minswap/minswap-dex-v2)                                                                                                       | 3 Critical<br>2 High<br>5 Medium<br>3 Low<br>0 Lowest            | ??/01/2022 |
| Cerra AMM             | Tweag          | Cerra           | Plinth   | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/CerraAMM-2024-03.pdf)                      |                                                                                                                                                         | 0 Critical<br>1 High<br>1 Medium<br>6 Low<br>0 Lowest            | 08/03/2024 |
| Djed                  | Tweag          | IOG             | Plinth   | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/Djed-2023-01.pdf)                          |                                                                                                                                                         | 0 Critical<br>4 High<br>7 Medium<br>5 Low<br>4 Lowest            | 29/11/2022 |
| Optim Bonds and Pools | Tweag          | Optim           | Plinth   | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/Optim-2022-09.pdf)                         |                                                                                                                                                         | 0 Critical<br>4 High<br>7 Medium<br>5 Low<br>4 Lowest            | 02/09/2022 |
| P2P Lending           | Tweag          | Cerra           | Plinth   | [Report](https://github.com/tweag/tweag-audit-reports/blob/main/Cerra-2024-02.pdf)                         | [Code](https://github.com/cerraio/lending-plutus/tree/48605787f534d5acb336d85dfb4115323d90ddb9)                                                         | 1 Critical<br>2 High<br>7 Medium<br>5 Low<br>1 Lowest            | 13/02/2024 |
