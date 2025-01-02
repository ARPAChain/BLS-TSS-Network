
# Mainnet Node Operation (CDK + Docker)

This is the instruction for node operators on mainnet.

We offer CDK scripts for deploying an AWS EC2 instance along wish all necessary resources and dependencies(VPC, Firewall rules, login method etc..) to host your ARPA node containers,

If you do not wish to use CDK, you can also use the dockerfiles and scripts in this folder to deploy your node containers on your infrastructure.

NOTE: It may be useful to test out the CDK scripts and instructions in [internet-test](../internet-test/README.md) and [localnet-test](../localnet-test/README.md) first to get a feel for the process.

## Install AWS CLI and AWS Session Manager plugin

[install aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

[install session manager plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

```bash
# Configure aws-cli
aws configure # configure aws cli (us-east-2, json, none, none)

# Verify session manager plugin install
session-manager-plugin
```

## Install CDK and deploy

[cdk getting started](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html)

[working with cdk python](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-python.html)

```bash
# Install CDK
npm install -g aws-cdk 
cdk --version # check version

# activate venv and install requirements.
cd ec2_cdk
python3 -m venv .venv 
source .venv/bin/activate
pip install -r requirements.txt

# Deploying
cdk bootstrap # initialize assets before deploy
cdk synth # emits the synthesized CloudFormation template

cdk deploy # deploy this stack to your default AWS account/region
```

Sample Stack Outputs after "cdk deploy"

```bash
Node ec2: 18.224.44.15
aws ssm start-session --target i-0da73558ad639280b
```

## Node EC2 Instructions

You will need to configure a config.yaml file for each node container. For details on how to configure this file, please see the [node-client readme](../../crates/arpa-node/README.md)

```bash

docker run -d --name node1 -p 50061:50061 -p 50091:50091 -v /Users/zen/dev/pr/BLS-TSS-Network/docker/mainnet/arpa-node/config_1.yml:/usr/src/app/external/config.yml arpa-node

# run arpa node containers
docker run -d --name node1 -p 50061:50061 -p 50091:50091 -v /tmp/BLS-TSS-Network/docker/mainnet/arpa-node/config_1.yml:/usr/src/app/external/config.yml arpachainio/node:latest
docker run -d --name node2 -p 50062:50061 -p 50092:50091 -v /tmp/BLS-TSS-Network/docker/mainnet/arpa-node/config_2.yml:/usr/src/app/external/config.yml arpachainio/node:latest
docker run -d --name node3 -p 50063:50061 -p 50093:50091 -v /tmp/BLS-TSS-Network/docker/mainnet/arpa-node/config_3.yml:/usr/src/app/external/config.yml arpachainio/node:latest

# check if nodes grouped successfully
docker exec -it node1 /bin/bash       
watch 'cat /var/log/randcast_node_client.log | grep "available"'

# deploy user contract / check if randomness worked.
docker exec -it contract-init /bin/bash
forge script /usr/src/app/script/GetRandomNumberLocalTest.s.sol:GetRandomNumberLocalTestScript --fork-url $ETH_RPC_URL --broadcast
cast call 0xa513e6e4b8f2a923d98304ec87f64353c4d5c853 "getLastRandomness()(uint256)" # should not show 0
cast call 0x712516e61C8B383dF4A63CFe83d7701Bce54B03e "lastRandomnessResult()(uint256)" # should match above
```

## Teardown

```bash
cdk destroy
```

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
