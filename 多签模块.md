## Overview
ChainX中实现了与Parity Ethereum的多签钱包合约类似的一个功能模块，但是目前这个模块仅开放给：
* 团队地址
* 议会
* 信托

多签的核心诉求为 **执行一个行为时，需要多个人参与并同意才可执行**。因此多签的步骤可以简单理解为：

* 一群人形成一个多签群体，并约定达到某个阈值可以执行操作
* 群体中的某个人可以提起一个执行提议（proposal）
* 群体中的其他人对这个提议进行同意（confirm）
* 当同意的数量达到某个阈值时，自动执行提议
* 其他人也可以撤销这个提议

针对以上步骤，在ChainX中设计多签的模型如下：

假设当前有4个用户（A，B，C，D）希望使用多签模块，并限定任意其中3人同意即可执行。

1. A，B，C，D其中有一个人为**多签部署者**，假设是A，其执行deploy，参数为`[B, C, D]`。因此`deploy([...])-> AccountId(multisig_addr)`
2. 多签部署后会生成一个**多签地址（账户）**，这个多签地址实际上是一个**没有私钥的ChainX账户**，其**所有行为和其他ChainX账户没有区别**，只是因为其没有私钥，**所有执行所有的行为都需要通过多签同意**才可操作
3. 通过多签模块，可以**对这个多签地址执行**操作，每次执行的proposal会赋予一个`multisig id`。`exec(multisig_addr, proposal)->mutlisig_id`
4. confirm某个提议是针对一个多签地址，执行一个`multisig id`的确认。`confirm(multisig_addr, multisig_id)`

因此多签的操作流程需要有用户A，B，C，D去发送交易执行`exec`，`confirm`交易。因此手续费扣除的是调用`exec`,`confirm`的手续费，而不是多签地址自身的手续费。

![](https://github.com/chainx-org/images/blob/master/multisig.png)

## 操作多签的方式

首先需要介绍，在ChainX上存在**地址**和**公钥**两个概念，其中在ChainX内部使用的所有标识用户的方式都是**公钥**

而公钥与地址的关系为：

> 公钥 <=> 加校验码，版本/检查校验码，版本 <=> base58编码 <=> 地址
>
> 公钥形同 0x42000fada2518d000083ccf6c5c7aa84ef5489e1355378e125d0000e3237a582，是32字节长度的16进制字符串
>
> 地址形同 5DZVtWLFnSAAAAo8NXGGGpeZ5QFUSehUjRRRRuTwKJCYpe9Q，是**以5开头**，以base58字符在其中的字符组成，长度不定，但一定大于32个字符

地址和公钥能够互相转换，一般在对ChainX的接口中使用地址，在ChainX内部使用公钥。

**在以下工具中，出现地址的位置，即可以填写地址，也可以填写公钥**

针对多签模块，ChainX提供了一个工具`multisig-tool`去执行相应的多签操作：

```bash
$ ./multisig-tool -- --help
multisig-tool 0.1.0
Chainpool <chainx.org>
An tool to operate multisig.

USAGE:
    multisig-tool [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
        --key <PRI_KEY>            Specify additional key seed
        --keystore-path <PATH>     Specify custom keystore path
    -w, --websocket <HOST:PORT>    specify node websocket addr, e.g. 127.0.0.1:8087

```

其中 `--key`指定操作多签的用户私钥，`--keystore-path`填写keystore的路径（暂时未支持），`-w`指定连接的的节点的`websocket`地址。

执行`multisig-tool`后，将会进入一个用户交互式的界面：

```bash
2019-04-24 18:17:57.934 INFO Connecting to ws://127.0.0.1:8087
2019-04-24 18:17:57.958 INFO Successfully connected
2019-04-24 18:17:57.986 INFO genesis hash is:0xcd89…97b2
> (等待输入，可以直接回车看help)
2019-04-24 18:19:39.459 INFO multisig-tool 0.1.0
Chainpool <chainx.org>

USAGE:
    multisig-tool [OPTIONS] <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -a, --acc <acceleration>    acceleration for this operation, default value is 2 [default: 2]
    -k, --key <PRI_KEY>         addition key only for this operation

SUBCOMMANDS:
    help         Prints this message or the help of the given subcommand(s)
    setkey       
    xassets      
    xmultisig    
```

其中

* `xassets`为资金相关的操作（注意非多签名），可以用来查询多签地址的余额和用户转账，但是不可用于多签的转账（因为“多签账户”自身是没有私钥的）
* `xmultisig`为多签操作的相关命令，接下来详细解释：

```bash
> xmultisig
2019-04-24 19:00:00.643 INFO xmultisig 0.1.0
Chainpool <chainx.org>

USAGE:
    xmultisig <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    confirm    # confirm 为 同意一个proposal使用的子命名
    exec       # exec 为提起一个proposal使用的子命名
    help       Prints this message or the help of the given subcommand(s)
    query      # query 为查询相关的数据的子命名

> 
```

执行列举

1. 需要将多签地址里的钱转移到某个账户上，假设这里的多签地址为 `ADDR`：

   多签账户归于于A，B，C，D，

   * A提起转账请求

   ```bash
   $ ./xmultisig-tool --key=<A的私钥>
   2019-04-24 19:03:27.092 INFO Connecting to ws://127.0.0.1:8087
   2019-04-24 19:03:27.116 INFO Successfully connected
   2019-04-24 19:03:27.138 INFO genesis hash is:0xcd89…97b2
   > xassets query <多签地址`ADDR`>
   2019-04-24 19:04:14.233 INFO 
   {
     "data": [
       {
         "details": {
           "Free": 34000000000, # 查询可看到这个多签地址上的PCX余额
           "ReservedCurrency": 0,
           "ReservedDexFuture": 0,
           "ReservedDexSpot": 0,
           "ReservedStaking": 0,
           "ReservedStakingRevocation": 0,
           "ReservedWithdrawal": 0
         },
         "name": "PCX"
       }
     ],
     "pageIndex": 0,
     "pageSize": 100,
     "pageTotal": 1
   }
   # 注意下面的命名名为：命令(xmultisig exec) 为<多签地址> <执行操作>
   # transfer 子命令接受的参数为 transfer [who(转账给谁)] -t <资产名，默认PCX，可选> [value(金额)]
   > xmultisig exec <多签地址`ADDR`（为xx地址执行xxx操作）> transfer <trasfer正常的接受转账的地址> 100000000
   2019-04-24 19:15:46.034 INFO Extrinsic hash is:0x420e39daf0a127669887840a087d394c534580b3258e84309ce7721994ceddf2 # exec 执行后的交易hash
   # 一笔exec执行后会生成针对这个proposal的 multisig_id
   > xmultisig query multisig-info  <多签地址`ADDR`(查询这个多签地址下的信息)>
   2019-04-24 19:21:26.119 INFO 
   MultiSigAddrInfo {
       addr_info: AddrInfo {
           addr_type: Normal,
           required_num: 3,
           owner_list: [
               # ...
           ],
       },
       pending: [ # pending 的列表即为待 confirm 的 proposal 列表
           MultiSigPending {
               id: 0xe85f21a414a20d63d0a19780ddc0bcc7d618e577a7f8032b6bc124c0f9b4064b, # 即为multisig_id
               state: PendingState {
                   yet_needed: 2, # 还需要几个人签名
                   owners_done: 1, # 已经有谁确认了，注意这里不是几个的意思，是**位掩码**，按照`owner_list`的顺序
                   # proposal 的内容，可以看到是一笔transfer交易
                   proposal: XAssets(
                       transfer(Id(<trasfer正常的接受转账的地址>), [80, 67, 88], 100000000, []), # [80, 67, 88] 代表(PCX)
                   ),
               },
           },
       ],
   }
   ```

   此时，可以等待其他用户进行confirm

   * B参与confirm

   A通知B，C，D说有一个proposal了，此时可以把上面的`multisig_id`发给他们，也可以让他们自己查看

   ```bash
   $ ./xmultisig-tool --key=<B的私钥>
   2019-04-24 19:03:27.092 INFO Connecting to ws://127.0.0.1:8087
   2019-04-24 19:03:27.116 INFO Successfully connected
   2019-04-24 19:03:27.138 INFO genesis hash is:0xcd89…97b2
   # B 查询多签地址`ADDR`的信息 
   > xmultisig query multisig-info  <多签地址`ADDR`(查询这个多签地址下的信息)>
   # ...
       pending: [ # pending 的列表即为待 confirm 的 proposal 列表
           MultiSigPending { # B 查询到刚才 A的proposal的 multisig_id
               id: 0xe85f21a414a20d63d0a19780ddc0bcc7d618e577a7f8032b6bc124c0f9b4064b, 
   #...
           },
       ],
   }
   > xmultisig confirm  <多签地址`ADDR`> <multisig_id>
   # B 提交confirm交易后的tx hash
   2019-04-24 19:32:09.620 INFO Extrinsic hash is:0x7abfb0c34f49dbf276e4a5f04f5a6c252db63b9534a1dd34185e2120a4f932e2
   ```

   此时B已经参与了确认，若查询地址信息可以看到` yet_needed: 1`，即还需要一个人签名

   * C参与confirm

   C使用自己的私钥执行上述相同的操作，提交confirm交易后即可发现transfer转账已经成功，钱已经从多签账户中转移到了<trasfer正常的接受转账的地址>上。

   **整体而言，手续费扣除的是 A 提起的`exec`，B，C提起的`confirm`，transfer本身不花费手续费。**

2. 其他信托操作

   ```bash
   $ ./xmultisig-tool
   2019-04-24 19:36:34.273 INFO Connecting to ws://127.0.0.1:8087
   2019-04-24 19:36:34.307 INFO Successfully connected
   2019-04-24 19:36:34.310 INFO genesis hash is:0xcd89…97b2
   > xmultisig exec  # 执行exec 可查看命令
   2019-04-24 19:36:39.167 INFO xmultisig-exec 0.1.0
   Chainpool <chainx.org>
   
   USAGE:
       xmultisig exec <addr> <SUBCOMMAND>
   
   FLAGS:
       -h, --help       Prints help information
       -V, --version    Prints version information
   
   ARGS:
       <addr>    
   
   SUBCOMMANDS:
       fix-withdrawal-state          just for Bitcoin trustees
       help                          Prints this message or the help of the given subcommand(s)
       other                         
       set-bitcoin-withdrawal-fee    just for Bitcoin trustees
       transfer                      
       transition-trustees           just for trustees
   
   > 
   ```

   因此可知，目前支持的命令有：

   * fix-withdrawal-state ：`xmultisig exec <addr> fix-withdrawal-state <withdrawal_id> <is_success>`  撤销或恢复用户申请的提现 填写用户提现申请的`withdrawal_id`与 撤销还是许可这笔提现的`is_success`

   * set-bitcoin-withdrawal-fee ：`xmultisig exec <addr> set-bitcoin-withdrawal-fee <fee>`  设置用户提Bitcoin的手续费

   * transfer  `xmultisig exec <addr> transfer [OPTIONS] <to> <value>`转账操作

   * transition-trustees  信托换届

     信托换届较复杂，请查看下文

## 使用多签进行信托换届

其中信托换届详情请看相关文档：[信托](信托)

根据换届逻辑，在选好下一届的信托用户后，即已经指定好下一届的信托的地址，则可进行以下操作：

### 发起信托换届的提议（exec）

其中当前届的**任意一个**信托者发起一个换届的 `exec` 的proposal为：

```bash
# 查询目前链上的特殊多签地址
> xmultisig query particular-accounts
2019-04-24 19:51:04.385 INFO 
{
  "councilAddress": "0x67df26a755e0c31ac81e2ed530d147d7f2b9a3f5a570619048c562b1ed00dfdd",
  "teamAddress": "0x42320fada2518d40e883ccf6c5c7aa84ef5489e1355378e125d4948e3237a582",
  "trusteesAddress": {
  # 可以看到对于Bitcoin信托而言，地址如下
    "Bitcoin": "0x69405700d3468a8dc4783c95cc7664eff7828a4815a0f05af269d9a8fd122bb1"
  }
}
> xmultisig exec transition-trustees --help
2019-04-24 19:42:02.531 INFO xmultisig-exec-transition-trustees 0.1.0
Chainpool <chainx.org>
just for trustees

USAGE:
    xmultisig exec <addr> transition-trustees [OPTIONS] <chain>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -t, --trustees <new_trustees>...    

ARGS:
    <chain>    
# 根据参数提示可以知，信托换届填写的参数如下： Bitcoin 为<chain> ，即为为Bitcoin链换信托
# 注意这里 -t 的顺序必须严格按照模拟生成信托换届的账户的顺序
# addr 即为刚才查询到的Bitcoin多签地址
> xmultisig exec <addr(这里即为当前届的信托多签地址)> transition-trustees Bitcoin -t <下一届公钥/地址1> -t <下一届公钥/地址2> -t <下一届公钥/地址3> -t <下一届公钥/地址4>
```

**强调！！注意这里 `-t` 指定地址的顺序必须 严格 按照模拟生成信托换届的账户的顺序，否则换届后，自动生成的信托地址会与测试中使用的信托地址不同**

举例：

```bash
xmultisig exec <addr(这里即为当前届的信托多签地址)> transition-trustees Bitcoin -t 
5EDc5F7DuasdfsfstLhvDdfsfv35NxyjkzMs9oJHFBUaikccb -t 0x23432525f65a22346a9159dee234bb00c85e43147bfdsf1921590c4df1 -t 0xb44325e96adad432432cb2d4fcb19f8ce213fa5d8f8a91de637f44cdfasf -t 0x42341c241c59e2a6a88efcf239df0eabb229b3697b4c844c788c9c51fcb2dfsfs
# 注意 -t ，以及 -t 的排序
# 5EDc5F7DuasdfsfstLhvDdfsfv35NxyjkzMs9oJHFBUaikccb 是ChainX地址
# 0x23432525f65a22346a9159dee234bb00c85e43147bfdsf1921590c4df1 等等是ChainX公钥，既可以填公钥也可以填地址
```

### 对信托发起的提议进行投票（confirm）

在这笔proposal存在后，其他信托者对这个笔proposal进行投票，投票成功后即可以进行换届：

```bash
$ ./xmultisig-tool --key=<其他信托者的私钥>
# 同样查询得到目前链上的特殊多签地址
> xmultisig query particular-accounts
2019-04-24 19:51:04.385 INFO 
{
  "councilAddress": "0x67df26a755e0c31ac81e2ed530d147d7f2b9a3f5a570619048c562b1ed00dfdd",
  "teamAddress": "0x42320fada2518d40e883ccf6c5c7aa84ef5489e1355378e125d4948e3237a582",
  "trusteesAddress": {
  # 可以看到对于Bitcoin信托而言，地址如下
    "Bitcoin": "0x69405700d3468a8dc4783c95cc7664eff7828a4815a0f05af269d9a8fd122bb1"
  }
}
> xmultisig query multisig-info  <多签地址`ADDR`(即为上面查询到的多签地址)>
# ...
    pending: [ # pending 的列表即为待 confirm 的 proposal 列表
        MultiSigPending { # B 查询到刚才 A的proposal的 multisig_id
            id: 0xe85f21a414a20d63d0a19780ddc0bcc7d618e577a7f8032b6bc124c0f9b4064b, 
#...
		  proposal: XMultiSig(
		  # 请信托群体履行好自己的职责，仔细检查下面列表是否符合预期，再执行confirm操作
                    transition_trustee_session(Bitcoin, [<下一届公钥/地址1>, <下一届公钥/地址2>, <下一届公钥/地址3>, <下一届公钥/地址4>]),
            ),
        },
    ],
}
# 进行了投票！
> xmultisig confirm  <多签地址`ADDR`> <multisig_id>
```

**注意！！请信托群体履行好自己的职责，仔细检查proposal中的列表是否符合预期，再执行confirm操作，你手中的confirm，相当于是自己对这个提议的投票，若发送confirm，代表你对这个提议提出了同意的投票**





以上即为多签模块的操作文档