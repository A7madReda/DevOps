# Configure SSH Key-Only Authentication

## 1. First Time SSH Key Setup

If you haven't created an SSH key pair yet:

### On Your Local Machine:
```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

### Copy Key to Server:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server-ip
```

## 2. Secure SSH Configuration

### Edit SSH Configuration
```bash
nano /etc/ssh/sshd_config
```

Add or modify these lines:
```
# Disable password authentication
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes

# Only allow key-based authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Disable root login
PermitRootLogin prohibit-password

# Only allow specific users (optional)
# AllowUsers your-username

# Specify which keys to accept (optional)
# AuthorizedKeysFile .ssh/authorized_keys
```

## 3. Set Correct Permissions

### On the Server:
```bash
# Set ownership
chown -R username:username ~/.ssh
# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 4. Apply Changes

```bash
# Test configuration
sshd -t

# Restart SSH service
systemctl restart sshd
```

## 5. Verify Configuration

Before logging out, open a new terminal and verify you can still log in with your SSH key:
```bash
ssh username@server-ip
```

## Security Note
- Keep your private key secure
- Back up your keys
- Consider using ssh-agent for key management
- Monitor auth.log for unauthorized access attempts

## Troubleshooting

If you get locked out:
1. Access server directly through console
2. Check SSH logs: `journalctl -u ssh`
3. Verify permissions on ~/.ssh directory
4. Check sshd_config syntax
