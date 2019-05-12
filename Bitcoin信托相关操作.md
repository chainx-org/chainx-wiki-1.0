## Bitcoin信托相关操作

### Overview

Bitcoin信托的主要职责有2个：

* 对于Bitcoin而言：根据当前届Bitcoin信托在链上注册的公钥信息**生成并控制一个热多签地址及一个冷多签地址**

  * 进行 Bitcoin 多签流程：

    * ChainX的多签提现：使用ChainX钱包，或根据该多签流程使用sdk编写自己的工具
    * 链外多签转账：使用ChainX准备的工具[chainx-multisig-verify-script](https://github.com/chainx-org/chainx-multisig-verify-script)，强烈建议节点设计符合自己需求的Bitcoin多签工具

  * Bitcoin多签地址的特殊性：

    * 由于Bitcoin脚本的限制，实际上一个多签地址对应着一个`redeemScript`，在执行多签流程时需要提供该地址对应的`redeemScript`才可进行签署，例如：

    ```json
    {
          "address": "2N24ytjE3MtkMpYWo8LrTfnkbpyQQQQQQQQ", 
          "redeemScript": "0x532102a79800dfed17ad4c78c52797adafasdfasd69421080f42d27790ee2103eceasdfasdfasda3e62ef6b2f6ad9774489e9aff1c8bc684d87dfdasfasdf2dd162e8d8614a4afbb8e2eb14eddf4036042b35asdfasdfd8ac4d3e055eae7551427487e281e3efba618bdd395f2f54ae"
    }
    ```

    * `redeemScript`在生成多签地址时同时生成，ChainX链上记录了热冷多签地址对应的`redeemScript`，可通过相应的rpc接口得到。

* 对于ChainX而言：ChainX将针对当前届的信托生成一个对应的ChainX多签地址，用于控制Bitcoin信托比如设置Bitcoin提现手续费，Bitcoin信托换届等功能

  * 进行ChainX多签流程，参考[多签模块](多签模块)，使用ChainX提供的`multisig-tool`工具

其他：

* 对于Bitcoin的多签地址，实际上由每个人的**公钥**（注意非地址）生成，因此下文的操作中请区分清楚Bitcoin下列概念的区别：
  * 公钥
  * 多签地址

### 设置Bitcoin信托节点

1. 要想成为信托节点首先需要注册成验证节点，然后再设置信托。如果已经是验证节点可以跳过此步。
2. 点击资产信托->设置信托 选择Bitcoin,设置自己BTC地址对应的热公钥，冷公钥。

  eg: ![设置信托](https://github.com/chainx-org/images/blob/master/t1.png)

​      ![设置信托](https://github.com/chainx-org/images/blob/master/t2.png)
​	信托换届时会根据当选信托的节点设置的热公钥、冷公钥分别生成热多签地址和冷多签地址

5. 点击信托设置，设置自己信任的BTC节点地址（当前为BTC测试网）

​      ![设置BTC节点地址](https://github.com/chainx-org/images/blob/master/t3.png)
​	ChainX搭建的BTC测试网节点地址为　(群内提供)
​    

### 信托节点处理提现申请

**节点处理提现本质上是使用ChainX钱包的信托部分或信托自己组建了一个Bitcoin多签提现，该多签提现称为“待签原文”，将该原文在其他信托节点之间转移并签名后，成为能够执行的多签提现交易，并广播到链上**

因此首先定义如下两个概念：

1. **ChainX的多签提现**：当用户提现时，需要有一个信托根据用户的提现**自己组装多签提现的原文**，并通过ChainX的交易发送到ChainX链上，其他信托获取链上的多签提现原文并使用自己的私钥进行签名，然后再通过ChainX的交易发送到链上，直至达成满足条件的签名，之后**Bitcoin的Relay会自动将这笔交易广播到Bitcoin网络，无需节点广播交易**
2. **链外多签转账**：与ChainX的处理流程无关，用于冷热地址互转，测试下一届多签地址有效性等等。**由于目前市面上几乎没有开源的多签钱包，因此信托需要准备自己的多签工具。**（ChainX仅准备一个简单的工具[chainx-multisig-verify-script](https://github.com/chainx-org/chainx-multisig-verify-script)以供参考与测试）

这个提现过程请注意以下本质：

无论使用ChainX钱包的信托部分多签提现，或者是节点使用sdk编写多签提现申请，从当前届的“热地址”的提现（从“热地址”转移资金出来）时，只能执行以下两种过程：

1. 执行**ChainX的多签提现**流程，该流程保证信托组建的提现交易与用户提现的申请完全对应
2. 执行**链外多签转账**流程，该流程**只能从“热地址”转移到“冷地址”**，不可执行其他转移操作

若出现从“热地址”既没有走用户提现组装流程与热冷转换的交易被广播，则会被ChainX识别为“出现了不可预期的资金转移（相当于有一笔钱在不可知其目的的情况下出现了转移）”

**这种情况会触发链上提现异常的开关，导致Bitcoin跨链相关业务暂停。** 这时需要“议会”介入，排查问题，弄清楚这笔资金转移的目的后才可由“议会”修复相应的存储，打开对应的开关。

#### ChainX 提现流程描述（用于节点使用sdk编写）

Bitcoin 提现流程如下

![](https://github.com/chainx-org/images/blob/master/bitcoin_withdrawal.png)

1. 用户申请提现：

   * 用户申请提现后，在ChainX链上会生成一个**唯一**的提现申请序列号，叫做`withdrawal_id`

2. 信托处理

   信托通过rpc接口`chainx.asset.getWithdrawTx(chain)`转入`Bitcoin`监听链上的`WithdrawalProposal`是否存在：

   1. 不存在：

      * **任意一个**信托通过rpc接口`chainx.asset.getWithdrawalList(chain, page_index, page_size)`传入`Bitcoin`获取当前的用户提现列表
      * 信托根据用户提现列表的相应信息组件提现**多签待签原文**，组件方式如下：
        * 通过rpc接口`chainx.asset.getMinimalWithdrawalValueByToken(token)`，填写参数`BTC`（注意不是Bitcoin，因为这个是针对币不是针对链）获取当前ChainX上对用户提现Bitcoin收取的手续费`fee`
        * **挑选**这次可以提现的用户提现记录（最多有一个上限，防止Output过大，当前最大不超过100），获取用户提现的`value`与提现地址`addr`，组建提现交易的`Outputs`，
          * 用户提现的`output`：**`Output`中的的value为`value - fee`**(value是用户提现记录中的值，fee是向用户手续的提现手续费，也就是最后到用户账上的钱为他申请的值扣除手续费)。

          * 找零的`output`：组件这笔交易后，根据交易长度，获取当前Bitcoin网络中的手续费率，**调整稍微高**一些后作为矿工费，余下作为找零`Output`中的value。

            **注意**：由于用户提现本来就扣除了一笔手续费，因此组建Output后，手续费是十分充裕的。由于ChainX每1小时才能提起一笔提现交易，那么一笔交易的output会很多，因此为了能尽快打包，我们强烈建议组建交易的信托多拿一些手续费出来付给Bitcoin矿工。 手续费的大小可参考目前公开的Bitcoin手续费比例，调高5-10倍后，根据当前的待签原文的长度计算应付的手续费。

            例如：一段时间内有3个用户提现，3个用户都扣除手续费，总共` 3 × fee`，然后组建一笔多签提现交易提出来，可能实际上付给矿工的只有`1 fee`，那么最后实际上会留下`2fee`在多签地址里。因此在多签地址中的钱会累积的越来越多，手续费不会出现短缺的现象。
        * 从**Bitcoin全节点**或一些**公开可信任的服务**根据当前的多签地址获取合适的UTXO，与上文中的`outputs`共同组成**多签待签原文**
      * 信托调用sdk将`chainx.asset.createWithdrawTx(withdrawalIdList, tx)`，将构造**多签待签原文**的用户提现的提现的`withdrawal_id`列表与多签待签原文作为参数传入
      * 若发送成功，则链上就会存在`WithdrawalProposal`，且`sig_state`为`UnFinish`

   2. 存在：

      检查`signStatus`的值

      1. 如果`signStatus`为`false`，即`WithdrawalProposal`中的`state`为`UnFinish`

         1. **所有信托**根据`getWithdrawTx`返回值，进行检查
         2. 其中`tx`即为**多签待签原文（或其他信托已签名过的交易）**，信托在这个tx的基础上使用自己的Bitcoin多签工具对这笔tx继续签署自己的信息
         3. 信托通过`chainx.asset.signWithdrawTx(tx)`将自己签名后的原文发送到链上（注意发送前先检查链上的原文是否有更新（或订阅发现更新），防止没有对最新的tx进行签署）
         4. 若``chainx.asset.signWithdrawTx()`不填写任何参数，**代表当前的信托对这笔提现投出反对**
            * 手续费觉得不合理
            * 提现列表的output觉得不合理
            * ...

      2. 若`signStatues`为`true`

         1. 信托不可进行任何操作，等待这笔tx被relay提交上Bitcoin即可

            1. 正常情况下这笔交易会正常上链，并被ChainX所识别，若这笔交易完全正常合法，则在ChainX上被确认后（1个小时），会自动清空ChainX链上的`WithdrawalProposal`存储，此时这笔提现流程执行结束，可重新进入信托提现处理流程。

            2. 非正常情况：

               1. 等待长时间未上链，此时唯一原因为创建这笔多签提现的手续费不足或交易非法，
               2. 上链了，但是在确认是认为这笔提现非法

               此时只能“议会”接入，移除链上的`WithdrawalProposal`并修复相应记录，之后才可重新进入信托提现处理流程。信托节点只能处于等待，无法进行其他操作。

3. 其他注意事项
   * 由于Bitcoin的签署必须在上一个已签署过的原文上继续签名（Bitcoin的技术限制），虽然链上已经做好防御，但是若信托节点之间**竞争式**提交提现交易的创建与对提现交易的签名，会让信托节点之间损失无畏的手续费，且给链上增加了无用的拥堵。因此对于Bitcoin而言，ChainX建议信托节点之间线下协商好创建与签名交易的顺序，免去无谓的竞争。如可以采取以下策略：
      * 节点可以首先做一个排序，然后按顺序创建提现交易：如上一次提现由序号1创建，这一次提现由序号2创建
      * 节点对提现交易的签名也可以按照排序进行，通过rpc获取到已签名数与自己序号相等时发送自己的签名
      * 其他策略...

信托节点可通过sdk编写相应的处理方式，根据自己拟定的策略自动化处理，如：

- 小额自动构建交易与自动签名确认
- 大额需要人工确认

#### ChainX钱包的信托部分操作方式

以下是ChainX已经提供好的信托可操作的提现流程。该流程目前只能人工操作。

1. 首先要导入BTC热私钥，输入自己的热私钥，并设置密码。如果已经导入过，不必重复导入（私钥只会保存到本地）

  ![导入BTC私钥](https://github.com/chainx-org/images/blob/master/import-privatekey1.png)

  ![导入BTC私钥](https://github.com/chainx-org/images/blob/master/import-privatekey2.png)  

2. 处理提现

   1. 如果提现列表中有状态为‘申请中’的提现申请，已经成为信托节点并且导入过btc热私钥的信托节点就可以点击‘构造多签提现’，在弹出窗的体现列表中选择提现ID即可生成提现交易原文，点击‘确定’，提现交易原文就会被发送到ChianX链上，此时对应的申请状态会由‘申请中’ 变为 ‘处理中’。然后接下来所有信托节点可以对构造的交易进行签名。

     ![构造交易](https://github.com/chainx-org/images/blob/master/create-withdraw-tx.png)

      **注意** 有时会出现异常情况

   - 节点未连接：请点击设置节点检查所配置的btc节点地址是否正确，或者本机与btc节点是否连通
   - 没有可用的UTXO：请检查当前多签地址余额是否充足，或者检查btc节点的钱包是否将当前多签地址导入并扫描过UTXO ，导入命令参考eg:

   ```shell
      ./bitcoin-cli -rpcport=8180 -rpcuser=123 -testnet importaddress 2N6mJFLkjN9muneSeHCsMCxWXVZ4ruLKfFo "" true
   ```

   2. 如果当前页面显示有‘响应多签提现’的按钮，则需要点击‘响应多签提现’进行签名或否决，点击‘确定’，签名或否决后的结果会被发送至ChainX链上，如果提现交易签名数量达到2/3，则这一笔提现交易会立即被relay程序广播到比特币网络中去。如果否决数量达到2/3，则链上会立即删除当前提现交易原文，信托节点可重新构造提现交易。

   3. 以上创建和签名之前，节点可以点击“复制待签原文”，并将原文decode，检查output的构建以及交易费的设置是否正确，若不正确，需要节点投出“否决”代表自己否定这笔提现交易。原文decode可通过该链接 [https://live.blockcypher.com/btc/decodetx/](https://live.blockcypher.com/btc/decodetx/)

### Bitcoin信托换届的Bitcoin模拟多签地址测试方式

在已经决策好下一届信托列表后，首先要进行地址有效性的测试。由于Bitcoin的特殊性，因此采用在链上生成Bitcoin模拟多签地址，并在链下使用其他开源工具使用相同参数生成，进行地址对比，来保证地址生成的正确性。

因此当前信托与下一届信托之间需要执行[信托#下一届信托节点准备](信托#下一届信托节点准备)

**由于Bitcoin的特殊性，其多签的构造只能由一个人首先构造带签原文，然后签署后将结果转交给下一个签名，因此请下一届信托节点自行协商这个构造过程**

#### 模拟在 ChainX上生成原链多签信息

通过rpc接口：

```bash
# post 127.0.01:8086
{
	"id":1,
	"jsonrpc":"2.0",
	"method":"chainx_getMockBitcoinNewTrustees",
	"params":[ ["0x471af9e69d41ee06426940fd302454662742405cb9dcc5bc68cexxxxx", "0x806a491666670aa087e04770c025d64b2ecebfd91a74efdc4f4329xxxxx","0x1cf70f57bf2a2036661819xxxxx","0xxxxcad99"] ]
	# params 为下一届信托列表的ChainX公钥 （注意这里使用的是公钥，非ss58的地址）
}
```

其返回值：

```bash
{
    "jsonrpc": "2.0",
    "result": {
        "coldEntity": {
            "address": "2N24ytjE3MtkMpYWo8LrTfnkbpyQQQQQQQQ", # bitcoin 冷多签地址
            "redeemScript": "0x532102a79800dfed17ad4c78c52797adafasdfasd69421080f42d27790ee2103eceasdfasdfasda3e62ef6b2f6ad9774489e9aff1c8bc684d87dfdasfasdf2dd162e8d8614a4afbb8e2eb14eddf4036042b35asdfasdfd8ac4d3e055eae7551427487e281e3efba618bdd395f2f54ae" # bitcoin 的 冷 redeemScript
        },
        "counts": {
            "required": 3,
            "total": 4  # 多签总共4个人，要求至少3个人参与
        },
        "hotEntity": {
            "address": "2N1CPZyyoKj1wFz2Fy4gEHpSCVqqqqqqq", # bitcoin 热多签地址
            "redeemScript": "0x532103f72sddfasdffsdfdsfszdfa1d179e0a8129bdba210306117a360e5dbe10e1938fdsfasfasdfasdf11b97de6b2917dd210311252930af8fdsafasdfasdfabac275bba77ae402dfdasdfsfdsafasfasdfasdf466cc4d8df674939a5538c354ae"# bitcoin 的 热 redeemScript
        },
        # 参与生成多签地址的信托者列表，注意这个列表的顺序与发送的参数的列表顺序完全一致，因为其会印象多签地址的生成
        "trusteeInfo": [  
        	{
                "accountId": "0x6488ceea630000b48fed318d13248ea7c566c0f4d2b8<ChainX公钥>",
                "coldPubkey": "0x...",
                "hotPubkey": "0x..."
            },
            {
                "accountId": "0x6488ceea630000b48fed318d13248ea7c566c0f4d2b8<ChainX公钥>",
                "coldPubkey": "0x...",
                "hotPubkey": "0x..."
            },
            {
                "accountId": "0x6488ceea630000b48fed318d13248ea7c566c0f4d2b8<ChainX公钥>",
                "coldPubkey": "0x...",
                "hotPubkey": "0x..."
            },
            {
                "accountId": "0x6488ceea630000b48fed318d13248ea7c566c0f4d2b8<ChainX公钥>",
                "coldPubkey": "0x...",
                "hotPubkey": "0x..."
            },
        ]
    },
    "id": 1
}
```

这个返回值的相应信息即为换届后，链上Bitcoin信托相关部分的相应信息。

#### 测试ChainX上生成的多签和其他开源工具生成的多签是否相同

若为了验证这个链上信息是否正确，可以通过其他工具进行相应的验证，请注意填写时必须保证和以上的`trusteeInfo`部分的列表顺序相同，填写后检查生成的冷热多签地址与`redeemScript`是否与`coldEntity`与`hotEntity`相同

[tool link](https://iancoleman.io/multisig/)

获取地址结果后，与ChainX链上模拟生成的结果进行比较。

#### 测试地址是否可正常的签名

在已知下一届的信息后，依照 [信托#下一届信托节点准备](信托#下一届信托节点准备) 部分进行操作。

对于Bitcoin而言，本质上就是**使用Bitcoin的多签工具进行多签转账**，保证多签地址能够正常的接受与发送。**这个过程与ChainX没有任何关系**

同时请信托节点注意以下内容：

* 由于这块与ChainX关系不大，因此该多签工具希望节点自己准备。ChainX仅准备一个简单的工具`chainx-multisig-verify-script`以提供测试。
* 请严格区分上文中的**ChainX的多签提现**与这里**链外多签转账**以测试地址多签有效性。

信托节点可以使用ChianX提供的签名测试工具[chainx-multisig-verify-script](https://github.com/chainx-org/chainx-multisig-verify-script)或其他可以测试签名操作的工具进行签名，现在以`chainx-multisig-verify-script`为例说明一下信托节点如何轮流签名。
`chainx-multisig-verify-script`工具配置详见[README](https://github.com/chainx-org/chainx-multisig-verify-script/blob/master/README.md)

1. 信托节点首先要构造出提现交易原文需要保证`config.js`文件中`toSignRawTransactionHex`字段值是""，必须填充`privateKey`，`multisigAddress`，`targetAddress`，`amount`，`redeemScriptHex`，`fee`字段。
   执行:

```
node index.js
```

此时输出交易原文：

```
02000000012d3ffc41dfc4a420d809edb1d5924fe1f0426f45bff4d8a1c7f6abecf1b0befd17a8ef7fabac275bba77ae40210227e54b65612152485a812b8856e92f41f64788858466cc4d8df674939a5538c354aeffffffff02204e00000000000017a91460c934b41baf6d1da84cd314fcd0f3d2106953008750e0c7200000000017a9145737c1979343920ceea40e7c7d68b264b0effa3e8700000000
```

由当前信托节点应将交易原文转发给下一个（任意）信托节点

2. 未签名的信托节点收到交易原文后将交易原文填入`config.js`文件中的`toSignRawTransactionHex`字段，并将与自己设置的BTC公钥对应的私钥填入`config.js`文件中的`privateKey`字段，然后运行:

```
node index.js
```

将输出的签名后的交易转发给另外未签名的信托节点，重复此步骤，直到签名数大于等于`信托节点数*2/3`，但是不能全部都签名，否则交易无法广播。比如信托节点数为４，需要签名数为３，最后广播的签名后交易只能有３个节点签名。信托节点数为６，需要签名数为４，最后广播的签名后交易只能有４个节点签名。

3. 当签名满足N/M签名条件后(即信托节点数为４，已经有３个节点签名)，信托节点可以将交易广播至BTC测试网，广播方式任意选择，此处推荐[blockcypher](https://live.blockcypher.com/btc-testnet/pushtx/)