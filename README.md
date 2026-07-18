AuraNodes LXC Discord Management Bot
A comprehensive, fully-featured Discord bot for managing LXC/LXD Virtual Private Servers (VPS) directly from your Discord server. This v7.0-PRO release features a completely optimized SQLite implementation, interactive UI elements, and multi-node cluster support.
Core Features
 * Interactive Control Panel: Button-based dashboard for users to start, stop, reinstall, and monitor their VPS instances.
 * Instant SSH Access: Generates secure, temporary web-based SSH sessions using tmate.
 * Multi-Node Support: Deploy and manage containers across local and remote LXC hosts.
 * Live Resource Monitoring: Real-time tracking of CPU, RAM, and Disk usage with automated suspension for abuse.
 * Network Management: Built-in automated port forwarding allocation and quota system.
 * Docker Ready: Containers are automatically configured with nesting, privileged mode, and FUSE support.
Prerequisites
 * A dedicated host or KVM VPS running Ubuntu 20.04+ or Debian 10+.
 * A Discord Bot Token with Server Members Intent enabled.
 * Basic understanding of Linux terminal commands.
Deployment Guide
1. System Preparation
Always update your OS repository index to prevent dependency conflicts.
sudo apt update && sudo apt upgrade -y

2. Install Virtualization Stack (LXC/LXD)
For Ubuntu:
sudo apt install lxc lxc-utils snapd bridge-utils uidmap -y
sudo systemctl enable --now snapd.socket
sudo snap install lxd
sudo usermod -aG lxd $USER
newgrp lxd
sudo lxd init

For Debian:
sudo apt install snapd lxc lxc-utils bridge-utils uidmap -y
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install lxd
sudo usermod -aG lxd $USER
newgrp lxd
sudo lxd init

(Accept the default configurations when prompted by the lxd init wizard).
3. Setup Python & Virtual Environment
Configure pip to accept global environments safely and install dependencies.
sudo apt install python3 python3-pip python3-venv git -y
mkdir -p ~/.config/pip
echo -e "[global]\nbreak-system-packages = true" > ~/.config/pip/pip.conf

# Clone/Move your files to /root/auranodes-bot
cd /root/auranodes-bot
python3 -m venv venv
source venv/bin/activate
pip install discord.py requests

4. Container SSH Configuration
Apply this script to ensure SSH access remains reliable inside your LXC containers and allows root login.
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

5. Start as a System Service
Run the bot in the background automatically using Systemd.
Create the service file:
sudo nano /etc/systemd/system/auranodes.service

Paste the following:
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

Enable and start the bot:
sudo systemctl daemon-reload
sudo systemctl restart auranodes
sudo systemctl enable auranodes
sudo systemctl status auranodes

Environment Configuration
Create a .env file in your bot's root directory (/root/auranodes-bot/.env) and configure the following variables:
| Variable | Description | Default / Example |
|---|---|---|
| DISCORD_TOKEN | Your Discord developer bot token | YOUR_TOKEN_HERE |
| PREFIX | Command prefix for the bot | ! |
| YOUR_SERVER_IP | Public IPv4 of your host machine | 127.0.0.1 |
| MAIN_ADMIN_ID | Your Discord User ID (Snowflake) | 1210291131301101618 |
| VPS_USER_ROLE_ID | ID of the role given to VPS owners | 1210291131301101618 |
| DEFAULT_STORAGE_POOL | LXD storage pool name | default |
| BOT_NAME | Custom name for your service | AuraNodes.fun |
| BOT_DEVELOPER | Your team name | AuraNodes Team |
| BOT_LOGO_URL | Direct image URL for embeds | YOUR_LOGO_URL |
Core Commands
| Command | Access | Description |
|---|---|---|
| !ping | Everyone | Verifies API network latency and Discord gateway status. |
| !myvps | Everyone | Opens the tenant control panel showing assigned container slices. |
| !manage | Everyone | Opens the interactive virtualization dashboard (Power, SSH, Reinstall). |
| !create | Admin | Allocates hardware profiles to a specific tenant via UI selection. |
| !status | Admin | Displays cluster health, active components, storage, and CPU load. |
Custom Branding Guide
To completely rebrand the bot from the default AuraNodes.fun theme, modify the following variables directly inside bot.py:
| Element | Location | Description |
|---|---|---|
| BOT_NAME | Line 22 | Global bot identifier shown in titles. |
| BOT_VERSION | Line 27 | Version tag displayed in footers. |
| BOT_DEVELOPER | Line 28 | Credit shown in the !about command. |
| OS_OPTIONS | Line 32 | Dropdown menu options for OS installations. |
| Footer Texts | Line 454+ | Core embed builder (create_embed function). |
