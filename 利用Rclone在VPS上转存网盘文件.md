# 利用Rclone在VPS上转存网盘文件

标签（空格分隔）： Rclone OneDrive GoogleDrive 


---

Rclone 可以获取 OneDrive、Google Drive、Dropbox 等国外常用网盘的授权，并在两个网盘之间（包括同一谷歌账号的两个团队盘之间）对文件进行复制、同步或移动等操作。本篇介绍如何使用 Rclone 在VPS端转存网盘文件。

首先安装并配置 Rclone（参考[前篇](/index.php/archives/1/)第三节），至少成功配置好两个网盘后，就可以在网盘之间转存文件了。详细指令和参数可以参考 Rclone 的[官方文档](https://rclone.org/docs/)。

## Rclone 简要用法

### 语法

```
rclone [指令] <源网盘:路径> <目标网盘:路径> [参数] [参数] 

示例： rclone copy onedrive:/download/ gdrive:/upload --transfers=1

```

### 指令

> * `rclone copy` -- 复制，略过已有文件

> * `rclone move` -- 移动，如果要在移动后删除空源目录，请加上 `--delete-empty-src-dirs` 参数

> * `rclone sync` -- 同步：将源目录同步到目标目录，只更改目标目录，**请谨慎使用**

> * `rclone size` -- 查看网盘文件占用大小

> * `rclone delete` -- 删除路径下的文件内容

> * `rclone purge` -- 删除路径及其所有文件内容

> * `rclone mkdir` -- 创建目录

> * `rclone rmdir` -- 删除目录

> * `rclone rmdirs` -- 删除指定灵境下的空目录。如果加上 `--leave-root` 参数，则不会删除根目录

> * `rclone check` -- 检查源和目的地址数据是否匹配

> * `rclone ls` -- 列出指定路径下的所有的文件以及文件大小和路径

> * `rclone lsd` -- 列出指定路径下的目录

> * `rclone lsf` -- 列出指定路径下的目录和文件


### 参数

> * `--cache-chunk-size SizeSuffi` -- 块的大小，默认5M，理论上是越大上传速度越快，同时占用内存也越多。如果设置得太大，可能会导致进程中断。

> * `--cache-chunk-total-size SizeSuffix` -- 块可以在本地磁盘上占用的总大小，默认10G（实测对等带宽下几乎不会占用本地磁盘空间）。
> * `--transfers=N` -- 并行文件数，默认为4 。

> * `--config string` -- 指定配置文件路径，string为配置文件路径。

> * `--ignore-errors` -- 跳过错误。比如 OneDrive 在传了某些特殊文件后会提示 "Failed to copy: failed to open source object: malwareDetected: Malware detected"，这会导致后续的传输任务被终止掉，此时就可以加上这个参数跳过错误。但需要注意 RCLONE 的退出状态码不会为0。

### 日志

Rclone 有 4 个级别的日志记录，ERROR，NOTICE，INFO 和 DEBUG。默认情况下，Rclone 将生成 ERROR 和 NOTICE 级别消息。

> * `-q` -- Rclone将仅生成 ERROR 消息。

> * `-v` -- Rclone将生成 ERROR，NOTICE 和 INFO 消息，**推荐此项**。

> * `-vv` -- Rclone 将生成 ERROR，NOTICE，INFO和 DEBUG 消息。

> * `--log-level LEVEL` -- 标志控制日志级别。

> * `--log-file=FILE` -- 此参数会将 Error，Info 和 Debug 消息以及标准错误重定向到 FILE，这里的 FILE 是你指定的日志文件路径。

### 文件过滤

- [ ]  `--exclude` -- 排除文件或目录。

- [x]  `--include` -- 包含文件或目录。

1. 过滤文件类型

> * `--exclude "*.png"` -- 排除所有 png 文件。

> * `--include "*.{avi,mp4}"` -- 只包含类型为 avi 和 mp4 文件。

> * `--delete-excluded` -- 删除被排除的文件,**谨慎使用**。需配合过滤参数，否则无效。

2. 过滤目录

目录过滤需要在目录名称后面加上 /，否则会被当做文件进行匹配。

> * `--exclude ".git/"` -- 排除所有目录下的 .git 目录。

> * `--exclude "/.git/"` -- 只排除根目录下的 .git 目录。

> * `--exclude "{Video,Software}/"` -- 排除所有目录下的 Video 和 Software 目录。

> * `--exclude "/{Video,Software}/"` -- 只排除根目录下的 Video 和 Software 目录。

> * `--include "/{Video,Software}/**"` -- 仅包含根目录下的 Video 和 Software 目录的所有内容。

3. 过滤文件大小

默认大小单位为 kBytes ，但可以使用 k 、M 、G 后缀。以下两个参数不能同时使用

> * `--min-size` -- 过滤小于指定大小的文件。例如 `--min-size 50` 表示不会传输小于 50k 的文件。

> * `--max-size` -- 过滤大于指定大小的文件。例如 `--max-size 1G` 表示不会传输大于 1G 的文件。

## 保持后台运行

当我们转存网盘文件时，如果关闭 ssh 连接，可能会使 Rclone 程序停止运行。可以通过 screen 创建一个后台窗口保证 rclone 的持续运行。

安装 screen ：
``` 
yum -y install screen 
```

创建名为rclone的后台窗口：
``` 
screen -S rclone
```
在新打开的窗口输入 rclone 相关指令，就可以持续执行了。

下一次 ssh 连接时可以进行以下操作：

```
screen -ls  #列出所有后台窗口
screen -r rclone  #进入rclone窗口
screen -X rclone quit  #关闭rclone窗口
```