## 安装

```bash
yarn add chainx.js
```

或者

```bash
npm install chainx.js
```

## 快速开始

示例：

```javascript
const Chainx = require('chainx.js').default;

(async () => {
  // 目前只支持 websocket 链接
  const chainx = new Chainx('wss://w1.chainx.org.cn/ws');

  // 等待异步的初始化
  await chainx.isRpcReady();

  // 查询某个账户的资产情况
  const bobAssets = await chainx.asset.getAssets('5DtoAAhWgWSthkcj7JfDcF2fGKEWg91QmgMx37D6tFBAc6Qg');

  console.log('bobAssets:', JSON.stringify(bobAssets));

  // 构造交易参数（同步）

  const extrinsic = chainx.asset.transfer();

  // 查看 method 哈希
  console.log(extrinsic.method.toHex());

  // 签名并发送交易
  extrinsic.signAndSend('<账户私钥>', response => {
    if (response.status === 'Finalised') {
      if (response.result === 'ExtrinsicSuccess') {
        console.log('交易成功');
      }
    }
  });
})();
```

## Chainx.js

目前只接受 websocket 的方式，调用 rpc。使用前需要等待程序初始化完成：

```js
const chainx = new Chainx('wss://w1.chainx.org.cn/ws');

chainx.isRpcReady().then(() => {
  // ...
});

// 监听事件
chainx.on('disconnected', () => {}) // websocket 链接断开
chainx.on('error', () => {}) // 发生一个错误
chainx.on('connected', () => {}) // websocket 已连接
chainx.on('ready', () => {}) // 初始化完成
```





## 类型定义

几乎所有 api 函数的参数都对应一个类型定义。此时只要传入参数能被解析，即是合法的。例如：`accountId` 有很多种表现形式：

- `bs58` 格式 `"5Ey5ZRPVfGee7DVview3h3FkwMU6WTw2NZjYxD7o3NED7bZe"`
- `hex` 格式 `"0x806a491666670aa087e04770c025d64b2ecebfd91a74efdc4f4329642de32365"`
- `Buffer` 对象 `<Buffer 80 6a 49 16 66 67 0a a0 87 e0 47 70 c0 25 d6 4b 2e ce bf d9 1a 74 ef dc 4f 43 29 64 2d e3 23 65>`
- `Uint8Array` 对象 `Uint8Array [ 128, 106, 73, 22, 102, 103, 10, 160, 135, 224, 71, 112, 192, 37, 214, 75, 46, 206, 191, 217, 26, 116, 239, 220, 79, 67, 41, 100, 45, 227, 35, 101]`

以上均是合法的参数。

大部分类型都存在两个基本的方法：`toJSON()`，`toHex()`，`toJSON()` 用于将数据转化为用于 javascript 中的基础类型。而 `toHex()` 则一般是用于获取该数据编码后的二进制的十六进制的表示。在 `chainx.js` 中存在几种基本的类型，其余的类型基本上都是从它们中派生出来的。

-  [`UInt`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/UInt.js)： 在 chainx.js 内部，几乎所有的无符号数字都是继承于 `UInt` 类型，它是基于 `Bn.js` 实现的。常见的如 `Balance`，`U32`，`U64` 都是它的子类。

- [`Text`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Text.js)：文本类型，继承于原生的 `String` ，`toHex` 的时候按 utf8 格式编码。

- [`Enum`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/Enum.js)：枚举类型，如：

  ```js
  export default class TxType extends Enum {
    constructor(index) {
      super(['Withdraw', 'Deposit'], index);
    }
  }
  ```

  此时说明枚举值有 'Withdraw', 'Deposit'。同时`new TxType('Withdraw') `和 `new TxType(0) `得到的同样的值。

- [`Vector`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/Vector.js)：表现为一个数组的形式。如 `Vector.with(U32)` 则表示，由 `U32` 组成的数组。通过 `toJSON()`方法，可以得到一个数组。

- [`Struct`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/Struct.js)：表现为一个 Object 的形式。如：

- ```js
  export class EventMetadata extends Struct {
    constructor(value) {
      super(
        {
          name: Text,
          arguments: Vector.with(Type),
          documentation: Vector.with(Text),
        },
        value
      );
    }
  }
  ```

  `eventMetadata.toJSON() `将得到

  ```js
  {
    name: 'text',
    arguments: [type, type, ...],
    documentation: ['text', 'text', ...],
  }
  ```

  这样形式的 object

- [`Compact`](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/Compact.js) ：该类型通常与其它类型复合组成一个新的类型，此时内部数据编码是被压缩过的。如 `Compact.with(Nonce)`，将会压缩 `Nonce`的编码。

## Account 模块

通过`chainx.account.from(seed | privateKey | seed | mnemonic)` 可以生成一个 account 对象。

```js
const alice = chainx.account.from('0x....')
alice.address() // bs58 地址
alice.publicKey() // 公钥 0x...
alice.privateKey() // 私钥 0x...
```



## 交易函数

我们需要有几个步骤，来完成一次完整的交易。首先确定我们需要调用的 Method，如下列所示。以下 api 均返回一个 [SubmittableExtrinsic](https://github.com/chainx-org/chainx.js/blob/master/packages/chainx/src/SubmittableExtrinsic.js) 对象，它继承于 [Extrinsic](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Extrinsic.js) 。注意所有涉及到金额的部分，均是以最小的精度为单位。不存在小数。

[SubmittableExtrinsic](https://github.com/chainx-org/chainx.js/blob/master/packages/chainx/src/SubmittableExtrinsic.js) 存在几个比较重要的方法：

- `sign(account, { nonce, acceleration, blockHash })`：使用 account 签名该交易，其中 nonce 指该账户当前交易数。acceleration 指加速倍率，倍率越高手续费被扣的越多，同时交易也越快。blockHash 默认为当前链的初始哈希。

- `signAndSend(account, options?, callback?)`：使用 account 签名并发送该笔交易。acount 可以是账户的私钥，也可以是 account 对象。options 是可选的，和 sign 方法的参数一样，其中 nonce 默认自动从链上获取，acceleration 默认为 1。callback 是回调函数，第一个参数是 error，第二个参数是 result。对于一个正常的交易，callback 将会被调用若干次：

- ```js
  e.signAndSend('0x....', { acceleration: 10 }, (error, result) => {
    /**
    将输出三次
   	{
      "result": null,
      "index": null,
      "events": null,
      // 交易 hash
      "txHash": "0x4e93b7cae2868273a8a891684581564cf4431a81e90d2e6c7ee377b26648ac1d",
      "blockHash": null,
      "broadcast": null,
      // 交易状态
      "status": "Ready"
  	}
  	{
      "result": null,
      "index": null,
      "events": null,
      "txHash": "0x4e93b7cae2868273a8a891684581564cf4431a81e90d2e6c7ee377b26648ac1d",
      "blockHash": null,
      // 广播
      "broadcast": [
        "QmYqN9CKmx3qrNoaqXzDf7PiqvDssc1ALRYBLS65J6Z5wB",
        "QmZGSfxrgWP4Kv9VWmnnYKAs8WNAq1ooQ7kXaAxuS78LZp",
        "QmfNwbvXLbHsxPCmByimLxv1J7ZJpAjSfm82mSEnbobRkY",
        "QmUNmpe6UoSpE3LhkL9knqTogbPDn7Lsg1EQErPDkUJzgL",
        "QmSAwXWpAg45Zsp5X7U6hrAU5dFVacXifowjHbnC2VruRv",
        "QmXxCbLmSMTfFGWep7TNnvUfHSYwGiJ8AyMVPDwyWVAbWn",
        "QmVV2AJ3ju8iwBoE3zzf3tuadjAWyJEaR7338TcXjjcFfL",
        "Qmdfxi8As1jUxx9SrQJYCDgQYPKhiXtgYVvX8vr2RsgYD1",
        "QmPSVr8NRaZdUeNf3eAJfHgmV8WNtePgAWCAX5cU3H4qtn",
        "QmaQSTwbvcxRjs83qAQLW4zfpZ4BnkBHzPhxUE9e85zuUw",
        "QmP39VaaWPBYQ57JoZPYgz8yQg8khMX1LLuN4zwv7xZZEh"
      ],
      // 广播中
      "status": "Broadcast"
    }
  	{
  	// ExtrinsicSuccess 是交易成功。ExtrinsicFail 是交易失败
    "result": "ExtrinsicSuccess",
    "index": 2,
    // 关联的 events 列表
    "events": [{ "phase": 2, "event": [{...}] }, { "phase": 2, "event": [{...}] }, { "phase": 2, "event": [{...}] }],
    "txHash": "0x4e93b7cae2868273a8a891684581564cf4431a81e90d2e6c7ee377b26648ac1d",
    // 上链的块哈希
    "blockHash": "0x56482c31dff1a35db9fbac22be02ae294545a6440dda53f49e240e6df5f2460d",
    "broadcast": null,
    // 交易完成
    "status": "Finalised"
  }

    **/
    console.log(result);
  })
  ```



### chainx.asset.transfer([dest](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/AccountId.js), [token](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/AccountId.js), [value](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js), [memo](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js))

转账

#### 参数

- `dest`：接收人的地址
- `token`：转账的币种
- `value`：转账金额（最小的精度，只能为整数）
- `memo`：备注

#### 例子：

```javascript
chainx.asset.transfer('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 'PCX', 1000, '转给你');
```

### chainx.asset.withdraw([token](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Token.js), [value](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js), [addr](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/AddrStr.js), [ext](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Memo.js))

提现

#### 参数

- `token`：提现的资产类型
- `value`：提现的数量
- `addr`：接收的地址
- `ext`：备注

#### 例子：

```javascript
chainx.asset.withdraw('BTC', 100, '5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', '提现');
```

### chainx.asset.createWithdrawTx([withdrawalIdList](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/codec/Vector.js), [tx](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Bytes.js))

提交构造的提现交易原文

#### 参数

- `withdrawalIdList`：提现ID列表
- `tx`：构造的待签原文

#### 例子：

```javascript
chainx.asset.createWithdrawTx([1, 2], '0x......');
```

### chainx.asset.signWithdrawTx([tx?](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Bytes.js))

提交签名后的交易原文

#### 参数

- `tx`：签名后的交易原文

#### 例子：

```javascript
chainx.asset.signWithdrawTx('0x......');
```

### chainx.trade.putOrder([pair_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/TradingPairIndex.js), [order_type](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/OrderType.js), [order_direction](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/OrderDirection.js), [amount](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js), [price](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Price.js))

用户挂单

#### 参数

- `pairid`：交易对ID
- `ordertype`：类型（限价单|市价单)
- `direction`：方向（买单|卖单)
- `amount`：数量
- `price`：价格

#### 例子：

```javascript
chainx.trade.putOrder(1, 'Limit', 'Buy', 100, 100);
```

### chainx.trade.cancelOrder([pair_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/TradingPairIndex.js), [order_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/OrderIndex.js))

取消挂单

#### 参数

- `pairid`：交易对ID
- `index`：用户委托编号

#### 例子：

```javascript
chainx.trade.cancelOrder(0, 2);
```

### chainx.stake.register([name](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Name.js))

注册节点。

#### 参数

- `name`：注册节点的名称

#### 例子：

```javascript
chainx.stake.register('节点');
```

###

### chainx.stake.nominate([targetAddress](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js), [value](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js), [memo](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Memo.js))

对节点投票。

#### 参数

- `targetAddress`：被投票的节点的地址
- `value`: 投票金额
- `memo`: 备注

#### 例子：

```javascript
chainx.stake.nominate('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1000, '投票');
```

### chainx.stake.unnominate([targetAddress](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js),  [value](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Balance.js),  [memo](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Memo.js))

撤销对节点的投票。

#### 参数

- `targetAddress` ：撤票节点
- `value` ：撤票金额
- `memo`：备注

#### 例子

```javascript
chainx.stake.refresh('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1000, '撤销投票');
```

### chainx.stake.refresh([url?](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/URL.js),  [desire_to_run?](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Bool.js),  [next_key?](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/SessionKey.js),  [about?](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Text.js))

更新节点网址，参选状态，出块地址和关于。所有参数均是可选项，不传需要填写 `null` 作为占位。

#### 参数

- `url`: 节点网址
- `desireToRun`: 是否参选
- `nextKey`: 出块地址
- `about`: 关于

#### 例子

```js
chainx.stake.refresh(null, true, null, '我是节点');
```

### chainx.stake.voteClaim([target](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js))

选举投票提息。

#### 参数

- `target`: 投票节点

#### 例子

```js
chainx.stake.voteClaim('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ');
```

### chainx.stake.depositClaim([token](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Token.js))

充值挖矿提息。

#### 参数

- `token`: 充值币种

#### 例子

```js
chainx.stake.depositClaim('BTC');
```

### chainx.stake.unfreeze([target](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Address.js),  [revocation_index](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/U32.js))

撤销投票后，撤销金额有一定的时候锁定期。锁定期满后，可以对锁定金额进行解冻，使其变为可用余额。

#### 参数

- `target`: 解冻节点

- `revocationIndex`: 解冻索引。 用户对同一解冻节点可能同时有多次解冻操作，该索引指明解冻编号

#### 例子

```js
chainx.stake.unfreeze('5FxL27izsvhViiQtgwBm6kP8XvMSZ3JjyoMCmaw7pGrgXqqJ', 1);
```

### chainx.stake.setupTrustee([chain](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Chain.js),  [about](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/Text.js),  [hot_entity](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/TrusteeEntity.js), [cold_entity](https://github.com/chainx-org/chainx.js/blob/master/packages/types/src/TrusteeEntity.js))

设置信托节点信息。

#### 参数

- `chain`: 链 ID

- `about`: 关于

- `hotEntity`: 链的热地址

- `coldEntity`: 链的冷地址

#### 例子

```js
chainx.stake.setupTrustee('ChainX', 'about', ['Bitcoin', '0x......'], ['Bitcoin', '0x......']);
```