# 基于FISCO 3.4.0部署DDCMS分布式数据管理

> 作者： 张宇豪 
>
> 学校：深圳职业技术大学

## 1、DDCMS介绍

### 1.1、什么是DDCMS？

`DDCMS(Distributed Data Collaboration Management Solution)`旨在多方协作场景中，面向数据管理需求，提供一套基于区块链的、安全可信、友好易用的分布式数据管理开源方案，服务于个人数据携带、企业间数据共享。各参与方围绕数据目录展开业务协作，使得各参与方能够快速、低成本的进行数据管理和共享、确保数据安全和隐私的同时，实现数据共享全流程可追溯、可监管、可审计。

### 1.2、DDCMS 角色介绍

DDCMS 角色介绍 在DDCMS中包括5中角色：数据归属方、数据提供方、数据使用方、数据见证方、系统运营方。

- 数据归属方：在数据传输共享过程中，对数据使用方的数据请求进行数据授权，在数据提供方进行授权管理。

- 数据提供方：对外提供数据目录服务，负责审核数据目录使用申请及核验用户授权。

- 数据使用方：对数据归属者提供服务，并经过授权通过数据目录获取数据。

- 数据见证方：负责对数据目录的全流程进行进行审核监管，不参与具体业务。

- 系统运营方：负责对各参与方进行KYC，确保参与方可信可靠。

  

谈到DDCMS时，可以将其比作一个分布式的云盘系统。传统的云盘系统将用户的文件都存储在一个中心服务器上，而DDCMS则采用了一种不同的方法。

想象一下，DDCMS就像是把你的文件分散保存在很多个硬盘上，这些硬盘被放置在世界各地的人们的电脑上。你的文件会被拆分成很多小块，并且复制到不同的电脑上进行备份。因此，即使某个电脑出现问题，你的文件仍然能够通过其他电脑进行访问。

这种方式有一些好处。首先，由于文件被分散保存在多个地方，因此单个电脑出现故障不会导致数据丢失。其次，因为文件存在于多个电脑上，所以即使有个别电脑无法连接，你仍然可以通过其他电脑来获取你的文件。

换句话说，DDCMS让你的文件不再受限于一个中心服务器的控制，而是分布在许多电脑上，形成了一个去中心化的存储网络。这样一来，你可以更安全地存储和共享文件，并且不容易受到限制或审查。

总之，DDCMS提供了一种分布式的文件存储和管理系统，类似于将文件分散保存在许多电脑上的云盘。这种方式让数据更安全，用户更加自由地管理和共享文件。



## 2、配置环境依赖

### 2.1、节点规划

> 单机部署

| 环境依赖         | 描述                    | 节点      |
| ---------------- | ----------------------- | --------- |
| JDK11            | Java环境                | localhost |
| MySQL8           | 数据库环境              | localhost |
| FISCO BCOS 3.4.0 | FISCO BCOS 普通安装环境 | localhost |
| NodeJs16.20      | 前端的依赖环境          | localhost |
| Ubuntu20.04      | 部署的操作系统环境      | localhost |



### 2.2、检查环境

在work的工作目录中，拉取三个项目。

- DDCMS-Contract项目是提供合约部署
- DDCMS-Service项目是提供Java的后端
- DDCMS-Front项目是提供React的前端

> 注意： git命令工具需要自己安装，apt-get install git

```shell
root@fb8gvqsmw00vu58:~/work# git clone https://github.com/WeBankBlockchain/DDCMS-Contract.git
Cloning into 'DDCMS-Contract'...
remote: Enumerating objects: 162, done.
remote: Counting objects: 100% (162/162), done.
remote: Compressing objects: 100% (111/111), done.
remote: Total 162 (delta 79), reused 122 (delta 44), pack-reused 0
Receiving objects: 100% (162/162), 129.45 KiB | 712.00 KiB/s, done.
Resolving deltas: 100% (79/79), done.
root@fb8gvqsmw00vu58:~/work# git checkout origin/dev

root@fb8gvqsmw00vu58:~/work# cd DDCMS-Contract/
root@fb8gvqsmw00vu58:~/work/DDCMS-Contract# git checkout origin/dev


root@fb8gvqsmw00vu58:~/work# git clone https://github.com/WeBankBlockchain/DDCMS-Service.git
Cloning into 'DDCMS-Service'...
remote: Enumerating objects: 8044, done.
remote: Counting objects: 100% (1329/1329), done.
remote: Compressing objects: 100% (628/628), done.
remote: Total 8044 (delta 613), reused 1118 (delta 482), pack-reused 6715
Receiving objects: 100% (8044/8044), 1.45 MiB | 3.21 MiB/s, done.
Resolving deltas: 100% (4109/4109), done.

root@fb8gvqsmw00vu58:~/work# cd DDCMS-Service
root@fb8gvqsmw00vu58:~/work/DDCMS-Service# git checkout origin/dev


root@fb8gvqsmw00vu58:~/work# git clone https://github.com/WeBankBlockchain/DDCMS-Front.git
Cloning into 'DDCMS-Front'...
remote: Enumerating objects: 2213, done.
remote: Counting objects: 100% (506/506), done.
remote: Compressing objects: 100% (171/171), done.
remote: Total 2213 (delta 328), reused 491 (delta 318), pack-reused 1707
Receiving objects: 100% (2213/2213), 935.41 KiB | 513.00 KiB/s, done.
Resolving deltas: 100% (1524/1524), done.

root@fb8gvqsmw00vu58:~/work# cd DDCMS-Front
root@fb8gvqsmw00vu58:~/work/DDCMS-Front# git checkout origin/dev

# 查看当前目录下的所有项目
root@fb8gvqsmw00vu58:~/work# ls -l
total 12
drwxr-xr-x 6 root root 4096 Sep  1 05:29 DDCMS-Contract
drwxr-xr-x 5 root root 4096 Sep  1 05:31 DDCMS-Front
drwxr-xr-x 6 root root 4096 Sep  1 05:30 DDCMS-Service
```



### 2.3、检查依赖

> 注意：我们需要底层的环境是Java JDK8+ 、MySQL8、Nodejs16+、Openssl，如果需要预装环境依赖，可以使用我的一键部署脚本进行安装依赖：https://blog.csdn.net/weixin_46532941/article/details/129910073?spm=1001.2014.3001.5502

检查当前的Java版本以及MySQL的版本

```shell
# 查看当前的Java版本
root@fb8gvqsmw00vu58:~/work# java --version
openjdk 11.0.19 2023-04-18
OpenJDK Runtime Environment (build 11.0.19+7-post-Ubuntu-0ubuntu120.04.1)
OpenJDK 64-Bit Server VM (build 11.0.19+7-post-Ubuntu-0ubuntu120.04.1, mixed mode, sharing)

# 查看当前的MySQL版本
root@fb8gvqsmw00vu58:~/work# mysql -V
mysql  Ver 8.0.33-0ubuntu0.20.04.2 for Linux on x86_64 ((Ubuntu))

# 安装Node的环境
root@fb8gvqsmw00vu58:~/work# curl -o- https://gitee.com/mirrors/nvm/raw/v0.33.2/install.sh | bash
# 加载nvm配置
root@fb8gvqsmw00vu58:~/work# source ~/.$(basename $SHELL)rc
# 安装Node.js 16
root@fb8gvqsmw00vu58:~/work# nvm install 16
# 使用Node.js 16
root@fb8gvqsmw00vu58:~/work# nvm use 16
```



## 3、DDCMS部署

### 3.1、部署FISCO BCOS

> 部署链接：https://fisco-bcos-doc.readthedocs.io/zh_CN/latest/docs/quick_start/air_installation.html

这里我们需要部署FISCO BCOS 3.4.0版本，DDCMS提供的智能合约版本为0.8.0+。因为我试过了2.9.2的FISCO BCOS版本，但是需要手动修改Solidity的版本和合约代码，所以直接上最新的3.4.0是没问题。其他的3.0版本还没测试如下部署部署FISCO BCOS3.4.0以及配置控制台的操作：

```shell
root@fb8gvqsmw00vu58:~/work# curl -#LO https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/FISCO-BCOS/FISCO-BCOS/releases/v3.4.0/build_chain.sh && chmod u+x build_chain.sh

root@fb8gvqsmw00vu58:~/work# bash build_chain.sh -l 127.0.0.1:4 -p 30300,20200
root@fb8gvqsmw00vu58:~/work# bash nodes/127.0.0.1/start_all.sh 
try to start node0
try to start node1
try to start node2
try to start node3
 node2 start successfully pid=2238732
 node0 start successfully pid=2238728
 node1 start successfully pid=2238735
 node3 start successfully pid=2238738

root@fb8gvqsmw00vu58:~/work#  curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/download_console.sh && bash download_console.sh
root@fb8gvqsmw00vu58:~/work# cp console/conf/config-example.toml console/conf/config.toml 
root@fb8gvqsmw00vu58:~/work# cp nodes/127.0.0.1/sdk/* console/conf/
root@fb8gvqsmw00vu58:~/work# bash console/start.sh 
=============================================================================================
Welcome to FISCO BCOS console(3.4.0)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______  
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \ 
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \ 
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group0]: /apps> 
[group0]: /apps> getGroupInfo
{
    "chainID":"chain0",
    "groupID":"group0",
    ······
}
```

这里记住一下是group0，或者等下配置的时候直接使用group0。



### 3.2、部署DDCMS合约

- 将默认的`DDCMS-Contract`项目中的`contracts`目录的所有合约拷贝到当前的控制台中的`contracts/solidity/`目录中
- 使用`get_account.sh` 脚本生成一个新的账户，然后使用该账户启动控制台

> 注意： 这里需要复制当前的账户私钥，当前的账户私钥是：0xa25decb8c3fe224ff5486aef440073b78eae553686088ee121d67185ccad0a92

```shell
root@fb8gvqsmw00vu58:~/work# cp -rf DDCMS-Contract/contracts/* console/contracts/solidity/
root@fb8gvqsmw00vu58:~/work# cd console/
root@fb8gvqsmw00vu58:~/work/console# bash get_account.sh 
[INFO] Account privateHex: 0xa25decb8c3fe224ff5486aef440073b78eae553686088ee121d67185ccad0a92
[INFO] Account publicHex : 0x07152568b6cd3e58f74fadb0dd6ec92574e417b8b13c56a83f36ec7dc333942c66bb70b8ac2a0ca798f6d2a6dbff1355d2997a59f78284ba3169cb62f9842f81
[INFO] Account Address   : 0x8c75e50c943e715311229d60eab2e9cdbd93efa1
[INFO] Private Key (pem) : accounts/0x8c75e50c943e715311229d60eab2e9cdbd93efa1.pem
[INFO] Public  Key (pem) : accounts/0x8c75e50c943e715311229d60eab2e9cdbd93efa1.pem.pub
root@fb8gvqsmw00vu58:~/work/console# bash start.sh group0 -pem accounts/0x8c75e50c943e715311229d60eab2e9cdbd93efa1.pem
=============================================================================================
Welcome to FISCO BCOS console(3.4.0)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______  
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \ 
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \ 
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group0]: /apps> 
```

然后分别按顺序部署三个合约，分别是`AccountContract`、`ProductContract`、`DataSchemaContract`。

- ProductContract合约中需要初始化构造函数：需要传入AccountContract合约的地址
- DataSchemaContract合于中需要初始化构造函数：需要传入第一个参数为AccountContract合约的地址，第二个参数为ProductContract的合约地址

```shell
# 部署AccountContract合约
[group0]: /apps> deploy AccountContract 
transaction hash: 0x4b035a62fbd12f1f5c810a6c02b230ac06272ced4865cc08fe3fca655c74ff03
contract address: 0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1
currentAccount: 0x8c75e50c943e715311229d60eab2e9cdbd93efa1

# 部署ProductContract合约
[group0]: /apps> deploy ProductContract 0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1
transaction hash: 0x2747d88239377533e3392073cf854603acba0f104573c17db4d1c25e15b76a52
contract address: 0x4721d1a77e0e76851d460073e64ea06d9c104194
currentAccount: 0x8c75e50c943e715311229d60eab2e9cdbd93efa1

# 部署DataSchemaContract合约
[group0]: /apps> deploy DataSchemaContract 0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1 0x4721d1a77e0e76851d460073e64ea06d9c104194 
transaction hash: 0x4e8c10706b9d498914604d2037f144d0a3396521b01ad4a4791e4162b03f5664
contract address: 0xc8ead4b26b2c6ac14c9fd90d9684c9bc2cc40085
currentAccount: 0x8c75e50c943e715311229d60eab2e9cdbd93efa1
```



### 3.3、配置数据库

- 创建一个名称为`ddcms`的数据库
- 然后使用`source`将`db-script.sql`文件，导入到该数据库中。

> 注意： /root/work/DDCMS-Service/src/main/resources/scripts/db-script.sql 这是我的绝对路径

```mysql
root@fb8gvqsmw00vu58:~/work/console# mysql -uroot -p123546
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 640
Server version: 8.0.33-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database ddcms;
Query OK, 1 row affected (0.01 sec)

mysql> use ddcms;
Database changed
mysql> source /root/work/DDCMS-Service/src/main/resources/scripts/db-script.sql
···
Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.01 sec)
```



### 3.4、配置后端

进到DDCMS-Service的项目中，对项目进行编译构建。

```shell
root@fb8gvqsmw00vu58:~/work# cd DDCMS-Service/
root@fb8gvqsmw00vu58:~/work/DDCMS-Service# ./gradlew bootJar

> Configure project :
delete /root/work/DDCMS-Service/dist

> Task :compileJava
Note: /root/work/DDCMS-Service/src/main/java/com/webank/ddcms/config/SecurityConfig.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

See https://docs.gradle.org/7.4/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 6s
4 actionable tasks: 4 executed

```

然后构建成功之后，这里需要复制两个默认的配置文件，然后修改application.yml。

```shell
root@fb8gvqsmw00vu58:~/work/DDCMS-Service# cd dist/conf/
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist/conf# ls
application-example.yml  config-example.toml
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist/conf# cp config-example.toml config.toml 
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist/conf# cp application-example.yml application.yml 
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist/conf# cp /root/work/console/conf/* . 
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist/conf# ls -l
total 44
-rw-r--r-- 1 root root 2436 Sep  1 06:29 application-example.yml
-rw-r--r-- 1 root root 2750 Sep  1 06:35 application.yml
-rw-r--r-- 1 root root 1103 Sep  1 06:45 ca.crt
-rw-r--r-- 1 root root  753 Sep  1 06:45 cert.cnf
-rw-r--r-- 1 root root  208 Sep  1 06:45 clog.ini
-rw-r--r-- 1 root root 2001 Sep  1 06:45 config-example.toml
-rw-r--r-- 1 root root 2001 Sep  1 06:45 config.toml
-rw-r--r-- 1 root root  914 Sep  1 06:45 log4j2.xml
-rw-r--r-- 1 root root 1131 Sep  1 06:45 sdk.crt
-rw------- 1 root root 1704 Sep  1 06:45 sdk.key
-rw-r--r-- 1 root root  513 Sep  1 06:45 sdk.nodeid
```

如下是我修改好的配置：

> 注意：https://ddcms-docs.readthedocs.io/en/latest/docs/DDCMS-Service/index.html 这里有很详细的参数配置，我们只需要修改如下的主要配置：
>
> - 数据库用户密码、数据库名称
> - 登录用户密码
> - jwt的secret
> - 部署合约的账户私钥、以及三个合约地址

```yaml
server:
  port: 10880

spring:
  mvc:
    async:
      request-timeout: 500
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/ddcms?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true
    username: root # 数据库用户名
    password: 123456 # 数据库用户密码
    driverClassName: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    defaultAutoCommit: false
    initialSize: 25
    maxActive: 128
    maxIdle: 50
    minIdle: 25
    testOnBorrow: false
    testOnReturn: false
    testWhileIdle: true
    validationQuery: SELECT 1
    timeBetweenEvictionRunsMillis: 34000
    minEvictableIdleTimeMillis: 34000
    removeAbandoned: false
    logAbandoned: true
    removeAbandonedTimeout: 54
    maxWait: 10000
    poolPreparedStatements: false
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

mybatis:
  typeAliasesPackage: com.webank.ddcms.dao.entity
  configuration:
    map-underscore-to-camel-case: true

jwt:
  secret: "Ok2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQ" 
  expiration: 8640000

auth:
  permitAllApiList:
    - /api/account/register
    - /api/account/pageQueryCompany
    - /api/account/queryCompanyByUsername
    - /api/account/queryPersonByUsername
    - /api/account/queryCompanyByAccountId
    - /api/account/getHotCompanies
    - /api/account/searchPerson
    - /api/schema/pageQuerySchema
    - /api/schema/querySchemaById
    - /api/schema/querySchemaAccessById
    - /api/file/download
    - /api/file/upload
    - /api/product/getHotProducts
    - /api/product/pageQueryProduct
    - /api/product/queryProductById
    - /api/tag/getHotTags
    - /v2/api-docs
    - /configuration/**
    - /swagger*/**
    - /webjars/**
    - /csrf
    - /
  anonymousApi: /api/account/login
  roleAuth:
    companyAuth:
      - /api/product/createProduct
      - /api/schema/createSchema
      - /api/product/updateProduct
    witnessAuth:
      - /api/product/approveProduct
    adminAuth:
      - /api/account/searchCompany
      - /api/account/approveAccount
system:
  bcos-cfg: "conf/config.toml"
  bcos-group-id: "group0"
  crypto-type: 0
  admin-account: 'admin'		# 登录用户
  admin-password: 'admin'		# 登录密码
  admin-private-key: '0xa25decb8c3fe224ff5486aef440073b78eae553686088ee121d67185ccad0a92' # 部署合约的管理员的私钥
  admin-company: '可信第三方'
  contractConfig:
    account-contract: "0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1" # account合约地址
    product-contract: "0x4721d1a77e0e76851d460073e64ea06d9c104194"  # product合约地址
    dataSchema-contract: "0xc8ead4b26b2c6ac14c9fd90d9684c9bc2cc40085" # dataSchema合约地址
  fileConfig:
    file-dir: "tmp"
swagger:
  enabled: true
```

主要修改的配置对应的图如下：

![image-20230901063249900](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010633478.png)

![image-20230901063517663](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010635741.png)



### 3.5、启动后端项目

> 注意：这里会有一点小问题存在，比如我是用的是root用户，可能会涉及到mysql不能远程连接的问题。

```shell
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist# bash start.sh 
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist# nohup: appending output to 'nohup.out'
```

报错的信息如下：

![image-20230901064951238](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010649352.png)

> 其实这里我可以提供两种完美的解决方案：

**第一种：**

直接创建一个用户支持远程连接访问。详细如下：

- 在上面的配置文件中，你可以直接使用如下的用户密码进行连接数据库操作

```sql
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist# mysql -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1136
Server version: 8.0.33-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create user 'ddcms'@'%' identified by '123456';
Query OK, 0 rows affected (0.02 sec)

mysql> grant all privileges on *.* to 'ddcms'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```



**第二种：**

修改root用户为可以远程连接访问：

```sql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | ddcms            |
| %         | webase           |
| localhost | debian-sys-maint |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
| webase    | webase           |
+-----------+------------------+
8 rows in set (0.00 sec)

mysql> update user set host ='%' where user='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

然后修改一下mysql的配置文件：

```shell
root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist# vim /etc/mysql/mysql.conf.d/mysqld.cnf
#将文件中的bind-address=127.0.0.1注释掉，在这行语句前面加 # 即为注释
#bind-address=127.0.0.1

root@fb8gvqsmw00vu58:~/work/DDCMS-Service/dist# systemctl restart mysql
```

![image-20230901065550388](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010655433.png)



> 重新启动后端项目：
>
> bash stop.sh && bash start.sh

![image-20230901065936038](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010659146.png)



### 3.5、编译构建前端

> 注意： 这里部分的node18版本可以直接使用npm isntall，然后使用npm start启动项目

```shell
root@fb8gvqsmw00vu58:~/work/DDCMS-Front# npm install -g yarn
root@fb8gvqsmw00vu58:~/work/DDCMS-Front# yarn
root@fb8gvqsmw00vu58:~/work/DDCMS-Front# yarn start
```

浏览器访问： http://localhost:3000

![image-20230901070805433](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010708517.png)

点击登录，用户admin，密码为设置的admin。

![image-20230901070916285](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309010709352.png)