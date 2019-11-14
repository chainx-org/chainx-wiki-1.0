# 加入ChainX测试网

在开始以下步骤前，请了解 [加入主网 ](Join-ChainX-Mainnet) 的相关内容，ChainX测试网的功能与主网相同。

ChainX的测试网采用：

1. ChainX地址与主网地址的版本不相同，也就是**相同的私钥，在ChainX主网和测试网下的地址是不相同的**
2. ChainX测试网的Bitcoin采用的是比特币测试网，即 `Bitcoin Testnet`，而非其他比特币测试网

**当前ChainX的测试网为 `ChainX Testnet Taoism` **，即测试网-道。从这个版本开始，测试网将会提供智能合约的功能。

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

**请下载最新的钱包，链接[下载](https://github.com/chainx-org/chainx-wallet/releases)**

钱包自`v1.0.4`版本之后提供了切换测试网的功能：

![image](https://user-images.githubusercontent.com/5023721/62120182-ae257c00-b2f3-11e9-9e5e-b2c1e65ee0f1.png)

1. 点击钱包右上角设置按钮
2. 选择网络类型中选择“测试网”，选择后，钱包页面会进行重置，此时钱包中的存储将会使用全新的存储，和主网存储隔离
3. 点击“添加节点”，可以添加**测试网节点**的websocket地址和端口，主网节点无法添加
4. 点击“添加api”，可以添加**测试网api**。

若设置节点成功，则该钱包正常使用测试网。

### ChainX 提供的测试网节点

ChainX当前提供了几个测试网节点：

* 节点1/2：websocket: `101.37.76.83:8087/8187`,  rpc: `101.37.76.83:8086/8186`
* 节点3/4：websocket: `120.27.210.87:8087/8187`,  rpc: `120.27.210.87:8086/8186`

**ChainX测试网的api**

* https://testnet.api.chainx.org.cn/

