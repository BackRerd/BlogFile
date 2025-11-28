---
title: SpringBoot连接fisco
tags:
  - Java
  - SpringBoot
  - Fisco
abbrlink: b8c034a0
date: 2024-11-27 13:57:19
categories: ["区块链"]
---

# 前置条件

- SpringBoot3
- Fisco3
- SpringWeb

# 1.Fisco区块链网络

搭建过程略过。

在控制台文件中~/console/输入命令

```sh
bash contract2java.sh solidity -s contracts/solidity/HelloWorld.sol -p site.backrer.fiscotest.hello.contract
```

- `contracts/solidity/HelloWorld.sol`目标合约
- `site.backrer.fiscotest.hello.contract`编译后的Java包名，和自己的对应好就可以

# 2.导入SpringBoot

## 导入所需要的依赖

maven:

```xml
<dependency>
	<groupId>org.fisco-bcos.java-sdk</groupId>
	<artifactId>fisco-bcos-java-sdk</artifactId>
	<version>3.6.0</version>
</dependency>
```

## 创建toml文件配置fisco

`config-fisco.toml`这是我的命名方式，配置文件信息

```toml
[cryptoMaterial]

certPath = "conf"                           # The certification path
useSMCrypto = "false"
enableSsl = "true"                        # Communication with nodes without SSL

# The following configurations take the certPath by default if commented
caCert = "conf/ca.crt"                    # CA cert file path
sslCert = "conf/sdk.crt"                  # SSL cert file path
sslKey = "conf/sdk.key"                   # SSL key file path

# The following configurations take the sm certPath by default if commented
# caCert = "conf/sm_ca.crt"                    # SM CA cert file path
# sslCert = "conf/sm_sdk.crt"                  # SM SSL cert file path
# sslKey = "conf/sm_sdk.key"                    # SM SSL key file path
# enSslCert = "conf/sm_ensdk.crt"               # SM encryption cert file path
# enSslKey = "conf/sm_ensdk.key"                # SM ssl cert file path

[network]
messageTimeout = "10000"
defaultGroup = "group0"
peers=["192.168.133.130:20201"]    # The peer list to connect


[account]
keyStoreDir = "account"         # The directory to load/store the account file, default is "account"
# accountFilePath = ""          # The account file path (default load from the path specified by the keyStoreDir)
accountFileFormat = "pem"       # The storage format of account file (Default is "pem", "p12" as an option)

# accountAddress = ""           # The transactions sending account address
# Default is a randomly generated account
# The randomly generated account is stored in the path specified by the keyStoreDir

# password = ""                 # The password used to load the account file

[threadPool]
threadPoolSize = "16"         # The size of the thread pool to process message callback
# Default is the number of cpu cores
```

`cryptoMaterial`配置密钥和证书的所在位置，提前将密钥和证书导入到项目中resources文件下

正常密钥和证书位置在fisco文件下`nodes/127.0.0.1/sdk`中。

`network`配置详细

- `messageTimeout`最大连接时间tick
- `defaultGroup`默认第一个连接的群组
- `peers`连接的节点ip，可配置多个

# 3.连接

在任意类中新建静态方法:

```java
public final static String configFile =BcosSDKTest.class.getClassLoader().getResource("config-fisco.toml").getPath();
```

`config-fisco.toml`其中的文件名称填写自己的文件名称

`BcosSDKTest`是当前类的名称

这样springboot和fisco连接就算完成了。

# 4.测试

在这里做个测试,获取块的高度，以及调用HelloWorld合约中的get和set方法:

```java
@RestController()
@RequestMapping("/fisco")
public class test {
    private final BcosSDK sdk =  BcosSDK.build(BcosSDKTest.configFile);
    private final Client client = sdk.getClient("group0");
    private HelloWorld helloWorld;
    //通过getAddress获取的地址
    private String address = "0x745d4de0cf93b7d1db8dd8892daf05ac745766ce";
    
    //部署HelloWorld合约
    @GetMapping("/deploy")
    public void initSDK() throws ContractException {
        // 向群组1部署HelloWorld合约
        CryptoKeyPair cryptoKeyPair = client.getCryptoSuite().getCryptoKeyPair();
        helloWorld = HelloWorld.deploy(client,cryptoKeyPair);
        System.out.println("-----deploy HelloWorld ok------");
    }
    //获取HelloWorld合约的地址以便于以后不用每次都部署
    @GetMapping("/getAddress")
    public String getAddress(){
        return helloWorld.getContractAddress();
    }
    //读取合约地址
    @GetMapping("/read")
    public String readHell() {
        helloWorld = HelloWorld.load(address,client,client.getCryptoSuite().getCryptoKeyPair());
        return "ok";
    }
    //获取区块链高度
    @GetMapping("/getBlockNumber")
    public String getBlockNumber(){
        System.out.println("-----getBlockNumber------");
        if(client == null){
            System.out.println("-----init BcosSDK failed------");
            return "-----init BcosSDK failed----";
        }
        BlockNumber blockNumber = client.getBlockNumber();
        return "block number is : " + blockNumber.getBlockNumber().toString();
    }
    //调用HelloWorld合约中的get方法
    @GetMapping("/get")
    public String getHelloWorld() throws ContractException {
        System.out.println("-----getHelloworld------");
        if(helloWorld == null){
            System.out.println("-----init BcosSDK failed------");
            return "-----init BcosSDK failed----";
        }
        String getValue = helloWorld.get();
        System.out.println("-----call HelloWorld get success------:" + getValue);
        return getValue;
    }
    //调用HelloWorld合约中的set方法
    @GetMapping("/set")
    public String setHelloWorld(@RequestParam(value = "val",required = false,defaultValue = "default val")String val) throws ContractException {
        System.out.println("-----setHelloworld------");
        if(helloWorld == null){
            System.out.println("-----init BcosSDK failed------");
            return "-----init BcosSDK failed----";
        }
        TransactionReceipt receipt = helloWorld.set(val);
        System.out.println("-----call HelloWorld get success------:" + receipt.getMessage());
        return "setHelloworld success";
    }
    
}
```



