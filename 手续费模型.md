## ChainX的手续费模型

ChainX参考了以太坊，波卡的手续费系统，设计了一套特有的手续费模型。

### ChainX手续费的基础概念：

1. 交易权重 `weight`
2. 交易基础费`transaction_base_fee`
3. 交易字节费`transaction_byte_fee`
4. 交易加速`accelerate`

在ChainX中

* `weight`：交易可调用的方法赋予了名为`weight`的权重，例如转账`transfer`的权重是1，`withdraw`的权重是3。`weight`代表了这个交易的复杂度，或者耗费的资源，其权重的基础值是`transaction_base_fee`。换句话说，几个交易赋予了多少了`weight`代表着这个交易的复杂度是基本交易的几倍。`weight`可由议会进行调整
* `transaction_base_fee`：交易权重的基本值，即其代表一倍权重对应的手续费大小
* `transaction_byte_fee`：交易每字节的手续费大小，由于交易需占用一定的空间，因此交易越大，对应的手续费就越多
* `accelerate`：交易的加速倍数，当2个交易相同时，付出的`accelerate`越高在交易池中的排序也越高，越容易被打包，对应的手续费也会耗费对应的倍数。该概念类似于以太坊的`gas price`，应由用户选择提供的数值。

### ChainX的手续费计算方式：

```bash
fee = (transaction_base_fee * weight + transaction_byte_fee * len(交易长度)) * accelerate
```

由该计算方式可以看到，其中：

* `weight`，`transaction_base_fee`和`transaction_byte_fee`由ChainX链上提供
* 交易长度`len`及`accelerate`由交易发送方决定

###  ChainX提供的手续费相关接口

当前ChainX提供了2个于手续费计算相关的rpc

1. [chainx_getFeeByCallAndLength](RPC#chainx_getFeeByCallAndLength)

   该接口传递交易编码过的function原文（注意不是交易体，是交易体中的function部分），交易长度，可以在ChainX节点中计算获取 **未加速前**的手续费值。

2. [chainx_getFeeWeightMap](RPC#chainx_getFeeWeightMap)

   该接口可获取当前ChainX链上的`weight`列表，`transaction_base_fee`及`transaction_byte_fee`。开发者可根据这三个参数在本地根据上述公式计算得到应付的手续费。