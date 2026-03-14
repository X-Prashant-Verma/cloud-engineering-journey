## What is Linux and Why Does It Matter in Cloud Engineering?

Linux is a free, open-source operating system that powers most of the world's servers, 
cloud platforms, and infrastructure.

Almost every major cloud provider — AWS, GCP, Azure — runs its virtual machines and 
containers on Linux under the hood.

As a Cloud/DevOps Engineer, you'll spend the majority of your time *inside* Linux 
terminals — managing servers, deploying apps, and automating workflows.

That's why Linux is the single most foundational skill to get right.

## Linux Filesystem Hierarchy

Linux organizes everything into a single tree starting from `/` (root).
Each directory has a specific, fixed purpose.

| Directory | Purpose |
|-----------|---------|
| `/`       | Root of the entire filesystem. Everything lives here. |
| `/etc`    | System and app configuration files (nginx, ssh, hosts) |
| `/var`    | Variable data — logs, caches, databases. Grows over time. |
| `/home`   | User home directories (`/home/username`) |
| `/tmp`    | Temporary files. Cleared on every reboot. |

### Why it matters as a Cloud Engineer
- `/etc` → Where you edit config files for servers and services
- `/var/log` → First place to check when something breaks on a server
- `/home` → Your SSH keys and shell configs live here

## Linux User & Group Model

Linux is a multi-user system. Every action is tied to a user identity with 
specific permissions.

### User Types
| Type | Description |
|------|-------------|
| `root` | Superuser (UID 0). Unrestricted access to everything. |
| Regular user | Normal login user. Limited to their own files and explicit permissions. |
| System user | Created by software (nginx, mysql). Runs services. No login shell. |

### sudo
Lets a trusted regular user run a single command as root.
Safer than logging in as root — every command is logged.
```bash
sudo apt install nginx
sudo systemctl restart ssh
```

### /etc/passwd
Stores user account info. Readable by all users.
Format: `username:x:UID:GID:comment:home_dir:shell`
Password field shows `x` — actual password is in /etc/shadow.

### /etc/shadow
Stores hashed passwords. Readable ONLY by root.
This separation is intentional — passwd is public info, shadow is protected.

### Groups
Groups let you assign permissions to multiple users at once.
```bash
groups prashant                    # check user's groups
sudo usermod -aG docker prashant   # add user to a group
```

### Why it matters as a Cloud Engineer
- AWS/GCP servers give you a regular user login (ubuntu, ec2-user) — never root
- "Permission denied" errors almost always trace back to user/group mismatches
- Server hardening = auditing /etc/passwd and restricting sudo access

## Linux File Permissions — rwx, chmod, chown

Every file has three permission groups: Owner / Group / Others
Each group has three bits: r (read=4), w (write=2), x (execute=1)

### Reading permissions
```bash
ls -l app.sh
# -rwxr-xr-- prashant developers app.sh
#  ^^^       → owner: rwx (full)
#     ^^^    → group: r-x (read+execute)
#        ^^^ → others: r-- (read only)
```

### chmod — Change permissions
```bash
# Symbolic
chmod u+x app.sh        # add execute for owner
chmod g-w config.yaml   # remove write from group

# Numeric (octal) — add r(4)+w(2)+x(1) per group
chmod 755 app.sh        # rwxr-xr-x
chmod 644 config.txt    # rw-r--r--
chmod 600 id_rsa        # rw------- (SSH private key — must be this)
```

### chown — Change ownership
```bash
chown prashant app.sh              # change owner
chown prashant:developers app.sh   # change owner and group
chown -R prashant /var/www/html    # recursive (entire folder)
```

### Why it matters as a Cloud Engineer
- SSH key must be chmod 600 — AWS rejects connections otherwise
- Web server (nginx/apache) needs read access to your app files
- Scripts need execute bit — chmod +x deploy.sh
- Containers inherit host permissions — misconfig here breaks apps inside Docker

## Linux Process Management — ps, top, kill, systemctl

Every running program is a process. Cloud Engineers need to monitor, 
manage, and control processes on live servers.

### ps — Snapshot of running processes
```bash
ps aux                    # show all running processes
ps aux | grep nginx       # find a specific process and its PID
```

### top / htop — Live process monitor
```bash
top                       # live CPU/memory view (press q to quit)
htop                      # friendlier version (sudo apt install htop)
```

### kill — Stop a process
```bash
kill <PID>                # graceful stop (SIGTERM) — try this first
kill -9 <PID>             # force kill (SIGKILL) — last resort only
killall nginx             # kill all processes with this name
```
⚠️ Always try kill before kill -9 — force-killing a DB mid-write can corrupt data.

### systemctl — Managing services
```bash
sudo systemctl start nginx      # start a service
sudo systemctl stop nginx       # stop a service
sudo systemctl restart nginx    # restart a service
sudo systemctl status nginx     # check if running — first debug command
sudo systemctl enable nginx     # auto-start on server reboot
sudo journalctl -u nginx -n 50  # view last 50 log lines for a service
```

### Why it matters as a Cloud Engineer
- Apps on EC2 run as systemctl services — so they survive reboots
- top/htop → instantly spot which process is killing your CPU or memory
- systemctl status → first command when a service goes down on a live server
- CI/CD pipelines restart services with systemctl after every deployment
