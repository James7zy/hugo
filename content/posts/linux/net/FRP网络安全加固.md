+++
date = '2024-11-18T13:27:19+08:00'
draft = false
title = 'FRP网络安全加固'
+++

# FRP网络安全加固

# 1 最基础架构（只有 FRP） 不安全

```
Internet
                    │
                    │
            黑客扫描端口
        (nmap / masscan)
                    │
                    ▼
             +-------------+
             |     VPS     |
             |-------------|
             | frps :7000  |
             | ssh  :22    |
             +------┬------+
                    │
                    │  FRP tunnel
                    │
                    ▼
            +-------------+
            |    Home     |
            |-------------|
            | frpc        |
            | NAS         |
            | Blog        |
            | SSH         |
            +-------------+
```

问题：

```
7000端口暴露
SSH暴露
FRP可能被爆破
```

黑客攻击路径：

```
scan → frps → token爆破 → 接管隧道
```

---

## 为什么不安全

### 黑客真实攻击路径（FRP直接暴露）

这是 **90%家庭服务器被入侵的路径**

```
                Internet
                    │
                    │
            ┌───────▼────────┐
            │  黑客扫描服务器 │
            │ masscan / nmap │
            └───────┬────────┘
                    │
            扫描公网IP端口
                    │
                    ▼
             +-------------+
             |     VPS     |
             |-------------|
             | 22  ssh     |
             | 7000 frps   |
             | 7500 panel  |
             +------┬------+
                    │
                    │
          ┌─────────▼─────────┐
          │ 发现 FRP 服务     │
          │ banner fingerprint│
          └─────────┬─────────┘
                    │
                    │
           尝试 FRP token 爆破
                    │
                    ▼
         +----------------------+
         | 伪造 frpc 客户端     |
         | hijack 隧道          |
         +-----------┬----------+
                     │
                     │
                     ▼
             +-------------+
             |   Home LAN  |
             |-------------|
             | NAS         |
             | SSH         |
             | Kubernetes  |
             | Dev Server  |
             +-------------+
                     │
                     │
                     ▼
             横向移动攻击
             (lateral move)

结果：

- NAS数据被偷
- SSH密钥泄露
- 家庭网络沦陷
```

------

### 加 WireGuard 后攻击路径

```
                Internet
                    │
                    │
             黑客扫描 VPS
                    │
                    ▼
             +-------------+
             |     VPS     |
             |-------------|
             | 51820 WG    |
             |             |
             | frps        |
             | (内网监听)  |
             +------┬------+
                    │
                    │
           尝试连接 WireGuard
                    │
                    ▼
            ❌ 无密钥拒绝连接
```

攻击到此结束。

因为：

```
FRP 不在公网
```



# 2 加 WireGuard（推荐架构）

```
				Internet
                    │
                    │
             UDP 51820
                    │
                    ▼
             +-------------+
             |     VPS     |
             |-------------|
             | WireGuard   |
             | wg0:10.0.0.1|
             |             |
             | frps        |
             | bind 10.0.0.1|
             +------┬------+
                    │
             WireGuard VPN
                    │
                    ▼
            +-------------+
            |    Home     |
            |-------------|
            | wg0:10.0.0.2|
            |             |
            | frpc        |
            | services    |
            +-------------+
```

核心思想：

**FRP 不直接暴露公网**

而是：

```
frpc → WireGuard VPN → frps
```

这样：

- 黑客 **无法直接扫描 FRP**
- 只有 **WireGuard 认证通过的设备** 才能访问
- 整体安全性提升一个量级

## frps 推荐配置

**VPS：**

```
[common]
bind_addr = 10.0.0.1
bind_port = 7000

authentication_method = token
token = very-long-random-token

dashboard_port = 0
```

注意：

```
bind_addr = 10.0.0.1
```

不是

```
0.0.0.0
```

------

## frpc 配置

**home服务器：**

```
[common]
server_addr = 10.0.0.1
server_port = 7000
token = very-long-random-token

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```



## 防火墙必须加

VPS：

```
ufw allow 51820/udp
ufw deny 7000
```

因为：

```
7000 只走 wg0
```

**注意：这里开启了防火墙后，还需要再云厂上的规则上放开对应配置的端口51820**

## WireGuard VPS部署

假设：

```
VPS IP: 1.2.3.4
VPN网段: 10.0.0.0/24
```

------

1 安装 WireGuard

```
sudo apt update
sudo apt install wireguard -y
```

------

2 生成密钥

```
wg genkey | tee privatekey | wg pubkey > publickey
```

查看：

```
cat privatekey
cat publickey
```

3 创建 WireGuard配置

```
sudo nano /etc/wireguard/wg0.conf
```

内容：

```
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = VPS_PRIVATE_KEY

PostUp = sysctl -w net.ipv4.ip_forward=1

[Peer]
PublicKey = HOME_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

------

4 启动 WireGuard

```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

检查：

```
ip a 检查
wg
```

## Home服务器部署

安装 WireGuard

```
sudo apt install wireguard -y
```

生成密钥：

```
wg genkey | tee privatekey | wg pubkey > publickey
```

------

配置：

```
sudo nano /etc/wireguard/wg0.conf
[Interface]
Address = 10.0.0.2/24
PrivateKey = HOME_PRIVATE_KEY

[Peer]
PublicKey = VPS_PUBLIC_KEY
Endpoint = VPS_IP:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

启动：

```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

## 推荐再加两个安全组件

**Fail2ban**

```
apt install fail2ban
```

防止 SSH 爆破。

---

**CrowdSec**

```
apt install crowdsec
```

共享全球黑客 IP 黑名单。

# 3 进阶安全架构.

目前不部署Clouflare有DNS的可以尝试！

```
Internet
                       │
                       │
                 Cloudflare
                       │
                       ▼
               +---------------+
               |      VPS      |
               |---------------|
               | Caddy/Nginx   |
               | Reverse Proxy |
               |               |
               | WireGuard     |
               | wg0:10.0.0.1  |
               |               |
               | frps          |
               | bind 10.0.0.1 |
               +-------┬-------+
                       │
                 WireGuard
                       │
                       ▼
               +---------------+
               |     Home      |
               |---------------|
               | wg0:10.0.0.2  |
               |               |
               | frpc          |
               |               |
               | NAS           |
               | Git           |
               | Blog          |
               | AI server     |
               +---------------+
```

安全层：

```
Cloudflare  →  WAF
WireGuard   →  VPN
FRP         →  tunnel
```

攻击者路径：

```
Internet
   │
   ├─ 被Cloudflare挡
   │
   ├─ 扫描不到FRP
   │
   └─ WireGuard没有key无法连接
```

---


下面给你一套 从 0 开始配置 Cloudflare + VPS + 家庭服务器 的完整流程。

目标是实现：

```
Internet
   │
   ▼
Cloudflare
   │
   ▼
VPS (反向代理)
   │
   ▼
WireGuard
   │
   ▼
Home Server
```

这样可以：

* 隐藏 VPS IP
* 防止 DDoS
* 自动 HTTPS
* WAF 防攻击
---

---

# 4 VPS安装反向代理

推荐用 Caddy（最简单）

安装：

```
sudo apt install -y debian-keyring debian-archive-keyring
curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/gpg.key \
| sudo gpg --dearmor -o /usr/share/keyrings/caddy.gpg
```

安装：

```
sudo apt install caddy
```

---

# 5 配置 Caddy

编辑：

```
/etc/caddy/Caddyfile
```

例子：

```
blog.example.com {

    reverse_proxy 127.0.0.1:8080

}

git.example.com {

    reverse_proxy 127.0.0.1:3000

}
```

如果你是 FRP：

```
blog.example.com {

    reverse_proxy 127.0.0.1:9000

}
```

---

启动：

```
systemctl restart caddy
```

Caddy 会自动申请：

```
Let's Encrypt HTTPS
```

---

---



