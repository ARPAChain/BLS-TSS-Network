# Node Client Docker Notes

## Manual Build Instructions

```bash
# manually build images
cd BLS-TSS-Network
# cargo build --bin node-client --release   # i think the build is happening in docker now
docker build -t node-client ./docker/node-client

# tag images for docker hub
docker tag node-client arpachainio/node-client:latest

# push images to docker hub
docker push arpachainio/node-client:latest

# pull images
docker pull arpachainio/node-client:latest

## pull for ghcr 
docker pull ghcr.io/arpa-network/node-client:latest
```

## Run containers from dockerhub

```bash
docker run -d \
  --name node1 \
  -p 50061:50061 -p 50091:50091 \
  -v ./docker/node-client/config_1.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/1:/app/log/1 \
  ghcr.io/arpa-network/node-client:latest

docker run -d \
  --name node2 \
  -p 50062:50062 -p 50092:50092 \
  -v ./docker/node-client/config_2.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/2:/app/log/2 \
  ghcr.io/arpa-network/node-client:latest

docker run -d \
  --name node3 \
  -p 50063:50063 -p 50093:50093 \
  -v ./docker/node-client/config_3.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/3:/app/log/3 \
  ghcr.io/arpa-network/node-client:latest
```

## Useful Alias

```bash

# node client killing
alias nodekill='pkill -f "node-client -c"'
alias logkill='rm -rf /home/ubuntu/BLS-TSS-Network/crates/arpa-node/log; rm /home/ubuntu/BLS-TSS-Network/crates/arpa-node/*.sqlite'

# python env
alias venv="python3 -m venv .venv"
alias activate=". .venv/bin/activate"

# Docker
function _docker_exec() { docker exec -it $1 /bin/bash; }
alias de='_docker_exec'
alias dl='docker logs -f'
alias dv='docker volume ls'
alias di='docker images'
# alias dnuke='docker kill node1 node2 node3; docker rm node1 node2 node3; sudo rm -rf /home/ubuntu/BLS-TSS-Network/docker/node-client/log'
alias dnukelog='sudo rm -rf /home/ubuntu/BLS-TSS-Network/docker/node-client/log'
alias dnuke='docker kill node1 node2 node3; docker rm node1 node2 node3;'
alias dbuild='docker build -t node-client ./docker/node-client'
alias drun='\
docker run -d \
  --name node1 \
  -p 50061:50061 -p 50091:50091 \
  -v ./docker/node-client/config_1.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/1:/app/log/1 \
  ghcr.io/arpa-network/node-client:latest

docker run -d \
  --name node2 \
  -p 50062:50062 -p 50092:50092 \
  -v ./docker/node-client/config_2.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/2:/app/log/2 \
  ghcr.io/arpa-network/node-client:latest

docker run -d \
  --name node3 \
  -p 50063:50063 -p 50093:50093 \
  -v ./docker/node-client/config_3.yml:/app/config.yml \
  -v ./docker/node-client/db:/app/db \
  -v ./docker/node-client/log/3:/app/log/3 \
  ghcr.io/arpa-network/node-client:latest'
```
