## 推荐机器配置

以阿里云为例，ChainX 主网推荐配置不低于: CPU 4 核, 内存 4 G, 带宽 10M，服务器费用支出每个月不到 1 K。

## 节点运维

### 1. 日志分割

若一直持续运行日志过大，可以对设置`crontab`的定时任务对日志进行切分，可参考[日志切分](https://blog.csdn.net/shawnhu007/article/details/50971084)编写脚本,或使用`logrotate`等工具

### 2. 使用交互式输入密码

  由于交互无法放置在后台,因此若想在后台启动,并且使用交互式输入密码,需要安装一些工具

  1. 安装 `screen`

     以 Ubuntu为例

     ```bash
     $ sudo apt install screen
     ```

  2. 使用screen托管启动节点

     ```bash
     # 注意最后的 -i 参数
     $ screen -L -S chainx ./chainx --config=<ConfigPath/chainx.conf> -i 
     ```

     启动后按一下命名可退出screen

     ```bash
     Ctrl-A-D
     ```

     此时在当前路径下会出现对应的日志文件

     若想attach到screen中,可以执行:

     ```bash
     # 列出screen
     $ screen -ls
     $ screen -r <上面 ls 出现的screen>
     ```

     即可进入screen中

     若需要停止节点,进入screen按 `Ctrl-C` 即可退出

### 3. 监控节点是否正常出块/同步

节点可以编写脚本监控自己的出块是否正常，主要接口都可以从[RPC](RPC)文档中获得说明

1. 获取当前自己节点的最高块高`chain_getHeader`
2. 获取自己相连的**其他节点**的块高`system_peers`
3. 从官方钱包中提供的rpc接口获取当前的块高

因此监控脚本可以根据2，3获取到的数据，去衡量1中获取的块高是否正常。

请节点根据自己情况设计自己的监控脚本。由于目前libp2p还不够完善，因此我们 **强烈建议** 当发现块高不正常后节点 **主动停止** 自己的节点，已防止后续造成大范围的网络分区。

该问题将会随着substrate与ChainX的升级不断完善，谢谢各位节点理解！

一般情况下若带宽允许的范围，可以将自己的p2p连接上限提升

通过在启动的参数配置文件中设置`in-peers`及`out-peers` 设置连接数上限，带宽允许的情况下设置为40以上为好

```bash
{
	"in-peers": 40,
	"out-peers": 40,
}
```


目前由于substrate p2p 不够完善的原因，会导致部分节点**有可能在运行过程中出现网络分区**，因此当监控脚本发现自己无法同步区块时可以做出以下操作**跳出网络分区**： 

1. 挑选种子

   1. 随机选择以下种子列表的**一个**：（不推荐，因为种子节点可能已经连满了，或者失效）

      ```bash
      "/ip4/47.96.134.203/tcp/20222/p2p/QmNzDfC4qTC9B5r8W7XxAR4n3qSF7SqgF6FsNGEWCL4foY",
      "/ip4/47.110.252.22/tcp/20222/p2p/Qmcwx277eRTNmeK2iTLrGDHqRwYW8FxL8ppwRXM3SnbUvS",
      "/ip4/47.96.97.52/tcp/20222/p2p/QmXABmmDgTT75o2UVCKS8iaYh75zPL5by3iSfS8msSWwEo",
      "/ip4/47.110.14.60/tcp/20222/p2p/QmR2Jdd4V93yzTDyX5gKQ7PE4M2KsbK3vjC8RvXCnzqHU6",
      "/ip4/47.110.14.60/tcp/30333/p2p/QmfDqzUo64DDDeinmWDif2umjgJpKyunYkKC81XNZ9Cwzt"，
      ```

   2. **或**通过过 [https://telemetry.polkadot.io/#/ChainX ](https://telemetry.polkadot.io/#/ChainX )，点击右边设置按钮，打开最下面一栏的`NetworkState`，然后回到主界面，此时每一个节点的最右边会出现网络链接的按钮。选择一个目前块高度正常的节点点开会跳转到一个新页面，是一个`json`数据。（推荐）

      - 其中

        ```bash
        {
            peerId: "xxxxxxxxxxxxxxxxxxxxxxxx",
            listenedAddresses: [
            "/ip4/172.16.35.147/tcp/20222",
            "/ip4/127.0.0.1/tcp/20222"
            ],
            externalAddresses: [
            "/ip4/<ip>/tcp/20222",  # ipv4
            "/ip6/<ip>/tcp/20222" # ipv6
            ]
        	...
        }
        ```

      - `peerId` 代表节点的网络标示（存储于`networks`文件夹中），`externalAddresses`代表对外监听的ip和端口

      - 因此种子节点由 `externalAddresses`与`peerId`构成，为：

        ```bash
        seed = <externalAddresses(ipv4部分)>/p2p/<peerId>
        # e.g.
        seed = /ip4/172.16.176.18/tcp/20222/p2p/QmRaP225FNXoyB7WE8twinY8B6dVJXyGsmEYFA1Fc54rw1
        ```

2. 挑选出**一个**种子节点后，停止节点，然后以一下命令启动：

   ```bash
   ./chainx --config=<你的配置文件名，如config.json> --in-peers=1 --out-peers=1 --bootnodes=<刚才挑选的种子节点，不带引号> 
   ```

   这样可以很快跳出网络分区

3. 跳出网络分区后，停止节点，以正常方式继续启动即可。

4. 推荐：刚才在第1步中挑选的种子节点可以配置到启动的参数配置文件中

   ```bash
   {
       "bootnodes": [
            "<种子1>",  # 注意这里需要用引号包起来
            "<种子2>",
            "<种子3>"
       ]
   }
   ```

   有能力的节点可以通获取到一些其他节点的网络信息，制定网络拓扑，找出和种子节点连接最近的节点，并**将他们的连接信息制作成种子节点**加入自己的种子列表当中，这样可以减少被网络分区的概率！

### 4. 节点退选，连接，云服务商，vps选择等问题

* 关于防止退选问题，更详细请查阅[FAQ#漏块](FAQ#漏块)，[FAQ#检查漏块的可能方式](FAQ#检查漏块的可能方式)，[FAQ#节点防止退选的方法](FAQ#节点防止退选的方法)
* 关于阿里云香港
  * 目前ChainX有一些服务放于阿里云（国际）香港上，但是至今已出现了**多次网络访问出现短暂性异常**的情况，因此建议使用阿里云（国际）香港的节点引起警惕