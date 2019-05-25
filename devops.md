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
