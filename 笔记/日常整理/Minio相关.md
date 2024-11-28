---
comment: true
---

# 自用整理


## minio Linux升级

- [参考文章](https://blog.csdn.net/WQH_Boss/article/details/130557537?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-2-130557537-blog-128815908.235^v40^pc_relevant_anti_t3_base&spm=1001.2101.3001.4242.2&utm_relevant_index=5) 【minio离线安装教程】

## 升级准备
### 1. 确认自己使用的linux 是什么类型的  32位还是64位
``` shell
uname -m
```
根据这个下载对应的minio版本

### 2. 下载对应的minio版本
arm的版本可以从这个链接下载-[arm版本](https://dl.min.io/server/minio/release/linux-arm64/)

x86_64的版本可以从这个链接下载-[x86_64版本](https://dl.min.io/server/minio/release/linux-amd64/)

### 3. 防止出现问题 先进行备份  包含了数据  配置文件等
备份data文件的时候 推荐使用 rsync 命令 **<span style="color: red;"> （推荐）</span>**：
``` shell
rsync -avz /home/MinIo/data /backup/minio_data
```
/backup/minio_data 是备份的目标目录。 其他文件也可以按照这个备份

### 4. 一些检查操作
#### 4.1 如果不知道当前启动的minio的账号密码 可以尝试通过下列方法查看
检查正在运行的 MinIO 实例：
``` shell
ps aux | grep minio
```
示例输出：
``` bash
root      2017  0.3  0.1 1531804 187976 ?      Ssl  10月29 144:12 /home/MinIo/minio server --address 0.0.0.0:9001 --console-address 0.0.0.0:9000 /home/MinIo/data
```
查看 ps 输出中是否包含以下环境变量：
  -	MINIO_ROOT_USER：MinIO 的用户名。
  -	MINIO_ROOT_PASSWORD：MinIO 的密码。

查看进程当中的环境变量:
使用 cat /proc/**<span style="color: red;"> PID</span>**/environ 检查（需要替换 PID 为实际的进程 ID，例如上例中的 2017:
``` bash
cat /proc/2017/environ | tr '\0' '\n' | grep MINIO
```
示例输出：
```
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=yourpassword
```
### 4.2 查看启动脚本或服务配置
检查 Systemd 服务配置：
``` bash
cat /etc/systemd/system/minio.service
```
或：
``` bash
cat /lib/systemd/system/minio.service
```
在文件中查找类似：
```
Environment="MINIO_ROOT_USER=admin"
Environment="MINIO_ROOT_PASSWORD=yourpassword"
```
如果没有类似的 可以查看 是否有额外的配置文件比如
```
EnvironmentFile=/etc/default/minio
```
### 5.更新minio
#### 5.1 停止minio
``` shell
systemctl stop minio
```
#### 5.2 替换掉原来的minio
根据我下载的文件 替换掉现在的文件

#### 5.3 重新启动minio
``` shell
systemctl start minio
```
#### 5.4 查看minio状态
``` shell
systemctl status minio
```
#### 5.5 查看minio版本
``` shell
minio --version
```
由于你是通过 systemd 启动的 MinIO 服务，查看 MinIO 服务的日志通常会包含 MinIO 的版本信息。可以使用以下命令来查看：
``` bash
journalctl -u minio.service
```
在输出的日志中，通常会显示 MinIO 启动时的信息，包括版本号。


如果你存在了**多个 MinIO 二进制文件**
检查 minio 命令的路径
``` bash
which minio
# 或
command -v minio
```
如果这两个命令返回的是旧版本的路径，例如 /usr/bin/minio，那么就说明你系统中有旧版本的 MinIO，可能需要删除旧版本或更新路径。

## 其他的可以在最开始的链接当中查询到对应的信息

