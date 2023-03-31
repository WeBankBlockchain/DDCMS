# Data-Brain

Data-Brain面向个人数据可携带权了，其提供一种由用户主导、安全可信、体验友好、商业可持续的解决方案，实现个人数据可携带。

# 基本概念
## 参与角色
Data-Brain的参与者包括：机构用户、可信第三方、见证方，它们围绕着数据目录的发布、管理、查阅。简单来说，一家机构需要先在DataBrain上面注册，经过可信第三方审核后，可以在Data-Brain申请发布该机构的数据目录，经见证方审核后，即可对外公示，他人可以浏览到该数据目录所提供的数据服务。机构的注册、审批，信息的发布、审核，均记录在链上，确保透明、可追溯。

### 机构用户
用户可以注册为机构，机构的注册需要提交相关材料，需要经过可信第三方的审核。

### 可信第三方
可信第三方承担了系统的部署、身份的审核，用于确保各账户身份可信。

### 见证方
用户可以选择注册为见证方，见证方用于确保业务可信性，包括对产品、数据目录进行审核，审核采用多方投票，只有赞成票过半，才算通过审核。

## 产品
产品用于对数据目录分类，每个数据目录均需要挂靠在某个产品下面。产品创建后，经见证方审核，才能生效。


## 数据目录
数据目录上面记录了该类数据的服务，包括数据类型、所属产品、数据定价、数据服务地址、数据传输协议、数据返回格式等内容。每个数据目录创建后，经见证方审核，才能生效。

# 部署方法
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

Data-Brain合约由三个部分构成：AccountContract、ProductContract、DataSchemaContract，分别管理账户、产品、数据目录。这里面的依赖关系为DataSchemaContract->ProductContract->AccountContract。 这是因为AccountContract记录了各个账户的身份，只有满足特定条件的身份才能进行产品和数据目录的创建、审核等操作；而数据目录隶属于产品，合约需要验证该关系。因此，在部署顺序上，需要先部署AccountContract，然后是ProductContract、DataSchemaContract。


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
cp [控制台目录]/conf/* ./conf
```

### 数据库配置
进入数据库，依次执行resources目录下的两个数据库脚本：
- db-scripts.sql
- insert-menu-scripts.sql


### 服务配置
在src/main/resources目录下创建application.yml，按照下面的模板进行配置，其中打了${}的需要自己配置。
```
server:
  port: ${服务端口号，例如8080} 

spring:
  mvc:
    async:
      request-timeout: 500
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
  datasource:
    url: ${JDBC连接地址}
    allowMultiQueries=true
    username: #{MySql用户名}
    password: #{MySql密码}
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

system:
  bcos-cfg: "config/config.toml"
  bcos-group-id: "group0"
  crypto-type: 0
  admin-private-key: ${见证方私钥地址}
  contractConfig:
    account-contract: ${AccountContract合约地址}
    product-contract: ${ProductContract合约地址}
    dataSchema-contract: ${DataSchemaContract合约地址}
  fileConfig:
    file-dir: "tmp"
swagger:
  enabled: true

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



```

### 启动

```
bash start.sh
```
这条指令会编译代码成springboot jar包，并执行。

## 启动前端
### 克隆代码
```
git clone https://github.com/WeBankBlockchain/Data-Brain-Server.git
cd Data-Brain-Server
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
- 登陆可信第三方账户，在管理台审核这两个机构。这个账户默认用户名为admin，密码123456.
- 申请注册一个“机构账户”
- 登陆可信第三方账户，在管理台审核这两个机构
- 以机构账户登陆，在管理台创建产品
- 依次登陆2个见证方账户，执行审批
- 审批通过后，以机构账户登陆，在管理台创建数据目录
- 依次登陆2个见证方账户，执行审批
- 审批通过后，在首页就可以浏览到。
