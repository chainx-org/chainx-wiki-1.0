无特别说明，则rpc调用的返回的均是 Promise。

### chainx.chain.getInfo()

获取链的基本信息

### 参数

无

### 例子

```js
chainx.chain.getInfo()
/**
{ name: 'ChainX V0.9.7', version: '0.9.7', era: 'ChainX' }
**/
```

### chainx.chain.getBlockPeriod()

获取出块的间隔

### 参数

无

### 例子

```js
chainx.chain.getBlockPeriod()
/**
2 // 2 秒一个块
**/
```

### chainx.chain.subscribeNewHead()

订阅最新的块，返回一个 rxjs 标准的 `observable`

### 参数

无

### 例子

```js
const subscription = chainx.chain.subscribeNewHead().subscribe(result => {
  console.log('当前块高：', result.number)
  subscription.unsubscribe() // 取消订阅
})
/**
{
  "number": 381887, // 当前块高
  "hash": "0x513fbb976d775d7cda6d1b4b659160a5fd8ae9aed0c3f296123a067948837cd7", // 当前块哈希
  "now": 1554363156 // 当前出块时间
}
**/
```

### chainx.trustee.getTrusteeSessionInfo([chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js))

分局链返回当前的信托相关信息

### 参数

- chain: Chain, 链ID

### 例子

```js
chainx.trustee.verifyAddressValidity("BTC", "2N8tR484JD32i1DY2FnRPLwBVaNuXSfzoAv", "")
/**
{
  "coldEntity": "2N24ytjE3MtkMpYWo8LrTfnkbpyaJGyQbCA", // 冷信托地址
  "hotEntity": "2N1CPZyyoKj1wFz2Fy4gEHpSCVxx44GtyoY", // 热信托地址
  "sessionNumber": 0, // 当前信托届数，每换届一次就会增加1
  "trusteeList": [
    // 当前这条链的信托节点账户列表
    "0x471af9e69d41ee06426940fd302454662742405cb9dcc5bc68ceb7bec979e5e4",
    "0x806a491666670aa087e04770c025d64b2ecebfd91a74efdc4f4329642de32365",
    "0x1cf70f57bf2a2036661819501164458bd6d94642d81b5e8f1d9bdad93bad49bb",
    "0x09a6acd8a6f4394c6ba8b5ea93ae0d473880823f357dd3fdfd5ff4ccf1fcad99"
  ]
}
**/
```

### chainx.trustee.getTrusteeInfoByAccount([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js))

获取该用户的信托信息（已注册信托后会显示）

### 参数

- `who`: AccountId 账户

### 例子

```js
chainx.trustee.getTrusteeInfoByAccount('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ')
/**
[
  {
    "chain": "Bitcoin", // 链 ID
    "coldEntity": "02a79800dfed17ad4c78c52797aa3449925692bc8c83de469421080f42d27790ee", // 冷公钥/地址
    "hotEntity": "03f72c448a0e59f48d4adef86cba7b278214cece8e56ef32ba1d179e0a8129bdba" // 热公钥/地址
  }
]
**/
```

### chainx.trustee.getWithdrawTx([chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js))

获取对应链的提现交易，已经与这个交易相关的ChainX申请提现的用户提现记录id列表（用于信托获取对应链上正在处理中的多签提现交易，如对于BTC而言，该接口返回信托需要进行签署的比特币提现待签原文以及相关信息）

### 参数

- chain: Chain, 链ID

### 例子

```js
chainx.trustee.getWithdrawTx('Bitcoin')
/**
{
  "signStatus": false, //签名状态： false 未签名完成， true 签名完成
  "tx": "0100000001283fe241ec9528a48e6ce79b1ede9aabb59dbe38edeee013a28744c31d3db7860000000000ffffffff0288130000000000001976a914a5155d5636db0a9b8314460812f5105d84a5ae3d88acf0d200000000000017a9145737c1979343920ceea40e7c7d68b264b0effa3e8700000000", //比特币提现的交易原文
  // 这笔交易对应的ChainX用户提现记录的id列表，与`getWithdrawalList`的返回值结合后可以获取用户提现申请的详细记录，用于签署者去验证是否与原文相对应
  "withdrawalIdList": [
  	1,2,3 ... 
  ],
  // 当前已做出的行动信托列表（包含签名确认或者否决），未做出行动的不在列表内，信托详细信息可以从`getTrusteeSessionInfo`获取
  “trusteeList”: [ 
  	["0x..........", true], // 已签名
  	["0x..........", false], // 已否决
  	["0x..........", true], // 已签名
  ]
}
**/
```

### chainx.asset.getAssetsByAccount([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js), [page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

用户资产信息

### 参数

- `who`: AccountId 账户
- `page_index`: u32 页码 (从0开始)
- `page_size`: u32 页大小 (<=100)

### 例子

```js
chainx.asset.getAssetsByAccount('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 0, 10)
/**
{
  "data": [
    // 资产列表， 内容同上
    {
      "details": {
        // key 是资产类型的枚举
        "Free": 1000000000,
        "ReservedDexFuture": 0,
        "ReservedDexSpot": 0,
        "ReservedStaking": 0,
        "ReservedStakingRevocation": 0,
        "ReservedWithdrawal": 0
      },
      "isNative": true,
      "name": "PCX"
    },
    {
      "details": {
        "Free": 1000000000,
        "ReservedDexFuture": 0,
        "ReservedDexSpot": 0,
        "ReservedStaking": 0,
        "ReservedStakingRevocation": 0,
        "ReservedWithdrawal": 0
      },
      "isNative": false,
      "name": "BTC"
    }
  ],
  "pageIndex": 0, //当前页码
  "pageSize": 10, //页大小
  "pageTotal": 1 //总共页数
}
**/
```

### chainx.asset.getAssets([page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

总资产信息

### 参数

- `page_index`：u32 页码 (从0开始)
- `page_size`：u32 页大小 (<=100)

### 例子

```js
chainx.asset.getAssets(0, 10)
/**
{
  "data": [
    // 资产列表
    {
      "chain": "ChainX",
      "desc": "PCX onchain token",
      "details": {
        // key 是 资产类别的 枚举
        "Free": 2000000000, // 总活动资产数目
        "ReservedDexFuture": 0, // 总期货交易锁定资产数目
        "ReservedDexSpot": 0, // 总交易锁定资产数目
        "ReservedStaking": 0, // 总投票锁定资产数目
        "ReservedStakingRevocation": 0, // 总投票赎回锁定资产数目
        "ReservedWithdrawal": 0 // 总提现锁定数目
      },
      "isNative": true, // 是否是 native 资产
      "name": "PCX", // token id, token 标示
      "precision": 3,
      "trusteeAddr": ""
    },
    {
      "chain": "Bitcoin",
      "desc": "BTC chainx",
      "details": {
        "Free": 2000000000,
        "ReservedDexFuture": 0,
        "ReservedDexSpot": 0,
        "ReservedStaking": 0,
        "ReservedStakingRevocation": 0,
        "ReservedWithdrawal": 0
      },
      "isNative": false,
      "name": "BTC",
      "precision": 8,
      "trusteeAddr": "2MtAUgQmdobnz2mu8zRXGSTwUv9csWcNwLU" //接收跨链转账的信托地址
    }
  ],
  "pageIndex": 0, //当前页码
  "pageSize": 10, //页大小
  "pageTotal": 1 //总共页数
}

**/
```

### chainx.asset.getWithdrawalList([chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js), [page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

当前所有提现中记录

### 参数

- `chain`: String 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]
- `page_index`: u32 页码 (从0开始)
- `page_size`: u32 页大小 (<=100)

### 例子

```js
chainx.asset.getWithdrawalList('Bitcoin', 0, 10)
/**
{
  "data": [
    {
      "address": "mjKE11gjVN4JaC9U8qL6ZB5vuEBgmwik7b", // 提现地址
      "accountid": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", // 提现申请人
      "balance": 10, // 提现金额
      "memo": "", // 提现备注 或其他
      "id": 0, // 提现序列号
      "status": "Applying", // 当前提现状态，[枚举类型] 当前有3个值 "Applying" 申请中,"Signing" 签名中, "Broadcasting" 广播中, "Processing"处理中
      "time": 1547536602, // 申请提现的时间
      "token": "BTC", // 提现资产名
      "txid": "" //交易id
    },
    {
      "address": "mjKE11gjVN4JaC9U8qL6ZB5vuEBgmwik7b",
      "accountid": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f",
      "balance": 10,
      "memo": "",
      "id": 1,
      "status": "Applying",
      "time": 1547536652,
      "token": "BTC",
      "txid": ""
    }
  ],
  "pageIndex": 0, // 同上
  "pageSize": 100,
  "pageTotal": 1
}
**/
```

### chainx.asset.getWithdrawalListByAccount([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js), [page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

当前用户的提现中记录

### 参数

- `who`: AccountId 账户
- `page_index`: u32 页码 (从0开始)
- `page_size`: u32 页大小 (<=100)

### 例子

```js
chainx.asset.getWithdrawalListByAccount('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 0, 10)
/**
与 chainx.asset.getWithdrawalList 返回值相同
**/
```

### chainx.asset.getDepositList([chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js), [page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

充值中列表

### 参数

- `chain`: String 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]
- `page_index`: u32 页码 (从0开始)
- `page_size`: u32 页大小 (<=100)

### 例子

```js
chainx.asset.getDepositList('Bitcoin', 0, 10)
/**
{
  "data": [
    {
      "accountid": "", //chainx账户
      "address": "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7", //btc地址
      "balance": 1000, //充值金额
      "confirm": 1, //确认数
      "memo": "", //备注
      "time": 1547003949, //时间戳
      "token": "BTC", //币种
      "totalConfirm": 3, //总确认数
      "txid": "20bf6f637c0f05dbf2936cb0dbdd365ef1292e166aa7ed8af8d6c8418850d58e" //btc交易id
    },
    {
      "accountid": "",
      "address": "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7",
      "balance": 2000,
      "confirm": 2,
      "memo": "",
      "time": 1547003080,
      "token": "BTC",
      "totalConfirm": 3,
      "txid": "4f0c8ebd3650f6e369a4b400330ffb579c9d163cd0fcea0d065644acff571dd9"
    }
  ],
  "pageIndex": 0,
  "pageSize": 3,
  "pageTotal": 1
}
**/
```

### chainx.asset.getAddressByAccount([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js), [chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js))

ChainX账户绑定BTC地址列表

### 参数

- `who`: AccountId 账户
- `chain`: String 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]

### 例子

```js
chainx.asset.getAddressByAccount('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 'Bitcoin')
/**
[
  "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7"
]
**/
```

### chainx.asset.verifyAddressValidity([token](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Text.js), [addr](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Text.js), [memo](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Memo.js))

验证提现地址正确性

### 参数

- `token`: 币种名
- `addr`: 地址
- `memo`: 备注 （注：3个参数与提起提现请求时候交易`withdraw` 的参数列表完全一致）

### 例子

```js
chainx.asset.verifyAddressValidity("BTC", "2N8tR484JD32i1DY2FnRPLwBVaNuXSfzoAv", "")
/**
true // 返回 true 代表地址校验正确，false代表错误
**/
```

### chainx.asset.getWithdrawalLimitByToken([token](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Token.js))

chainx 链上与提现金额相关的限制，如提现的最小值与用户提现扣除的手续费

### 参数

- token: 币种名

### 例子

```js
chainx.asset.getWithdrawalLimitByToken("BTC")
/**
{
	minimalWithdrawal: 150000,
	fee: 100000
}
**/
```

### chainx.stake.getNominationRecords([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js))

用户投票信息

### 参数

- `who`: AccountId 账户

### 例子

```js
chainx.stake.getNominationRecords('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ')
/**
[
  [
    "0x9886b933e38461399f5997ef162b1ef90f8bcb19d3a2f256be0a8d2f85a80d98", // 投票节点
    {
      "lastVoteWeight": 26000, // 票龄
      "lastVoteWeightUpdate": 3461, // 票龄更新高度
      "nomination": 600, // 投票金额
      "revocations": [
        {
          "blockNumer": 117,
          "value": 1
        }
      ] // 撤票记录，117 为可赎回高度，1 为届时的可赎回金额
    }
  ]
]
**/
```

### chainx.stake.getIntentions()

节点列表

### 参数

无

### 例子

```js
chainx.stake.getIntentions()
/**
[
  {
    "about": "", // 关于
    "account": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", // 账户 ID
    "isActive": true, // 是否参选
    "isTrustee": true, // 是否是信托节点
    "isValidator": true, // 是否是结算节点
    "jackpot": 387096776, // 奖池金额
    "jackpotAddress": "0xce153e3235448f29ca9052a660e36abd9b9fdc72f80a4059a2427ff06b1a3706", // 奖池地址
    "lastTotalVoteWeight": 0, // 总票龄
    "lastTotalVoteWeightUpdate": 0, // 总票龄更新时间
    "name": "Alice", // 节点名
    "selfVote": 1250000000, // 节点自投票
    "sessionKey": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", // 节点出块地址， 类型为 AccountId
    "totalNomination": 1250000000, // 节点总投票
    "url": "Alice.com" // 节点网址
  }
]

**/
```

### chainx.stake.getPseduIntentions()

充值挖矿列表

### 参数

无

### 例子

```js
chainx.stake.getPseduIntentions()
/**
[
  {
    "circulation": 2000000000, // 总发行量
    "id": "BTC", // 资产ID
    "jackpot": 0, // 奖池金额
    "lastTotalDepositWeight": 0, // 总票龄
    "lastTotalDepositWeightUpdate": 0, // 总票龄更新高度
    "power": 10000000, // 单位资产对PCX算力
    "price": 1000000000 // 单位资产对PCX移动均价
  }
]
**/
```

### chainx.stake.getPseduNominationRecords([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js))

用户投票信息

### 参数

- `who`: AccountId 账户

### 例子

```js
chainx.stake.getPseduNominationRecords('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ')
/**
[
  {
    "balance": 1000000000, // 投票金额
    "id": "BTC", // 资产 ID
    "lastTotalDepositWeight": 0, // 票龄
    "lastTotalDepositWeightUpdate": 0 // 票龄更新高度
  }
]

**/
```

### chainx.stake.getBondingDuration()

获取普通用户撤销投票的锁定期，

### 参数

无

### 例子

```js
chainx.stake.getBondingDuration()
/**
300 //300 个块
**/
```

### chainx.stake.getIntentionBondingDuration()

获取节点用户撤销投票的锁定期

### 参数

无

### 例子

```js
chainx.stake.getIntentionBondingDuration()
/**
3000 //3000 个块
**/
```

### chainx.stake.getNextKeyFor([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js))

获取节点出块地址

### 参数

- `who`: AccountId 账户

### 例子

```js
chainx.stake.getNextKeyFor('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ')
/**
5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ
**/
```

### chainx.stake.getTokenDiscount()

获取虚拟挖矿的折扣

### 参数

无

### 例子

```js
chainx.stake.getNextKeyFor()
/**
10
**/
```

### chainx.trade.getOrderPairs()

交易对列表

### 参数

无

### 例子

```js
chainx.trade.getOrderPairs()
/**
[
  {
    "assets": "PCX", //交易对first
    "currency": "BTC", //交易对second
    "id": 0, //交易对的ID
    "precision": 5, //交易对价格精度 10.pow(precision)
    "used": true //交易对状态 上线||下线
  }
]
**/
```

### chainx.trade.getQuotations([id](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [piece](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

交易对报价列表

### 参数

- `id`:`OrderPairID` 从`chainx.trade.getOrderPairs`接口中返回的交易对ID
- `piece`:`u32` ?档 必须<=10

### 例子

```js
chainx.trade.getQuotations(0,10)
/**
{
  "buy": [
    //买N档
    [
      100000, //价格
      20 //数量
    ]
  ],
  "id": 0, //交易对ID
  "piece": 10, //N档
  "sell": [] //卖N档
}
**/
```

### chainx.trade.getOrders([who](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js), [page_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js), [page_size]())

用户挂单列表

### 参数

- `who`: AccountId 账户
- `page_index`: u32 页码 (从0开始)
- `page_size`: u32 页大小 (<=100)

### 例子

```js
chainx.trade.getOrders('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 0, 10)
/**
{
  "data": [
    {
      "amount": 40, //数量
      "class": "Limit", //订单类型（限价单|市价单)
      "createTime": 1547551914, //创建时间戳
      "direction": "Buy", //订单方向（卖|买）
      "fillIndex": [], //成交ID
      "hasfillAmount": 0, //已成交数量
      "index": 2, //订单编号
      "lastupdateTime": 1547551914, //最后更新时间戳
      "pair": 0, //交易对ID
      "price": 100000, //价格
      "reserveLast": 4000000, //锁定数量（若是卖，则是assets的数量）
      "status": "FillNo",
      //订单状态 (FillNo,FillPart,FillAll,FillPartAndCancel,Cancel)
      "user": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f" //用户账户
    }
  ],
  "pageIndex": 0, //当前页码
  "pageSize": 100, //页大小
  "pageTotal": 1 //总共页数
}

**/
```