---
title: webase-front搭建方法
tags:
  - fisco
abbrlink: 4402d474
date: 2025-01-08 11:45:45
categories: ["区块链"]
---

前置条件:

| fisco   | >=2.0          |
| ------- | -------------- |
| openssl |                |
| curl    |                |
| java    | >=jdk8,jdk14<= |

先正确搭建一个区块链，看以往教程。

# 需构建版

拉取webase-front并进行编译(需构建版)。

```sh
git clone https://gitee.com/WeBank/WeBASE-Front.git
cd WeBASE-front
# 编译代码
chmod +x ./gradlew && ./gradlew build -x test

cp -r dist/conf_template dist/conf #拷贝模版
cp ~/fisco/nodes/127.0.0.1/sdk/* dist/conf/ #拷贝证书文件
vim dist/conf/application.yml #修改配置
```

`./gradlew build -x test`途中会出现多次失败，尤其是第一步下载gradle6.6.1所以如果出现BUILD FAILED类似字眼就多试几次。

# 免构建版（建议）

```sh
wget https://github.com/WeBankBlockchain/WeBASELargeFiles/releases/download/v1.5.5/webase-front.zip
cd webase-front/
cp ../fisco/nodes/127.0.0.1/sdk/* ./conf/ #复制密钥
```

# 检查与使用

查看启动是否成功:

```sh
grep -B 3 "main run success" log/WeBASE-Front.log
```

出现与下方相似即是启动成功

![image.png](https://s2.loli.net/2025/01/08/saBLk8yZgxHWEG6.png)

`main run success`代表启动成功。

访问网站:http://localhost:5002/WeBASE-Front/

操作与webase差不多
