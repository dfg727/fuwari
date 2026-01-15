---
title: VPS init
published: 2025-12-30
description: 'Linux服务器初始化环境'
image: '/images/vps-doc.jpg'
tags: ['Markdown', 'VPS', 'Linux', '运维']
category: 'VPS 运维'
draft: false 
---


## SSH 一键安装脚本
更新系统: `bash <(curl -Ls ssh_tool.eooce.com)`

aaPanel Internal Address: https://{IP}:{Port}/0e82fc5c
username: 随机字符串
password: *****

## 宝塔海外版
1. 一键安装脚本 `wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh forum`
2. 系统放行端口 `ufw allow 端口号` 多个:`ufw allow 9898,9899/tcp`
3. 如果是vps, 则添加安全组规则 `出站 / TCP / 端口号`
4. 根据 #1 安装完显示的路径访问宝塔, 安装推荐软件, nginx, docker


## 3x-ui docker安装 (注：最后也需要关闭cloudflare的代理状态，关闭橙色云让域名直解析到服务器真实 IP， 不然代理时转不了)
1. Cloudflare - 域名 - DNS - 添加记录 - 添加 A 记录 `类型:A / 名称:3x-ui / 内容:服务器IP / 代理状态:关闭` - 关闭代理(不然#3生成证书报错)
					 - SSL/TSL - 概述 - 配置 - 选择 "灵活" 模式 - 保存
2. 宝塔 - 网站 - tab选反向代理 
	- 添加代理 - 域名填`3x-ui.94sub.qzz.io` - 地址填`https://127.0.0.1:2053`（代理url填http之后访问域名时需要带端口号，填https不用） - 保存 
3. 配置刚刚新加的域名`3x-ui.94sub.qzz.io` 
	- 右侧选SSL - 点`让我们加密`tab - 点击生成证书 - 点`Current Certs`tab - 保存
	```
   证书将保存到:
	公钥路径：`/www/server/panel/vhost/cert/3x-ui.94sub.qzz.io/fullchain.pem`
	私钥路径：`/www/server/panel/vhost/cert/3x-ui.94sub.qzz.io/privkey.pem`
	注：标记部分需要替换成为自己的域名
	```
4. 增加docker容器
```yaml
version: "3.9"
services:
    3x-ui:
        image: ghcr.io/mhsanaei/3x-ui:latest
        container_name: 3x-ui
        restart: unless-stopped
        volumes:
        - ./db:/etc/x-ui
        - /www/server/panel/vhost/cert/3x-ui.94sub.qzz.io:/root/cert
        environment:
            XRAY_VMESS_AEAD_FORCED: "false"
        tty: true
        network_mode: host
```
5. 系统放行端口 `ufw allow 端口号` 多个:`ufw allow 2053,2096/tcp`
6. 登陆3x-ui: `https://3x-ui.94sub.qzz.io` 或者 `https://3x-ui.94sub.qzz.io:2053`, 
	默认用户名密码:`admin /*****`
7. 可选: 3x-ui - 面板设置, 修改用户名密码, 面板默认 URI 路径, 订阅默认 URI 路径
8. 3x-ui - 面板设置 - 常规 - 证书
	公钥路径：`/root/cert/fullchain.pem`
	私钥路径：`/root/cert/privkey.pem`
9. 3x-ui - 入站列表 - 添加入站
	`协议:vless / 传输:WebScoket / 安全:TLS / ALPN:清空选项 / 填入数字证书或者点从面板设置证书` - 保存
		- 站点列表新增一行 - 点菜单列icon - 导出链接 - 复制vless链接到客户端使用
10. 系统放行订阅端口 `ufw allow 58969`


## Uptime Kuma 易于使用的自托管监控工具
```bash
docker run -d \
  -p 3001:3001 \
  -v /etc/dockers/uptime-kuma/uptime-kuma-data:/app/data \
  --restart=always \
  --name=uptime-kuma \
  louislam/uptime-kuma:2
```

## easytier
`docker-compose -f docker-compose.easytier.yml up -d`

# 哪吒面板 https://nezha.wiki/guide/dashboard.html
1. Cloudflare - 域名 - DNS - 添加记录 - 添加 A 记录 `类型:A / 名称:nz / 内容:服务器IP / 代理状态:关闭` - 关闭代理(不然#3生成证书报错)
					 - SSL/TSL - 概述 - 配置 - 选择 "灵活" 模式 - 保存
2. 宝塔 - 网站 - tab选反向代理 
	- 添加代理 - 域名填`nz.94sub.qzz.io` - 地址填`https://127.0.0.1:8008`（代理url填http之后访问域名时需要带端口号，填https不用） - 保存 
3. 配置刚刚新加的域名`nz.94sub.qzz.io` 
	- 右侧选SSL - 点`让我们加密`tab - 点击生成证书 - 点`Current Certs`tab - 保存
	```
	证书将保存到:
	公钥路径：`/www/server/panel/vhost/cert/nz.94sub.qzz.io/fullchain.pem`
	私钥路径：`/www/server/panel/vhost/cert/nz.94sub.qzz.io/privkey.pem`
	注：标记部分需要替换成为自己的域名
	```
4. 一键安装: `curl -L https://gitee.com/naibahq/scripts/raw/main/install.sh -o nezha.sh && chmod +x nezha.sh && sudo CN=true ./nezha.sh`
	- 默认8008
	- agent端口 `nz.94sub.qzz.io:8008`
	- TLS: no
5. 打开url: `https://nz.94sub.qzz.io` 登录面板
	- 默认用户名密码: `admin / *****`
6. 复制安装命令到服务器执行安装agent
	`curl -L https://raw.githubusercontent.com/nezhahq/scripts/main/agent/install.sh -o agent.sh && chmod +x agent.sh && env NZ_SERVER=nz.94sub.qzz.io:8008 NZ_TLS=false NZ_CLIENT_SECRET=Y8tOGEN6IZaa7ZyhXcJr3xdaFMk97iau ./agent.sh`
7. 修改config.yml重启, 卸载agent
	`sudo /opt/nezha/agent/nezha-agent service restart`
	`sudo /opt/nezha/agent/nezha-agent service uninstall`
	` rm -rf /opt/nezha/agent`