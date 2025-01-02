# Randcast DevOps IAC

## Overview

This folder contains dockerfiles, cdk scripts, and config files for facilitating streamlined randcast deployments (both for testing and production use on mainnet).

## Folder Structure

Scripts are split into the following sections:

[localnet-test](./deployments/localnet-test/README.md): deploy testnet locally using docker network. This is the easiest way to get started with randcast and is recommended for testing purposes.

- The dockerfiles provided in this folder must be built locally and are not hosted on dockerhub.

[internet-test](./deployments/internet-test/README.md): deploy testnet online using AWS EC2 + docker. This gives a more practical example of how nodes are deployed on mainnet.

- The dockerfiles provided in this folder have been built and hosted on dockerhub and can be pulled directly from there.
- EC2 CDK scripts provided for deploying a full fledged test network (Anvil EC2 hosting anvil container + NODE EC2 for running contract-init and  node containers)

[mainnet](./deployments/mainnet/README.md): Sample deployment CDK script for randcast on mainnet.

- Uses same node docker image as internet-test.
- CDK script creates EC2 instance for hosting node containers only.

## Docker Images

[ARPA Dockerhub](https://hub.docker.com/u/arpachainio)

[arpachainio/anvil-test:latest](https://hub.docker.com/r/arpachainio/anvil-test/tags): Spin up an anvil test chain

[arpachainio/contracts-test:latest](https://hub.docker.com/r/arpachainio/contracts-test/tags): Deploy randcast contracts to anvil test chain

[arpachainio/node:latest](https://hub.docker.com/r/arpachainio/node/tags): Deploy arpa node that interfaces with the deployed contracts to generate randomness

## Manual Build Instructions

```bash
# manually build images
cd BLS-TSS-Network
docker build -t anvil-chain ./docker/internet-test/anvil-chain
docker build -t contract-init -f ./docker/internet-test/contract-init/Dockerfile .
docker build -t arpa-node ./docker/internet-test/arpa-node

# tag images for docker hub
docker tag anvil-chain arpachainio/anvil-test:latest
docker tag contract-init arpachainio/contracts-test:latest
docker tag arpa-node arpachainio/node:latest

# push images to docker hub
docker push arpachainio/anvil-test:latest
docker push arpachainio/contracts-test:latest
docker push arpachainio/node:latest

# pull images
docker pull arpachainio/anvil-test:latest
docker pull arpachainio/contracts-test:latest
docker pull arpachainio/node:latest
```
