rpc接口分为多个体系：

继承至substrate体系的rpc有：

* `author` 与提交交易相关
* `chain` 与获取链的数据相关（非业务），如区块头，区块等
* `state` 获取链上的状态信息（世界状态），该接口是最原始接口
* `system` 获取链的系统信息，如版本等

ChainX对substrate的rpc做了定制，提供了ChainX特有的rpc接口用于获取链上的一些关键数据：

* `chainx` 获取ChainX的Runtime中的一些关键数据（业务相关）

由于substrate使用的rpc服务为`jsonrpc`库，ChainX继承于这个体系，因此所有的rpc请求的方式为：

- 使用POST发送数据

- POST的参数为：

  ```bash
  {
      "id":1, # 用于并行（异步）发请求的请求标示，会在返回值中返回，如果单次请求随便一个值就可以
      "jsonrpc":"2.0", # 版本，一般为2.0
      "method":"<方法名>",
      "params":[ "<参数1>", "<参数2>" ]
  }
  ```

- rpc的返回值为json，格式为

  ```bash
  {
      "jsonrpc": "2.0",
      "result": <数据内容>,
      "id": 1 # 发送请求中的”id“，原样返回
  }
  ```

例如获取ChainX上的用户资产数据的方式为：

参数

```bash
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getAssetsByAccount", # 获取某个用户资产的方法名
	# 对应的参数，这里为 "用户公钥"，分页（从第几页开始），分页（每页多少个数据）
	"params":["0xc69287ef1ea8f806bcf3d56e39c3bec395e8aeacbab9f60b2ee23bef049e3548", 0, 100]
}
```

返回

```bash
{
    "jsonrpc": "2.0",
    "result": { # result 为返回的结果，这里是这个用户的所有资产信息
        "data": [
            {
                "details": {
                    "Free": 237463297,
                    "ReservedCurrency": 0,
                    "ReservedDexFuture": 0,
                    "ReservedDexSpot": 100000000,
                    "ReservedStaking": 150257350474,
                    "ReservedStakingRevocation": 0,
                    "ReservedWithdrawal": 0
                },
                "name": "PCX"
            },
            {
                "details": {
                    "Free": 0,
                    "ReservedCurrency": 0,
                    "ReservedDexFuture": 0,
                    "ReservedDexSpot": 0,
                    "ReservedStaking": 0,
                    "ReservedStakingRevocation": 0,
                    "ReservedWithdrawal": 0
                },
                "name": "BTC"
            }
        ],
        "pageIndex": 0,
        "pageSize": 100,
        "pageTotal": 1
    },
    "id": 1
}
```

以下对所有rpc做详细介绍，但是不再完整的写清楚发送的参数与返回值，只写：

- `"method"`：对应的\<方法名\>
- `"params"`：方法名对应的参数
- `"result"`：调用rpc的返回值

文中的引用部分为substrate源码中的注释

## author

`author` 一般是**提交交易**相关的RPC接口

### author_submitExtrinsic

提交一个16进制编码的交易到链上

> Submit hex-encoded extrinsic for inclusion in block.

调用：

- 方法名：`author_submitExtrinsic`
- 参数：`[ "0x..........." ]` 签名过且编码好的交易

返回

```bash
"result": "0x.............", # 这笔交易的hash
```

### author_pendingExtrinsics

返回目前这个节点交易池中等待打包的交易，以发送者排序

> Returns all pending extrinsics, potentially grouped by sender.

调用

- 方法名：`author_pendingExtrinsics`
- 参数： `[]` 

返回

```bash
"result": [
	"0x...............", # tx1
	"0x...............", # tx2
	...
]
```

### author_submitAndWatchExtrinsic

**仅用于websocket，见其他文档**

### author_submitAndWatchExtrinsic

**仅用于websocket，见其他文档**

## chain

获取链的一些基本数据，如区块头，区块数据

### chain_getHeader

根据hash获取对应的区块头，如果不填写hash就返回最新的区块头

> Get header of a relay chain block.

调用

- 方法名：`chain_getHeader`
- 参数： 
  - `[]`  获取最新的块头
  -  **或** `["0x......"]` 获取对应hash的块头

返回

```bash
"result": {
	# 区块头摘要信息
    "digest": {
        "logs": [
            "0x04617572612101d4d4732e000000002308a997d92c861af8ed2ca75b6bf8c380a784b106933927e251992ecfe26def3c8c7be0f538045c3b6591cca1a202309a1d81c3e1e531a013622f1c0e872d00"
        ]
    },
    # 交易根
    "extrinsicsRoot": "0x5026c75b7c7a0c841594d79ea2105c45f08f1d2ebec3ae2f33b3849f48285ac7",
    # 块高
    "number": "0x59359",
    # 父hash
    "parentHash": "0xf9378d4652d227bc65a2fb762b407dfc613b2dd59a21699543f2903dce3a4792",
    # 状态根
    "stateRoot": "0x194c78dd16986ae63849638df713ccdcf7a9391c6b60a55131b7e8ced3bd8e76"
}
```

### chain_getBlock

根据hash获取对应的区块信息（区块中包含交易），如果不填写hash就返回最新的区块

> Get header and body of a relay chain block.

调用

- 方法名：`chain_getBlock`
- 参数： 
  - `[]`  获取最新的块头
  -  **或** `["0x......"]` 获取对应hash的块头

返回

```bash
"result": {
    "block": {
    	# 当前区块中的所有交易
        "extrinsics": [
            "0x2001010003dcaae75c",
            "0x8c010600e1bb5dce598f45732d6f5796adaca4ca261387ed5b5cda96c3c588f256e8205b"
        ],
        # 区块头，与上面的接口返回相同
        "header": {
            "digest": {
                "logs": [
                    "0x046175726121016ed5732e00000000e4c6afd655c39b33df49bf4b8e3a3955db7145ec1429587ebabbcc8de6d7ab77d94f2bde910f0ea17a2d182d9cb60a75e82247975f3e285f65519381ef9d4405"
                ]
            },
            "extrinsicsRoot": "0xac8fa9c0713ad622a3fc5560cc503e458f4a4aa01a169b3faf267a467666dd88",
            "number": "0x593f3",
            "parentHash": "0x084d1353edcf83845a87320c4fc381e73039308ff6541aae1ed536f4f6a7e678",
            "stateRoot": "0x9a0b96f81cb4c92a4f98b09f5918303f2691ea601fb280c98c8db635a1e3ae31"
        }
    },
    # 验证者列表，目前已经是废弃字段
    "justification": null
}
```

### chain_getBlockHash

根据高度获取区块的hash

>Get hash of the n-th block in the canon chain. By default returns latest block hash.

调用

- 方法名：`chain_getBlockHash`
- 参数： 
  - `[]`  获取最新的块头hash
  -  **或** `[ 1000 ]` 获取对应块高为1000的块头hash

返回

```bash
"result": "0x259d61439a9c40237b3d583ed673a0d2875285f29b9f1a0a7ef522c41d4c4136"
```

### chain_getFinalizedHead

根据当前这个节点的 `finalized `的块头hash

> Get hash of the last finalized block in the canon chain.

调用

- 方法名：`chain_getFinalizedHead`
- 参数： `[]`  无参数

返回

```bash
"result": "0x259d61439a9c40237b3d583ed673a0d2875285f29b9f1a0a7ef522c41d4c4136"
```

### chain_subscribeNewHead

**仅用于websocket，见其他文档**

### chain_unsubscribeNewHead

**仅用于websocket，见其他文档**

### chain_subscribeFinalizedHeads

**仅用于websocket，见其他文档**

### chain_unsubscribeFinalizedHeads

**仅用于websocket，见其他文档**

## state

### state_call

基于一个状态（块高）调用一个Runtime API，基本上用户不需要使用

> Call a contract at a block's state.

调用

- 方法名：`state_call`

- 参数： 

  - 在最新状态（块高）上调用

      ```bash
      [
        "<runtime api 的方法名>",
        "<参数>" # 参数需要被 `encode` 序列化
      ]
      ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```bash
    [
      "<runtime api 的方法名>",
      "<参数>", # 参数需要被 `encode` 序列化
      "0x....................." # 对应的块高
    ]  
    ```

返回

```bash
"result": "0x.........................."  # result 的结果需要使用对应数据结构为类型 `decode` 反序列化得到
```

### state_getKeys

这个接口会产生潜在的攻击风险，因此从ChainX中移除

> Returns the keys with prefix, leave empty to get all the keys

### state_getStorage

> Returns a storage entry at a specific block's state.

基于一个状态（块高）给予查询状态存储的key，返回对应的状态数据，若没查找到返回`null`。由于该接口比较基础，因此一般不需要调用

调用

- 方法名：`state_call`

- 参数： 

  - 在最新状态（块高）上调用

    ```bash
    [
      "0x....", # key 被 `encode` 序列化并使用对应的hash
    ]
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```bash
    [
      "0x....", # key 被 `encode` 序列化并使用对应的hash
      "0x....................." # 对应的块高
    ]  
    ```

返回

```bash
"result": "0x........" result 的结果需要使用对应数据结构为类型 `decode` 反序列化得到
```

或未查询得到

```bash
"result": null
```

### state_getStorageHash

与`state_getStorage`接口对应，返回的是数据的hash

> Returns a storage entry at a specific block's state.

### state_getStorageSize

与`state_getStorage`接口对应，返回数据的长度

> Returns the size of a storage entry at a block's state.

### state_getMetadata

> Returns the runtime metadata as an opaque blob.

基于一个状态（块高）返回当前的`metadata`，sdk获取链上runtime信息的重要接口

调用

- 方法名：`state_getMetadata`

- 参数： 

  - 在最新状态（块高）上调用

    ```bash
    []
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```bash
    [  "0x....................." ]   # 对应的块高
    ```

返回

```bash
"result": "0x........"  # 返回值为编码过的metadata
```

注：该返回值需要使用对应的metadata编码解析，ChainX已经封装于`js sdk`当中，不对外提供解析方式。

### state_getRuntimeVersion

基于一个状态（块高）获取当前的runtime version

> Get the runtime version.

调用

- 方法名：`state_getRuntimeVersion`

- 参数： 

  - 在最新状态（块高）上调用

    ```bash
    []
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```bash
    [  "0x....................." ]   # 对应的块高
    ```

返回

```bash
"result": {
    "apis": [
        [
            "0xdf6acb689907609b",
            2
        ],
		# ...
    ],
    "authoringVersion": 1,
    "implName": "chainx-net",
    "implVersion": 0,
    "specName": "chainx",
    "specVersion": 0
}
```

### state_queryStorage

查询从一个块高到另一个块高之间一系列key对应存储的变化情况

> Query historical storage entries (by key) starting from a block given as the second parameter.
>
> NOTE This first returned result contains the initial state of storage for all keys. Subsequent values in the vector represent changes to the previous state (diffs).

改方法几乎不使用，暂不提供案例

### state_subscribeRuntimeVersion

**仅用于websocket，见其他文档**

### state_unsubscribeRuntimeVersion

**仅用于websocket，见其他文档**

### state_subscribeStorage

**仅用于websocket，见其他文档**

### state_unsubscribeStorage

**仅用于websocket，见其他文档**

## system

### system_name

返回链的名称

> Get the node's implementation name. Plain old string.

调用

- 方法名：`system_name`

- 参数： `[]`  无参数

返回

```bash
"result": "ChainX"
```

### system_version

返回链的当前的版本信息

> Get the node implementation's version. Should be a semver string.

调用

- 方法名：`system_version`

- 参数： `[]`  无参数

返回

```bash
"result": "1.0.0"
```

### system_chain

返回链的类型信息

> Get the chain's type. Given as a string identifier.

调用

- 方法名：`system_chain`

- 参数： `[]`  无参数

返回

```bash
"result": "ChainX mainnet v1.0.0"
```

### system_properties

返回链的其他附属信息

> Get a custom set of properties as a JSON object, defined in the chain spec.

调用

- 方法名：`system_properties`

- 参数： `[]`  无参数

返回

```bash
"result": {
    "address_type": 44,  # chainx 目前对于 ss58check 的地址类型
    "bitcoin_type": "mainnet", # 比特币网络类型
    "network_type": "mainnet" # chainx 网络类型
}
```

### system_health

返回链的健康信息

> Return health status of the node.
>
> Node is considered healthy if it is:
>
> - connected to some peers (unless running in dev mode)
> - not performing a major sync

调用

- 方法名：`system_health`

- 参数： `[]`  无参数

返回

```bash
"result": {
    "isSyncing": false,  # 是否在同步中
    "peers": 19,  # p2p 连接的节点
    "shouldHavePeers": false
}
```

### system_peers

返回链的p2p连接信息

> Returns currently connected peers

调用

- 方法名：`system_peers`

- 参数： `[]`  无参数

返回

```bash
"result": [
    {
        "bestHash": "0xd14421e3c60f50218659dc100689457621a380ff77f60a413442a018e58a2e0e",
        "bestNumber": 368205,
        "peerId": "QmNdXNb82c2SYkSHhMeAfH3iRSJpdgSmsDjsw5ZPYUfjVt",
        "protocolVersion": 3,
        "roles": "AUTHORITY"
    },
    {
        "bestHash": "0x3e98ffdc4e7375fffc43ac7051c641b65f655847b4c2fbfa205c4f5b164f551d",
        "bestNumber": 368208,
        "peerId": "QmVNcDfHhiex6EhzDScC92NvgwN2krf6wpB19L6ruk2Hdt",
        "protocolVersion": 3,
        "roles": "AUTHORITY"
    }
    # ...
}
```

### system_networkState

返回链的网络信息

> Returns current state of the network.

调用

- 方法名：`system_networkState`

- 参数： `[]`  无参数

返回

> 暂时不提供

## chainx

ChainX部分的rpc与ChainX链上的业务逻辑相关，主要返回ChainX自己定义的相应数据

### 链相关部分

#### chainx_getBlockByNumber

由于在`chain`部分，substrate只提供的提供块高，返回区块hash，再由区块hash查询区块数据，因此ChianX提供使用块高查询区块数据的方式，若不提供块高，则放回目前最高的块的数据

调用

- 方法名：`chainx_getBlockByNumber`

- 参数： 

  - 获取最新的块信息

    ```bash
    []
    ```

  - **或**给予一个块高，返回对应的块数据

    ```bash
    [  100 ]   # 对应的块高
    ```

返回

> 返回结果与`chain_getBlock`相同

### 资产部分

#### chainx_getAssetsByAccount

获取用户资产信息

调用

* 方法名：`chainx_getAssetsByAccount`

* 参数：

  ```bash
  [
  	"0x................", # 需要查询的账户的公钥
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```bash
"result": {
	"data": [
		# 资产列表， 内容同上
		{
			"details": { # key 是资产类型的枚举
				"Free": 1000000000,
				"ReservedDexFuture": 0,
				"ReservedDexSpot": 0,
				"ReservedStaking": 0,
				"ReservedStakingRevocation": 0,
				"ReservedWithdrawal": 0
			},
			"name": "PCX"
		},
		# ...
	],
	"pageIndex": 0, # 当前页码
	"pageSize": 10, # 页大小
	"pageTotal": 1 # 总共页数
}
```

#### chainx_getAssets

获取资产信息

调用

- 方法名：`chainx_getAssets`

- 参数：

  ```bash
  [
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```bash
"result": {
	"data": [
		{
			"chain": "ChainX",  # 资产归属
			"desc": "ChainX's crypto currency in Polkadot ecology",
			"details": {
				"Free": 5229681012820, # 总活动资产数目
				"ReservedCurrency": 0, 
				"ReservedDexFuture": 0, # 总期货交易锁定资产数目
				"ReservedDexSpot": 39340717400, # 总交易锁定资产数目
				"ReservedStaking": 7013178269779, # 总投票锁定资产数目
				"ReservedStakingRevocation": 22800000001, # 总投票赎回锁定资产数目
				"ReservedWithdrawal": 0 # 总提现锁定数目
			},
			"name": "PCX",
			"online": true,
			"precision": 8,
			"tokenName": "Polkadot ChainX"
		},
		# ...
	]
}
```

### 充值提现相关部分

#### chainx_getWithdrawalList

获取某条链的当前所有提现中记录

调用

- 方法名：`chainx_getWithdrawalList`

- 参数：

  ```bash
  [
  	"Bitcoin", # 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```bash
{
  "data": [
    {
      "address": "mjKE11gjVN4JaC9U8qL6ZB5vuEBgmwik7b", # 提现地址
      "accountid": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", # 提现申请人
      "balance": 10, # 提现金额
      "memo": "", # 提现备注 或其他
      "id": 0, # 提现序列号
      "status": {
      		"value": "Applying", # 提现进入对应链上的提现状态， "Applying" 申请中, "Signing" 签名中, "Broadcasting" 广播中, Confirming 确认中
      		"confirm": 1,  # 只有当`value`为`Confirming`的时候才会有这个值，表示这个交易已有几个确认
      		"totalConfirm": 4, # 只有当`value`为`Confirming`的时候才会有这个值，表示这个交易总共需要几个确认
      },
      "applicationStatus": # 针对交易记录的状态["Applying" 申请中, "Processing" 处理中]
      "height": 1000 # 申请提现的块高
      "token": "BTC", # 提现资产名
      "txid": "" # 对应链的交易id，如Bitcoin就是btc交易的hash
    },
    # ...
  ],
  "pageIndex": 0, # 同上
  "pageSize": 100,
  "pageTotal": 1
}

```

#### chainx_getDepositList

获取某条链的充值中列表

调用

- 方法名：`chainx_getDepositList`

- 参数：

  ```bash
  [
  	"Bitcoin", # 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```js
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
```

#### chainx_getWithdrawalLimitByToken

chainx 链上与提现金额相关的限制，如提现的最小值与用户提现扣除的手续费

调用

- 方法名：`chainx_getWithdrawalLimitByToken`

- 参数：

  ```bash
  ["BTC" ] # 资产名称 目前支持的是 BTC 其他不支持
  ```

返回

```bash
{
	minimalWithdrawal: 150000,
	fee: 100000
}
```

#### chainx_getAddressByAccount

ChainX账户绑定BTC地址列表

调用

- 方法名：`chainx_getAddressByAccount`

- 参数：

  ```bash
  [
  	"0x..........", # 账户公钥
  	"Bitcoin" # 链ID 枚举[“ChainX”, “Bitcoin”, “Ethereum”, “Polkadot”]
  ] 
  ```

返回

```bash
[
  "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7" # Bitcoin 地址列表
]
```

#### chainx_verifyAddressValidity

验证提现地址正确性

调用

- 方法名：`chainx_verifyAddressValidity`

- 参数：

  ```bash
  [
  	"BTC", # 币种名
  	"mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7", # 需要校验的提现地址
  	"memo" # 备注 （注：3个参数与使用js sdk 提起提现请求时候交易`withdraw` 的参数列表完全一致）
  ] 
  ```

返回

```bash
[
	true # 或者 false, 返回 true 代表地址校验正确，false代表错误
]
```

### 验证者部分

#### chainx_getNominationRecords

用户投票信息

调用

- 方法名：`chainx_getNominationRecords`

- 参数：

  ```bash
  [ "0x........." ] # 查询的账户公钥
  ```

返回

```js
"result": [
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
```

#### chainx_getIntentions

获取节点列表

调用

- 方法名：`chainx_getNominationRecords`
- 参数：`[]` 无参数

返回

```js
"result": [
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
```

#### chainx_getPseduIntentions

充值挖矿列表

调用

- 方法名：`chainx_getPseduIntentions`
- 参数：`[]` 无参数

返回

```js
"result": [
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
```

#### getPseduNominationRecords

用户投票信息

调用

- 方法名：`chainx_getPseduIntentions`
- 参数：`["0x........."]` 查询用户的公钥

返回

```js
"result": [
  {
    "balance": 1000000000, // 投票金额
    "id": "BTC", // 资产 ID
    "lastTotalDepositWeight": 0, // 票龄
    "lastTotalDepositWeightUpdate": 0 // 票龄更新高度
  }
]
```

### 交易所部分

#### chainx_getTradingPairs

交易对列表

调用

- 方法名：`chainx_getTradingPairs`
- 参数：`[]` 无参数

返回

```bash
"result": [
	{
		"assets": "PCX", # 交易对first
		"averPrice": 271325, # 平均价
		"buyOne": 1605400, # 买一价
		"currency": "BTC", # 交易对second
		"id": 0, # 交易对的ID
		"lastPrice": 400000,
		"maximumBid": 1615500,
		"minimumOffer": 1595400,
		"online": true, # 交易对状态 上线||下线
		"precision": 9, # 交易对价格精度 10.pow(precision)
		"sellOne": 1605500,
		"unitPrecision": 2,
		"updateHeight": 371940
	},
	# ...
]
```

#### chainx_getQuotations

交易对报价列表

调用

- 方法名：`chainx_getQuotations`

- 参数：

  ```bash
  [
  	0, # id`:`OrderPairID` 从`chainx_getTradingPairs`接口中返回的交易对ID
  	10, # 档 必须<=10
  ]
  ```

返回

```bash
"result": {
  "buy": [
    //买N档
    [
      100000, //价格
      20 //数量
    ]
      # ...
  ],
  "id": 0, //交易对ID
  "piece": 10, //N档
  "sell": [] //卖N档
}
```

#### chainx_getOrders

用户挂单列表

调用

- 方法名：`chainx_getOrders`

- 参数：

  ```bash
  [
  	"0x..............", # 用户公钥
  	0, # 分页，起始页
  	100 # 分页，每页大小
  ]
  ```

返回

```bash
"result": {
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
```

### 信托部分

#### chainx_getTrusteeSessionInfo

根据传入的链返回当前届的信托相关信息

调用

- 方法名：`chainx_getTrusteeSessionInfo`

- 参数：

  ```bash
  [	"Bitcoin" ]  # 链id, 对于这个接口只有Bitcoin
  ```

返回

```js
"result": {
	"coldEntity": {
        "addr": "3Ne676pvdydEp425KdHDAEsff5dososuKL",
        "redeemScript": "0x522102132a8a42447c734deca086bd3087c73e0e3b323310b0490eb370532793d1f95f2102ba70207d7c47c22bd46acf91340bc725ddb8e33c65880ca2d1dba585491492542102fabd0895598ee8c11a6d4f4610a8b872fe5cefa2d62e82fef12825ec7901e32c53ae"
    },
    "counts": {
        "required": 2,
        "total": 3
    },
    "hotEntity": {
        "addr": "3FK7PJiukCS57aPhMdtc6soqKTrQogX2Zz",
        "redeemScript": "0x52210265d615b6bb17e1d93a7836060ea58ed07aa31da5da25ee75ff3ed6c90c787509210342878f854c8c13e160d6aadb611aff7caea2e24d8c661bfce14920d9d27ede892102a3ee44550a9c13575e99f96fc56db20e2f7f686276c82a5737bacca07163571f53ae"
    },
	"sessionNumber": 0, // 当前信托届数，每换届一次就会增加1
	"trusteeList": [
		// 当前这条链的信托节点账户列表
    	{
        	"accountId": "0xf75d1c241c59e2a6a88efcf239df0eabb229b3697b4c844c788c9c51fcb25d2d",
        	"props": {
        	    "about": "bitocin",
        	    "coldEntity": "0x02132a8a42447c734deca086bd3087c73e0e3b323310b0490eb370532793d1f95f",
        	    "hotEntity": "0x0265d615b6bb17e1d93a7836060ea58ed07aa31da5da25ee75ff3ed6c90c787509"
        	}
        },
        {
            "accountId": "0x5f423525f65a2b6eb5866a9159dee5edebb00c85e43147bf02ef1921590c4df1",
            "props": {
                "about": "bitocin",
                "coldEntity": "0x02ba70207d7c47c22bd46acf91340bc725ddb8e33c65880ca2d1dba58549149254",
                "hotEntity": "0x0342878f854c8c13e160d6aadb611aff7caea2e24d8c661bfce14920d9d27ede89"
            }
        },
        {
            "accountId": "0x98c51995fdc63afbe50b010c71b5cabe14e270d1665e82f958c39bdd5bfb8f9a",
            "props": {
                "about": "bitocin",
                "coldEntity": "0x02fabd0895598ee8c11a6d4f4610a8b872fe5cefa2d62e82fef12825ec7901e32c",
                "hotEntity": "0x02a3ee44550a9c13575e99f96fc56db20e2f7f686276c82a5737bacca07163571f"
            }
        }
	]
}
```

#### chainx_getMockBitcoinNewTrustees

仅对于Bitcoin而言，由于信托换届需要进行下届信托地址有效性的测试，因此首先通过该接口模拟生成下一届的地址信息

调用

- 方法名：`chainx_getMockBitcoinNewTrustees`

- 参数：

  ```bash
  [ 
  	[ # 需要模拟参与生成的信托候选的公钥，是个列表
  		"0x........",
  		"0x........",
  		# ..
      ]
  ] 
  ```

返回（注：返回值和`chainx_getTrusteeSessionInfo`一样，因为使用的相同的实现）

```bash
"result": {
	"coldEntity": {
        "addr": "3Ne676pvdydEp425KdHDAEsff5dososuKL",
        "redeemScript": "0x522102132a8a42447c734deca086bd3087c73e0e3b323310b0490eb370532793d1f95f2102ba70207d7c47c22bd46acf91340bc725ddb8e33c65880ca2d1dba585491492542102fabd0895598ee8c11a6d4f4610a8b872fe5cefa2d62e82fef12825ec7901e32c53ae"
    },
    "counts": {
        "required": 2,
        "total": 3
    },
    "hotEntity": {
        "addr": "3FK7PJiukCS57aPhMdtc6soqKTrQogX2Zz",
        "redeemScript": "0x52210265d615b6bb17e1d93a7836060ea58ed07aa31da5da25ee75ff3ed6c90c787509210342878f854c8c13e160d6aadb611aff7caea2e24d8c661bfce14920d9d27ede892102a3ee44550a9c13575e99f96fc56db20e2f7f686276c82a5737bacca07163571f53ae"
    },
	"sessionNumber": 0, // 当前信托届数，每换届一次就会增加1
	"trusteeList": [
		// 当前这条链的信托节点账户列表
    	{
        	"accountId": "0xf75d1c241c59e2a6a88efcf239df0eabb229b3697b4c844c788c9c51fcb25d2d",
        	"props": {
        	    "about": "bitocin",
        	    "coldEntity": "0x02132a8a42447c734deca086bd3087c73e0e3b323310b0490eb370532793d1f95f",
        	    "hotEntity": "0x0265d615b6bb17e1d93a7836060ea58ed07aa31da5da25ee75ff3ed6c90c787509"
        	}
        },
        {
            "accountId": "0x5f423525f65a2b6eb5866a9159dee5edebb00c85e43147bf02ef1921590c4df1",
            "props": {
                "about": "bitocin",
                "coldEntity": "0x02ba70207d7c47c22bd46acf91340bc725ddb8e33c65880ca2d1dba58549149254",
                "hotEntity": "0x0342878f854c8c13e160d6aadb611aff7caea2e24d8c661bfce14920d9d27ede89"
            }
        },
        {
            "accountId": "0x98c51995fdc63afbe50b010c71b5cabe14e270d1665e82f958c39bdd5bfb8f9a",
            "props": {
                "about": "bitocin",
                "coldEntity": "0x02fabd0895598ee8c11a6d4f4610a8b872fe5cefa2d62e82fef12825ec7901e32c",
                "hotEntity": "0x02a3ee44550a9c13575e99f96fc56db20e2f7f686276c82a5737bacca07163571f"
            }
        }
	]
}
```

#### chainx_getTrusteeInfoByAccount

获取该用户的信托信息

调用

- 方法名：`chainx_getTrusteeInfoByAccount`

- 参数：

  ```bash
  [ "0x.........." ] # 用户的公钥
  ```

返回

```bash
"result": {
    "Bitcoin": {
        "about": "bitocin", # 注册信息
        "coldEntity": "0x02132a8a42447c734deca086bd3087c73e0e3b323310b0490eb370532793d1f95f", # 注册的冷公钥
        "hotEntity": "0x0265d615b6bb17e1d93a7836060ea58ed07aa31da5da25ee75ff3ed6c90c787509" # 注册的热公钥
    },
    # ...
},
```

#### chainx_getWithdrawTx

获取对应链的提现交易，已经与这个交易相关的ChainX申请提现的用户提现记录id列表（用于信托获取对应链上正在处理中的多签提现交易，如对于BTC而言，该接口返回信托需要进行签署的比特币提现待签原文以及相关信息）

调用

- 方法名：`chainx_getWithdrawTx`

- 参数：

  ```bash
  [ "Bitcoin" ] # 链id，当前只有Bitcoin
  ```

返回

```bash
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
```

### 其他

#### chainx_particularAccounts

ChainX链上的特殊地址

调用

- 方法名：`chainx_particularAccounts`
- 参数：`[]` 无参数

返回

```bash
"result": {
    "councilAddress": "0x67df26a755e0c31ac81e2ed530d147d7f2b9a3f5a570619048c562b1ed00dfdd",
    "teamAddress": "0x6193a00c655f836f9d8a62ed407096381f02f8272ea3ea0df0fd66c08c53af81",
    "trusteesAddress": { # 当前信托多签地址
        "Bitcoin": "0x8dc93cedb19de3341ca919160a58fe2497ed23b38169933f240007d1d703e2d5"
    }
}
```

