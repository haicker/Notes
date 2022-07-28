# VPS安装Transmission进行PT下载

标签（空格分隔）： PT

---

理论上VPS利用Aria2客户端进行PT下载是可行的，毕竟PT跟BT没有本质区别，只要将DHT等选项禁用就好。但是经过测试可以正常下载却迟迟没有上传量，令人费解，无奈安装Transmission客户端。

## 安装

Transmission 包含在 EPEL 拓展仓库中，如果没有安装 EPEL 源，安装前需要输入以下命令安装 EPEL 源：

```
yum -y install epel-release
yum -y update
```
安装 Transmission：

```
yum install transmission-daemon
```

## 配置

### 配置Transmission客户端
Transmission在安装完成后需要启动一次才会生成配置文件，所以先进行一次启动和停止：

```
systemctl start transmission-daemon.service
systemctl stop transmission-daemon.service
```

接着用Vi或Nano修改配置文件：

```
vi /var/lib/transmission/.config/transmission-daemon/settings.json
```
根据自己的需要，修改用户名、密码、下载路径等，需要注意的是一定要禁用DHT和IP白名单：

```
"dht-enabled": false,
"rpc-authentication-required": true,
"rpc-enabled": true,
"rpc-password": "输入你的管理密码",
"rpc-username": "管理你的用户名",
"rpc-whitelist-enabled": false,
```

### 美化Web UI

Transmission 自带的网页 UI 比较简陋，可以安装transmission-web-control进行美化：

```
wget https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control.sh --no-check-certificate
bash install-tr-control.sh
```

### 配置防火墙(CentOS 7)

如果不需要用到防火墙，可以直接禁用：

```
systemctl stop firewalld
systemctl disable firewalld
```

如果环境不允许禁用防火墙，需要放行9091端口：

```
firewall-cmd --permanent --zone=public --add-port=9091/tcp
firewall-cmd --reload
```

如果PT站显示可连接性为否，可以启用端口转发或者放行加密端口，默认端口51413（大概）。

## 运行使用

开启Transmission客户端并设置开机启动：

```
systemctl start transmission-daemon.service
systemctl enable transmission-daemon.service
```

在浏览器地址栏输入 http://域名:9091 或 http://IP地址:9091 即可进入客户端。

