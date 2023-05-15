# Frp使用教程
本教程适用于有内网穿透需要的人群，即从本地机器访问互联网内部的机器。一定程度上实现较为安全的内网穿透策略，即本地机器和内网机器都使用frpc，而非直接将内网机器的端口直接暴露在公网IP上，规避不法分以端口扫描方式带来的风险。

#### 相关下载地址
[中文安装教程](https://gofrp.org/docs/setup/)|[配置示例](https://gofrp.org/docs/examples/ssh/)|[Linux下使用systemd](https://gofrp.org/docs/setup/systemd/)|[frp发行版](https://github.com/fatedier/frp/releases)|[frp源码](https://github.com/fatedier/frp)

## Frp配置流程
frp可在拥有公网IP的服务器上，进行本地机器和内网中机器的通信。
因此涉及到以下三者的配置(均需进行配置)：

- 公网服务器(需要购买公网服务器，可通过华为云、腾讯云、阿里云等，首次购买有优惠)，因公网服务器为Linux系统，因此后续以Linux系统为例
- 内网机器(受访者)，内网机器可为Linux，也可为Windows；此处以Linux为例
- 本地机器(访问者)，同内网机器，为全面展示，此处以Windows为例
- 
在[frp发行版](https://github.com/fatedier/frp/releases)中选择对应的操作系统及PC架构；
Linux下通过以下命令查看PC架构：
```
uname -a
```

#### 公网服务器
以x86_64为例，获取frp并解压
```
wget https://github.com/fatedier/frp/releases/download/v0.48.0/frp_0.48.0_linux_amd64.tar.gz
tar -zxvf frp_0.48.0_linux_amd64.tar.gz
```
需要用到**frps(Linux下)**和**frps.ini**文件，只需配置**frps.ini**文件，务必将**frps**和**frps.ini**文件放在/etc/frp/下(使用systemd情况下)

##### **frps.ini**文件内容
```bash
[common]
bind_port = 7000
# token需自己设置
token=*************
```

##### 使用systemd
这个示例将会演示在 Linux 系统下使用 systemd 控制 frps 及配置开机自启。
```bash
sudo vim /etc/systemd/system/frps.service
#保存完毕后，重新加载systemd服务配置：
sudo systemctl daemon-reload
#设置开机自启动并启动frpc服务：
sudo systemctl enable frps
sudo systemctl start frps
#如果需要查看服务的状态，可以运行以下命令：
sudo systemctl status frps
#停止服务
sudo systemctl stop frps
```

##### **frps.service**文件内容
```bash
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /etc/frp/frps -c /etc/frp/frps.ini

[Install]
WantedBy = multi-user.target
```


#### 内网机器(受访者)
需要用到**frpc(Linux下)**和**frps.ini**文件，只需配置**frps.ini**文件，务必将**frps**和**frps.ini**文件放在/etc/frp/下(使用systemd情况下)

##### **frpc.ini**文件内容
```bash
[common]
server_addr = <公网服务器IP>
server_port = 7000
# token同frps.ini中
token=*************

[secret_rdp]
type=stcp
# sk需自己设置
sk=***************
use_encryption=True
use_compression=True
local_ip = 127.0.0.1
local_port = 22
```

##### 使用systemd
这个示例将会演示在 Linux 系统下使用 systemd 控制 frps 及配置开机自启。
```bash
# frps.service内容如下
sudo vim /etc/systemd/system/frpc.service
#保存完毕后，重新加载systemd服务配置：
sudo systemctl daemon-reload
#设置开机自启动并启动frpc服务：
sudo systemctl enable frpc
sudo systemctl start frpc
#如果需要查看服务的状态，可以运行以下命令：
sudo systemctl status frpc
#停止服务
sudo systemctl stop frpc
```
##### **frpc.service**文件内容
```bash
[Unit]
# 服务名称，可自定义
Description = frp client
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /etc/frp/frpc -c /etc/frp/frpc.ini

[Install]
WantedBy = multi-user.target
```

#### 本地机器(访问者)
需要用到**frpc(Windows下)**和**frpc.ini**文件，只需配置**frpc.ini**文件，两文件均在**frpc**文件夹中
```bash
[common]
server_addr = <公网服务器IP>
server_port = 7000
# token同frps.ini中
token=*************

[secret_rdp_visitor]
type=stcp
role=visitor
# sk需自己设置，同内网机器(受访者)的frpc.ini
sk=***************
server_name=secret_rdp
use_encryption=True
use_compression=True
bind_addr=127.0.0.1
bind_port=8000
```

为方便在Windows系统中启动，创建**frpc.bat**文件，并与**frpc**文件夹在同意目录
##### **frpc.bat**内容
```bash
cd .\frpc
.\frpc.exe -c .\frpc.ini
```
完成后双击**frpc.bat**文件即可启动，注意不要关闭终端