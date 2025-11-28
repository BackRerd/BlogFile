---
title: fisco之多群组多节点
tags:
  - fisco
abbrlink: 1b6eafa3
date: 2025-01-05 09:57:07
categories: ["区块链"]
---

# 搭建一个4机构3群组8节点

此篇章是以官方为基础搭建的

前置需求:

| 机构                 | 群组      | IP        |
| -------------------- | --------- | --------- |
| 机构A(2节点) node0,1 | 群组1,2,3 | 127.0.0.1 |
| 机构B(2节点) node2,3 | 群组1     |           |
| 机构C(2节点) node4,5 | 群组2     |           |
| 机构D(2节点) node6,7 | 群组3     |           |

先将构建fisco的脚本下载下来

```sh
curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v2.11.0/build_chain.sh && chmod u+x build_chain.sh
```

创建一个配置文件

```sh
vim ipconf
```

配置文件内容

```
127.0.0.1:2 A 1,2,3
127.0.0.1:2 B 1
127.0.0.1:2 C 2
127.0.0.1:2 D 3
```

文件结构详解

```
IP:节点数量 机构名称 所在群组
.... .... ....
```

使用命令一键创建

```sh
bash build_chain.sh -f ipconf -p 30300,20200,8545
```

`-f`指定一个文件，这个文件编写的内容是创建信息

![image.png](https://s2.loli.net/2025/01/05/YEODv4SphVk2JAc.png)

启动所有节点

```sh
bash nodes/127.0.0.1/start_all.sh 
```

查看节点进程

```sh
ps aux | grep fisco-bcos
```

![image.png](https://s2.loli.net/2025/01/05/lS6kxMGV8BcyIL1.png)

查看所有群组的共识

```sh
tail -f nodes/127.0.0.1/node0/log/* | grep "g.*++"
```

查看单个群组共识

```sh
tail -f nodes/127.0.0.1/node0/log/* | grep "g:1.*++"
```

`g:int`将int改成所要查询的群组

`node0`将node0也可以改成其他节点，查看其他节点的群组是否正常或者存在

![image.png](https://s2.loli.net/2025/01/05/4Df8x7nU3VWNvEs.png)

# 扩容节点

## 前置要求:

| 节点         | 群组  | IP       |
| ------------ | ----- | -------- |
| 机构D(1节点) | 3群组 | 127.0.01 |

获取证书的生成脚本

```sh
curl -#LO https://raw.githubusercontent.com/FISCO-BCOS/FISCO-BCOS/master-2.0/tools/gen_node_cert.sh
```

下载失败使用这个:`curl -#LO https://gitee.com/FISCO-BCOS/FISCO-BCOS/raw/master-2.0/tools/gen_node_cert.sh`

为新节点生成密钥

```sh
bash gen_node_cert.sh -c nodes/cert/D/ -o node8
```

`-c`指定要在哪个机构下新增节点

`-o`新的节点名称

![image.png](https://s2.loli.net/2025/01/05/XlxrMCcSdJGh2jN.png)

拷贝所需要的配置文件(由于这里是在机构D新增的节点所以拷贝节点6和节点7都可以)

```sh
cp nodes/127.0.0.1/node7/config.ini nodes/127.0.0.1/node7/start.sh nodes/127.0.0.1/node7/stop.sh node8/
```

这里是将配置文件启动文件都拷贝了过去

编辑node8的配置文件

```sh
vim node8/config.ini
```

```sh
[rpc]
    channel_listen_ip=0.0.0.0
    channel_listen_port=20208
    jsonrpc_listen_ip=127.0.0.1
    jsonrpc_listen_port=8553
    disable_dynamic_group=false
[p2p]
    listen_ip=0.0.0.0
    listen_port=30308
    ; nodes to connect
    node.0=127.0.0.1:30300
    node.1=127.0.0.1:30301
    node.2=127.0.0.1:30302
    node.3=127.0.0.1:30303
    node.4=127.0.0.1:30304
    node.5=127.0.0.1:30305
    node.6=127.0.0.1:30306
    node.7=127.0.0.1:30307
    node.8=127.0.0.1:30308
```

`p2p`加入自身的信息

`listen_port`端口自增一下，我这里拷贝的是node7所有自增一下就可以，如果拷贝的node6那就需要自增两下

`channel_listen_port`同上

`jsonrpc_listen_port`同上



将群组的连接信息和配置文件拷贝进去

```sh
cp nodes/127.0.0.1/node7/conf/group.3.genesis nodes/127.0.0.1/node7/conf/group.3.ini node8/
```

先将新节点复制到当前的nodes文件下,然后启动节点

```sh
cp -r node8/ nodes/127.0.0.1/
bash node8/start.sh
```

查看节点连接

```sh
tail -f nodes/127.0.0.1/node8/log/log*  | grep "connected count"
```

![image.png](https://s2.loli.net/2025/01/05/RQMWcnpedYHg8K4.png)

## 将节点添加到群组中

查看节点nodeid

```sh
cat nodes/127.0.0.1/node8/conf/node.nodeid
```

![image.png](https://s2.loli.net/2025/01/05/oLuTIpXvUgq27Mj.png)

进入控制台3群组

```sh
bash console/start.sh 3
```

![image.png](https://s2.loli.net/2025/01/05/7GorkeIRwU6EBFz.png)

`addObserver`观察节,不参与共识，但能实时同步链上数据的节点。

`addSealer`共识节,参与共识的节点，拥有群组的所有数据（搭链时默认都生成共识节点）。

这里需要共识所有添加到共识节点内

```sh
addSealer 替换成自己节点的nodeid
```

![image.png](https://s2.loli.net/2025/01/05/kZaorvQ8DXe3WgI.png)

# 新增群组

## 前置需求

将node8节点添加至新增的群组中

------

获取时间戳

```sh
echo $(($(date '+%s')*1000))
```

将时间戳记下来后开启控制台

```sh
bash console/start.sh
```

查看节点IP与Post端口号

```
getAvailableConnections
```

获取节点nodeid

```sh
getSealerList
```

选择第一个nodeid

为节点动态创建一个新群组

```
generateGroup 127.0.0.1:20200 4 1736048077000 8b77bf44e79b207fcefe2d350ee4d9af0f45ef9425a7ce12cc61f6c87effc00c24dbbe5aa874179b9800584906b54b79a0f88ccda9f76c2b91058807a23985cf
```

命令详解: generateGroup 获取的IP和Post端口 新群组id 时间戳 节点nodei

![image.png](https://s2.loli.net/2025/01/05/Xxi4rWE936Rhngl.png)

启动新群组,切换新群组

```
startGroup 127.0.0.1:20200 4
switch 4
```

将node8节点添加到新群组中

```sh
cat nodes/127.0.0.1/node8/conf/node.nodeid #获取节点id
addSealer 8b77bf44e79b207fcefe2d350ee4d9af0f45ef9425a7ce12cc61f6c87effc00c24dbbe5aa874179b9800584906b54b79a0f88ccda9f76c2b91058807a23985cf
```

![image.png](https://s2.loli.net/2025/01/05/NEBevq6bY8lXGtK.png)

# 结尾

小作业:

1.创建四机构三群组10节点的区块链，详细要求如下:

| 机构         | 群组  | 节点        |
| ------------ | ----- | ----------- |
| 机构A(3节点) | 1,2,3 | node0,1,2   |
| 机构B(1节点) | 1     | node3       |
| 机构C(4节点) | 2     | node4,5,6,7 |
| 机构D(2节点) | 1,3   | node8,9     |

2.新增节点node10,node11,node12详细要求如下:

| 机构  | 群组 | 节点      |
| ----- | ---- | --------- |
| 机构A | 3    | node11    |
| 机构B | 1    | node10,12 |

3.新增群组4,并将node10,node11,node12添加至其中
