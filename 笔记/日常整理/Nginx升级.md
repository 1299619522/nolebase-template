---
comment: true
---

# 用到了就进行记录，不一定适用（是从1.24升级到1.26）

[//]: # (- [参考文章]&#40;https://blog.csdn.net/WQH_Boss/article/details/130557537?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-2-130557537-blog-128815908.235^v40^pc_relevant_anti_t3_base&spm=1001.2101.3001.4242.2&utm_relevant_index=5&#41; 【minio离线安装教程】)

### 说明：这个是我在遇到修复漏洞的时候进行的升级操作，仅供参考。

## 升级准备
### 1. 确认自己需要升级的版本
我自己的原先是1.24是通过yum安装的  需要升级到1.26 原本还是想通过rpm下载升级升级，但是由于没有找到对应的rpm包，所以只能通过源码编译的方式进行安装,下面的是链接地址。

[rpm下载链接](https://nginx.org/packages/centos/7/x86_64/RPMS/)

### 2. 下载nginx
[Nginx下载地址](https://nginx.org/en/download.html)

选择对应的版本进行下载
```shell
cd /usr/local/src
wget https://nginx.org/download/nginx-1.26.2.tar.gz
tar -xzvf nginx-1.26.2.tar.gz
cd nginx-1.26.2
```


如果没有安装依赖，可以先进行安装依赖，这个根据你nginx用到的模块进行安装
``` shell
yum install -y gcc pcre-devel zlib-devel openssl-devel
```

安装依赖可能遇到的问题

可能会出现有些地址无法下载出现类似下述错误
yum 可能会尝试访问 mirrorlist.centos.org，但这个地址已经废弃，或在某些地区无法解析
```shell
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
Could not retrieve mirrorlist http://mirrorlist.centos.org/?
```
解决方法：替换为阿里云的镜像源 **<span style="color: red;"> （推荐）</span>**
```shell
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.bak

wget http://mirrors.aliyun.com/repo/Centos-7.repo -O CentOS-Base.repo

yum clean all
yum makecache
```

然后再重新尝试进行安装依赖

## 编译 Nginx

- 进入 Nginx 源码目录：
```shell
cd /usr/local/src/nginx-1.26.2
```
- 执行 ./configure 命令
```shell
./configure \
  --prefix=/usr/local/nginx-1.26.2 \
  --conf-path=/etc/nginx/nginx.conf \
  --sbin-path=/usr/local/nginx-1.26.2/sbin/nginx \
  --pid-path=/var/run/nginx.pid \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_stub_status_module \
  --with-http_gzip_static_module \
  --with-http_rewrite_module \
  --with-http_realip_module
```

- 编译安装
```shell
make
make install
```

- 检查版本是否生效
 ```shell
/usr/local/nginx-1.26.2/sbin/nginx -v  # 正确版本

# 你也可以做个软链接让系统默认路径指向新版
ln -sf /usr/local/nginx-1.26.2/sbin/nginx /usr/bin/nginx

nginx -v  # 现在也能看到新版
```

- 编译完成可以通过下述方式验证Nginx配置是否正确
```shell
/usr/local/nginx-1.26.2/sbin/nginx -t -c /etc/nginx/nginx.conf
```
如果一切顺利，配置应该会成功通过验证。



# 配置systemd服务

- 由于原本就已经通过systemd进行管理，所以可以直接进行替换，如果找不到nginx.service文件，可以通过下述命令进行查找
``` shell
systemctl status nginx
```
输出中通常会显示 Loaded 行，例如：
```shell
Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
```
这一行告诉你 nginx.service 文件的位置是：
```shell
/usr/lib/systemd/system/nginx.service
```
- 以防万一可以先备份一下
``` shell
cp /usr/lib/systemd/system/nginx.service /usr/lib/systemd/system/nginx.service.bak
```
进行修改
``` shell
vim /usr/lib/systemd/system/nginx.service
```
修改 [Service] 中的nginx路径
``` shell
ExecStart=/usr/local/nginx-1.26.2/sbin/nginx -c /etc/nginx/nginx.conf
```
修改成类似这样

- 重载 systemd 并启动
```shell
# 让 systemd 重新识别服务配置
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# 启动 nginx（用的是新版本）
sudo systemctl start nginx

# 设置开机自启（可选）
sudo systemctl enable nginx

# 查看运行状态
systemctl status nginx
```







