# Linux Homelab Setup

This repository documents my Linux homelab setup built primarily for self-taught learning, project development, cybersecurity experimentation, and preparation for the **Red Hat Certified Systems Administrator (RHCSA)** exam.


![linux_homelab](https://github.com/user-attachments/assets/ea7842d8-af6b-4cb4-9043-58cb279b6121)

## 📚 Table of Contents

- [📌 Homelab Overview](#-homelab-overview)
- [💻 Hardware](#-hardware)
- [🖥️ Operating Systems and VMs](#️-operating-systems-and-vms)
  - [Linux VMs](#linux-vms)
  - [Cybersecurity VMs](#cybersecurity-vms)
  - [Windows VM](#windows-vm)
- [🚀 VirtualBox Installation (on Fedora 41)](#-virtualbox-installation-on-fedora-41)
- [📈 Monitoring Setup: Grafana](#-monitoring-setup-grafana)
- [🛡️ Security and Hardening](#️-security-and-hardening)
- [🔄 VM Management](#-vm-management)
- [📚 RHCSA Preparation](#-rhcsa-preparation)
- [🔐 Cybersecurity Practice](#-cybersecurity-practice)
- [📂 Scripts and Automation](#-scripts-and-automation)
  - [Bash](#bash)
  - [Python](#python)
- [🧪 Terraform Lab for IaC Experimentation and GitOps Integration](#-terraform-lab-for-iac-experimentation-and-gitops-integration)
  - [AWS](#aws)
  - [Azure](#azure)
  - [Linode](#linode)
- [🖼️ Architecture Diagram](#️-architecture-diagram)
- [📸 Screenshots](#-screenshots)
- [📕 Troubleshooting](#-troubleshooting)
- [🔗 Resource Links and Learning Path](#-resource-links-and-learning-path)
- [🚨 Continuous Improvement](#-continuous-improvement)
- [⚙️ Skills Highlighted](#️-skills-highlighted)



## 📌 Homelab Overview

This section provides a concise overview of the foundational components that make up the homelab environment. From the host operating system to virtualization tools and monitoring solutions, this snapshot outlines the essential software and system architecture used as the basis for all VM configurations, testing environments, and learning objectives.

-   **Laptop Host OS**: Fedora 41 (Daily Driver)
    
-   **Virtualization Software**: VirtualBox
    
-   **Monitoring Solution**: Grafana (previously Cockpit)
    

## 💻 Hardware

My homelab runs on the following hardware configuration:

-   **Laptop**: Intel Core i7, 64 GB RAM, 1 TB NVMe SSD
    
-   **Storage**: External HDD 4 TB (Snapshots, ISO files, and backups)
    
-   **Network**: Gigabit Ethernet
    

## 🖥️ Operating Systems and VMs

All operating systems listed below run as virtual machines managed through VirtualBox:

### Linux VMs:

-   **Ubuntu Desktop** (2 vCPUs, 4 GB RAM, 40 GB Disk)
    
-   **Ubuntu Server** (2 vCPUs, 4 GB RAM, 40 GB Disk)
    
-   **Debian Server** (2 vCPUs, 4 GB RAM, 40 GB Disk)
    
-   **Red Hat Enterprise Linux (RHEL)** (4 vCPUs, 8 GB RAM, 50 GB Disk)
    
-   **Rocky Linux** (4 vCPUs, 8 GB RAM, 50 GB Disk)
    

### Cybersecurity VMs:

-   **Parrot OS** (4 vCPUs, 8 GB RAM, 50 GB Disk)
    
-   **Black Arch Linux** (4 vCPUs, 8 GB RAM, 50 GB Disk)
    

### Windows VM:

-   **Windows Server 2022** (4 vCPUs, 8 GB RAM, 60 GB Disk)
    

## 🚀 VirtualBox Installation (on Fedora 41)

```bash
sudo dnf install @development-tools kernel-headers kernel-devel dkms
sudo dnf install VirtualBox
sudo usermod -aG vboxusers $USER
sudo /sbin/vboxconfig

```

## 📈 Monitoring Setup: Grafana

```bash
sudo dnf install grafana
sudo systemctl enable --now grafana-server

```

[http://localhost:3000](http://localhost:3000/)

-   Default login: `admin/admin`
    
-   Change credentials immediately after logging in.
    

```bash
sudo dnf install prometheus
sudo systemctl enable --now prometheus

```

-   Prometheus URL in Grafana: `http://localhost:9090`
    

## 🛡️ Security and Hardening

```bash
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --add-port=3000/tcp --permanent
sudo firewall-cmd --reload

```

-   SELinux in enforcing mode
    
-   SSH key-based authentication
    
-   Root login disabled
    

## 🔄 VM Management

```bash
rsync -avz --delete ~/VirtualBox\ VMs /mnt/backup/homelab-vms/

```

## 📚 RHCSA Preparation


Using Red Hat Enterprise Linux and Rocky Linux VMs extensively for:
- User/group management (`useradd`, `groupadd`)
- SELinux configuration (`setenforce`, policy adjustments)
- Service management (`systemctl`, `journalctl`)
- Package management (`dnf`/`yum`)
- Network and firewall configuration (`nmcli`, `firewalld`)
- LVM management (create logical volumes, extend volumes)
  ```bash
  sudo lvcreate -L 10G -n mylv myvg
  sudo mkfs.ext4 /dev/myvg/mylv
  ```
- Creating and managing containers with Podman
  ```bash
  podman run -d --name nginx -p 8080:80 nginx
  
## 🔐 Cybersecurity Practice

-   Parrot OS and BlackArch Linux
    
-   Metasploit, Burp Suite, Wireshark
    
-   Network analysis, Pen testing
    

## 📂 Scripts and Automation

### Bash

```bash
#!/bin/bash
VM_LIST=("UbuntuServer" "RHEL" "RockyLinux")
for vm in "${VM_LIST[@]}"; do
  timestamp=$(date +%Y%m%d_%H%M%S)
  VBoxManage snapshot "$vm" take "snap_$timestamp" --pause
  echo "Snapshot taken for $vm at $timestamp"
done

```

### Python

```python
import subprocess
import datetime
vms = ["UbuntuServer", "RHEL", "RockyLinux"]
timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
for vm in vms:
    subprocess.run(["VBoxManage", "snapshot", vm, "take", f"snap_{timestamp}", "--pause"])
    print(f"Snapshot taken for {vm} at {timestamp}")

```

## 🧪 Terraform Lab for IaC Experimentation and GitOps Integration

### AWS

```hcl
provider "aws" {
  region = "us-east-1"
}
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "HomelabInstance"
  }
}

```

### Azure

```hcl
provider "azurerm" {
  features {}
}
resource "azurerm_resource_group" "lab" {
  name     = "homelab-resources"
  location = "East US"
}
resource "azurerm_linux_virtual_machine" "vm" {
  name                = "homelabVM"
  resource_group_name = azurerm_resource_group.lab.name
  location            = azurerm_resource_group.lab.location
  size                = "Standard_B1s"
  admin_username      = "adminuser"
  disable_password_authentication = true
  network_interface_ids = []
  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "20_04-lts"
    version   = "latest"
  }
}

```

### Linode

```hcl
provider "linode" {
  token = var.linode_token
}
resource "linode_instance" "peppermint" {
  label     = "peppermint-helpdesk"
  region    = "us-east"
  type      = "g6-standard-1"
  image     = "linode/ubuntu22.04"
  root_pass = var.root_password
  authorized_keys = [file("~/.ssh/id_rsa.pub")]
  tags = ["peppermint", "helpdesk"]
}

```

```
- name: Terraform Apply
  run: terraform apply -auto-approve
  env:
    TF_VAR_linode_token: ${{ secrets.LINODE_TOKEN }}
    TF_VAR_root_password: ${{ secrets.ROOT_PASSWORD }}

```




## 🖼️ Architecture Diagram

**Text-based Architecture Diagram Outline:**

```
Laptop (Fedora 41)
│
├── VirtualBox Manager
│   ├── Linux VMs (Ubuntu, Debian, RHEL, Rocky)
│   ├── Cybersecurity VMs (Parrot OS, BlackArch Linux)
│   └── Windows Server 2022 VM
│
├── Grafana & Prometheus
│   └── Monitoring and metrics collection
│
└── External HDD (4 TB)
    └── Backups, Snapshots, ISOs
```

## 📸 Screenshots

Screenshots of Grafana dashboards and VM management interfaces are provided in the `screenshots/` directory.

## 📕 Troubleshooting

This section outlines common troubleshooting practices and examples to help maintain a healthy and functional homelab.


### VirtualBox Issues
- **Kernel Driver Not Installed**:
  - Run: `sudo /sbin/vboxconfig`
  - Ensure DKMS and kernel-devel are installed after updating:
    ```bash
    sudo dnf update -y

    sudo dnf install dkms kernel-devel kernel-headers -y
    ```

- **VM Network Not Working**:
  - Check if VirtualBox network adapter is attached.
  - Restart NetworkManager: `sudo systemctl restart NetworkManager`

- **VMs Not Starting After Kernel Update**:
  - Rebuild kernel modules: `sudo dnf update -y
sudo akmods --force
sudo dracut --force
sudo reboot`

### Fedora/Grafana Issues
- **Grafana Port Inaccessible**:
  - Verify firewall rule: `sudo firewall-cmd --list-all`
  - Add rule: `sudo firewall-cmd --add-port=3000/tcp --permanent && sudo firewall-cmd --reload`

- **Prometheus Not Starting**:
  - Check service logs: `sudo journalctl -u prometheus`
  - Verify config: `/etc/prometheus/prometheus.yml`

### SELinux & Permission Errors
- **Access Denied on Services**:
  - Use: `sudo ausearch -m avc -ts recent`
  - Allow via policy: `sudo setsebool -P httpd_can_network_connect 1`

### Disk/Backup Issues
- **External HDD Not Mounting**:
  - Verify with `lsblk` or `blkid`
  - Mount manually: `sudo mount /dev/sdX1 /mnt/backup`

- **`rsync` Fails Due to Permission Denied**:
  - Run with elevated privileges: `sudo rsync -avz --delete ~/VirtualBox\ VMs /mnt/backup/homelab-vms/`

### VM Performance Optimization
- Use VirtIO drivers where possible
- Enable 2D/3D acceleration (where supported)
- Allocate sufficient memory and disk I/O bandwidth

More detailed examples and logs are available in `TROUBLESHOOTING.md`.

## 🔗 Resource Links and Learning Path

### Distros and Documentation:
- [Fedora Documentation](https://docs.fedoraproject.org/en-US/docs/)
- [Ubuntu Documentation](https://ubuntu.com/server/docs)
- [Debian Documentation](https://www.debian.org/doc/)
- [Red Hat Enterprise Linux Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux)
- [Rocky Linux Documentation](https://docs.rockylinux.org/)
- [Parrot OS Documentation](https://parrotsec.org/docs/)
- [BlackArch Linux Documentation](https://blackarch.org/guide.html)

### Recommended Learning Path:
1. **Fedora and Red Hat Basics**: Start with system administration, package management, SELinux, and basic networking.
2. **Ubuntu and Debian Server Administration**: Practice server installations, web server setups (Apache/Nginx), database management, and scripting.
3. **Advanced RHCSA Preparation (RHEL/Rocky Linux)**: Deep dive into advanced concepts like LVM, container management with Podman, firewall and SELinux policy management.
4. **Cybersecurity Methodologies (Parrot OS/BlackArch Linux)**: Engage with ethical hacking, penetration testing exercises, advanced vulnerability scanning, and exploit development.

## 🚨 Continuous Improvement

Plans for future homelab enhancements:
- Implement Infrastructure as Code (IaC) with Ansible and Terraform
- Container orchestration with Kubernetes
- Advanced network segmentation and security practices
- Preparation for the **Red Hat Certified Engineer (RHCE)** exam

## ⚙️ Skills Highlighted

- Linux administration and security hardening
- Virtualization and VM lifecycle management
- Advanced monitoring and observability with Grafana
- Infrastructure automation and scripting
- Cybersecurity practices and tools
- Documentation and version control best practices

---

**Feedback and suggestions are welcome!** 🌟

