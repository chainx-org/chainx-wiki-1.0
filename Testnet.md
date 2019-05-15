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

1. 访问在线钱包 [wallet.chainx.org](https://wallet.chainx.org) 和 浏览器[scan.chainx.org](https://scan.chainx.org) 。

2. 点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`，导出**账户私钥**形如 `0x30530...c95aee43`。

3. ~首先在资产页领取 1 个 PCX 测试币，以便能够发起交易，进行注册，更新节点等操作~。 添加群秘微信 469305052 进群领取测试币。

4. 在投票选举页，点击注册节点，设置 **节点名称**，比如 NodeABC。

5. 在投票选举页的候选节点里，可以看到你的节点处于 **退选** 状态。

6. 在 https://github.com/chainx-org/ChainX/releases/tag/v0.9.10 下载 v0.9.10 测试网 ChainX 二进制 `chainx`, 目前仅支持 Ubuntu 16.04+ 或 macOS。

7. 启动节点。

    1. 生成节点 keystore

    ```bash
    # 输入密码后即可断开节点
    $ ./chainx --keystore-path=<keystore路径> -i --base-path=<数据存放路径>
    Password:
    Repeat again:
    ......
    2019-05-15 11:26:50.510 INFO Roles: FULL
    ######### 下行打印即为节点生成的 keystore 信息
    2019-05-15 11:26:50.514 INFO Generated a new keypair for keystore: 593a11d6d5930ab2e68fa5d07082ba0102fc7740eee38b79b2793d7d34a2442a (5E5hNNEi...)
    ......
    2019-05-15 11:26:50.610 INFO [runtime|xrml_xdex_spot] [add_trading_pair] currency_pair: CurrencyPair: SDOT/PCX, point_precision: 4, tick_precision: 2, price: 100000, online: true
    ########## 看到下行打印后即可使用 <Ctrl-C> 断开节点
    2019-05-15 11:26:50.624 INFO Initializing Genesis block/state (state: 0x9499…b6c3, header-hash: 0xdb82…e55d)
    ....
    ```

    上述命令会在指定的 <keystore存放路径> 生成一个文件名为**节点出块地址公钥**的文件，形如: d41bf5ce34ec4503e633ed0d003356e111186954ce6143d0cfeb6d0a3e51e662。

    - 注意要保存好 `keystore-path`, `base-path`, 和输入的 password, 启动节点时需要用到这些信息。

    - 该节点出块公钥粘贴到浏览器 https://scan.chainx.org 搜索框获得该公钥对应的地址，该地址将用于下一步更新出块地址。

      ![image](https://user-images.githubusercontent.com/8850248/57749106-4fe2f700-770f-11e9-93e1-50921b6b769c.png)

    2. 更新出块地址

        用上一步得到的地址更新出块地址：

        ![image](https://user-images.githubusercontent.com/8850248/57748969-bfa4b200-770e-11e9-843f-c841f3b887e6.png)

    3. 准备节点启动配置文件

       ```bash
       # 如果之前运行过老版本的节点，请先删除老数据
       rm -rf 数据存放路径
       ```

       首先需要准备好 JSON 的配置文件, [下载配置文件模板](https://gist.github.com/liuchengxu/3b3ed4ce027e39fc89b5a5a6c289bfaf):

       ```json
       {
           "validator": true,
           "rpc-external": false,
           "ws-external": false,
           "log": "info,runtime=info",
           "port": 20222,
           "ws-port": 8087,
           "rpc-port": 8086,
           "other-execution": "NativeElseWasm",
           "syncing-execution": "NativeElseWasm",
           "block-construction-execution": "NativeElseWasm",
           "bootnodes": [],
           "name": "Your-Node-Name",
           "validator-name": "Your-Validator-Name",
           "base-path": "<启动节点步骤1中指定的数据存放路径>",
           "keystore-path": "<启动节点步骤1中指定的keystore路径>"
       }
       ```

       比如将JSON配置模板下载到与 chainx 二进制同级目录下， 文件名为 `config.json`。

       ```bash
       $ curl https://git.io/fjlZU -o config.json
       $ ls
        ├── chainx
        ├── chainx.md5 
        └── config.json
       ```

    3. 在配置文件中更新实际的节点信息

        - `name`: 在 telemetry 显示的节点名
        - `validator-name`: 注册节点时使用的名称
        - `base-path`: 数据存放路径，注意与启动节点步骤1中的相对应
        - `keystore-password`: 启动节点步骤1中输入的 password
        - `validator`: 如果设置为 `true`, 则启动验证节点，否则为同步节点。

        其他参数可在配置文件自行修改。

    4. 通过配置文件启动节点

      ```bash
      $ ./chainx --config=$(pwd)/config.json
      ```

      若需要放置后台运行,可执行

      ```bash
      $ nohup ./chainx --config=$(pwd)/config.json > chainx.log 2>&1 &
      ```

    待节点部署完毕，并在监控台等待自己的节点同步到最新，监控台地址:

    - https://telemetry.polkadot.io/#/ChainX%20V0.9.10
    - https://stats.chainx.org/#/ChainX%20V0.9.10

    其他节点运维相关见：[ChainX 节点运维](devops)

9. 在投票选举页，点击更新节点，填写:

    - 出块地址(**账户地址**)
    - 网址
    - 简介
    - 选择“参选”。

10. 请求社区投票至前 100 名，每 10 分钟（300块的整数倍）会进行一次验证人选举，如果在前 100 名就可以出块了，如果看到命令行打印 Prepared block for proposing at 6467之类的，就代表在出块了 。

     如果由于节点部署不当等导致的节点掉线，系统会逐步惩罚节点奖池，漏一个块的罚金是出一个块奖励的 3 倍，罚金从节点奖池扣除。如果奖池被惩罚至0，会自动强制退选，节点需检查部署情况后，再次更新节点至参选状态，等待下一轮换届。

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
