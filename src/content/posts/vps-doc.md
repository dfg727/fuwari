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

更新系统:

```bash
bash <(curl -Ls ssh_tool.eooce.com)
```

- **aaPanel Internal Address:** `https://{IP}:{Port}/0e82fc5c`
- **Username:** 随机字符串
- **Password:** `*****`

## 宝塔海外版 (aaPanel)

1. **一键安装脚本**
   ```bash
   wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh forum
   ```

2. **系统放行端口**
   ```bash
   ufw allow 端口号
   # 示例：放行多个端口
   ufw allow 9898,9899/tcp
   ```

3. **安全组规则 (VPS)**
   添加规则：`出站 / TCP / 端口号`

4. **初始化配置**
   根据步骤 1 安装完成后显示的路径访问宝塔面板，安装推荐软件（Nginx, Docker 等）。

## 3x-ui Docker 安装

> **注意**：最后需要关闭 Cloudflare 的代理状态（关闭橙色云），让域名直解析到服务器真实 IP，否则代理可能无法转发。

1. **Cloudflare 设置**
   - **DNS**: 添加 A 记录
     - 类型: `A`
     - 名称: `3x-ui`
     - 内容: `服务器IP`
     - 代理状态: **关闭** (必须关闭，否则步骤 3 生成证书会报错)
   - **SSL/TLS**: 设置为 "Flexible" (灵活) 模式 -> 保存。

2. **宝塔反向代理设置**
   - 进入 **网站** -> 点击网站名 -> **反向代理**。
   - 添加代理:
     - 域名: `3x-ui.94sub.qzz.io`
     - 地址: `https://127.0.0.1:2053` (填 HTTPS 不需要带端口号，填 HTTP 需要)
     - 保存。

3. **申请 SSL 证书**
   - 在网站设置右侧选择 **SSL** -> **Let's Encrypt** (让我们加密)。
   - 点击 **生成证书**。
   - 成功后点击 **Current Certs** 查看证书路径。
   
   **证书路径示例** (请替换为自己的域名):
   - 公钥: `/www/server/panel/vhost/cert/3x-ui.94sub.qzz.io/fullchain.pem`
   - 私钥: `/www/server/panel/vhost/cert/3x-ui.94sub.qzz.io/privkey.pem`

4. **部署 Docker 容器**

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

5. **放行端口**
   ```bash
   ufw allow 2053,2096/tcp
   ```

6. **登录 3x-ui**
   - 地址: `https://3x-ui.94sub.qzz.io` 或 `https://3x-ui.94sub.qzz.io:2053`
   - 默认账号/密码: `admin` / `admin` (请及时修改)

7. **面板设置 (可选)**
   - 修改用户名、密码。
   - 修改面板默认 URI 路径、订阅默认 URI 路径。

8. **配置证书**
   - 进入 **面板设置** -> **常规** -> **证书**。
   - 公钥路径: `/root/cert/fullchain.pem`
   - 私钥路径: `/root/cert/privkey.pem`
   - 保存并重启面板。

9. **添加入站节点**
   - **入站列表** -> **添加入站**。
   - 配置示例:
     - 协议: `vless`
     - 传输: `WebSocket`
     - 安全: `TLS`
     - ALPN: 清空
     - 证书: 填入路径或从面板设置获取
   - 保存后，点击操作栏图标 -> **导出链接** -> 复制 VLESS 链接到客户端。

10. **放行订阅端口**
    ```bash
    ufw allow 58969
    ```

## Uptime Kuma (自托管监控工具)

```bash
docker run -d \
  -p 3001:3001 \
  -v /etc/dockers/uptime-kuma/uptime-kuma-data:/app/data \
  --restart=always \
  --name=uptime-kuma \
  louislam/uptime-kuma:2
```

## EasyTier

```bash
docker-compose -f docker-compose.easytier.yml up -d
```

## 哪吒面板 (Nezha Dashboard)

官方文档: [https://nezha.wiki/guide/dashboard.html](https://nezha.wiki/guide/dashboard.html)

1. **Cloudflare 设置**
   - **DNS**: 添加 A 记录
     - 类型: `A`
     - 名称: `nz`
     - 内容: `服务器IP`
     - 代理状态: **关闭**
   - **SSL/TLS**: 设置为 "Flexible" (灵活) 模式。

2. **宝塔反向代理设置**
   - 添加代理:
     - 域名: `nz.94sub.qzz.io`
     - 地址: `https://127.0.0.1:8008`
     - 保存。

3. **申请 SSL 证书**
   - 参考上文步骤，申请 Let's Encrypt 证书。
   - **证书路径示例**:
     - 公钥: `/www/server/panel/vhost/cert/nz.94sub.qzz.io/fullchain.pem`
     - 私钥: `/www/server/panel/vhost/cert/nz.94sub.qzz.io/privkey.pem`

4. **一键安装面板**
   ```bash
   curl -L https://gitee.com/naibahq/scripts/raw/main/install.sh -o nezha.sh && chmod +x nezha.sh && sudo CN=true ./nezha.sh
   ```
   - 端口默认: `8008`
   - Agent 端口: `nz.94sub.qzz.io:8008`
   - TLS: `no` (因为我们在宝塔层做了反代和SSL)

5. **登录面板**
   - 地址: `https://nz.94sub.qzz.io`
   - 默认账号/密码: `admin` / `admin`

6. **安装 Agent (被控端)**
   - 在面板中添加服务器，复制安装命令并在被控端执行：
   ```bash
   curl -L https://raw.githubusercontent.com/nezhahq/scripts/main/agent/install.sh -o agent.sh && chmod +x agent.sh && env NZ_SERVER=nz.94sub.qzz.io:8008 NZ_TLS=false NZ_CLIENT_SECRET=YOUR_SECRET ./agent.sh
   ```

7. **维护命令**
   - 重启 Agent:
     ```bash
     sudo /opt/nezha/agent/nezha-agent service restart
     ```
   - 卸载 Agent:
     ```bash
     sudo /opt/nezha/agent/nezha-agent service uninstall
     rm -rf /opt/nezha/agent
     ```
