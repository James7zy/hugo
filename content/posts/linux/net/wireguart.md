> **Quick Answer:** Install: `apt install wireguard`. Generate keys: `wg genkey | tee private.key | wg pubkey > public.key`. Create `/etc/wireguard/wg0.conf` with your keys and network settings. Start: `wg-quick up wg0`. Enable on boot: `systemctl enable wg-quick@wg0`. Or use [SamNet-WG](https://github.com/SamNet-dev/wg-orchestrator) to automate everything with a TUI.

## Why WireGuard?

|  | WireGuard | OpenVPN | IPSec |
| --- | --- | --- | --- |
| **Speed** | Fastest | Good | Good |
| **Code size** | 4,000 lines | 100,000+ lines | 400,000+ lines |
| **Setup** | Simple | Complex | Complex |
| **Protocol** | UDP | TCP or UDP | ESP |
| **Encryption** | ChaCha20, Curve25519 | OpenSSL (configurable) | Various |
| **Mobile** | Excellent (roaming) | OK | OK |
| **Default port** | 51820 | 1194 | 500/4500 |

WireGuard is built into the Linux kernel since 5.6. It is faster, simpler, and more secure than any alternative.

> **Need a VPS?** [Vultr](https://www.vultr.com/?ref=9891619) (free credit), [DigitalOcean](https://m.do.co/c/0faa855ea85d) ($200 free credit), or [RackNerd](https://my.racknerd.com/aff.php?aff=19190) (cheap annual deals).

## Quick Setup with SamNet-WG (Recommended)

Setting up WireGuard manually works, but managing peers, generating configs, handling firewall rules, and creating QR codes for every client gets tedious fast — especially when you have 5+ users.

[**SamNet-WG (wg-orchestrator)**](https://github.com/SamNet-dev/wg-orchestrator) is a complete WireGuard VPN management solution that turns any Linux server into a fully managed VPN appliance in under 5 minutes.

### One-Command Install

```bash
curl -sSL https://raw.githubusercontent.com/SamNet-dev/wg-orchestrator/main/install.sh | sudo bash
```

The interactive installer handles everything — subnet configuration, firewall rules, kernel module checks, and creates your first peer with a scannable QR code.

### What SamNet-WG Gives You

| Feature | Manual WireGuard | SamNet-WG |
| --- | --- | --- |
| **Setup time** | 30+ minutes | Under 5 minutes |
| **Add a peer** | Edit config, generate keys, restart | One command or TUI click |
| **QR codes for mobile** | Install qrencode, manually generate | Built-in, instant |
| **Bandwidth tracking** | Not available | Per-peer with ASCII graphs |
| **Bandwidth limits** | Not available | Set per peer (e.g., 10GB/month) |
| **Temporary access** | Manual expiry tracking | Auto-expiring links |
| **Web dashboard** | Not available | React-based admin panel |
| **Firewall management** | Manual iptables | Built-in port manager |
| **Kill switch** | Manual config | One toggle |

### TUI Management

```bash
sudo samnet
```

The terminal UI gives you:

- **Dashboard** — server status, connected peers, bandwidth overview
- **Peers** — add, remove, disable, set limits, generate QR codes
- **Firewall** — manage open ports, auto-detects existing services
- **Settings** — change port, DNS, subnet, enable/disable features
- **Logs** — live connection logs with filtering

### Quick Peer Management

```bash
# Add a peer
sudo samnet → Peers → Add Peer → Enter name → Scan QR code on phone

# Or from command line:
sudo samnet peer add alice
sudo samnet peer add bob --limit 10G --expires 2026-12-31

# List peers with status
sudo samnet peer list

# Get QR code for existing peer
sudo samnet peer qr alice

# Remove a peer
sudo samnet peer remove alice

# Set bandwidth limit
sudo samnet peer limit bob 5G
```

### Web Dashboard (Optional)

SamNet-WG includes a React-based web dashboard for remote management:

```bash
sudo samnet web enable
# Dashboard available at https://your-server:8443
```

Features: peer management, real-time bandwidth graphs, add/remove peers from your phone, CSRF protection, Argon2id authentication.

### Bi-Directional Sync

Changes made in the TUI, CLI, API, or Web Dashboard all stay in sync automatically. Edit a peer in the web UI and it is immediately reflected in the terminal and vice versa.

### Why Use SamNet-WG Instead of Manual Setup?

- **Zero config file editing** — never touch wg0.conf by hand
- **No restart needed** — peers are added/removed live
- **Per-peer bandwidth tracking** — see who is using how much
- **Temporary access** — create links that auto-expire
- **Multi-server** — sync config across multiple VPN servers
- **Security** — Argon2id password hashing, strict IP validation, CSRF protection

Get it at: [github.com/SamNet-dev/wg-orchestrator](https://github.com/SamNet-dev/wg-orchestrator)

---

If you want to understand what is happening under the hood, or prefer manual setup, continue below.

## Manual Setup

### Step 1: Install WireGuard

```bash
# Ubuntu / Debian
apt update && apt install wireguard -y

# CentOS / RHEL
yum install epel-release elrepo-release -y
yum install kmod-wireguard wireguard-tools -y

# Verify
wg --version
```

### Step 2: Generate Server Keys

```bash
cd /etc/wireguard
umask 077

# Generate server keypair
wg genkey | tee server_private.key | wg pubkey > server_public.key

# View keys
cat server_private.key
cat server_public.key
```

### Step 3: Server Configuration

Create `/etc/wireguard/wg0.conf`:

```
[Interface]
# Server private key
PrivateKey = SERVER_PRIVATE_KEY_HERE

# VPN subnet — the server gets .1
Address = 10.66.66.1/24

# Port to listen on
ListenPort = 51820

# Enable IP forwarding and NAT when interface comes up
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# DNS for this server when tunnel is up (clients set their own DNS)
# DNS is set on client side, not server

[Peer]
# Client 1
PublicKey = CLIENT1_PUBLIC_KEY_HERE
AllowedIPs = 10.66.66.2/32
```

**Replace `eth0`** with your actual network interface. Find it with `ip route show default`.

### Step 4: Enable IP Forwarding

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

### Step 5: Open Firewall

```bash
# UFW
ufw allow 51820/udp

# iptables
iptables -A INPUT -p udp --dport 51820 -j ACCEPT
```

### Step 6: Start WireGuard

```bash
# Start
wg-quick up wg0

# Enable on boot
systemctl enable wg-quick@wg0

# Check status
wg show
```

## Client Configuration

### Generate Client Keys

```bash
# On the server (or client machine)
wg genkey | tee client1_private.key | wg pubkey > client1_public.key
```

### Client Config File

Create `client1.conf`:

```
[Interface]
# Client private key
PrivateKey = CLIENT1_PRIVATE_KEY_HERE

# Client IP in the VPN subnet
Address = 10.66.66.2/24

# DNS servers to use when VPN is active
DNS = 1.1.1.1, 1.0.0.1

[Peer]
# Server public key
PublicKey = SERVER_PUBLIC_KEY_HERE

# Server public IP and port
Endpoint = YOUR_SERVER_IP:51820

# Route ALL traffic through VPN
AllowedIPs = 0.0.0.0/0

# Keep connection alive (important for NAT)
PersistentKeepalive = 25
```

### AllowedIPs Explained

| AllowedIPs | What it does |
| --- | --- |
| `0.0.0.0/0` | Route ALL traffic through VPN (full tunnel) |
| `10.66.66.0/24` | Only route VPN subnet traffic (split tunnel) |
| `10.66.66.0/24, 192.168.1.0/24` | VPN subnet + specific remote network |

### Add Client to Server

Add the client's public key to the server config (`/etc/wireguard/wg0.conf`):

```
[Peer]
PublicKey = CLIENT1_PUBLIC_KEY_HERE
AllowedIPs = 10.66.66.2/32
```

Then reload:

```bash
wg-quick down wg0 && wg-quick up wg0
# Or add peer without restart:
wg set wg0 peer CLIENT1_PUBLIC_KEY allowed-ips 10.66.66.2/32
```

## Mobile Setup

### iPhone / Android

1. Install the **WireGuard** app from App Store / Play Store
2. Generate a QR code from the client config:

```bash
apt install qrencode -y
qrencode -t ansiutf8 < client1.conf
```

1. In the WireGuard app: tap **+** → **Scan from QR Code** → scan the terminal QR

### Generate QR from Config

```bash
# Show QR in terminal
qrencode -t ansiutf8 < /etc/wireguard/clients/client1.conf

# Save QR as image
qrencode -o client1-qr.png < /etc/wireguard/clients/client1.conf
```

## Adding More Clients

For each new client:

```bash
# 1. Generate keys
wg genkey | tee client2_private.key | wg pubkey > client2_public.key

# 2. Create client config (same as above, change Address to .3)
# Address = 10.66.66.3/24

# 3. Add to server config
cat >> /etc/wireguard/wg0.conf << EOF

[Peer]
PublicKey = $(cat client2_public.key)
AllowedIPs = 10.66.66.3/32
EOF

# 4. Reload
wg-quick down wg0 && wg-quick up wg0
```

## Management Commands

```bash
# Show status and connected peers
wg show

# Show specific interface
wg show wg0

# Show transfer stats
wg show wg0 transfer

# Add peer without restart
wg set wg0 peer PUBLIC_KEY allowed-ips 10.66.66.4/32

# Remove peer
wg set wg0 peer PUBLIC_KEY remove

# Start / stop
wg-quick up wg0
wg-quick down wg0

# Check if running
systemctl status wg-quick@wg0
```

## Troubleshooting

### Can't connect

```bash
# Check WireGuard is running
wg show

# Check port is open
ss -ulnp | grep 51820

# Check firewall
iptables -L -n | grep 51820
ufw status | grep 51820

# Check IP forwarding
sysctl net.ipv4.ip_forward
```

### Connected but no internet

```bash
# Check NAT rules
iptables -t nat -L -n

# Check interface name in PostUp/PostDown (must match your actual interface)
ip route show default
# If it shows "dev ens3" not "dev eth0", update wg0.conf
```

### DNS not working

```bash
# Make sure DNS is set in client config
# Try different DNS: 8.8.8.8 instead of 1.1.1.1

# On Linux client, check resolv.conf
resolvectl status
```

### High latency

```bash
# Check MTU — WireGuard default is 1420, some networks need lower
# In [Interface] section of client config:
MTU = 1280
```

## Security Best Practices

1. **Never share private keys** — each device gets its own keypair
2. **Use `umask 077`** when generating keys — prevents other users from reading them
3. **Restrict AllowedIPs** on the server — each client should only have its own IP (`/32`)
4. **Change the default port** from 51820 if targeted by censorship
5. **Enable the kill switch** on clients — set AllowedIPs to `0.0.0.0/0` to route everything
6. **Keep WireGuard updated** — `apt upgrade wireguard`

## Test Your VPN

After connecting, verify everything works:

- [VPN Leak Test](https://www.samnet.dev/tools/vpn-leak-test/) — check for IP, DNS, and WebRTC leaks
- [What's My IP](https://www.samnet.dev/tools/whats-my-ip/) — verify your IP changed to the VPN server's IP
- [Speed Test](https://www.samnet.dev/speedtest/) — check VPN speed impact
- [DNS Toolbox](https://www.samnet.dev/tools/dns-toolbox/) — verify DNS goes through the VPN

### Tip: SaveConfig

Add `SaveConfig = true` to your `[Interface]` section to auto-save runtime peer changes back to the config file. Without it, `wg set` changes are lost on restart.
