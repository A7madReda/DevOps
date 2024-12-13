# Proxmox VE Installation Guide on Debian


### Pre-Installation Steps

1. Update your system:
```bash
apt update && apt upgrade -y
```

2. Install required packages:
```bash
apt install sudo curl wget gnupg2 software-properties-common apt-transport-https ca-certificates -y
```

3. Add Proxmox VE repository:
```bash
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
```

4. Add Proxmox VE repository key:
```bash
wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
```

## Installing Proxmox VE

1. Update repository information:
```bash
apt update
```

2. Install Proxmox VE packages:
```bash
apt install proxmox-ve postfix open-iscsi -y
```

During the installation:
- For postfix configuration, select "Internet Site"
- Enter your system's mail name when prompted

3. Reboot your system:
```bash
reboot
```

## Post-Installation Configuration

### Network Configuration

1. Edit the network configuration file:
```bash
nano /etc/network/interfaces
```

2. Add your network configuration (example):
```
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.1
        bridge_ports eno1
        bridge_stp off
        bridge_fd 0
```

Replace:
- `192.168.1.100/24` with your desired IP address
- `192.168.1.1` with your gateway address
- `eno1` with your actual network interface name

3. Restart networking:
```bash
systemctl restart networking
```

### Hostname Resolution

1. Edit hosts file:
```bash
nano /etc/hosts
```

2. Add your server entry:
```
192.168.1.100    your-hostname
```

Replace with your actual IP and hostname.

## Web Interface Setup

### Creating Admin User

1. Create a new user:
```bash
pveum useradd your-username@pam
```

2. Set password:
```bash
pveum passwd your-username@pam
```

3. Create admin group and add user:
```bash
pveum groupadd admins
pveum usermod your-username@pam -group admins
```

### Accessing Web Interface

Access the Proxmox VE web interface at:
```
https://YOUR-SERVER-IP:8006
```

Note: You'll receive a certificate warning on first access due to the self-signed certificate.

## Cluster Configuration

### Creating a New Cluster

To initialize a single-node cluster:
```bash
pvecm create CLUSTERNAME
```

### Joining Existing Cluster

1. On existing node, generate join token:
```bash
pvecm generatejointoken
```

2. On new node, join cluster:
```bash
pvecm add EXISTING_NODE_IP --token TOKEN_STRING
```

## Common Issues and Troubleshooting

### Unable to Start Cluster Service

If you see error about hostname resolution:
1. Check /etc/hosts has correct entry
2. Ensure hostname resolves to non-loopback IP
3. Restart service:
```bash
systemctl restart pve-cluster
```

### Enterprise Repository Warning

To remove enterprise repository warning:
```bash
rm /etc/apt/sources.list.d/pve-enterprise.list
```

### Network Bridge Issues

If VM networking doesn't work:
1. Verify bridge configuration in /etc/network/interfaces
2. Ensure primary interface is added to bridge
3. Check network interface name matches system

## Security Recommendations

1. Change root password:
```bash
passwd
```

2. Enable firewall:
```bash
apt install proxmox-firewall
```

3. Configure SSH key authentication instead of password

## Maintenance

### Regular Updates

Update system and Proxmox VE:
```bash
apt update
apt upgrade
```

### Backup Configuration

Backup important configuration files:
- /etc/pve/
- /etc/network/interfaces
- /etc/hosts

## Additional Resources

- [Official Proxmox Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [Proxmox Forum](https://forum.proxmox.com)
- [Proxmox Bug Tracker](https://bugzilla.proxmox.com)
