## ChainX v0.9.9 公开测试网已结束测试

ChainX v0.9.9 公开测试网已结束测试，请参与这次测试的节点停止v0.9.9的进程，并删除所有数据

* 若启动时指定了`-d`或`--base-path`指定了目录，则将这个目录完全删除即可

* 若启动时没有指定路径，则删除：

    * Linux(ubuntu):

        ```bash
        $ rm -rf $HOME/.local/share/ChainX
        ```

    * macOS:

        ```bash
        $ rm -rf $HOME/Library/Application\ Support/ChainX
        ```

## ChainX v0.9.10 公开测试网

chainx当前已经提供了

- 在线钱包 [https://wallet.chainx.org](https://wallet.chainx.org) 
- 浏览器[https://scan.chainx.org](https://scan.chainx.org) 
- 监控台[https://stats.chainx.org](https://stats.chainx.org/)

进行ChainX的相关操作，下文以**“钱包”**，**“浏览器”**和**“监控台”**进行描述。

### 用户

1. 进入钱包后，点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`。
2. 即可开始使用ChainX相关功能。由于ChainX与以太坊模型类似，使用功能时需要付出手续费，因此需要初始费用
   1. 充值一定BTC后会直接获取一定量的PCX进行操作
   2. 从交易所购买或别人转账

### 节点

节点分为两类

- 同步节点
- 验证节点

### [使用 Docker 运行节点](Docker)

### 使用 release binary 运行节点

#### 1. 启动节点准备

1. 删除老的数据

   ```bash
   # 如果之前运行过老版本的节点，请先删除老数据
   rm -rf 数据存放路径
   ```

2. 下载节点：

   在 https://github.com/chainx-org/ChainX/releases/tag/v0.9.10 下载 v0.9.10 测试网 ChainX 二进制 `chainx`，目前仅支持 Ubuntu 16.04+ 或 macOS。

   v0.9.10 我们已经进行了一次链上升级，目前的最新版本是: chainx-v0.9.10-x86_64-<....>-onchain-upgrade.tgz, 请直接下载最新的 onchain-upgrade 版本。


3. 准备启动配置文件或启动命令：

   当前ChainX已经提供了**从配置文件读取参数启动节点**的方式，因此

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

   出现的所有命令可以配置在配置文件中，由于目前采用的是`json`作为配置文件的格式，因此其中

    - `FLAGS` 命令：对应于在`json`中value为`true/false`的配置。如：

    ```bash
    {
        "validator": true, # 代表 `./chainx --validator`
    }
    ```

    - `OPTIONS`命令：对应于在`json`中其他配置，如：

    ```bash
    {
        "base-path": "<ChainX数据路径>", # 代表 `./chainx --base-path=<ChainX数据路径>`
    }
    ```

    因此后文使用`json`配置的方式替代使用命令行输入进行举例，若希望使用命令行输入只需要组织成为对应的命令行即可。
   
   配置文件编写好后将该配置文件保存某处，**请做好对应的安全策略**
   
   下文为简便将config.json放置到与 chainx 二进制同级目录下， 文件名为 `config.json`。
   
   ```bash
   $ ls
    ├── chainx
    ├── chainx.md5
    └── config.json
   ```

#### 2. 启动同步节点

对于同步节点的启动，我们推荐以下配置：

```bash
{
    "log": "info,runtime=info", # 日志等级配置，若不需要runtime的打印可以配置为 "log": "info,runtime=warn", 
    "name": "<Your-Node-Name>", # 在 监控台 中显示的名称
    "port": 20222, # 节点的p2p端口
    "ws-port": 8087, # 节点的websocket 端口
    "rpc-port": 8086, # 节点的rpc
    "rpc-external": true, # true 代表该rpc端口开放给外部访问，建议只能提供服务同步节点开启，验证节点及不提供服务的同步节点建议关闭
    "ws-external": true, # true 代表该rpc端口开放给外部访问，建议只能提供服务同步节点开启，验证节点及不提供服务的同步节点建议关闭
    "base-path": "<Your-DB-Path>", # chainx 数据路径
    "other-execution": "NativeElseWasm",
    "syncing-execution": "NativeElseWasm",
    "block-construction-execution": "NativeElseWasm",
    "importing-execution": "NativeElseWasm",
    "bootnodes": [
        # 填写引导节点，ChainX已经在节点内配置设置好了一些引导节点
    ]
}
```

之后可以采用以下命令启动节点：

```bash
./chainx --config=$(pwd)/config.json
```

若希望运行在后台，可使用命令进行启动

```bash
nohup ./chainx --config=$(pwd)/config.json > chainx.log 2>&1 &
```

#### 3. 启动验证者节点

对于验证者节点，为了保护验证者私钥的安全性，ChainX需要验证者提供：

- 验证者公钥（validator_key）
- 出块地址公钥（session_key）

两个概念，验证者公钥与出块地址公钥为绑定关系，通过钱包注册绑定关系，其中我们建议：

* 验证者公钥（validator_key）由钱包生成
* 出块地址公钥（session_key）在启动节点时生成

ChainX强烈建议验证者公钥与出块地址公钥不相同，并建议验证者保护好自己的验证者公钥（validator_key）。因为ChainX的验证者公钥（validator_key）与出块地址公钥（session_key）分离，因此即使出块的机器被黑，暴露了出块地址公钥（session_key）也不会对验证者的资产产生影响。

##### 1. 验证者公钥生成与注册

1. ~首先在资产页领取 1 个 PCX 测试币，以便能够发起交易，进行注册，更新节点等操作~。 添加群秘微信 469305052 进群领取测试币。
2. 进入钱包后，点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`。
3. 在钱包投票选举页，点击注册节点，设置 **节点名称**，比如 NodeABC。该节点名为`validator-name`
4. 在投票选举页的候选节点里，可以看到你的节点处于 **退选** 状态。

##### 2. 启动验证者的出块节点

1. ChainX 建议使用`chainx`执行文件通过以下命令生成`keystore`，生成出块地址公钥（session_key）：

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

    上述命令会在指定的` <keystore存放路径>`下生成一个文件名为 **节点出块地址公钥** 的文件，形如: `d41bf5ce34ec4503e633ed0d003356e111186954ce6143d0cfeb6d0a3e51e662`，该文件名即为出块地址公钥（session_key）。

   **文件中的内容为这个公钥对应的助记词，请注意该助记词体系与钱包中使用的助记词体系不一样，因此无法将该助记词导入到钱包中生成相同的公钥进行使用**

   该助记词体系为：

   >  助记词+密码 -> 私钥

   钱包中的助记词体系为：

   > 助记词 -> 私钥


   - 注意要保存好以下信息，启动节点时需要用到这些信息

        -  `keystore-path`
        -  `base-path`
        - 输入的 `password`,
   - 该节点出块地址公钥（session_key）粘贴到浏览器 https://scan.chainx.org 搜索框获得该公钥对应的地址，该地址将用于下一步更新出块地址。   ![image](https://user-images.githubusercontent.com/8850248/57749106-4fe2f700-770f-11e9-93e1-50921b6b769c.png)

   2. 启动验证者节点

      对于验证者节点，我们建议如下配置：

      首先需要准备好 JSON 的配置文件, [下载配置文件模板](https://gist.github.com/liuchengxu/3b3ed4ce027e39fc89b5a5a6c289bfaf)，可通过以下方式下载

      ```bash
      $ curl https://git.io/fjlZU -o config.json
      ```

      下载下来的文件中内容如下：

      ```bash
      {
          "validator": true, # 验证者节点必须为 true
          "rpc-external": false, # 验证者节点建议关闭对外的rpc端口
          "ws-external": false, # 验证者节点建议关闭对外的ws端口
          "log": "info,runtime=info",
          "port": 20222,
          "ws-port": 8087,
          "rpc-port": 8086,
          "other-execution": "NativeElseWasm",
          "syncing-execution": "NativeElseWasm",
          "block-construction-execution": "NativeElseWasm",
          "bootnodes": [],
          "name": "Your-Node-Name",
          "validator-name": "Your-Validator-Name", # 注册节点时使用的名称
          "base-path": "<步骤1中指定的数据存放路径>",
          "keystore-path": "<步骤1中指定的keystore路径>",
          "keystore-password": "<步骤1中生成的keystore时使用的命令>"
      }
      ```

      在配置文件中一些关键的配置含义为：

      - `name`: 在 telemetry 显示的节点名
      - `validator-name`: 注册节点时使用的名称，**请注意一定要与“验证者公钥生成与注册”步骤中注册的节点名字相同**
      - `base-path`: 数据存放路径，注意与启动节点步骤1中的相对应
      - `keystore-path`: 启动节点步骤1中指定的`keystore`路径
      - `keystore-password`: 启动节点步骤1中输入的 `password`
      - `validator`: 如果设置为 `true`, 则启动验证节点，否则为同步节点。

      其他参数可在配置文件自行修改。

      接下来可以通过命令启动节点：

      ```bash
      ./chainx --config=$(pwd)/config.json
      ```

      若希望运行在后台，可使用命令进行启动

      ```bash
      nohup ./chainx --config=$(pwd)/config.json > chainx.log 2>&1 &
      ```

      > 其他：
      >
      >若不希望在config.json里面配置私钥以防泄露，可以采用添加在启动时添加`-i`参数进入交互式换进输入密码。方法如下：
      >
      >1. 由于进程无法在后台进行交互式，因此进程需要被其他进程托管。这里以screen为例：
      >
      >   ubuntu:
      >
      >   ```bash
      >   $ sudo apt install screen
      >   ```
      >
      >   其他linux发新版类似安装
      >
      >2. 使用screen托管启动节点
      >
      >   使用screen托管启动：
      >
      >   ```bash
      >   # 注意最后的 -i 参数
      >   $ screen -L -S chainx0.9.10 ./chainx --config=<ConfigPath/chainx.conf> -i 
      >   ```
      >
      >   启动后输入以下命令可退出screen
      >
      >   ```bash
      >   Ctrl-A-D
      >   ```
      >
      >   此时在当前路径下会出现对应的日志文件，若想attach到screen中，可以执行以下命令即可进入screen中:
      >
      >   ```bash
      >   # 列出screen
      >   $ screen -ls
      >   $ screen -r <上面 ls 出现的screen>
      >   ```
      >
      >   需要停止节点，进入screen按 `Ctrl-C` 即可退出
      >
      >   此时在当前路径下会出现对应的日志文件
      >
      >   若想attach到screen中,可以执行:
      >
      >   ```bash
      >   # 列出screen
      >   $ screen -ls
      >   $ screen -r <上面 ls 出现的screen>
      >   ```
      >
      >   即可进入screen中
      >
      >   若需要停止节点,进入screen按 `Ctrl-C` 即可退出
      >
      

      启动后请观察日志：
      
      ```bash
      2019-05-15 11:37:58.824 INFO load key from keystore. key:<该公钥为keystore的公钥，也就是出块地址公钥（session_key）>
      2019-05-15 11:37:58.824 INFO Using authority key <该公钥为keystore的公钥，也就是出块地址公钥（session_key）>
      ```

      **请保证上面日志里的公钥为出块地址公钥（session_key），若不是，会导致后续无法出块**

3. 更新出块地址

   在钱包投票选举页，点击更新节点，在出块地址处填写第一步中得到的出块地址公钥（session_key）：   ![image](https://user-images.githubusercontent.com/8850248/57748969-bfa4b200-770e-11e9-843f-c841f3b887e6.png)

   **更新出块地址完成后，验证出块地址与生成的节点公钥是否匹配：**

   首先进入区块链浏览器的节点详情页面，点击出块地址, 然后会看到该地址对应的公钥，检查是否与出块地址公钥（session_key）相一致。如果一致，则正确，否则出块地址不正确。

   ![image](https://user-images.githubusercontent.com/8850248/57750530-38a70800-7715-11e9-969a-a7277739be6f.png)

  4. 待节点部署完毕，**并在监控台等待自己的节点同步到最新**，监控台地址:

       - https://telemetry.polkadot.io/#/ChainX%20V0.9.10
       - https://stats.chainx.org/#/ChainX%20V0.9.10

       其他节点运维相关见：[ChainX 节点运维](devops)

5. 确认节点成功运行后，在投票选举页，点击更新节点，使节点进入参选状态:

    - 出块地址
    - 网址
    - 简介
    - 选择“参选”。

6. 请求社区投票至前 100 名，每 10 分钟（300块的整数倍）会进行一次验证人选举，如果在前 100 名就可以出块了，如果看到命令行打印 `Prepared block for proposing at 6467`之类的，就代表在出块了 。

7.  **如果由于节点部署不当等导致的节点掉线，系统会逐步惩罚节点奖池**，漏一个块的罚金是出一个块奖励的 3 倍，罚金从节点奖池扣除。如果奖池被惩罚至0，会自动强制退选，节点需检查部署情况后，再次更新节点至参选状态，等待下一轮换届。

## ChainX v0.9.10 信托测试

ChainX v0.9.10 将会进行信托相关功能的测试。当前ChainX上只有Bitcoin部分需要信托功能，因此本次测试只针对Bitcoin信托:

> ”信托“为ChainX上托管对应链的代币的角色，因此”信托“实际上跨链代币的托管者。因此”信托“的参与者必须经过严格的KYC进行验证并公布自己的节点信息，且在验证节点中排名靠前，以保证充足的利益绑定关系防止作恶。
>
> ”信托“一定周期后会进行更换，每一轮更换后的”信托“称为一届信托节点（Session）
>
> ChainX的给予了”信托“极大的灵活性及权利，因此”信托“只可由上一届信托群体选定下一届信托群体，选定的最核心条件为保证下一届信托的可靠性
> 因此判定条件可以通过下面进行筛选，并经过当前届的信托群体的**多签投票**:
> * 节点愿意当选信托
> * 节点排名靠前
> * 节点抵押数较大且投票数较大
> * 节点能灵活操作冷热钱包，有冷钱包管理技术
> * 节点身份已知，且背后有较大利益关系背书（如公司，名望等）
> * 其他有效筛选条件

**本次信托测试时间之后放出**

## CHANGELOG

- ~v0.9.9~
- ~v0.9.8~
- ~v0.9.7~
- ~v0.9.6~
- ~v0.9.5~
- ~v0.9.4~
- ~v0.9.3~
