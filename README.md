# ssh-remote-server-setup

SSH Configuration with Asymmetric Key Authentication Enforced by Fail2Ban.
This repository is intended to document and demonstrate the configuration process for secure remote access via the SSH (Secure Shell) protocol using an asymmetric key pair (public/private).

Key-based authentication is considered much more secure than traditional passwords, as it is immune to dictionary-based brute-force attacks.

## Workflow

> **Lab environment:**
> - **SERVER** — Debian 13 Trixie (remote server)
> - **CLIENT** — Windows 11 (host machine)

Asymmetric authentication is based on two locally generated keys:

- **Private Key:** Remains on your computer (client) and must never be shared.
- **Public Key:** Copied to the remote server.

### 1. Key Pair Generation

To generate a new key pair using the modern Ed25519 algorithm (more secure and faster than RSA), run the following command in the terminal:

```powershell
ssh-keygen -t ed25519 -C "email@example.com" -f $env:USERPROFILE\.ssh\keyfile
```

### 2. Install OpenSSH Server on the Debian Server

```bash
apt update && sudo apt install -y openssh-server
```

### 3. Install Fail2Ban and Unattended Upgrades

Install Fail2Ban to enforce IP bans on repeated failed SSH login attempts, and Unattended Upgrades to enable automatic security updates on Debian:

```bash
apt update && sudo apt install -y fail2ban unattended-upgrades
```

Enable and start the automatic security updates service:

```bash
systemctl enable unattended-upgrades
systemctl start unattended-upgrades
systemctl status unattended-upgrades
```

#### 3.1. Configure Fail2Ban

```bash
cd /etc/fail2ban
cp jail.conf jail.local
vim jail.local
```

Add or update the `[sshd]` section in `jail.local`:

```ini
[sshd]
enabled  = true
port     = 2222        # We will change the SSH port shortly for security reasons
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
findtime = 300
bantime  = 3600
ignoreip = 127.0.0.1/8
```

Enable and start Fail2Ban:

```bash
systemctl enable fail2ban
systemctl start fail2ban
systemctl status fail2ban
```

### 4. Copy the Public Key from Client to Server

Run this command in **PowerShell** to transfer your public key to the remote server.

> **Note:** Make sure the `~/.ssh` directory already exists on the server before running this command (`mkdir -p ~/.ssh`).

```powershell
type $env:USERPROFILE\.ssh\keyfile.pub | ssh user@remote_address "cat >> .ssh/authorized_keys"
```

### 5. Server Access

Once configured, you can log in without a password:

```bash
ssh user@remote_address
```

### 6. Change the Default SSH Port and Harden the Configuration

Edit the SSH daemon configuration file:

```bash
vim /etc/ssh/sshd_config
```

Apply the following changes:

```
Port 2222                  # Remove the '#' and update jail.local accordingly
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no         # Highly recommended
```

#### 6.1. CRITICAL: Do Not Close Your Current Session!

Verify the new configuration works before closing your existing session. Restart the SSH daemon:

```bash
systemctl restart ssh
```

> On Debian, the SSH service is named `ssh` (not `sshd`).

#### 6.2. Test the New Connection

Open a **new terminal** on the Windows client and connect using the new port and key:

```powershell
ssh -i C:\Users\username\.ssh\keyfile -p 2222 user@remote_address
```

If the login succeeds — well done! You can now safely close the old session.

### 7. Verify Banned IPs

Check the list of IPs currently banned by Fail2Ban:

```bash
fail2ban-client status sshd
```

---

Reference: <https://roadmap.sh/projects/ssh-remote-server-setup>
