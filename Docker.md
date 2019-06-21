## 使用 Docker 运行 ChainX 节点

ChainX 主网使用镜像: [chainxorg/chainx:v1.0.2](https://hub.docker.com/r/chainxorg/chainx/tags).

### 1. 生成节点 keystore

该步骤目的是生成节点出块地址。

```bash
$ docker run -it --rm -v $(pwd)/data:/data -v $(pwd)/log:/log -v $(pwd)/keystore:/keystore chainxorg/chainx:v1.0.2 chainx -i --keystore-path=/keystore --base-path=/data --log-dir=/log
Password:
Repeat again:
...
##### 输入密码完成可使用 <Ctrl-C> 退出
```

- 上述命令会提示输入一个密码，该密码即为节点 keystore 的密码。

- 输入密码完成后，会当前目录下生成了一个名为 `keystore` 的文件夹，文件夹下有一个文件名为节点公钥的文件，该公钥即为我们要使用的节点出块地址对应的公钥。

```bash
$ ls keystore
f1a10ac84641d72074f89c8b4dcaa10ab2a8e982921a81c292f2839f9bf6080f
```

将该公钥粘贴到浏览器，可以看到节点公钥所对应的地址。

### 2. 启动同步节点

首先准备一个节点启动的配置文件 `config.json`:

```json
{
    "validator": false,
    "rpc-external": true,
    "ws-external": true,
    "log": "info,runtime=info",
    "name": "<Your-Name>",
    "port": 20222,
    "ws-port": 8087,
    "rpc-port": 8086,
    "base-path": "/data",
    "log-dir": "/log",
    "keystore-path": "/keystore",
    "keystore-password": "<Your-Keystore-Password>",
    "other-execution": "NativeElseWasm",
    "syncing-execution": "NativeElseWasm",
    "block-construction-execution": "NativeElseWasm",
    "bootnodes": []
}
```

- `keystore-password`: 这里填写上一步输入的 Password.

启动同步节点 Docker：

```bash
$ docker run -d --restart always --name chainx -p 8087:8087 -p 8086:8086 -p 20222:20222 -v $(pwd)/data:/data -v $(pwd)/log:/log -v $(pwd)/keystore:/keystore -v $(pwd)/config.json:/config.json chainxorg/chainx:v1.0.2 chainx --config=/config.json
```

查看 Docker 日志：

```bash
# 部分Error信息等
$ docker logs -f chainx

# 运行时日志
$ tail -f log/chainx.log
```

### 3. 注册并更新节点进入参选状态

- 首先注册节点，所填写的节点名称即为启动验证节点时使用的 `validator-name`。

- 注册节点完成后，默认处于退选状态。选择更新节点，使用节点进入参选状态，只有节点处于参选状态且总得票数不为0的节点才有机会参与验证节点选举。

- 注意更新节点时所填写的出块地址，为步骤 1 中生成的节点出块地址。

### 4. 启动验证人节点

在同步节点完全同步后，修改 `config.json` 加入启动验证人节点的必要信息，然后重启 Docker 进入验证人模式。

修改 `config.json`:

- 将 `validator` 设置为 true,
- 添加 `validator-name` 字段，`validator-name` 为注册节点时填写的 name.

```json
{
    "validator": true,
    "validator-name": "<Your-Valitor-Name>",
    "rpc-external": true,
    "ws-external": true,
    "log": "info,runtime=info",
    "name": "<Your-Name>",
    "port": 20222,
    "ws-port": 8087,
    "rpc-port": 8086,
    "base-path": "/data",
    "log-dir": "/log",
    "keystore-path": "/keystore",
    "keystore-password": "<Your-Keystore-Password>",
    "other-execution": "NativeElseWasm",
    "syncing-execution": "NativeElseWasm",
    "block-construction-execution": "NativeElseWasm",
    "bootnodes": []
}
```

然后重启 chainx container 即可:

```bash
$ docker restart chainx
```
