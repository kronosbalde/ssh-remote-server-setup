# ssh-remote-server-setup
SSH Configuration with Asymmetric Key Authentication Enforced by Fail2Ban.
This repository is intended to document and demonstrate the configuration process for secure remote access via the SSH (Secure Shell) protocol using an asymmetric key pair (public/private).

Key-based authentication is considered much more secure than traditional passwords, as it is immune to dictionary-based brute-force attacks.

# Workflow
NB: For this Lab, we used
 - Debian 13 Trixie as the remote server - SERVER
 - Windows 11 as the host machine - CLIENT

Asymmetric authentication is based on two locally generated keys:
 - Private Key: It remains on your computer (client) and must never be shared.
 - Public Key: It is copied to the remote server.

1. Key pair generation
To generate a new key pair using the modern Ed25519 algorithm (which is more secure and performs better than RSA), run the following command in the terminal:

```ssh-keygen -t ed25519 -C "email@example.com" -f C:\Users\username\.ssh\keyfile```

2. Install OpenSSH Server on Debian Server

```sudo apt update && sudo apt install -y openssh-server```

3. Install Fail2Ban and Unattended Upgrades to ensure Debian activate auto security updates.

```sudo apt update; sudo apt install -y fail2ban unattended-upgrades```

We install fail2ban to enforce IP ban when someone try to log in via SSH into our Server and unattended-upgrades to enable automatic security updates on Debian.

Now enable security updates via Systemd:

Automatic starts on boot
```systemctl enable unattended-upgrades```

Start service

```systemctl start unattended-upgrades```

Check if service has some problems to start

```systemctl status unattended-upgrades```

    3.1. Let's configure Fail2Ban
    
    ``` cd /etc/fail2ban ```
    ``` cp jail.conf jail.local ```

    Edit jail.local to write the snippet code to activate the guard of SSH
    ``` sudo vim jail.local ```

    Jail.local
    ```
    [sshd]  
    enabled = true  
    port = 2222                           # We will change the port soon for security reasons  
    filter = sshd  
    logpath = /var/log/auth.log  
    maxretry = 3  
    findtime = 300  
    bantime = 3600  
    ignoreip = 127.0.0.1
    ```

    Enable Fail2ban
    ```systemctl enable fail2ban```

    Start Fail2ban
    ```systemctl start fail2ban```

    Check if fail2ban has problem on start
    ```systemctl status fail2ban```

4. Copy the public key generated on client to the server
Use this script with POWERSHELL to transfer your public key to the destination server:

```type $env:USERPROFILE\.ssh\keyfile.pub | ssh user@remote_address "cat >> .ssh/authorized_keys"```

5. Server Access
Once it's set up, you can log in without entering your password:

```ssh user@remote_address```

6. Change the standard SSH port & Hardening

Edit the configuration file:
```sudo vim /etc/ssh/sshd_config```

Apply these changes:
- Port 2222 (Ensure you remove the '#' and update your Fail2Ban jail.local accordingly)
- PubkeyAuthentication yes
- PasswordAuthentication no
- PermitRootLogin no (Highly recommended)

6.1. CRITICAL: Do not close your current session until you verify the new one works!    
    Restart SSH daemon via Systemd:
    
    ``` systemctl restart ssh ```

    6.2 Open another terminal on Windows Client and try to login via SSH

    ``` ssh -i C:\Users\username\.ssh\keyfile -p 2222 user@remote_address ```

And now we can also check the banned IPs with this command:
```sudo fail2ban-client status sshd```

NB: If you log in into server, fine. So, well done!


https://roadmap.sh/projects/ssh-remote-server-setup