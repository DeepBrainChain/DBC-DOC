## DBC-Blockchain 第二阶段测试API

**说明1：** 第二阶段的测试中，一些API可能进行调整。本文档适用最新代码，当前分支[`alpha-v2.1`](https://github.com/DeepBrainChain/DeepBrainChain-MainChain/tree/alpha-v2.1)，测试用 `wss://innertest2.dbcwallet.io`

**说明1.1** 除了`websocket接口`，可以自己搭建同步节点使用自己的节点提供的`HTTP接口`获取数据。同步节点搭建方式请查看[如何搭建自己的API节点]，使用HTTP接口获取数据的例子，可以查看本文件夹下的`dbc_chain_rpc.postman_collection.json`，导入Postman进行查看。

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

**说明4：** 本文档仅包含 **`自定义RPC的说明`** ，如果想查看 **`基于波卡链原生RPC的文档说明`** ，可以参考: https://polkadot.js.org/docs/substrate/rpc/；

**说明5:**  查看所有RPC的方法：可到dbc网页钱包：https://www.dbcwallet.io/ ，点击左侧切换网络，在自定义终端中，输入websocket接口。然后如下图导航到RPC-calls，查看区块链支持的所有RPC。如图，选择 `rpc`模块的`methods`方法，点击右边的`提交RPC调用`，将列出所有的RPC方法名。

另外， **`自定义的RPC`** 暂时不支持通过网页调用。

![image-20210813113734192](README.assets/image-20210813113734192.png)

### 如何搭建自己的API节点

可以自己搭建同步节点，通过自己的节点API获取区块链数据。

方法：

```bash
# 配置环境
curl https://getsubstrate.io -sSf | bash -s -- --fast
source ~/.cargo/env

# 编译dbc-chain
git clone https://github.com/DeepBrainChain/DeepBrainChain-MainChain.git
cd DeepBrainChain-MainChain && git checkout alpha-v2.1
cargo build --release

# 运行同步节点：
./target/release/dbc-chain --base-path ./db_data --chain ./dbcSpecRaw.json --pruning archive -rpc-cors all --bootnodes /ip4/111.44.254.180/tcp/20337/p2p/12D3KooWNtUXjdy8Q9hvdJ35a1jWpPnVBRgpT7nP8LmEgBWGqTpm


# 重要参数：
--rpc-port 9933 #  指定你的节点监听RPC的端口。 9933 是默认值，因此该参数也可忽略
--ws-port 9945 # 指定你的节点用于监听 WebSocket 的端口。 默认端口为 9944
--port 30333 # 指定你用于监听 p2p 流量的节点端口。 30333 是默认端口，若无需更改，可以忽略该 flag
```

如上方法运行了同步节点之后，可以通过 ws://127.0.0.1:9945 调用websocket接口，通过http://127.0.0.1:9933 调用http接口。

如果想远程访问，需要为websocket或http配置域名，以支持 wss 或者 https

### DBC Custom RPC

#### **关于`块高`，与`Era`的说明**：

1 Era 为2880个区块高度，在DBC区块链上，约是1天。每天发放奖励的时间，为区块链高度的2880的整数倍。

例如，当最新块高为3000时，调用RPC时，Era最多为0 (从0开始)，取的结果表示在区块链上第一天发放的奖励。

比如当区块高度为 2880 * 2 + 50，我们可以传的Era值，可以为0，1，分别表示区块链上第一天，第二天发放的奖励。


#### 0. 获取当前块高

+ 方法： `chain_getBlock`

+ 调用方法：

  ```
  {
       "jsonrpc":"2.0",
        "id":1,
        "method":"chain_getBlock",
        "params": []
  }
  ```

+ 返回结果：

  ```
  {
      "jsonrpc": "2.0",
      "result": {
          "block": {
  			...
              "header": {
  				...
                  "number": "0x2d8",
                  "parentHash": "0xc0e1b239fafc0edf3e08a908b5caecb27c2b351ed0daef3fc60c5600b28d6d7d",
                  "stateRoot": "0x55ce4794b2cdd21275656a80febf5133d882d909a2de6d40d7b8887bd65628bc"
              }
          },
          "justification": null
      },
      "id": 1
  }
  ```
  
  其中，"number": "0x2d8" 即为块高，转为十进制为：728



#### 1. 查询某个资金账户控制的所有机器



#### 2. 查询机器某个Era获得收益

+ 方法：`onlineProfile_getMachineEraReward`

+ 调用方法：

  ```
  {
       "jsonrpc":"2.0",
        "id":1,
        "method":"onlineProfile_getMachineEraReward",
        "params": ["ee0d003006f8ddbccb97dff96bcb4bee1b8c1aeaf7c64e0ca9d0f31752d17875", 1]
  }
  ```

  其中，参数分别为`MachineId`， `Era`

+ 结果：

  ```
  {
      "jsonrpc": "2.0",
      "result": "123456",
      "id": 1
  }
  ```

#### 3. 查询机器某个Era解锁收益

+ 方法：`onlineProfile_getMachineEraReleasedReward`

+ 调用方法

  ```
  {
       "jsonrpc":"2.0",
        "id":1,
        "method":"onlineProfile_getMachineEraReleasedReward",
        "params": ["ee0d003006f8ddbccb97dff96bcb4bee1b8c1aeaf7c64e0ca9d0f31752d17875", 1]
  }
  ```

  其中，参数分别为`MachineId`， `Era`

+ 结果：

  ```
  {
      "jsonrpc": "2.0",
      "result": "123456",
      "id": 1
  }
  ```

#### 4. 查询资金账户某个Era获得收益

+ `onlineProfile_getStashEraReward`

+ 调用方法：

  ```
  {
       "jsonrpc":"2.0",
        "id":1,
        "method":"onlineProfile_getStashEraReward",
        "params": ["5DhR2dxiPZquPhFjfPzFg5jZENdr375hbX643kr9FBXMVa2z", 1]
  }
  ```

  其中，参数分别为 `资金账户`，`Era`

+ 结果：

  ```
  {
      "jsonrpc": "2.0",
      "result": "123456",
      "id": 1
  }
  ```

#### 5. 查询资金账户某个Era解锁奖励

+ `onlineProfile_getStashEraReleasedReward`

+ 调用方法：

  ```
  {
       "jsonrpc":"2.0",
        "id":1,
        "method":"onlineProfile_getStashEraReward",
        "params": ["5DhR2dxiPZquPhFjfPzFg5jZENdr375hbX643kr9FBXMVa2z", 1]
  }
  ```

  其中，参数分别为 `资金账户`，`Era`

+ 结果：

  ```
  {
      "jsonrpc": "2.0",
      "result": "123456",
      "id": 1
  }
  ```

#### 6. 查询机器详细信息

