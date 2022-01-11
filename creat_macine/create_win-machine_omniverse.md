### 创建windows虚拟机应用omniverse

____

## 步骤一: 确定要租用的机器

- 打开[主网钱包](https://www.dbcwallet.io/?rpc=wss://info.dbcwallet.io)

- 创建钱包：账户-->添加账户 （助记词一定要保存好，丢失了助记词，账户就无法找回，币就丢失掉了）

- 到 [银河竞赛机器列表](https://galaxyrace.deepbrainchain.org/table) 找到对应类型的空闲机器

  ![find_machine](./assets/rent_machine.assets/find_machine.png)

## 步骤二：租用链上机器

- 导航到 `开发者`---`交易`---`rentMachine` ----`rentMachine(machine_id, duration)`

- machine_id 输入要租用的机器 id，输入框里面的`0x`要先删除掉

- duration 输入需要租用的天数

- 输入完成后点击提交交易，并在三十分钟内确认机器是否可用。（如果 30 分钟内不确认租用，支付的`dbc`会退回，但是交易手续费 10 `dbc`是无法退回的）

- 创建windows虚拟机：

  >`请求方式`：POST
  >
  >`请求URL`：http://<**dbc_client_ip**>:<**dbc_client_port**>/api/v1/tasks/start
  >
  >`请求body`：
  >```json
  >{
  >   "peer_nodes_list": [
  >       // 请求机器的node_id
  >       "58fb618aa482c41114eb3cfdaefd3ba183172da9e25251449d045043fbd37f45"
  >   ],
  >   "additional": {
  >        "ssh_port": "",
  >       //远程登录时默认端口(每个虚拟机设置一个不同的值)
  >        "rdp_port":"3389",
  >       //虚拟机镜像名(确保虚拟机或镜像管理中心有所写的镜像名)
  >        "image_name": "windows_1909.qcow2",
  >       // 创建数据盘名称可不填默认创建
  >        "data_file_name": "",
  >        // gpu数量（大于等于 0）
  >        "gpu_count": "1",
  >       // cpu数量（大于0）
  >        "cpu_cores": "8",
  >       // 内存大小（大于0，单位：G）
  >        "mem_size": "32",
  >       // 磁盘大小（大于0，单位：G）
  >        "disk_size": "1",
  >       // 使用vnc连接该虚拟机时的端口号（每个虚拟机设置一个不同的值）
  >        "vnc_port": "5907",
  >       // windowns 系统(必填)
  >        "operation_system": "win10",
  >        "bios_mode": "uefi",
  >        "vm_xml": "",
  >        "vm_xml_url": ""
  >   },
  >
  >   "session_id": "租用者分发的session_id",
  >   "session_id_sign": "租用者分发的session_id_sign"
  >}
  >```
  >示例：
  >![create_win](create_win.png)

  * 创建过程的时间长短，会根据配置的不同而不同，大约在五分钟到十五分钟之间。
  * 可以通过请求`虚拟机详细信息`，查询到虚拟机`登录方式`以及虚拟机的`当前状态`（当状态值为"creating"，表示虚拟机正在创建过程中）

  相关操作请[参考](https://github.com/DeepBrainChain/DBC-DOC/blob/master/creat_macine/create_macine.md)

## 步骤三：确认可用并租赁

::: warning
确认之前必须要确认虚拟机能够正常启动。确认租用成功过后，表示机器租用成功，DBC 租金无法退回
:::

- 导航到`开发者`----`交易`----`rentMachine`----`confirmRent(machine_id)`

- 输入机器 id 并提交交易即可

## 步骤四：机器续租

::: warning
机器到期会自动停止虚拟机，确保在租用到期之前续租成功
:::

- 导航到`开发者`----`交易`----`rentMachine`----`reletMachine(machine_id, add_duration)`
- 输入机器 id 以及续租天数，提交交易

## 步骤五：远程连接虚拟机

* 查看到虚拟机登录方式后，在本地打开远程连接

  ![connect](connect.png)

## 步骤六：下载omniverse

* 打开`NVIDIA`官网下载`NVIDIA omniverse`：https://www.nvidia.cn/omniverse/#

* 根据官网的文档安装完成后：导航到`EXCHANGE(交易所)`在`Apps（应用）`部分中找到 `Audio2Face`，然后依次点击`“Install”（安装）`和`“Launch”（启动）`。

  ![install](install.png)

* 启动后，可以看到默认的头像及随附的语音及各模板参数（加载模板引擎时需等待几分钟）

  更多操作详情请[参考](https://docs.omniverse.nvidia.com/app_audio2face/app_audio2face/overview.html)

  ![face_info](face_info.png)