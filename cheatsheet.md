# Command Cheat Sheets
```bash

View your repo config files
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
## SSHD Configuration (/etc/ssh/sshd_config)

```bash
# Edit the config file
vi /etc/ssh/sshd_config

```

Common directives you'll be asked to set, just update them exactly as below:

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


# For crontab structure use run:

```bash
cat /etc/crontab

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed


To view Repo tamplete run: 

cat /etc/yum.repos.d/*.repo
