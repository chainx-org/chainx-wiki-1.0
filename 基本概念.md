### 发行模式

- ChainX 采用类似比特币的发行模式, 没有预挖，没有任何形式的公私募，没有 IEO, 所有发行的 PCX 均由 PoS 挖矿而来。

- 每 150 个块为一个分红周期，其中每个块 2 s，即每个分红周期大约需要 5 分钟。

- 初始分红周期发行 50 PCX，每隔 21 万个分红周期发行量减半， 最终 PCX 总量维持在 2100 万左右。

- ChainX 采用 PoS 共识，创世块中共发行 50 个PCX，平均分配给几个创世节点，作为系统初始抵押。

### 节点账户公钥和出块公钥

对于验证者节点，为了保护验证者私钥的安全性，ChainX需要验证者提供：

- 验证者公钥（validator_key）
- 出块地址公钥（session_key）

两个概念，验证者公钥与出块地址公钥为绑定关系，通过钱包注册绑定关系，其中我们建议：

* 验证者公钥（validator_key）由钱包生成
* 出块地址公钥（session_key）在启动节点时生成

ChainX强烈建议验证者公钥与出块地址公钥不相同，并建议验证者保护好自己的验证者公钥（validator_key）。因为ChainX的验证者公钥（validator_key）与出块地址公钥（session_key）分离，因此即使出块的机器被黑，暴露了出块地址公钥（session_key）也不会对验证者的资产产生影响。

### 相关链接

- ChainX 官网：https://chainx.org
- ChainX 钱包：https://wallet.chainx.org
- 比特币钱包: https://bitx.chainx.org
- 节点浏览器: https://stats.chainx.org
- 区块链浏览器: https://scan.chainx.org
- Telegram: https://t.me/chainx_org
- Twitter: https://twitter.com/chainx_org
- 加入测试网：https://github.com/chainx-org/ChainX/wiki/Testnet
