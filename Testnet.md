## ChainX v0.9.7 公开测试网

1. 访问在线钱包 [wallet.chainx.org](https://wallet.chainx.org)。

2. 点击创建账户，生成节点账户，**账户地址**形如 `5HbT8...S9yg`，导出**账户私钥**形如 `0x30530...c95aee43`。

3. 首先在资产页领取 1 个 PCX 测试币，以便能够发起交易，进行注册，更新节点等操作。

4. 在投票选举页，点击注册节点，设置 **节点名称**，比如 NodeABC。

5. 在投票选举页的候选节点里，可以看到你的节点处于 **退选** 状态。

6. 在 https://github.com/chainx-org/ChainX/releases/tag/v0.9.7 下载 v0.9.7 测试网 ChainX 二进制 `chainx`, 目前仅支持 Ubuntu 16.04+ 或 macOS。

7. 启动节点。

    ```bash
    # 如果之前运行过老版本的节点，请先删除老数据
    rm -rf 数据存放路径

    # 启动验证人节点
    ./chainx --key=账户私钥 --validator-name=节点名称 --name=监控台名称 --base-path=数据存放路径 --validator --chain=local --pruning archive --block-construction-execution=native --other-execution=native

    # 启动同步节点
    ./chainx --name=监控台名称 --base-path=数据存放路径 --chain=local --pruning archive --block-construction-execution=native --other-execution=native
    ```

    待节点部署完毕，并在监控台等待自己的节点同步到最新，监控台地址:

    - https://telemetry.polkadot.io/#/ChainX%20V0.9.7
    - https://stats.chainx.org/#/ChainX%20V0.9.7

8. 在投票选举页，点击更新节点，填写:

    - 出块地址(**账户地址**)
    - 网址
    - 简介
    - 选择“参选”。

9. 请求社区投票至前 30 名，每 10 分钟（300块的整数倍）会进行一次验证人选举，如果在前 30 名就可以出块了，如果看到命令行打印 Prepared block for proposing at 6467之类的，就代表在出块了 。

10. 如果由于节点部署不当等导致的节点掉线，系统会逐步惩罚节点奖池，如果惩罚只0.33以下，会自动退选，节点需检查部署情况后，再次更新节点至参选状态，等待下一轮换届。

## DOT 所有者和测试用户如何领取 SDOT

首先在[ChainX钱包](https://wallet.chainx.org)中要拥有账户，进入[资产页面](https://wallet.chainx.org/asset)，在 SDOT 栏点击绑定并跳过，会出现具体操作信息。

DOT 所有者和测试用户使用 **支持 Data** 的以太坊钱包，向任意地址(建议为自己的地址)发起任意金额 (建议为0) 的转账交易，并在 **Data** 中输入下方 **十六进制 (Hex)** 信息：`0x35435366663736534b377163575971354d70766f48445652726a57467770787572775575364271773235684b50516979`

交易打包成功后，在具体页面中输入 **交易 ID (Txid/TxHash)**，交易签名验证无误后，即可完成映射并领取 SDOT。

当前 0.9.7 测试网的具体测试用户地址只有以下列出的（每位测试用户拥有 5 个 SDOT）：

- 0x6064d1a20e529ea15b06551e1690c8f50342edf2 (测试时已代为领取)
- 0x4b9d7504014bc1810572979739ea00317c80308a
- 0x0076c0e890cc13e3d74bff8b72ba6a6ef7af1120
- 0xd20375fabc2a8b1716a7d4cab4ecff4cc931c699
- 0x59753c459cbf746b1f9289cb5c23daddd95bb194
- 0xf4d0b7cd43945f27c0f43b8936a93674ae7091ff
- 0x94c90e0a573db26467e0e812090a9220c20edcd3
- 0x2684445d42e93876de6d41ed685081b9ae9bdd31
- 0xd40c1e3c54b1a2443b533b14505b267c1329a25d
- 0x6dea4ea1c5fc6ad3d7e9933665799936a9fdd89c
- 0xfa442b554221380fd87c117bd513b8a414e919fc
- 0xf897a81a6ab5aa5cc24e18c9976b3882cd0f4ccf
- 0x0628dae391a37ccb6ccae7e6b6495c2622d69cda
- 0x39c702b9263a42775ee1a6d6a11e00aa6a91f66b
- 0x5ba3750a16b1f63b2265a571bf8920e218f077a9

PS: 目前 **支持 Data** 的钱包有: imToken、Parity、MyEtherWallet、Jaxx、MyCrypto、Trust、Bitpie、Coinomi 等。

## CHANGELOG

- ~v0.9.6~
- ~v0.9.5~
- ~v0.9.4~
- ~v0.9.3~