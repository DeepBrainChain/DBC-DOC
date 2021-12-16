## 链上机器租用 

### 租用流程 

***

> note: 图片中部分配置仅作参考

#### 步骤一

* 打开浏览器 https://www.dbcwallet.io/?rpc=wss%3A%2F%2Finfo.dbcwallet.io

* 创建钱包：账户-->添加账户 （助记词一定要保存好，丢失了助记词，账户就无法找回，币就丢失掉了）

* 到https://galaxyrace.deepbrainchain.org/table 找到对应类型的空闲机器

  ![find_machine](find_machine.png)

#### 步骤二 

* 租用链上机器 
     - 导航到 `开发者`---`交易`---`rentMachine` ----`rentMachine(machine_id, duration)`
     - machine_id输入要租用的机器id，输入框里面的`0x`要先删除掉
     - duration输入需要租用的天数
     - 输入完成后点击提交交易，并在三十分钟内确认机器是否可用。（如果30分钟内不确认租用，支付的`dbc`会退回，但是交易手续费10 `dbc`是无法退回的）
* 创建虚拟机等相关操作请参考： https://github.com/DeepBrainChain/DBC-DOC/blob/master/creat_macine/creat_macine.md

#### 步骤三：确认可用并租赁 （确认之前必须要确认虚拟机能够正常启动，否则这一步确认过后，表示机器租用成功，DBC是无法退回的）

* 导航到`开发者`----`交易`----`rentMachine`----`confirmRent(machine_id)`
* 输入机器id并提交交易即可

#### 机器续租 （机器到期会自动停止虚拟机，确保在到期之前续租成功）

* 导航到`开发者`----`交易`----`rentMachine`----`reletMachine(machine_id, add_duration)`
* 输入机器id以及续租天数，提交交易





