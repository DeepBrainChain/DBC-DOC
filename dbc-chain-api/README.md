## DBC-Blockchain 第二阶段测试API

说明：第二阶段的测试中，一些API可能进行调整。本文档适用最新代码，当前分支`alpha-v2.1`，测试用`wss://innertest2.dbcwallet.io`

说明2：请求方式：

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



