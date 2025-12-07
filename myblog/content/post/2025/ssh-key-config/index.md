---
date : '2024-12-19T16:01:05+11:00'
title : 'SSH Connection (Without Password) to a Remote Server'
image : "/img/posts/2025/ssh.jpg"
Description  : "Setting Up SSH Connection Without Password to a Remote Server'..."
---
# Introduction

Secure Shell (SSH) is a powerful tool for remotely accessing and managing servers. Setting up SSH key-based authentication eliminates the need for passwords and enhances security. This guide walks you through the process of setting up an SSH connection without a password to a remote server.

---

## Step 1: Generate an SSH Key Pair

The first step is to create a public and private SSH key pair on your local machine. If you already have an SSH key pair, you can skip this step.

1. Open a terminal on your local machine.
2. Run the following command:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   > **_NOTE:_**   `ssh-keygen` uses default options if no flag provided.
   - `-t rsa`: Specifies the RSA algorithm.(optional)
   - `-b 4096`: Generates a 4096-bit key for extra security. (optional)
   - `-C "your_email@example.com"`: Adds a comment to identify the key. (optional)

3. Save the key to the default location (`~/.ssh/id_rsa`) with a key name to store the key when prompted.
4. Consider adding a passphrase to the key for extra security.

---

## Step 2: Copy the Public Key to the Remote Server

You need to transfer the public key to the remote server so it knows to trust your private key.

### Option 1: Use `ssh-copy-id` (Recommended)
Run the following command:
```bash
ssh-copy-id -i ~/.ssh/<key_name> <user>@<remote_host>
```
- Replace `<key_name>` with your key's name you set earlier.
- Replace `<user>` with your remote username.
- Replace `<remote_host>` with the server's IP address or domain name.

### Option 2: Manually Copy the Key
1. Display the contents of your public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
2. Copy the output and paste it into the `~/.ssh/authorized_keys` file on the remote server:
   ```bash
   vim ~/.ssh/authorized_keys
   ```
3. Ensure the file has the correct permissions:
   ```bash
   chmod 600 ~/.ssh/authorized_keys
   ```

---

## Step 3: Test the Connection

Verify that you can connect to the server without a password:
```bash
ssh -i .ssh/<key_name> <user>@<remote_host>
```
> **_NOTE:_**  The "-i" flag is used to point to your key's location, not needed to specify when key's name is default as you should be logged in without being prompted for a password.

---

## Step 4: Disable Password Authentication (Optional but Recommended)

To enhance security, disable password authentication on the remote server. This ensures only users with the private key can log in.

> **_NOTE:_**   This will block password login from ssh but still allow on-site direct connection with hdmi and monitor.
1. Open the SSH daemon configuration file on the remote server:
   ```bash
   sudo vim /etc/ssh/sshd_config
   ```
2. Set the following options:
   ```
   PasswordAuthentication no
   PermitRootLogin no
   Port 666 # Default is 22, everyone knows that :/
   ```
   > **_NOTE:_**  Add flag "-p <PORT_NUM>" when ssh if changed PORT.
3. Restart the SSH service:
   ```bash
   sudo systemctl restart sshd
   ```

### SUPER IMPORTANT !!!
Before disabling password authentication, ensure you have tested key-based access and confirmed it works.

---

## Troubleshooting

### 1. SSH Still Asks for a Password
- Ensure the public key is in `~/.ssh/authorized_keys` on the remote server.
- Check file permissions:
  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```
- Verify the SSH configuration:
  ```bash
  sudo vim /etc/ssh/sshd_config
  ```
  Ensure `PubkeyAuthentication yes` is enabled.

### 2. Debug the SSH Connection
Use verbose mode to debug:
```bash
ssh -v <user>@<remote_host>
```
This will provide detailed output about the connection process.

---

## Conclusion

Setting up SSH key-based authentication not only improves security but also makes logging into remote servers more convenient. By following the steps above, you can eliminate the need for passwords and protect your server against brute-force attacks.

Remember to keep your private key secure, and consider disabling password authentication for maximum security.

Happy server managing!

