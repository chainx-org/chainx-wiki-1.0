# ChainX Dev 模式

ChainX 可以提供Dev（开发者模式）启动，用于**本机进行**调试使用。如用户测试合约等功能。

~~**当前ChainX Dev请下载 v1.0.7 的pre-release版本，正式版未发布。**[下载最新版本 ChainX v1.0.7-beta.1](https://github.com/chainx-org/ChainX/releases/tag/v1.0.7-beta.1).~~ 

ChainX 已开源，请直接 Clone https://github.com/chainx-org/ChainX 执行 `cargo build --release` 进行本地编译, 编译好的可执行文件在 `./target/release/chainx`。

ChainX对于Dev模式默认提供了4个验证人：

* Alice
  - 私钥：0xabf8e5bdbe30c65656c0a3cbd181ff8a56294a69dfedd27982aace4a76909115
  - 公钥：0x88dc3417d5058ec4b4503e0c12ea1a0a89be200fe98922423d4334014fa6b0ee

* Bob
  - 私钥：0x3b7b60af2abcd57ba401ab398f84f4ca54bd6b2140d2503fbcf3286535fe3ff1
  - 公钥：d17c2d7823ebf260fd138f2d7e27d114c0145d968b5ff5006125f2414fadae69

* Charlie
  - 私钥：0x072c02fa1409dc37e03a4ed01703d4a9e6bba9c228a49a00366e9630a97cba7c
  - 公钥：0x439660b36c6c03afafca027b910b4fecf99801834c62a5e6006f27d978de234f

* Dave
  - 私钥：0x771f47d3caf8a2ee40b0719e1c1ecbc01d73ada220cf08df12a00453ab703738
  - 公钥：0x5e639b43e0052c47447dac87d6fd2b6ec50bdd4d0f614e4299c665249bbd09d9

其中**Alice默认参选**，另3个处于未参选状态。

注意！以Dev模式启动的节点没有提供正常的比特币充提功能，请勿将测试网比特币充值进来

## 最简模式启动

### 启动节点

**注意：这种启动方式默认使用的Alice私钥，启动单验证者节点**

```bash
./chainx --dev --default-log -d <指定数据目录> --log=runtime=debug --no-telemetry --block-construction-execution=native --other-execution=native
```

即可直接启动节点。

其中 `--dev` 将会指定

* --validator
* --validator-name=Alice
* --key=//Alice

三个参数，意味者将会使用Alice为私钥启动节点。

`--default-log`表示将日志直接打印到控制台中，而不是输入到日志文件中。`-d`将会指定数据目录，若不指定，将会位于默认目录当中（与Substrate默认配置一致）。`--log=runtime=debug`将指定Runtime部分的日志以debug的级别进行打印。`--no-telemetry`表示不向监控台汇报数据。

`--block-construction-executio`和`--other-execution`指定为`native`表示

启动的节点使用默认端口，请检查一下端口是否被占用，否则请指定其他端口：

* p2p：20222 ，使用`--port`指定
* rpc：8086，使用`--rpc-port`指定
* websocket：8087，使用`--ws-port`指定

一般情况下启动单验证者已经足够绝大部分调试需求。注意如果调试的部分和Runtime相关的话，建议将Runtime的log设置为debug级别，即`--log=runtime=debug`

若需要删除数据，则只需要直接删除启动命令中指定的`-d`参数对应的目录即可。

### 启动钱包

钱包相关操作请参考 [ChainX新钱包](wallet#新钱包) 或者 [ChainX老钱包](wallet#老钱包) 相关操作。

对于新老钱包，均可以直接将Alice的私钥导入，进行相应操作。请注意在ChainX中，需要节点出块数已经超过150个块才会第一次发放PCX资产。

对于ChainX Dev节点，没有API。API是ChainX提供的对于节点网络的公共服务。

## 启动其他节点

若需要调试多节点相关的场景，需要搭建多节点环境。

启动Alice验证者后，可以直接在本机启动其他节点。若需要在其他机器启动，则需要在启动Alice节点的日志中找到网络的种子id:

```bash
2019-11-01 15:04:23.781 INFO Local node identity is: QmTB5QBb4SqEecEksXEB6a35yNgvtTyjYDG3LV43yEEibv
```

然后种子信息即为：

```bash
--bootnodes=/ip4/<Alice节点对外ip>/tcp/<Alice节点p2p端口>/p2p/QmTB5QBb4SqEecEksXEB6a35yNgvtTyjYDG3LV43yEEibv
```

也就是`/p2p`后面的部分与Alice种子相同。

启动其他节点的方式：

启动Bob验证者

```bash
./chainx --chain=dev --key=//Bob --validator-name=Bob --validator -d <目录>  --log=runtime=debug --default-log --bootnodes=<Alice的种子>
```

相对应的，启动Charlie和Dave需要把 `--key`，`--validator-name`改成相应的名字。

## 调试合约

**ChainX的dev默认配置允许合约代码中含有`println`，而ChainX的`Testnet`及`Mainnet`都不允许合约中含有`println`。**

因此开发者可以使用ChainX的dev模式本地对合约进行调试，而在部署到公开测试网`Testnet Taoism`及主网时要把合约代码中的所有`println`移除。

调试合约时，一定要以Native模式启动节点，并且把日志的Runtime部分设置成debug以下的级别

```bash
./chainx --dev --default-log -d <指定数据目录> --log=runtime=debug --no-telemetry --block-construction-execution=native --other-execution=native
```

也就是其中的 `--block-construction-execution` 和 ` --other-execution`都要指定为`Native`或者`NativeElseWasm`，并且把日志的Runtime的debug级别打开。

其中：

* `--block-construction-execution`代表着交易执行时运行的模式
* `--other-execution`代表着rpc调用时运行的模式

以这种模式启动的节点，在合约中含有`println`并被交易或者rpc调用时，在日志中会出现以：

```bash
2019-11-01 15:04:23.781 Debug [runtime|xrml_xcontracts] [ext_println]|......
```

形式的日志，因此若需要查看合约的`println`的结果，只需要`grep`关键字`ext_println`即可。
