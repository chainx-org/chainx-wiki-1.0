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

  ```jsonc
  {
      "id":1, # 用于并行（异步）发请求的请求标示，会在返回值中返回，如果单次请求随便一个值就可以
      "jsonrpc":"2.0", # 版本，一般为2.0
      "method":"<方法名>",
      "params":[ "<参数1>", "<参数2>" ]
  }
  ```

- rpc的返回值为json，格式为

  ```jsonc
  {
      "jsonrpc": "2.0",
      "result": <数据内容>,
      "id": 1 # 发送请求中的”id“，原样返回
  }
  ```

例如获取ChainX上的用户资产数据的方式为：

参数

```jsonc
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getAssetsByAccount", # 获取某个用户资产的方法名
	# 对应的参数，这里为 "用户公钥"，分页（从第几页开始），分页（每页多少个数据）
	"params":["0xc69287ef1ea8f806bcf3d56e39c3bec395e8aeacbab9f60b2ee23bef049e3548", 0, 100]
}
```

返回

```jsonc
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

注意：

- 以下接口中以 V1 结尾与无 V1 的区别在于其中一个字段的类型不同，其他均一致。对于类型不同的字段，V1 版本中类型为 String, 无 V1 标识的版本中对应字段类型为 Number(对应 rust 中类型为 u64). 这是由于 rust [serde-json](https://github.com/serde-rs/json/issues/559) 库 u128 反序列化问题而采取的 workaround.

<!-- TOC GFM -->

* [author](#author)
    * [author_submitExtrinsic](#author_submitextrinsic)
    * [author_pendingExtrinsics](#author_pendingextrinsics)
    * [author_submitAndWatchExtrinsic](#author_submitandwatchextrinsic)
    * [author_submitAndWatchExtrinsic](#author_submitandwatchextrinsic-1)
* [chain](#chain)
    * [chain_getHeader](#chain_getheader)
    * [chain_getBlock](#chain_getblock)
    * [chain_getBlockHash](#chain_getblockhash)
    * [chain_getFinalizedHead](#chain_getfinalizedhead)
    * [chain_subscribeNewHead](#chain_subscribenewhead)
    * [chain_unsubscribeNewHead](#chain_unsubscribenewhead)
    * [chain_subscribeFinalizedHeads](#chain_subscribefinalizedheads)
    * [chain_unsubscribeFinalizedHeads](#chain_unsubscribefinalizedheads)
* [state](#state)
    * [state_call](#state_call)
    * [state_getKeys](#state_getkeys)
    * [state_getStorage](#state_getstorage)
    * [state_getStorageHash](#state_getstoragehash)
    * [state_getStorageSize](#state_getstoragesize)
    * [state_getMetadata](#state_getmetadata)
    * [state_getRuntimeVersion](#state_getruntimeversion)
    * [state_queryStorage](#state_querystorage)
    * [state_subscribeRuntimeVersion](#state_subscriberuntimeversion)
    * [state_unsubscribeRuntimeVersion](#state_unsubscriberuntimeversion)
    * [state_subscribeStorage](#state_subscribestorage)
    * [state_unsubscribeStorage](#state_unsubscribestorage)
* [system](#system)
    * [system_name](#system_name)
    * [system_version](#system_version)
    * [system_chain](#system_chain)
    * [system_properties](#system_properties)
    * [system_health](#system_health)
    * [system_peers](#system_peers)
    * [system_networkState](#system_networkstate)
* [chainx](#chainx)
    * [链相关部分](#链相关部分)
        * [chainx_getBlockByNumber](#chainx_getblockbynumber)
        * [chainx_getExtrinsicsEventsByBlockHash](#chainx_getetrinsicseventsbyblockhash)
    * [资产部分](#资产部分)
        * [chainx_getAssetsByAccount](#chainx_getassetsbyaccount)
        * [chainx_getAssets](#chainx_getassets)
    * [充值提现相关部分](#充值提现相关部分)
        * [chainx_getWithdrawalList](#chainx_getwithdrawallist)
        * [chainx_getDepositList](#chainx_getdepositlist)
        * [chainx_getWithdrawalLimitByToken](#chainx_getwithdrawallimitbytoken)
        * [chainx_getDepositLimitByToken](#chainx_getdepositlimitbytoken)
        * [chainx_getAddressByAccount](#chainx_getaddressbyaccount)
        * [chainx_verifyAddressValidity](#chainx_verifyaddressvalidity)
    * [验证者部分](#验证者部分)
        * [chainx_getStakingDividendByAccount](#chainx_getstakingdividendbyaccount)
        * [chainx_getCrossMiningDividendByAccount](#chainx_getcrossminingdividendbyaccount)
        * [chainx_getNominationRecords](#chainx_getnominationrecords)
        * [chainx_getNominationRecordsV1](#chainx_getnominationrecordsv1)
        * [chainx_getNextRenominateByAccount](#chainx_getnextrenominatebyaccount)
        * [chainx_getIntentionByAccount](#chainx_getintentionbyaccount)
        * [chainx_getIntentionByAccountV1](#chainx_getintentionbyaccountv1)
        * [chainx_getIntentions](#chainx_getintentions)
        * [chainx_getIntentionsV1](#chainx_getintentionsv1)
        * [chainx_getPseduIntentions](#chainx_getpseduintentions)
        * [chainx_getPseduIntentionsV1](#chainx_getpseduintentionsv1)
        * [chainx_getPseduNominationRecords](#chainx_getpsedunominationrecords)
        * [chainx_getPseduNominationRecordsV1](#chainx_getpsedunominationrecordsv1)
    * [交易所部分](#交易所部分)
        * [chainx_getTradingPairs](#chainx_gettradingpairs)
        * [chainx_getQuotations](#chainx_getquotations)
        * [chainx_getOrders](#chainx_getorders)
    * [信托部分](#信托部分)
        * [chainx_getTrusteeSessionInfo](#chainx_gettrusteesessioninfo)
        * [chainx_getMockBitcoinNewTrustees](#chainx_getmockbitcoinnewtrustees)
        * [chainx_getTrusteeInfoByAccount](#chainx_gettrusteeinfobyaccount)
        * [chainx_getWithdrawTx](#chainx_getwithdrawtx)
    * [手续费部分](#手续费部分)
        * [chainx_getFeeByCallAndLength](#chainx_getfeebycallandlength)
        * [chainx_getFeeWeightMap](#chainx_getfeeweightmap)
    * [合约部分](#合约部分)
        * [chainx_contractCall](#chainx_contractCall)
        * [chainx_contractGetStorage](#chainx_contractGetStorage)
        * [chainx_contractXRCTokenInfo](#chainx_contractXRCTokenInfo)
        * [chainx_contractXRC20Call](#chainx_contractXRC20Call)
    * [其他](#其他)
        * [chainx_particularAccounts](#chainx_particularaccounts)

<!-- /TOC -->

## author

`author` 一般是**提交交易**相关的RPC接口

对于`author` 的所有接口，错误处理均遵循以下约定

错误以以下格式返回：
```jsonc
{
	"code": <错误码>,
	"message": <错误信息>,
	"data": <详细信息或者一个int类型的数字，标识错误>
}
```
其中code与messge和data的关系如下：
> 交易格式错误： code: 1001 -> message: "Extrinsic has invalid format."，data: null
>
> Runtime内部验证错误(一般不会发生)：code:1002 -> message: <根据内部错误决定>，data：<根据内部错误决定>
>
> 非法交易：code: 1010 -> message："Invalid Transaction"
>
> 			date 是一个int类型的数字，用于区分不同的错误类型：
> 			
> 			date: 0  -> 签名验证错误
> 			
> 			date: 1  -> account nonce 小于当前链上记录的nonce
> 			
> 			date: 2  -> account nonce 超过当前链上记录的nonce
> 			
> 			date: 3  -> 发送用户的资金不够（小于手续费的检查）
> 			
> 			date: -1  -> 当前不允许调用这个方法
> 			
> 			date: -2  -> 当前的发送者已经被禁止发送交易
> 			
> 			date: -10  -> 解析出错误的 account nonce
> 			
> 			date: -20  -> 未解析出发送者
> 			
> 			date: -30  -> 加速 acc 不能为0，或者未设置
> 			
> 			date: -127  -> 未知内部错误（一般不发生）
>
> 交易有效性无法验证：code: 1011 -> message："Unknown Transaction Validity"， date: -10  -> 解析出错误的 account nonce
>
> 交易暂时被交易池禁止：code：1012 -> message: "Transaction is temporarily banned", date: null 
>
> 			出现这种情况一般需要调高交易的加速acc再重新发送
>
> 交易已经存在于交易池中：code：1013 -> message: "Transaction Already Imported", date: <交易hash>
>
> 当前交易的优先级过低：code: 1014 -> message: "Priority is too low: ({} vs {})", data: "The transaction has too low priority to replace another transaction already in the pool."
>
> 			出现这种情况一般需要调高交易的加速acc再重新发送
>
> 交易依赖循环(当前不会出现)：code: 1015 -> message: "Cycle Detected", data: null  
>
> 当前交易池已经达到上限，不接受交易：code: 1016 -> message: "Immediately Dropped", data: "The transaction couldn't enter the pool because of the limit"
>

### author_submitExtrinsic

提交一个16进制编码的交易到链上

> Submit hex-encoded extrinsic for inclusion in block.

调用：

- 方法名：`author_submitExtrinsic`
- 参数：`[ "0x..........." ]` 签名过且编码好的交易

返回

```jsonc
"result": "0x.............", # 这笔交易的hash
```



### author_pendingExtrinsics

返回目前这个节点交易池中等待打包的交易，以发送者排序

> Returns all pending extrinsics, potentially grouped by sender.

调用

- 方法名：`author_pendingExtrinsics`
- 参数： `[]` 

返回

```jsonc
"result": [
	"0x...............", # tx1
	"0x...............", # tx2
	...
]
```

### author_submitAndWatchExtrinsic

**仅用于websocket，待Substrate官方文档完成后补充**

### author_submitAndWatchExtrinsic

**仅用于websocket，待Substrate官方文档完成后补充**

## chain

获取链的一些基本数据，如区块头，区块数据

对于`chain`的所有接口，错误处理均遵循以下约定

错误以以下格式返回：

```jsonc
{
	"code": <错误码>,
	"message": <错误信息>,
	"data": <详细信息或者一个int类型的数字，标识错误>
}
```

其中code与messge和data的关系如下：

> 内部错误：-32603 -> message: <依据错误类型决定>， data: <依据错误决定>
>
> 其他错误：3001  -> message: <其他错误信息>, data: null

### chain_getHeader

根据hash获取对应的区块头，如果不填写hash就返回最新的区块头

> Get header of a relay chain block.

调用

- 方法名：`chain_getHeader`
- 参数： 
  - `[]`  获取最新的块头
  -  **或** `["0x......"]` 获取对应hash的块头

返回

```jsonc
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

```jsonc
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

```jsonc
"result": "0x259d61439a9c40237b3d583ed673a0d2875285f29b9f1a0a7ef522c41d4c4136"
```

### chain_getFinalizedHead

根据当前这个节点的 `finalized `的块头hash

> Get hash of the last finalized block in the canon chain.

调用

- 方法名：`chain_getFinalizedHead`
- 参数： `[]`  无参数

返回

```jsonc
"result": "0x259d61439a9c40237b3d583ed673a0d2875285f29b9f1a0a7ef522c41d4c4136"
```

### chain_subscribeNewHead

**仅用于websocket，待Substrate官方文档完成后补充**

### chain_unsubscribeNewHead

**仅用于websocket，待Substrate官方文档完成后补充**

### chain_subscribeFinalizedHeads

**该接口目前暂时不可用**

### chain_unsubscribeFinalizedHeads

**该接口目前暂时不可用**

## state

`state`是用于直接获取链上的raw存储，或者基于某个状态调用runtime api

对于`state`的所有接口，错误处理均遵循以下约定

错误以以下格式返回：

```jsonc
{
	"code": <错误码>,
	"message": <错误信息>,
	"data": <详细信息或者一个int类型的数字，标识错误>
}
```

其中code与messge和data的关系如下：

> 内部错误：-32603 -> message: <依据错误类型决定>， data: <依据错误决定>
>
> 非法区块范围：4001  -> message: "Cannot resolve a block range ['{:?}' ... '{:?}]. {}", data: null

### state_call

基于一个状态（块高）调用一个Runtime API，基本上用户不需要使用

> Call a contract at a block's state.

调用

- 方法名：`state_call`

- 参数： 

  - 在最新状态（块高）上调用

      ```jsonc
      [
        "<runtime api 的方法名>",
        "<参数>" # 参数需要被 `encode` 序列化
      ]
      ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```jsonc
    [
      "<runtime api 的方法名>",
      "<参数>", # 参数需要被 `encode` 序列化
      "0x....................." # 对应的块高
    ]  
    ```

返回

```jsonc
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

    ```jsonc
    [
      "0x....", # key 被 `encode` 序列化并使用对应的hash
    ]
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```jsonc
    [
      "0x....", # key 被 `encode` 序列化并使用对应的hash
      "0x....................." # 对应的块高
    ]  
    ```

返回

```jsonc
"result": "0x........" result 的结果需要使用对应数据结构为类型 `decode` 反序列化得到
```

或未查询得到

```jsonc
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

    ```jsonc
    []
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```jsonc
    [  "0x....................." ]   # 对应的块高
    ```

返回

```jsonc
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

    ```jsonc
    []
    ```

  - **或**给予一个区块hash，在这个hash对应的状态（块高）上调用

    ```jsonc
    [  "0x....................." ]   # 对应的块高
    ```

返回

```jsonc
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

**仅用于websocket，待Substrate官方文档完成后补充**

### state_unsubscribeRuntimeVersion

**仅用于websocket，待Substrate官方文档完成后补充**

### state_subscribeStorage

**仅用于websocket，待Substrate官方文档完成后补充**

### state_unsubscribeStorage

**仅用于websocket，待Substrate官方文档完成后补充**

## system

`system`用于提供一些当前节点系统信息的查询，如查询网络状态，当前节点版本等等

对于`system`的所有接口，错误处理均遵循以下约定

错误以以下格式返回：

```jsonc
{
	"code": <错误码>,
	"message": <错误信息>,
	"data": <详细信息或者一个int类型的数字，标识错误>
}
```

其中code与messge和data的关系如下：

> 非健康状态：2001  -> message: "Node is not fully functional: {}", data: \<json 形式的字符串\>

### system_name

返回链的名称

> Get the node's implementation name. Plain old string.

调用

- 方法名：`system_name`

- 参数： `[]`  无参数

返回

```jsonc
"result": "ChainX"
```

### system_version

返回链的当前的版本信息

> Get the node implementation's version. Should be a semver string.

调用

- 方法名：`system_version`

- 参数： `[]`  无参数

返回

```jsonc
"result": "1.0.0"
```

### system_chain

返回链的类型信息

> Get the chain's type. Given as a string identifier.

调用

- 方法名：`system_chain`

- 参数： `[]`  无参数

返回

```jsonc
"result": "ChainX mainnet v1.0.0"
```

### system_properties

返回链的其他附属信息

> Get a custom set of properties as a JSON object, defined in the chain spec.

调用

- 方法名：`system_properties`

- 参数： `[]`  无参数

返回

```jsonc
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

```jsonc
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

```jsonc
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

ChainX部分的rpc与ChainX链上的业务逻辑相关，主要返回ChainX自己定义的相应数据。

对于`chainx`的所有接口，错误处理均遵循以下约定。由于`chainx`接口除了“合约”相关部分，均为查询接口，因此返回的错误表示其链上数据已经是错误的，或是查询使用的参数有错误。

错误以以下格式返回：

```jsonc
{
	"code": <错误码>,
	"message": <错误信息>,
	"data": <详细信息或者一个int类型的数字，标识错误>
}
```

其中code与messge和data的关系如下：

> 当前未实现：code: 1 -> message: <依据错误类型决定>， data: null
>
> 挂单错误：code: 1605-> message: "Quotations Piece Err: piece:{}", data: null
>
> 以页查询时传入的页大小参数错误：code: 1607-> message: "Page Size Must Between 0~100, size:{}", data: null
>
> 以页查询时传入的页索引参数错误：code: 1608-> message: "Page Index Error, index:{}", data: null
>
> 解码错误：code: 1609-> message: "Decode Data Error", data: null
>
> 十六进制参数传入的字符串必须以0x开头：code: 1610-> message: "Start With 0x", data: null
>
> 十六进制参数解析错误：code: 1611-> message: "Decode Hex Err", data: null
>
> Runtime 内部查询错误：code: 1613-> message: "Runtime error, reason: {{{:}}}", data: <内部错误信息>
>
> v0版本的rpc接口废弃：code: 1614-> message: "{:} is Deprecated, Please Use {:}V1 Instead", data: null
>
> 存储不存在或非archive查不到过于久远的历史信息：code: 1615-> message: "Storage record does not exist or not in archive", data: null
>
> 区块不存在：code: 1616-> message: "BlockNumber not exist for this hash", data: null
>
> 获取合约存储部分：
>
> ​		合约不存在：code: 1700-> message: "The specified contract doesn't exist.", data: null
>
> ​		合约当前是墓碑状态，不可获取存储（当前不会发生）：code: 1701-> message: "The contract is a tombstone and doesn't have any storage.", data: null
>
> 内部错误：-32603 -> message: <依据错误类型决定>， data: <依据错误决定>

**注意：**

除了 `chainx_getBlockByNumber`，如未做特殊说明， `chainx_*`的所有RPC接口均支持在最后一个参数指定块hash查询某个块下的状态，用于重现该块的那个时刻下的历史数据。如果不填默认是best_hash.

例如：

使用`chainx_getAssetsByAccount`获取某个账户下的资产金额，若发送参数为：

```bash
[
	"0x................", # 需要查询的账户的公钥
	0, # 分页（从第几页开始）
	100  # 分页（每页多少个数据）
]
```

则是获取最新块状态下的资产金额。

若在以上参数末尾加上某个块的块hash

```bash
[
	"0x................", # 需要查询的账户的公钥
	0, # 分页（从第几页开始）
	100,  # 分页（每页多少个数据）
	"0x................." # 指定某一个块的块hash
]
```

则可以获取到该块的那个时候下的该账户的资产金额。

### 链相关部分

#### chainx_getBlockByNumber

由于在`chain`部分，substrate只提供的提供块高，返回区块hash，再由区块hash查询区块数据，因此ChianX提供使用块高查询区块数据的方式，若不提供块高，则放回目前最高的块的数据

调用

- 方法名：`chainx_getBlockByNumber`

- 参数： 

  - 获取最新的块信息

    ```jsonc
    []
    ```

  - **或**给予一个块高，返回对应的块数据

    ```jsonc
    [  100 ]   # 对应的块高
    ```

返回

> 返回结果与`chain_getBlock`相同

#### chainx_getExtrinsicsEventsByBlockHash

#####  v1.0.6

提供一个区块的hash，返回这个区块下的所有events的信息。由于`Event`结构多样且复杂，目前ChainX提供的`Event`列表**没有进行结构化处理**，而是以纯文本的形式展示。用户对于简单的event处理**可以采用正则匹配**的方式去获取自己需要的信息。今后若event的需求大时，ChainX会考虑将`Event`的数据结构处理成结构化的形式返回。

该rpc返回对应块下，交易于块中index对应的Event列表。也就是说，从`chain_getBlock`或者`chainx_getBlockByNumber`rpc接口返回数据的`extrinsics` 部分的顺序，按照其`index`即是该接口`events`部分的map对应的索引。

例：通过`hash1` 调用`chain_getBlock` 获得区块信息如下：

```jsonc
"result": {
    "block": {
    	# 当前区块中的所有交易
        "extrinsics": [
            "0x2001010003dcaae75c", # 内部交易
            "0x8c010600e1bb5dce598f45732d6f5796adaca4ca261387ed5b5cda96c3c588f256e8205b"， # 内部交易
            "0x......................................." # 外部交易
        ],
        # 区块头，与上面的接口返回相同
        "header": {
            # ....
        }
    },
    # 验证者列表，目前已经是废弃字段
    "justification": null
}
```

则使用相同的`hash1`调用该接口后将会返回例如：

```jsonc
{
	"events": {
		"0": [ "...", "..."], # 对应 上文`extrinsics` 中列表中的第一条内部交易
		"1": [ "...", "..."], # 对应 上文`extrinsics` 中列表中的第二条内部交易
		"2": [ "...", "..."] # 对应 上文`extrinsics` 中列表中的第三条外部交易
	}
}
```

调用

- 方法名：`chainx_getExtrinsicsEventsByBlockHash`

- 参数： 

  - 若不提供任何参数，默认返回最近块下的Event

    ```jsonc
    []
    ```

  - **或**给予一个块的hash ，返回对应的块下的Event

    ```jsonc
    [  "0x.............." ]   # 对应的块的hash
    ```

返回

```jsonc
"result": {
        "blockHash": "0x99ed77b4737117e6920bc5c48c22c5e034ce96851790044b9f69f248fdadf3ff",
        "events": {
            "0": [
                "system(ExtrinsicSuccess)"
            ],
            "1": [
                "system(ExtrinsicSuccess)"
            ],
            "2": [
                "xassets(Move([80, 67, 88], d7d5ed3026fb60790e690b43f0dbd6858aabb824e6f78e03008f88a50cce09a6 (5UX2DeGC...), Free, a2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53 (5TJgYKHP...), Free, 11880))",
                "xfee_manager(FeeForProducer(a2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53 (5TJgYKHP...), 11880))",
                "xassets(Move([80, 67, 88], d7d5ed3026fb60790e690b43f0dbd6858aabb824e6f78e03008f88a50cce09a6 (5UX2DeGC...), Free, eacbf3fbb456f3227ace4450db99235021736165a5dd33da819aafd84b84f8c9 (5UwtAaDg...), Free, 106920))",
                "xfee_manager(FeeForJackpot(eacbf3fbb456f3227ace4450db99235021736165a5dd33da819aafd84b84f8c9 (5UwtAaDg...), 106920))",
                "xbitcoin(InsertHeader(536870912, 0xeecd86ec439a1748586a619e931ce17fcd8d0116e4e899055db95c2900000000, 1576451, 0x2664edaf0ed4a13a09ab650d7a7a2b0d2143c2663a10c5a5b9b4083f00000000, 0xc653f0fe33fa5c04034596cf638a21e4dd61fd3b7e1538eaf3d16a2d9ece64cc, 1567182098, 3675665054, 1576448, 0x28e629b0b8ceb6e9f71d1ad066a3d71ed3dec0eb54d2bd113287b00500000000))",
                "system(ExtrinsicSuccess)"
            ]
        }
```

**若提供了不存在的区块hash，会由于找不到对应的状态根而返回错误！**例：

```jsonc
{
    "jsonrpc": "2.0",
    "error": {
        "code": -32603,
        "message": "Unknown error occured",
        "data": "Error(Client(UnknownBlock(\"Unknown state for block Hash(0x99ed77b4737117e6920bc5c48c22c5e034ce96851790044b9f69f248fd1df3ff)\")), State { next_error: None, backtrace: InternalBacktrace { backtrace: None } })"
    },
    "id": 1
}
```

对于Event的返回值解释如下（原理与js sdk的解析相同，只是这里在节点内部将Event解析好以字符形式返回）：

例：

Event字符构成如下：

简单来说Event是归属于某个模块下的，因此首先有模块名，而在后续括号内部是event名

```bash
system(ExtrinsicSuccess)
模块名 (Event名)
```

稍微复杂的例子：

```bash
xassets(Move([80, 67, 88], d7d5ed3026fb60790e690b43f0dbd6858aabb824e6f78e03008f88a50cce09a6 (5UX2DeGC...), Free, eacbf3fbb456f3227ace4450db99235021736165a5dd33da819aafd84b84f8c9 (5UwtAaDg...), Free, 106920))
模块名xassets(Event名Move(参数1 [80, 67, 88]即ascii码PCX, 参数2 公钥1, 参数3 from类型, 参数4 公钥2，参数5 to类型，参数6 数值))
因此该Event语义为将PCX从公钥1的Free类型资产移动到公钥2的Free类型，数值为106920 (即转账的一直表述形式)
```

**另：由于大部分使用者只关心交易是否成功的`Event`，这条`Event`即为每条交易Event列表的最后一条Event（js sdk的逻辑相同）**

即 

* `system(ExtrinsicSuccess)`  表示该交易执行成功
* `system(ExtrinsicFailed)`   表示该交易执行失败

因此若使用者想通过该rpc获取交易执行是否成功，只需要获取对应Event列表的最后一条Event进行判断即可。

例上述例子中：

```jsonc
"events": {
    # ...
    "2": [
        "xassets()",
        "xfee_manager()",
        "xassets()",
        "xfee_manager()",
        "xbitcoin()",
        "system(ExtrinsicSuccess)"
    ]
}
```

在该例显示，这个块下的第3条交易（即index为2）的Events列表如上，则判断这笔交易是否成功，只需要检查这个列表的最后一项`system(ExtrinsicSuccess|ExtrinsicFailed)` 即可。

### 资产部分

#### chainx_getAssetsByAccount

获取用户资产信息

调用

* 方法名：`chainx_getAssetsByAccount`

* 参数：

  ```jsonc
  [
  	"0x................", # 需要查询的账户的公钥
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```jsonc
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

  ```jsonc
  [
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```jsonc
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

  ```jsonc
  [
  	"Bitcoin", # 链ID 枚举[ “Bitcoin”]
  	0, # 分页（从第几页开始）
  	100  # 分页（每页多少个数据）
  ]
  ```

返回

```jsonc
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

  ```jsonc
  [
  	"Bitcoin", # 链ID 枚举[“Bitcoin”]
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

  ```jsonc
  ["BTC" ] # 资产名称 目前支持的是 BTC 其他不支持
  ```

返回

```jsonc
{
	minimalWithdrawal: 150000,
	fee: 100000
}
```

#### chainx_getDepositLimitByToken

chainx 链上与充值金额相关的限制，如充值的最小值

调用

- 方法名：`chainx_getDepositLimitByToken`

- 参数：

  ```jsonc
  ["BTC" ] # 资产名称 目前支持的是 BTC 其他不支持
  ```

返回

```jsonc
{
	minimalDeposit: 100000,
}
```

#### chainx_getAddressByAccount

ChainX账户已绑定BTC地址列表，这里的地址列表表示充值X-BTC的地址列表，并非L-BTC的锁仓列表，锁仓列表只能通过api获取

调用

- 方法名：`chainx_getAddressByAccount`

- 参数：

  ```jsonc
  [
  	"0x..........", # 账户公钥
  	"Bitcoin" # 链ID 枚举[“Bitcoin”, “Ethereum”]， “Ethereum” 这里只是表示SDot的绑定
  ] 
  ```

返回

```jsonc
[
  "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7" # Bitcoin 地址列表
]
```

#### chainx_verifyAddressValidity

验证提现地址正确性

调用

- 方法名：`chainx_verifyAddressValidity`

- 参数：

  ```jsonc
  [
  	"BTC", # 币种名
  	"mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7", # 需要校验的提现地址
  	"memo" # 备注 （注：3个参数与使用js sdk 提起提现请求时候交易`withdraw` 的参数列表完全一致）
  ] 
  ```

返回

```jsonc
[
	true # 或者 false, 返回 true 代表地址校验正确，false代表错误
]
```

### 验证者部分

#### chainx_getStakingDividendByAccount

##### v1.0.6

获取用户投票利息

调用:

```jsonc
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getStakingDividendByAccount",
	"params":["0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53"] // 参数填写账户公钥
	
}
```

返回:

```:jsonc
{
    "jsonrpc": "2.0",
    "result": {
        "0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53": 7201153980 // key 为投票节点公钥，value 为用户对该节点的待领投票利息
    },
    "id": 1
}
```

#### chainx_getCrossMiningDividendByAccount

##### v1.0.6

获取用户跨链挖矿利息

调用：

```jsonc
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getCrossMiningDividendByAccount",
	"params":["0x4662b210b9ce571828cd3f4c27582c4af708be9121cc6ef4849fd21728df5d8b"]
	
}
```

返回：

```jsonc
{
    "jsonrpc": "2.0",
    "result": {
        "BTC": {
            "referral": 286930008, // 推荐人利息(注: 跨链挖矿用户提息时，总挖矿利息的10%归推荐人，90%归用户自己)
            "unclaimed": 2582370072 // 跨链挖矿用户的待领利息
        },
        "SDOT": {
            "referral": 0,
            "unclaimed": 0
        }
    },
    "id": 1
}
```

#### chainx_getNominationRecords

用户投票信息

调用

- 方法名：`chainx_getNominationRecords`

- 参数：

  ```jsonc
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

#### chainx_getNominationRecordsV1

##### v1.0.6

除了返回的字段 `lastVoteWeight` 类型为 String, 其他内容与 `chainx_getNominationRecords` 一样。

#### chainx_getNextRenominateByAccount

获取用户下次可切换投票的高度。

- 方法名: `chainx_getNextRenominateByAccount`

- 参数:

  ```jsonc
  [ "0x........." ] # 查询的账户公钥
  ```

返回:

```
{
    "jsonrpc": "2.0",
    "result": null,
    "id": 1
}
```

#### chainx_getIntentionByAccount

##### v1.0.6

获取单个节点信息

调用:

```jsonc
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getIntentionByAccount",
	"params":["0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53"] // 参数填写节点账户公钥
	
}
```

返回:

```jsonc
{
    "jsonrpc": "2.0",
    "result": {
        "about": "",
        "account": "0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53",
        "isActive": true,
        "isTrustee": [],
        "isValidator": true,
        "jackpot": 12601153980,
        "jackpotAccount": "0xeacbf3fbb456f3227ace4450db99235021736165a5dd33da819aafd84b84f8c9",
        "lastTotalVoteWeight": "0",
        "lastTotalVoteWeightUpdate": 0,
        "name": "Laocius",
        "selfVote": 1000000000,
        "sessionKey": "0x5917d1850e940bcd23254c73dcb936b730dd950ee6b750af02cc8caaedcd83ba",
        "totalNomination": 1000000000,
        "url": "polkadot.network"
    },
    "id": 1
}
```

`chainx_getIntentionByAccount` 为 `chainx_getIntentions` 的单元素版本, 详情参见 `chainx_getIntentions`。

#### chainx_getIntentionByAccountV1

##### v1.0.6

除了返回的字段 `lastTotalVoteWeight` 类型为 String, 其他内容与 `chainx_getIntentionByAccount` 一样。

#### chainx_getIntentions

获取节点列表

调用

- 方法名：`chainx_getNominationRecords`
- 参数：`[]` 无参数

返回

```jsonc
"result": [
  {
    "about": "", // 关于
    "account": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", // 账户 ID
    "isActive": true, // 是否参选
    "isTrustee": true, // 是否是信托节点
    "isValidator": true, // 是否是结算节点
    "jackpot": 387096776, // 奖池金额
    "jackpotAccount": "0xce153e3235448f29ca9052a660e36abd9b9fdc72f80a4059a2427ff06b1a3706", // 奖池公钥
    "lastTotalVoteWeight": 0, // 总票龄
    "lastTotalVoteWeightUpdate": 0, // 总票龄更新时间
    "name": "Alice", // 节点名
    "selfVote": 1250000000, // 节点自投票
    "sessionKey": "0xd172a74cda4c865912c32ba0a80a57ae69abae410e5ccb59dee84e2f4432db4f", // 节点出块公钥， 类型为 AccountId
    "totalNomination": 1250000000, // 节点总投票
    "url": "Alice.com" // 节点网址
  }
]
```

#### chainx_getIntentionsV1

##### v1.0.6

除了返回的字段 `lastTotalVoteWeight` 类型为 String, 其他内容与 `chainx_getIntentions` 一样。

#### chainx_getPseduIntentions

充值挖矿列表

调用

- 方法名：`chainx_getPseduIntentions`
- 参数：`[]` 无参数

返回

```jsonc
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

#### chainx_getPseduIntentionsV1

##### v1.0.6

除了返回的字段 `lastTotalDepositWeight` 类型为 String, 其他内容与 `chainx_getPseduIntentions` 一样。

#### chainx_getPseduNominationRecords

用户投票信息

调用

- 方法名：`chainx_getPseduNominationRecords`
- 参数：`["0x........."]` 查询用户的公钥

返回

```jsonc
"result": [
  {
    "balance": 1000000000, // 投票金额
    "id": "BTC", // 资产 ID
    "lastTotalDepositWeight": 0, // 票龄
    "lastTotalDepositWeightUpdate": 0 // 票龄更新高度
    "next_claim": 100, // 针对该资产的下次可提息高度
  }
]
```

#### chainx_getPseduNominationRecordsV1

##### v1.0.6

除了返回的字段 `lastTotalDepositWeight` 类型为 String, 其他内容与 `chainx_getPseduNominationRecords` 一样。

### 交易所部分

#### chainx_getTradingPairs

交易对列表

调用

- 方法名：`chainx_getTradingPairs`
- 参数：`[]` 无参数

返回

```jsonc
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

  ```jsonc
  [
  	0, # id`:`OrderPairID` 从`chainx_getTradingPairs`接口中返回的交易对ID
  	10, # 档 必须<=10
  ]
  ```

返回

```jsonc
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

  ```jsonc
  [
  	"0x..............", # 用户公钥
  	0, # 分页，起始页
  	100 # 分页，每页大小
  ]
  ```

返回

```jsonc
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

  ```jsonc
  [	"Bitcoin" ]  # 链id, 对于这个接口只有Bitcoin
  ```

返回

```jsonc
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

  ```jsonc
  [ 
  	[ # 需要模拟参与生成的信托候选的公钥，是个列表
  		"0x........",
  		"0x........",
  		# ..
      ]
  ] 
  ```

返回（注：返回值和`chainx_getTrusteeSessionInfo`一样，因为使用的相同的实现）

```jsonc
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

  ```jsonc
  [ "0x.........." ] # 用户的公钥
  ```

返回

```jsonc
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

  ```jsonc
  [ "Bitcoin" ] # 链id，当前只有Bitcoin
  ```

返回

```jsonc
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

### 手续费部分

该部分具体内容请参照[ChainX的手续费模型](手续费模型)

#### chainx_getFeeByCallAndLength

调用者提供**编码过**的交易方法（注意不是交易本体（交易由签名与交易方法组成）），与交易编码过原文的长度，可获取到该交易**加速前**的手续费大小（一倍加速手续费）

调用

- 方法名：`chainx_getFeeByCallAndLength`

- 参数：

  ```jsonc
  [ "0x111111111111111", 100 ] 
  ```

返回

```jsonc
{
	10000
}
```

#### chainx_getFeeWeightMap

##### v1.0.6

返回当前链上的手续费权重，基础手续费与字节手续费。手续费权重由`模块名 <空格> 交易方法名`组成

调用

- 方法名：`chainx_getFeeWeightMap`

- 参数：

  ```jsonc
  [  ] 
  ```

返回

```jsonc
{
"feeWeight": {
    "XAssets transfer": 1,
    "XAssetsProcess revoke_withdraw": 10,
    "XAssetsProcess withdraw": 3,
    # ....
    },
"transactionBaseFee": 10000,
"transactionByteFee": 100
}
```

### 合约部分

合约相关的RPC自v1.1.0版本开始提供，主要用于获取合约存储相关的调用。

#### chainx_contractCall

##### v1.1.0

从rpc直接调用一个合约中的方法。Substrate Contracts合约平台中的合约方法不区分是否可变，因此合约中的所有方法都可以从该接口触发。但由于对存储状态的修改只可以通过区块执行的方式，实际上**合约中会修改状态的方法虽然可以调用，但是不会生效，但是会有正常的返回值。**

因此该RPC接口可用于：

1. 合约中可以编写将合约中存储组织结构化返回的函数，通过该RPC去调用，获取合约中存储的数值。
2. 调试合约

调用

* 方法名：`chainx_contractCall`

* 参数：

  ```jsonc
  [
      {
          "dest": "0x4367ca48b692e35f459d8376cfa71ad5e10962e7e3be13cdc46efc1364a4845a",
          "gasLimit": 100000000,
          "inputData": "0xfe4a4425",
          "origin": "0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53"
      }
  ]
  ```

  * dest: 合约地址的公钥
  * gasLimit: origin 对应的账户需要付费的gasLimit，注意这笔**gas不会真的被花费掉**，而是只会用于提供检查是否能够在这个gasLimit的限制下执行这个合约的方法调用。因此在用于获取合约返回值时，这个值可以随便填写一个比较大的值（低于origin中PCX的Free类型的资产数目），在用于调试合约时，可以用来测试出对于执行该合约方法合适的gasLimit
  * inputdata: 合约方法的 selector 连上这个合约方法需要提供的参数的encode编码。即`selector+encode(params)`。selector可以从合约编译产生的`old_abi.json`或者`abi.json`中获取到，标示调用该合约的方法（类似以太坊合约中的函数签名）
  * origin：提供gasLimit的账户，该账户必须持有能够提供gasLimit的PCX。**PCX不会在调用该RPC的过程中被花费**

  或者可以加上块hash，在该块下执行相应合约方法

  ```jsonc
  [
  	{
  		// 参数如上
  	},
  	"0x121212121212121212121..."  // 区块hash
  ]
  ```

返回

```jsonc
{
    "data": "0x12341234",
    "status": 0
}
```

* data：调用该合约后的返回值，**该返回值是合约方法定义返回值的encode编码，即encode(返回值)**。(请注意当前Substrate中类似合约的返回值是encode(encode(返回值))，ChainX已在该接口返回部分做过一次decode)
* status：合约调用的状态码，一般情况下为0，若为其他值则是在合约调用中出现了异常

#### chainx_contractGetStorage

##### v1.1.0

通过合约的key直接获取对应的存储。该方法获取合约存储比较困难，因为key的值一般情况下需要通过计算才能获取。

调用

- 方法名：`chainx_contractGetStorage`

- 参数：

  ```jsonc
  [
      "0x121212121212121...",  // 合约地址的公钥
      "0x343434343434343...",  // 存储的key，是H256，即是32字节的Bytes。
  ]
  ```

  或者可以加上块hash，获取该块状态下的合约存储

  ```jsonc
  [
   	"0x121212121212121...",  // 合约地址的公钥
      "0x343434343434343...",  // 存储的key，是H256，即是32字节的Bytes。
  	"0x121212121212121212121..."  // 区块hash
  ]
  ```

返回

```jsonc
"0x123121212121212"  // 与`state_getstorage`的返回值类似，是encode的编码
```

#### chainx_contractXRCTokenInfo

##### v1.1.0

返回链上已经注册的币种对应的XRC代币合约信息，包含XRC20实例的地址以及selectors，将来可能会返回XRC777等信息。

调用

- 方法名：`chainx_contractXRCTokenInfo`

- 参数：

  ```jsonc
  [ ]
  ```

  或者可以加上块hash，获取该块状态下注册信息

  ```jsonc
  [	"0x121212121212121212121..."  // 区块hash ]
  ```

返回

```jsonc
"BTC": {
    "XRC20": {
        "address": "0x2ec0d10abb966aab967031003044329dad1dbd376e1a2a9d880aed4cbb205d6b",
        "selectors": {
            "BalanceOf": "0xe41dbb26",
            "Decimals": "0xe0446593",
            "Destroy": "0x18c9374b",
            "Issue": "0x9d64838c",
            "Name": "0xd0411301",
            "Symbol": "0x54e93e98",
            "TotalSupply": "0x38935d92"
        }
    }
}
```

#### chainx_contractXRC20Call

##### v1.1.0

对ChainX内集成的XRC20合约的调用封装。该RPC可以在不需要XRC20合约abi和地址的情况下，获取XRC20合约中的资产，代币信息等信息。该接口目前仅仅支持ChainX中的XRC20 -BTC。

调用

- 方法名：`chainx_contractXRC20Call`

- 参数：

  ```jsonc
  [
      {
          "token": "BTC",
          "selector": "BalanceOf", 
          "inputData": "0x88dc3417d5058ec4b4503e0c12ea1a0a89be200fe98922423d4334014fa6b0ee"
      }
  ]
  ```

  * token: **目前只支持 BTC**
  * selector: 针对XRC20的合约，selector支持以下参数：`[BalanceOf, TotalSupply, Name, Symbol, Decimal]`，即对应于`chainx_contractXRCTokenInfo`币种返回的XRC20信息中的selectors，与XRC20合约中的abi信息相对应，即合约中支持的部分方法。
  * inputData：根据selector含义对应的参数。注意如果类似`TotalSupply`没有参数的selector，inputData需要填写`"0x"`

  或者可以加上块hash，获取该块状态下相应的结果

  ```jsonc
  [
  	{
  		// 参数如上
  	},
  	"0x121212121212121212121..."  // 区块hash
  ]
  ```

返回：

该接口的返回值与`chainx_contractCall`类似，但是对应相应的selector已经做了解码：

```jsonc
{
    "data": 100000,
    "status": 0
}
```

其中：

* status 含义与`chainx_contractCall`的返回结果相同
* data：`[BalanceOf, TotalSupply, Decimal]` 返回数字，对于`[Name, Symbol]` 返回字符

### 其他

#### chainx_particularAccounts

ChainX链上的特殊公钥

调用

- 方法名：`chainx_particularAccounts`
- 参数：`[]` 无参数

返回

```jsonc
"result": {
    "councilAddress": "0x67df26a755e0c31ac81e2ed530d147d7f2b9a3f5a570619048c562b1ed00dfdd",
    "teamAddress": "0x6193a00c655f836f9d8a62ed407096381f02f8272ea3ea0df0fd66c08c53af81",
    "trusteesAddress": { # 当前信托多签公钥
        "Bitcoin": "0x8dc93cedb19de3341ca919160a58fe2497ed23b38169933f240007d1d703e2d5"
    }
}
```

