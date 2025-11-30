✅ **Direct server SSH** || **Cloudflare Tunnel (Docker) SSH mode**


---

# 📘 **SSH Key Authentication Setup Guide**

### **Windows PC (VS Code) → Ubuntu Server (Direct IP or Cloudflare Tunnel)**

**Supports Cloudflare Tunnel (docker-cloudflared).**

---

# 1. Generate SSH Key on Windows

Open **PowerShell**:

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Press **Enter** for default path:

```
C:\Users\<WIN-USER>\.ssh\id_ed25519
```

Press **Enter** for passphrase unless you want one.

Created files:

| File             | Location    | Purpose                             |
| ---------------- | ----------- | ----------------------------------- |
| `id_ed25519`     | private key | Keep secure (NEVER copy to server!) |
| `id_ed25519.pub` | public key  | Upload to server                    |

---

# 2. Copy Public Key to Ubuntu Server

## **A. Direct SSH (Server has IP)**

```powershell
scp C:\Users\<WIN-USER>\.ssh\id_ed25519.pub username@SERVER_IP:/tmp/mykey.pub
```

---

## **B. SSH Through Cloudflare Tunnel (Docker)**

Your docker command likely looks like:

```
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <TOKEN>
```

Cloudflare tunnel creates an SSH endpoint like:

```
ssh://<your-tunnel-name>.cfargotunnel.com → localhost:22
```

### To copy your key using the tunnel:

```powershell
scp -P <CLOUDFLARE_SSH_PORT> C:\Users\<WIN-USER>\.ssh\id_ed25519.pub username@localhost:/tmp/mykey.pub
```

Examples:

**Default SSH (22):**

```powershell
scp -P 22 C:\Users\aolmd\.ssh\id_ed25519.pub erp@localhost:/tmp/mykey.pub
```

**If using custom SSH port (e.g., 2443):**

```powershell
scp -P 2443 C:\Users\aolmd\.ssh\id_ed25519.pub erp@localhost:/tmp/mykey.pub
```

---

# 3. Install the Key on Ubuntu Server

Login once with password:

## **A. Direct IP**

```powershell
ssh username@SERVER_IP
```

## **B. Cloudflare Tunnel**

```powershell
ssh -p <CLOUDFLARE_PORT> username@localhost
```

---

## Then run on Ubuntu:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat /tmp/mykey.pub > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm /tmp/mykey.pub
```

❗Important: The server should **not** have its own `id_ed25519` inside `~/.ssh/`.

If present:

```bash
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub 2>/dev/null
```

---

# 4. Fix Permissions (MANDATORY)

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R username:username ~/.ssh
```

SSH will reject the key if permissions are incorrect.

---

# 5. Verify SSH Server Settings

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure:

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

Optional (if you later disable passwords):

```
PasswordAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

# 6. Windows SSH Config (Recommended)

Edit:

```
C:\Users\<WIN-USER>\.ssh\config
```

Add BOTH options:

---

## **A. Direct Server SSH**

```
Host myserver
    HostName SERVER_IP
    User username
    IdentityFile C:\Users\<WIN-USER>\.ssh\id_ed25519
```

---

## **B. Cloudflare Tunnel SSH**

Cloudflare gives a hostname like:

```
ssh.example.com or <tunnel>.cfargotunnel.com
```

OR you connect via localhost (docker-cloudflared mapped to SSH port).

### **If Tunnel Mapped to Port 22:**

```
Host myserver-tunnel
    HostName localhost
    Port 22
    User username
    IdentityFile C:\Users\<WIN-USER>\.ssh\id_ed25519
```

### **If Tunnel Mapped to Custom SSH Port:**

```
Host myserver-tunnel
    HostName localhost
    Port 2443
    User username
    IdentityFile C:\Users\<WIN-USER>\.ssh\id_ed25519
```

### **If using Cloudflare SSH hostname directly:**

```
Host myserver-tunnel
    HostName <your-tunnel-name>.cfargotunnel.com
    User username
    Port 22
    IdentityFile C:\Users\<WIN-USER>\.ssh\id_ed25519
```

---

# 7. Connect Using SSH

### Direct IP:

```powershell
ssh myserver
```

### Cloudflare Tunnel:

```powershell
ssh myserver-tunnel
```

---

# 8. Connect Using VS Code

1. Open VS Code
2. Press **Ctrl + Shift + P → Remote-SSH: Connect to Host**
3. Choose:

   * `myserver` (direct IP)
   * or `myserver-tunnel` (Cloudflare Tunnel)

VS Code will open a remote session over SSH.

---

# 9. Disable Password Login (Optional but Secure)

After confirming SSH key login works:

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```
PasswordAuthentication no
```

Apply:

```bash
sudo systemctl restart ssh
```

---

# 10. Troubleshooting (Cloudflare + SSH)

Run:

```bash
tail -n 40 /var/log/auth.log
```

Common issues:

| Issue                                 | Fix               |
| ------------------------------------- | ----------------- |
| Wrong key in `authorized_keys`        | Re-copy using SCP |
| Incorrect permissions                 | Run chmod + chown |
| Server still has its own `id_ed25519` | Remove it         |
| Wrong Cloudflare SSH port             | Update SSH config |
| Cloudflared container not running     | Restart docker    |

Check tunnel status:

```bash
docker logs <cloudflared_container_name>
```

Restart tunnel:

```bash
docker restart <cloudflared_container_name>
```

---
