ChainX's RPC system is derived from the substrate rpc crate, with custom `chainx` module added. The following document about `author`, `chain`, `state` and `system` are basically retrived from the original substrate rpc comments.

## author

Substrate authoring RPC API, mainly used for submitting extrinsics.

### `author_submitExtrinsic`
Submit hex-encoded extrinsic for inclusion in block.

### `author_pendingExtrinsics`
Returns all pending extrinsics, potentially grouped by sender.

### `author_submitAndWatchExtrinsic`
Submit an extrinsic to watch, for WebSocket.

### `author_unwatchExtrinsic`
Unsubscribe from extrinsic watching, for WebSocket.

## chain

Substrate blockchain API, mainly used for accessing the primary data of blockchain.

### `chain_getHeader`
Get header of a relay chain block.

### `chain_getBlock`
Get header and body of a relay chain block.

### `chain_getBlockHash`
Get hash of the n-th block in the canon chain.
By default returns latest block hash.

### `chain_getFinalizedHead`
Get hash of the last finalized block in the canon chain.

### `subscribe_newHead`
New head subscription, for WebSocket.

### `unsubscribe_newHead`
Unsubscribe from new head subscription, for WebSocket.

### `chain_subscribeFinalisedHeads`
New head subscription, for WebSocket.

### `chain_unsubscribeFinalisedHeads`
Unsubscribe from new head subscription, for WebSocket.

## state

Substrate state API

### `state_call`
Call a contract at a block's state.

### `state_getKeys`
Returns the keys with prefix, leave empty to get all the keys.

**Notes: This API has been disabled in ChainX due to the safety reasons.**

### `state_getStorage`
Returns a storage entry at a specific block's state.

### `state_getStorageHash`
Returns the hash of a storage entry at a block's state.

### `state_getStorageSize`
Returns the size of a storage entry at a block's state.

### `state_getMetadata`
Returns the runtime metadata as an opaque blob.

### `state_getRuntimeVersion`
Get the runtime version.

### `state_queryStorage`
Query historical storage entries (by key) starting from a block given as the second parameter.

NOTE This first returned result contains the initial state of storage for all keys.
Subsequent values in the vector represent changes to the previous state (diffs).

### `chain_subscribeRuntimeVersion`
New runtime version subscription, for WebSocket.

### `chain_unsubscribeRuntimeVersion`
Unsubscribe from runtime version subscription, for WebSocket.

### `state_subscribeStorage`
New storage subscription, for WebSocket.

### `state_unsubscribeStorage`
Unsubscribe from storage subscription, for WebSocket.

## system

Substrate system RPC API.

### `system_name`
Get the node's implementation name. Plain old string.

### `system_version`
Get the node implementation's version. Should be a semver string.

### `system_chain`
Get the chain's type. Given as a string identifier.

### `system_properties`
Get a custom set of properties as a JSON object, defined in the chain spec.

### `system_health`
Return health status of the node.

Node is considered healthy if it is:
- connected to some peers (unless running in dev mode)
- not performing a major sync

### `system_peers`
Returns currently connected peers

### `system_networkState`
Returns current state of the network.

## chainx

ChainX API

### `chainx_getBlockByNumber`

Returns the block of a storage entry at a block's Number.

### `chainx_getNextRenominateByAccount`

### `chainx_getStakingDividendByAccount`

### `chainx_getCrossMiningDividendByAccount`

### `chainx_getAssetsByAccount`

### `chainx_getAssets`

### `chainx_verifyAddressValidity`

### `chainx_getWithdrawalLimitByToken`

### `chainx_getDepositLimitByToken`

### `chainx_getDepositList`

### `chainx_getWithdrawalList`

### `chainx_getNominationRecords`

### `chainx_getNominationRecordsV1`

### `chainx_getIntentions`

### `chainx_getIntentionsV1`

### `chainx_getIntentionByAccount`

### `chainx_getIntentionByAccountV1`

### `chainx_getPseduIntentions`

### `chainx_getPseduIntentionsV1`

### `chainx_getPseduNominationRecords`

### `chainx_getPseduNominationRecordsV1`

### `chainx_getTradingPairs`

### `chainx_getQuotations`

### `chainx_getOrders`

### `chainx_getAddressByAccount`

### `chainx_getTrusteeSessionInfo`

### `chainx_getTrusteeInfoByAccount`

### `chainx_getFeeByCallAndLength`

### `chainx_getFeeWeightMap`

### `chainx_getWithdrawTx`

### `chainx_getMockBitcoinNewTrustees`

### `chainx_particularAccounts`
