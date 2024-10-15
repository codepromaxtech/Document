To find **active users** who still have access to your server (especially those that may log in via SSH), and filter out deleted users whose home directories still exist, follow these steps:

---

### **Step 1: List Existing Users from `/etc/passwd`**
This lists users currently present on the system.

```bash
awk -F: '$3 >= 1000 {print $1}' /etc/passwd
```
- This command shows **non-system users** (UID >= 1000) who might have login access.

---

### **Step 2: Check Users with SSH Access**  
If users can access your server remotely, they must be allowed in the SSH configuration.

1. **View the list of allowed SSH users** (if `AllowUsers` is specified):
   ```bash
   grep "^AllowUsers" /etc/ssh/sshd_config
   ```

2. If `AllowUsers` is not used, check if SSH login is enabled for all users:
   ```bash
   cat /etc/passwd | grep "/bin/bash" | awk -F: '{print $1}'
   ```

---

### **Step 3: List Users with Existing Home Directories**  
This will show directories still present under `/home/` (even if the users were deleted):

```bash
ls /home/
```

If any of these directories belong to deleted users, you’ll need to **compare with active users** from `/etc/passwd` (Step 1).  

---

### **Step 4: Find Home Directories without Matching Users**
To identify **home directories without matching active users**, run:

```bash
for user in $(ls /home); do
  if ! id -u "$user" &>/dev/null; then
    echo "Home directory exists but user $user is deleted."
  fi
done
```

---

### **Step 5: Check for Users with Valid SSH Keys**  
Even if users are deleted, someone might have **left SSH keys** that allow access:

```bash
find /home/*/.ssh/authorized_keys
```

- This will show if any `authorized_keys` files remain in deleted users' home directories.  
- If you find any, you might need to **manually remove those files** to prevent unauthorized access.

---

### **Step 6: Disable or Lock Old Users**
For users you find unnecessary, you can lock or delete them:

- **Lock a user** (disables login):
  ```bash
  usermod -L username
  ```
  - **Delete a user with home directories** :
  ```bash
   sudo userdel -r username
  ```

- **Remove old user directories** (after verification):
  ```bash
  rm -rf /home/username
  ```

---
To **list all SSH keys** authorized to access your system and **remove specific ones**, follow these steps depending on your operating system and environment:

---

## **1. List All SSH Keys**
Authorized SSH keys are stored in the `~/.ssh/authorized_keys` file of each user.

### **Check SSH Keys for a User**
```bash
cat ~/.ssh/authorized_keys
```

### **List SSH Keys for All Users**  
To check SSH keys for all users on the system:
```bash
for user in $(cut -f1 -d: /etc/passwd); do
  if [ -e /home/$user/.ssh/authorized_keys ]; then
    echo "SSH keys for user: $user"
    cat /home/$user/.ssh/authorized_keys
  fi
done
```

For **root** user:
```bash
cat /root/.ssh/authorized_keys
```

---

## **2. Remove an SSH Key**
SSH keys can be removed by editing the `authorized_keys` file.

### **Manually Edit Authorized Keys**
```bash
nano ~/.ssh/authorized_keys
```
- Find the key you want to remove (each key is usually a single line).
- Delete the entire line containing the unwanted key.
- Save the file and exit.

### **Automatically Remove an SSH Key by Matching Content**
To remove a specific SSH key (e.g., with a matching comment):
```bash
sed -i '/<key_or_comment>/d' ~/.ssh/authorized_keys
```
Replace `<key_or_comment>` with a unique part of the key or comment to identify it.

---

## **3. Verify the SSH Key Removal**
After removing the key(s), verify with:
```bash
cat ~/.ssh/authorized_keys
```

---

## **4. Restart SSH Service (Optional)**
If your setup requires it, restart the SSH service:
```bash
systemctl restart sshd
```

---

This method ensures that only trusted keys remain, reducing unauthorized access risks. If you are managing multiple servers, you can use a **configuration management tool** (e.g., Ansible) to automate these steps across systems.
These steps should help you identify actual active users and clean up any remaining home directories for deleted users. Let me know if you need further help!
