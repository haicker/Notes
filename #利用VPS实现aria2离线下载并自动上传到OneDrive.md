# 利用VPS实现aria2离线下载并自动上传到OneDrive

本教程基于CentOS 7 64位



## 1.环境准备

#### 更新软件、设置时区

``` 
yum update -y

yum install -y curl vim wget unzip git nano

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```


#### 安装宝塔面板

##### CentOS原版

``` 
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

输入 y，回车开始安装，安装完成后记住面板登录信息。

##### 宝塔7.4.5专业版

Centos安装命令：

```
yum install -y wget && wget -O install.sh http://download.ddinin.com/install/install_6.0.sh && sh install.sh
```
Ubuntu/Deepin安装命令：

```
wget -O install.sh http://download.ddinin.com/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

Debian安装命令：

```
wget -O install.sh http://download.ddinin.com/install/install-ubuntu_6.0.sh && bash install.sh
```

#### 在宝塔面板中安装Nginx（此步可跳过）



## 2.安装Aria2

#### 运行一键脚本

```
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/aria2.sh && chmod +x aria2.sh && bash aria2.sh
```



#### 授权认证

授权地址→[【国际版、个人版(家庭版)】](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=78d4dc35-7e46-42c6-9023-2d39314433a5&response_type=code&redirect_uri=http://localhost/onedrive-login&response_mode=query&scope=offline_access%20User.Read%20Files.ReadWrite.All)、[【中国版(世纪互联)】](https://login.chinacloudapi.cn/common/oauth2/v2.0/authorize?client_id=dfe36e60-6133-48cf-869f-4d15b8354769&response_type=code&redirect_uri=http://localhost/onedrive-login&response_mode=query&scope=offline_access%20User.Read%20Files.ReadWrite.All)。

授权后会跳转到一个打不开的链接，在浏览器地址栏复制好整个链接地址，包括http://localhost。



## 3.安装OneDriveUploader

#### 一键安装脚本

```
wget https://raw.githubusercontent.com/MoeClub/OneList/master/OneDriveUploader/amd64/linux/OneDriveUploader -P /usr/local/bin/
```

给予权限

```
chmod +x /usr/local/bin/OneDriveUploader
```



#### 初始化配置

```
#国际版，将url换成你上面复制的授权地址，包括http://loaclhost。
OneDriveUploader -a "url"

#个人版(家庭版)，将url换成你上面复制的授权地址，包括http://loaclhost。
OneDriveUploader -ms -a "url"

#中国版(世纪互联)，将url换成你上面复制的授权地址，包括http://loaclhost。
OneDriveUploader -cn -a "url"
```

返回 Init config file: /root/auth.json 信息，则初始化成功。



## 4.配置Aria2自动上传



打开宝塔面板，在 /root文件夹中创建一个空白文件，文件名 `rcloneupload.sh`

在该文件中复制以下代码

```
#!/bin/bash

GID="$1";
FileNum="$2";
File="$3";
MaxSize="15728640";
Thread="3";  #默认3线程，自行修改，服务器配置不好的话，不建议太多
Block="20";  #默认分块20m，自行修改
RemoteDIR="";  #上传到Onedrive的路径，默认为根目录，如果要上传到MOERATS目录，""里面请填成MOERATS
LocalDIR="/www/download/";  #Aria2下载目录，记得最后面加上/
Uploader="/usr/local/bin/OneDriveUploader";  #上传的程序完整路径，默认为本文安装的目录
Config="/root/auth.json";  #初始化生成的配置auth.json绝对路径，参考第3步骤生成的路径


if [[ -z $(echo "$FileNum" |grep -o '[0-9]*' |head -n1) ]]; then FileNum='0'; fi
if [[ "$FileNum" -le '0' ]]; then exit 0; fi
if [[ "$#" != '3' ]]; then exit 0; fi

function LoadFile(){
  if [[ ! -e "${Uploader}" ]]; then return; fi
  IFS_BAK=$IFS
  IFS=$'\n'
  tmpFile="$(echo "${File/#$LocalDIR}" |cut -f1 -d'/')"
  FileLoad="${LocalDIR}${tmpFile}"
  if [[ ! -e "${FileLoad}" ]]; then return; fi
  ItemSize=$(du -s "${FileLoad}" |cut -f1 |grep -o '[0-9]*' |head -n1)
  if [[ -z "$ItemSize" ]]; then return; fi
  if [[ "$ItemSize" -ge "$MaxSize" ]]; then
    echo -ne "\033[33m${FileLoad} \033[0mtoo large to spik.\n";
    return;
  fi
  ${Uploader} -c "${Config}" -t "${Thread}" -b "${Block}" -s "${FileLoad}" -r "${RemoteDIR}" -skip
  if [[ $? == '0' ]]; then
    rm -rf "${FileLoad}";
  fi
  IFS=$IFS_BAK
}
LoadFile;
```

在以上代码中修改OneDrive上传路径和Aria2下载目录，修改完成后保存退出。



授权

```
chmod +x rcloneupload.sh
```



在/root/.aria2/ 中找到  `aria2.conf` ，在其最后添加一行

```
on-download-complete=/root/rcloneupload.sh
```



安装`dos2unix`并转换格式

```
yum install dos2unix -y
dos2unix /root/rcloneupload.sh
```

