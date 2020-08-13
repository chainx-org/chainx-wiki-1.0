
# 节点运维
<!-- TOC GFM -->

* [1. 推荐机器配置](#1-推荐机器配置)
* [2. 日志分割](#2-日志分割)
  * [2.1 新日志系统](#21-新日志系统)
* [3. 使用交互式输入密码](#3-使用交互式输入密码)
* [4. 监控节点是否正常出块/同步](#4-监控节点是否正常出块同步)
* [5. 设置节点的libp2p最大连接数](#5-设置节点的libp2p最大连接数)
* [6. 增加节点的Bootnodes](#6-增加节点的Bootnodes)
* [7. 关闭节点本地网络的mDNS](#7-关闭节点本地网络的mDNS)
* [8. 节点退选，连接，云服务商，vps选择等问题](#8-节点退选连接云服务商vps选择等问题)
* [9. ChainX(Substrate)模型下的数据存储位置](#9-chainxsubstrate模型下的数据存储位置)
* [10. 数据运行中出错](#10-数据运行中出错)
* [11. 本机数据备份](#11-本机数据备份)
* [12. 信托节点自建比特币全节点的参考](#12-信托自建比特币全节点的参考)
* [13. 节点数据库与状态缓存](#13-节点数据库与状态缓存)
* [14. 合理停止或重启节点（针对V1.1.0以后版本）](#14-合理停止或重启节点（针对V1.1.0以后版本）)
* [15. 打开p2p监听端口的防火墙](#15-打开p2p监听端口的防火墙)<!-- /TOC -->


## 1. 推荐机器配置

以阿里云为例，ChainX 主网推荐配置不低于: CPU 4 核, 内存 4 G, 带宽 10M，磁盘使用SSD, 服务器费用支出每个月不到 1 K。截止 2020-08-13，ChainX 1.0 主网数据大约320G, 在 ChainX 2.0 升级前加入的新节点，建议磁盘空间不低于 400G.

## 2. 日志分割

若一直持续运行日志过大，可以对设置`crontab`的定时任务对日志进行切分，可参考[日志切分](https://blog.csdn.net/shawnhu007/article/details/50971084)编写脚本,或使用`logrotate`等工具

## 2.1 新日志系统

ChainX于 `V1.0.2` 中**默认**使用了`log4rs` 日志库替代`substrate`默认的日志库`env_logger`，因此对于日志系统有新的特性。

>  如果新的日志系统出现问题，或不需要新的日志系统特性，则可以恢复使用老的日志系统，并配合[2. 日志分割](2. 日志分割)进行操作，恢复老的日志系统可在`V1.0.2`版本的执行启动命令加入
>
> ```bash
> {
>     "default-log": true
> }
> ```
>
> 即可启用老日志系统

对于新的默认使用日志系统，将会具备如下特性：

* 日志根据大小自动切分
* 日志切分有上限限制，当超过上限会自动删除老日志
* 切分的日志可以进行自动压缩
* 日志可以指定目录，以及日志的文件名
* 日志**默认**不输出到控制台中，只输出到文件

因此对于 ChainX 的 `V1.0.2`版本，若不配置任何日志选项，可以用如下命令启动：

```bash
$ nohup ./chainx --config=config.json > error.log 2>&1 &
```

启动后，在当前路径下与日志有关的文件及目录如下：

```bash
./
|- error.log  # panic 或其他异常信息会打印到这个文件中，即原来的标准输入输出
|- log/  # 自动在当前目录下生成 log 目录
	|- chainx.log  # 默认日志的文件名是 chainx.log
```

其中若无异常情况，则`error.log`为空白，其余日志将会输出到`log/chainx.log`中。

**默认**配置提供了：

* 默认的文件名为`chainx.log`
* 默认输出到启动节点时的路径的相对路径`log/`目录下
* `chainx.log `最大为`300M`
* 当`chainx.log`超过`300M`后，会在`log/`目录下将老日志重命名为`chainx.log.0`，并新生成一个文件`chainx.log`继续输出，进行**日志轮转**。当日志满后，将会把`chainx.log.0`命名为`chainx.log.1`，把`chainx.log`命名为`chainx.log.0`，并对新生成的`chainx.log`继续输入。老日志将会以`chainx.log.0`，`chainx.log.1`，`chainx.log.2`以此类推进行命令。
* 默认情况下日志轮转次数为`10`，当超过`10`次后，会**自动删除最老**的日志，及`chainx.log.9`
* 默认情况下老日志不进行压缩

即：**使用新的日志系统以后，默认情况下无需对日志进行其他运维操作。**

`V1.0.2` 日志系统提供的参数如下：

```bash
{
	"log-compression": false, # 开启日志压缩，若开启这个参数，则老日志以 `chainx.log.gz.0` 命名并压缩，默认不开启
	"log-console": false, # 日志输出到文件的同时，同时输出到控制台的标准输入输出，默认不开启
	"log-dir": "./log", # 日志输出的目录，默认为`./log`，请保证日志目录有写入权限，若路径不存在，会自动创建
	"log-name": "chainx.log", # 日志的文件名，默认为`chainx.log`，若设置成其他，则日志轮转也会一直使用配置的文件名
	"log-roll-count": 10, # 日志轮转的上限，默认为10
	"log-size": 300, # 日志的大小上限，单位为 MB，即单个日志最大为300M。`log-size` * `log-roll-count`将是`log-dir`目录最大占用的空间
}

```

## 3. 使用交互式输入密码

由于交互无法放置在后台,因此若想在后台启动,并且使用交互式输入密码,需要安装一些工具

1. 安装 `screen`

 以 Ubuntu为例

 ```bash
 $ sudo apt install screen
 ```

2. 使用screen托管启动节点

 ```bash
 # 注意最后的 -i 参数
 $ screen -L -S chainx ./chainx --config=<ConfigPath/chainx.conf> -i 
 ```

 启动后按一下命名可退出screen

 ```bash
 Ctrl-A-D
 ```

 此时在当前路径下会出现对应的日志文件

 若想attach到screen中,可以执行:

 ```bash
 # 列出screen
 $ screen -ls
 $ screen -r <上面 ls 出现的screen>
 ```

 即可进入screen中

 若需要停止节点,进入screen按 `Ctrl-C` 即可退出

## 4. 监控节点是否正常出块/同步

节点可以编写脚本监控自己的出块是否正常，主要接口都可以从[RPC](RPC)文档中获得说明

1. 获取当前自己节点的最高块高`chain_getHeader`
2. 获取自己相连的**其他节点**的块高`system_peers`
3. 从官方钱包中提供的rpc接口获取当前的块高

因此监控脚本可以根据2，3获取到的数据，去衡量1中获取的块高是否正常。

请节点根据自己情况设计自己的监控脚本。由于目前libp2p还不够完善，因此我们 **强烈建议** 当发现块高不正常后节点 **主动停止** 自己的节点，已防止后续造成大范围的网络分区。

该问题将会随着substrate与ChainX的升级不断完善，谢谢各位节点理解！**当前请参考以下5,6,7指示。**

##　5. 设置节点的libp2p最大连接数

一般情况下若带宽允许的范围，可以将自己的p2p连接上限提升

通过在启动的参数配置文件中设置`in-peers`及`out-peers` 设置连接数上限，带宽允许的情况下设置为40-100以上为好

```json
{
	"in-peers": 100,
	"out-peers": 100,
}
```

## 6. 增加节点的Bootnodes

### 6.1 增加节点的种子以保证正常运行的稳定

1. 挑选种子

   1. 随机选择以下种子列表的**一个**：（不推荐，因为种子节点可能已经连满了，或者失效）

      ```bash
      "/ip4/47.96.97.52/tcp/20222/p2p/QmXABmmDgTT75o2UVCKS8iaYh75zPL5by3iSfS8msSWwEo",
      "/ip4/47.110.14.60/tcp/20222/p2p/QmR2Jdd4V93yzTDyX5gKQ7PE4M2KsbK3vjC8RvXCnzqHU6",
      ```

   2. **或**通过过 [https://telemetry.polkadot.io/#/ChainX ](https://telemetry.polkadot.io/#/ChainX )，或者[https://stats.chainx.org/](https://stats.chainx.org/)点击右边设置按钮，打开最下面一栏的`NetworkState`，然后回到主界面，此时每一个节点的最右边会出现网络链接的按钮。选择一个目前块高度正常的节点点开会跳转到一个新页面，是一个`json`数据。（推荐）

      - 其中

        ```bash
        {
            peerId: "xxxxxxxxxxxxxxxxxxxxxxxx",
            listenedAddresses: [
            "/ip4/172.16.35.147/tcp/20222",
            "/ip4/127.0.0.1/tcp/20222"
            ],
            externalAddresses: [
            "/ip4/<ip>/tcp/20222",  # ipv4
            "/ip6/<ip>/tcp/20222" # ipv6
            ]
        	...
        }
        ```

      - `peerId` 代表节点的网络标示（存储于`networks`文件夹中），`externalAddresses`代表对外监听的ip和端口

      - 因此种子节点由 `externalAddresses`与`peerId`构成，为：

        ```bash
        seed = <externalAddresses(ipv4部分)>/p2p/<peerId>
        # e.g.
        seed = /ip4/172.16.176.18/tcp/20222/p2p/QmRaP225FNXoyB7WE8twinY8B6dVJXyGsmEYFA1Fc54rw1
        ```

2. **推荐：刚才在第1步中挑选的种子节点可以配置到启动的参数配置文件中**

   ```bash
   {
       "bootnodes": [
            "<种子1>",  # 注意这里需要用引号包起来
            "<种子2>",
            "<种子3>"
       ]
   }
   ```

   有能力的节点可以通获取到一些其他节点的网络信息，制定网络拓扑，找出和种子节点连接最近的节点，并**将他们的连接信息制作成种子节点**加入自己的种子列表当中，这样可以减少被网络分区的概率！

### 6.2 跳出网络分区

目前由于substrate p2p 不够完善的原因，会导致部分节点**有可能在运行过程中出现网络分区**，因此当监控脚本发现自己无法同步区块时可以做出以下操作**跳出网络分区**： 

1. 挑选出**一个**种子节点后，停止节点，然后以一下命令启动：

   ```bash
./chainx --config=<你的配置文件名，如config.json> --in-peers=1 --out-peers=1 --bootnodes=<刚才挑选的种子节点，不带引号> 
   ```
   
   这样可以很快跳出网络分区
   
3. 跳出网络分区后，停止节点，重新配置种子，恢复`in-peers`和`out-peers`，以正常方式继续启动即可。


## 7. 关闭节点本地网络的mDNS

由于当前Substrate版本的libp2p尚不完善，因此在节点发现部分会尝试和其他非ChainX节点但是是基于Substrate的区块链相连接。这种情况下，在日志中会出现以下类似的日志：

```bash
2020-06-30 11:42:19:954 WARN Connected to a non-Substrate node: IdentifyInfo { public_key: Secp256k1(PublicKey(PublicKey(Affine { x: Field { n: [5746656, 58711843, 20195489, 29518243, 59441395, 63997608, 24017478, 31578399, 15341797, 3019465], magnitude: 1, normalized: true }, y: Field { n: [26294390, 36271856, 9444379, 38122875, 7362362, 16689314, 50897596, 22588675, 55409781, 4074490], magnitude: 1, normalized: true }, infinity: false }))), protocol_version: "/totem-meccano/1.0", agent_version: "totem-meccano-node/v1.4.3-b8ac2e37-x86_64-linux-gnu (unknown)", listen_addrs: ["/ip4/149.28.31.182/tcp/16181", "/ip4/10.13.6.234/tcp/16181", "/ip4/10.13.2.38/tcp/16181", "/ip4/10.13.4.201/tcp/16181", "/ip4/10.13.4.157/tcp/16181", "/ip4/10.13.2.119/tcp/16181", "/ip4/10.13.2.69/tcp/16181", "/ip4/127.0.0.1/tcp/16181", "/ip4/149.28.31.182/tcp/16181", "/ip6/2001:19f0:7001:1342:5400:2ff:fe8a:f282/tcp/16181", "/ip6/::1/tcp/16181"], protocols: ["/ipfs/ping/1.0.0", "/substrate/meccano/2", "/ipfs/kad/1.0.0", "/ipfs/id/1.0.0"] }
```

为了减少这种情况，可以在配置文件中配置：

```bash
{
    "no-mdns": true,
}
```

以减少这种情况的影响（不能根除）。

## 8. 节点退选，连接，云服务商，vps选择等问题

* 关于防止退选问题，更详细请查阅[FAQ#漏块](FAQ#漏块)，[FAQ#检查漏块的可能方式](FAQ#检查漏块的可能方式)，[FAQ#节点防止退选的方法](FAQ#节点防止退选的方法)
* 关于阿里云香港
  * 目前ChainX有一些服务放于阿里云（国际）香港上，但是至今已出现了**多次网络访问出现短暂性异常**的情况，因此建议使用阿里云（国际）香港的节点引起警惕

## 9. ChainX(Substrate)模型下的数据存储位置

对于ChainX（或使用Substrate开发的链），其数据存储的位置都是类似的。

启动命令`-d/--base-path`或配置文件中`base-path`指定的路径为节点存储的根路径。假设`base-path`指定的路径为`DATABASE`，则**启动节点后**该路径下会出现：

```bash
<DATABASE>/chains/<节点指定的链名>/db
```

该`db`文件夹下的存储即为该链数据存储的路径。

如对于ChainX而言:

* 若使用主网启动（`chain=mainnet`或不指定），则主网的数据路径为：`<DATABASE>/chains/chainx_mainnet/db`
* 若使用测试网启动（`chain=testnet`），则测试网的数据路径为：`<DATABASE>/chains/chainx_testnet/db`

因此若需要备份，替换节点存储的数据，即对该`db`文件夹进行操作：

1. 若需要备份数据，则**停止节点后**对该`db`目录进行打包
2. 若使用备份数据或数据包进行数据替换，则是**删除或移走已有的db文件夹**，并将备份的`db`文件夹解压到该目录下`<DATABASE>/chains/<节点指定的链名>`，并保证数据文件夹名字为`db`

## 10. 数据运行中出错

由于substrate底层可能存在一定的bug，目前已知substrate会在某些特定情况下出现以下崩溃（`panic`）问题

```bash
# ...
  30: start_thread (0x7f0e3a4686da)
  31: __clone (0x7f0e39f7988e)
  32: <unknown> (0x0)
# 注意这里的关键字 "state already discarded"，关键字 state
Thread 'main-tokio-1' panicked at 'called `Result::unwrap()` on an `Err` value: UnknownBlock("State already discarded for Number(261843)")', src/libcore/result.rs:999
```

或者出现

```bash
# ...
  51: start_thread (0x7fba817f56b9)
  52: clone (0x7fba8131541c)
  53: <unknown> (0x0)
# 注意这里的关键字 "Trie lookup error, Invalid state root" , 关键字 state
Thread 'main-tokio-3' panicked at 'Externalities not allowed to fail within runtime: "Trie lookup error: Invalid state root: 0xf8eb2f38dc9b10a06f7ec6f798a85cd33c763d7cf1ecfbff63e3bf9a8ee8cdc4"', src/libcore/result.rs:999
```

目前已知这类错误应该**只会发生在验证者节点**上，原因是由于在验证者在出块的同时，收到相同高度的分叉区块，重组区块时可能在substrate内部诱发了一些隐藏问题，导致节点读取不到一个已经废弃的状态根或数据，导致`panic`

目前这个问题只能等待substrate修复，无其他可行方案，见issue: [https://github.com/paritytech/substrate/issues/2637](https://github.com/paritytech/substrate/issues/2637)。

解决方案：

1. 首先先尝试重新启动节点，目前已有案例重启节点能够恢复节点的运行。

2. 若重启节点后仍然`panic`，那么该份数据已经无法恢复。只能**重新开始同步**或**使用已有数据恢复**

   1. 为应对这种情况，ChainX目前提供了`2019.07.11`备份过的数据镜像并提供下载，节点可通过以下命令下载最近的数据镜像，并基于该部分进行同步

      ```bash
      wget 114.55.243.192/db_no_archive.tar.gz
      # 或者命令
      wget 114.55.243.192/db_archive.tar.gz
      ```

   2. 其中 `db_no_archive.tar.gz`数据镜像代表使用**默认的裁剪模式**启动的数据（即不添加`"pruning":"archive"`），请注意，若使用此数据，配置文件中**一定不能**添加`"pruning":"archive"`！

   3. 其中`db_archive.tar.gz`数据镜像代表使用**存档模式**启动的数据（即在配置中添加`"pruning":"archive"`），请注意，若使用此数据，配置文件中**一定要添加**`"pruning":"archive"`！（ChainX**比较推荐**节点目前使用该模式启动节点）

## 11. 本机数据备份

由于ChainX项目处于早期，可能存在目前不可知的因素导致数据错误，由于当出现数据错误时，会导致节点无法出块，受到惩罚，因此ChainX**建议**配置足够（cpu，存储空间）的节点，在跑验证者节点的那台机器上同时运行一个备份数据节点，当验证者节点的数据出现异常且无法恢复时，可以直接使用备份数据启动验证者节点！

该备份节点可以通过本机内网与验证者相连的方式同步数据，不对外进行访问

1. 运行本机备份数据且不占用外部带宽的方式：

   假设验证者节点位于`<validator>/`路径下，相应的目录组织如下：

   ```bash
   <validator>/
        ├── chainx
        ├── config.json
        ├── keystore/
        ├── database/            # config.json 中的 "base-path" 字段指向的目录
        │   └── chains/
        │       └── chainx_mainnet/
        │           ├── db       # 该目录为ChainX数据目录
        │           └── network  # 该目录记录了 ChainX节点 在p2p网络中的 peerId
   ```

   那么首先可以仿照验证者的配置，建立备份节点的路径`<backup>/`如下：

   ```bash
   <backup>/
        ├── chainx
        └── config_bak.json
   ```

2. 通过rpc接口[RPC#system_networkstate](RPC#system_networkstate)查阅验证者的p2p网络的peerId

   ```bash
   # 例如可以使用如下命令获取网络情况
   curl -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{"id":100, "jsonrpc":"2.0", "method":"system_networkState", "params":[]}' 127.0.0.1:8086
   ```

   该rpc将会返回

   ```jsonc
   {
       "jsonrpc":"2.0",
       "result":{
           # ...
           "listenedAddresses":[
               "/ip4/172.16.62.73/tcp/40333",
               "/ip4/127.0.0.1/tcp/40333"
           ],
           "notConnectedPeers":Object{...},
           "peerId":"QmRSeJH46AbP53ibLULqTaQSoVyVHQwUqz8EsRapbYKAqR",
           # ...
   }
   ```

   其中

   - `listenedAddresses`ip可以查阅到其监听内网ip及端口

   - `peerId`为验证者在p2p网络中的公钥

   - 根据 `listenedAddresses`与`peerId`可以组件种子节点：

     ```bash
     # listenedAddresses中的内网（如 127开头，172开头等等）+ "p2p" + peerId
     # 如：
     /ip4/127.0.0.1/tcp/40333/p2p/QmRSeJH46AbP53ibLULqTaQSoVyVHQwUqz8EsRapbYKAqR
     ```

3. 配置备份数据的配置文件如下：

   配置`<backup>/config_bak.json`如下：

   ```jsonc
   {
     "validator": false,  // 该字段一定要为false
     "log": "error",      // 建议提升error等级，减少日志的打印
     "port": 30333        // 建议修改成和 validator中不一样的端口，如 30300
     "ws-port": 8187,     // 建议修改成和 validator中不一样的端口，如 8187
     "rpc-port": 8186,    // 建议修改成和 validator中不一样的端口，如 8186
     "other-execution": "NativeElseWasm", // 不变
     "syncing-execution": "NativeElseWasm",
     "block-construction-execution": "NativeElseWasm",
     "name": "Your-Node-Name",         // 在节点浏览器中显示的节点名，建议换一个与validator不同的名字，或者使用`"no-telemetry": true`，该参数会让该节点不发送信息给节点浏览器
     "base-path": "<backup>/database", // 备份数据的路径
     "in-peers": 1, // 关键！in-peers 与 out-peers 一定要设置成1
     "out-peers": 1, 
     "pruning": "archive", // 可选，启用存档模式，若不使用存档模式启动，则不需要这一行
     "bootnodes": [
     	// 在2中查询到的本机验证者种子节点，需要用双引号括起来
     	"/ip4/127.0.0.1/tcp/40333/p2p/QmRSeJH46AbP53ibLULqTaQSoVyVHQwUqz8EsRapbYKAqR"
     ],
   }
   ```

   其中 ：

   - `"in-peers"`和`"out-peers" ` 的值一定要设置为1
   - `"bootnodes"`的值为在第2步查询到的节点
   - 对于`"pruning":true` 是否开启存档模式，ChainX目前**建议**备份节点开启存档模式，请注意若验证者没有开存档模式，而备份节点开了，那么之后若出现异常时，需要使用备份节点的数据替换验证者节点的数据时，验证节点**一定**要加上`"pruning": "archive"` ，否则会出现更多的数据异常！
   - 其他参数请查阅`./chainx --help` 进行自己节点需要的配置

4. 启动备份节点

   1. 正常启动备份节点，等待数据同步（不需要停止验证者节点，但是需要同步一段时间）

   2. 若目前验证者节点没有出现过任何异常，可以将验证者的数据直接拷贝给备份节点，让备份节点从验证者的数据直接启动（需要停止节点，但可以很快启动备份节点）

      操作方法：以**将验证者节点停止**下来，进入`<validator>/database/chains/chainx_mainnet/`目录，将db压缩

      ```bash
      tar -czf db.tar.gz db/
      ```

      压缩结束后，**可重新启动验证者节点**，并将压缩包移动到备份节点的数据目录下, 并解压：

      ```bash
      mv <validator>/database/chains/chainx_mainnet/db.tar.gz <backup>/database/chains/chainx_mainnet/
      cd <backup>/database/chains/chainx_mainnet/
      tar -xzf db.tar.gz
      ```

      之后可以使用`<backup>/config_bak.json`配置文件直接使用该数据启动

5. 验证者节点数据异常后的操作方法

   验证者数据若出现异常，则可使用同步节点的数据启动！

   1. 首先必须将验证者节点与备份节点都停止下来
   2. 重新启动验证者：
      1. 切换配置，在备份数据上直接启动
         1. 将验证者的配置文件`<validator>/config.json`拷贝到`<backup>`下
         2. （可选，推荐这样做）将`<validator>/database/chains/chainx_mainnet/networks`目录也拷贝到`<backup>/database/chains/chainx_mainnet/`目录下
         3. 修改`<backup>/config.json`中的`"base-path"`字段指向`<backup>/databash`目录
         4. （**重要！**）检查`<backup>/config.json`中的`“keystore-path”`是否指向原来的keystone路径`"<validator>/keystore"`
         5. （**重要！**）检查`<backup>/config.json`中的`"validator":true` 字段一定要设置为`true`且`"validator-name":xxx`字段一定填写正确
         6. 重新启动验证者
      2. **或**采用数据拷贝（移动）启动
         1. 删除`<validator>/database/chains/chainx_mainnet/db`
         2. 将备份数据目录下的`<backup>/database/chains/chainx_mainnet/db`压缩并拷贝到`<validator>/database/chains/chainx_mainnet/`目录下
         3. **或者**直接将`<backup>/database/chains/chainx_mainnet/db` mv 到 `<validator>/database/chains/chainx_mainnet/`目录下
         4. 重新启动验证者
   3. 根据需要重新启动同步数据节点

##  12. 信托自建比特币全节点的参考

   建议信托节点使用自己搭建的比特币全节点

   1. 直接在 [bitcoin.org](https://bitcoin.org/bin) 下载二进制包
      或 

      ```bash
      $ wget https://bitcoin.org/bin/bitcoin-core-0.17.0.1/bitcoin-0.17.0.1-x86_64-linux-gnu.tar.gz
      ```
   2. 解压,创建软连接
      
      ```bash
      $ tar zxf bitcoin-0.17.0.1-x86_64-linux-gnu.tar.gz
      $ ln -fs /opt/bitcoin-0.17.0 /opt/bitcoin
      $ ln -fs /opt/bitcoin-0.17.0/bin/bitcoind /usr/local/bin/bitcoind
      $ ln -fs /opt/bitcoin-0.17.0/bin/bitcoin-cli /usr/local/bin/bitcoin-cli
      ```

   3. 指定数据目录，或者使用默认路径

      ```bash
      $ mkdir -p /bitcoin-mainnet/btc_data
      ```

   4. 创建配置文件

      ```bash
      $ mkdir ~/.bitcoin
      $ vim ~/.bitcoin/bitcoin.conf
      ```
      ```bash
      # server=1 tells Bitcoin-Qt and bitcoind to accept JSON-RPC commands
      server=1
      # On client-side, you add the normal user/password pair to send commands:
      rpcuser=远程访问的认证用户
      rpcpassword=远程访问密码

      # How many seconds bitcoin will wait for a complete RPC HTTP request.
      # after the HTTP connection is established. 
      rpcclienttimeout=30

      # it is also read by bitcoind to determine if RPC should be enabled 
      # 远程访问的ip或网段 建议指定ip范围，不允许陌生ip访问
      #rpcallowip=10.1.1.34/255.255.255.0

      # Listen for RPC connections on this TCP port:
      rpcport=8332
      rpcservertimeout=60

      # Specify where to find wallet, lockfile and logs. If not present, those files will be
      # created as new.
      wallet=/bitcoin-mainnet/wallets
      walletdir=/bitcoin-mainnet/wallets

      datadir=/bitcoin-mainnet/data
      blocksdir=/bitcoin-mainnet/data

      # Log timestamps with microsecond precision.
      logtimemicros=1
      # Location of the debug log
      debuglogfile=/bitcoin-mainnet/debug.log
      ```

   5. 启动比特币全节点
      ```bash
      $ nohup bitcoind -conf=~/.bitcoin/bitcoin.conf -daemon &
      ```

   6. 测试比特币节点rpc服务

      ```bash
      $ curl -s -X POST --user btc:btc2018 -H 'content-type: text/plain;' http://127.0.0.1:8332/ --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getmininginfo", "params": [] }'

      ```
   7. 往钱包导入信托冷热多签地址
      每当信托换届或者信托冷热多签地址更换时，需要运维人员将更换的新地址导入到钱包，以便信托节点在使用ChainX-wallet构造提现交易时可以获取到该地址的UTXO列表。导入命令如下：
      ```bash
      $ bitcoin-cli -rpcport=8332 -rpcuser=btc -rpcpassword=btc2018 importaddress 比特币地址 "" true
      ```

## 13. 节点数据库与状态缓存

当前ChainX所产生的数据量已经比较大，对于状态的索引及底层rocksdb数据库的查询均会耗费大量cpu资源，因此**强烈建议**节点在内存允许的情况下在配置中添加数据库与状态的缓存。例如推荐的配置如下：

```bash
{
    "db-cache": 1024,
    "state-cache-size": 2147483648,
}
```

其中：

* `db-cache`对应与rocksdb的缓存，单位是`MB`，例如该推荐中即为开启rocksdb的内存缓存1G
* `state-cache-size`对应与状态树的缓存，单位是`B`，例如改推荐中即为开启状态缓存2G（2G = 2\*1024\*1024）

**节点可以根据自己的内存状态自行调整，一般尽量分配`state-cache-size`是`db-cache`的两倍**

## 14. 合理停止或重启节点（针对V1.1.0以后版本）

由于自ChainX `V1.1.0`版本开始，为了支持合约模块正常运行采用了比`v1.0.6`往后的新版本的Substrate（依旧是Substrate 1.0，并非2.0）。但是**由于该Substrate版本在处理操作系统的中断信号不够完善**，因此会出现当需要停止或重启节点时，给运行中的节点发送正常的中止信号后（`kill <pid>`），节点进程依旧存活，没有停止的现象。

此时实际上在Substrate内部已经捕获了`kill -2`信号，但是同步区块的模块没有中止，因此进程依旧存活，会等到同步区块处理完队列中所有的区块后才会停止。

因此若正常发送`kill <pid>`中断信号后，节点仍未停止的情况下，可以再使用`kill -9 <pid>` 使节点强制停止。

例如以下`stop.sh`脚本：

```bash
#!/bin/bash
dir_path=<you path>
pgrep -f $dir_path/chainx | awk '{print $1}' | xargs kill

sleep 1
count=$(pgrep -fc $dir_path/chainx)
if [ "$count" = 1 ];then
    echo "not killed, use kill -9"
    pgrep -f $dir_path/chainx | awk '{print $1}' | xargs kill -9
fi
```

若在`v1.0.6`之前部署了定时重启节点的开发者，此时需要将`kill`节点的部分换成以上脚本过程。

例如以下`restart.sh`脚本

```bash
#!/bin/bash
# print executive time
date

dir_path=<you path>
pgrep -f $dir_path/chainx | awk '{print $1}' | xargs kill

sleep 1
count=$(pgrep -fc $dir_path/chainx)
if [ "$count" = 1 ];then
    echo "not killed, use kill -9"
    pgrep -f $dir_path/chainx | awk '{print $1}' | xargs kill -9
fi
echo "restart..."
sleep 6
cd $dir_path
# change `bash ./start.sh` to your start script
bash ./start.sh
```

## 15. 打开p2p监听端口的防火墙

由于当前运行节点的机器大部分都会配置防火墙策略，因此请运行的节点注意自己的监听的libp2p端口对应的防火墙策略是否是打开的。若防火墙端口未打开，则会表现为节点自身尝试连接别的节点能连上，而别的节点尝试连接自己连不上。这样有可能会导致本节点的Libp2p连接的peer们对本节点打分下降，最终导致自己的节点和别的节点断连。

因此请运行中的节点**务必**检查自己监听的Libp2p端口的防火墙策略是否是打开的。

测试自己的节点运行中/节点防火墙是否打开的方式，在其他机器/本机上执行：

```bash
> telnet <ip> <port>
```

若返回

```bash
Trying <ip>...
Connected to <ip>.
Escape character is '^]'.
```

则表示端口防火墙正常开放，节点正常运行。

若返回

```bash
Trying <ip>...
telnet: Unable to connect to remote host: Connection refused
```

或者

```bash
Trying <ip>...
# 长时间没有任何返回
```

则标示端口防火墙没有开放，或节点没有运行。