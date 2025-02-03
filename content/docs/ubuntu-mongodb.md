+++
title = 'Ubuntu 20.04 安装 MongoDB 7.0'
date = 2024-03-16T15:40:12+08:00
+++

## 添加MongoDB 7.0存储库

更新 APT 包索引：

``` sh
sudo apt update -y
```

安装所有必需的软件包：

``` sh
sudo apt install wget curl gnupg2 software-properties-common apt-transport-https ca-certificates lsb-release vim
```

接下来，导入 MongoDB 7.0 存储库的 GPG 密钥：

``` sh
curl -fsSL https://pgp.mongodb.com/server-7.0.asc |sudo gpg  --dearmor -o /etc/apt/trusted.gpg.d/mongodb-server-7.0.gpg
```

在 Ubuntu 22.04 上添加 MongoDB 7.0 的存储库：

``` sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

## 安装MongoDB 7.0包

添加存储库后，使用以下命令更新包索引并安装 MongoDB 7.0：

``` sh
sudo apt update && sudo apt install mongodb-org -y
```

启动并启用服务：

``` sh
sudo systemctl enable --now mongod
```

验证服务是否启动：

``` sh
systemctl status mongod
```

验证安装的 MongoDB 版本：

``` sh
mongod --version
```

## 配置MongoDB 7.0

MongoDB 将其配置文件存储在 /etc/mongod.conf 中。在该文件中，可以对 MongoDB 进行所有所需的配置，包括数据库路径、日志目录等。

以下是我们可以为 MongoDB 7.0 进行的一些设置。都需要打开文件进行编辑：

``` sh
sudo vim /etc/mongod.conf
```

### 启用密码验证
要在 MongoDB 7.0 上启用密码身份验证，请取消配置中以下行的注释：

security:
  authorization: enabled

### 启用远程访问

为了允许远程访问数据库，我们需要将其设置为绑定到所有接口，而不仅仅是本地主机。为此，请进行以下修改：

```
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.
```

设置完成后，保存配置并重新启动服务：

``` sh
sudo systemctl restart mongod
```

如果启用了服务器的防火墙，请允许该服务通过。云服务器厂商通常关闭服务器内部的防火墙，并由云服务器控制台统一管理。

``` sh
sudo ufw allow 27017
```

## 使用MongoDB 7.0数据库

要连接到 MongoDB shell，请使用：`mongosh`命令。

示例输出：

``` sh
Current Mongosh Log ID:	64eb1de75716266ec1174a00
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.10.6
Using MongoDB:		7.0.0
Using Mongosh:		1.10.6

For mongosh info see: https://docs.mongodb.com/mongodb-shell/


To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

test> 
```

### 在 MongoDB 上创建用户

将通过创建一个名为 testuser 的示例用户来进行测试。我们还将为用户分配管理员角色：

```
use admin
db.createUser(
{
user: "testuser",
pwd: passwordPrompt(), // or cleartext password
roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
}
)
```

为用户设置所需的密码，然后使用`exit`命令退出。

我们将通过使用用户连接来测试一切是否正常：

``` sh
mongosh -u testuser -p --authenticationDatabase admin
```

### 创建数据库和集合
要在 MongoDB 上创建数据库，只需切换到一个不存在的数据库，如 testdb 所示：

``` sh
use testdb
```

输出：`switched to db testdb`。

现在可以在数据库中创建集合：

```
db.employeedetails.insertOne(
   {F_Name: "John",
    L_NAME: "Doe",
    ID_NO: "23245",
     AGE: "25",
     TEL: "63365467666"
   }
)
```

### 查看创建的集合

```
show collections
```