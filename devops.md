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

请节点根据自己情况设计自己的监控脚本。



目前由于substrate p2p 不够完善的原因，会导致部分节点**有可能在运行过程中出现网络分区**，因此当监控脚本发现自己无法同步区块时可以做出以下操作： 

1. 将自己的p2p的连接上线提升，通过在启动参数中设置`in-peers`及`out-peers` 设置连接数上限，带宽允许的情况下设置为40以上为好

   ```bash
   {
   	"in-peers": 40,
   	"out-peers": 40,
   }
   ```

2. 若出现网络分区，可以停止节点，然后删除数据目录下的`networks` 目录，如启动时指定的数据路径是`<DB_ROOT>`，那么`networks`目录在`<DB_ROOT>/chains/chainx_mainnet` 路径下。（可选）

3. 若出现网络分区，除了可选删除`networks`，可以尝试在启动参数的`bootnodes`中配置如下种子节点：

   ```bash
   {
   	"bootnodes": [
   		"/ip4/47.96.134.203/tcp/20222/p2p/QmNzDfC4qTC9B5r8W7XxAR4n3qSF7SqgF6FsNGEWCL4foY",
   		"/ip4/47.110.252.22/tcp/20222/p2p/Qmcwx277eRTNmeK2iTLrGDHqRwYW8FxL8ppwRXM3SnbUvS",
   		"/ip4/47.96.97.52/tcp/20222/p2p/QmXABmmDgTT75o2UVCKS8iaYh75zPL5by3iSfS8msSWwEo",
   		"/ip4/47.110.14.60/tcp/20222/p2p/QmR2Jdd4V93yzTDyX5gKQ7PE4M2KsbK3vjC8RvXCnzqHU6",
   		"/ip4/47.110.14.60/tcp/30333/p2p/QmfDqzUo64DDDeinmWDif2umjgJpKyunYkKC81XNZ9Cwzt"，
   		"/ip4/47.96.146.181/tcp/20222/p2p/QmY8BWCfTVeutMi9i2Rn92kr31BWn8gEdYFwS8MSzYaAM6"
   	]
   }
   ```

   同时我们也欢迎节点给我们提供种子节点！

   方式：rpc接口`system_networkState`

   * 其中

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

   * `peerId` 代表节点的网络标示（存储于`networks`文件夹中），`externalAddresses`代表对外监听的ip和端口

   * 因此种子节点由 `externalAddresses`与`peerId`构成，为：

     ```bash
     seed = <externalAddresses(ipv4部分)>/p2p/<peerId>
     # e.g.
     seed = /ip4/172.16.176.18/tcp/20222/p2p/QmRaP225FNXoyB7WE8twinY8B6dVJXyGsmEYFA1Fc54rw1
     ```

### 4. 节点退选，连接，云服务商，vps选择等问题

* 关于防止退选问题，更详细请查阅[FAQ#漏块](FAQ#漏块)，[FAQ#检查漏块的可能方式](FAQ#检查漏块的可能方式)，[FAQ#节点防止退选的方法](FAQ#节点防止退选的方法)
* 关于阿里云香港
  * 目前ChainX有一些服务放于阿里云（国际）香港上，但是至今已出现了**多次网络访问出现短暂性异常**的情况，因此建议使用阿里云（国际）香港的节点引起警惕