# 加入ChainX测试网

[English Version](https://github.com/chainx-org/ChainX/wiki/Join-ChainX-Testnet-EN)

在开始以下步骤前，请了解 [加入主网 ](Join-ChainX-Mainnet) 的相关内容，ChainX测试网的功能与主网相同。

ChainX的测试网采用：

1. ChainX地址与主网地址的版本不相同，也就是**相同的私钥，在ChainX主网和测试网下的地址是不相同的**
2. ChainX测试网的Bitcoin采用的是比特币测试网，即 `Bitcoin Testnet`，而非其他比特币测试网

**当前ChainX的测试网为 `ChainX Testnet Taoism`**，即测试网-道。从这个版本开始，测试网将会提供智能合约的功能。

**ChainX Testnet Taoism 请下载v1.1.0的pre-release版本**，[最新版本下载链接 ChainX v1.1.0-alpha](https://github.com/chainx-org/ChainX/releases/tag/v1.1.0-alpha).

测试网相关配套设施：

* 测试网浏览器： [https://testnet.scan.chainx.org.cn](https://testnet.scan.chainx.org.cn/)
* 测试网api： [https://testnet.api.chainx.org.cn/](https://testnet.api.chainx.org.cn/)
* 测试网监控台：[https://stats.chainx.org/#list/ChainX%20Testnet%20Taoism](https://stats.chainx.org/#list/ChainX Testnet Taoism)

## 运行ChainX测试网节点

ChainX测试网节点自`v1.0.4`版本可使用主网的二进制进行启动，只需要稍许修改配置即可。

ChainX测试网启动配置`config.json`：

```bash
{
	"chain": "testnet", // 当 chain 指定位 testnet时，以测试网的genesis即配置启动节点
	// 其他配置请参照主网配置
	"bootnodes":[  // 由于测试网没有内置种子节点，因此启动测试网的用户请一定填写测试网的种子
	
	]
}
```

**其他配置项请参照主网配置进行修改**

ChainX目前提供的种子节点：

```bash
"bootnodes":[
	"/ip4/120.27.210.87/tcp/31128/p2p/QmYvnWNdryPM4aKTcjZLMittkA7KTXi3fMSSQ7aWytFi8v"
]
```

**启动方式也与主网相同**

### 钱包接入测试网节点

钱包相关操作请参考 [ChainX新钱包](wallet#新钱包) 或者 [ChainX老钱包](wallet#老钱包) 相关操作。

请注意对于将钱包设置为测试网时，**新钱包连接测试网节点需要ssh证书及域名，而老钱包不需要**。

### ChainX 提供的测试网节点

#### 节点连接

ChainX当前提供了几个测试网节点（老钱包可连）：

* 节点1/2：websocket: `101.37.76.83:8087/8187`,  rpc: `101.37.76.83:8086/8186`
* 节点3/4：websocket: `120.27.210.87:8087/8187`,  rpc: `120.27.210.87:8086/8186`

测试网节点连接的域名（新老钱包均可连）：

* `wss://testnet.chainx.org.cn/ws`

#### ChainX测试网的api

* https://testnet.api.chainx.org.cn/

-------------

#### 内部新测试网

- 47.114.131.193
  - RPC: 8086, 8186
  - WS: 8087, 8187

- 47.114.150.67
  - RPC: 8086, 8186
  - WS: 8087, 8187
