# RHCSA Exam — Study Guide & Solutions

Primary & Secondary Machine Tasks

---

## PRIMARY MACHINE

### Q1. Setup IPv4 Address for Primary Virtual Machine

> Set ip addr 172.25.X.11, subnet mask 255.255.255.0, Default gateway 172.25.X.254, nameserver 172.25.254.254, hostname as primary.netX.example.com

```bash
# Step 1: Check existing connections
nmcli connection show

# Step 2: Modify the connection (replace X with your number)
nmcli connection modify "cloud-init enp0s3" ipv4.addresses 172.25.X.11/24 ipv4.gateway 172.25.X.254 ipv4.dns 172.25.254.254 ipv4.method manual

# Step 3: Bring the connection down then up
nmcli connection down "cloud-init enp0s3"
nmcli connection up "cloud-init enp0s3"

# Step 4: Set the hostname
hostnamectl set-hostname primary.netX.example.com

# Step 5: Verify
ip addr show enp0s3
hostname
```
> Replace X with your assigned number throughout this task.

---

### Q2. YUM Repository Configuration (Both Machines)

> Configure repositories: http://content.example.com/rhel8.0/x86_64/dvd/BaseOS and http://content.example.com/rhel8.0/x86_64/dvd/AppStream

```bash
# Step 1: Create the repo file
vi /etc/yum.repos.d/exam.repo
```
```ini
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
```
```bash
# Step 2: Save
ESC > :wq

# Step 4: Verify repositories are loaded
dnf repolist
```
> Repeat all steps on the second machine (student2 / 172.168.122.11) via SSH.

---

### Q3. Configure httpd on Custom Port with SELinux

```bash
# Step 1: Install httpd if not already installed
dnf install httpd -y

# Step 2: Configure httpd to listen on port 82
vi /etc/httpd/conf/httpd.conf
# Find 'Listen 80' and change to: Listen 82

# Step 3: Install SELinux management tools
dnf install -y policycoreutils-python-utils setroubleshoot

# Step 4: Allow SELinux to use port 82 for httpd
semanage port -a -t http_port_t -p tcp 82

# Step 5: Restore correct SELinux context on web files
restorecon -Rv /var/www/html

# Step 6: Open port 82 in the firewall
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload

# Step 7: Enable httpd to start at boot and start it now
systemctl enable --now httpd

# Step 8: Verify
systemctl status httpd
curl http://localhost:82
```

---

### Q4. Configure a Cron Job on Primary Machine

> User natasha must configure a cron job that runs daily at 13:30 and executes /bin/echo hello

```bash
# Step 1: Open natasha's crontab
crontab -u natasha -e

# Step 2: Add the following line
30 13 * * * /bin/echo "hello"

# Step 3: Verify
crontab -u natasha -l
```
> Format: minute hour day month weekday command. `30 13 * * *` = daily at 13:30

---

### Q5. Create Users, Groups, and Group Memberships

> Group: sysadmin. Users: ntombi, thapelo (sysadmin secondary group), tomas (no shell, not in sysadmin). All passwords: atenorth

```bash
# Step 1: Create the sysadmin group
groupadd sysadmin

# Step 2-3: Create users with sysadmin as secondary group
useradd -G sysadmin ntombi
useradd -G sysadmin thapelo

# Step 4: Create user tomas with no interactive shell
useradd -s /sbin/nologin tomas

# Step 5: Set password 'atenorth' for all users
echo 'atenorth' | sudo passwd --stdin ntombi
echo 'atenorth' | sudo passwd --stdin thapelo
echo 'atenorth' | sudo passwd --stdin tomas

# Step 6: Verify
grep 'ntombi\|thapelo\|tomas\|sysadmin' /etc/group
```
> The `-s /sbin/nologin` flag prevents interactive shell access for tomas.

---

### Q6. Create a Collaborative Directory /common/street

> Group ownership: withincode. Readable/writable/accessible to withincode only. New files auto-inherit group ownership.

```bash
# Step 1: Create the directory
mkdir -p /common/street

# Step 2: Create the withincode group if it doesn't exist
groupadd withincode

# Step 3: Set group ownership
chown :withincode /common/street

# Step 4: Set permissions (rwx for group, none for others) + SGID bit
chmod 2770 /common/street

# Step 5: Verify
ls -la /common/
```
> The `2` in `2770` sets the SGID bit — this ensures new files inherit the withincode group ownership automatically.

---

### Q7. Create and Extract Archives

> Create a compressed archive of /etc named /root/etc-backup.tar.gz, then extract it to /root/etc-extracted

```bash
# Step 1: Create a compressed tar archive of /etc
tar -czvf /root/etc-backup.tar.gz /etc

# Step 2: Verify the archive was created
ls -la /root/etc-backup.tar.gz

# Step 3: Create extraction directory
mkdir -p /root/etc-extracted

# Step 4: Extract the archive into it
tar -xzvf /root/etc-backup.tar.gz -C /root/etc-extracted

# Step 5: Verify contents were extracted
ls /root/etc-extracted
```
> `-c` create, `-z` gzip compress, `-x` extract, `-v` verbose, `-f` specify filename. `-C` extracts into a target directory instead of the current one.

### Q8. Configure NTP Client

> Configure your system as an NTP client of classroom.example.com

```bash
# Step 1: Edit the chrony configuration
vi /etc/chrony.conf
# Find 'pool'/'server' line and replace/add:
server classroom.example.com iburst

# Step 2: Restart chronyd
systemctl restart chronyd

# Step 3: Verify time sync
chronyc sources -v
```
> You should see classroom.example.com listed as a time source with an asterisk (*) indicating it's selected.

---

### Q9. Find Files Owned by 'thomas' and Copy to /root/found

```bash
# Step 1: Create Directory
mkdir -p /root/found

# Step 2: Find
find / -user thomas -type f -exec cp {} /root/found \; 2>/dev/null

# Step 3: Verify
ls -la /root/found
```

---

### Q10. Find Files with SUID Permission

```bash
# Step 1: Create Directory
mkdir -p /root/SUID-files

# Step 2: Find
find / -perm -4000 -type f -exec cp {} /root/SUID-files \; 2>/dev/null

# Step 3: Verify
ls -la /root/SUID-files
```

---

### Q11. Find String 'strato' from Dictionary

```bash
# Step 1: Grep
grep 'strato' /usr/share/dict/words > /searchfile.txt

# Step 2: Verify
cat /searchfile.txt
```

---

### Q12. Configure Autofs for NFS Home Directories

> Automount netuserX home directory from classroom.example.com:/home/guests/netuserX. Must be writable. Password: ablerate

```bash
# Step 1: Install required packages
dnf install -y nfs-utils autofs

# Step 2: Add the master autofs map entry
vi /etc/auto.master
# Add: /home/guests /etc/auto.guests

# Step 3: Create the auto.guests map file
vi /etc/auto.guests
# Add: * -rw classroom.example.com:/home/guests/&

# Step 4: Enable and start autofs
systemctl enable --now autofs

# Step 5: Verify by switching to the user
su - netuserX
pwd
```
> The `&` symbol substitutes the username automatically. The `-rw` flag makes the mount writable.

---

### Q13. Create User with Specific UID

> Create user barry with UID 2112 and set password atenorth

```bash
# Step 1: Add User it doesn't exist
useradd -u 2112 barry

# Step 2: Add Password
echo 'atenorth' | sudo passwd --stdin barry

# Step 3: Verify
id barry
```

---

### Q14. Grant Sudo Privileges Without Password

> Group 'elite' must have administrative permission without password

```bash
# Step 1: Create the group if it doesn't exist
groupadd elite

# Step 2: Edit sudoers file
visudo
# Add: %elite ALL=(ALL) NOPASSWD: ALL

# Verify
visudo -c
```
> Never edit /etc/sudoers directly. Always use visudo to prevent syntax errors.

---

### Q15. Download and Build Container Image

> Download Containerfile from http://classroom.example.com/Containerfile. Do not modify. Build the image.

```bash
# Step 1: Create Directories (As Root)
mkdir -p /opt/files /opt/processed
chown Xanadu:xanadu /opt/files /opt/processed
chmod 777 /opt/files /opt/processed
loginctl enable-linger Xanadu

# Step 2: SSH to user
ssh xanadu@ip_address

# Step 3: Login to podman using registry details
podman login classroom.example.com/Containerfile
# Username:
# Password:

# Step 4: Get Containerfile
curl -O http://classroom.example.com/Containerfile

# Step 5: Build the image
podman build -t myimage .

# Step 6: Verify
podman images
```

---

### Q16. Configure Container as Systemd Service

> Create container 'mycontainer' from built image. Mount /opt/file to /opt/incoming and /opt/processed to /opt/outgoing. Run as user xanadu. Auto-start on reboot.

```bash
# Step 1: Run the container with volume mounts
podman run -d --name mycontainer -v /opt/file:/opt/incoming:Z -v /opt/processed:/opt/outgoing:Z myimage

# Step 2: Create systemd user directory and generate service file
mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user
podman generate systemd --files --name mycontainer --new

# Step 3: Enable and start the service
systemctl --user daemon-reload
systemctl --user enable --now container-mycontainer.service

# Step 4: Verify
systemctl --user status container-mycontainer.service
podman images
podman ps
```

---

## SECONDARY MACHINE

### Secondary Q1. Crack/Reset Root Password (Alternative Method)

> Break into the secondary machine and reset the root password

```
Step 1: Reboot the machine and interrupt the boot at GRUB menu
  - Press arrow keys when GRUB menu appears to stop auto-boot
  - Select the kernel entry and press 'e' to edit
```
```bash
# Step 2: Find the line starting with 'linux', change 'ro' to 'rw', ctrl+e
# and append init=/bin/bash

# Step 3: Boot with the edited line
Ctrl+X

# Step 4: Reset the root password
passwd root
# Type the new password twice

# Step 5: Force SELinux to relabel the filesystem on next boot
touch /.autorelabel

# Step 6: Force a reboot
/sbin/reboot -f
```

---

### Secondary Q2. YUM Repository Configuration

> Configure repositories: http://content.example.com/rhel8.0/x86_64/dvd/BaseOS and http://content.example.com/rhel8.0/x86_64/dvd/AppStream

```bash
# Step 1: Create the repo file
vi /etc/yum.repos.d/exam.repo
```
```ini
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
```
```bash
# Step 2: Save
ESC > :wq

# Step 4: Verify repositories are loaded
dnf repolist
```

---

### Secondary Q3. Set Recommended Tuning Profile

```bash
# Step 1: Install tuned if not available
dnf install -y tuned

# Step 2: Enable and start tuned
systemctl enable --now tuned

# Step 3: Check recommended profile
tuned-adm recommend

# Step 4: Apply the recommended profile
tuned-adm profile <recommended-profile-name>

# Step 5: Verify
tuned-adm active
```
> The recommended profile is auto-detected from your hardware. Use `tuned-adm list` for all available profiles.

---

### Secondary Q4. Create a 250MB SWAP Partition

> Create SWAP partition of 250MB and make it available at next reboot. Partition already available.

```bash
# Step 1: Identify the available partition
lsblk

# Step 2: Create swap partition using fdisk
fdisk /dev/sdb
```
```
Inside fdisk:
  n   -> new partition
  p   -> primary
  <partition number, e.g. 1>
  <Enter>   -> accept default first sector
  +250M     -> size
  t         -> change type
  82 (or 'swap')
  w         -> write and quit
```
```bash
# Step 3: Create the swap filesystem
mkswap /dev/sdb1
blkid /dev/sdb1

# Step 4: Get the UUID
lsblk -f

# Step 5: Add to /etc/fstab for persistence
vi /etc/fstab
# Add: UUID=<your-uuid> swap swap defaults 0 0

# Step 6: Activate and verify
sudo swapon -a
swapon
```

---

### Secondary Q5. Create LVM with VG myvol and LV mydatabase

> VG: myvol with 8MiB PE. LV: mydatabase with 100 PE. Format as vfat. Mount on /database permanently.

```bash
# Step 1: Create the volume group with 8MiB physical extents
vgcreate -s 8M myvol /dev/sdb

# Step 2: Create the logical volume with 100 extents
lvcreate -l 100 -n mydatabase myvol

# Step 3: Install dosfstools and format as vfat
dnf install -y dosfstools
mkfs.vfat /dev/myvol/mydatabase

# Step 4: Create the mount directory
mkdir /database

# Step 5: Get UUID and add to /etc/fstab
lsblk -f
blkid /dev/myvol/mydatabase

vi /etc/fstab
# Add: UUID=<your-uuid> /database vfat defaults 0 0

# Step 6: Mount and verify
systemctl daemon-reload
mount -a
df -h /database
```
> 100 PE × 8MiB = 800MiB total size for the logical volume.

---

### Secondary Q6. Resize LVM Partition 'home' to 150MiB

```bash
# Step 1: Check current size
lvs

# Step 2: Resize the logical volume (-r also resizes the filesystem)
lvresize -rL {size} {path}

# Step 3: Verify the new size
lvs
df -h /home
```
> The `-r` flag automatically resizes the filesystem along with the LV. If shrinking, ensure the data fits within 150MiB first.
