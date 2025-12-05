# Expanding Ubuntu LVM Root Filesystem to Use Full Disk

This guide explains how to expand an Ubuntu Linux server’s LVM-based root filesystem when the underlying disk is larger than the logical volume.  
Common scenario: a server has a 1TB disk but `/` only shows 100GB because the LVM Logical Volume (LV) was created smaller.

---

## 1. Check Disk and LVM Layout

### View block devices:
```bash
lsblk

---

## **Step 4: Format and Mount the Logical Volume**

### **1. Unmount the Volume (If Needed)**
```bash
sudo umount /mnt/storage
```

### **2. Format the Volume (Optional, This Erases Data!)**
```bash
sudo mkfs.ext4 /dev/ubuntu-vg-1/ubuntu-lv
```

### **3. Create a Mount Point**
```bash
sudo mkdir -p /mnt/storage
```

### **4. Mount the Logical Volume**
```bash
sudo mount /dev/ubuntu-vg-1/ubuntu-lv /mnt/storage
```

### **5. Verify the Mount**
```bash
mount | grep storage
```

### **6. Fix Mount Issues if Necessary**
```bash
sudo systemctl daemon-reload
```

### **7. Make the Mount Persistent**
Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```
Add this line:
```
/dev/ubuntu-vg-1/ubuntu-lv  /mnt/storage  ext4  defaults  0  2
```
Apply changes:
```bash
sudo mount -a
```

---

## **Step 5: Configure Docker to Use the Mounted Volume**

### **1. Stop Docker**
```bash
sudo systemctl stop docker
```

### **2. Ensure No Active Containers Are Running**
```bash
docker ps
```
If any containers are running, stop them to prevent data corruption:
```bash
docker stop $(docker ps -q)
```

### **3. Move Existing Data (If Needed)**
```bash
sudo mv /var/lib/docker /mnt/storage/
```

### **4. Bind Mount Instead of Symlink**
```bash
sudo mount --bind /mnt/storage/docker /var/lib/docker
```

### **5. Update Docker’s Storage Location**
Edit the Docker config:
```bash
sudo nano /etc/docker/daemon.json
```
Add:
```json
{
  "data-root": "/mnt/storage/docker"
}
```

### **6. Restart Docker**
```bash
sudo systemctl daemon-reload
sudo systemctl start docker
```

### **7. Verify Docker Storage**
```bash
docker info | grep "Docker Root Dir"
```

---

## **Step 6: Check Docker Disk Usage**

### **1. Check Docker Disk Usage**
```bash
docker system df --verbose
```

### **2. Check Disk Space in Docker’s Root Directory**
```bash
du -sh /mnt/storage/docker
```

### **3. Check Space Used by Containers & Images**
```bash
docker ps -a --size
docker images
```

---

## **Conclusion**
Your **2.7T storage** is now mounted at `/mnt/storage` and configured for Docker.

For troubleshooting:
```bash
dmesg | grep -i lvm
```

Happy computing! 🚀
