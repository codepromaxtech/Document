

# 📘 **SSH Key Authentication Setup Guide**

### **Windows PC (VS Code) → Ubuntu Server**

This document describes the correct, secure, and permanent steps to configure SSH key-based authentication from a Windows workstation to any Ubuntu server.

---

# 1. **Generate SSH Key on Windows**

1. Open **PowerShell** on Windows.
2. Generate an ED25519 key pair:

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

3. When prompted for a location, press **Enter** (default):

```
C:\Users\<windows-user>\.ssh\id_ed25519
```

4. Press **Enter** for passphrase (or set one).

This creates:

| File             | Location                | Description             |
| ---------------- | ----------------------- | ----------------------- |
| `id_ed25519`     | `C:\Users\<user>\.ssh\` | Private key (keep safe) |
| `id_ed25519.pub` | Same folder             | Public key              |

---

# 2. **Copy Public Key to Ubuntu Server**

Use SCP to upload the public key.

### If connecting directly to server IP:

```powershell
scp C:\Users\<user>\.ssh\id_ed25519.pub username@SERVER_IP:/tmp/mykey.pub
```

### If connecting via Cloudflare Tunnel:

```powershell
scp -P <PORT> C:\Users\<user>\.ssh\id_ed25519.pub username@localhost:/tmp/mykey.pub
```

---

# 3. **Install the Key on Ubuntu Server**

Login once with password:

```bash
ssh username@SERVER_IP
```

Then run:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat /tmp/mykey.pub > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm /tmp/mykey.pub
```

**Important:**
The server must **NOT** have its own `id_ed25519` inside `~/.ssh/`.
Only Windows keys should be allowed.

If present, remove them:

```bash
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub 2>/dev/null
```

---

# 4. **Verify Key Permissions**

SSH will refuse the key if permissions are wrong.

Run:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R username:username ~/.ssh
```

This is **mandatory**.

---

# 5. **Ensure SSH Server Allows Public Key Auth**

Open SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure these lines exist and are not commented (`#` removed):

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication yes   # Optional: disable later
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

# 6. **Windows SSH Config (Recommended)**

Create/edit:

```
C:\Users\<user>\.ssh\config
```

Add:

```
Host myserver
    HostName 192.168.32.128
    User erp
    IdentityFile C:\Users\<user>\.ssh\id_ed25519
```

If using Cloudflare Tunnel:

```
Host myserver
    HostName localhost
    Port <PORT>
    User erp
    IdentityFile C:\Users\<user>\.ssh\id_ed25519
```

Now connect with:

```powershell
ssh myserver
```

---

# 7. **Using VS Code Remote SSH**

1. Open **VS Code**.
2. Press **Ctrl + Shift + P**.
3. Select:

```
Remote-SSH: Add New SSH Host
```

Enter:

```
ssh myserver
```

4. Save to SSH config.
5. Connect from the **Remote Explorer** panel.

---

# 8. **Test Passwordless Login**

From Windows:

```powershell
ssh myserver
```

If it logs in without password → **SUCCESS** ✔️

---

# 9. **Troubleshooting**

Check if key gets rejected:

```bash
tail -n 40 /var/log/auth.log
```

Common issues:

| Problem                         | Solution                   |
| ------------------------------- | -------------------------- |
| Wrong permissions               | Fix using chmod + chown    |
| Wrong username                  | Must match Ubuntu username |
| Server has its own `id_ed25519` | Remove it                  |
| Key pasted incorrectly          | Re-copy using SCP          |
| Wrong IdentityFile on Windows   | Fix in SSH config          |

---

# 10. **Optional: Disable Password Login (Highly Secure)**

After confirming key login works flawlessly:

Edit sshd:

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

Now SSH only allows key-based authentication.

---

# ✅ **DOCUMENTATION FINISHED**

If you want, I can convert this into:

📄 PDF
📘 Markdown
📑 HTML
📝 Notion-style page

Just tell me the format.
