# Ubuntu Server Setup Guide

This guide shows how to install and configure **SSH, Apache2, SFTP user, and Webmin** on a fresh Ubuntu server.

---

## 1. Install and Enable OpenSSH Server

Run on your Ubuntu server console (local machine or cloud provider terminal):

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Verify SSH is running:

```bash
sudo systemctl status ssh
```

---

## 2. Access the Server via SSH

From your **local computer**, connect to the server (replace `your_ip` and `your_user`):

```bash
ssh your_user@your_server_ip
```

---

## 3. Install Apache2 Web Server

Once connected via SSH, install Apache2:

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

Test by visiting in your browser:
```
http://YOUR_SERVER_IP
```

---

## 4. Configure SFTP Restricted User

### Create group and user (replace `mfh_sftp` with your username):

```bash
sudo groupadd sftpusers
sudo useradd -m -g sftpusers -s /bin/false mfh_sftp
sudo passwd mfh_sftp
```

### Setup directories:

```bash
sudo mkdir -p /var/www/sftp/
sudo chown root:root /var/www/sftp
sudo chmod 755 /var/www/sftp

sudo mkdir -p /var/www/sftp/html
sudo mount --bind /var/www/html /var/www/sftp/html
```

### Make mount permanent in `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add this line:

```
/var/www/html /var/www/sftp/html none bind 0 0
```

### Fix permissions:

```bash
sudo chown -R root:sftpusers /var/www/html
sudo chmod -R 775 /var/www/html
sudo usermod -aG sftpusers www-data
```

### Configure SSH for SFTP user:

```bash
sudo nano /etc/ssh/sshd_config
```

Add at the bottom:

```
Match User mfh_sftp
    ChrootDirectory /var/www/sftp
    ForceCommand internal-sftp
    X11Forwarding no
    AllowTcpForwarding no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 5. Install Webmin

### Install dependencies:

```bash
sudo apt install software-properties-common apt-transport-https curl -y
```

### Add Webmin key and repository:

```bash
curl -fsSL https://download.webmin.com/developers-key.asc | sudo gpg --dearmor -o /usr/share/keyrings/webmin.gpg

echo "deb [signed-by=/usr/share/keyrings/webmin.gpg] https://download.webmin.com/download/newkey/repository stable contrib" | sudo tee /etc/apt/sources.list.d/webmin.list
```

### Install Webmin:

```bash
sudo apt update
sudo apt install webmin -y
```

👉 Access Webmin via:
```
https://YOUR_SERVER_IP:10000
```

---

✅ Final Setup:
- SSH access enabled
- Apache2 web server running
- Secure SFTP user (`mfh_sftp`) configured
- Webmin installed for web-based management
