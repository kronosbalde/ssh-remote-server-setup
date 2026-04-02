# ssh-remote-server-setup
SSH Configuration with Asymmetric Key Authentication
This repository is intended to document and demonstrate the configuration process for secure remote access via the SSH (Secure Shell) protocol using an asymmetric key pair (public/private).

Key-based authentication is considered much more secure than traditional passwords, as it is immune to dictionary-based brute-force attacks.

# Workflow
Asymmetric authentication is based on two locally generated keys:
    - Private Key: It remains on your computer (client) and must never be shared.
    - Public Key: It is copied to the remote server.

1. Key pair generation
To generate a new key pair using the modern Ed25519 algorithm (which is more secure and performs better than RSA), run the following command in the terminal:

ssh-keygen -t ed25519 -C "email@example.com"

2. Copy the public key to the server
Use the ssh-copy-id utility to transfer your public key to the destination server:

ssh-copy-id utente@remote_address

3. Server Access
Once it's set up, you can log in without entering your password:

ssh user@remote_address

https://roadmap.sh/projects/ssh-remote-server-setup