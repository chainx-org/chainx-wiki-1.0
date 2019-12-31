<!-- TOC GFM -->

* [Setup](#setup)
    * [ChainX Binary](#chainx-binary)
    * [Rust Environment for ink!](#rust-environment-for-ink)
    * [ink! CLI](#ink-cli)
* [Creating an ChainX ink! Project](#creating-an-chainx-ink-project)
* [Running a ChainX Node](#running-a-chainx-node)
* [Deploying Your Contract](#deploying-your-contract)
* [Calling Your Contract](#calling-your-contract)

<!-- /TOC -->

## Setup

### ChainX Binary

请前往 [chainx-org/ChainX](https://github.com/chainx-org/ChainX) 下载 [ChainX v1.0.7-alpha](https://github.com/chainx-org/ChainX/releases/tag/v1.0.7-alpha).

### Rust Environment for ink!

安装 Rust 环境， 详情请参考 [rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).

### ink! CLI

安装好 Rust 环境后， 安装方便构建合约项目的命令行工具 `cargo-contract`. 注意，请安装 [chainx-org/cargo-contract ink2.0 branch](https://github.com/chainx-org/cargo-contract/tree/ink2.0), 而不是安装 `paritytech/cargo-contract`。

```bash
$ cargo install --git https://github.com/chainx-org/cargo-contract --branch ink2.0 cargo-contract --force
```

## Creating an ChainX ink! Project

在基本用法上， `chainx-org/cargo-contract` 与 `paritytech/cargo-contract` 一致，区别在于 `chainx-org/cargo-contract` 使用了定制的 `chainx-org/ink`, 而非 `paritytech/ink`。

创建一个 flipper 的示例项目:

```bash
$ cargo-contract new flipper
```

运行测试：

```bash
$ cargo +nightly test
```

编译 WASM 合约文件：

```bash
$ cargo-contract contract build
```

## Running a ChainX Node

启动单节点模式：

```bash
$ ./chainx --dev --default-log -d <指定数据目录> --log=runtime=debug --no-telemetry --block-construction-execution=native --other-execution=native
```

关于本地启动节点的更多内容，可以阅读 wiki [ChainX-Dev](https://github.com/chainx-org/ChainX/wiki/ChainX-Dev), 一般来说调试合约使用单节点模式即可。

## Deploying Your Contract

1. 安装 ChainX 浏览器插件。

2. 在浏览器插件中将网络切换到测试网, 合约功能目前只在测试网开启。

## Calling Your Contract
