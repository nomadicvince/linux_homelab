# Complete Guide: Installing Ubuntu Server and Building a Kubernetes Cluster

## Project Goals
1. Convert a Windows tower into an Ubuntu Server
2. Configure the server for external backup storage
3. Set up a Kubernetes cluster using the tower as the control plane node
4. Add three Raspberry Pi devices (1x Pi 5, 2x Pi 4) as worker nodes
5. Deploy Nextcloud on one of the nodes for file storage and collaboration
6. Create a secure, reliable infrastructure for data storage and container orchestration

This guide walks you through replacing Windows with Ubuntu Server on your tower computer, setting up external backup storage, deploying a Kubernetes cluster with Raspberry Pi worker nodes, and installing Nextcloud.

## Table of Contents
- Part 1: Creating Installation Media
- Part 2: Booting from USB and Installing Ubuntu Server
- Part 3: Post-Installation Configuration
- Part 4: Making Your Server Internet-Facing
- Part 5: Verifying Installation
- Part 6: External Backup Storage Configuration
- Part 7: Kubernetes Cluster Setup
- Part 8: Deploying Nextcloud on Kubernetes

## Part 1: Creating Installation Media

### 1. Download Ubuntu Server
- Visit: https://ubuntu.com/download/server
- Download the latest LTS version (Ubuntu Server 24.04.2 LTS)

### 2. Create Bootable USB Drive with Balena Etcher
- Get a USB drive with at least 4GB capacity
- Download Balena Etcher from https://www.balena.io/etcher/
- Install and open Balena Etcher
- Click "Flash from file" and select your downloaded Ubuntu Server ISO
- Click "Select target" and choose your USB drive
- Click "Flash!" and wait for the process to complete
- Etcher will automatically verify the flash when done

## Part 2: Booting from USB and Installing Ubuntu Server

### 1. Boot From USB
- Insert the USB drive into your tower
- Restart your computer
- During startup, press the key to enter boot menu (often F12, F2, F10, ESC, or Delete)
- Select your USB drive from the boot menu
- When you see the Ubuntu boot screen, select "Install Ubuntu Server"

### 2. Basic Setup
- Select your language preference
- Select your keyboard layout
- Configure network connections (choose either DHCP for automatic or static IP)
- Configure proxy if needed (leave blank if not using a proxy)
- Configure Ubuntu archive mirror (leave default unless you have a specific preference)

### 3. Storage Configuration (This Step Removes Windows)
- When you reach the "Guided storage configuration" screen, select "Use An Entire Disk"
- Select the disk where Windows is currently installed
- On the "Storage configuration" screen, confirm that you want to use the entire disk
- Review the partitioning layout and confirm with "Done"
- When prompted with "Confirm destructive action," select "Continue" (this will erase Windows)

### 4. User Setup
- Create a profile:
  - Enter your name
  - Choose a server name
  - Pick a username
  - Set a strong password

### 5. SSH Setup
- Choose whether to install OpenSSH server (recommended for remote administration)
- If you choose to install SSH, you can optionally import SSH identity from GitHub

### 6. Featured Server Snaps
- Select any additional software you want to install (you can leave this blank)
- Click "Done"

### 7. Installation
- Wait for the installation to complete (10-30 minutes depending on your hardware)
- When prompted, select "Reboot Now"
- Remove the USB drive when prompted and press Enter

## Part 3: Post-Installation Configuration

### 1. Login and Update
- After reboot, log in with the username and password you created
- Update your system:
  ```
  sudo apt update
  sudo apt upgrade -y
  ```

### 2. Configure vim as Default Editor
```
sudo update-alternatives --config editor
```
- Select the number corresponding to vim and press Enter

### 3. Comprehensive Firewall Setup

#### 3.1. Basic UFW Configuration
```
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status verbose
```

#### 3.2. Configure Firewall for Common Services
If you'll be running web services:
```
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
```

For secure FTP:
```
sudo ufw allow 22/tcp    # SFTP uses SSH port
```

For mail services:
```
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 143/tcp   # IMAP
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 110/tcp   # POP3
sudo ufw allow 995/tcp   # POP3S
```

Limit connection attempts to SSH:
```
sudo ufw limit ssh
```

#### 3.3. Check Firewall Status
```
sudo ufw status numbered
```

### 4. Network Configuration

#### 4.1. Identify Network Interfaces
```
ip a
```
- Note the name of your main network interface (e.g., `enp0s3`, `ens33`, `eth0`)

#### 4.2. Set a Static IP (Using vim)
```
sudo vim /etc/netplan/00-installer-config.yaml
```
- Press `i` to enter insert mode
- Configure it similar to this (adjust for your network):
  ```yaml
  network:
    ethernets:
      enp0s3:  # Replace with your interface name from step 4.1
        dhcp4: no
        addresses: [192.168.1.100/24]  # Your desired internal IP
        gateway4: 192.168.1.1  # Your router IP
        nameservers:
          addresses: [8.8.8.8, 8.8.4.4]  # Google DNS or your preferred DNS
    version: 2
  ```
- Press `Esc` to exit insert mode
- Type `:wq` and press Enter to save and quit
- Apply the changes:
  ```
  sudo netplan apply
  ```

#### 4.3. Verify Network Configuration
```
ip a
ip route
cat /etc/resolv.conf
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### 5. Install Essential Server Packages
```
sudo apt install -y curl wget htop net-tools iotop iftop fail2ban
```

### 6. Configure Timesyncd (NTP)
```
sudo vim /etc/systemd/timesyncd.conf
```
- Press `i` to enter insert mode
- Uncomment and modify the NTP line:
  ```
  [Time]
  NTP=pool.ntp.org ntp.ubuntu.com
  ```
- Press `Esc`, type `:wq` and press Enter
- Restart the service:
  ```
  sudo systemctl restart systemd-timesyncd
  ```

### 7. Configure SSH for Security and Remote Access

#### 7.1. Basic SSH Security Configuration
```
sudo vim /etc/ssh/sshd_config
```
- Press `i` to enter insert mode
- Find and modify these settings (or add them if not present):
  ```
  PermitRootLogin no
  PasswordAuthentication yes  # Change to 'no' if using key-based auth only
  X11Forwarding no
  MaxAuthTries 3
  LoginGraceTime 60
  PermitEmptyPasswords no
  Protocol 2
  ```
- Press `Esc`, type `:wq` and press Enter
- Restart SSH:
  ```
  sudo systemctl restart sshd
  ```

#### 7.2. Set Up SSH Key-Based Authentication (More Secure)
On your client machine (not the server):
```
ssh-keygen -t ed25519 -C "your_email@example.com"  # Generate key pair
ssh-copy-id username@server_ip_address             # Copy key to server
```

After successfully copying your key:
```
sudo vim /etc/ssh/sshd_config
```
- Find `PasswordAuthentication` and change it to:
  ```
  PasswordAuthentication no
  ```
- Restart SSH:
  ```
  sudo systemctl restart sshd
  ```

#### 7.3. Test SSH Access
From your client machine:
```
ssh username@server_ip_address
```

### 8. Set Up Automatic Security Updates
```
sudo apt install -y unattended-upgrades
sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
```
- Press `i` to enter insert mode
- Ensure these lines are uncommented:
  ```
  Unattended-Upgrade::Allowed-Origins {
      "${distro_id}:${distro_codename}";
      "${distro_id}:${distro_codename}-security";
      "${distro_id}ESMApps:${distro_codename}-apps-security";
      "${distro_id}ESM:${distro_codename}-infra-security";
  };
  ```
- Press `Esc`, type `:wq` and press Enter
- Enable automatic updates:
  ```
  sudo vim /etc/apt/apt.conf.d/20auto-upgrades
  ```
- Press `i` and add:
  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Unattended-Upgrade "1";
  ```
- Press `Esc`, type `:wq` and press Enter

### 9. Configure Logrotate for System Logs
```
sudo vim /etc/logrotate.d/custom-logs
```
- Press `i` and add:
  ```
  /var/log/syslog
  /var/log/auth.log
  {
      rotate 14
      daily
      missingok
      notifempty
      compress
      delaycompress
      sharedscripts
      postrotate
          reload rsyslog >/dev/null 2>&1 || true
      endscript
  }
  ```
- Press `Esc`, type `:wq` and press Enter

### 10. Set Up Swap Space (If Not Already Configured)
```
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
- Make it permanent:
  ```
  sudo vim /etc/fstab
  ```
- Press `i` and add this line:
  ```
  /swapfile none swap sw 0 0
  ```
- Press `Esc`, type `:wq` and press Enter

### 11. Install and Configure Fail2ban (Additional Security)
```
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```
- Press `i` to enter insert mode
- Find the `[sshd]` section and modify it:
  ```
  [sshd]
  enabled = true
  port = ssh
  filter = sshd
  logpath = /var/log/auth.log
  maxretry = 3
  bantime = 3600
  ```
- Press `Esc`, type `:wq` and press Enter
- Restart fail2ban:
  ```
  sudo systemctl restart fail2ban
  ```

## Part 4: Making Your Server Internet-Facing

### 1. Dynamic DNS Setup (If You Don't Have a Static IP)

#### 1.1. Install ddclient
```
sudo apt install -y ddclient
```

#### 1.2. Configure ddclient
```
sudo vim /etc/ddclient.conf
```
- Press `i` to enter insert mode
- For a service like No-IP, configure as follows:
  ```
  protocol=noip
  use=web
  server=dynupdate.no-ip.com
  login=your_noip_username
  password=your_noip_password
  your-domain.ddns.net
  ```
- For Cloudflare:
  ```
  protocol=cloudflare
  use=web
  server=api.cloudflare.com
  login=your_cloudflare_email
  password=your_global_api_key
  zone=yourdomain.com
  yourdomain.com
  ```
- Press `Esc`, type `:wq` and press Enter
- Restart ddclient:
  ```
  sudo systemctl restart ddclient
  ```

### 2. Port Forwarding on Your Router

- Access your router's admin interface (typically 192.168.1.1 or similar)
- Locate the Port Forwarding section (may be under Advanced Settings)
- Create new port forwarding rules:
  - Forward port 22 to your server's internal IP for SSH
  - Forward port 80 to your server's internal IP for HTTP (if applicable)
  - Forward port 443 to your server's internal IP for HTTPS (if applicable)
  - Forward any other service ports you plan to use

### 3. SSL Certificate Setup with Certbot

#### 3.1. Install Certbot
```
sudo apt install -y certbot
```

#### 3.2. For Nginx
```
sudo apt install -y python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

#### 3.3. For Apache
```
sudo apt install -y python3-certbot-apache
sudo certbot --apache -d yourdomain.com -d www.yourdomain.com
```

#### 3.4. Standalone (if no web server yet)
```
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com
```

### 4. Advanced Security for Internet-Facing Servers

#### 4.1. Install Fail2ban with Extended Jail Configuration
```
sudo vim /etc/fail2ban/jail.local
```
- Add these additional sections:
  ```
  [http-auth]
  enabled = true
  port = http,https
  filter = apache-auth
  logpath = /var/log/apache*/*access.log
  maxretry = 5
  bantime = 3600
  
  [nginx-http-auth]
  enabled = true
  filter = nginx-http-auth
  port = http,https
  logpath = /var/log/nginx/error.log
  maxretry = 5
  bantime = 3600
  ```
- Restart fail2ban:
  ```
  sudo systemctl restart fail2ban
  ```

#### 4.2. Install and Configure ModSecurity (Web Application Firewall)
For Nginx:
```
sudo apt install -y nginx-plus-module-modsecurity
```

For Apache:
```
sudo apt install -y libapache2-mod-security2
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo vim /etc/modsecurity/modsecurity.conf
```
- Change `SecRuleEngine DetectionOnly` to `SecRuleEngine On`
- Restart your web server

#### 4.3. Set Up Intrusion Detection with OSSEC
```
sudo apt install -y ossec-hids-server
```
- Follow the prompts to configure OSSEC
- Enable email alerts for security incidents

### 5. Server Monitoring and Notification Setup

#### 5.1. Install and Configure Monit for Service Monitoring
```
sudo apt install -y monit
sudo vim /etc/monit/monitrc
```
- Customize to monitor key services
- Enable email alerts
- Enable the web interface (if needed)
```
sudo systemctl restart monit
```

#### 5.2. Set Up Network Monitoring with Netdata
```
sudo apt install -y netdata
```
- Access at http://your-server-ip:19999

### 6. Verify External Accessibility

- Test SSH access from outside your network
- Test web services if applicable
- Use online port scanners to verify open ports
- Check if your domain name resolves correctly

### 7. Backup Strategy for Your Server

#### 7.1. Install Backup Tools
```
sudo apt install -y restic
```

#### 7.2. Set Up Regular Backups
```
# Initialize a restic repository
sudo restic init --repo /path/to/backup

# Create a backup script
sudo vim /usr/local/bin/backup.sh
```
- Press `i` and add:
  ```bash
  #!/bin/bash
  
  # Backup important directories
  restic -r /path/to/backup backup /etc /var/www /home
  
  # Keep only the last 7 daily, 4 weekly, and 6 monthly backups
  restic -r /path/to/backup forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6
  ```
- Make it executable:
  ```
  sudo chmod +x /usr/local/bin/backup.sh
  ```

#### 7.3. Schedule Regular Backups
```
sudo crontab -e
```
- Add:
  ```
  0 2 * * * /usr/local/bin/backup.sh
  ```

## Part 5: Verifying Installation

### 1. Check System Status
```
df -h                 # Verify disk space
free -m               # Verify memory usage
sudo systemctl status # Verify system services
ip a                  # Verify network configuration
```

### 2. Check Connectivity
```
ping -c 4 8.8.8.8     # Verify internet connectivity
ping -c 4 google.com  # Verify DNS resolution
nmap localhost        # Check open ports locally
```

### 3. Check External Connectivity (from a different network)
- Try connecting to your server via SSH using your domain name
- Try accessing any web services you've set up
- Use a port scanning tool to verify only intended ports are open

## Part 6: External Backup Storage Configuration

### 1. Install Required Packages
```
sudo apt install -y nfs-kernel-server samba rsync
```

### 2. Configure Storage Drives

#### 2.1. Identify Storage Drives
```
sudo fdisk -l
sudo lsblk
```
- Identify the drive(s) you want to use for backup storage

#### 2.2. Format the Drive (if needed)
```
sudo mkfs.ext4 /dev/sdX  # Replace X with your drive letter
```

#### 2.3. Create Mount Point and Mount the Drive
```
sudo mkdir -p /mnt/backup
sudo vim /etc/fstab
```
- Press `i` to enter insert mode
- Add this line (adjust according to your drive):
  ```
  /dev/sdX    /mnt/backup    ext4    defaults    0 2
  ```
- Press `Esc`, type `:wq` and press Enter
- Apply changes:
  ```
  sudo mount -a
  ```

#### 2.4. Set Appropriate Permissions
```
sudo chown -R nobody:nogroup /mnt/backup
sudo chmod -R 775 /mnt/backup
```

### 3. Configure NFS for Network Backup Access

#### 3.1. Configure NFS Exports
```
sudo vim /etc/exports
```
- Press `i` to enter insert mode
- Add this line (adjust according to your network):
  ```
  /mnt/backup    192.168.1.0/24(rw,sync,no_subtree_check)
  ```
- Press `Esc`, type `:wq` and press Enter
- Apply changes:
  ```
  sudo exportfs -a
  sudo systemctl restart nfs-kernel-server
  ```

#### 3.2. Configure Firewall for NFS
```
sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw status
```

### 4. Configure Samba for Windows/Mac Clients Access

#### 4.1. Configure Samba Shares
```
sudo vim /etc/samba/smb.conf
```
- Press `i` to enter insert mode
- Add at the end:
  ```
  [Backup]
    path = /mnt/backup
    browseable = yes
    read only = no
    create mask = 0775
    directory mask = 0775
    valid users = @backup
  ```
- Press `Esc`, type `:wq` and press Enter

#### 4.2. Create Samba User Group and User
```
sudo addgroup backup
sudo smbpasswd -a your_username
sudo usermod -aG backup your_username
```

#### 4.3. Restart Samba and Configure Firewall
```
sudo systemctl restart smbd
sudo ufw allow samba
```

### 5. Set Up Automatic Backup Rotation

#### 5.1. Create Backup Script
```
sudo vim /usr/local/bin/backup-rotate.sh
```
- Press `i` to enter insert mode
- Add:
  ```bash
  #!/bin/bash
  
  # Configuration
  BACKUP_DIR="/mnt/backup"
  MAX_DAILY=7
  MAX_WEEKLY=4
  MAX_MONTHLY=6
  
  # Create timestamp
  DATE=$(date +%Y-%m-%d)
  DAY=$(date +%d)
  WEEKDAY=$(date +%u)
  MONTH=$(date +%m)
  
  # Create backup directories if they don't exist
  mkdir -p ${BACKUP_DIR}/daily
  mkdir -p ${BACKUP_DIR}/weekly
  mkdir -p ${BACKUP_DIR}/monthly
  
  # Create daily backup
  tar -czf ${BACKUP_DIR}/daily/backup-${DATE}.tar.gz -C / home etc var/www
  
  # Create weekly backup (on Sundays)
  if [ "$WEEKDAY" = "7" ]; then
    cp ${BACKUP_DIR}/daily/backup-${DATE}.tar.gz ${BACKUP_DIR}/weekly/
  fi
  
  # Create monthly backup (on the 1st)
  if [ "$DAY" = "01" ]; then
    cp ${BACKUP_DIR}/daily/backup-${DATE}.tar.gz ${BACKUP_DIR}/monthly/
  fi
  
  # Rotate backups - remove old ones
  # Delete daily backups older than MAX_DAILY days
  find ${BACKUP_DIR}/daily -type f -name "*.tar.gz" -mtime +${MAX_DAILY} -delete
  
  # Keep only MAX_WEEKLY weekly backups
  cd ${BACKUP_DIR}/weekly
  ls -t *.tar.gz 2>/dev/null | tail -n +$((MAX_WEEKLY+1)) | xargs -r rm
  
  # Keep only MAX_MONTHLY monthly backups
  cd ${BACKUP_DIR}/monthly
  ls -t *.tar.gz 2>/dev/null | tail -n +$((MAX_MONTHLY+1)) | xargs -r rm
  ```
- Press `Esc`, type `:wq` and press Enter
- Make it executable:
  ```
  sudo chmod +x /usr/local/bin/backup-rotate.sh
  ```

#### 5.2. Schedule with Cron
```
sudo crontab -e
```
- Add:
  ```
  0 1 * * * /usr/local/bin/backup-rotate.sh
  ```

### 6. Monitor Disk Usage and Health

#### 6.1. Install Monitoring Tools
```
sudo apt install -y smartmontools hdparm
```

#### 6.2. Check Disk Health
```
sudo smartctl -a /dev/sdX  # Replace X with your drive letter
```

#### 6.3. Set Up Email Alerts for Low Disk Space
```
sudo vim /usr/local/bin/disk-space-check.sh
```
- Press `i` to enter insert mode
- Add:
  ```bash
  #!/bin/bash
  
  THRESHOLD=90
  BACKUP_DRIVE="/mnt/backup"
  EMAIL="your_email@example.com"
  
  USAGE=$(df -h ${BACKUP_DRIVE} | grep ${BACKUP_DRIVE} | awk '{print $5}' | cut -d'%' -f1)
  
  if [ ${USAGE} -gt ${THRESHOLD} ]; then
    echo "ALERT: Backup drive is ${USAGE}% full" | mail -s "Disk Space Alert" ${EMAIL}
  fi
  ```
- Press `Esc`, type `:wq` and press Enter
- Make it executable:
  ```
  sudo chmod +x /usr/local/bin/disk-space-check.sh
  ```
- Add to cron:
  ```
  sudo crontab -e
  ```
- Add:
  ```
  0 7 * * * /usr/local/bin/disk-space-check.sh
  ```

## Part 7: Kubernetes Cluster Setup

### 1. Prepare All Nodes

#### 1.1. Install Required Packages on All Nodes (Tower and Raspberry Pis)
Before proceeding, ensure all Raspberry Pis have Ubuntu 64-bit installed and are accessible on your network.

On each node (including the tower and all Raspberry Pis), run:
```
sudo apt update
sudo apt install -y openssh-server
```

#### 1.2. Enable cgroup Memory on Raspberry Pis
SSH into each Raspberry Pi and modify the boot parameters:
```
sudo vim /boot/firmware/cmdline.txt
```
- Add the following to the end of the line:
  ```
  cgroup_enable=memory cgroup_memory=1
  ```
- Save, exit, and reboot:
  ```
  sudo reboot
  ```

### 2. Set Up MicroK8s on Tower (Control Plane)

#### 2.1. Install MicroK8s
On your tower server:
```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
```

#### 2.2. Check Status and Wait for Ready
```
microk8s status --wait-ready
```

#### 2.3. Enable Essential Add-ons
```
microk8s enable dns dashboard storage metallb:192.168.1.100-192.168.1.110
```
Note: Adjust the MetalLB IP range based on your network.

#### 2.4. Create an Alias for kubectl (Optional)
```
echo "alias kubectl='microk8s kubectl'" >> ~/.bashrc
source ~/.bashrc
```

### 3. Join Raspberry Pis to the Cluster

#### 3.1. Generate Join Token on Tower
```
microk8s add-node
```
- This will output a join command with a token

#### 3.2. Join Each Raspberry Pi to the Cluster
SSH into your Raspberry Pi 5 and run:
```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
```

Then run the join command from the tower:
```
microk8s join <IP_OF_TOWER>:25000/<TOKEN>
```

Repeat for each Raspberry Pi 4.

#### 3.3. Verify Nodes
On the tower:
```
microk8s kubectl get nodes
```

### 4. Configure Kubernetes Storage for Backup Solution

#### 4.1. Create a Storage Class for Backup Volume
```
cat <<EOF | microk8s kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: backup-storage
provisioner: microk8s.io/hostpath
reclaimPolicy: Retain
volumeBindingMode: Immediate
EOF
```

#### 4.2. Create a Persistent Volume Claim
```
cat <<EOF | microk8s kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: backup-storage
  resources:
    requests:
      storage: 500Gi
EOF
```

### 5. Deploy a Simple Backup Pod (Example)

```
cat <<EOF | microk8s kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backup-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backup-server
  template:
    metadata:
      labels:
        app: backup-server
    spec:
      containers:
      - name: backup-server
        image: ubuntu:latest
        command:
        - "/bin/bash"
        - "-c"
        - "apt-get update && apt-get install -y rsync openssh-server && mkdir -p /backup/data && chmod 777 /backup/data && tail -f /dev/null"
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
EOF
```

#### 5.1. Expose the Backup Server
```
cat <<EOF | microk8s kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: backup-server
spec:
  selector:
    app: backup-server
  ports:
  - port: 22
    targetPort: 22
  type: LoadBalancer
EOF
```

### 6. Deploy the Kubernetes Dashboard UI (Optional)

#### 6.1. Access the Dashboard
```
microk8s kubectl describe secret -n kube-system microk8s-dashboard-token
```
- Note the token value

#### 6.2. Start the Kubernetes Dashboard Proxy
```
microk8s dashboard-proxy
```
- Access the dashboard URL provided
- Use the token from the previous step to log in

### 7. Set Up Monitoring for the Cluster

#### 7.1. Enable Prometheus and Grafana
```
microk8s enable prometheus
```

#### 7.2. Access Grafana
```
microk8s kubectl port-forward -n monitoring service/grafana 3000:3000
```
- Access Grafana at http://localhost:3000
- Default credentials: admin/admin

### 8. Maintaining Your Kubernetes Cluster

#### 8.1. Update MicroK8s
```
sudo snap refresh microk8s
```

#### 8.2. Backup Kubernetes Configuration
```
microk8s kubectl get all --all-namespaces -o yaml > k8s-backup-$(date +%Y%m%d).yaml
```

#### 8.3. Using kubectl to Manage the Cluster
```
microk8s kubectl get nodes
microk8s kubectl get pods --all-namespaces
microk8s kubectl get services --all-namespaces
```

### 9. Troubleshooting

#### 9.1. Check Node Status
```
microk8s kubectl describe node <node-name>
```

#### 9.2. Check Pod Logs
```
microk8s kubectl logs <pod-name>
```

#### 9.3. Restart MicroK8s if Needed
```
microk8s stop
microk8s start
```

## Part 8: Deploying Nextcloud on Kubernetes

Nextcloud is a self-hosted productivity platform that provides file storage, synchronization, sharing, and collaboration features. This section covers how to deploy Nextcloud on one of your Kubernetes nodes.

### 1. Install Helm (if not already installed)

```bash
# On your tower server
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 2. Add the Nextcloud Helm Repository

```bash
helm repo add nextcloud https://nextcloud.github.io/helm/
helm repo update
```

### 3. Create a Values File for Nextcloud

Create a file named `nextcloud-values.yaml` with the following content:

```yaml
# Basic Nextcloud Configuration
nextcloud:
  host: nextcloud.local  # Change to your domain or local network name
  username: admin        # Initial admin username
  password: "StrongPasswordHere"  # Initial admin password
  configs:
    custom.config.php: |-
      <?php
      $CONFIG = array (
        'trusted_domains' => 
        array (
          0 => 'nextcloud.local',  # Your domain or IP
          1 => 'localhost',
          2 => '192.168.1.100',   # Your server IP
        ),
        'overwriteprotocol' => 'https',
        'default_phone_region' => 'US',  # Change to your region
      );

# Persistence configuration
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 10Gi
  storageClass: "microk8s-hostpath"  # Use MicroK8s storage class

# Database - using MariaDB
mariadb:
  enabled: true
  auth:
    database: nextcloud
    username: nextcloud
    password: "NextcloudDBPassword"  # Change this
    rootPassword: "NextcloudDBRootPassword"  # Change this
  primary:
    persistence:
      enabled: true
      storageClass: "microk8s-hostpath"
      size: 8Gi

# Redis for performance
redis:
  enabled: true
  auth:
    password: "RedisStrongPassword"  # Change this
  master:
    persistence:
      enabled: true
      storageClass: "microk8s-hostpath"
      size: 2Gi

# Ingress configuration (optional, if you want external access)
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: 4G
    nginx.ingress.kubernetes.io/proxy-buffering: "off"
    nginx.ingress.kubernetes.io/proxy-request-buffering: "off"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # If using cert-manager
  tls:
    - hosts:
        - nextcloud.local  # Your domain
      secretName: nextcloud-tls

# Resource limits
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 2000m
    memory: 2Gi
```

### 4. Create a Namespace for Nextcloud

```bash
microk8s kubectl create namespace nextcloud
```

### 5. Deploy Nextcloud Using Helm

```bash
helm upgrade --install nextcloud nextcloud/nextcloud \
  --namespace nextcloud \
  --values nextcloud-values.yaml
```

### 6. Monitor the Deployment

```bash
microk8s kubectl -n nextcloud get pods
microk8s kubectl -n nextcloud get services
```

### 7. Get the Nextcloud URL

If using LoadBalancer or NodePort:
```bash
microk8s kubectl -n nextcloud get service nextcloud
```

If using Ingress:
```bash
microk8s kubectl -n nextcloud get ingress
```

### 8. Configure DNS or /etc/hosts (for local access)

Add an entry to your local computer's `/etc/hosts` file if you're using a local domain:
```
192.168.1.100  nextcloud.local  # Replace with your server's IP
```

### 9. Post-Installation Configuration

#### 9.1. Set Up External Storage (Optional)
After logging in to Nextcloud:
1. Go to Apps and install the "External Storage Support" app
2. Go to Settings > Administration > External Storages
3. Add your backup storage location as an external storage

#### 9.2. Configure Automatic Backups
Create a Kubernetes CronJob to back up Nextcloud:

```bash
cat <<EOF | microk8s kubectl apply -f -
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nextcloud-backup
  namespace: nextcloud
spec:
  schedule: "0 2 * * *"  # Run at 2 AM every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: bitnami/kubectl
            command:
            - /bin/sh
            - -c
            - |
              kubectl -n nextcloud exec \$(kubectl -n nextcloud get pods -l app.kubernetes.io/name=nextcloud -o jsonpath='{.items[0].metadata.name}') -- php occ maintenance:mode --on
              kubectl -n nextcloud exec \$(kubectl -n nextcloud get pods -l app.kubernetes.io/instance=nextcloud-mariadb -o jsonpath='{.items[0].metadata.name}') -- mysqldump -u root -p\${MYSQL_ROOT_PASSWORD} --single-transaction nextcloud > /backup/nextcloud-db-\$(date +%Y%m%d).sql
              kubectl -n nextcloud exec \$(kubectl -n nextcloud get pods -l app.kubernetes.io/name=nextcloud -o jsonpath='{.items[0].metadata.name}') -- tar -czf /backup/nextcloud-data-\$(date +%Y%m%d).tar.gz -C /var/www/html/data .
              kubectl -n nextcloud exec \$(kubectl -n nextcloud get pods -l app.kubernetes.io/name=nextcloud -o jsonpath='{.items[0].metadata.name}') -- php occ maintenance:mode --off
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: nextcloud-mariadb
                  key: mariadb-root-password
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: backup-pvc
EOF
```

### 10. Troubleshooting

#### 10.1. Check Pod Logs
```bash
microk8s kubectl -n nextcloud logs -l app.kubernetes.io/name=nextcloud
```

#### 10.2. Check Database Logs
```bash
microk8s kubectl -n nextcloud logs -l app.kubernetes.io/name=mariadb
```

#### 10.3. Restart Nextcloud Pod if Needed
```bash
microk8s kubectl -n nextcloud rollout restart deployment nextcloud
```

#### 10.4. Accessing Nextcloud Console for Maintenance
```bash
microk8s kubectl -n nextcloud exec -it $(microk8s kubectl -n nextcloud get pods -l app.kubernetes.io/name=nextcloud -o jsonpath='{.items[0].metadata.name}') -- bash
```

From inside the pod, you can run Nextcloud OCC commands:
```bash
cd /var/www/html
php occ maintenance:mode --on|--off
php occ db:add-missing-indices
```

### 11. Connecting with Mobile and Desktop Clients

Once your Nextcloud instance is running, you can download and install clients to access your files from:

1. **Desktop Clients**: Available for Windows, macOS, and Linux from the Nextcloud website
2. **Mobile Apps**: Available for Android and iOS from their respective app stores

When setting up clients, use the URL of your Nextcloud instance (e.g., `https://nextcloud.local` or the IP address with port if using NodePort).

### 12. Integrating Nextcloud with Your Backup System

To utilize your backup system with Nextcloud:

#### 12.1. Mount the NFS Share Directly in Nextcloud

Edit your `nextcloud-values.yaml` file to add an additional volume:

```yaml
# Add this under the Nextcloud configuration
extraVolumes:
  - name: backup-nfs
    nfs:
      server: 192.168.1.100  # Your server IP
      path: /mnt/backup

extraVolumeMounts:
  - name: backup-nfs
    mountPath: /backups
    readOnly: false
```

Apply the changes:
```bash
helm upgrade nextcloud nextcloud/nextcloud \
  --namespace nextcloud \
  --values nextcloud-values.yaml
```

#### 12.2. Configure Nextcloud to Use the Backup Storage

1. Log in to your Nextcloud web interface as admin
2. Go to Settings > Administration > External Storages
3. Add a new external storage with type "Local" pointing to `/backups`
4. Set appropriate permissions and user access

Your Nextcloud instance is now deployed on Kubernetes with persistent storage, database, and caching, ready to serve as your self-hosted file storage and collaboration platform. The setup integrates with your existing backup infrastructure, providing a complete solution for secure data management.

## Conclusion

Congratulations! You have successfully:

1. Replaced Windows with Ubuntu Server on your tower
2. Configured external backup storage
3. Set up a Kubernetes cluster with your tower as the control plane
4. Added three Raspberry Pi devices as worker nodes
5. Deployed Nextcloud for file storage and collaboration

This infrastructure provides a robust, secure, and flexible platform for storing, backing up, and accessing your data. You can expand it by adding more nodes or deploying additional applications on your Kubernetes cluster as needed.

For ongoing maintenance:

- Regularly update your Ubuntu Server with `sudo apt update && sudo apt upgrade`
- Update Kubernetes with `sudo snap refresh microk8s`
- Monitor disk usage and system health
- Review logs periodically for any issues
- Test your backups regularly to ensure they're working correctly

Remember to keep your system secure by:

- Regularly changing passwords
- Keeping software up to date
- Monitoring for suspicious activity
- Maintaining proper firewall rules
- Using encryption for sensitive data

With this setup, you've created a powerful, self-hosted infrastructure that gives you complete control over your data and applications.