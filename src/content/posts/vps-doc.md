---
title: VPS init
published: 2025-12-30
description: 'Linux服务器初始化环境'
image: '/images/vps-doc.jpg'
tags: ['Markdown', 'VPS', 'Linux', '运维']
category: 'VPS 运维'
draft: false 
---

# 一键脚本 Linux服务器运维
`bash <(curl -Ls ssh_tool.eooce.com)`
- 输入 2 更新系统
- 输入 5 安装BBR加速
- 输入 9 进入面板工具
    - 输入 2 安装宝塔国际版 (查看宝塔信息: `bt default`)
    - 进入宝塔web, 安装建站组件(nginx等), docker
- 输入 12 进入节点搭建
    - 输入 8 老王小钢炮 安装argo, 各类型节点

# 常用命令
- 放行端口: 
`ufw allow 端口号` 多个:`ufw allow 9898,9899/tcp`
- 查看端口占用的进程: 
`sudo lsof -i :9000` 或者 `sudo ss -ltnp | grep 9000`, 无结果 → 9000 未被监听