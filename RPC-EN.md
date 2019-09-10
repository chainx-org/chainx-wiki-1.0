ChainX's RPC system is derived from the substrate rpc crate, with custom `chainx` module added. The following document about `author`, `chain`, `state` and `system` are basically retrived from the original substrate rpc comments.


<!-- TOC GFM -->

* [author](#author)
    * [author_submitExtrinsic](#author_submitextrinsic)
    * [author_pendingExtrinsics](#author_pendingextrinsics)
    * [author_submitAndWatchExtrinsic](#author_submitandwatchextrinsic)
    * [author_unwatchExtrinsic](#author_unwatchextrinsic)
* [chain](#chain)
    * [chain_getHeader](#chain_getheader)
    * [chain_getBlock](#chain_getblock)
    * [chain_getBlockHash](#chain_getblockhash)
    * [chain_getFinalizedHead](#chain_getfinalizedhead)
    * [subscribe_newHead](#subscribe_newhead)
    * [unsubscribe_newHead](#unsubscribe_newhead)
    * [chain_subscribeFinalisedHeads](#chain_subscribefinalisedheads)
    * [chain_unsubscribeFinalisedHeads](#chain_unsubscribefinalisedheads)
* [state](#state)
    * [state_call](#state_call)
    * [state_getKeys](#state_getkeys)
    * [state_getStorage](#state_getstorage)
    * [state_getStorageHash](#state_getstoragehash)
    * [state_getStorageSize](#state_getstoragesize)
    * [state_getMetadata](#state_getmetadata)
    * [state_getRuntimeVersion](#state_getruntimeversion)
    * [state_queryStorage](#state_querystorage)
    * [chain_subscribeRuntimeVersion](#chain_subscriberuntimeversion)
    * [chain_unsubscribeRuntimeVersion](#chain_unsubscriberuntimeversion)
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
    * [chainx_getBlockByNumber](#chainx_getblockbynumber)
    * [chainx_getNextRenominateByAccount](#chainx_getnextrenominatebyaccount)
    * [chainx_getStakingDividendByAccount](#chainx_getstakingdividendbyaccount)
    * [chainx_getCrossMiningDividendByAccount](#chainx_getcrossminingdividendbyaccount)
    * [chainx_getAssetsByAccount](#chainx_getassetsbyaccount)
    * [chainx_getAssets](#chainx_getassets)
    * [chainx_verifyAddressValidity](#chainx_verifyaddressvalidity)
    * [chainx_getWithdrawalLimitByToken](#chainx_getwithdrawallimitbytoken)
    * [chainx_getDepositLimitByToken](#chainx_getdepositlimitbytoken)
    * [chainx_getDepositList](#chainx_getdepositlist)
    * [chainx_getWithdrawalList](#chainx_getwithdrawallist)
    * [chainx_getNominationRecords](#chainx_getnominationrecords)
    * [chainx_getNominationRecordsV1](#chainx_getnominationrecordsv1)
    * [chainx_getIntentions](#chainx_getintentions)
    * [chainx_getIntentionsV1](#chainx_getintentionsv1)
    * [chainx_getIntentionByAccount](#chainx_getintentionbyaccount)
    * [chainx_getIntentionByAccountV1](#chainx_getintentionbyaccountv1)
    * [chainx_getPseduIntentions](#chainx_getpseduintentions)
    * [chainx_getPseduIntentionsV1](#chainx_getpseduintentionsv1)
    * [chainx_getPseduNominationRecords](#chainx_getpsedunominationrecords)
    * [chainx_getPseduNominationRecordsV1](#chainx_getpsedunominationrecordsv1)
    * [chainx_getTradingPairs](#chainx_gettradingpairs)
    * [chainx_getQuotations](#chainx_getquotations)
    * [chainx_getOrders](#chainx_getorders)
    * [chainx_getAddressByAccount](#chainx_getaddressbyaccount)
    * [chainx_getTrusteeSessionInfo](#chainx_gettrusteesessioninfo)
    * [chainx_getTrusteeInfoByAccount](#chainx_gettrusteeinfobyaccount)
    * [chainx_getFeeByCallAndLength](#chainx_getfeebycallandlength)
    * [chainx_getFeeWeightMap](#chainx_getfeeweightmap)
    * [chainx_getWithdrawTx](#chainx_getwithdrawtx)
    * [chainx_getMockBitcoinNewTrustees](#chainx_getmockbitcoinnewtrustees)
    * [chainx_particularAccounts](#chainx_particularaccounts)

<!-- /TOC -->

## author

Substrate authoring RPC API, mainly used for submitting extrinsics.

#### author_submitExtrinsic
Submit hex-encoded extrinsic for inclusion in block.

#### author_pendingExtrinsics
Returns all pending extrinsics, potentially grouped by sender.

#### author_submitAndWatchExtrinsic
Submit an extrinsic to watch, for WebSocket.

#### author_unwatchExtrinsic
Unsubscribe from extrinsic watching, for WebSocket.

## chain

Substrate blockchain API, mainly used for accessing the primary data of blockchain.

#### chain_getHeader
Get header of a relay chain block.

#### chain_getBlock
Get header and body of a relay chain block.

```json
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chain_getBlock",
	"params":[]
	
}
```

```json
{
    "jsonrpc": "2.0",
    "result": {
        "block": {
            // all extrinsics in this blocks
            "extrinsics": [
                "0x20010100038e41775d",
                "0x8c01060031000d19a3e9607d92b3697a661a6e7e9fbb65361846680d968cfc86c9561103"
            ],
            // block header info
            "header": {
                "digest": {
                    "logs": [
                        "0x04617572612101c7a0bb2e00000000fea73789dee91d4a2fbb8d908256c8e379641e9c8694471868ea18ff47b4a82f6867e824721f5f382c6f9bf3d452d3259289d788c053c9a3da621a972d5dc20c"
                    ]
                },
                "extrinsicsRoot": "0x83b78e84c61c67a48f621e1bc43230db2e1bea9b84c93e425b6f471935289768",
                "number": "0x4688e7",
                "parentHash": "0x44fa98eade637edfcd381123730e91bfc14f5b7d5c63952db91d30ef2db74c18",
                "stateRoot": "0xc748bfbbe679d02aa9a84dd6e20a656f47596bd77297c689711974ee77857b2b"
            }
        },
        // this field is deprecated.
        "justification": null
    },
    "id": 1
}
```

#### chain_getBlockHash
Get hash of the n-th block in the canon chain.
By default returns latest block hash.

#### chain_getFinalizedHead
Get hash of the last finalized block in the canon chain.

#### subscribe_newHead
New head subscription, for WebSocket.

#### unsubscribe_newHead
Unsubscribe from new head subscription, for WebSocket.

#### chain_subscribeFinalisedHeads
New head subscription, for WebSocket.

#### chain_unsubscribeFinalisedHeads
Unsubscribe from new head subscription, for WebSocket.

## state

Substrate state API

#### state_call
Call a contract at a block's state.

#### state_getKeys
Returns the keys with prefix, leave empty to get all the keys.

**Notes: This API has been disabled in ChainX due to the safety reasons.**

#### state_getStorage
Returns a storage entry at a specific block's state.

#### state_getStorageHash
Returns the hash of a storage entry at a block's state.

#### state_getStorageSize
Returns the size of a storage entry at a block's state.

#### state_getMetadata
Returns the runtime metadata as an opaque blob.

#### state_getRuntimeVersion
Get the runtime version.

#### state_queryStorage
Query historical storage entries (by key) starting from a block given as the second parameter.

NOTE This first returned result contains the initial state of storage for all keys.
Subsequent values in the vector represent changes to the previous state (diffs).

#### chain_subscribeRuntimeVersion
New runtime version subscription, for WebSocket.

#### chain_unsubscribeRuntimeVersion
Unsubscribe from runtime version subscription, for WebSocket.

#### state_subscribeStorage
New storage subscription, for WebSocket.

#### state_unsubscribeStorage
Unsubscribe from storage subscription, for WebSocket.

## system

Substrate system RPC API.

#### system_name
Get the node's implementation name. Plain old string.

#### system_version
Get the node implementation's version. Should be a semver string.

#### system_chain
Get the chain's type. Given as a string identifier.

#### system_properties
Get a custom set of properties as a JSON object, defined in the chain spec.

#### system_health
Return health status of the node.

Node is considered healthy if it is:
- connected to some peers (unless running in dev mode)
- not performing a major sync

#### system_peers
Returns currently connected peers

#### system_networkState
Returns current state of the network.

## chainx

ChainX API

#### chainx_getBlockByNumber

##### Params

- optional: `block_number`

Get the block given the block number. Return the latest block if no `block_number` specified.

#### chainx_getNextRenominateByAccount

#### chainx_getStakingDividendByAccount

Get the staking dividend info given the account. Added since v1.0.6.

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getStakingDividendByAccount",
	"params":["0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53": 7201153980 // the key is the intention account that you're voting for，the value is the dividend.
    },
    "id": 1
}
```

#### chainx_getCrossMiningDividendByAccount

Get the cross mining dividend info given the account. Added since v1.0.6

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getCrossMiningDividendByAccount",
	"params":["0x4662b210b9ce571828cd3f4c27582c4af708be9121cc6ef4849fd21728df5d8b"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "BTC": {
            "referral": 286930008, // 10% of the total cross mining dividend will be rewarded to your referral when you perform the claim action.
            "unclaimed": 2582370072 // the rest 90% is for the cross miner.
        },
        ......
    },
    "id": 1
}
```


#### chainx_getAssetsByAccount

Get the asset info of the given account.

##### Params

1. accound id
2. page_index
3. page_size

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getAssetsByAccount",
	"params":["0xe25854dc8e9fa03f8fb8b9f290a5e783d294226acd2cfb0b38f0b4dc26ae4007", 0, 10]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "data": [
            {
                "details": {
                    "Free": 14650664456,
                    "ReservedCurrency": 0,
                    "ReservedDexFuture": 0,
                    "ReservedDexSpot": 0,
                    "ReservedStaking": 435375000000,
                    "ReservedStakingRevocation": 0,
                    "ReservedWithdrawal": 0
                },
                "name": "PCX"
            },
            {
                "details": {
                    "Free": 10112904,
                    "ReservedCurrency": 0,
                    "ReservedDexFuture": 0,
                    "ReservedDexSpot": 0,
                    "ReservedStaking": 0,
                    "ReservedStakingRevocation": 0,
                    "ReservedWithdrawal": 0
                },
                "name": "L-BTC"
            }
        ],
        "pageIndex": 0,
        "pageSize": 10,
        "pageTotal": 1
    },
    "id": 1
}
```

#### chainx_getAssets

Get global asset info.

##### Params

1. page_index
2. page_size

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getAssets",
	"params":[0, 100]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "data": [
            {
                "chain": "ChainX",
                "desc": "ChainX's crypto currency in Polkadot ecology",
                "details": {
                    "Free": 23459091736720,
                    "ReservedCurrency": 0,
                    "ReservedDexFuture": 0,
                    "ReservedDexSpot": 324402210637,
                    "ReservedStaking": 121503871194434,
                    "ReservedStakingRevocation": 8857634858209,
                    "ReservedWithdrawal": 0
                },
                "limitProps": {
                    "CanDeposit": false,
                    "CanDestroyFree": false,
                    "CanDestroyWithdrawal": false,
                    "CanMove": true,
                    "CanTransfer": true,
                    "CanWithdraw": false
                },
                "name": "PCX",
                "online": true,
                "precision": 8,
                "tokenName": "Polkadot ChainX"
            },
            ......
        ],
        "pageIndex": 0,
        "pageSize": 100,
        "pageTotal": 1
    },
    "id": 1
}
```

#### chainx_verifyAddressValidity

#### chainx_getWithdrawalLimitByToken

Get the withdrawal limit given the token type.

##### Params

1. token_type(`BTC`)

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getWithdrawalLimitByToken",
	"params":["BTC"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "fee": 100000,
        "minimalWithdrawal": 150000
    },
    "id": 1
}
```

#### chainx_getDepositLimitByToken

Get the deposit limit given the token type.

##### Params

1. token_type(`BTC`)

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getDepositLimitByToken",
	"params":["BTC"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "minimalDeposit": 100000
    },
    "id": 1
}
```

#### chainx_getDepositList

Get the current deposit info given the chain_id.

##### Params

1. chain_id(`Bitcoin`)
2. page_index
3. page_size

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getDepositList",
	"params":["Bitcoin", 0, 100]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "data": [
            {
              "accountid": "", // ChainX account
              "address": "mfioc8QgfcTVwuVd787JrdHMVjHACFWGX7", // BTC address
              "balance": 1000, // deposit amount
              "confirm": 1, // confirmation number
              "memo": "", // memo
              "time": 1547003949, // timestamp
              "token": "BTC", // token type
              "totalConfirm": 3, // total confirmation number
              "txid": "20bf6f637c0f05dbf2936cb0dbdd365ef1292e166aa7ed8af8d6c8418850d58e" // BTC txid
            },
            ......
        ],
        "pageIndex": 0,
        "pageSize": 100,
        "pageTotal": 0
    },
    "id": 1
}
```

#### chainx_getWithdrawalList

##### Params

1. chain_id(`ChainX`, `Bitcoin`, `Ethereum`, `Polkadot`)
2. page_index
3. page_size

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getWithdrawalList",
	"params":["Bitcoin", 0, 100]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "data": [
            {
                "accountid": "0x9b039dae8c888de7fcdbaa47b7879a166f5ecc6317f36c38db6c7b605af620f5",
                "address": "1GU7Y99BUi3oM9oPwQ7WjPnathLVJwUvsf",
                "applicationStatus": "Processing",
                "balance": 1739410000,
                "height": 4554356,
                "id": 538,
                "memo": "",
                "status": {
                    "value": "Broadcasting"
                },
                "token": "BTC",
                "txid": ""
            },
            ......
            }
        ],
        "pageIndex": 0,
        "pageSize": 100,
        "pageTotal": 1
    },
    "id": 1
}
```

#### chainx_getNominationRecords

Get nomination records of the given account.

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getNominationRecords",
	"params":["0x3f53e37c21e24df9cacc2ec69d010d144fe4dace6b2f087f466ade8b6b72278f"]
	
}
```

Response:

```
{
    "jsonrpc": "2.0",
    "result": [
        [
            "0x3f53e37c21e24df9cacc2ec69d010d144fe4dace6b2f087f466ade8b6b72278f",
            {
                "lastVoteWeight": 0,
                "lastVoteWeightUpdate": 0,
                "nomination": 1000000000,
                "revocations": []
            }
        ]
    ],
    "id": 1
}
```

#### chainx_getNominationRecordsV1

#### chainx_getIntentions

##### Params

none

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getIntentions",
	"params":[]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": [
        {
            "about": "Bridging Libra and ChainX",
            "account": "0xba6a4b3becacb3b7c5d3fabadcacf18c8ce2bc407ba88e14415e2589faf8c17a",
            "isActive": true,
            "isTrustee": [],
            "isValidator": false,
            "jackpot": 250377049,
            "jackpotAccount": "0xf1eef2c94c3e2aa9a5332745c9a2d82b90927e506ff11d0427caf05d7040cf3d",
            "lastTotalVoteWeight": 669595205587503,
            "lastTotalVoteWeightUpdate": 4590413,
            "name": "Libra",
            "selfVote": 4077000000,
            "sessionKey": "0xf7f2ed8da4cf97ad1665e28bf33da2d3d95a5b2c3f2b45a781d6e8034ab7dff4",
            "totalNomination": 36808106671,
            "url": "libra.org"
        }
    ],
    "id": 1
}
```

#### chainx_getIntentionsV1

#### chainx_getIntentionByAccount

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getIntentionByAccount",
	"params":["0xa2308187439ac204df9e299e1e54afefafea4bf348e03dad679737c91871dc53"] // 参数填写节点账户公钥
	
}
```

```
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

#### chainx_getIntentionByAccountV1

#### chainx_getPseduIntentions

##### Params

none

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getPseduIntentions",
	"params":[]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": [
        {
            "circulation": 53880391471,
            "discount": 10,
            "id": "BTC",
            "jackpot": 124520989759,
            "jackpotAccount": "0x6e97404385fde81240956d6a67cb59f07d12445438f0a28aa091c3f8a016e27a",
            "lastTotalDepositWeight": 13706990789661680,
            "lastTotalDepositWeightUpdate": 4622289,
            "power": 5092984514,
            "price": 273505769604
        },
        ......
        }
    ],
    "id": 1
}
```

#### chainx_getPseduIntentionsV1

#### chainx_getPseduNominationRecords

Get cross mining records of the given account.

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getPseduNominationRecords",
	"params":["0xe25854dc8e9fa03f8fb8b9f290a5e783d294226acd2cfb0b38f0b4dc26ae4007"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": [
        {
            "balance": 0,
            "id": "BTC",
            "lastTotalDepositWeight": 0,
            "lastTotalDepositWeightUpdate": 0,
            "nextClaim": 0
        },
        ......
        }
    ],
    "id": 1
}
```

#### chainx_getPseduNominationRecordsV1

#### chainx_getTradingPairs

##### Params

none

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getTradingPairs",
	"params":[]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": [
        {
            "assets": "PCX",
            "averPrice": 365623,
            "buyOne": 339700,
            "currency": "BTC",
            "id": 0,
            "lastPrice": 330700,
            "maximumBid": 350000,
            "minimumOffer": 329700,
            "online": true,
            "precision": 9,
            "sellOne": 340000,
            "unitPrecision": 2,
            "updateHeight": 4622603
        },
        ......
        }
    ],
    "id": 1
}
```

#### chainx_getQuotations

##### Params

1. trading pair ID
2. quotations level

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getQuotations",
	"params":[0, 10]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "buy": [
            [
                330000, // price
                6908078364 // amount
            ],
            [
                330300,
                37154414169
            ],
            [
                330700,
                12052631086
            ],
            [
                330800,
                35000000000
            ]
        ],
        "id": 0, // trading pair ID
        "piece": 10, // quotations level
        "sell": [
            [
                340000,
                60846
            ]
        ]
    },
    "id": 1
}
```

#### chainx_getOrders

Get open orders of the given account.

##### Params

1. account_id
2. page_index
3. page_size

##### Params

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getOrders",
	"params":["0xdd0c0033360e2ce0195dc722c706184b821ef6c3701a20d120d8e8edd683122e", 0, 100]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "data": [
            {
                "alreadyFilled": 0,
                "amount": 23750000000,
                "createdAt": 4625314,
                "executedIndices": [],
                "index": 1041,
                "lastUpdateAt": 4625314,
                "orderType": "Limit",
                "pairIndex": 0,
                "price": 330800,
                "remaining": 7856500,
                "side": "Buy",
                "status": "ZeroFill",
                "submitter": "0xdd0c0033360e2ce0195dc722c706184b821ef6c3701a20d120d8e8edd683122e"
            },
            ......
            }
        ],
        "pageIndex": 0,
        "pageSize": 100,
        "pageTotal": 1
    },
    "id": 1
}
```

#### chainx_getAddressByAccount

Get the binded address info given the account.

##### Params

1. account_id
2. chain_id(`Bitcoin`)

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getAddressByAccount",
	"params":["0xd62c4c8431ed2d371bc479eec0a4ab8028b6fc25d7c6f667d5170b6131bc28fe", "Bitcoin"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": [
        "14eLfYDr7jiS6DG1BSYEkFWW3WBmtbm96M"
    ],
    "id": 1
}
```

#### chainx_getTrusteeSessionInfo

Get info of current ChainX trustees, only appliable for `Bitcoin`.

##### Params

1. chain_id(`Bitcoin`)

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getTrusteeSessionInfo",
	"params":["Bitcoin"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "coldEntity": {
            "addr": "37DzisAW8DpyY2oqAfT3NfEwpG3SFVUUoR",
            "redeemScript": "0x542103615bee4a2f2e80605be8730dc9630b002ad83b068a902df03b155797357030f721037f8d0b44a282a89352b238b2d09f996df290aa65e0a95e6c99a445072ce390ce210281687791324c3d99d9bd39370baf4c138b1e1670a9939a406e3ac22577e39c00210386b58f51da9b37e59c40262153173bdb59d7e4e45b73994b99eec4d964ee7e88210299b5c30667f2e80ddccbac8d112e52387fa1056ef2510c0b7a627215eb0a45502102c179b0e69b342bf295200fa072bd2a4e956a2b74d7319c256946bc349c67d20956ae"
        },
        "counts": {
            "required": 4,
            "total": 6
        },
        "hotEntity": {
            "addr": "3LrrqZ2LtZxAcroVaYKgM6yDeRszV2sY1r",
            "redeemScript": "0x542102e2b2720a9e54617ba87fca287c3d7f9124154d30fa8dc9cd260b6b254e1d7aea210219fc860933a1362bc5e0a0bbe1b33a47aedf904765f4a85cd166ba1d767927ee2102b921cb319a14c6887b12cee457453f720e88808a735a578d6c57aba0c74e5af32102df92e88c4380778c9c48268460a124a8f4e7da883f80477deaa644ced486efc6210346aa7ade0b567b34182cacf9444deb44ee829e14705dc87175107dd09d5dbf4021034d3e7f87e69c6c71df6052b44f9ed99a3d811613140ebf09f8fdaf904a2e1de856ae"
        },
        "sessionNumber": 1,
        "trusteeList": [
            {
                "accountId": "0x80269f1c8712f25eb590fc849b89c79cc9b2309b2b2696e96d5610a08581b8aa",
                "props": {
                    "about": "",
                    "coldEntity": "0x03615bee4a2f2e80605be8730dc9630b002ad83b068a902df03b155797357030f7",
                    "hotEntity": "0x02e2b2720a9e54617ba87fca287c3d7f9124154d30fa8dc9cd260b6b254e1d7aea"
                }
            },
            ......
            }
        ]
    },
    "id": 1
}
```

#### chainx_getTrusteeInfoByAccount

Get trustee info of the given account.

##### Params

1. account_id

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getTrusteeInfoByAccount",
	"params":["0x80269f1c8712f25eb590fc849b89c79cc9b2309b2b2696e96d5610a08581b8aa"]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "Bitcoin": {
            "about": "",
            "coldEntity": "0x03615bee4a2f2e80605be8730dc9630b002ad83b068a902df03b155797357030f7",
            "hotEntity": "0x02e2b2720a9e54617ba87fca287c3d7f9124154d30fa8dc9cd260b6b254e1d7aea"
        }
    },
    "id": 1
}
```

#### chainx_getFeeByCallAndLength

#### chainx_getFeeWeightMap

Get the fee weight of all callable methods as well as the base fee and byte fee.

##### Params

none

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getFeeWeightMap",
	"params":[]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "feeWeight": {
            "XAssets transfer": 1,
            "XAssetsProcess revoke_withdraw": 10,
            "XAssetsProcess withdraw": 3,
            "XBridgeFeatures setup_bitcoin_trustee": 1000,
            "XBridgeOfBTC create_withdraw_tx": 5,
            "XBridgeOfBTC push_header": 10,
            "XBridgeOfBTC push_transaction": 50,
            "XBridgeOfBTC sign_withdraw_tx": 5,
            "XBridgeOfBTCLockup push_transaction": 50,
            "XBridgeOfSDOT claim": 2,
            "XFisher report_double_signer": 5,
            "XSpot cancel_order": 2,
            "XSpot put_order": 8,
            "XStaking claim": 3,
            "XStaking nominate": 5,
            "XStaking refresh": 10000,
            "XStaking register": 100000,
            "XStaking renominate": 800,
            "XStaking unfreeze": 2,
            "XStaking unnominate": 3,
            "XTokens claim": 3
        },
        "transactionBaseFee": 10000,
        "transactionByteFee": 100
    },
    "id": 1
}
```

#### chainx_getWithdrawTx

##### Params

1. chain_id("Bitcoin")

##### Example

#### chainx_getMockBitcoinNewTrustees

#### chainx_particularAccounts

Get special accounts in ChainX.

##### Params

none

##### Example

```
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_particularAccounts",
	"params":[]
	
}
```

```
{
    "jsonrpc": "2.0",
    "result": {
        "councilAccount": "0x67df26a755e0c31ac81e2ed530d147d7f2b9a3f5a570619048c562b1ed00dfdd",
        "teamAccount": "0x6193a00c655f836f9d8a62ed407096381f02f8272ea3ea0df0fd66c08c53af81",
        "trusteesAccount": {
            "Bitcoin": "0x79f4a1f074d0607ba964fe7dce5c5c896e35f983ea13d860041ae0fc394d7569" // current Bitcoin MultiSig address of ChainX Bitocin trustees
        }
    },
    "id": 1
}
```
