# 搭建DBC客户端节点

## 1. 客户端节点的作用

+ DBC客户端节点在整个网络当中作为信任人的身份，可以查询到当前网络中的正确部署功能节点的机器。当功能节点部署完成后，可以通过http请求客户端节点来查看您的功能节点是否成功连接到客户端节点。当通过客户端请求能成功看到您的功能节点机器ID时，这代表您已经成功部署完成功能节点。
+ 建议：由于官方提供的客户端节点也并不能永久保持稳定在线，所以，建议每家矿池搭建两个客户端节点以作备用，同时也可以壮大DBC网络。
+ 提示：客户端节点对硬件设备要求很低，可以正常运行的公网服务器都可以搭建，占用内存非常小，只要可以通过公网访问即可。

## 2.安装客户端节点

```shell
sudo wget http://111.44.254.179:22244/dbc/dbc-linux-client-0.3.7.3.tar.gz
sudo tar -zxvf dbc-linux-client-0.3.7.3.tar.gz
cd 0.3.7.3/dbc_repo/


#修改conf目录下的core.conf配置文件：
rest_ip 改为0.0.0.0
rest_port=4545     #此处建议使用5000以下端口
#启动dbc：
./dbc -d
```

## 3.查看客户端节点是否正确部署

+ 查看端口是否正确开放 `ss -unltp | grep 41107` 被dbc占用即为成功。
+ 执行`ps -ef | grep dbc`,有显示`./dbc -d`即可。

## 4.通过HTTP请求查看功能节点是否正确加入到网络（通过浏览器访问）

+ 下载安装postman，具体下载请去官网根据操作系统安装

+ 下载json文件：http://114.116.21.175:22244/dbc-develop-0.3.7.5.postman_collection.json
   
+ 导入json文件：fiel----import----选择json文件导入 import
    ![image](https://user-images.githubusercontent.com/32829693/133870420-b790637c-cab6-44f9-ba00-493eadc951cd.png)
    
+ 查看宿主机详细信息：
   
   签名工具下载地址：https://github.com/DeepBrainChain/DBC-AIComputingNet/releases/download/0.3.7.3/sign_tool
   
   安装包：sudo apt-get install libvirt
   
   添加执行权限：chmod 777 sign_tool ,  然后签名执行：./sign_tool   钱包地址   钱包私钥种子
   
   怎么获得钱包私钥种子？https://github.com/DeepBrainChain/DBC-DOC/blob/master/chain_ops/bonding_machine.md#%E6%9C%BA%E5%99%A8%E4%B8%8A%E7%BA%BF%E6%AD%A5%E9%AA%A4
   
   ![image](https://user-images.githubusercontent.com/32829693/133870889-61976abb-ae6b-4cd6-97e3-9e9205745346.png)

   在下图中填入：sign、nonce、wallet （注意：同一个机器sign、nonce只能使用一次），可以查询到机器信息
   <img width="751" alt="1631934612(1)" src="https://user-images.githubusercontent.com/32829693/133870573-04dbcb84-9112-4837-b8e4-20db8538c079.png">

