masb.sh

Alpine Linux 下的 sing-box 一键部署脚本（Reality / TLS / Hysteria2）

masb.sh 是一个面向 Alpine Linux（musl） 的 sing-box 服务端一键安装与配置脚本，目标是：

一次执行，直接可用，不输出任何“看起来能用但实际连不上”的配置。

✨ 功能概览
支持的入站协议（3 个）
协议	用途	特点
VLESS + Reality	主用	抗封锁、无需域名
VLESS + TLS	备用	兼容性最好
Hysteria2	高速	UDP、低延迟
🔐 TLS 证书模式（交互选择）

运行时会提示选择：

1) 有域名 → Let's Encrypt（HTTP-01，需开放 80）
2) 无域名 → 自签证书

模式对比
项目	Let's Encrypt	自签
是否需要域名	✅	❌
是否需要开放 80	✅	❌
客户端 insecure	0	1
分享链接地址	域名（TLS/HY2）	IP
🌐 分享链接策略（重要）

脚本刻意区分 Reality 与 TLS/HY2 的使用方式：

协议	分享链接使用
VLESS + Reality	始终使用 IP
VLESS + TLS	LE 模式用域名，否则用 IP
Hysteria2	LE 模式用域名，否则用 IP

这是目前最稳妥、最不容易出问题的组合方式。

🚀 使用方法
运行前准备（Alpine 必看）

Alpine 最小化系统默认不包含 bash 和 curl：

apk update
apk add --no-cache bash curl

一键运行

```
bash <(curl -Ls https://raw.githubusercontent.com/hooghub/Alpine01/main/Encrypt.sh)
```
```
bash <(wget -qO- https://raw.githubusercontent.com/hooghub/Alpine01/main/Encrypt.sh)
```

🧭 部署流程说明
1️⃣ 公网出口检测（强制）

自动检测 IPv4 / IPv6 出站

至少有一个可用才会继续

2️⃣ 端口选择

以下端口可手动输入或回车随机：

VLESS Reality

VLESS TLS

Hysteria2

3️⃣ Reality 伪装站点

可手动输入

或从内置池随机选择，例如：

- www.apple.com
- www.google.com
- www.gstatic.com
- www.bing.com
- www.wikipedia.org
- www.netflix.com
- www.microsoft.com
- www.yahoo.com


Reality 的 sni 与 handshake.server 强制一致

🔑 Reality Keypair（关键）

每次部署 重新生成

落盘保存，作为唯一事实来源：

/etc/sing-box/reality_private_key.txt
/etc/sing-box/reality_public_key.txt


启动前会做一致性校验
👉 若不匹配，直接拒绝输出分享链接

🔐 Let's Encrypt（HTTP-01）逻辑说明

当选择 1（LE） 时，脚本会：

校验域名解析

优先校验 A 记录是否指向本机公网 IPv4

公网可达性预检（best-effort）

临时在 :80 启 httpd

通过公网代理访问

失败仅警告，不强制中断

检查是否已有证书

SAN 包含域名

剩余有效期 ≥ 30 天

key 与 cert 匹配
👉 满足则直接复用

否则才调用 acme.sh 签发

不需要邮箱
续期后会自动重启 sing-box

📂 文件结构
/etc/sing-box/
├── config.json
├── reality_private_key.txt
├── reality_public_key.txt
├── v2rayn_links.txt
└── tls/
    ├── fullchain.pem
    └── key.pem

/var/log/sing-box/
├── sing-box.log
└── sing-box.err

▶️ 服务管理（OpenRC）
rc-service sing-box start
rc-service sing-box stop
rc-service sing-box restart


开机启动：

rc-update add sing-box default

📜 日志查看
tail -f /var/log/sing-box/sing-box.log

⚠️ 常见说明 / 坑位
IPv6 说明

部分 VPS 提供 仅出站 IPv6（NAT/Proxy）

脚本检测的是“能否出站”

入站是否可用以 IPv4 为准

Reality 客户端

强烈建议使用 Xray-core

v2rayN 中选择 Xray-core

🧠 设计理念

宁可部署失败，也不输出一个一定连不上的链接。

这个脚本的目标不是“最少步骤”，而是：

可验证

可复现

可直接导入使用

📌 适合谁使用？

Alpine Linux 用户

Reality / sing-box 使用者

不想反复排错 TLS / Reality / LE 的人
