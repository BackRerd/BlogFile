---
title: webase前置安装
tags: ubuntu
abbrlink: '65771201'
date: 2024-09-26 14:12:28
categories: ["区块链"]
---
# 安装mysql
更新安装程序:

```sh
apt-get upgrade
```

安装Mysql

```sh
apt-get install mysql-server
```

开启mysql服务

```sh
service mysql start
service mysql status
netstat -tap | grep mysql
```

进入mysql界面

```sh
mysql -u root -p
```

解决中文乱码:

```sh
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```sql
character_set_server=utf-8
#bind-address = 0.0.0.0  #设置远程访问
```

设置mysql密码:

```sql
sudo mysql
USE mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
FLUSH PRIVILEGES;
EXIT;
```

创建新用户并授权本地访问,以及数据库

```sql
create user 'root'@'%' identified by '123456';
grant all privileges on *.* to 'root'@'%' with grant option;
create user 'test'@'localhost' identified by '123456';
grant all privileges on *.* to 'test'@'localhost' with grant option;
flush privileges;
create database webasenodemanager;
exit
```



# 安装Java

安装JDK8

```sh
sudo apt install openjdk-8-jdk
```

验证安装

```sh
java -version
```

配置环境变量-编辑配置文件

```
which java
```

```sh
vim /etc/profile
```

```sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

加载配置

```sh
source /etc/profile
```

验证输出

```sh
echo $JAVA_HOME
```

# 安装Python

```sh
sudo apt install python-is-python3
```

验证安装

```sh
python3 --version
```

安装pip

```sh
sudo apt install python3-pip
pip3 --version
```

设置环境变量

```sh
vim ~/.bashrc
```

编辑配置文件

```sh
export PYTHON_HOME=/usr/bin/python3.8
export PATH=$PATH:$PYTHON_HOME
```

加载配置文件

```sh
source ~/.bashrc
```

安装PyMySQL

```sh
pip3 install pymysql
```

验证安装

```sh
pip3 show pymysql
```

如若需要配置环境可以按照以下步骤，不用则跳过

```sh
vim ~/.bashrc
```

编辑配置文件

```sh
export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.8/dist-packages
```

加载配置

```sh
source ~/.bashrc
```



# webase的一些配置文件说明
```
vi common.properties
```
```
# WeBASE子系统的最新版本(v1.1.0或以上版本)
webase.web.version=v1.5.2
webase.mgr.version=v1.5.2
webase.sign.version=v1.5.0
webase.front.version=v1.5.2
# 节点管理子系统mysql数据库配置
mysql.ip=127.0.0.1
mysql.port=3306
mysql.user=dbUserame
mysql.password=dbPassword
mysql.database=webasenodemanager
# 签名服务子系统mysql数据库配置
sign.mysql.ip=localhost
sign.mysq!.port=3306
sign.mysq!.user=dbUsername
sign.mysgl.password=dbPassword
sign.mysql.database=webasesian
#节点前置子系统h2数据库名和所属机构
front.h2.name=webasefront
front.org=fisco
# WeBASE管理平台服务端口
web.port=5000
#启用移动端管理平台(0: disable,1: enable)
web.h5.enable=1
# 节点管理子系统服务端口
mgr:port=5001
#节点前置子系统端口
front.port=5002
#签名服务子系统端口
sign.port=5004
# 节点监听lp
node.listenlp=127.0.0.1
# 节点p2p端口
node.p2pPort=30300
#节点链上链下端口
node.channelPort=20200
# 节点rpc端口
node.rpcPort=8545
# 加密类型(0: ECDSA算法,1: 国密算法)
encrypt.type=0
# SSL连接加密类型(0: ECDSA SSL, 1:国密SSL)
# 只有国密链才能使用国密
SSLencrypt.ssl7ype=0
# 是否使用已有的链(yes/no)
if exist.fisco=no
# 使用己有链时需配置
# 已有链的路径，start al.sh脚本所在路径
#路径下要存在sdk目录(sdk目录中包含了SSL所需的证书，即ca.crt、sdk.crt、sdk.key和gm目录(包含国密SSL证书gmca.crt、gmsdk.crt、gmsdk.key、gmensdk.crt和gmensdk.key)
fisco.dir=/datalapp/nodes/127.0.0.1
# 前置所连接节点的绝对路径#节点路径下要存在conf文件来，conf里存放节点证书(ca.crt、node.crt和node.key)
node.dir=/datalapp/nodes/127.0.0.1/node0
# 搭建新链时需配置
# FISCO-BCOS版本fisco.version=2.7.2
# 搭建节点个数(默认两个)
node.counts=nodeCounts
```

运行

```
#部署并启动所有服务：
python3 deploy.py installAll
#执行命令部署服务将自动部署FISCO BCOS节点，并部署 WeBASE 中间件服务，包括签名服务（sign）、节点前置（front）、节点管理服务（node-mgr）、节点管理前端（web）
```

详细命令:

```
# 一键部署
部署并启动所有服务       python3 deploy. py installAll
停止一键部署的所有服务    python3 deploy.py  stopAll
启动一键部署的所有服务    python3 deploy.py  startAll
#各子服务启停
启动FISCO-BCOS节点       python3 deploy.py startNode
停止HSCO-BCOS节点        python3 deploy.py stopNode
启动WeBASE-Web: python3 deploy.py startWeb
停止WeBASE-Web: python3 deploy.py stopWeb
启动WeBASE-Node-Manager: python3 deploy.py startManager
停止WeBASE-Node-Manager: python3 deploy.py stopManager
启动WeBASE-Sign: python3 deploy.py startSign
停止WeBASE-Sign: python3 deploy.py stopSign
启动WeBASE-Front: python3 deploy.py startFront
停止WeBASE-Front: python3 deploy.py stopFront
# 可视化部署
部署并启动可视化部署的所有服务 python3 deploy.py installWeBASE
停止可视化部署的所有服务 python3 deploy.py stopWeBASE
启动可视化部署的所有服务 python3 deploy.py startWeBASE
```

检测操作

```
#节点进程
ps -ef | grep node
#检查webase-front
ps -ef | grep webase.front
#检查webase-node-mannager
ps -ef  | grep webase.node.mgr
#检查nignx
ps -ef | grep nginx
#检查webase-sign
ps -ef  | grep webase.sign

#检查端口
netstat -anlp | grep 20200 
netstat -anlp | grep 5002
netstat -anlp | grep 5001
netstat -anlp | grep 5000
netstat -anlp | grep 5004
```

**localhost:5002/WeBASE-Front 为节点控制台**

**http://localhost:5000 webase管理平台**



# 企业搭建webase

Webase-Front需要创建数据库

```sql
CREATE DATABASE IF NOT EXISTS webase_sign_tb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

Webase-Node-Manager需要创建数据库

```sql
CREATE DATABASE IF NOT EXISTS webase_node_mgr_tb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

在配置Front时需要将fisco-bcos的sdk密钥copy到dist/conf内

```
cp -r nodes/127.0.0.1/sdk/* ./dist/conf/
```

在配置Front中application.yml中的nodePath的路径需要配置到node0

WeBASE-Node-Manager配置的时候需要先配置script/webase.sh文件都数据库进行配置，然后使用

```sh
bash webase.sh ip port  #进行启动
```

Webase-Web配置需要先下载solc-bin:`bash ./get_solc_js.sj`

# generator

## 笔记

先从群组1开始搭建,搭建好后再将各个节点的密钥复制进去。

`./generator/script/install.sh`进行generator的安装。

`./generator --cdn --download_fisco ./meta`将fisco的二进制文件拷贝进来

`./one_click_generator.sh -b ./tmp_one_click`将配置好的机构进行生成

`tail -f ./tmp_one_click/agency*/node/node*/log/log* |grep g:x |grep +++ `查询状态

`./generator --download_console ./ --cdn`将控制台拷贝下来

`bash ./tmp_one_click/agencyx/node/node_xx_xx/scrpipts/load_new_groups.sh`生成新的群组

`./generator --generate_chain_certificate ./dir_chain_ca`生成链证书

在生成第二个节点时:需要将生成第一个节点的ca文件拷贝过去即:

```sh
cp tmp_one_click/ca.* tmp_one_click_expand/
```

以及群组1的创世区块,和p2p的连接文件`peers.txt`

```sh
cp tmp_one_click/group.1.genesis tmp_one_click_expand
cp tmp_one_click/peers.txt tmp_one_click_expand
```

如果新增的节点是在已有的机构上新增的化，需要将已有机构的密钥拷贝过去.

```sh
cp -r tmp_one_click/agencyA/agency_cert tmp_one_click_expand/agencyA
```

新增群组时，先要将对应的证书拷贝过来，比如机构A有两个节点，就需要将两个证书拷贝过来。

```sh
cp tmp_one_click/agencyA/node/node_xxx_xx/conf/node.crt meta/cert_xxx_xx.crt
```

新增群组需要新增创世区块,并且将创世区块加入到现有节点中:

```sh
./generator --create_group_genesis ./group2
./generator --add_group ./group2/group.2.genesis ./tmp_one_click_expand/agencyx/node/node_xx_x
```

全部内容完成之后清理meta文件

```sh
rm ./meta/cert_*
rm ./meta/group*
```

## 实操

前置要求:8节点4机构3群组

![要求图](https://i-blog.csdnimg.cn/blog_migrate/542a9057b81a2dc6540e7f497f397f9d.png)



| 机构  | 节点   | 所属群组  | P2P地址 | RPC监听地址 | Channel监听地址 | IP地址    |
| ----- | ------ | --------- | ------- | ----------- | --------------- | --------- |
| 机构A | 节点0  | 群组1,2,3 | 30300   | 20200       | 8545            | 127.0.0.1 |
|       | 节点1  | 群组1,2,3 | 30301   | 20201       | 8546            | 127.0.0.1 |
| 机构B | 节点2  | 群组1,2   | 30302   | 20202       | 8547            | 127.0.0.1 |
|       | 节点3  | 群组1,2   | 30303   | 20203       | 8548            | 127.0.0.1 |
| 机构C | 节点4  | 群组2，4  | 30304   | 20204       | 8549            | 127.0.0.1 |
|       | 节点5  | 群组2，4  | 30305   | 20205       | 8550            | 127.0.0.1 |
| 机构D | 节点6  | 群组3     | 30306   | 20206       | 8551            | 127.0.0.1 |
|       | 节点7  | 群组3     | 30307   | 20207       | 8552            | 127.0.0.1 |
| 机构F | 节点8  | 群组4     | 30308   | 20208       | 8553            | 127.0.0.1 |
|       | 节点9  | 群组4     | 30309   | 20209       | 8554            | 127.0.0.1 |
|       | 节点10 | 群组4     | 30310   | 20210       | 8555            | 127.0.0.1 |
| 机构E | 节点11 | 群组1     | 30311   | 20211       | 8556            | 127.0.0.1 |

由于是四个机构，所以复制四个目录

```sh
cp -r generator/ generator-A
cp -r generator/ generator-B
cp -r generator/ generator-C
cp -r generator/ generator-D
```

```sh
git clone http://gitlab.rs.nlecloud.com/fisco/generator.git
```

### 第一步生成链

一条链只有一条唯一的链证书`ca.crt`

```sh
./generator --generate_chain_certificate ./dir_chain_ca
```

然后生成各个机构的证书，并且复制到工作目录，以便于使用

```sh
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyA
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyB
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyC
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyD
```

将这些机构认证文件拷贝到各自目录

```
cp -r dir_agency_ca/agencyA/* ../generator-A/meta/
cp -r dir_agency_ca/agencyB/* ../generator-B/meta/
cp -r dir_agency_ca/agencyC/* ../generator-C/meta/
cp -r dir_agency_ca/agencyD/* ../generator-D/meta/
```

### 第二部编辑deployment并生成

```sh
vim generator-x/node_deployment.ini
```

对应的端口填写上。

生成机构A，B的证书和p2p连接信息文件

```sh
./generator --generate_all_certificates ./agencyA_node_info
```

将机构A的peers.txt连接信息文件拷贝给机构B 用于进行连接

```sh
cp agencyA_node_info/peers.txt ../generator-B/meta/peersA.txt
```

`ps: 由于群组1 需要创建创世区块，我们选择在机构A里进行创建，所以我们机构B除了复制peers.txt文件外，还需要复制机构B的两个节点crt文件给机构A`

```sh
cp -r  agencyB_node_info/peers.txt  ../generator-A/meta/peersB.txt
cp -r agencyB_node_info/cert_*  ../generator-A/meta/
```

在机构A结合四个节点证书文件生成创世区块,并将创世区块给b

```sh
./generator --create_group_genesis ./group
cp group/group.1.genesis  ../generator-B/meta/
```

在机构A中使用`--build_install_package`，第一个参数是放置群组内其他节点的连接信息，第二个是节点文件的输出目录

```sh
./generator --build_install_package ./meta/peerB.tst ./nodeA
```

同理生成B，不写了

查询共识情况:

```sh
tail -f ./node*/node*/log/log*  | grep g:2.*+++
```

### 第三步群组2

编辑机构C的`conf/node_deployment.ini`配置文件，对应的端口编辑好,`group_id=2`

然后生成证书连接信息,并拷贝给A，以及将A的证书和连接信息拷贝到C

```sh
./generator --generate_all_certificates ./agencyC_node_info
cp -r agencyC_node_info/peers.txt  ../generator-A/meta/peersC.txt
cp -r ../generator-A/agencyA_node_info/cert*  meta/
cp -r ../generator-A/agencyA_node_info/peers.txt  meta/peersA.txt
```

修改C的`conf/group_genesis.ini`将对应的端口写上并且将`group_id`改成2

在机构C生成群组2的创世区块文件,并且拷贝到机构A

```sh
./generator --create_group_genesis ./group
cp -r group/group.2.genesis ../generator-A/meta/
```

生成机构C节点并且启动

```
./generator --build_install_package ./meta/peersA.txt ./nodeC
```

机构A为现有节点初始化群组2

使用`--add_group`第一个参数选择群组2的创世块,第二个参数选择需要加入的群组的节点目录

```sh
./generator --add_group ./meta/group.2.genesis ./nodeA
```

使用`--add_peers` ,添加机构C节点连接文件peers至已有节点

```sh
./generator --add_peers ./meta/peersC.txt ./nodeA
```

重启机构A

查询A，C机构是否有共识

```sh
tail -f ./node*/node*/log/log* | grep g:2.+++
```

群组3搭建方法与2一致
