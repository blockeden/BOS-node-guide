# BOS 节点部署指南
- [BOS 节点部署指南](#bos-%E8%8A%82%E7%82%B9%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97)
  - [硬件配置](#%E7%A1%AC%E4%BB%B6%E9%85%8D%E7%BD%AE)
  - [目录结构](#%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84)
  - [配置文件](#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  - [docker方式部署节点](#docker%E6%96%B9%E5%BC%8F%E9%83%A8%E7%BD%B2%E8%8A%82%E7%82%B9)
    - [前置组件](#%E5%89%8D%E7%BD%AE%E7%BB%84%E4%BB%B6)
    - [拉取镜像](#%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F)
    - [修改 docker-compose 配置文件](#%E4%BF%AE%E6%94%B9-docker-compose-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
    - [启动docker容器](#%E5%90%AF%E5%8A%A8docker%E5%AE%B9%E5%99%A8)
    - [暂停/重启 同步](#%E6%9A%82%E5%81%9C%E9%87%8D%E5%90%AF-%E5%90%8C%E6%AD%A5)
    - [docker常用命令](#docker%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)
  - [docker方式使用客户端](#docker%E6%96%B9%E5%BC%8F%E4%BD%BF%E7%94%A8%E5%AE%A2%E6%88%B7%E7%AB%AF)
    - [启动 client](#%E5%90%AF%E5%8A%A8-client)
    - [创建本地钱包](#%E5%88%9B%E5%BB%BA%E6%9C%AC%E5%9C%B0%E9%92%B1%E5%8C%85)
    - [生成公私钥对](#%E7%94%9F%E6%88%90%E5%85%AC%E7%A7%81%E9%92%A5%E5%AF%B9)
    - [导入私钥到钱包](#%E5%AF%BC%E5%85%A5%E7%A7%81%E9%92%A5%E5%88%B0%E9%92%B1%E5%8C%85)
    - [注册账号](#%E6%B3%A8%E5%86%8C%E8%B4%A6%E5%8F%B7)
    - [注册成为候选节点](#%E6%B3%A8%E5%86%8C%E6%88%90%E4%B8%BA%E5%80%99%E9%80%89%E8%8A%82%E7%82%B9)
    - [其他操作](#%E5%85%B6%E4%BB%96%E6%93%8D%E4%BD%9C)
  - [多节点部署](#%E5%A4%9A%E8%8A%82%E7%82%B9%E9%83%A8%E7%BD%B2)
  
<!-- [View in English](README_EN.md) -->

本文旨在给希望运行BOS节点的开发者提供部署指南和架构建议。

如果你是EOS老手, 请直接查看:[BOS与EOS差异](BOS-EOS.md)

启动一个BOS节点大致的流程为:

- 修改配置文件
- 启动节点程序
- 注册BOS账号、注册成为节点
- 部署节点官网

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
abi-serializer-max-time-ms = 3000
wasm-runtime = wabt
chain-state-db-size-mb = 65535
reversible-blocks-db-size-mb = 1024
contracts-console = false
https-client-validate-peers = 1
p2p-max-nodes-per-host = 5
allowed-connection = any
max-clients = 25
network-version-match = 0 
sync-fetch-span = 500
connection-cleanup-period = 30
max-implicit-request = 1500
http-validate-host = false
p2p-discoverable = true
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


## docker方式部署节点

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


## docker方式使用客户端

部署完节点之后,需要向区块链发送交易来注册成为节点,这个时候需要使用客户端.

### 启动 client

client 可以部署在任何地方,可以是云端服务器,也可以是本地笔记本,前置条件是需要安装:

- docker
- docker-compose
  
参考本项目client文件目录下的内容, 在client目录下执行一下命令来启动客户端容器.

```
docker-compose up -d
```

在客户端容器启动的情况下执行以下命令进入容器内部:
```
docker exec -it client_keosd_1 bash
```
这里的**client_keosd_1**需要换成实际的docker容器名称.

进入容器后就可以使用cleos命令行工具和keosd钱包工具了.

**该小节所有后续都在docker容器内部执行.**

### 创建本地钱包

钱包只需要创建一次.

```
cleos wallet create --file /root/eosio-wallet/wallet-key
```
该命令将会创建一个本地钱包,钱包密码会存在容器内部的/root/eosio-wallet/wallet-key文件中.

### 生成公私钥对
```
cleos create key --to-console
```
该命令会生成一个公私钥对,类似:
```
Private key: 5KKZJiGGMcW2nEjqTrHfMioKFJP6conCw8c42kQGE33BsLs7dCz
Public key: EOS6mmGZDVtujn3qsXNUVVX7C1WjkLr3iw3bbNyoGB6Q78DGYeEuU
```

### 导入私钥到钱包
```
cleos wallet import
```
然后在输入要导入的私钥,钱包中需要有操作账户相应的私钥才能发起交易.

### 注册账号

最开始需要拥有账号的人为你注册账号,将账户名(12位,a-z或1-5)和公钥发给注册人.

如果已经拥有账号,可以使用命令创建账号
```
cleos system newaccount 
```

### 注册成为候选节点

执行命令
```
cleos -u api节点 system regproducer 账户名 出块公钥 官网地址 时区
```

参考:
```
cleos -u http://47.244.42.171:87 system regproducer blockedencom EOS8jdVGxpfdHpY94FZZbcn3pbTZ45DkD7CLkQFxi8kpuRiF1YDdu http://blockeden.com 1
```
时区字段为:-11～12, 允许小数.

### 其他操作

解锁钱包:
```
cleos wallet unlock < /root/eosio-wallet/wallet-key
```
命令中需要注意钱包密码的文件位置

转账:
```
cleos -u api节点 transfer from to quantity memo
```

## 多节点部署

BOS节点程序有多种不同类型:
- BP节点,负责出块,对安全性要求比较高.
- P2P节点,负责广播交易.
- API节点,负责接受客户端发起的交易.
  
因为节点分工不同,为了更好服务用户,需要多节点部署方案.

可以参考慢雾提供的安全部署指南:

https://github.com/slowmist/eos-bp-nodes-security-checklist

大致的思路是:
- 将BP节点程序放在局域网的内网,不对外暴露.
- 多台API节点提供负载均衡的API服务器.
- 多太P2P节点提供公网和VPN网络的数据传输.
