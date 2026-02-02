# Plinth Static Analyzer
The Plinth Static Analyzer is a static analysis tool for Cardano smart contracts written in Haskell using the Plinth library, based on the [Stan](https://github.com/kowainik/stan?tab=readme-ov-file#what-this-tool-is-about) analyzer.

This repository collects and analyzes smart contract audit reports from the Cardano ecosystem (prioritizing Plinth, but also including Aiken and Plutarch) to identify syntactically detectable anti-patterns in on-chain code. From these findings, we aim to design and implement new analysis rules spanning security and performance.

Each rule not only detects issues but also provides descriptions and potential solutions to help developers write safer, more efficient smart contracts.