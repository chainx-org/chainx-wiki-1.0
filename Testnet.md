## ChainX v0.9.8 公开测试网已结束测试

ChainX v0.9.8 公开测试网已结束测试，请参与这次测试的节点停止v0.9.8的进程，并删除所有数据

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

## ChainX v0.9.9 公开测试网

1. 访问在线钱包 [wallet.chainx.org](https://wallet.chainx.org) 和 浏览器[scan.chainx.org](https://scan.chainx.org) 。

2. 点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`，导出**账户私钥**形如 `0x30530...c95aee43`。

3. 首先在资产页领取 1 个 PCX 测试币，以便能够发起交易，进行注册，更新节点等操作。

4. 在投票选举页，点击注册节点，设置 **节点名称**，比如 NodeABC。

5. 在投票选举页的候选节点里，可以看到你的节点处于 **退选** 状态。

6. 在 https://github.com/chainx-org/ChainX/releases/tag/v0.9.9 下载 v0.9.9 测试网 ChainX 二进制 `chainx`, 目前仅支持 Ubuntu 16.04+ 或 macOS。

7. 启动节点。

    ```bash
    # 如果之前运行过老版本的节点，请先删除老数据
    rm -rf 数据存放路径

    # 启动验证人节点
    ./chainx --key=账户私钥 --validator-name=节点名称 --name=监控台名称 --base-path=数据存放路径 --validator --chain=local --pruning archive --block-construction-execution=NativeElseWasm --other-execution=NativeElseWasm --syncing-execution=NativeElseWasm

    # 启动同步节点
    ./chainx --name=监控台名称 --base-path=数据存放路径 --chain=local --pruning archive --block-construction-execution=NativeElseWasm --other-execution=NativeElseWasm --syncing-execution=NativeElseWasm
    ```

    - `--key`: 指定验证人节点的出块私钥
    - `--name`: 这里的 name 指的是在 telemetry 显示的名字，并非节点注册时指定的 name。
    - `--validator-name`: validator-name 填的是节点注册时指定的 name， 必须和 `--key` 相匹配。

    待节点部署完毕，并在监控台等待自己的节点同步到最新，监控台地址:

    - https://telemetry.polkadot.io/#/ChainX%20V0.9.9
    - https://stats.chainx.org/#/ChainX%20V0.9.9

8. 在投票选举页，点击更新节点，填写:

    - 出块地址(**账户地址**)
    - 网址
    - 简介
    - 选择“参选”。

9. 请求社区投票至前 100 名，每 10 分钟（300块的整数倍）会进行一次验证人选举，如果在前 100 名就可以出块了，如果看到命令行打印 Prepared block for proposing at 6467之类的，就代表在出块了 。

10. 如果由于节点部署不当等导致的节点掉线，系统会逐步惩罚节点奖池，漏一个块的罚金是出一个块奖励的 3 倍，罚金从节点奖池扣除。如果奖池被惩罚至0，会自动强制退选，节点需检查部署情况后，再次更新节点至参选状态，等待下一轮换届。

## ChainX v0.9.9 信托测试

ChainX v0.9.9 将会进行信托相关功能的测试。当前ChainX上只有Bitcoin部分需要信托功能，因此本次测试只针对Bitcoin信托:

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

**本次信托测试预计2019年4月26日下午3点（北京时间）进行第一届信托切换（第0届信托为ChainX），由ChainX指定第一届信托参与者，条件如以上介绍。**

成为信托候选的相关文档：（后续更新）

## CHANGELOG

- ~v0.9.8~
- ~v0.9.7~
- ~v0.9.6~
- ~v0.9.5~
- ~v0.9.4~
- ~v0.9.3~
