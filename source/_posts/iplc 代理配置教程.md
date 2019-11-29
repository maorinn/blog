# iplc 代理配置教程
本教程采用 shadowsocks-libev + nginx 方案，个人认为这是一个高效、安全、简便的方案。阅读本教程需要基本 Linux 服务器操作知识。

## 配置环境说明
iplc 两端服务器的操作系统为`Debian 10`，使用 root 账户操作。  
入口端为 nat 共享 ip 产品，公网 ip `1.2.3.4`，开放端口`10001`。
出口端内网 ip `10.10.10.10`。

## 出口端（通常是境外服务器）配置
出口端配置的主要工作是配置 [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)  
1. 安装 shadowsocks-libev
```bash
apt update
apt install shadowsocks-libev
```

2. 修改 shadowsocks 配置文件
```bash
nano /etc/shadowsocks-libev/config.json
```
修改如下
```json
{
    "server": "0.0.0.0",
    "mode":"tcp_and_udp",
    "server_port":8388,
    "local_port":1080,
    "password":"此处设置密码",
    "timeout":60,
    "method":"aes-128-cfb"
}
```
这里监听来自所有地址的请求，建议通过防火墙限制来源地址。  
`server_port`和`method`可以根据需要自行修改

3. 重启 shadowsocks-libev 使配置生效
```bash
systemctl restart shadowsocks-libev
```

### 入口端（通常是境内服务器）配置
入口端配置的主要工作是配置 nginx 4层代理
1. 安装 nginx
```bash
apt update
apt install nginx
```

2. 备份 nginx 配置文件
```bash
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

3. 修改 nginx 配置文件
```bash
nano /etc/nginx/nginx.conf
```

修改如下  
```conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    use epoll;
}

stream {
    server {
	listen 10001;
        proxy_pass 10.10.10.10:8388;
    }
    server {
	listen 10001 udp reuseport;
        proxy_pass 10.10.10.10:8388;
    }
}
```

4. 重启 nginx 使配置生效  
```bash
systemctl restart nginx
```

## 本地 shadowsocks 配置
服务器：`1.2.3.4`  
远程端口：`10001`  
加密方式：`aes-128-cfb`  
