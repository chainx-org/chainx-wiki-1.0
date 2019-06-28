ChainX是BTC最大的侧链和二层网络，跨链中继所有BTC块头和交易证明，并通过链上轻节点模式来演算BTC的PoW算法得出最长链，保证交易和资产安全。

## 什么是X-BTC

用户通过向ChainX公共多签托管地址发起BTC充值交易，同时在OP_RETURN中附带自己的ChainX账户地址信息，ChainX中继会自动提交交易证明，并在获得ChainX轻节点安全确认后，OP_RETURN中所附带的ChainX账户将自动获得1:1的X-BTC。X-BTC可以参与充值挖矿、快速转账和DEX交易。

ChainX采用全额储备金的公开透明模式1:1发行X-BTC，赋予了BTC在ChainX网络内2s的快速转账能力和去中心化交易能力，并且将此最高信用的X-BTC输送给Polkadot网络。

## 什么是L-BTC

为了进一步扩大ChainX转接桥在BTC社区中的用户覆盖和社区共识。ChainX即将在1.0.3版本升级之后，支持BTC自锁仓跨链挖矿。

用户无需再把BTC充值进入多签托管地址，而是只需要生成一个自己掌握私钥的独立钱包地址（仅支持1和3开头的地址），并把自己的BTC转入这个独立地址“自锁仓”任意时间，同时在OP_RETURN中附带自己的ChainX账户地址信息，ChainX转接桥就会中继这类交易，为OP_RETURN中所附带的ChainX账户1:1自动发放Lock-up BTC（L-BTC），即可参与锁仓挖矿。

L-BTC同X-BTC一样采用0.1倍价格折扣参与跨链挖矿。L-BTC只能持有，不能转账，一旦Bitcoin链上锁仓的金额被花费掉，L-BTC将立即自动进行销毁，锁仓挖矿停止。

## X-BTC充值交易规则（须全部满足）
1. INPUT的地址必须是1或3开头的普通地址（p2pk|p2pkh|p2sh）
2. 其中一个OUTPUT的地址是ChainX的公共多签托管地址（通过RPC [chainx_particularAccounts接口](https://github.com/chainx-org/ChainX/wiki/RPC#chainx_particularaccounts)实时获取）
3. 其中一个OP_RETURN且格式为  用户`ChainX地址[@channel]` （channel为节点名，作为可选项，代表渠道方可以自动获得10%的挖矿收益）

## L-BTC自锁仓交易规则（须全部满足）
1. 含有value的OUTPUT不超过2个（即一个锁仓utxo，一个找零utxo），OP_RETURN不超过1个（即output为2-3个，其中一个output是OP_RETURN），INPUT不超过10个(为了限制矿池及交易所交易，筛选出普通用户交易)
2. 其中一个OUTPUT的地址是锁仓地址，必须是1或3开头的地址（p2pk|p2pkh|p2sh），即包含1开头的普通地址，3开头的脚本地址或隔离见证地址。OUTPUT的金额是锁仓金额，必须 >=0.1 BTC且 <= 10 BTC
3. ChainX链上会记录所有的锁仓地址，单地址最高上限不能超过10BTC，否则刚满足的锁仓交易会被拒绝
    1. 例：假设A向地址addr1已经锁定了9BTC，之后 **任意人** 又向 **相同的地址addr1** 发送了一笔2BTC的锁仓交易，这笔2BTC的锁仓会被拒绝
4. 所有OUTPUT不能出现充值地址，否则视为充值X-BTC处理
5. 有且只有一个OP_RETURN且格式为 `ChainX:用户ChainX地址[@channel]:用户BTC锁仓地址[0..4]` （channel为节点名，作为可选项，代表渠道方可以自动获得10%的挖矿收益，比特币锁仓地址取前4位，用于从utxo中匹配出用户锁仓的地址，若匹配不出将不认可这笔锁仓交易） 
    注：OP_RETURN最大字段为80bytes，因此内容长度最大为
    ```
    # ChainX:chainx_addr[@channel]:btc_addr[0..4]  # [@channel] 代表可选，[0..4]代表去前4个字节
    5(ChainX) + 1(:) + 48(ChainX_addr_len) + 1(@) + 12(channel最大长度12) + 1(:) + 4(比特币地址取前4位) = 72 Bytes
    ```

**注意：OP_RETURN的编码为 字符串=>Hex=>Bytes 写入BTC交易**


## 钱包接入
OP_RETURN是BTC交易携带跨链信息的重要通道，是BTC用户进入跨链世界的基础条件。希望业界有更多BTC钱包支持OP_RETURN功能。

若钱包希望能支持ChainX的BTC充值或自锁仓挖矿功能，可以定制用户转账接口，根据上文所列规则，在OP_RETURN中附带相应信息即可。（钱包可以将自身在ChainX的节点名设为默认渠道，可以自动获得用户充值或锁仓挖矿收益10%的渠道收益）