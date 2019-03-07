ChainX默认集成了第一期500万枚DOT投资者的名单，依据的以太坊合约地址为[0xb59f67a8bff5d8cd03f6ac17265c550ed8f33907](https://etherscan.io/token/0xb59f67a8bff5d8cd03f6ac17265c550ed8f33907)，

#### 启动配置

* 余额列表：3049个投资人的Ethereum地址和DOT余额。
* 跨链备注的载体是：交易中的InputData字段。

#### 交易接口：提交交易 birdge_sdot/push_tx

参数：Ethereum地址、交易原文、签名
判断：任何人都可以提交，签名需正确，交易原文的InputData格式满足要求。

如果该Ethereum用户还未领取，则发放对应余额列表中的SDOT给该用户，如果存在节点名称，则记录该节点为用户的充值渠道。

## SDOT中继系统

理论上用户可以使用自己持有DOT的Ethereum账户向任意地址发起一笔任意金额的交易，只要InputData格式满足要求，即可以成为一笔合法的ChainX映射交易，但安全起见，推荐用户往自己的地址发送一起0金额的自转账交易，并填写InputData，之后将交易ID记录下，手工前往钱包的中继程序界面，申请中继转发。

中继程序也可以自动监听所有投资者的交易，主动提交合法的映射交易至ChainX。
