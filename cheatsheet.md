# Command Cheat Sheets

## YUM/DNF Repository

Repositories tell RHEL where to get software and packages.

```bash
# View repository configuration
cat /etc/yum.repos.d/*.repo
```

Example local repository:
```ini
[BaseOS]
name=BaseOS
baseurl=file:///mnt/iso/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=file:///mnt/iso/AppStream
enabled=1
gpgcheck=0
```
- `[BaseOS]` = repository name
- `baseurl` = where packages are located
- `enabled=1` = repository is ON
- `gpgcheck=0` = package signature checking is disabled

```bash
dnf repolist
dnf clean all
dnf install package-name
```

---

## SSHD Configuration (`/etc/ssh/sshd_config`)

```bash
# Edit the config file
vi /etc/ssh/sshd_config
```

Common directives you'll be asked to set, just update them as below:

```
PermitRootLogin yes
PasswordAuthentication yes
PubkeyAuthentication yes
```

```bash
# Apply changes
systemctl restart sshd

# Validate syntax before restarting (catches typos)
sshd -t

# Verify
systemctl status sshd
```

---

## SELinux Port Labeling (how to build the `semanage port` line)

```bash
# To find this line: semanage port -a -t http_port_t -p tcp 82, run:
man semanage-port

# To find the /var/www/html, restorecon lines, run:
man semanage-fcontext
```

**Quick reference for common services:**

| Service | SELinux type   | Default port |
|---------|----------------|--------------|
| httpd   | `http_port_t`  | 80, 443      |
| sshd    | `ssh_port_t`   | 22           |
| ftp     | `ftp_port_t`   | 21           |
| nfs     | `nfs_port_t`   | 2049         |
| samba   | `smbd_port_t`  | 445          |

---

## SELinux File Contexts

Use file contexts when SELinux needs to recognize a directory as web content (or another service's content type).

```bash
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
ls -Z /web
```
> `semanage fcontext` sets the rule; `restorecon` applies the label.

---

## Resizing Filesystems on an LVM Logical Volume

General pattern: grow the LV first, then grow the filesystem on top of it.

```bash
# vfat — unmount first, resize LV, then resize filesystem
umount /database
lvresize -L +500M /dev/myvol/mydatabase
fatresize -s max /dev/myvol/mydatabase
mount -a

# ntfs — unmount first, resize LV, then resize filesystem
umount /database
lvresize -L +500M /dev/myvol/mydatabase
ntfsresize -f /dev/myvol/mydatabase
mount -a

# xfs — must be mounted to grow, no shrink support
lvresize -L +500M /dev/myvol/mydatabase
xfs_growfs /database

# ext4 — can grow mounted or unmounted
lvresize -L +500M /dev/myvol/mydatabase
resize2fs /dev/myvol/mydatabase
```

**Must the filesystem be unmounted to resize?**

| Filesystem | Grow                                                     | Shrink                    |
|------------|------------------------------------------------------------|----------------------------|
| xfs        | No — `xfs_growfs` requires the filesystem to be mounted    | Not supported at all       |
| ext4       | No — `resize2fs` can grow while mounted                    | Yes — must unmount first   |
| vfat       | Yes — `fatresize` requires it to be unmounted               | Yes — must unmount first   |
| ntfs       | Yes — `ntfsresize` requires it to be unmounted               | Yes — must unmount first   |

## Cron Jobs

Cron automatically runs commands at scheduled times.

```bash
cat /etc/crontab
```
```
minute hour day month weekday user command
```
The five time fields are: MINUTE → HOUR → DAY → MONTH → WEEKDAY.

```
0 2 * * * root /backup.sh
```
This example runs `/backup.sh` as root at 02:00 every day.

For a user's own crontab, use the `crontab` command instead of editing a file directly:
```bash
crontab -e              # edit your own crontab
crontab -l               # view your own crontab
crontab -u natasha -l    # view another user's crontab, as root
```

---

## Useful RHCSA Commands

**Files & Directories**
```bash
pwd
ls
ls -l
cd /directory
mkdir directory
touch file
cp source destination
mv old new
rm file
rm -r directory
find / -name filename
cat file
vi file
```

**Services**
```bash
systemctl start service
systemctl stop service
systemctl restart service
systemctl enable service
systemctl status service
```

**Networking**
```bash
ip a
ip r
hostname
```

**Users**
```bash
useradd username
passwd username
usermod
userdel username
```

**Permissions**
```bash
chmod
chown
chgrp
```

**Storage**
```bash
lsblk
blkid
df -h
du -h
mount
umount
```

**LVM**
```bash
pvs
vgs
lvs
pvcreate
vgcreate
lvcreate
lvresize
```

**SELinux**
```bash
getenforce
sestatus
semanage
restorecon
ls -Z
```

**Firewall**
```bash
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

**Packages**
```bash
dnf install package
dnf remove package
dnf update
dnf search package
dnf repolist
```

---

## RHCSA Exam Thinking Pattern

```
WHAT DO THEY WANT?
    ↓
WHICH SERVICE / FILE / STORAGE?
    ↓
WHICH COMMAND CHANGES IT?
    ↓
DO I NEED SELINUX?
    ↓
DO I NEED FIREWALL?
    ↓
HOW DO I VERIFY IT?
```
> Always verify your work after making a change.
