The ChainX mainnet has been up and running since May 25, 2019.

Useful links:

- online wallet: https://wallet.chainx.org
- Blockchain explorer: https://scan.chainx.org
- Telemetry: https://stats.chainx.org/#/ChainX and https://telemetry.polkadot.io/#list/ChainX

## 1. Download ChainX program

Currently ChainX is not open source yet. You have two ways to get the ChainX program at present:

- the released binary at https://github.com/chainx-org/ChainX/releases/tag/v1.0.0
- using docker(WIP)

## 2. Run a syncing node

It's highly recommended to use a JSON as the `chainx` configuration file by passing `--config`. It is not secure to pass arguments from the command line directly, especially for the `--key` option.

See all the `FLAGS` and `OPTIONS` via `chainx --help`.

`config.json`:

```jsonc
{
    "log": "info,runtime=info",    // log level, set to "info,runtime=warn" if you don't want the runtime logging
    "name": "<Your-Node-Name>",    // name displayed at https://stats.chainx.org/#/ChainX and https://telemetry.polkadot.io/#list/ChainX
    "port": 20222,                 // p2p port
    "ws-port": 8087,               // Websocket port
    "rpc-port": 8086,              // RPC port
    "rpc-external": true,          // accessible by external
    "ws-external": true,           // accessible by external
    "base-path": "<Your-DB-Path>", // where is the DB stored
    "other-execution": "NativeElseWasm",
    "syncing-execution": "NativeElseWasm",
    "block-construction-execution": "NativeElseWasm",
    "importing-execution": "NativeElseWasm",
    "bootnodes": [
    ]
}
```

Run a syncing node given the above config:

```bash
./chainx --config=$(pwd)/config.json
```

Then you could see your node at https://stats.chainx.org/#/ChainX and https://telemetry.polkadot.io/#list/ChainX .

## 3. Run a validator node

Before running a validator node, you ought to make sure the syncing node is fully synced.

### 1. Generate session key

Every validator has two keys:

- `validator_key`: key of your node account.
- `session_key`: key for block authoring, you can update it as you wish.

To generate a `session_key` by creating a keystore interactively:

```bash
$ ./chainx --keystore-path=<keystore-path> -i --base-path=<DB-path>
Password:
Repeat again:
......
2019-05-15 11:26:50.510 INFO Roles: FULL
######### This indicates that you have generated the keystore successfully.
2019-05-15 11:26:50.514 INFO Generated a new keypair for keystore: 593a11d6d5930ab2e68fa5d07082ba0102fc7740eee38b79b2793d7d34a2442a (5E5hNNEi...)
......
2019-05-15 11:26:50.610 INFO [runtime|xrml_xdex_spot] [add_trading_pair] currency_pair: CurrencyPair: SDOT/PCX, point_precision: 4, tick_precision: 2, price: 100000, online: true
########## You can stop the process by <Ctrl-C> when you see this line.
2019-05-15 11:26:50.624 INFO Initializing Genesis block/state (state: 0x9499…b6c3, header-hash: 0xdb82…e55d)
....
```

Ensure you have saved these information:

- `keystore-path`
- `base-path`
- The password you just inputed.

### 2. Modify `config.json` to enable validator mode

Now, you need to add `validator` and `validator-name` options to run as a validator node.

```jsonc
{
  "validator": true,                       // true for validator mode
  "validator-name": "Your-Validator-Name", // name used when registering your node in https://wallet.chainx.org
  "name": "Your-Node-Name",                // any nickname displayed at https://stats.chainx.org/#/ChainX and https://telemetry.polkadot.io/#list/ChainX
  "rpc-external": false,                   // false is recommended for validator node
  "ws-external": false,                    // false is recommended for validator node
  "log": "info,runtime=info",
  "port": 20222,
  "ws-port": 8087,
  "rpc-port": 8086,
  "other-execution": "NativeElseWasm",
  "syncing-execution": "NativeElseWasm",
  "block-construction-execution": "NativeElseWasm",
  "base-path": "<DB-path in the last step>",
  "keystore-path": "<keystore-path in the last step>",
  "keystore-password": "<password you inputed in the latest step>",
  "bootnodes": []
}
```

Notes:

- `name`: this can be anything you like, used to be displayed in the telemetry only.
- `validator-name`: this is the name you used when registering a node from https://wallet.chainx.org

```bash
./chainx --config=$(pwd)/config.json
```
