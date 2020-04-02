# ChainX ink!

[chainx-org/ink](https://github.com/chainx-org/ink) fork 自 [paritytech/ink](https://github.com/paritytech/ink)。 虽然从 ink2.0 开始， ink 框架已经可以方便地添加和使用自定义的 RuntimeType 类型， 对于基于近期 substrate 的链来说，基本不用修改 ink 框架，直接定义支持自身 runtime 的 RuntimeType 即可。但是由于 [chainx-org/ChainX](https://github.com/chianx-org/ChainX) 启动的比较早，底层的编码库仍然使用早期的 codec3.5 而不是最新的 parity-scale-codec, 所以选择 fork [paritytech/ink](https://github.com/paritytech/ink) 添加 `old-codec` 以支持 ChainX.

目前 [chainx-org](https://github.com/chainx-org/ink) 已经支持 PCX 和比特币在合约中的转账，同时可以获取用户的资产信息，开发者可以基于这些功能设计，开发和验证自己的 Dapp 产品。

对于 ChainX 合约平台的开发者而言, 需要注意必须使用 `DefaultXrmlTypes` 进行 ChainX 的合约开发， `DefaultXrmlTypes` 与 默认的 `DefaultRrmlTypes` 的区别在于:

- `DefaultXrmlTypes` 将关联类型 `Balance` 从 `u128` 改为 `u64`.
- 适配关联类型 `Call` 以支持 PCX 转账和将跨链资产划转过去的 XRC20 Token 转回 ChainX 跨链资产, 合约开发者可以基于 PCX 和 ChainX 的跨链资产设计自己的 Dapp。

    #[ink::contract(version = "0.1.0", env = DefaultXrmlTypes)]
    ...
    ```

除了以上两点改动，在合约编写上与 [paritytech/ink](https://github.com/paritytech/ink) 保持一致。

# cargo-contract

[chainx-org/cargo-contract](https://github.com/chainx-org/cargo-contract/) fork 自 [paritytech/cargo-contract](https://github.com/paritytech/cargo-contract), 是一个 `cargo` 命令行插件, 可用于创建和编译 `ink!` 所编写的 `WebAssembly` 智能合约。

```bash
cargo install --git https://github.com/chainx-org/cargo-contract cargo-contract --force
```

具体细节请见 [cargo-contract/README](https://github.com/chainx-org/cargo-contract/blob/master/README.md) 

### ink 合约编写相关资源:

- [substrate-contracts-workshop](https://substrate.dev/substrate-contracts-workshop/#/0/introduction): ink 官方教程
- [polkaworld-org/workshop](https://github.com/polkaworld-org/workshop): polkaworld 合约相关技术讲座

### ChainX 合约集锦

- 智能合约投票 Dapp
