# Terraform Proxmox VM Automation - Complete Guide

## Overview
This guide shows how to create multiple VMs in Proxmox using Terraform with unique DHCP IP addresses.

## Prerequisites
- Proxmox server running
- API token configured
- Template VM with cloud-init support
- DHCP server on your network

## Key Configuration Files

### 1. variables.tf
```hcl
variable "pm_api_url" {
  description = "Proxmox API URL"
  type        = string
  default     = "https://pve.devopsmaffia.net"
}

variable "pm_api_token_id" {
  description = "Proxmox API Token ID"
  type        = string
  default     = "root@pam!terraform"
}

variable "pm_api_token_secret" {
  description = "Proxmox API Token Secret"
  type        = string
  sensitive   = true
}

variable "pm_password" {
  description = "Root password for VM access"
  type        = string
  sensitive   = true
}

variable "pm_node" {
  description = "Proxmox Node name"
  type        = string
  default     = "devopsmaffia"  # Replace with your node name
}

variable "template_id" {
  description = "Template VM ID to clone"
  type        = number
  default     = 8888  # Replace with your template VM ID
}
```

### 2. terraform.tfvars
```hcl
pm_api_token_secret = "77f55d1d-d5ce-4cb8-a586-942a10f322de"  # Your actual token
pm_password = "your-root-password-here"  # Root password for template access
```

### 3. main.tf
```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = ">=0.60.0"
    }
  }
}

provider "proxmox" {
  endpoint = var.pm_api_url
  api_token = "${var.pm_api_token_id}=${var.pm_api_token_secret}"
  insecure = true
}

resource "proxmox_virtual_environment_vm" "automation_vm" {
  count     = 5
  name      = "automation-vm-${count.index + 1}"
  node_name = var.pm_node
  vm_id     = 900 + count.index

  clone {
    vm_id = var.template_id
    full  = true
  }

  cpu {
    cores = 2
    sockets = 1
  }

  memory {
    dedicated = 4096
  }

  disk {
    datastore_id = "vms-data-dir"  # Your storage name
    interface    = "scsi0"
    iothread     = true
    discard      = "on"
    size         = 32
  }

  network_device {
    bridge = "vmbr1"  # Your bridge name
    model  = "virtio"
  }

  # CRITICAL: Specify datastore_id to avoid local-lvm error
  initialization {
    datastore_id = "vms-data-dir"  # Must match your storage
    ip_config {
      ipv4 {
        address = "dhcp"
      }
    }
  }

  boot_order = ["scsi0", "net0"]
  agent {
    enabled = true
  }
  
  started = false
}
```

## Template Preparation

### 1. Install Cloud-Init in Template VM
```bash
sudo apt update
sudo apt install cloud-init qemu-guest-agent

# Configure cloud-init for Proxmox
sudo nano /etc/cloud/cloud.cfg.d/99-pve.cfg
```

Add this content:
```yaml
datasource_list: [ConfigDrive, NoCloud]
```

### 2. Clean Template Before Converting
```bash
# Clean machine ID (CRITICAL for unique DHCP IPs)
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id

# Remove SSH host keys
sudo rm /etc/ssh/ssh_host_*

# Clear history
history -c
sudo rm -f /root/.bash_history
sudo rm -f /home/ubuntu/.bash_history

# Clean packages
sudo apt clean

# Shutdown
sudo shutdown -h now
```

### 3. Convert to Template
```bash
curl -k -H 'Authorization: PVEAPIToken=root@pam!terraform=YOUR_TOKEN' \
  -X POST https://pve.devopsmaffia.net/api2/json/nodes/devopsmaffia/qemu/8888/template
```

## Deployment Commands

### 1. Initialize Terraform
```bash
terraform init
```

### 2. Plan and Apply
```bash
terraform plan
terraform apply
```

### 3. Check VM IPs
```bash
curl -k -H 'Authorization: PVEAPIToken=root@pam!terraform=YOUR_TOKEN' \
  https://pve.devopsmaffia.net/api2/json/nodes/devopsmaffia/qemu/900/agent/network-get-interfaces
```

## Common Issues and Solutions

### Issue 1: "storage 'local-lvm' does not exist"
**Solution:** Add `datastore_id = "vms-data-dir"` to the initialization block

### Issue 2: All VMs get same IP address
**Solution:** Clean machine-id in template before converting:
```bash
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
```

### Issue 3: Authentication failures
**Solution:** Verify API token and check token permissions in Proxmox

### Issue 4: Network bridge errors
**Solution:** Check actual bridge name with:
```bash
curl -k -H 'Authorization: PVEAPIToken=root@pam!terraform=YOUR_TOKEN' \
  https://pve.devopsmaffia.net/api2/json/nodes/devopsmaffia/network
```

## Key Configuration Parameters

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `datastore_id` | Storage location | `"vms-data-dir"` |
| `node_name` | Proxmox node | `"devopsmaffia"` |
| `bridge` | Network bridge | `"vmbr1"` |
| `template_id` | Source template | `8888` |
| `vm_id` | Starting VM ID | `900` |

## Results
- Creates 5 VMs with IDs 900-904
- Each VM gets unique DHCP IP address
- VMs named: automation-vm-1, automation-vm-2, etc.
- All VMs start in stopped state for verification

## Verification
After deployment, each VM should have:
- Unique MAC address
- Unique IP from DHCP server
- Proper cloud-init configuration
- SSH access enabled