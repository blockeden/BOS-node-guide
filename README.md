# BOS 节点部署指南

[View in English](README_EN.md)

本文旨在给希望运行BOS节点的开发者提供部署指南和架构建议。

## 硬件配置

BP节点配置建议：

- 4GHz 4核 CPU
- 公网IP 50M带宽
- 32G 内存

4GHz的cpu配置是对BP的基本要求，后续将会有cpu性能检测的工具来监督BP的硬件配置。

除了BP出块节点外，fullnode全节点和api节点可以根据实际情况酌情决定。


## 目录结构

本项目作为配置参考，目录结构为：

```
.
├── config
│   ├── config.ini      (bos配置文件,需要修改)
│   └── genesis.json    (需要替换,每个链有一个相应的文件,测试链和主链不同)
├── data                (区块数据,不能手动修改)
├── docker-compose-init.yaml    (首次启动docker-compose配置文件,需要修改)
└── docker-compose.yaml         (正常启动docker-compose配置文件,需要修改)
```

这是一个BOS配置和数据文件基本的示例。

## 配置文件

config.ini文件是节点配置文件，下面是一个配置文件参考：

```
## CHANGE THESE
p2p-server-address = p2p公网地址
agent-name = "节点名"

## UNCOMMENT AND CHANGE THESE ONLY IF YOU WANT TO PRODUCE BLOCKS
signature-provider = 出块公钥=KEY:出块私钥 
producer-name = 节点账户名

## MAYBE CHANGE THESE
http-server-address = 0.0.0.0:8888
p2p-listen-endpoint = 0.0.0.0:9876

## USUALLY DONT CHANGE THESE
blocks-dir = "blocks"
abi-serializer-max-time-ms = 2000
wasm-runtime = wabt
chain-state-db-size-mb = 65535
reversible-blocks-db-size-mb = 1024
contracts-console = false
https-client-validate-peers = 1
p2p-max-nodes-per-host = 1
allowed-connection = any
max-clients = 25
network-version-match = 0 
sync-fetch-span = 500
http-validate-host = false
## eosio::producer_plugin
enable-stale-production = true
pause-on-startup = false
max-transaction-time = 180
max-irreversible-block-age = -1
keosd-provider-timeout = 5

## BLACKLIST
# actor-blacklist = 

plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin

## PEERS
p2p-peer-address = 120.197.130.116:9000
p2p-peer-address = 13.230.87.138:9866
p2p-peer-address = 120.197.130.117:9000
p2p-peer-address = 120.197.130.117:9001
p2p-peer-address = 120.197.130.117:9002
p2p-peer-address = 120.197.130.117:9003
p2p-peer-address = 45.79.145.205:9000
p2p-peer-address = 47.91.244.124:443
p2p-peer-address = 8.208.9.182:443
p2p-peer-address = 47.254.82.241:443
p2p-peer-address = 47.254.134.167:443
p2p-peer-address = 149.129.133.66:443

```

最上面几行是需要修改的配置，配置参考：

```
## CHANGE THESE
p2p-server-address = blockeden.com:9876
agent-name = "BlockEden"

## UNCOMMENT AND CHANGE THESE ONLY IF YOU WANT TO PRODUCE BLOCKS
signature-provider = EOS8JFv68pMiosAvLFRa4k2wjL3ppNVQNvZSTsN8XaxEHskZUsCiU=KEY:5K5bse7AX5YYDpGpf7RRtK4jbKB6mmfDiDpiauDe9g9ZLWyRcWx
producer-name = blockedencom
```

p2p-server-address 是你节点的公网p2p地址。

signature-provider是出块账户注册时使用的公私钥。

producer-name是出块账户的账户名

配置文件最后是其他p2p节点地址的配置，具体内容根据其他节点提供的地址配置。

其他配置项的具体作用，请参考 https://github.com/boscore


## docker部署方式

本教程用docker方式启动，后续也会有本地编译版本的指南。

### 前置组件

- docker
- docker-compose

### 拉取镜像

拉取 bos 节点程序镜像：

```
docker pull boscore/bos
```

### 修改 docker-compose 配置文件

首次启动docker-compose配置文件 : docker-compose-init.yaml

```
version: "2"

services:
  nodeosd:
    image: boscore/bos:latest
    command: /opt/eosio/bin/nodeos --data-dir=/data --genesis-json=/etc/nodeos/genesis.json --config-dir=/etc/nodeos --delete-all-blocks
    hostname: nodeosd
    ports:
      - 8890:8888
      - 9878:9876
    volumes:
      - 项目目录/config:/etc/nodeos
      - 项目目录/data:/data
```

后续正常启动docker-compose配置文件 : docker-compose.yaml

```
version: "2"

services:
  nodeosd:
    image: boscore/bos:latest
    command: /opt/eosio/bin/nodeos --data-dir=/data --config-dir=/etc/nodeos
    hostname: nodeosd
    ports:
      - 8890:8888
      - 9878:9876
    volumes:
      - 项目目录/config:/etc/nodeos
      - 项目目录/data:/data
```

两个配置文件的不同在于启动参数的不同，初次启动需要清空数据文件并且指定genesis.json。

需要将配置文件中的 **项目目录** 改为实际的文件目录。

配置文件中的ports需要更根据实际情况修改，冒号前面是服务器端口，后面是docker内部端口。

### 启动docker容器

首次启动，在项目目录执行 
```
docker-compose -f docker-compose-init.yaml up -d
```

后续正常启动，在项目目录执行
```
docker-compose -f docker-compose.yaml up -d
```

### 暂停/重启 同步

暂停:

```
docker-compose -f docker-compose.yaml down
```

重启:

```
docker-compose -f docker-compose.yaml down
docker-compose -f docker-compose.yaml up -d
```

### docker常用命令

查看容器状态
```
docker ps -a
```

查看docker日志
```
docker logs -f --tail=20 容器名称
```

