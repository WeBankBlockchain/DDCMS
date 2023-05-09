# Data-Brain

数据要素已成为数字经济时代最核心的生产要素，各种数据共享和交易解决方案不断出现，极大的推动了产业数字化的落地和发展。但由于缺乏统一可信的数据发布管理能力、数据传输的不透明不安全等已经无法满足日益增长的数字产业需求。

- 在多方协同合作场景下，需要一套可控、可信、可靠的面向数据管理的解决方案，降低企业管理、对接数据目录的成本。
- 在数据传输过程中，需要明确数据的提供方、使用方、使用场景、数据用量、使用时间等信息，确保个人数据可携带权的同时，实现数据传输全流程的可追溯可审计；
- 在数据传输和使用过程中，需要确保数据安全，防止数据被盗或过度暴露的同时，实现数据的可用而不可见，达到使用和挖掘数据价值的目的；

为解决以上问题，我们希望通过Data Brain项目的建设，在多方协同合作场景中，面向数据管理需求，提供一套基于区块链的、安全可信、友好易用的分布式数据管理开源方案，服务于个人数据携带、企业间数据共享。在数据共享过程中，各参与方围绕数据目录展开各种业务协作，使得各参与方能够快速、高效的进行数据管理、数据需求对接、传输和使用数据的同时，确保数据的安全、可信、可靠、可追溯、可监管、可审计。

Data Brain是对DDTP实现数据要素“可信传输”、“协同生产”的落地探索和补充，在降低数据业务对接成本、提升对接效率的同时，确保数据流通过程的安全、可信、可靠。


# Data Brain 角色介绍

在Data Brain中包括5中角色：数据归属方、数据提供方、数据使用方、数据见证方、系统运营方。

![Data Brain 角色](/images/roles.png)

- 数据归属方：在数据传输共享过程中，对数据使用方的数据请求进行数据授权，在数据提供方进行授权管理。
- 数据提供方：对外提供数据目录服务，负责审核数据目录使用申请及核验用户授权。
- 数据使用方：对数据归属者提供服务，并经过授权通过数据目录获取数据。
- 数据见证方：负责对数据目录的全流程进行进行审核监管，不参与具体业务。
- 系统运营方：负责对各参与方进行KYC，确保参与方可信可靠。


# Data Brain核心概念

在DataBrain中，从数据的角度而言，包括三个概念：数据提供方、数据归属业务、数据目录、
- 数据提供方：如前文所述，数据提供方对外提供数据目录，负责审核数据目录使用申请及核验用户授权。
- 数据目录：数据目录简单来说是数据提供方对其提供的数据的描述，包括数据类型、归属业务、归属机构、服务类型、数据定价、数据服务地址、数据传输协议、数据请求格式、数据响应格式等内容。
- 数据业务：数据业务是数据目录所属的系列，每个数据目录均需要挂靠在某个业务上。

它们之间的关系，可以用下图描述：

![](/images/org.png)

# Data Brain核心功能

Data Brain由三个核心功能模块构成： 统一可信的数据目录管理、可追溯审计的数据传输过程、丰富安全的数据处理组件管理。

## 数据目录管理
提供统一的数据目录管理功能，所有操作均上链存证、确保数据目录可信，提升管理和运营效率，降低接入成本。数据目录管理具体又包含用户管理、业务管理、数据目录管理、审核管理等功能。

## 数据传输过程管理
每次数据的传输过程均会上链，确保数据流通的定量、定时、定场景，实现全流程可管理可追溯可监管审计。传输过程管理具体而言，又包括授权认证、过程存证、用量统计、定价及清算等功能。

## 数据处理组件管理
提供对各种数据处理组件的管理能力，企业可按需接入和使用、避免二次开发、降低接入和使用成本。数据处理组件又包含零知识证明、联邦学习、选择性披露、隐私计算等模块。



# 部署方法

Data-Brain由三个子项目构成：Data-Brain-Contract、Data-Brain-Server、Data-Brain-Front。
- Data-Brain-Contract：提供项目的智能合约，将会部署在FISCO BCOS上，它包含了数据目录管理、数据传输管理的留痕存证功能。
- Data-Brain-Server：项目的后台服务器，用于支持前端。
- Data-Brain-Front：项目的前端。



## 环境要求
| 依赖软件 | 说明 |备注|
| --- | --- | --- |
| Java |>= JDK[1.8] | 64bit|
| NodeJs |>= 14| |
| Git | 下载安装包需要使用Git | |

## 合约部署

### 克隆代码
```
git clone https://github.com/WeBankBlockchain/Data-Brain-Contract.git
cd Data-Brain-Contract
git checkout origin/dev
```

Data-Brain合约由三个部分构成：AccountContract、ProductContract、DataSchemaContract，分别管理账户、业务、数据目录。这里面的依赖关系为DataSchemaContract->ProductContract->AccountContract。 这是因为AccountContract记录了各个账户的身份，只有满足特定条件的身份才能进行目录和数据目录的创建、审核等操作；而数据目录隶属于目录，合约需要验证该关系。因此，在部署顺序上，需要先部署AccountContract，然后是ProductContract、DataSchemaContract。


### 链准备
首先需要有一条FISCO BCOS链，如果没有，可以部署一个。FISCO BCOS支持Air版、Pro版、Max版，对应不同的能力和复杂性，具体部署方式可以参考[链搭建文档](https://fisco-bcos-doc.readthedocs.io/zh_CN/latest/docs/tutorial/air/index.html#)。

接下来需要和FISCO BCOS交互，可以使用控制台或者webase。

### 私钥准备
只有可信第三方可以部署合约，所以需要一个可信第三方私钥，可以使用webase或者控制台生成，合约的部署均以可信第三方身份进行。

### 部署合约

然后，依次部署三个合约，以控制台为例，先将contracts目录下的合约拷贝到控制台contracts/solidity目录下，以可信第三方身份启动控制台后执行部署：
```
[group0]: /apps> deploy AccountContract
transaction hash: 0xade6832deb8f9641009336b955a3637bffbbc4fc9ffadb8a9de03ab6200f6093
contract address: 0x59fc2d7bbbb013ac9fc74a4c79bd25fdce02cba2
currentAccount: 0xcbe29631c0933c4319694dafc50723093f0de937

[group0]: /apps> deploy ProductContract 0x59fc2d7bbbb013ac9fc74a4c79bd25fdce02cba2
transaction hash: 0xefcc3503414490eb28dff330b9d42701bd1359f1fc784014f109ba16212fbfac
contract address: 0x24829babadbdc5fe4205353f6a2a6a79e7eef6a5
currentAccount: 0xcbe29631c0933c4319694dafc50723093f0de937

[group0]: /apps> deploy DataSchemaContract 0x59fc2d7bbbb013ac9fc74a4c79bd25fdce02cba2 0x24829babadbdc5fe4205353f6a2a6a79e7eef6a5
transaction hash: 0x27c43f0ca480a5f349205b2dcad6cc045ce4636ee4e41fbddfebb68b60104625
contract address: 0xa085baa5914e1d082c94ce24baf49473caa33ff1
currentAccount: 0xcbe29631c0933c4319694dafc50723093f0de937
```

其中：
- ProductContract部署参数为AccountContract地址，请替换为实际的地址
- DataSchemaContract部署参数为AccountContract地址和ProductContract地址，请替换为实际的地址


## 后端准备

### 克隆代码
```
git clone https://github.com/WeBankBlockchain/Data-Brain-Server.git
cd Data-Brain-Server
git checkout origin/dev
```

### 链配置
由于后端服务需要访问区块链，因此需要先配置链信息，包括证书、sdk配置，这些信息可以直接从sdk里拷贝。

首先拷贝控制台里的证书到Data-Brain-Server工程的conf目录下:

```
cp [控制台目录]/conf/* ./config
```

### 数据库初始化
进入数据库，依次执行resources目录下的数据库脚本：
- scripts/db-script.sql


### 服务配置
在src/main/resources目录下创建application.yml，按照下面的模板进行配置，其中打了${}的需要自己配置。
```
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
    url: ${数据库连接}
    username: ${数据库用户名}
    password: ${数据库密码}
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
  typeAliasesPackage: com.webank.databrain.dao.entity
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
    - /api/account/searchCompany
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
      - /api/account/approveAccount
system:
  bcos-cfg: "config/config.toml"
  bcos-group-id: "group0"
  crypto-type: 0
  admin-account: ${系统运营方账户}
  admin-password: ${系统运营方密码}
  admin-private-key: ${见证方私钥，例如11afa82f974469792aa0172931b813d4fc7dd9177f3211779efc5f955d5e480f}
  admin-company: '系统运营方'
  contractConfig:
    account-contract: "0x1e0171e2f59d00a1851c5f8ca8af1b208f52627b"
    product-contract: "0x2f45c1aac14531d8bd3269d7d2c9ffc342798dcb"
    dataSchema-contract: "0xb426ac2d4436c051ae71a149621c76620733ede3"
  fileConfig:
    file-dir: "tmp"
swagger:
  enabled: true
```

需要自己配置的属性包括：

| 配置项 | <div style="width:150px">说明</div> | 示例 |
| -- |-- | --------|
| spring.datasource.url | 数据库连接url |jdbc:mysql://localhost:3306/databrain?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true |
| spring.datasource.username | 用户名 | root |
| spring.datasource.password | 密码 | 123456 |
| system.admin-account  | 系统运营方账户  | admin |
| system.admin-password | 系统运营方密码 | admin |
| system.admin-private-key  | 系统运营方私钥 |11afa82f974469792aa0172931b813d4fc7dd9177f3211779efc5f955d5e480f|

 
### 启动

```
bash start.sh
```
这条指令会编译代码成springboot jar包，并执行。

## 启动前端
### 克隆代码
```
git clone https://github.com/WeBankBlockchain/Data-Brain-Front.git
cd Data-Brain-Front
git checkout origin/dev
```

### 修改配置
创建.env（可以参考.env.example）中，将REACT_APP_SERVER_API修改为实际的后端api地址，例如：
```
REACT_APP_SERVER_API=http://localhost:10880/api/
```


更新依赖：
```
yarn
```

启动：
```
yarn start
```

然后用户可以访问http://localhost:3000进行体验

## 使用流程

为了快速体验流程，用户可以：
- 申请注册两个“见证方账户”
- 登陆可信第三方账户，在管理台进行机构审核。可信第三方的默认用户名为admin，密码123456.
- 申请注册一个“机构账户”
- 登陆可信第三方账户，在管理台审核该机构
- 以机构账户登陆，在管理台创建一个数据业务
- 依次登陆2个见证方账户，执行审批
- 审批通过后，以机构账户登陆，在管理台创建数据目录
- 依次登陆2个见证方账户，执行审批
- 审批通过后，在首页就可以浏览到。
