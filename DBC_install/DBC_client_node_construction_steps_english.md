# DBC client node construction steps

## 1. The role of the client node

+ The DBC client node acts as the identity of the trustee in the entire network, and can query the machine where the function node is correctly deployed in the current network. When the function node is deployed, you can request the client node through http to check whether your function node is successfully connected to the client node. When the machine ID of your function node can be successfully seen through the client request, it means that you have successfully deployed the function node.
+ Suggestion: Since the official client nodes cannot remain stable online forever, it is recommended that each mining pool set up two client nodes as a backup, and at the same time, it can also strengthen the DBC network.
+ Tip: The client node has very low requirements for hardware equipment. Any public network server that can run normally can be built(container mode can also be used), and the memory is very small, as long as it can be accessed through the public network.

## 2.Install the client node

```shell
sudo wget http://116.85.24.172:20444/static/dbc/dbc-linux-client-0.3.7.3.tar.gz
sudo tar -zxvf dbc-linux-client-0.3.7.3.tar.gz
cd 0.3.7.3/dbc_repo/


#Modify the core.conf configuration file in the conf directory：
rest_ip=0.0.0.0
rest_port=4545     #It is recommended to use ports below 5000
#start dbc：
./dbc -d
```

## 3.Check whether the client node is deployed correctly

+ Check if the port is open correctly. `ss -unltp | grep 41107` is occupied by dbc, which means success.
+ Execute `ps -ef | grep dbc`, and display `./dbc -d`.

## 4.Check whether the function node is correctly added to the network through HTTP request (accessed through a browser)

+ Officially provided node 1: http://111.44.254.162:41107/api/v1/mining_nodes/<machine ID> (for now, it will be updated during the public beta)
+ After the mining pool is successfully set up: http://<client server ip>:41107/api/v1/mining_nodes/<machine ID>

## 5.Description

+ Client nodes can be built on one public network server with docker, only need to modify the listening address of the `core.conf `file.

