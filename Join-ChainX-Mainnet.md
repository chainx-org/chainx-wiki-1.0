## 加入 ChainX 主网

<!-- TOC GFM -->

* [1. 下载节点运行程序](#1-下载节点运行程序)
* [2. 启动节点前准备](#2-启动节点前准备)
    * [删除老旧测试网节点](#删除老旧测试网节点)
    * [准备节点启动配置文件](#准备节点启动配置文件)
* [3. 启动节点](#3-启动节点)
    * [1. 启动同步节点](#1-启动同步节点)
    * [2. 启动验证者节点](#2-启动验证者节点)
        * [1. 节点账户公钥生成与注册](#1-节点账户公钥生成与注册)
        * [2. 生成出块公钥](#2-生成出块公钥)
        * [3. 修改启动配置文件启动验证者出块节点](#3-修改启动配置文件启动验证者出块节点)
            * [如何更新出块地址](#如何更新出块地址)
* [4. 出块节点正常运行后其他工作](#4-出块节点正常运行后其他工作)

<!-- /TOC -->

在开始以下步骤前，请了解以下内容:

- 钱包: https://wallet.chainx.org ，用户可以通过钱包创建账户，注册更新节点等各种操作。
- 节点浏览器: https://stats.chainx.org/#/ChainX ，节点浏览器可以看到当前ChainX网络所有节点的运行信息。
- 区块链浏览器: https://scan.chainx.org ，可以看到ChainX网络的各种链上信息，包括查询验证人节点的漏块信息等。

### 1. 下载节点运行程序

我们提供了以下两种途径:

- 在 https://github.com/chainx-org/ChainX/releases/tag/v1.0.0 下载 v1.0.0 测试网 ChainX 二进制 `chainx`，目前仅支持 Ubuntu 16.04+ 或 macOS。
- 如果是其他系统，可采用 [Docker 运行 ChainX 节点](Docker)。

### 2. 启动节点前准备

#### 删除老旧测试网节点

ChainX 主网已经于 2019 年 5 月 25 日正式启动，如果之前有运行过测试网节点，请先删除所有测试网数据：

* 若启动时通过 `-d` 或 `--base-path` 指定了DB存放目录，则将这个目录完全删除即可。

* 若启动时没有指定路径，则删除默认的DB目录：

    * Linux(ubuntu):

        ```bash
        $ rm -rf $HOME/.local/share/ChainX
        ```

    * macOS:

        ```bash
        $ rm -rf $HOME/Library/Application\ Support/ChainX
        ```

#### 准备节点启动配置文件

当前ChainX已经提供了**从配置文件读取参数启动节点**的方式，完整的启动参数可参见 `chainx --help`:

```bash
./chainx --help
ChainX 0.9.10-0f95c36-x86_64-linux-gnu

ChainX community
Fully Decentralized Interchain Crypto Asset Management on Polkadot

USAGE:
   chainx [FLAGS] [OPTIONS]
   chainx <SUBCOMMAND>

FLAGS:
   ......
......
OPTIONS:
   ......
   ......
```

help 中出现的所有命令均可在配置文件中进行配置，我们目前提供了 `json` 的配置文件格式。`chainx` 程序可选参数分为 `[FLAGS]` 和 `[OPTIONS]` 两部分, 示例如下:

- `FLAGS`。`true` 表示启用该FLAG, `false` 表示不启用该FLAG, 如：

```jsonc
{
    // 等价于在命令行传入 --validator， `./chainx --validator`
    "validator": true,
}
```

- `OPTIONS`。如：

```jsonc
{
    // 等价于在命令行传入 --base-path=<ChainX数据路径>, `./chainx --base-path=<ChainX数据存放路径>`
    "base-path": "<ChainX数据存放路径>",
}
```

不推荐使用命令行直接传参的方式启动节点，出于节点安全考虑，请使用 `json` 配置文件的方式启动节点。配置文件编写好后将该配置文件保存某处，**请做好对应的安全策略**。

下文为简便，将启动配置文件命名为 `config.json`，并将其放置到与 chainx 二进制程序的同级目录下。

```bash
$ ls
├── chainx
├── chainx.md5
└── config.json
```

### 3. 启动节点

对于有意愿当验证人的节点，我们强烈建议至少运行两个节点，即一个同步节点和一个出块节点。这样即使当出块节点出现问题时，也可以随时将备用的同步节点切换为出块节点，避免因为没有按时出块导致节点被惩罚。

我们推荐使用 `json` 配置文件的方式启动节点, [下载配置文件模板](https://gist.github.com/liuchengxu/3b3ed4ce027e39fc89b5a5a6c289bfaf)，可通过以下方式下载:

```bash
$ curl https://git.io/fjlZU -o config.json
```

**!!! 请不要直接使用该文件启动节点，务必参照以下启动指引修改配置文件模板后运行, 各参数将在下文解释。**

#### 1. 启动同步节点

对于同步节点的启动，我们推荐以下配置：

```jsonc
{
    "log": "info,runtime=info",    // 日志等级配置，若不需要runtime的打印可以配置为 "log": "info,runtime=warn",
    "name": "<Your-Node-Name>",    // 在监控台 https://stats.chainx.org 中显示的名称
    "port": 20222,                 // 节点的 p2p 端口
    "ws-port": 8087,               // 节点的 Websocket 端口
    "rpc-port": 8086,              // 节点的 RPC 端口
    "rpc-external": true,          // true 代表该RPC端口开放给外部访问，建议只能提供服务同步节点开启，验证节点及不提供服务的同步节点建议关闭
    "ws-external": true,           // true 代表该RPC端口开放给外部访问，建议只能提供服务同步节点开启，验证节点及不提供服务的同步节点建议关闭
    "base-path": "<Your-DB-Path>", // chainx 数据存放路径
    "other-execution": "NativeElseWasm",
    "syncing-execution": "NativeElseWasm",
    "block-construction-execution": "NativeElseWasm",
    "importing-execution": "NativeElseWasm",
    "bootnodes": [
        // 填写引导节点，ChainX已经内置了一些种子节点，一般无需填写
    ]
}
```

- `name`: 在 telemetry 显示的节点名
- `base-path`: 数据存放路径，注意与启动节点步骤1中的相对应

准备好启动参数的配置文件后，可采用以下命令启动同步节点：

```bash
./chainx --config=$(pwd)/config.json
```

若希望运行在后台，可使用如下命令进行启动:

```bash
nohup ./chainx --config=$(pwd)/config.json > chainx.log 2>&1 &
```

待同步节点同步到最新高度后，即可参考下一节内容，暂停同步节点并修改同步节点的启动配置文件，添加 `validator`, `validator-name` 等参数，再次启动节点进入验证人模式。

#### 2. 启动验证者节点

##### 1. 节点账户公钥生成与注册

1. 如果还没有账户，请先到钱包 https:://wallet.chainx.org 点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`。
2. 在钱包投票选举页，点击注册节点，设置 **节点名称**，比如 NodeABC，注意：该节点名为即后面启动文件要用到的 `validator-name` 字段。
3. 在投票选举页的候选节点里，可以看到你的节点处于 **退选** 状态。此时需要

##### 2. 生成出块公钥

如果还不了解节点账户公钥和出块公钥的区别，请[点击这里](https://github.com/chainx-org/ChainX/wiki/%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5#%E8%8A%82%E7%82%B9%E8%B4%A6%E6%88%B7%E5%85%AC%E9%92%A5%E5%92%8C%E5%87%BA%E5%9D%97%E5%85%AC%E9%92%A5)。以下内容均认为读者已经了解了账户公钥和出块公钥的区别。

ChainX 建议使用 `chainx` 执行文件通过以下命令生成 `keystore`，即生成出块地址公钥（session_key）：

```bash
# 输入密码后即可断开节点
$ ./chainx --keystore-path=<keystore路径> -i --base-path=<数据存放路径>
Password:
Repeat again:
......
2019-05-15 11:26:50.510 INFO Roles: FULL
######### 下行打印即为节点生成的 keystore 信息，这串公钥即为出块地址公钥（session_key）
2019-05-15 11:26:50.514 INFO Generated a new keypair for keystore: 593a11d6d5930ab2e68fa5d07082ba0102fc7740eee38b79b2793d7d34a2442a (5E5hNNEi...)
......
2019-05-15 11:26:50.610 INFO [runtime|xrml_xdex_spot] [add_trading_pair] currency_pair: CurrencyPair: SDOT/PCX, point_precision: 4, tick_precision: 2, price: 100000, online: true
########## 看到下行打印后即可使用 <Ctrl-C> 断开节点
2019-05-15 11:26:50.624 INFO Initializing Genesis block/state (state: 0x9499…b6c3, header-hash: 0xdb82…e55d)
....
```

上述命令会在指定的 `<keystore存放路径>` 下生成一个文件名为 **节点出块地址公钥** 的文件，形如: `d41bf5ce34ec4503e633ed0d003356e111186954ce6143d0cfeb6d0a3e51e662`，该文件名即为出块地址公钥（sessin_key）。

**文件中的内容为这个公钥对应的助记词，请注意该助记词体系与钱包中使用的助记词体系不一样，因此无法将该助记词导入到钱包中生成相同的公钥进行使用。**

- keystore助记词体系为：助记词+密码 ==> 私钥
- 在线钱包中的助记词体系为：助记词 ==> 私钥

注意要保存好以下信息，启动节点时需要用到这些信息:

-  `keystore-path`
-  `base-path`
- 输入的 `password`,

该节点出块地址公钥（session_key）粘贴到浏览器 https://scan.chainx.org 搜索框获得该公钥对应的地址，该地址将用于更新出块地址。

 ![image](https://user-images.githubusercontent.com/8850248/57749106-4fe2f700-770f-11e9-93e1-50921b6b769c.png)

##### 3. 修改启动配置文件启动验证者出块节点

对于验证者节点，我们建议如下配置：

```jsonc
{
  "validator": true, //  验证者节点必须为 true
  "rpc-external": false, // 验证者节点建议关闭对外的rpc端口
  "ws-external": false, // 验证者节点建议关闭对外的ws端口
  "log": "info,runtime=info",
  "port": 20222,
  "ws-port": 8087,
  "rpc-port": 8086,
  "other-execution": "NativeElseWasm",
  "syncing-execution": "NativeElseWasm",
  "block-construction-execution": "NativeElseWasm",
  "bootnodes": [],
  "name": "Your-Node-Name",             // 在节点浏览器中显示的节点名
  "validator-name": "Your-Validator-Name", // 注册节点时使用的名称
  "base-path": "<步骤1中指定的数据存放路径>",
  "keystore-path": "<步骤1中指定的keystore路径>",
  "keystore-password": "<步骤1中生成的keystore时使用的密码>"
}
```

在配置文件中一些关键配置解释：

- `name`: 在 telemetry 显示的节点名
- `validator-name`: 注册节点时使用的名称，**请注意一定要与“验证者公钥生成与注册”步骤中注册的节点名字相同**
- `base-path`: 数据存放路径，注意与启动节点步骤1中的相对应
- `keystore-path`: 启动节点步骤1中指定的`keystore`路径
- `keystore-password`: 启动节点步骤1中输入的 `password`
- `validator`: 如果设置为 `true`, 则启动验证节点，否则为同步节点。

接下来可以通过命令启动节点：

```bash
./chainx --config=$(pwd)/config.json
```

若希望运行在后台，可使用命令进行启动

```bash
nohup ./chainx --config=$(pwd)/config.json > chainx.log 2>&1 &
```

启动后请观察日志：

```bash
2019-05-15 11:37:58.824 INFO load key from keystore. key:<该公钥为keystore的公钥，也就是出块地址公钥（session_key）>
2019-05-15 11:37:58.824 INFO Using authority key <该公钥为keystore的公钥，也就是出块地址公钥（session_key）>
```

**请保证上面日志里的公钥为出块地址公钥（session_key），若不是，会导致后续无法出块**

###### 如何更新出块地址

在钱包投票选举页，点击更新节点，在出块地址处填写第一步中得到的出块地址公钥(session_key):

![image](https://user-images.githubusercontent.com/8850248/57748969-bfa4b200-770e-11e9-843f-c841f3b887e6.png)

**更新出块地址完成后，验证出块地址与生成的节点公钥是否匹配：**

首先进入区块链浏览器的节点详情页面，点击出块地址, 然后会看到该地址对应的公钥，检查是否与出块地址公钥（session_key）相一致。如果一致，则正确，否则出块地址不正确。

![image](https://user-images.githubusercontent.com/8850248/57750530-38a70800-7715-11e9-969a-a7277739be6f.png)

其他节点运维相关见：[ChainX 节点运维](devops)

### 4. 出块节点正常运行后其他工作

确认出块节点成功运行后，如果节点处于退选状态，则需要在投票选举页点击更新节点，使节点进入参选状态:

- 出块地址
- 网址
- 简介
- 选择“参选”。

请求社区投票至前 100 名，每 60 分钟（1800块的整数倍）会进行一次验证人选举，如果在前 100 名就可以出块了，如果看到命令行打印 `Prepared block for proposing at 6467`之类的，就代表在出块了 。

**如果由于节点部署不当等导致的节点掉线，系统会在每个分红周期惩罚节点奖池**，漏一个块的罚金是出一个块奖励的 3 倍，罚金从节点奖池扣除。如果奖池被惩罚至0，则会自动强制退选，节点需检查部署情况后，再次更新节点至参选状态，等待下一轮换届。
