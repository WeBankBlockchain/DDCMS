# DDCMS 后端容器化解决方案：实现优雅的环境隔离，提升开发者体验

> 作者： 张宇豪
>
> 学校： 深圳职业技术大学



浅聊一下，最近在参与DDCMS的组件开发和测试，碰到的一些问题，这个项目涉及了DDCMS-Contract和DDCMS-Service以及DDCMS-Front。合约、后端、前端。合约整理完了整体的业务和设计，当前在弄后端的时候，每一次部署稍微麻烦了一点，所以就干脆直接使用Docker隔离当前的开发环境，前端目前还没有弄，打算使用Github Action的CI功能实现自动化部署项目也是容器，方便更新前端代码的同时可以直接测试部署。好了，我们还是要先解决怎么把DDCMS-Service这个后端项目容器化，为了解决这个问题那我们就开始手撸Dockerfile之旅吧！



当前项目的仓库地址在我个人的仓库中：

- 我删除了jdk8对应的离线包（可以自己修改）
- 还有console控制台的离线包

> > > https://github.com/CN-ZHANGYH/DDCMS-Docker.git

## 1、隔离环境的优势？

Docker 的隔离环境为应用程序提供了多个重要的优势，使得它成为容器化和微服务架构的流行选择之一。以下是 Docker 隔离环境的主要优势：

1. **隔离性**：Docker 使用 Linux 内核的容器技术来隔离应用程序和它们的依赖项。每个容器都有自己的文件系统、进程空间、网络堆栈和用户空间。这种隔离性使得容器可以互相独立运行，避免了互相干扰的问题。这也意味着一个容器的崩溃或故障不会影响其他容器。
2. **轻量级**：与虚拟机相比，Docker 容器更加轻量级。容器共享主机操作系统的内核，因此它们不需要独立的操作系统镜像，这降低了资源消耗和启动时间。这使得容器更适合部署和扩展。
3. **可移植性**：Docker 容器可以在不同的环境中运行，包括开发、测试和生产环境，而不会出现依赖问题。容器包含了应用程序及其所有依赖项，确保了应用程序在不同环境中的一致性。
4. **版本控制**：Docker 容器使用镜像来定义应用程序的环境。这些镜像可以版本控制，因此您可以轻松地管理和更新容器的环境，确保应用程序的可重复性和稳定性。
5. **快速部署和扩展**：Docker 容器可以快速启动和停止，使得应用程序的部署和扩展变得更加灵活和高效。您可以根据需求快速创建新容器副本，以应对流量的变化。
6. **资源管理**：Docker 允许您限制容器的资源使用，例如 CPU 和内存，以确保容器之间的公平共享和资源分配。这有助于防止一个容器消耗所有可用资源。
7. **生态系统**：Docker 生态系统拥有丰富的容器仓库（如 Docker Hub）和工具，可以加速容器的开发、部署和管理。您可以轻松地找到和共享容器镜像，以及使用工具来自动化各种操作。
8. **安全性**：Docker 提供了多层安全机制，包括命名空间隔离、控制组、容器镜像签名等，以增强容器的安全性。此外，容器可以隔离应用程序和主机系统，减少了潜在的攻击面。

> 根据如下的图不难看出来，我把FISCO和MYSQL以及DDCMS装进了三个不一样的箱子里，做到了环境的隔离。用过docker和docker-compose的小伙伴门都知道，任何项目越是轻量化越是方便，本期我将会手搓Dockerfile实现正常业务上线的容器化开发部署。

![image-20230903143713325](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309031438171.png)

## 2、项目环境要求

- 操作系统我默认是使用的是`Ubuntu18.04`，安装的`docker 20.10 + docker-compose`工具。
- 这里给出了网络模式，因为`FISCO BCOS`节点使用network网卡模式比较麻烦，所以直接使用host网络模式，其他的都是走`ddcms-service_dccms—network`网络。

| 操作系统    | 环境依赖                      | 网络模式     |
| ----------- | ----------------------------- | ------------ |
| Ubuntu18.04 | Docker 20.10 + Docker-compose | network+host |
| Ubuntu20.04 | Docker 20.10 + Docker-compose | network+host |
| Ubuntu22.04 | Docker 20.10 + Docker-compose | network+host |



## 3、项目的目录结构

当前是我的工作目录，该`docker-compose`文件是部署`ddcms-server`和`ddcms-mysql`的。

```shell
root@node1:~/ddcms-service# ll
total 24
drwxr-xr-x  5 root root 4096 Sep  3 14:24 ./
drwx------ 14 root root 4096 Sep  3 14:36 ../
drwxr-xr-x  3 root root 4096 Sep  3 14:36 ddcms/
-rw-r--r--  1 root root 1274 Sep  3 14:24 docker-compose.yaml
drwxr-xr-x  5 root root 4096 Sep  3 14:54 fisco/
drwxr-xr-x  2 root root 4096 Sep  3 02:36 mysql/
```

使用`tree -L 3`查看一下当前的目录结构。

> 经常有容器开发的应该比较好理解：
>
> - 一个目录对应一个Dockerfile，构建一个镜像
> - docker-compose部署的时候直接内部构建镜像，再进行部署操作
> - 这是我们构建镜像需要准备的东西

```shell
root@node1:~/ddcms-service# tree -L 3
.
├── ddcms								# DDCMS-Service后端镜像构建目录
│   ├── DDCMS-Service				   # 未编译的项目 直接放到镜像中去编译
│   │   ├── LICENSE
│   │   ├── README.md
│   │   ├── build.gradle
│   │   ├── conf
│   │   ├── gradle
│   │   ├── gradlew
│   │   ├── gradlew.bat
│   │   ├── settings.gradle
│   │   └── src
│   ├── Dockerfile
│   ├── ca.crt							# 这里的节点连接证书不用多说，直接控制台拷贝到当前目录
│   ├── jdk-8u281-linux-x64.tar.gz		# 可以下载当前版本的jdk版本 我默认使用这个
│   ├── sdk.crt
│   └── sdk.key
├── docker-compose.yaml
├── fisco
│   ├── build_chain.sh
│   ├── console
│   │   ├── account
│   │   ├── accounts
│   │   ├── apps
│   │   ├── conf
│   │   ├── console.sh
│   │   ├── contract2java.sh
│   │   ├── contracts
│   │   ├── deploylog.txt
│   │   ├── get_account.sh
│   │   ├── get_gm_account.sh
│   │   ├── lib
│   │   ├── log
│   │   └── start.sh
│   ├── console.tar.gz					# 控制台压缩包
│   ├── contracts						# DDCMS-Contract拷贝的合约
│   │   ├── AccountContract.sol
│   │   ├── DataSchemaContract.sol
│   │   ├── ProductContract.sol
│   │   └── libs
│   ├── docker-compose.yaml				# docker-compose部署FISCO BCOS文件
│   └── nodes
│       ├── 127.0.0.1
│       └── ca
└── mysql								# MySQL镜像构建目录
    ├── Dockerfile
    ├── db-script.sql					# DDCMS-Service项目中拷贝过来
    └── mysql_init.sh
```

如下是项目最后构建的docker镜像。

```shell
root@node1:~/ddcms-service/ddcms# docker images
REPOSITORY                                                 TAG                            IMAGE ID       CREATED          SIZE
ddcms-service_ddcms-server                                 latest                         4a83d50f65ab   31 minutes ago   1.45GB
ddcms-service_ddcms-mysql                                  latest                         1d08b30f2a32   14 hours ago     443MB
fiscoorg/fiscobcos                                         v3.4.0                         0662d5e74dfb   2 months ago     191MB
```



## 4、Dockerfile构建MySQL镜像

### Mysql执行脚本

这个脚本是一个 Bash 脚本，主要用于初始化 MySQL 数据库的一些操作，包括创建数据库、创建用户、授权用户权限以及导入一个 SQL 脚本。

```sh
root@node1:~/ddcms-service/mysql# cat mysql_init.sh
#!/bin/bash

mysql -uroot -p$MYSQL_ROOT_PASSWORD <<EOF
CREATE DATABASE IF NOT EXISTS $DDCMS_USER;
CREATE USER '$DDCMS_USER'@'%' IDENTIFIED BY '$DDCMS_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO '$DDCMS_USER'@'%';
USE ddcms;source /mysql/db-script.sql;
FLUSH PRIVILEGES;
EOF
```

### MySQL的Dockerfile

这里我们基于默认的 MySQL 8.0.16 镜像，并添加了一些自定义操作，以便在容器启动时初始化数据库。

1. 将本地主机上的 `mysql_init.sh` 文件复制到容器内的 `/docker-entrypoint-initdb.d` 目录中。这是 MySQL 镜像约定的目录，容器启动时会自动执行该目录中的脚本。
2. 因为在容器启动时，`/docker-entrypoint-initdb.d` 目录下的脚本需要具有可执行权限，以便执行初始化操作。

```dockerfile
# 使用默认镜像mysql:8.0.16
FROM mysql:8.0.16

# 创建存放sql的目录
RUN mkdir -p /mysql
COPY db-script.sql /mysql
COPY mysql_init.sh /docker-entrypoint-initdb.d

# 执行创建用户、添加权限、创建数据库、导入数据库操作
RUN chmod a+x /docker-entrypoint-initdb.d/mysql_init.sh
```



## 5、Dockerfile构建DDCMS-Service镜像

### DDCMS后端的Dockerfile

基于CentOS 7.9.2009基础镜像，并包含了一系列操作来配置和部署一个Java应用程序（DDCMS-Service）。

- 这里构建该镜像手动添加了JDK8的Java环境
- 将DDCMS-Service的后端项目添加到/opt下
- 自动修改配置文件
- 镜像启动的时候执行启动Java项目

```dockerfile
# 使用 CentOS 7.9 作为基础镜像
FROM centos:centos7.9.2009

# 构建镜像时使用的变量
ARG DB_USERNAME=""
ARG DB_PASSWORD=""
ARG DB_NAME=""
ARG SECRET=""
ARG USER_NAME=""
ARG USER_PASSWORD=""
ARG PRIVATE_KEY=""
ARG ACCOUNT_ADDRESS=""
ARG PRODUCT_ADDRESS=""
ARG DATA_SCHEMA_ADDRESS=""
ARG NODE0=""
ARG NODE1=""

# 复制 JDK 8 的离线压缩包到容器中
ADD jdk-8u281-linux-x64.tar.gz /usr/local/
ADD DDCMS-Service/ /opt/DDCMS-Service/
# 解压 JDK 压缩包并配置 Java 环境
RUN mv /usr/local/jdk1.8.0_281 /usr/local/java
ENV JAVA_HOME=/usr/local/java
ENV PATH=$PATH:$JAVA_HOME/bin

# 清理临时文件
RUN rm -f /tmp/jdk-8u281-linux-x64.tar.gz

# 配置DDCMS-Service
WORKDIR /opt/DDCMS-Service/
RUN ./gradlew bootJar
RUN cd dist/conf/ && cp config-example.toml config.toml && \
         cp application-example.yml application.yml
RUN sed -i "s/username: test/username: $DB_USERNAME/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/dbName/$DB_NAME/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/127.0.0.1/ddcms-mysql/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/admin-password: ''/bak/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/password:/password: $DB_PASSWORD/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/secret: \"\"/secret: '$SECRET'/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/admin-private-key: ''/admin-private-key: \"$PRIVATE_KEY\"/g" /opt/DDCMS-Service/dist/conf/application.yml  &&  \
        sed -i "s/account-contract: \"\"/account-contract: \"$ACCOUNT_ADDRESS\" /g" /opt/DDCMS-Service/dist/conf/application.yml  &&  \
        sed -i "s/product-contract: \"\"/product-contract: \"$PRODUCT_ADDRESS\"/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/dataSchema-contract: \"\"/dataSchema-contract: \"$DATA_SCHEMA_ADDRESS\"/g" /opt/DDCMS-Service/dist/conf/application.yml  && \
        sed -i "s/bak/admin-password: '$USER_PASSWORD'/g" /opt/DDCMS-Service/dist/conf/application.yml && \
        sed -i "s/127.0.0.1:20200/$NODE0:20200/g" /opt/DDCMS-Service/dist/conf/config.toml && \
        sed -i "s/127.0.0.1:20201/$NODE1:20201/g" /opt/DDCMS-Service/dist/conf/config.toml
ADD ca.crt dist/conf/ca.crt
ADD sdk.crt dist/conf/sdk.crt
ADD sdk.key dist/conf/sdk.key

# 暴露10880端口
EXPOSE 10880
# 切换当前的工作目录
WORKDIR /opt/DDCMS-Service/dist
# 启动Java项目
CMD ["java", "-jar", "ddcms_0.0.1-SNAPSHOT.jar", "--spring.config.name=application", "--spring.config.location=classpath:/,file:/opt/DDCMS-Service/dist/conf/"]
```



## 6、容器化FISCO BCOS节点

### 记录账户合约地址

> 可以使用如下的方式启动docker的fisco，也可以直接使用如下命令：
>
> cd fisco && docker-compose up -d

- 记录账户私钥
  - `0x5a0f8cfa0593fc9f0437d6f9988648cf56ddd140f815e30448054d5ef6717a0b`
- 记录三个合约的地址分别是如下：
  - `0x37a44585bf1e9618fdb4c62c4c96189a07dd4b48`
  - `0x31ed5233b81c79d5adddeef991f531a9bbc2ad01`
  - `0x6546c3571f17858ea45575e7c6457dad03e53dbb`

```shell
root@node1:~/ddcms-service/fisco# bash nodes/127.0.0.1/start_all.sh
try to start node0
try to start node1
291482e57e7420cf4a961ef467dad8b6815e6f84abee417307137d5092131f79
a68f48f736bac5978f5de9085c9e88a074e9a0436e1aedc441ae16600d999c78
 node1 start successfully pid=a68f48f736ba
 node0 start successfully pid=291482e57e74
root@node1:~/ddcms-service/fisco# bash console/get_account.sh
[INFO] Account privateHex: 0x5a0f8cfa0593fc9f0437d6f9988648cf56ddd140f815e30448054d5ef6717a0b
[INFO] Account publicHex : 0x20e2b4279fe9301f5c909f7596e973c5229304f95ae03b85eb95cf2659150ee3ed027b2694df39924bd867b822d61ccc119eae96517214f71024f85f8a7a6855
[INFO] Account Address   : 0xcf6288115fbcd235ed5b09ca9c5ee90bbc108de3
[INFO] Private Key (pem) : accounts/0xcf6288115fbcd235ed5b09ca9c5ee90bbc108de3.pem
[INFO] Public  Key (pem) : accounts/0xcf6288115fbcd235ed5b09ca9c5ee90bbc108de3.pem.pub
root@node1:~/ddcms-service/fisco# bash console/start.sh group0 -pem accounts/0xcf6288115fbcd235ed5b09ca9c5ee90bbc108de3.pem
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
[group0]: /apps> deploy AccountContract
transaction hash: 0xc45abc91bb8a34a10fd01a82371e08f93ba837d6aaf374766b7a410a804b49bc
contract address: 0x37a44585bf1e9618fdb4c62c4c96189a07dd4b48
currentAccount: 0xc0b9ea9f9f8465ab31076cccbc85fd55778dca99

[group0]: /apps> deploy ProductContract 0x37a44585bf1e9618fdb4c62c4c96189a07dd4b48
transaction hash: 0x430d85959d75efe90ae36220e0e455f9be4c087b54612348db0a4282d72e11c5
contract address: 0x31ed5233b81c79d5adddeef991f531a9bbc2ad01
currentAccount: 0xc0b9ea9f9f8465ab31076cccbc85fd55778dca99

[group0]: /apps> deploy DataSchemaContract 0x37a44585bf1e9618fdb4c62c4c96189a07dd4b48 0x31ed5233b81c79d5adddeef991f531a9bbc2ad01
transaction hash: 0x82912aee126ec0c6cb339e43c4813bd18a4751331a985e43cd8e77570b184b5e
contract address: 0x6546c3571f17858ea45575e7c6457dad03e53dbb
currentAccount: 0xc0b9ea9f9f8465ab31076cccbc85fd55778dca99
```



## 7、Docker-compose文件

> 只需要配置`ddcms-mysql`和`ddcms-server`的**environment**以及**arg**部分的参数即可。

### 定义服务

> ####  `ddcms-mysql` 服务

- `build`：指定了如何构建`ddcms-mysql`容器，它将使用`./mysql`目录下的Docker镜像上下文来构建容器。
- `ports`：将容器内部的MySQL服务端口（3306）映射到主机的3306端口，以便从外部访问MySQL数据库。
- `volumes`：指定了两个数据卷的挂载点，分别用于MySQL配置文件和数据文件。这样可以保持数据的持久性。
- `environment`：设置了容器内部的环境变量，包括MySQL的root用户密码、DDCMS用户及其密码。
- `container_name`：为容器指定了一个名称，即`ddcms-mysql`。
- `networks`：将容器连接到名为`ddcms-network`的自定义网络。

> #### **`ddcms-server` 服务**

- `build`：指定了如何构建`ddcms-server`容器，它将使用`./ddcms`目录下的Docker镜像上下文，并传递一些构建参数。
- `args`：为构建参数传递了一些值，如`NODE0`、`NODE1`、`DB_USERNAME`、`DB_PASSWORD`等。
- `container_name`：为容器指定了一个名称，即`ddcms-server`。
- `ports`：将容器内部的10880端口映射到主机的10880端口，以便从外部访问`ddcms-server`服务。
- `depends_on`：指定了`ddcms-server`服务依赖于`ddcms-mysql`服务，确保在启动`ddcms-server`之前，`ddcms-mysql`服务已经启动。
- `networks`：将容器连接到名为`ddcms-network`的自定义网络。

```yaml
version: '3'
services:
  ddcms-mysql:
    build:
      context: ./mysql
    ports:
      - "3306:3306"
    volumes:
      - /usr/local/docker/mysql/conf:/etc/mysql/conf.d
      - /usr/local/docker/mysql/data:/var/lib/mysql
    environment:
      # mysql的root用户密码
      MYSQL_ROOT_PASSWORD: Aa123123.
      # ddcms用户
      DDCMS_USER: ddcms
      # ddcms用户密码
      DDCMS_PASSWORD: ddcms
    container_name: ddcms-mysql
    networks:
      - ddcms-network
  ddcms-server:
    build:
      context: ./ddcms
      args:
      	# node0的节点IP
        - NODE0=10.3.108.51
        # node1的节点IP
        - NODE1=10.3.108.51	 
        # 数据库用户名
        - DB_USERNAME=ddcms  
        # 数据库密码
        - DB_PASSWORD=ddcms  
        # 数据库名称
        - DB_NAME=ddcms      
        # secret参数 默认即可
        - SECRET=Ok2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQOk2Q0AZiRTT1N6CJ3q8ZvQ  		
        # 登录用户名 默认即可
        - USER_NAME=admin		
        # 登录密码
        - USER_PASSWORD=admin	
        # 部署合约的私钥
        - PRIVATE_KEY=0x5a0f8cfa0593fc9f0437d6f9988648cf56ddd140f815e30448054d5ef6717a0b	
        # AccountContract合约地址
        - ACCOUNT_ADDRESS=0x37a44585bf1e9618fdb4c62c4c96189a07dd4b48	
        #ProductContract合约地址
        - PRODUCT_ADDRESS=0x31ed5233b81c79d5adddeef991f531a9bbc2ad01  
        # DataSchemaContract合约地址
        - DATA_SCHEMA_ADDRESS=0x6546c3571f17858ea45575e7c6457dad03e53dbb 
    container_name: ddcms-server
    ports:
      - "10880:10880" # Adjust the port as needed
    depends_on:
      - ddcms-mysql
    networks:
      - ddcms-network
networks:
  ddcms-network:
```



### 项目启动

在项目的主目录下，执行`docker-compose up -d`。

```shell
root@node1:~/ddcms-service# docker-compose up -d
[+] Running 2/2
 ⠿ Container ddcms-mysql   Running 0.0s
 ⠿ Container ddcms-server  Running 0.0s
root@node1:~/ddcms-service# docker-compose ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
ddcms-mysql         "docker-entrypoint.s…"   ddcms-mysql         running             0.0.0.0:3306->3306/tcp
ddcms-server        "java -jar ddcms_0.0…"   ddcms-server        running             0.0.0.0:10880->10880/tcp
```

查看当前的日志：

![image-20230903152747271](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309031527933.png)