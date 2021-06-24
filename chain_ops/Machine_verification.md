# 委员会如何验证机器

0. 成为委员会

   可以通过社区投票参加委员会。

   委员会需要提交用于信息加密的公钥，才能正常的派单与抢单。

   ```bash
   # 生成公钥，需要利用脚本，指定自己的私钥
   
   node gen_boxpubkey.js --key "0x868020ae0687dda7d57565093a69090211449845a7e11453612800b663307246"
   ```

   生成了公钥之后，到`committee` -- `committeeSetBoxPubkey` 提交交易进行设定。

   ![image-20210623145108399](bonding_machine.assets/image-20210623145108399.png)

1. 查看系统分配给自己的订单。导航到 `开发者`--`链状态`--`存储`，在其中选择`leaseCommittee`模块的`committeeMachine`存储，点击右侧的`+`号，可以看到委员会的订单情况。如果所示，该委员会有一个系统分配的订单

   ![image-20210601164137286](bonding_machine.assets/image-20210601164137286.png)

2. 查看系统分配给该委员会进行验证的时间区间：导航到 `开发者`--`链存储`--`存储`，选择`leaseCommittee`的`committeeOps`方法，并输入自己的委员会帐号，与上一步委派的机器 ID，可以查询到类似下面的信息：

   ![image-20210601164631426](bonding_machine.assets/image-20210601164631426.png)

   其中，booked_time 表示派单时间，注意，派单之后的 36~48 小时(也就是区块高度 booked_time + 4320 ~ booked_time + 5760)之间，委员会提交原始信息。

   `verify_time` 表示系统分派的，委员会验证机器信息的开始时间。如图，该委员会被分派了 9 次机会来验证机器，每次持续时间为 4 个小时，也就是 480 个块高。此时，委员会可以挑选自己方便的时间，通过前端查询该机器的登录信息，登录到系统中验证机器。

3. 委员会计算获得机器信息的 hash

   我们已经提供了脚本来计算需要填写的信息的 Hash：

   `https://github.com/DeepBrainChain/DeepBrainChain-MainChain/blob/feature/staking_v3.0.0_online_profile/scripts/hash_str.py`

   当获取到要求的信息后，修改该脚本，并执行，得到 hash 值。**请保存好所填写的信息，直到该机器上线成功，或者上线失败**

   ```bash
   python3 hash_str.py
   ```

4. 委员会提交机器信息的 Hash。如图，在 36 小时之前提交机器信息的 Hash

   ![image-20210601165736511](bonding_machine.assets/image-20210601165736511.png)
   
5. 委员会提交机器的原始信息。**请确保提交机器原始信息时，在派单之后的 36~48 小时之间！**

   ![image-20210601165851303](bonding_machine.assets/image-20210601165851303.png)