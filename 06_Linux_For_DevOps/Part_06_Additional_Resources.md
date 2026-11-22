# Part 06 — Command Reference and Resources

Use this file for lookup while practising, not as a second textbook. Each command includes the smallest useful explanation and one example. For the reasoning behind them, see the [study notes](Part_06_Study_Notes.md).

**Conventions:** `$` is a normal-user prompt, `#` is root, and `<...>` is a value you replace. Commands marked **supplement** were not demonstrated in the lecture but are needed for real work.

## 1. Navigation and location

| Command | What it does | Example |
|---|---|---|
| `pwd` | Print the absolute path of the current directory | `pwd` |
| `ls` | List directory names | `ls` |
| `ls -l` | Long format with permissions and ownership | `ls -l /var/www` |
| `ls -a` | Include hidden files (dotfiles) | `ls -a ~` |
| `ls -lah` | Long, hidden, human-readable sizes | `ls -lah /var/log` |
| `cd <dir>` | Change directory | `cd /etc/nginx` |
| `cd ..` | Go to the parent directory | `cd ..` |
| `cd ~` | Go to your home directory | `cd ~` |
| `cd -` | Return to the previous directory | `cd -` |
| `tree` **supplement** | Show a directory tree | `tree -L 2 /etc/nginx` |

```bash
# Where am I, and what is around me?
pwd
ls -lah
cd ..
pwd
```

`tree` may need `sudo apt install tree`.

## 2. Files and directories

| Command | What it does | Example |
|---|---|---|
| `mkdir <dir>` | Create a directory | `mkdir website` |
| `mkdir -p a/b/c` | Create missing parents | `mkdir -p app/static/css` |
| `touch <file>` | Create an empty file or update its timestamp | `touch index.html` |
| `cp <src> <dst>` | Copy a file | `cp notes.txt notes.bak` |
| `cp -r <dir> <dst>` | Copy a directory recursively | `cp -r site site-backup` |
| `mv <src> <dst>` | Move or rename | `mv old.txt new.txt` |
| `rm <file>` | Delete a file (permanent) | `rm notes.bak` |
| `rm -i <file>` | Delete with confirmation | `rm -i notes.bak` |
| `rm -r <dir>` | Delete a directory and its contents | `rm -r old-site` |
| `rmdir <dir>` | Remove an empty directory only | `rmdir empty-dir` |
| `ln -s <target> <link>` | Create a symbolic link | `ln -s /var/www/site/current /var/www/html` |

**Before any recursive delete:**

```bash
echo "$DIR"
ls -la "$DIR"
rm -rf -- "$DIR"
```

There is no recycle bin. On a VM, take a snapshot first.

## 3. Viewing, editing and searching text

| Command | What it does | Example |
|---|---|---|
| `cat <file>` | Print a small file | `cat /etc/hostname` |
| `cat -n <file>` | Print with line numbers | `cat -n notes.txt` |
| `less <file>` | Page through a large file; `q` quits, `/` searches | `less /var/log/syslog` |
| `head -n 20 <file>` | First 20 lines | `head -n 20 access.log` |
| `tail -n 20 <file>` | Last 20 lines | `tail -n 20 error.log` |
| `tail -f <file>` | Follow new lines | `tail -f /var/log/nginx/access.log` |
| `wc -l <file>` | Count lines | `wc -l access.log` |
| `vim <file>` | Edit in vim (`i`, `Esc`, `:wq`) | `vim index.html` |
| `nano <file>` | Simpler editor (`Ctrl+O`, `Ctrl+X`) | `nano index.html` |
| `grep <pattern> <file>` | Print matching lines | `grep " 500 " access.log` |
| `grep -i` | Case-insensitive search | `grep -i error syslog` |
| `grep -r` | Search recursively | `grep -r "listen 80" /etc/nginx` |
| `grep -n` | Show line numbers | `grep -n "root" site.conf` |
| `find <path> -name <pattern>` | Find files by name | `find /var/log -name "*.log"` |
| `find <path> -type f -size +100M` | Find large files | `find / -type f -size +100M 2>/dev/null` |

**Pipes and filters**

```bash
ps aux | grep nginx
cat /var/log/nginx/access.log | grep " 404 " | wc -l
grep -c "Failed password" /var/log/auth.log
```

`grep -c` counts matches directly; the second pipeline is shown for practice, not because `cat` is required. `grep pattern file` is simpler.

`find / ...` can be slow and permission-noisy. Redirect errors:

```bash
find / -type f -size +500M 2>/dev/null
```

## 4. Users and groups

| Command | What it does | Example |
|---|---|---|
| `whoami` | Current username | `whoami` |
| `id` | UID, GID and groups | `id` |
| `groups [user]` | Group memberships | `groups om` |
| `who` / `w` | Who is logged in | `w` |
| `last` | Recent login history | `last -n 5` |
| `sudo adduser <user>` | Create a user interactively (Ubuntu) | `sudo adduser om` |
| `sudo useradd -m -s /bin/bash <user>` | Create a user with home and shell | `sudo useradd -m rohan` |
| `sudo passwd <user>` | Set/change a password | `sudo passwd rohan` |
| `sudo usermod -aG sudo <user>` | Grant sudo (append!) | `sudo usermod -aG sudo om` |
| `sudo usermod -aG <group> <user>` | Add to another group | `sudo usermod -aG developers rohan` |
| `su - <user>` | Start a login shell as that user | `su - om` |
| `exit` | Leave the current shell | `exit` |
| `sudo userdel -r <user>` | Delete user and home | `sudo userdel -r olduser` |
| `sudo groupadd <group>` | Create a group | `sudo groupadd developers` |
| `sudo groupdel <group>` | Delete a group | `sudo groupdel oldteam` |
| `sudo gpasswd -a <user> <group>` | Add a member | `sudo gpasswd -a lavish developers` |
| `sudo gpasswd -d <user> <group>` | Remove a member | `sudo gpasswd -d lavish developers` |

**Critical:** `usermod -G` without `-a` replaces all supplementary groups. Always use `-aG`.

```bash
# Verify after adding
groups om
su - om
sudo whoami
```

## 5. Permissions and ownership

| Command | What it does | Example |
|---|---|---|
| `ls -l` | Show mode, owner, group | `ls -l` |
| `namei -l <path>` | Show permissions on every path component | `namei -l /var/www/site/index.html` |
| `chmod <mode> <path>` | Change permissions | `chmod 644 index.html` |
| `chmod u+x <file>` | Add execute for owner | `chmod u+x deploy.sh` |
| `chmod g+w <file>` | Add write for group | `chmod g+w shared.txt` |
| `chmod o-rwx <file>` | Remove all access for others | `chmod o-rwx private/` |
| `chmod -R <mode> <dir>` | Apply recursively | `chmod -R 755 site` |
| `chown <user> <path>` | Change owner | `sudo chown om site` |
| `chown <user>:<group> <path>` | Change owner and group | `sudo chown -R om:developers site` |
| `chgrp <group> <path>` | Change group | `sudo chgrp developers site` |
| `umask` | Show default permission mask | `umask` |

**Permission values**

```text
r = 4   w = 2   x = 1

7 = rwx    6 = rw-    5 = r-x    4 = r--
3 = -wx    2 = -w-    1 = --x    0 = ---

755 = rwxr-xr-x    directories served publicly
644 = rw-r--r--    normal files
600 = rw-------    secrets / SSH keys
775 = rwxrwxr-x    shared group directory
2775 = rwxrwsr-x   shared group dir, new files inherit group
```

**Diagnose a permission problem**

```bash
id
ls -ld /var/www /var/www/site
namei -l /var/www/site/index.html
```

Then decide: does the process/user need access, or is the path wrong? Do not default to `777`.

## 6. Packages with apt

| Command | What it does |
|---|---|
| `sudo apt update` | Refresh the package index |
| `sudo apt upgrade` | Install available upgrades for installed packages |
| `sudo apt full-upgrade` | Upgrade, allowing dependency changes |
| `sudo apt install <pkg>` | Install a package and dependencies |
| `sudo apt install -y <pkg>` | Install non-interactively |
| `sudo apt remove <pkg>` | Remove program, keep configuration |
| `sudo apt purge <pkg>` | Remove program and configuration |
| `sudo apt autoremove` | Remove unneeded dependencies |
| `apt show <pkg>` | Show version, size, dependencies |
| `apt list --installed` | List installed packages |
| `apt list --upgradable` | List available upgrades |
| `apt search <term>` | Search package names/descriptions |
| `dpkg -l` | List installed `.deb` packages |
| `dpkg -L <pkg>` | List files installed by a package |

```bash
sudo apt update
sudo apt install nginx
apt show nginx | head -n 20
dpkg -L nginx | head
sudo apt remove nginx
sudo apt purge nginx
sudo apt autoremove
```

On RHEL-family systems, the equivalents are `dnf install`, `dnf remove`, `dnf upgrade`, `dnf list installed`.

## 7. Processes

| Command | What it does | Example |
|---|---|---|
| `ps` | Processes in the current session | `ps` |
| `ps aux` | All processes, all users | `ps aux` |
| `ps -ef` | All processes, full format | `ps -ef` |
| `pgrep -a <name>` | Find PIDs with command line | `pgrep -a nginx` |
| `pstree -p` | Process tree with PIDs | `pstree -p` |
| `top` | Live resource view; `q` quits | `top` |
| `htop` | Friendlier live view; `F6` sort, `F9` signal, `F10` quit | `htop` |
| `uptime` | Load average | `uptime` |
| `free -h` | Memory and swap | `free -h` |
| `kill <pid>` | Send SIGTERM (graceful) | `kill 30414` |
| `kill -9 <pid>` | Send SIGKILL (forced) | `kill -9 30414` |
| `pkill -f <pattern>` | Signal processes matching the full command | `pkill -f "python3 main.py"` |
| `jobs` | Background jobs in this shell | `jobs` |
| `fg %1` / `bg %1` | Foreground / background a job | `fg %1` |
| `command &` | Start in the background | `python3 app.py &` |

**Find, inspect, then stop:**

```bash
pgrep -a python3
ps -fp 30414
kill 30414
sleep 2
pgrep -a python3
```

Use `kill -9` only after graceful termination fails.

## 8. Services with systemctl

| Command | What it does |
|---|---|
| `systemctl status <unit>` | State, PID, recent logs |
| `sudo systemctl start <unit>` | Start now |
| `sudo systemctl stop <unit>` | Stop now |
| `sudo systemctl restart <unit>` | Stop and start |
| `sudo systemctl reload <unit>` | Re-read config without a full restart |
| `sudo systemctl enable <unit>` | Start at boot |
| `sudo systemctl disable <unit>` | Do not start at boot |
| `sudo systemctl enable --now <unit>` | Enable and start |
| `systemctl is-active <unit>` | `active` / `inactive` / `failed` |
| `systemctl is-enabled <unit>` | `enabled` / `disabled` |
| `systemctl list-units --type=service` | List loaded services |
| `sudo systemctl daemon-reload` | Reload unit definitions after editing |
| `journalctl -u <unit> -f` | Follow service logs |
| `journalctl -u <unit> --since "10 min ago"` | Recent logs |
| `journalctl -p err -b` | Errors since boot |

```bash
systemctl status nginx
sudo systemctl reload nginx
systemctl is-enabled nginx
journalctl -u nginx -n 50 --no-pager
```

## 9. Environment variables

| Command | What it does | Example |
|---|---|---|
| `VAR=value` | Create a shell variable (no spaces) | `COURSE=linux` |
| `export VAR=value` | Create an environment variable | `export APP_ENV=staging` |
| `echo "$VAR"` | Print a value | `echo "$HOME"` |
| `printenv` | List environment variables | `printenv` |
| `env` | List or run with a modified environment | `env` |
| `set` | List all shell variables | `set \| less` |
| `unset VAR` | Remove a variable | `unset APP_ENV` |
| `which <cmd>` | Locate a command in `PATH` | `which python3` |
| `type <cmd>` | Builtin, alias or file | `type ls` |
| `command -v <cmd>` | Portable lookup | `command -v nginx` |
| `source <file>` | Execute in the current shell | `source ~/.bashrc` |
| `alias name='cmd'` | Create a shortcut | `alias ll='ls -alF'` |

```bash
echo "$PATH"
export PATH="$PATH:$HOME/bin"
which myscript.sh
```

Persist in `~/.bashrc`, then `source ~/.bashrc`. For services, use systemd environment settings instead.

## 10. Archives and compression

| Command | What it does |
|---|---|
| `tar -cf out.tar dir/` | Create an uncompressed archive |
| `tar -tf out.tar` | List archive contents |
| `tar -xf out.tar` | Extract |
| `tar -xvf out.tar` | Extract verbosely |
| `tar -czf out.tar.gz dir/` | Create a gzip-compressed archive |
| `tar -xzf out.tar.gz` | Extract a `.tar.gz` |
| `tar -tzf out.tar.gz` | List a `.tar.gz` |
| `tar -xzf out.tar.gz -C /tmp/` | Extract into another directory |
| `gzip file` | Compress a file (replaces it with `.gz`) |
| `gzip -k file` | Compress and keep the original |
| `gunzip file.gz` | Decompress |
| `zip -r out.zip dir/` | Create a ZIP archive |
| `unzip out.zip` | Extract a ZIP |
| `unzip -l out.zip` | List a ZIP |
| `sha256sum file` | Checksum for integrity |

```bash
tar -czf ~/backups/site-$(date +%F).tar.gz /var/www/website
tar -tzf ~/backups/site-2026-11-22.tar.gz | head
sha256sum ~/backups/site-2026-11-22.tar.gz
```

## 11. Cron quick reference

```text
* * * * *   command
│ │ │ │ │
│ │ │ │ └── day of week  0–7 (0/7 = Sunday)
│ │ │ └──── month        1–12
│ │ └────── day of month 1–31
│ └──────── hour         0–23
└────────── minute       0–59
```

| Expression | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour at :00 |
| `0 8 * * *` | Daily 08:00 |
| `30 2 * * 1` | Monday 02:30 |
| `0 9 * * 1-5` | Weekdays 09:00 |
| `0 0 1 * *` | First of the month, midnight |
| `0 0 1 1 *` | January 1, midnight |

| Command | Purpose |
|---|---|
| `crontab -l` | List your jobs |
| `crontab -e` | Edit your jobs |
| `crontab -r` | Remove all your jobs (careful) |
| `sudo crontab -e` | Edit root's jobs (only if needed) |
| `systemctl status cron` | Check the scheduler |
| `journalctl -u cron` | Scheduler logs |

Production template:

```cron
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
0 8 * * * /home/om/backup-site.sh >> /home/om/backup.log 2>&1
```

Always use full paths and test the script manually first.

## 12. Disks, memory and system information

| Command | Shows |
|---|---|
| `df -h` | Filesystem usage |
| `du -sh <dir>` | Total size of a directory |
| `du -h --max-depth=1 /var \| sort -h` | Largest items under `/var` |
| `lsblk` | Block devices and mounts |
| `mount \| column -t` | Current mounts |
| `free -h` | RAM and swap |
| `uptime` | Uptime and load average |
| `uname -a` | Kernel and architecture |
| `hostnamectl` | Hostname, OS, kernel |
| `lsb_release -a` | Distribution version |
| `lscpu` | CPU details |
| `dmesg \| tail` | Kernel messages (may need sudo) |

## 13. Networking checks for web servers (supplement)

| Command | Use |
|---|---|
| `ip a` | Interface addresses |
| `ss -tulpn` | Listening TCP/UDP ports and owning processes |
| `curl -I http://localhost` | Response headers from a local server |
| `curl -v http://example.com` | Verbose request/response |
| `ping -c 4 host` | Basic reachability (ICMP may be blocked) |
| `dig example.com` | DNS lookup |
| `traceroute host` | Path (may be blocked) |
| `sudo ufw status verbose` | Firewall rules |
| `nc -zv host 80` | Test a TCP port (if `netcat` installed) |

```bash
ss -tulpn | grep ':80'
curl -I http://127.0.0.1
```

A listening socket plus a local `200` narrows the problem to the firewall or external network.

## 14. File transfer

| Command | Use |
|---|---|
| `scp file user@host:/path/` | Upload one file over SSH |
| `scp -r dir user@host:/path/` | Upload a directory |
| `scp user@host:/path/file .` | Download a file |
| `rsync -avz dir/ user@host:/path/` | Sync changed files |
| `rsync -avz --dry-run dir/ user@host:/path/` | Preview a sync |
| `sftp user@host` | Interactive SFTP shell |

FileZilla: protocol **SFTP**, port **22**, host = IP/domain (no `http://`), username, password or key.

## 15. Practice labs

### Lab 1 — Navigation and file operations (15 minutes)

```bash
cd ~
mkdir -p linux-lab/site/css linux-lab/backup
cd linux-lab/site
touch index.html style.css
printf '<h1>Hello Linux</h1>\n' > index.html
cat -n index.html
cp index.html ../backup/
mv ../backup/index.html ../backup/index-v1.html
ls -l ../backup
cd ../..
pwd
```

**Verify:** you can state the absolute path of every file created, and explain the difference between `cp` and `mv`.

### Lab 2 — Users, groups and shared permissions (20 minutes)

```bash
sudo groupadd projectteam
sudo useradd -m alice
sudo useradd -m bob
sudo passwd alice          # set a practice password
sudo passwd bob
sudo usermod -aG projectteam alice
sudo usermod -aG projectteam bob
groups alice
groups bob

sudo mkdir -p /srv/project
sudo chgrp projectteam /srv/project
sudo chmod 2775 /srv/project
ls -ld /srv/project
```

**Verify:** `alice` and `bob` can both create files in `/srv/project`; files created there inherit the `projectteam` group. Explain why `2775` is better than `777`.

### Lab 3 — Install a service and read its state (15 minutes)

```bash
sudo apt update
sudo apt install -y nginx
systemctl status nginx
curl -I http://localhost
sudo systemctl stop nginx
systemctl is-active nginx
sudo systemctl start nginx
systemctl is-enabled nginx
journalctl -u nginx -n 20 --no-pager
```

**Verify:** you can explain the difference between `active` and `enabled`, and between `restart` and `reload`.

### Lab 4 — Environment variable and cron log (20 minutes)

```bash
export LAB_ENV=staging
echo "$LAB_ENV"
bash -c 'echo "$LAB_ENV"'      # should print staging (exported)
LAB_TEMP=short
bash -c 'echo "$LAB_TEMP"'     # should print nothing (not exported)

mkdir -p ~/cron-lab
cat > ~/cron-lab/stamp.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail
echo "$(date -Is) LAB_ENV=${LAB_ENV:-unset}" >> "$HOME/cron-lab/stamp.log"
SH
chmod +x ~/cron-lab/stamp.sh
~/cron-lab/stamp.sh
cat ~/cron-lab/stamp.log
```

Add a temporary job with `crontab -e`:

```cron
* * * * * /home/<youruser>/cron-lab/stamp.sh
```

Wait two minutes, check the log, then remove the line and verify with `crontab -l`.

**Verify:** the log gains one line per minute. Explain why the script uses absolute paths and `$HOME` rather than a relative path.

### Lab 5 — Deploy, break and repair a site (30 minutes)

Follow the [capstone](Part_06_Study_Notes.md#15-capstone-a-small-production-shaped-server), then deliberately cause each failure and read the evidence:

1. Stop Nginx → connection refused.
2. `sudo chmod 000 /var/www/website/index.html` → 403, read `error.log`.
3. Rename `index.html` → 403/404 depending on config.
4. Change the `root` to a wrong path → 404.
5. Add a syntax error (remove `;`) → `nginx -t` fails; do not reload until fixed.

Restore after each step. **Verify:** you can name the layer and the log/command that revealed it.

### Lab 6 — Archive and restore (15 minutes)

```bash
tar -czf ~/linux-lab.tar.gz ~/linux-lab
tar -tzf ~/linux-lab.tar.gz | head
mkdir -p /tmp/restore-test
tar -xzf ~/linux-lab.tar.gz -C /tmp/restore-test
find /tmp/restore-test -maxdepth 3 -type f
```

**Verify:** the restored tree matches the original. Explain why `tar -tf` should come before `tar -xf` for an unknown archive.

## 16. Curated external resources

These are starting points, not a required syllabus. Prefer official documentation when commands or versions matter.

**Foundations and practice**

- **Linux Journey** — `https://linuxjourney.com` — short, beginner-friendly lessons with exercises.
- **Ubuntu Server documentation** — `https://ubuntu.com/server/docs` — official guides for installation, users, networking, services and security.
- **Ubuntu community terminal guide** — `https://help.ubuntu.com/community/UsingTheTerminal` — gentle command introductions.
- **OverTheWire: Bandit** — `https://overthewire.org/wargames/bandit/` — learn shell commands by solving progressively harder SSH challenges.
- **ExplainShell** — `https://explainshell.com` — paste a command and see each argument explained.
- **GNU Bash manual** — `https://www.gnu.org/software/bash/manual/` — authoritative shell reference.
- **man7.org** — `https://man7.org/linux/man-pages/` — searchable manual pages.

**Administration**

- **systemd manual** — `https://www.freedesktop.org/software/systemd/man/latest/systemctl.html` — exact `systemctl` behaviour and unit options.
- **systemd project** — `https://systemd.io` — concepts, unit files and timers.
- **Filesystem Hierarchy Standard** — `https://www.pathname.com/fhs/` — why directories exist.
- **Debian Administrator's Handbook** — `https://debian-handbook.info` — deeper package, user and service administration.
- **DigitalOcean tutorials** — `https://www.digitalocean.com/community/tutorials` — practical Ubuntu/Nginx/SSH walkthroughs; check the Ubuntu version in each article.

**Web serving and transfer**

- **Nginx documentation** — `https://nginx.org/en/docs/` — directives, server blocks, proxying, TLS.
- **Nginx Beginner's Guide** — `https://nginx.org/en/docs/beginners_guide.html` — the shortest official introduction.
- **Certbot instructions** — `https://certbot.eff.org` — obtain and renew Let's Encrypt certificates.
- **FileZilla documentation** — `https://filezilla-project.org` — client setup and SFTP.
- **OpenSSH manual** — `https://www.openssh.com/manual.html` — `ssh`, `scp`, `sftp`, keys and configuration.

**Cron**

- **crontab.guru** — `https://crontab.guru` — translate and test five-field schedules.
- **cron man page** — `man 5 crontab` on your machine — authoritative field and environment behaviour.

**Security**

- **Ubuntu security notices** — `https://ubuntu.com/security/notices` — why patching matters.
- **Mozilla SSL configuration generator** — `https://ssl-config.mozilla.org` — modern TLS settings for Nginx and others.
- **SSH key best practices** — search the OpenSSH manual for `authorized_keys` restrictions; prefer keys over passwords and disable root password login after testing.

## 17. Glossary

| Term | Meaning |
|---|---|
| **Kernel** | Core of the OS; manages CPU, memory, devices, filesystems and permissions |
| **Distribution** | Kernel plus user-space software, package manager and defaults |
| **Shell** | Program that reads and executes commands (`bash`, `zsh`) |
| **Terminal** | Interface/window carrying text to the shell |
| **VPS** | Virtual Private Server; a rented virtual machine |
| **Hypervisor** | Software that runs and isolates virtual machines |
| **Host / guest** | Physical machine running the hypervisor / the virtual machine |
| **ISO** | Disk image containing an installer or live system |
| **LTS** | Long Term Support release with extended maintenance |
| **SSH** | Encrypted remote login and file-transfer protocol |
| **SFTP** | SSH File Transfer Protocol; encrypted file transfer over SSH |
| **Root** | UID 0; unrestricted administrative user |
| **sudo** | Run one command with elevated privileges, subject to policy |
| **UID / GID** | Numeric user ID / group ID |
| **Primary group** | The group recorded for a user in `/etc/passwd` |
| **Supplementary group** | Additional group membership in `/etc/group` |
| **umask** | Mask controlling default permissions for new files/directories |
| **Process** | Running instance of a program |
| **PID** | Process identifier |
| **Daemon / service** | Long-running background program, usually managed by systemd |
| **systemd** | Init and service manager on most modern distributions |
| **Unit** | systemd definition of a service, timer, mount, socket, etc. |
| **Journal** | systemd's structured log, queried with `journalctl` |
| **Environment variable** | Named value inherited by child processes |
| **PATH** | Colon-separated directories searched for commands |
| **Shebang** | `#!` line naming a script's interpreter |
| **Archive** | One container holding many files (`.tar`) |
| **Compression** | Encoding data to use fewer bytes (`.gz`, `.zip`) |
| **Cron** | Service that runs commands on a schedule |
| **Crontab** | Per-user table of cron jobs |
| **Document root** | Directory from which a web server serves a site |
| **Server block** | Nginx virtual-host configuration |
| **Reverse proxy** | Server that forwards requests to an upstream application |
| **Firewall** | Rules that allow or block network traffic |
| **UFW** | Uncomplicated Firewall, a front end for netfilter on Ubuntu |
| **Load average** | Average number of runnable/uninterruptible processes |
| **FHS** | Filesystem Hierarchy Standard |

## 18. What to do next

1. Finish the [revision quiz](Part_06_Revision_Quiz.md) without notes; review only the sections you miss.
2. Repeat the capstone on a fresh VM without looking at the commands. Aim for a working site, a backup job and a firewall in under 30 minutes.
3. Add one realistic next step: SSH key login, HTTPS with Certbot, a reverse-proxied app, or a systemd service for a small script.
4. Read `man` pages for the five commands you use most. You will discover options that replace fragile one-off recipes.
