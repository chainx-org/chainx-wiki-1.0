
<!-- TOC GFM -->

* [Installation](#installation)
* [Get started](#get-started)
    * [Quick demo](#quick-demo)
    * [Sign Offline(http)](#sign-offlinehttp)
    * [Sign offline(websocket)](#sign-offlinewebsocket)
    * [Demo for exchanges](#demo-for-exchanges)
* [Chainx.js](#chainxjs)
* [Account module](#account-module)
* [Create account](#create-account)
* [Send extrinsics](#send-extrinsics)
    * [chainx.trustee.createWithdrawTx(withdrawalIdList, tx)](#chainxtrusteecreatewithdrawtxwithdrawalidlist-tx)
        * [Args](#args)
        * [Example](#example)
    * [chainx.trustee.signWithdrawTx(tx?)](#chainxtrusteesignwithdrawtxtx)
        * [Args](#args-1)
        * [Example](#example-1)
    * [chainx.trustee.setupBitcoinTrustee(about, hotEntity, coldEntity)](#chainxtrusteesetupbitcointrusteeabout-hotentity-coldentity)
        * [Args](#args-2)
        * [Example](#example-2)
    * [chainx.asset.transfer(dest, token, value, memo)](#chainxassettransferdest-token-value-memo)
        * [Args](#args-3)
        * [Example](#example-3)
    * [chainx.asset.withdraw(token, value, addr, ext)](#chainxassetwithdrawtoken-value-addr-ext)
        * [Args](#args-4)
        * [Example](#example-4)
    * [chainx.trade.putOrder(pair_index, order_type, order_direction, amount, price)](#chainxtradeputorderpair_index-order_type-order_direction-amount-price)
        * [Args](#args-5)
        * [Example](#example-5)
    * [chainx.trade.cancelOrder(pair_index, order_index)](#chainxtradecancelorderpair_index-order_index)
        * [Args](#args-6)
        * [Example](#example-6)
    * [chainx.stake.register(name)](#chainxstakeregistername)
        * [Args](#args-7)
        * [Example](#example-7)
    * [chainx.stake.nominate(targetAddress, value, memo)](#chainxstakenominatetargetaddress-value-memo)
        * [Args](#args-8)
        * [Example](#example-8)
    * [chainx.stake.unnominate(targetAddress, value, memo)](#chainxstakeunnominatetargetaddress-value-memo)
        * [Args](#args-9)
        * [Example](#example-9)
    * [chainx.stake.refresh(url?, desire_to_run?, next_key?, about?)](#chainxstakerefreshurl-desire_to_run-next_key-about)
        * [Args](#args-10)
        * [Example](#example-10)
    * [chainx.stake.voteClaim(target)](#chainxstakevoteclaimtarget)
        * [Args](#args-11)
        * [Example](#example-11)
    * [chainx.stake.depositClaim(token)](#chainxstakedepositclaimtoken)
        * [Args](#args-12)
        * [Example](#example-12)
    * [chainx.stake.unfreeze(target, revocation_index)](#chainxstakeunfreezetarget-revocation_index)
        * [Args](#args-13)
        * [Example](#example-13)
    * [chainx.stake.setupTrustee(chain, about, hot_entity, cold_entity)](#chainxstakesetuptrusteechain-about-hot_entity-cold_entity)
* [Getters](#getters)
    * [chainx.chain.getInfo()](#chainxchaingetinfo)
    * [chainx.chain.getBlockPeriod()](#chainxchaingetblockperiod)
    * [chainx.chain.subscribeNewHead()](#chainxchainsubscribenewhead)
    * [chainx.trustee.getTrusteeSessionInfo(chain)](#chainxtrusteegettrusteesessioninfochain)
    * [chainx.trustee.getTrusteeInfoByAccount(who)](#chainxtrusteegettrusteeinfobyaccountwho)
    * [chainx.trustee.getWithdrawTx(chain)](#chainxtrusteegetwithdrawtxchain)
    * [chainx.asset.getAssetsByAccount(who, page_index, page_size)](#chainxassetgetassetsbyaccountwho-page_index-page_size)
    * [chainx.asset.getAssets(page_index, page_size)](#chainxassetgetassetspage_index-page_size)
    * [chainx.asset.getWithdrawalList(chain, page_index, page_size)](#chainxassetgetwithdrawallistchain-page_index-page_size)
    * [chainx.asset.getWithdrawalListByAccount(who, page_index, page_size)](#chainxassetgetwithdrawallistbyaccountwho-page_index-page_size)
    * [chainx.asset.getDepositList(chain, page_index, page_size)](#chainxassetgetdepositlistchain-page_index-page_size)
    * [chainx.asset.getAddressByAccount(who, chain)](#chainxassetgetaddressbyaccountwho-chain)
    * [chainx.asset.verifyAddressValidity(token, addr, memo)](#chainxassetverifyaddressvaliditytoken-addr-memo)
    * [chainx.asset.getWithdrawalLimitByToken(token)](#chainxassetgetwithdrawallimitbytokentoken)
    * [chainx.stake.getNominationRecords(who)](#chainxstakegetnominationrecordswho)
    * [chainx.stake.getIntentions()](#chainxstakegetintentions)
    * [chainx.stake.getPseduIntentions()](#chainxstakegetpseduintentions)
    * [chainx.stake.getPseduNominationRecords(who)](#chainxstakegetpsedunominationrecordswho)
    * [chainx.stake.getBondingDuration()](#chainxstakegetbondingduration)
    * [chainx.stake.getIntentionBondingDuration()](#chainxstakegetintentionbondingduration)
    * [chainx.stake.getNextKeyFor(who)](#chainxstakegetnextkeyforwho)
    * [chainx.trade.getTradingPairs()](#chainxtradegettradingpairs)
    * [chainx.trade.getQuotations(id, piece)](#chainxtradegetquotationsid-piece)
    * [chainx.trade.getOrders(who, page_index, page_size)](#chainxtradegetorderswho-page_index-page_size)

<!-- /TOC -->

## Installation

```bash
# via yarn
yarn add chainx.js

# via npm
npm install chainx.js
```

## Get started

### Quick demo

```javascript
const Chainx = require('chainx.js').default;

(async () => {
  // Only support websocket for now
  const chainx = new Chainx('wss://w1.chainx.org/ws');

  // Wait for finishing the initialization of async
  await chainx.isRpcReady();

  // Get the balance info of some account
  const bobAssets = await chainx.asset.getAssetsByAccount('5DtoAAhWgWSthkcj7JfDcF2fGKEWg91QmgMx37D6tFBAc6Qg', 0, 10);

  console.log('bobAssets: ', JSON.stringify(bobAssets));

  // Build the extrinsic arguments (sync)
  const extrinsic = chainx.asset.transfer('5DtoAAhWgWSthkcj7JfDcF2fGKEWg91QmgMx37D6tFBAc6Qg', 'PCX', '1000', '转账 PCX');

  // Check out the hash of method
  console.log('Function: ', extrinsic.method.toHex());

  // Sign and send the transaction.
  // 0x0000000000000000000000000000000000000000000000000000000000000000 is the private key of signer.
  extrinsic.signAndSend('0x0000000000000000000000000000000000000000000000000000000000000000', (error, response) => {
    if (error) {
      console.log(error);
    } else if (response.status === 'Finalized') {
      if (response.result === 'ExtrinsicSuccess') {
        console.log('Transfer is successful');
      }
    }
  });
})();
```

### Sign Offline(http)

```javascript
const { ApiBase, HttpProvider } = require('chainx.js');
const nacl = require('tweetnacl');

// Sign the message using ed25519 algorithm.
async function ed25519Sign(message) {
  // 32 byte private key of the signer
  const privateKey = Buffer.from('5858582020202020202020202020202020202020202020202020202020202020', 'hex');

  // Sign the message using ed25519 algorithm.
  return nacl.sign.detached(message, nacl.sign.keyPair.fromSeed(privateKey).secretKey);
}

(async () => {
  const chainx = new ApiBase(new HttpProvider('https://w1.chainx.org/rpc'));

  await chainx._isReady;

  // Public key of the signer
  const address = '5CjVPmj6Bm3TVUwfY5ph3PE2daxPHp1G3YZQnrXc8GDjstwT';

  // Claim the staking dividend
  const extrinsic = chainx.tx.xStaking.claim(address);

  // Get nonce of the transactor
  const nonce = await chainx.query.system.accountNonce(address);

  // Get the message to be signed.
  // Args: extrinsic.encodeMessage(address, { nonce, acceleration = 1, blockHash = genesisHash, era = '0x00' })
  const encoded = extrinsic.encodeMessage(address, { nonce, acceleration: 10 });

  // Get the transaction fee.
  const fee = await extrinsic.getFee(address, { acceleration: 1 });

  console.log('fee', fee)

  // Sign the message offline
  const signature = await ed25519Sign(encoded);

  // Inject the signed signature
  extrinsic.appendSignature(signature);

  // Send the transaction
  const txHash = await extrinsic.send();

  console.log(txHash);
})();
```

### Sign offline(websocket)

```javascript
const Chainx = require('chainx.js').default;
const nacl = require('tweetnacl');

// Sign the message using ed25519 algorithm.
async function ed25519Sign(message) {
  // 32 byte private key of the signer
  const privateKey = Buffer.from('5858582020202020202020202020202020202020202020202020202020202020', 'hex');

  // Sign the message using ed25519 algorithm.
  // tweetnacl uses 64 byte secretKey, which is actually privateKey + publicKey.
  // You can get secretKey from privateKey via nacl.sign.keyPair.fromSeed.
  return nacl.sign.detached(message, nacl.sign.keyPair.fromSeed(privateKey).secretKey);
}

(async () => {
  // Only support websocket for now
  const chainx = new Chainx('wss://w1.chainx.org/ws');

  // Wait for finishing the initialization of async
  await chainx.isRpcReady();

  // Public key of the signer
  const address = '5CjVPmj6Bm3TVUwfY5ph3PE2daxPHp1G3YZQnrXc8GDjstwT';

  // Claim the staking dividend
  const extrinsic = chainx.stake.voteClaim(address);

  // Get nonce of the transactor
  const nonce = await extrinsic.getNonce(address);

  // Get the message to be signed.
  // Args: extrinsic.encodeMessage(address, { nonce, acceleration = 1, blockHash = genesisHash, era = '0x00' })
  const encoded = extrinsic.encodeMessage(address, { nonce, acceleration: 10 });

  // Sign offline
  const signature = await ed25519Sign(encoded);

  // Inject signature
  extrinsic.appendSignature(signature);

  // Check out the data sent to the chain.
  // console.log(extrinsic.toHex())

  // Send the transaction
  extrinsic.send((error, result) => {
    console.log(error, result);
  });
})();
```

### Demo for exchanges

Generally speaking, if you want to get the amount, sender and receiver of a certain PCX transfer, the workflow is:

1. Listen to the block height event to get the latest block height.
2. Get all the extrinsics in the latest block.
3. Parse every extrinsic to collect all the transfer extrinsics. If an extrinsic's method is `xAssets.transfer` and the last event is `Success`, then it's a successful transfer:
    1. Parse the arguments of transfer extrinsic to get sender, receiver(dest), amount, asset type and memo.
    2. If the receiver(dest) is the address you are watching at, then continue to handle your buiness.

```javascript
const { ApiBase, HttpProvider, WsProvider } = require('chainx.js');

(async () => {
  // Use http
  const api = new ApiBase(new HttpProvider('https://w1.chainx.org.cn/rpc'));
  // Use websocket
  // const api = new ApiBase(new WsProvider('wss://w1.chainx.org.cn/ws'))

  await api.isReady;

  const transferCallIndex = Buffer.from(api.tx.xAssets.transfer.callIndex).toString('hex');

  async function getTransfers(blockNumber) {
    const blockHash = await api.rpc.chain.getBlockHash(blockNumber);
    const block = await api.rpc.chain.getBlock(blockHash);
    const estrinsics = block.block.extrinsics;
    const transfers = [];

    for (let i = 0; i < estrinsics.length; i++) {
      const e = estrinsics[i];
      if (Buffer.from(e.method.callIndex).toString('hex') === transferCallIndex) {
        const allEvents = await api.query.system.events.at(blockHash);
        events = allEvents
          .filter(({ phase }) => phase.type === 'ApplyExtrinsic' && phase.value.eqn(i))
          .map(event => {
            const o = event.toJSON();
            o.method = event.event.data.method;
            return o;
          });
        result = events[events.length - 1].method;

        transfers.push({
          index: i,
          blockHash: blockHash.toHex(),
          blockNumber: blockNumber,
          result,
          tx: {
            signature: e.signature.toJSON(),
            method: e.method.toJSON(),
          },
          events: events,
          txHash: e.hash.toHex(),
        });
      }
    }

    return transfers;
  }

  const transfers = await getTransfers(237509);

  console.log(JSON.stringify(transfers));
})();
```

## Chainx.js

Currently you have to use the RPC functionality via websocket. Before using it, you'll need to wait for the initialization is done:

```javascript
const chainx = new Chainx('wss://w1.chainx.org.cn/ws');

chainx.isRpcReady().then(() => {
  // ...
});

// Listen to the events
chainx.on('disconnected', () => {}) // websocket disconnected
chainx.on('error', () => {}) // an error occurs
chainx.on('connected', () => {}) // websocket connected
chainx.on('ready', () => {}) // initialization is done

// Close the websocket connection.
chainx.provider.websocket.close()
```

During the initialization, chainx.js will fetch the network version automatically. The address format is different between the mainnet and testnet. Notes: the initialization is async, the network version is testnet by default before finishing the initialization. You can also specify the network version manually via `chainx.account.setNet('mainnet')` or `chainx.account.setNet('testnet')`.

## Account module

You can create an account object via `chainx.account.from(seed | privateKey | mnemonic)`.

```javascript
const alice = chainx.account.from('0x....')
alice.address() // base58 encoded address
alice.publicKey() // public key 0x...
alice.privateKey() // private key 0x...
```

## Create account

```javascript
const { Account } = require('chainx.js');

const account1 = Account.generate();

const publicKey1 = account1.publicKey();
console.log('publicKey1: ', publicKey1);

const privateKey1 = account1.privateKey();
console.log('privateKey1: ', privateKey1);

const address1 = account1.address();
console.log('address1: ', address1);

const mnemonic = Account.newMnemonic();
console.log('mnemonic: ', mnemonic);

const account2 = Account.from(mnemonic); // mnemonic -> account

const address2 = Account.encodeAddress(account2.publicKey()); // publicKey -> address
console.log('address2: ', address2);

const publicKey2 = Account.decodeAddress(address2); // address -> publicKey
console.log('publicKey2: ', publicKey2);

Account.setNet('testnet'); // Set the network vertion to testnet
const address3 = Account.encodeAddress(publicKey2); // address for testnet
console.log('address3:', address3);

Account.setNet('mainnet'); // Set the network version to mainnet
const address4 = Account.encodeAddress(publicKey2); // address for mainnet
console.log('address4:', address4);

const account3 = Account.from(privateKey1); // privateKey -> account
console.log('address:', account3.address()); // address
```

## Send extrinsics

### chainx.trustee.createWithdrawTx(withdrawalIdList, tx)

#### Args

#### Example

```javascript
chainx.trustee.signWithdrawTx('0x......');
```

### chainx.trustee.signWithdrawTx(tx?)

#### Args

#### Example

```javascript
chainx.trustee.setupBitcoinTrustee('111'， '0x......', '0x......');
```

### chainx.trustee.setupBitcoinTrustee(about, hotEntity, coldEntity)

#### Args

#### Example

```javascript
chainx.trustee.setupBitcoinTrustee('111'， '0x......', '0x......');
```

### chainx.asset.transfer(dest, token, value, memo)

Transfer asset.

#### Args

#### Example

```javascript
chainx.asset.transfer('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 'PCX', 1000, '转给你');
```

### chainx.asset.withdraw(token, value, addr, ext)

#### Args

#### Example

### chainx.trade.putOrder(pair_index, order_type, order_direction, amount, price)

New order.

#### Args

1. `pair_index`: trading pair id.
2. `order_type`: Only `Limit` for now.
3. `order_direction`: `Buy` or `Sell`.
4. `amount`
5. `price`

#### Example

```javascript
chainx.trade.putOrder(1, 'Limit', 'Buy', 100, 100);
```

### chainx.trade.cancelOrder(pair_index, order_index)

Cancel the open order.

#### Args

1. `pair_index`: trading pair id.
2. `order_index`: user oder index.

#### Example

```javascript
chainx.trade.cancelOrder(0, 2);
```

### chainx.stake.register(name)

#### Args

1. `name`: intention name

#### Example

```javascript
chainx.stake.register('节点');
```

### chainx.stake.nominate(targetAddress, value, memo)

#### Args

#### Example

```javascript
chainx.stake.nominate('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1000, 'Nominate');
```

### chainx.stake.unnominate(targetAddress, value, memo)

#### Args

#### Example

```javascript
chainx.stake.refresh('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1000, 'Unnominate');
```

### chainx.stake.refresh(url?, desire_to_run?, next_key?, about?)

#### Args
1. `url`: your domain name.
2. `desire_to_run`: if the transactor want to participate in the validators election.
3. `next_key`: address for block authoring.
4. `about`: brief intention introduction.

#### Example

```javascript
chainx.stake.refresh(null, true, null, 'intention about');
```

### chainx.stake.voteClaim(target)
Claim the staking dividend.

#### Args

1. `target`: the intention voted

#### Example

```javascript
chainx.stake.voteClaim('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ');
```

### chainx.stake.depositClaim(token)
Claim the cross mining dividend.

#### Args

1. `token`: token type

#### Example

```javascript
chainx.stake.depositClaim('BTC');
```

### chainx.stake.unfreeze(target, revocation_index)

There is a lock period after you perform an `unnominate` action. When the locked balance is due, you can unfreeze it to make it be the free balance.

#### Args

1. `target`: intention account
2. `revocation_index`: user can have serveral unfreeze items, this index is used for specifing which one to unfreeze.

#### Example

```javascript
chainx.stake.unfreeze('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1);
```

### chainx.stake.setupTrustee(chain, about, hot_entity, cold_entity)

## Getters

### chainx.chain.getInfo()
### chainx.chain.getBlockPeriod()
### chainx.chain.subscribeNewHead()
### chainx.trustee.getTrusteeSessionInfo(chain)
### chainx.trustee.getTrusteeInfoByAccount(who)
### chainx.trustee.getWithdrawTx(chain)
### chainx.asset.getAssetsByAccount(who, page_index, page_size)
### chainx.asset.getAssets(page_index, page_size)
### chainx.asset.getWithdrawalList(chain, page_index, page_size)
### chainx.asset.getWithdrawalListByAccount(who, page_index, page_size)
### chainx.asset.getDepositList(chain, page_index, page_size)
### chainx.asset.getAddressByAccount(who, chain)
### chainx.asset.verifyAddressValidity(token, addr, memo)
### chainx.asset.getWithdrawalLimitByToken(token)
### chainx.stake.getNominationRecords(who)
### chainx.stake.getIntentions()
### chainx.stake.getPseduIntentions()
### chainx.stake.getPseduNominationRecords(who)
### chainx.stake.getBondingDuration()
### chainx.stake.getIntentionBondingDuration()
### chainx.stake.getNextKeyFor(who)
### chainx.trade.getTradingPairs()
### chainx.trade.getQuotations(id, piece)
### chainx.trade.getOrders(who, page_index, page_size)
