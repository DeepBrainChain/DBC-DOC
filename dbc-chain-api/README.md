## DBC-Blockchain 第二阶段测试API

**说明1：** 第二阶段的测试中，一些API可能进行调整。本文档适用最新代码，当前分支[`alpha-v2.1`](https://github.com/DeepBrainChain/DeepBrainChain-MainChain/tree/alpha-v2.1)，测试用 `wss://innertest2.dbcwallet.io`

**说明2：** 当前测试网，使用 `wss://infotest.dbcwallet.io` 不适用本文档。文档链接：https://github.com/DeepBrainChain/DeepBrainChain-MainChain/blob/alpha-v2.0-fix/docs/Custom_RPC.md

**说明3：** 请求方式：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": method_name,
  "params": [params_1, params_2...]
}    
```

其中，method_name 为RPC方法名，params_1, params_2... 根据需要替换成需要的参数。

例如，利用 Postman 连接 websocket 查询`机器信息`：

[TODO]()

**说明4：** 本文档仅包含**`自定义RPC的说明`**，如果想查看 **`基于波卡链原生RPC的文档说明`** ，可以参考: https://polkadot.js.org/docs/substrate/rpc/；

**说明5:**  查看所有RPC的方法：可到dbc网页钱包：https://www.dbcwallet.io/ ，点击左侧切换网络，在自定义终端中，输入websocket接口。然后如下图导航到RPC-calls，查看区块链支持的所有RPC。如图，选择 `rpc`模块的`methods`方法，点击右边的`提交RPC调用`，将列出所有的RPC方法名。

另外，**`自定义的RPC` ** 暂时不支持通过网页调用。

![image-20210813113734192](README.assets/image-20210813113734192.png)

### 如何搭建自己的API节点

可以自己搭建同步节点，通过自己的节点API获取区块链数据。

方法：TODO

### DBC Custom RPC

关于`块高`，与`Era`的说明

#### 1. 查询机器某个Era获得收益



#### 2. 查询机器某个Era解锁收益



#### 3. 查询资金账户某个Era获得收益



#### 4. 查询资金账户某个Era获得奖励



#### 5. 查询机器详细信息

