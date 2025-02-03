+++
title = 'GZCTF 搭建部署笔记（Docker）'
date = 2024-03-10T21:57:38+08:00
comments = true
+++

> Github: [GZTimeWalker/GZCTF](https://github.com/GZTimeWalker/GZCTF)
>
> 部署方式：Docker-compose（GZCTF、数据库、题目容器均在同一Docker实例中）
>
> 系统：Ubuntu20.04-Docker20（因此默认系统已经安装了Docker和Docker Compose）

## 将当前用户加入 docker 组

若出现下面的报错才需配置。

```shell
[xxxx@xxxx ~]$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

问题出在用户为访问 `/var/run/docker.sock` 的权限，只需给用户增加权限即可。命令行输入：

```shell
sudo chown root:docker /var/run/docker.sock	# 修改docker.sock权限为root:docker
sudo groupadd docker          				# 添加docker用户组 
sudo gpasswd -a $USER docker  				# 将当前用户添加至docker用户组
newgrp docker                 				# 更新docker用户组
```

## 配置文件

新建两个文件，位于同一个文件夹。这里的文件夹名称为 `GZCTF`，文件为 `appsettings.json` 和 `docker-compose.yml` 。

`appsettings.json` 文件内写入：

```json
{
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Database": "Host=db:5432;Database=gzctf;Username=postgres;Password=<String1>"
      //<String1>换成数据库密码，随机密码且长度足够
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
    //邮箱配置
  "EmailConfig": {
    "SendMailAddress": "Admin@xxx.com",		// 填入邮箱
    "UserName": "ctf_noreply",				    // 发件人名称
    "Password": "******",                 // 邮箱密码，部分服务商需要填入授权码
    "Smtp": {
      "Host": "smtp.163.com",				  // 此处为163邮箱服务器，具体自定
      "Port": 465
    }
  },
  "XorKey": "<String2>",					    // 自定XorKey
  "ContainerProvider": {
    "Type": "Docker",
    "PublicEntry": "xx.xx.xx.xx",			// 域名或IP配置，用于容器生成,域名不带http/https
    "DockerConfig": {
      "SwarmMode": false,
      "Uri": ""								        // 本地配置Docker因此此处置空
    }
  },
  "RequestLogging": false,
  "DisableRateLimit": false,
  "RegistryConfig": {
    "UserName": "",
    "Password": "",
    "ServerAddress": ""
  },
    
    //谷歌验证码配置
  "GoogleRecaptcha": {
    "VerifyAPIAddress": "https://www.recaptcha.net/recaptcha/api/siteverify",
    "Sitekey": "",
    "Secretkey": "",
    "RecaptchaThreshold": "0.5"
  }
}
```

`docker-compose.yml` 写入：

```yml
version: '3.0'
services:
  gzctf:
    image: gztime/gzctf:latest
    restart: always
    environment:
      - "GZCTF_ADMIN_PASSWORD=<String3>" # <String3>换成管理员账户密码，账号为Admin
    ports:
      - "80:8080" # 对外端口号，前为外部端口。
    networks:
      default:
    volumes:
      - "./data/files:/app/uploads"
      - "./appsettings.json:/app/appsettings.json:ro"
      - "./logs:/app/log"
      - "./data/keys:/root/.aspnet/DataProtection-Keys"
      # - "./k8sconfig.yaml:/app/k8sconfig.yaml:ro"
      - "/var/run/docker.sock:/var/run/docker.sock"
    depends_on:
      - db

  db:
    image: postgres:alpine
    restart: always
    environment:
      - "POSTGRES_PASSWORD=<String1>" # 数据库密码，务必要和appsettings.json中的配置一致
    networks:
      default:
    volumes:
      - "./data/db:/var/lib/postgresql/data"

networks:
  default:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.12.0/24

```

## 命令行

通过 Xshell 和 Xftp 连接服务器，上传 `GZCTF` 文件夹。开始部署。

```shell
cd GZCTF
docker-compose up -d
```

部署完成后，建议查看 log，看部署是否成功。主要是看 gzctf 容器是否连接上了数据库。

`docker ps` 查看容器 ID：

```shell
ubuntu@VM-16-8-ubuntu:~$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED        STATUS        PORTS                NAMES
5b9bc7a475h2   gztime/gzctf:latest   "dotnet CTFServer.dll"   17 hours ago   Up 17 hours   0.0.0.0:80->8080/tcp   gzctf_gzctf_1
f00e8850dee4   postgres:alpine       "docker-entrypoint.s…"   17 hours ago   Up 17 hours   5432/tcp             gzctf_db_1
```

看到 gzctf 的 ID 是 `5b9bc7a475h2`。使用 `docker logs 5b9bc7a475h2` 查看日志。看到连接上数据库的日志即可成功配置。

## 平台测试

登录域名或 IP（域名需要在域名服务商配置）。用 Admin 和前面设置的密码登录。（中间就自己添加比赛什么的）然后添加需要容器的题目，并测试几个 Web 容器。

![](http://picgo-1258675557.cos.accelerate.myqcloud.com/202304011351351.png)

可以测试的 Docker Hub 镜像如下。这些镜像本身部署是没有问题的。若出现错误就是平台配置问题了。服务端口均为 80。

```
vaalacat/push_f12
glzjin/hctf_2018_warmup
ctftraining/hbctf_2017_dameixian
ctftraining/qwb_2019_smarthacker
ctftraining/buuctf_2018_online_tool
ctftraining/qwb_2019_upload
ctftraining/qwb_2019_supersqli
```

需要注意的是，**部分镜像需要内存和存储较多，可能出现创建容器成功但无法打开网页的情况**。

## 容器其他错误排查

- 部署报错：
  - **不会两个文件内容改都不改吧？**
  - 检查 pip 版本，检查 Docker 版本。

- 容器无法创建：数据库是否连上？看前面的配置和使用 `docker logs xxx` 命令查看日志。

- 容器创建成功但无法打开网页：
  - 域名访问：服务商的域名配置是否正确？
  
  - 内存限制和存储限制是否太小？
  
  - 服务器的防火墙端口是否打开了？
