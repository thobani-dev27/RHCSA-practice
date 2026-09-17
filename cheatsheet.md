# Command Cheat Sheets

## SSHD Configuration (`/etc/ssh/sshd_config`)

```bash
# Edit the config file
vi /etc/ssh/sshd_config
```

Common directives you'll be asked to set:

```
Port 22                          # change listening port
PermitRootLogin no               # yes | no | prohibit-password
PasswordAuthentication no        # disable password auth (key-only)
PubkeyAuthentication yes
PermitEmptyPasswords no
AllowUsers ntombi thapelo        # restrict to specific users
AllowGroups sysadmin             # restrict to specific group
MaxAuthTries 3
ClientAliveInterval 300
Banner /etc/issue.net
```

```bash
# Validate syntax before restarting (catches typos)
sshd -t

# Apply changes
systemctl restart sshd

# Verify it's listening
ss -tlnp | grep sshd
```
> If you change `Port`, remember to also update the firewall (`firewall-cmd --add-port=<port>/tcp --permanent`) and SELinux (`semanage port -a -t ssh_port_t -p tcp <port>`), same pattern as the httpd example below.

---

## SELinux Port Labeling (how to build the `semanage port` line)

The pattern is always:

```bash
semanage port -a -t <selinux_type> -p <protocol> <port_number>
```

- `-a` = add a new port mapping (use `-m` to modify an existing one instead)
- `-t <selinux_type>` = the SELinux port type the service expects (e.g. `http_port_t`, `ssh_port_t`)
- `-p <protocol>` = `tcp` or `udp`
- `<port_number>` = the port you're opening

**How to find the right `-t` type for a service:**

```bash
# List all ports already associated with a type
semanage port -l | grep http_port_t
# Example output: http_port_t   tcp   80, 81, 443, 488, 8008, 8009, 8443, 9000

# Search by keyword if you don't know the exact type name
semanage port -l | grep -i http

# Check what type a running service actually expects (from its process context)
ps -eZ | grep httpd
```

So for httpd on a non-default port (82), you look up that `httpd` uses `http_port_t`, confirm 82 isn't already listed, then add it:

```bash
semanage port -a -t http_port_t -p tcp 82

# Verify it was added
semanage port -l | grep http_port_t
```

> If you get "port already defined" when adding, it means another service already owns that port under a different type — use `-m` (modify) instead of `-a`, or pick a different port.

**Quick reference for common services:**

| Service | SELinux type      | Default port |
|---------|-------------------|--------------|
| httpd   | `http_port_t`     | 80, 443      |
| sshd    | `ssh_port_t`       | 22           |
| ftp     | `ftp_port_t`       | 21           |
| nfs     | `nfs_port_t`       | 2049         |
| samba   | `smbd_port_t`      | 445          |
