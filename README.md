<h1 align="center">Randcast BLS-TSS-Network</h1>

## Overview

The infrastructure of Randcast (DKG and Threshold-BLS based randomness service).

## Build Guide

Build with `cargo build --release`.

Test with `cargo test --all -- --test-threads=1 --nocapture`.

## Directory Structure

This repository contains several Rust crates and Solidity contracts that implement the different building blocks of Randcast. The high-level structure of the repository is as follows:

- [`arpa-node`](crates/arpa-node): Rust crate that provides a long-running client and a CLI tool for the node operator.
- [`user-cli`](crates/user-cli): Rust crate that provides a user side CLI tool to interact with the smart contracts.
- [`core`](crates/core): Rust crate that provides basic types, utilities and traits for the node and the CLI tool.
- [`contract-client`](crates/contract-client): Rust crate that provides types and traits for interacting with the smart contracts.
- [`dal`](crates/dal): Rust crate that provides the implementation of the data access layer.
- [`log`](crates/log): Rust crate that provides an attribute macro for logging at debug level.
- [`dkg-core`](crates/dkg-core): Rust crate that provides the implementation utilities for the DKG.
- [`threshold-bls`](crates/threshold-bls): Rust crate that provides the implementation of threshold BLS signatures for BN254 and BLS12-381.
- [`contracts`](contracts): Solidity contracts including Controller, Adapter, Coordinator, libraries and scripts for deployment and testing.

## Acknowledgements

Rust crates [`dkg-core`](crates/dkg-core), [`threshold-bls`](crates/threshold-bls) and Solidity contracts under [libraries/](contracts/src/libraries) have been adapted from the following resources:

- [celo-threshold-bls-rs](https://github.com/celo-org/celo-threshold-bls-rs)
- [bls_solidity_python](https://github.com/ChihChengLiang/bls_solidity_python)
