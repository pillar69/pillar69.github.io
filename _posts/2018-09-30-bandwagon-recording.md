---
layout: default
title: Tools
---

# bandwagon 记录

## 更改密码
进入 KiwiVM 后台，让 VPS 处于开机状态（running），点击左侧菜单的 Root shell – basic，输入如下命令：`echo "root:mypassword" | chpasswd`

## SSH 登录
打开终端，输入如下命令：`ssh -p your-port root@your-ip`，然后输入密码即可连接

## 安装服务端 SS
先要确保装有 python 2.6或2.7，然后从 PIP 安装：`pip install shadowsocks`（如遇“pip command not found”，需要先[安装](https://pip.pypa.io/en/stable/installing/) pip）

## 配置服务端 SS
`vi /etc/shadowsocks.json`输入：
```json
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_port":1080,
    "password":"barfoo!",
    "timeout":600,
    "method":"aes-256-cfb"
}
```

## SS 相关命令
启动：`ssserver -c /etc/shadowsocks.json -d start`
停止：`ssserver -c /etc/shadowsocks.json -d stop`
重启：`ssserver -c /etc/shadowsocks.json -d restart`

查看是否启动成功：`ps -aux | grep ssserver`

## 设置开机启动 SS
`vim /etc/rc.local`输入：`sudo ssserver -c /etc/shadowsocks/config.json -d start`
