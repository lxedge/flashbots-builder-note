# flashbots builder 学习笔记

根据 [flashbots/builder](https://github.com/flashbots/builder) 仓库展开学习

## 背景

### PBS - Proposer Builder Separation

提议者和构建者分离策略

由于 ETH2.0 升级，带来了新的审查和舞弊的可能。

### MEV - Maximal Extractable Value

最大可提取价值

## local devnet

参考 https://github.com/avalonche/eth-pos-devnet 仓库

### 123

### 解决的问题，`create-beacon-chain-genesis` 容器总是失败，无法生成`genesis.ssz` 文件

**出现错误**
msg="Could not generate beacon chain genesis state" error="could not set config params: config name=interop conflicts with existing config named=mainnet: configset cannot add config with conflicting fork version schedule"

通过排查 DockerFile docker-compose.yml config.yml 文件

- compose 中，`beacon-chain` 服务的 `interop-genesis-state` 参数应为 `genesis-state`。（根据 prysm 文档）
- 信标链的配置，缺少 `CAPELLA_FORK` 参数。[shanghai-and-capella-forks](https://beincrypto.com/eth-core-devs-roll-out-shanghai-and-capella-forks/)，核实最后提交时间确定问题

### 尚未完全解决的问题

- flashbots-pos-geth-1: could not fetch validators map, Syncing to latest head, not ready to respond
- flashbots-pos-beacon-chain-1: wanted chain ID 32382, got 1

<!-- ### Goerli -->
