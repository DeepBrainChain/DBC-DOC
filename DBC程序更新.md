# DBC程序更新

## 说明，本文档会及时更新每一次更新内容，注意下载最新文件，并覆盖掉原有程序

+ 2021-7.15： DBC程序更新，适配postman接收请求
  + DBC程序下载地址：http://111.44.254.179:22244/dbc_update/dbc
+ 更新步骤：
  + 删除掉原有程序：`sudo rm -f /home/dbc/0.3.7.3/dbc_repo/dbc`
  + 下载最新程序：`sudo wget http://111.44.254.179:22244/dbc_update/dbc`
  + 重新启动dbc服务：`systemctl restart dbc`

#### 注意：此文档适用于小更新，7.14晚7点之前部署的DBC算力节点可以根据更新步骤操作，7.15最新安装无需更新

