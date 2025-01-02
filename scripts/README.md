# ARPA Network L1/L2 Devnet Automation


## Deployment methods

The below flags can be set in the .env file, and will allow you to perform different kinds of deployments using the deployment script. 

```bash
LOCAL_TEST="true" # solidity script will fund wallets with test eth. If set to false, must manually fund all wallets.
ARPA_EXISTS="true" # Deploy using existing ARPA contracts. EXISTING_OP_ARPA_ADDRESS and EXISTING_L1_ARPA_ADDRESS need to be set in .env.
L2_ONLY="true" # Deploy L2 contracts only, useful when L1 contracts are already deployed. You will need to set all existing L1 contract addresses in the .env file.
```

## Useful Alias
```bash
# rm docker node logs
alias dnukelog='sudo rm -rf /home/ubuntu/BLS-TSS-Network/docker/node-client/log'

# kill docker node containers
alias dnuke='docker kill node1 node2 node3; docker rm node1 node2 node3;'


# venv stuff
alias venv="python3 -m venv .venv"
alias activate=". .venv/bin/activate"

# rm db, logs, and kill nodes
sudo rm -rf db; sudo rm -rf log; docker kill node1 node2 node3; docker rm node1 node2 node3;

```

## L2_ONLY Test Workflow

### L1 and L2 deployment to make sure L1 contracts are deployed

```bash
# Set .env flags
LOCAL_TEST="true"
ARPA_EXISTS="false"
L2_ONLY="false"

# 1. Kill nodes / clear artifacts and run deployment script
python3 main.py
```

### L2 Deployment Only

```bash
# Set .env flags
LOCAL_TEST="true"
ARPA_EXISTS="false"
L2_ONLY="true"

# 2. Copy all existing L1 addresses to .env

# 3. If this ControllerRelayer has not yet been deployed: deploy it manually and copy the address to the .env file.
# comment out all lines in main() except for deploy_controller_relayer()
# double check that EXISTING_L1_CONTROLLER_ADDRESS is set in .env
python3 main.py

# 4. Kill the nodes but preserve the artifacts, run the deployment script
python3 main.py
```

## ARPA_EXISTS Test Workflow

### (L1 and L2 deployment with existing ARPA addresses)
```bash
# Set .env flags
LOCAL_TEST="true"
ARPA_EXISTS="true"
L2_ONLY="false"

# 1. Set existing arpa addresses:
   EXISTING_OP_ARPA_ADDRESS
   EXISTING_L1_ARPA_ADDRESS

# 2. Kill nodes / clear artifacts and run deployment script
python3 main.py
```

### (L1 and L2 deployment with existing ARPA addresses)
```bash
# Set .env flags
LOCAL_TEST="true"
ARPA_EXISTS="true"
L2_ONLY="true"

# 3. Copy all existing L1 values to .env

# 4. If this ControllerRelayer has not yet been deployed: deploy it manually and copy the address to the .env file.
# comment out all lines in main() except for deploy_controller_relayer()
# double check that EXISTING_L1_CONTROLLER_ADDRESS is set in .env
python3 main.py

# 5. Kill the nodes, but preserve the artifacts, and run the deployment script. 
python3 main.py
```

---

## OP Devnet Notes

Start Optimism Devnet

```bash
git clone https://github.com/wrinkledeth/optimism
cd optimism
git submodule update --init --recursive
make devnet-up-deploy
```

Clean and redeploy OP devnet
```bash
cd optimism
make devnet-clean
make devnet-up-deploy
```


## Other Notes

Build node client 

```bash
cd crates/arpa-node
cargo build --bin node-client
``` 

Run the deployment script
```bash
# configure .env in contracts directory.
cd scripts
python3 -m venv .venv # create venv
source .venv/bin/activate # activate venv
pip3 install -r requirements.txt # install dependencies
python3 main.py # run script
```
