# SSH Authentication Troubleshooting

## If authentication still fails after adding keys:

### 1. Check server SSH configuration
SSH into your server and verify:

```bash
# Check SSH daemon config
sudo cat /etc/ssh/sshd_config | grep -E "PubkeyAuthentication|PasswordAuthentication|PermitRootLogin"

# Should see:
# PubkeyAuthentication yes
```

If `PubkeyAuthentication` is `no`, enable it:
```bash
sudo sed -i 's/PubkeyAuthentication no/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 2. Verify file permissions on server
```bash
# On the server
ls -la ~/.ssh/
# Should show:
# drwx------ (700) for .ssh directory
# -rw------- (600) for authorized_keys file

# Fix if needed:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 3. Check authorized_keys content
```bash
# On the server
cat ~/.ssh/authorized_keys
# Should contain the public key on a SINGLE line with no extra spaces
```

### 4. Test SSH connection with verbose logging
```bash
# From your local machine
ssh -vvv -i ~/drone_deploy_key ubuntu@210.79.128.168
```

### 5. Check server auth logs
```bash
# On the server
sudo tail -f /var/log/auth.log
# Or on some systems:
sudo tail -f /var/log/secure
```

### 6. Verify GitHub Secret format
The private key in GitHub Secrets must:
- Include the `-----BEGIN OPENSSH PRIVATE KEY-----` header
- Include the `-----END OPENSSH PRIVATE KEY-----` footer
- Have NO extra spaces or newlines at the beginning or end
- Preserve all newlines in the key body

### 7. Alternative: Use password authentication (not recommended)
If you need a quick workaround, you can use password authentication:

```yaml
- name: Deploy on ExCloud
  uses: appleboy/ssh-action@v1
  with:
    host: 210.79.128.168
    username: ubuntu
    password: ${{ secrets.SSH_PASSWORD }}
    script: |
      # your commands
```

But public key authentication is more secure.

### 8. Check if the user has a shell
```bash
# On the server
grep ubuntu /etc/passwd
# Should show: /bin/bash or /bin/sh at the end
```

### 9. SELinux issues (if applicable)
```bash
# On the server (if SELinux is enabled)
sudo restorecon -R ~/.ssh
```

### 10. Check server firewall
```bash
# On the server
sudo ufw status
# Ensure port 22 is allowed
```
