+ 项目地址：https://github.com/DeepBrainChain/DeepBrainChain-MainChain
+ 当前开发分支：`alpha-v2.1`
+ 最新release: [pre-main-v1.0](https://github.com/DeepBrainChain/DeepBrainChain-MainChain/releases/tag/pre-main-1.0)
+ 项目基于substrate [v3.0.0](https://github.com/paritytech/substrate/releases/tag/v3.0.0)



+ 项目结构说明

  ```
  # 项目主要逻辑位于pallets文件夹, 模块的层次关系如下：
  # dbc-staking, generic-func, dbc-price-ocw独立于其他模块
  # committee, online-profile 依赖 generic-func, dbc-price-ocw
  # maintain-committee, online-committee, rent-machine 依赖上一层模块
  
  pallets
  ├── dbc-staking 	# 用于节点POA出块验证奖励，修改自substrate staking模块
  ├── generic-func 	# 一些帮助函数，如序列化/一些常用计算
  ├── dbc-price-ocw 	# offchain-worker，用于获取dbc实时价格
  |
  ├── committee 		# 委员会模块，用于后续链上机器的管理
  ├── online-profile 	# 机器管理模块，主要用于机器上线/下线/在线奖励发放
  |
  ├── maintain-committee 	# 机器上线后出现问题的举报模块，租用者可以通过该模块进行举报，委员会进行审核
  ├── online-committee 	# 机器上线审核模块，主要用于机器上线时分派给委员会进行审核
  ├── rent-machine 		# 租用机器/续租
  |
  └── simple-rpc # 仅用于提供RPC，为了testcase更简单，将一部分RPC单独放在该palelt下
  ```

  

+ 测试：根据pallets模块之间的依赖关系，testcase位于`maintain-committee`, `online-committee`, `rent-machine`

  ```bash
  cargo test -p online-committee
  cargo test -p rent-machine
  cargo test -p maintain-committee
  ```



### dbc-chain主要模块功能说明

1. 委员会模块(`committee` pallet)：通过公投（Root权限）添加委员会，委员会成员(committee)质押一定DBC，参与机器的管理并获取DBC奖励

2. 机器上线主要逻辑 (`online-profile` pallet 和 `online-committee` pallet)
   1. 机器资金帐号(stash) 绑定 机器控制帐号(controller) 
   2. 控制帐号提交机器上线请求（提交机器私钥签名，用于机器认证stash账户），并质押stash账户一定DBC
   3. 机器被派单给状态正常的委员会，允许在特定时间登录机器进行验证。被派单的委员会被记录“使用了一部分质押”
   4. committee **依次**验证机器信息（链下验证），并提交确认的结果（认可上线/拒绝上线）的Hash
   5. 到特定时间/所有委员会提交了Hash之后，允许委员会提交原始确认结果（同意/拒绝机器上线）
   6.1 机器成功上线（同意上线的委员会>拒绝的委员会）：从下一天开始，机器和委员会获得奖励。同时，拒绝上线/提交了与其他委员会不同信息的委员会，会被系统记录，并添加一个待执行的惩罚
   6.2 机器被拒绝上线（同意上线的委员会<=拒绝的委员会） ：支持上线的委员会和机器stash会被记录，并添加一个待执行的惩罚
   6. 执行惩罚：在系统中存在惩罚，惩罚执行前（两天内）允许被惩罚人进行申述（额外质押DBC），技术委员会(technical committee)在惩罚执行前可以取消惩罚，如果申述通过，则惩罚被删除。否则，惩罚在两天后(惩罚发生算起)执行。
   7. 机器成功上线后，控制账户的操作
     1. 领取资金账户获取的奖励
     2. 使机器下线（租用/在线时），以处理故障（避免被举报）。在这种情况下，机器上线时（需要质押待惩罚金额），统计机器离线时长，根据离线时的状态进行惩罚。
     3. 下线以用于修改机器配置。不同于直接下线，这种情况下，需要委员会重新审核才能上线。
   
3. 机器维护模块(`maintain-committee`)主要逻辑

   1. 成功上线的机器，可以被普通用户租用(renter)。租用之前/租用过程中发现问题，可以通过`maintain-committee`进行举报。

   2. 可以举报的情况：

      1). 当机器处于租用状态，机器无法访问

      2). 当机器处于租用状态，机器硬件出现故障：

      3). 当机器处于租用状态，发现硬件参数配置造假

      4). 当机器处于闲置状态，机器无法租用

#### 奖励与惩罚说明

##### 奖励

stash账户的奖励：

1. 机器在线/被租用，将会按比例分得每天发放的奖励。
2. ***银河竞赛*** 开启前(系统中在线的GPU达到一定数量开启)，租金将发放给stash账户。

committee的奖励

1. 机器初次成功上线时，有效审核的委员会将被记录，并在随后的两年内，每天分得机器获得奖励的1%
2. 机器上线失败时，拒绝上线的委员会将获得机器上线时stash账户质押的一定比例的奖励
3. 机器因修改配置下线时，需要stash质押一定金额，并进行委员会审核。随后机器成功上线时，有效的审核的委员会，将分得质押的金额。
4. 当机器被举报时，并且举报被通过时，委员会将获得一定比例的对stash的惩罚

##### 惩罚

stash 账户的惩罚

1. 机器上线被拒绝
2. 机器上线之后下线
3. 被举报而下线
4. 下线修改配置信息，审核不通过

committee的惩罚

1. 没有做完分派的任务：机器上线时，会随机挑选委员会进行验证。没有完成验证任务的将会被惩罚
2. 与其他委员会意见不一致：当需要审核（包括机器上线时的系统自动派单，以及机器被举报时委员会主动抢单）时，通常需要委员会先提交结果的Hash，等到所有委员会提交完Hash后，再提交原始信息。如果提交的原始信息与大多数委员会不一致，将被惩罚



