# AuraNodes LXC Discord Management Bot

> A comprehensive, high-performance Discord bot for provisioning and managing LXC/LXD Virtual Private Servers directly from your Discord server.

Built with Python and `discord.py`, AuraNodes transforms a standard Linux host into a fully automated, interactive VPS hosting platform — complete with a button-based control panel, real-time monitoring, and multi-node cluster support.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Commands](#commands)
- [Custom Branding](#custom-branding)

---

## Features

| Feature | Description |
|---|---|
| 🎛️ Interactive Control Panel | Button and dropdown-based dashboard to start, stop, reinstall, and monitor VPS instances |
| 🌐 Multi-Node Cluster | Deploy and manage containers across local and remote LXC host machines |
| 🔑 Instant SSH Access | Auto-generates secure, temporary web SSH sessions via `tmate` — no key management needed |
| 📈 Live Resource Monitoring | Real-time CPU, RAM, and disk tracking with automated suspension for abuse |
| 🔌 Port Forwarding | Built-in network management with customizable per-user allocation quotas |
| 🐳 Docker-Ready Containers | Automatically configures nesting, privileged mode, FUSE, and kernel modules |
| 💾 Snapshots & Migrations | Admin tools for 1-click cloning, snapshots, and live node-to-node migrations |
| ⚡ Optimized Backend | SQLite with WAL (Write-Ahead Logging) to prevent locking under concurrent operations |

---

## Prerequisites

- **OS:** Ubuntu 20.04+ (recommended) or Debian 10+ — KVM VPS or bare-metal
- **Runtime:** Python 3.8+, `snapd`, `git`
- **Discord:** A registered bot token with **Server Members Intent** enabled in the [Developer Portal](https://discord.com/developers/applications)

---

## Installation

### 1. Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install LXC/LXD

**Ubuntu:**
```bash
sudo apt install lxc lxc-utils snapd bridge-utils uidmap -y
sudo systemctl enable --now snapd.socket
sudo snap install lxd
sudo usermod -aG lxd $USER
newgrp lxd
sudo lxd init
```

**Debian:**
```bash
sudo apt install snapd lxc lxc-utils bridge-utils uidmap -y
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install lxd
sudo usermod -aG lxd $USER
newgrp lxd
sudo lxd init
```

> Accept the default configuration when prompted by the `lxd init` wizard.

### 3. Set Up Python & Virtual Environment

```bash
sudo apt install python3 python3-pip python3-venv git -y

mkdir -p ~/.config/pip
echo -e "[global]\nbreak-system-packages = true" > ~/.config/pip/pip.conf

mkdir -p /root/auranodes-bot && cd /root/auranodes-bot

python3 -m venv venv
source venv/bin/activate
pip install discord.py requests
```

### 4. Container SSH Configuration

Apply this script to enable root password logins inside LXC containers:

```bash
sudo bash -c 'cat <<EOF > /etc/ssh/sshd_config
# SSH LOGIN SETTINGS
PasswordAuthentication yes
PermitRootLogin yes
PubkeyAuthentication no
ChallengeResponseAuthentication no
UsePAM yes

# SFTP SETTINGS
Subsystem sftp /usr/lib/openssh/sftp-server
EOF
systemctl restart ssh 2>/dev/null || service ssh restart
passwd root'
```

### 5. Run as a Systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/auranodes.service
```

Paste the following:

```ini
[Unit]
Description=AuraNodes LXC Discord Management Bot
After=network.target

[Service]
User=root
WorkingDirectory=/root/auranodes-bot
ExecStart=/root/auranodes-bot/venv/bin/python /root/auranodes-bot/bot.py
Restart=always
RestartSec=5
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

Enable and start the bot:

```bash
sudo systemctl daemon-reload
sudo systemctl restart auranodes
sudo systemctl enable auranodes
sudo systemctl status auranodes
```

---

## Configuration

Create a `.env` file at `/root/auranodes-bot/.env`:

```env
# Core Discord Settings
DISCORD_TOKEN=your_bot_token_here
PREFIX=!
MAIN_ADMIN_ID=your_discord_user_id
VPS_USER_ROLE_ID=your_vps_role_id

# Host Configuration
YOUR_SERVER_IP=127.0.0.1
DEFAULT_STORAGE_POOL=default

# Branding & Versioning
BOT_NAME=AuraNodes.fun
BOT_VERSION=7.0-PRO
BOT_DEVELOPER=AuraNodes Team
BOT_LOGO_URL=https://your_logo_url_here.png
```

| Variable | Description |
|---|---|
| `DISCORD_TOKEN` | Your bot token from the Discord Developer Portal |
| `PREFIX` | Command prefix (default: `!`) |
| `MAIN_ADMIN_ID` | Your Discord User ID (Snowflake) |
| `VPS_USER_ROLE_ID` | ID of the role granted to VPS owners |
| `YOUR_SERVER_IP` | Public IPv4 address of the host machine |
| `DEFAULT_STORAGE_POOL` | LXD storage pool name (default: `default`) |
| `BOT_NAME` | Display name shown in embeds and titles |
| `BOT_DEVELOPER` | Credit shown in the `!about` command |
| `BOT_LOGO_URL` | Direct image URL used in embed thumbnails |

---

## Commands

### User Commands

| Command | Description |
|---|---|
| `!ping` | Check bot API and gateway latency |
| `!myvps` | Open your personal VPS dashboard |
| `!manage` | Open the interactive UI to start, stop, SSH, or reinstall your VPS |
| `!ports` | View and manage your assigned port forwards |
| `!share-user @user <id>` | Grant another user access to manage your VPS |

### Admin Commands

| Command | Description |
|---|---|
| `!create <ram> <cpu> <disk> @user` | Deploy a new VPS for a user via the UI |
| `!status` | View cluster health, CPU/RAM load, and node availability |
| `!delete-vps @user <id>` | Forcefully delete a VPS and wipe its data |
| `!add-resources <vps>` | Dynamically adjust a container's RAM, CPU, or disk |
| `!node create` | Register a new remote LXC node to the cluster |
| `!suspend-vps <vps> <reason>` | Immediately pause a VPS for policy violations |

---

## Custom Branding

To rebrand the bot for your own community, edit the following variables in `bot.py`:

| Element | Location | Description |
|---|---|---|
| `BOT_NAME` | Line 22 | Global bot identifier shown in titles |
| `BOT_VERSION` | Line 27 | Version tag displayed in footers |
| `BOT_DEVELOPER` | Line 28 | Credit shown in `!about` |
| `OS_OPTIONS` | Line 32 | Dropdown menu options for OS installs |
| Footer Texts | Line 454+ | Core embed builder (`create_embed` function) |

Alternatively, set `BOT_NAME` and `BOT_LOGO_URL` in your `.env` file — all embeds, headers, and UI footers update automatically.

---

Built with ❤️ for Discord communities and hosting providers by the **AuraNodes Team**.
