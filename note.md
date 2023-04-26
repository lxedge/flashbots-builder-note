# flashbots builder 学习笔记

根据 [flashbots/builder](https://github.com/flashbots/builder) 仓库展开学习

## 背景

### MEV - Maximal Extractable Value

用户交易提交成功后，进入 mempool，mempool 是由像你我这样的普通用户提交的待处理交易的数据库，MEV 游戏就是从这里开始的。

一旦交易进入内存池，搜索者就会开始扫描黑暗森林以寻找有利可图的 MEV 机会。搜索者通常是运行套利机器人的大型机构和自营交易平台，但有时也包括个人。他们支付高昂的 Gas 费来让验证者接受他们的交易订单，而不是通过公共池。

虽然搜索者本身并不是中心化风险，但他们通过与区块构造者勾结，在实现巨大的中心化部分发挥了作用。它们在协议层中的作用是打包交易并将它们传递给黑暗森林下一个梯级的区块构造者。搜索者和矿工开始在地下市场勾结，以最有效地选择交易以最大化他们自己的 MEV 利润。

### PBS - Proposer Builder Separation

提议者和构建者分离策略

由于 ETH2.0 升级，带来了新的审查和舞弊的可能。

## Local Devnet

参考: https://github.com/avalonche/eth-pos-devnet

### 解决的问题

`create-beacon-chain-genesis` 无法生成 `genesis.ssz` 文件

**出现错误**
msg="Could not generate beacon chain genesis state" error="could not set config params: config name=interop conflicts with existing config named=mainnet: configset cannot add config with conflicting fork version schedule"

通过排查 DockerFile docker-compose.yml config.yml 文件

- compose 中，`beacon-chain` 服务的 `interop-genesis-state` 参数应为 `genesis-state`。（根据 prysm 文档）
- 创建信标链创世节点的配置缺少 `CAPELLA_FORK` 参数。[shanghai-and-capella-forks](https://beincrypto.com/eth-core-devs-roll-out-shanghai-and-capella-forks/)，核实最后提交时间确定问题

### 尚未完全解决的问题

- flashbots-pos-geth-1: could not fetch validators map, Syncing to latest head, not ready to respond
- flashbots-pos-beacon-chain-1: wanted chain ID 32382, got 1

## Testnet

**参考**

- https://github.com/flashbots/mev-boost/wiki/Testing
- https://docs.prylabs.network/docs/install/install-with-script

#### Step 1 - 安装 prysm，生成 jwt 密钥

因为 http 在信标节点和执行节点的通信需要使用 jwt 加密，使用

```bash
./prysm.sh beacon-chain generate-auth-secret`
```

生成 jwt.hex 文件

#### Step 2 - 使用 geth 运行 execution node - Execution Layer

```bash
geth --goerli --http --http.api eth,net,engine,admin --authrpc.jwtsecret /path/to/jwt.hex
```

<img src="images/execution-node.png" />

#### Step 3 - 使用 prysm 运行 beacon node - Consensus Layer

```bash
# 使用该仓库的 genesis.ssz
git clone https://github.com/eth-clients/eth2-networks

# 运行信标节点，替换 jwt-secret，genesis-state 路径，替换收费地址
./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --prater --jwt-secret=path/to/jwt.hex --genesis-state=genesis.ssz --suggested-fee-recipient=0x01234567722E6b0000012BFEBf6177F1D2e9758D9
```

<img src="images/beacon-node.png" />

使用 [checking-status](https://docs.prylabs.network/docs/monitoring/checking-status) 查看节点状态

#### Step 4 - 使用 prysm 运行 validator node - Consensus Layer

验证者运行在信标链，需要质押 32ETH 成为验证者。所以需要生成 keys

```bash
# 下载 deposit
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.5.0/staking_deposit-cli-d7b5304-darwin-amd64.tar.gz
tar xvf staking_deposit-cli-d7b5304-darwin-amd64.tar.gz

# 生成 validator-keys
./deposit new-mnemonic --num_validators=1 --mnemonic_language=english --chain=prater

# 关联账户和节点
./prysm.sh validator accounts import --keys-dir=<YOUR_FOLDER_PATH> --prater
```

把 `deposit_data-*.json` 上传到 [goerli](https://goerli.launchpad.ethereum.org/en/overview)，并调用 metamask 授权转账。

```
# 运行验证节点
./prysm.sh validator --wallet-dir=<YOUR_FOLDER_PATH> --prater
```

<img src="images/validator-node.png" />

#### Step 5 - mev-boost

```bash
git clone https://github.com/flashbots/mev-boost
cd mev-boost
make build

# 运行
./mev-boost -goerli -relay-check -relays https://0xafa4c6985aa049fb79dd37010438cfebeb0f2bd42b115b89dd678dab0670c1de38da0c4e9138c9290a398ecd9a0b3110@builder-relay-goerli.flashbots.net
```

<img src="images/mev-boost.png" />

## TODO

- 使用 docker 运行 local devnet
