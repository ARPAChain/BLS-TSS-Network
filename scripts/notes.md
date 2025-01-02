# Randcast OP Automation Script

## Working OP Repo.

```bash
git clone https://github.com/wrinkledeth/optimism 
# changes made in this repo can be found below

# reset to latest stable build
git reset --hard f30fdd3f2a25c87c57531f0a33dbf3d902a7ca57

# edit bedrock-devnet/devnet 
['docker-compose', 'build', '--progress', 'plain'] # replace this with
["docker", "compose", "build", "--progress", "plain"] # this

# Makefile
docker-compose # replace this with
docker compose # this

# dockerfiles
# (search for string $BUILDPLATFORM) and add this to top:
ARG BUILDPLATFORM=linux/amd64
```


## Start Commands

```bash
git submodule update --init
make devnet-up-deploy 
make devnet-clean
```

## Troubleshoot Batcher

```bash
# error
ubuntu@jammy:~/optimism$ docker logs f6804639456b
t=2023-08-11T19:46:55+0000 lvl=info msg=“Initializing Batch Submitter”
t=2023-08-11T19:46:55+0000 lvl=error msg=“Unable to create Batch Submitter” error=“querying rollup config: Post \“http://op-node:8545\“: dial tcp 172.18.0.4:8545: connect: connection refused”
t=2023-08-11T19:46:55+0000 lvl=crit msg=“Application failed”               message=“querying rollup config: Post \“http://op-node:8545\“: dial tcp 172.18.0.4:8545: connect: connection refused”

# just restart the container
docker start be5377ffaac6  ## OR

# in path: optimism/ops-bedrock
cd ops-bedrock && docker compose start op-batcher && cd ..
```

----

# Script automation Dev

## Ruoshan modifications

```bash
ubuntu@jammy:~/BLS-TSS-Network$ git status
On branch script_automation
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   contracts/.env.example
	modified:   contracts/README.md
	modified:   contracts/script/ControllerLocalTest.s.sol

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	contracts/script/OPControllerOracleInitializationLocalTest.s.sol
	contracts/script/OPControllerOracleLocalTest.s.sol
	contracts/src/ControllerOracle.sol
	contracts/src/ControllerRelayer.sol
	contracts/src/OPChainMessenger.sol
	contracts/src/interfaces/IChainMessenger.sol
	contracts/src/interfaces/IControllerOracle.sol
	contracts/src/interfaces/IOPCrossDomainMessenger.sol
	contracts/test/ControllerOracle.t.sol
	contracts/test/MockL2CrossDomainMessenger.sol
```

## Scripting Steps

1. Initialize devnet (L1 + L2)
2. Deploy L2 randcast contracts
3. 
4. Deploy L1 randcast contracts.
5. Deploy 3 randcast nodes with docker compose and wait for grouping.
6. (Test request randomness on L1)
7. (Test request randomness on L2)

```bash
# prep env file + dependencies
cd BLS-TSS_NETWORK/contracts
cp .env.example .env
forge test # download dependencies

# deploy L1 contracts
forge script script/ControllerLocalTest.s.sol:ControllerLocalTestScript --fork-url http://localhost:8545 --broadcast

forge script script/StakeNodeLocalTest.s.sol:StakeNodeLocalTestScript --fork-url http://localhost:8545 --broadcast -g 150

## deploy L2 contracts
forge script script/OPControllerOracleLocalTest.s.sol:OPControllerOracleLocalTestScript --fork-url http://localhost:9545 --broadcast

forge script script/OPControllerOracleInitializationLocalTest.s.sol:OPControllerOracleInitializationLocalTestScript --fork-url http://localhost:9545 --broadcast

# start randcast nodes
cd BLS-TSS_NETWORK/scripts
docker-compose up -d

# 

```

```bash
# L2: OP Controller Oracle Local Test (update the L2 contract addresses in .env)
forge script script/OPControllerOracleLocalTest.s.sol:OPControllerOracleLocalTestScript --fork-url http://localhost:9545 --broadcast

# L1 Controller Local Test (update the L1 contract addresses in .env)
forge script script/ControllerLocalTest.s.sol:ControllerLocalTestScript --fork-url http://localhost:8545 --broadcast

# OP Controller Oracle Initialization
forge script script/OPControllerOracleInitializationLocalTest.s.sol:OPControllerOracleInitializationLocalTestScript --fork-url http://localhost:9545 --broadcast

# Init staking local test
forge script script/InitStakingLocalTest.s.sol:InitStakingLocalTestScript --fork-url http://localhost:8545 --broadcast -g 150

# StakeNode Local test script
forge script script/StakeNodeLocalTest.s.sol:StakeNodeLocalTestScript --fork-url http://localhost:8545 --broadcast -g 150
```


## Dockercompose commands

```bash
docker-compose up -d
docker-compose down

# checks
docker exec -it node1 /bin/bash       
watch 'cat /var/log/randcast_node_client.log | grep "available"'

# cleanup
docker kill $(docker ps -q -f ancestor=arpachainio/node:latest) # kill node containers
docker rm -f $(docker ps -a -q -f ancestor=arpachainio/node:latest) # remove them

#combined
docker kill $(docker ps -q -f ancestor=arpachainio/node:latest); docker rm -f $(docker ps -a -q -f ancestor=arpachainio/node:latest)

# wtf?

{"time":"2023-08-15T01:57:52.952434970+00:00","message":"Calling contract view get_coordinator: 0xfc77afc6e3a15989647d64309884957332640638","module_path":"arpa_node_contract_client","file":"crates/arpa-node/src/node/contract_client/src/lib.rs","line":106,"level":"INFO","target":"arpa_node_contract_client","thread":"tokio-runtime-worker","thread_id":140235320608512,"node_id":"running","mdc":{},"node_info":"","group_info":""}
{"time":"2023-08-15T01:57:52.979325525+00:00","message":"Calling contract transaction post_process_dkg: 0xd8d9b3e3137d8a11e4c55c3ee192c780c724363164032fada4d8bcac6fa9862a","module_path":"arpa_node_contract_client","file":"crates/arpa-node/src/node/contract_client/src/lib.rs","line":49,"level":"INFO","target":"arpa_node_contract_client","thread":"tokio-runtime-worker","thread_id":140235320608512,"node_id":"running","mdc":{},"node_info":"","group_info":""}
{"time":"2023-08-15T01:57:55.987730071+00:00","message":"Transaction successful(post_process_dkg), receipt: TransactionReceipt { transaction_hash: 0xd8d9b3e3137d8a11e4c55c3ee192c780c724363164032fada4d8bcac6fa9862a, transaction_index: 0, block_hash: Some(0x9254195250ce0d7b4b5f5840f4ffa71f5d9ed29d114d905178048b5c60e2d012), block_number: Some(59557), from: 0xbcd4042de499d14e55001ccbb24a551f3b954096, to: Some(0x0ac85d55ebfc7f7b0cf4c13bb3bd6eaf3909d62d), cumulative_gas_used: 139807, gas_used: Some(139807), contract_address: None, logs: [Log { address: 0x0ac85d55ebfc7f7b0cf4c13bb3bd6eaf3909d62d, topics: [0x8353a804115421789f3ab2eeb3f5215943906ce12100c91d40fc865caf742b6f, 0x000000000000000000000000bcd4042de499d14e55001ccbb24a551f3b954096], data: Bytes(0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000064), block_hash: Some(0x9254195250ce0d7b4b5f5840f4ffa71f5d9ed29d114d905178048b5c60e2d012), block_number: Some(59557), transaction_hash: Some(0xd8d9b3e3137d8a11e4c55c3ee192c780c724363164032fada4d8bcac6fa9862a), transaction_index: Some(0), log_index: Some(0), transaction_log_index: None, log_type: None, removed: Some(false) }], status: Some(1), root: None, logs_bloom: 0x00000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000100000000000000000000000000000000000000000000000000400000000000000000000000000000000000001000000100000000000000000000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080000000000000000080000000000000000000000000000000000000, transaction_type: Some(2), effective_gas_price: Some(3000000007) }","module_path":"arpa_node_contract_client","file":"crates/arpa-node/src/node/contract_client/src/lib.rs","line":67,"level":"INFO","target":"arpa_node_contract_client","thread":"tokio-runtime-worker","thread_id":140235320608512,"node_id":"running","mdc":{},"node_info":"","group_info":""}
{"time":"2023-08-15T01:57:55.988808513+00:00","message":"-------------------------call post process successfully-------------------------","module_path":"arpa_node::node::subscriber::post_grouping","file":"crates/arpa-node/src/node/subscriber/post_grouping.rs","line":132,"level":"INFO","target":"arpa_node::node::subscriber::post_grouping","thread":"tokio-runtime-worker","thread_id":140235320608512,"node_id":"running","mdc":{},"node_info":"","group_info":""}
```



## Tutorial: Communication between contracts on OP Mainnet and Ethereum

[crosshain calls tutorial](https://github.com/ethereum-optimism/optimism-tutorial/tree/main/cross-dom-comm)

```bash
export GOERLI_URL=http://localhost:8545
export OP_GOERLI_URL=http://localhost:9545

export GREETER_L1=0x4d0fcc1Bedd933dA4121240C2955c3Ceb68AAE84
export GREETER_L2=0xE8B462EEF7Cbd4C855Ea4B65De65a5c5Bab650A9




# deploy l1 to l2 contract:
export FROM_L1_CONTROLLER=0x13cac39046b6f66d544476b8a4FF72f884c56833

# deploy l2 to l2 contract:
export FROM_L2_CONTROLLER=0x13cac39046b6f66d544476b8a4FF72f884c56833

# tx hash:
export HASH=0x504b801ec9af5ca50b8521592dfa7b2bcd97ff9490304512122f05f2a49c9bd7

