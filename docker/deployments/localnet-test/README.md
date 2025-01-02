# Local Testnet using Docker Network (no ports exposed externally)

This folder contains everything needed to deploy a local testnet using docker network.
The three folders in this directory each contain all files needed to build the corresponding docker image.

- anvil-chain: start a local anvil chain
- contract-init: deploy contracts to local anvil chain
- arpa-node: stand up an arpa node that interfaces with the deployed contracts to generate randomness

NOTE: These built container images differ from the ones provided in [internet-test](../internet-test/README.md) and [mainnet](../mainnet/README.md). Do not try to use these images for internet-test or mainnet deployments. They will strictly not work and are only suitable for local testing using docker network.

## Useful Commands

The node-test ec2 comes with a dockerized version of foundry. You can run foundry commands using it like this:
[docs](https://book.getfoundry.sh/tutorials/foundry-docker)

```bash
# run cast using foundry docker image (included in node-test ec2)
docker run foundry "cast block --rpc-url $RPC_URL latest"

# Cleanup Docker images
docker kill $(docker ps -q)
docker rm $(docker ps -a -q)
```

## Sample Workflow

[docker networking](https://docs.docker.com/network/)

```bash
# create network
docker network create randcast_network 

# build images
cd BLS-TSS-Network
docker build -t anvil-chain ./docker/localnet-test/anvil-chain
docker build -t contract-init -f ./docker/localnet-test/contract-init/Dockerfile .
docker build -t arpa-node ./docker/localnet-test/arpa-node


# Start anvil chain
docker run -d --network randcast_network --name anvil-chain anvil-chain:latest

# Run contract init (ensure .env configured correctly)
docker run -d --network randcast_network --name contract-init -v ./contracts/.env.example:/usr/src/app/external/.env contract-init:latest 
# Wait for all contracts to deploy (check docker logs)

# Run 3 arpa nodes (ensure config files are correct)
docker run -d --network randcast_network --name node1 -v ./docker/localnet-test/arpa-node/config_1.yml:/usr/src/app/external/config.yml arpa-node:latest 
docker run -d --network randcast_network --name node2 -v ./docker/localnet-test/arpa-node/config_2.yml:/usr/src/app/external/config.yml arpa-node:latest 
docker run -d --network randcast_network --name node3 -v ./docker/localnet-test/arpa-node/config_3.yml:/usr/src/app/external/config.yml arpa-node:latest 

# check if nodes grouped successfully 
# (exec into node1 container)
watch 'cat /usr/src/app/log/1/node.log | grep "available"'
  # "Group index:0 epoch:1 is available, committers saved."

# deploy user contract
# (exec into contract-init container)
forge script /usr/src/app/script/GetRandomNumberLocalTest.s.sol:GetRandomNumberLocalTestScript --fork-url http://anvil-chain:8545 --broadcast

# check the randomness result recorded by the adapter and the user contract respectively
export ETH_RPC_URL=http://anvil-chain:8545

cast call 0xa513e6e4b8f2a923d98304ec87f64353c4d5c853 "getLastRandomness()(uint256)"
cast call 0x712516e61C8B383dF4A63CFe83d7701Bce54B03e "lastRandomnessResult()(uint256)"

# the above two outputs of uint256 type should be identical
```