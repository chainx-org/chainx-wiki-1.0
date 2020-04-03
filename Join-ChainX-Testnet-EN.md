# Join ChainX Testnet

Before starting the following steps, please refer to the content of joining the main network, for ChainX’s testnet is the same as the main network in terms of the functionalities.

Notes for ChainX testnet:

1. It has a different address from the mainnet, that is, the same private key has different addresses under the ChainX main net and the testnet.

2. Bitcoins on ChainX’s testnet adopts Bitcoin Testnet, instead of other bitcoin test networks.

**The current ChainX testnet is ChainX Testnet Taoism** , which is the testnet-taosim. The smart contract is avaliable since this version.

Please download the pre-release version ChainX Testnet Taoism v1.1.0. download link https://github.com/chainx-org/ChainX/releases/tag/v1.1.0-alpha

Testnet infrastructures:

* Block explorer: https://testnet.scan.chainx.org.cn
* Api: https://testnet.api.chainx.org.cn/
* Telemetry: [https://stats.chainx.org/#list/ChainX%20Testnet%20Taoism](https://stats.chainx.org/#list/ChainX Testnet Taoism)

## Run ChainX testnet nodes

You can run the testnet node using the binary of mainnet since v1.0.4 with some modifications to the `config.json`.

ChainX testnet bootstrap configuration `config.json`:

```
{
    "chain": "testnet", // When specified to testnet, the testnet’s genesis is configured to start the node.
    // Please refer to the mainnet config.json for more detailed explanations.
    "bootnodes": [
        // Since the testnet does not have a built-in seed node, users who start the test net must fill in the seeds first.
    ]
}
```

The seed node currently provided by ChainX:

```
"bootnodes":[
    "/ip4/120.27.210.87/tcp/31128/p2p/QmYvnWNdryPM4aKTcjZLMittkA7KTXi3fMSSQ7aWytFi8v"
]
```

The starting method is also the same as the mainnet.

### Connect to testnet from wallet

Please refer to the ChainX new wallet or ChainX old wallet for more information.

Please note when setting up a wallet, the new wallet requires the ssh certificate and domain name to connect the node, while old wallet does not.

### Testnet nodes provided by ChainX

ChainX currently offers several nodes (available to old wallets):

* Node 1/2: websocket: `101.37.76.83:8087/8187`, rpc: `101.37.76.83:8086/8186`
* Node 3/4: websocket: `120.27.210.87:8087/8187`, rpc: `120.27.210.87:8086/8186`

The domain name for the testnet node connection (available to both new and old wallets):

* Wss://testnet.chainx.org.cn/ws

### ChainX testnet api

* Https://testnet.api.chainx.org.cn/
