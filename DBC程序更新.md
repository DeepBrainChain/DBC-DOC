# DBC程序更新

## 说明，本文档会及时更新每一次更新内容，注意下载最新文件，并覆盖掉原有程序

+ 2021-7.15： DBC程序更新，适配postman接收请求
  + DBC程序下载地址：http://111.44.254.179:22244/dbc_update/dbc
+ 更新步骤：
  + 删除掉原有程序：`sudo rm -f /home/dbc/0.3.7.3/dbc_repo/dbc`
  + 下载最新程序：`sudo wget http://111.44.254.179:22244/dbc_update/dbc`
  + 重新启动dbc服务：`systemctl restart dbc`

#### 注意：此文档适用于小更新，7.14晚7点之前部署的DBC算力节点可以根据更新步骤操作，7.15最新安装无需更新

+ 2021-7.23: DBC程序更新，解决种子节点向算力节点发送请求却返回内网ip或空ip问题。
+ 早于7.14安装的DBC程序请重新部署DBC程序（执行以下操作）
```shell
sudo cp /home/dbc/0.3.7.3/dbc_repo/dat/node.dat /data
sudo rm -rf /home/dbc/0.3.7.3/
sudo wget http://111.44.254.179:22244/install_dbc_ry_machine.sh
sudo bash install_dbc_ry_machine.sh -d
sudo bash install_dbc_ry_machine.sh -i /home/dbc           #(钱包地址还请输入原钱包地址）
sudo cp /data/node.dat /home/dbc/0.3.7.3/dbc_repo/data/
sudo bash /home/dbc/0.3.7.3/dbc_repo/tool/node_info/node_info.sh
systemctl restart dbc
```
+ 7.15日之后有过更新的请下载更新脚本并执行即可
+ http://111.44.254.179:22244/update_dbc.sh

