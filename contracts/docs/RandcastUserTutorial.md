# Arpa Randcast User Tutorial

- [Arpa Randcast User Tutorial](#arpa-randcast-user-tutorial)
  - [1. Export your Environment Variables](#1-export-your-environment-variables)
  - [2. Create Subscription](#2-create-subscription)
  - [3. Gather Subscription Details](#3-gather-subscription-details)
  - [4. Fund the Subscription](#4-fund-the-subscription)
  - [5. Deploy the Consumer Contract](#5-deploy-the-consumer-contract)
  - [6. Add Consumer to Adapter](#6-add-consumer-to-adapter)
  - [7. Request Randomness](#7-request-randomness)
  - [8. Check Last Randomness](#8-check-last-randomness)

The below instructions outline the steps needed for a user to start requesting randomness from ARPA Randcast.

## 1. Export your Environment Variables

You will need to export several environment variables to streamline the execution of the subsequent commands.

Mainnet Exports

```bash
export ADAPTER_CONTRACT=0xbd57b868bb3374faa88722d2ee7ba3023c744e05 # mainnet adapter contract
export RPC_URL= # Mainnet Alchemy / Infura RPC URL Here
export USER_PUBLIC_KEY= # Eth User You are using to deploy consumer / user contract
export USER_PRIVATE_KEY= # Corresponding Private Key
```

Local Testnet Exports (See [Internet-test notes](../docker/internet-test-notes/../../README.md))

```bash
export ADAPTER_CONTRACT=0xa513E6E4b8f2a923D98304ec87F64353C4D5C853
export RPC_URL=http://52.15.52.16:8545
export USER_PUBLIC_KEY=0x70997970C51812dc3A010C7d01b50e0d17dc79C8
export USER_PRIVATE_KEY=0x59C6995E998F97A5A0044966F0945389DC9E86DAE88C7A8412F4603B6B78690D
```

## 2. Create Subscription

In this step, a subscription is created on the ADAPTER_CONTRACT using the `createSubscription` method. This subscription ID will be used later. The `cast send` command broadcasts a transaction to the Ethereum network.

Reference: [cast send](https://book.getfoundry.sh/reference/cast/cast-send)

```bash
cast block-number --rpc-url $RPC_URL # This will be useful for step 3
cast send $ADAPTER_CONTRACT "createSubscription()(uint64)" --private-key $USER_PRIVATE_KEY --rpc-url $RPC_URL  # returns subid
```

## 3. Gather Subscription Details

The subscription id is then retrieved from the contract event logs to be used in subsequent steps.

You can provide the block number from step 2 in order to speed up the event search.

Reference: [cast logs](https://book.getfoundry.sh/reference/cast/cast-logs)

```bash
cast logs --from-block 174270 --to-block latest 'SubscriptionCreated(uint64 indexed subId, address indexed owner)' "" $USER_PUBLIC_KEY --address $ADAPTER_CONTRACT --rpc-url $RPC_URL

#   topics: [ . # Sample response topics
#   	0x464722b4166576d3dcbba877b999bc35cf911f4eaf434b7eba68fa113951d0bf # event sig
#   	0x0000000000000000000000000000000000000000000000000000000000000001 # subId
#   	0x00000000000000000000000070997970c51812dc3a010c7d01b50e0d17dc79c8 # user public key
#   ]
export SUB_ID= # export subid for your newly created subscription
```

## 4. Fund the Subscription

Next, fund your subscription with Ethereum. These funds will be used to pay for your subsequent randomness requests.

```bash
cast send $ADAPTER_CONTRACT "fundSubscription(uint64)" $SUB_ID --value 1ether --private-key $USER_PRIVATE_KEY --rpc-url $RPC_URL
```

## 5. Deploy the Consumer Contract

The user must now deploy a contract that will consume the randomness provided by randcast. "forge create" is used to compile and deploy the contract.

A sample is provided here: [GetRandomNumberLocalTest.sol](https://github.com/ARPA-Network/Randcast-User-Contract/tree/main/contracts/user/examples/GetRandomNumberExample.sol)

Reference: [forge create](https://book.getfoundry.sh/forge/deploying)

```bash
cd /usr/src/app/
forge create --rpc-url $RPC_URL --private-key $USER_PRIVATE_KEY src/user/examples/GetRandomNumberExample.sol:GetRandomNumberExample --constructor-args $ADAPTER_CONTRACT
```

## 6. Add Consumer to Adapter

In this step, the previously created consumer contract is added to the adapter contract and linked to the subscription you created previously, via the adapter's `addConsumer` method.

```bash
cast send $ADAPTER_CONTRACT "addConsumer(uint64, address)" $SUB_ID $USER_CONTRACT --private-key $USER_PRIVATE_KEY --rpc-url $RPC_URL
```

## 7. Request Randomness

We can now request the randomness via the user contract with the method `getRandomNumber`.

```bash
cast send $USER_CONTRACT "getRandomNumber()" --private-key $USER_PRIVATE_KEY --rpc-url $RPC_URL
```

## 8. Check Last Randomness

Finally, retrieve the last random number generated with the `getLastRandomness` method.

```bash
cast call $ADAPTER_CONTRACT "getLastRandomness()(uint256)" --rpc-url $RPC_URL
```

After completing the entire process, you should have a user contract that is capable of requesting randomness from the randcast adapter contract.
