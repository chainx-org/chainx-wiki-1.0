# ChainX change log

* [v1.1.0](#v110)
* [v1.0.6](#v106)
* [v1.0.5](#v105)
* [v1.0.4](#v104)
* [v1.0.3](#v103)
* [v1.0.2](#v102)
* [v1.0.1](#v101)
* [v1.0.0](#v100)

## v1.1.0

大版本升级

### Added

#### 1. 09号提案

见 [ChainX 09号提案：关于 ChainX 挖矿收益分配的调整](https://scan.chainx.org/referendum/finished)

#### 2. ChainX 智能合约

1. **在本次升级中会包含ChainX智能合约的所有功能，但是不会立即开启该功能。该功能将会等到Polkadot正式上线后开启。**
2. ChainX智能合约提供了相关的新RPC接口：详细见：[合约相关rpc](https://github.com/chainx-org/ChainX/wiki/RPC#%E5%90%88%E7%BA%A6%E9%83%A8%E5%88%86)
3. ChainX智能合约开发文档见:
    - https://github.com/chainx-org/ChainX/wiki/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6
    - https://github.com/chainx-org/ChainX/wiki/ChainX-ink

#### 3. rpc 的调用受到参数 `rpc-cors`的限制

```bash
--rpc-cors <ORIGINS>
    Specify browser Origins allowed to access the HTTP & WS RPC servers. It's a comma-separated list of origins
    (protocol://domain or special `null` value). Value of `all` will disable origin validation. Default is to
    allow localhost, https://polkadot.js.org and https://substrate-ui.parity.io origins. When running in --dev
    mode the default is to allow all origins.
```

在该版本的升级中引入了Substrate提供的rpc跨域的限制。若已有的节点提供RPC服务，**请务必对该参数进行设置**。若没有安全性要求，那么请在配置文件中添加`"rpc-cors":"all"` 即可。

## v1.0.6

### Enhancement

- 提升票龄类型可扩展性。票龄类型支持至 `u128`, 当票龄范围超过 `u64` 最大值时，会自动切换到 `u128`，  **此时 RPC 的相关接口需要切换到 V1 版本访问 ** ，详细信息见: [#90](https://github.com/chainx-org/ChainX/issues/90) 。
- 针对 `chainx_getIntentions` 接口新增 RPC 缓存功能，大幅提升 RPC 性能，增强节点可用性。
- RPC 新增接口等其他变动, 详情见: [#90](https://github.com/chainx-org/ChainX/issues/90) .

## v1.0.5

### Enhancement

- 01号提案: 加强节点自抵押限制
- 02号提案：限制用户频繁切换投票
- 06号提案：提高跨链挖矿用户的提息门槛
- 添加用户同时最多可撤票数为10的限制，出于系统安全考虑，即用户最多同时只能有10笔处于赎回锁定期的撤票，否则必须等待部分撤票期满后，才能继续赎回投票

### Added

- 新增 RPC `chainx_getNextRenominateByAccount` 获取用户下一次可切换投票的高度, 参数 `AccountId`，返回结果是 `BlockNumber`。如果返回结果为 null, 表示用户可以立即切换投票。否则如果当前高度小于等于返回结果，则用户此时不能在当前高度切换投票。

### Changed

- RPC `chainx_getPseduNominationRecords` 返回字段中新增 `nextClaim`, 类型是 `BlockNumber`，表示高度在 `nextClaim` 后才可以进行下一次跨链资产提息。如果 `nextClaim` 为 0，表示跨链资产提息无频率限制，此时可以提息。如果 `nextClaim` 大于0，表示如果当前高度小于等于 `nextClaim` 时，用户受到跨链资产提息的频率限制，此时无法提息。

## v1.0.4

### Fixed:

修复BTC 锁仓的bug。

假设 BTC addr1 已经有一些金额锁仓，然后在同一笔交易中把这个addr1再次锁定addr1，会导致addr1记录的锁仓金额出现一点问题。这个问题虽然不会影响资产变化，但是会影响之后再往这个地址锁仓时的最大值检查, 最终导致无法再向这个地址继续锁仓。

## v1.0.3

### Fixed

- 修复DEX在特殊case下可能导致成交时不符合时间优先规则的问题。

### Changed

- 跨链资产进入ChainX时，不再按照跨链资产确认期间的累计票龄所占比例发放奖励，而是直接从对应的跨链资产奖池给跨链用户发放0.001PCX。

### Added

- 支持BTC自锁仓挖矿, [详情见wiki](https://github.com/chainx-org/ChainX/wiki/如何接入BTC充值和自锁仓跨链挖矿)
- 支持跨链资产与原生资产的staking分红比例动态调整模型, [详情见wiki](https://github.com/chainx-org/ChainX/wiki/充值挖矿#跨链资产与参与staking的原生资产分红比例动态调整模型)
- 添加成为候选验证人的最小总投票数限制，目前最小总投票数阈值为10PCX。当验证人正常换届时，如果候选验证人的总得票数小于10PCX时，会将该候选人强制退选，取消验证人候选资格。
- 增强各public交易安全校验。
- 增强BTC各类安全校验。
- 添加设置websocket最大连接数命令行参数 `--ws-max-connections`。
- 添加资产属性扩展是否可转账，可销毁, 可挖矿等。
- 其他优化。

## v1.0.2

#### Fixed

- 恢复 finalize.
- dev 模式崩溃.
- 节点可以切换自投票从而使节点30天的抵押锁定期失效。目前节点自抵押只能撤回，不能切换。
- btc信托否决提现后，不释放申请，而是将申请还原回申请中.
- DEX由于精度转换造成订单完全成交后可能会有极小的金额被锁.
- 覆盖ETH签名(r,s,v)中v为0x1b(27)和0x1c(28)的情况.

#### Added

- log4rs support, 支持日志切分功能。目前默认日志文件位于当前目录下的 `log/chainx.log`.
  - 该部分通过提供`log-dir`参数可以进行设置日志文件路径，其他的关于日志的配置请使用`--help`查询
  - 若需要恢复日志直接打印控制台的形式，可使用参数`"default-log":true`的形式
- 调高相关操作手续费, 调整至节点注册(10PCX)，更新(1PCX)，切换投票手续费(0.1PCX).
- 双签惩罚. 如果节点出现双签行为，将会罚空奖池，强制退选，并从当前验证人集合中移除。
- 除了 `chainx_getBlockByNumber`, 其他所有 `chainx_*` RPC接口均支持在最后一个参数指定块哈希查询，如果不填默认是best_hash.

## v1.0.1

修复 bitcoin 转接桥中解析 OpReturn 失败导致无法出块的问题。

## v1.0.0

ChainX发布
