# Part 06 — Interview Questions

Twelve essential questions cover the beginner-to-junior operations core. Six optional scenarios test whether the same concepts help you reason when something breaks. Answer in your own words and, where useful, write the command you would run.

Use the [study notes](Part_06_Study_Notes.md) for explanations and the [diagrams](Part_06_Diagrams.md) when an answer needs a sketch.

## Essential questions

### 1. What is Linux? Is it an operating system?

Linux is, strictly, a **kernel**: the part of the system that manages CPU scheduling, memory, filesystems, devices and permission checks. When people say “Linux” in everyday speech, they usually mean a **distribution** — the kernel plus GNU utilities, a shell, a package manager, libraries and default configuration. Ubuntu, Debian, Fedora and Alpine are distributions.

The kernel sits between user-space programs and hardware. Applications request work through system calls such as `open`, `read` and `write`. That separation is why the same kernel can run on a server, a phone and a container host.

### 2. Why is Linux important for DevOps and cloud work?

Most cloud instances, container base images and Kubernetes nodes are Linux. Servers are usually accessed through SSH with no graphical desktop, so navigation, permissions, services and logs must be handled from a shell. Linux also provides the kernel features containers rely on, and its package manager and service manager make repeatable automation possible.

The practical skills are: find your way around the filesystem, manage users and permissions, install software, run and supervise services, read logs, schedule work, and expose a service safely.

### 3. Compare a VPS, a virtual machine and WSL for learning Linux.

A **VPS** is a virtual machine rented from a provider. It has a public IP, real SSH access and a real firewall, so it is closest to production; it costs money and is exposed to the Internet. A **VirtualBox VM** runs on your computer and is fully disposable: take a snapshot, break it, restore it. It consumes host RAM/CPU and is not publicly reachable by default. **WSL 2** runs a real Linux kernel and user space inside Windows with low overhead; it is fastest to set up, but networking, `systemd` and boot behaviour differ from a server.

For a first learner, WSL or a VM is safest for command practice; a cheap VPS is best for the deployment capstone.

### 4. What is the Linux filesystem layout? Name the directories that matter.

Linux has a single tree rooted at `/`; disks are mounted at directories inside it. Important paths:

- `/` — root of the filesystem, not the root user's home.
- `/root` — home of the root user.
- `/home/<user>` — regular users' home directories.
- `/etc` — system-wide configuration.
- `/var` — changing data; `/var/log` for logs, `/var/www` for web content on Debian/Ubuntu.
- `/usr` — programs and libraries; `/usr/bin` for most commands.
- `/tmp` — temporary files, often cleared at boot.
- `/boot` — kernel and bootloader.
- `/proc` and `/sys` — virtual filesystems exposing kernel/process state.
- `/dev` — device files.

This layout follows the Filesystem Hierarchy Standard, with distribution-specific additions.

### 5. Explain absolute and relative paths, and the meaning of `.`, `..`, `~` and `/`.

An **absolute path** starts at the root, e.g. `/var/log/nginx`. A **relative path** is interpreted from the current directory, e.g. `nginx` or `../html`.

- `/` — root of the filesystem, also the path separator.
- `~` — the current user's home directory.
- `.` — the current directory.
- `..` — the parent directory.
- `cd -` — the previous directory.

`cd /` goes to the filesystem root; `cd /root` goes to the root user's home. These are different places.

### 6. How do Linux permissions work?

Every file has an owner, a group and a permission triad for owner, group and others. In `ls -l`, `-rwxr-x---` means: regular file, owner `rwx`, group `r-x`, others `---`.

For files, `r` reads contents, `w` changes contents, `x` executes. For directories, `r` lists names, `w` creates/deletes entries, and `x` allows traversal/entry. Deleting a file requires write and execute on its **directory**.

Numeric values are `r=4`, `w=2`, `x=1`, summed per triad: `755` = `rwxr-xr-x`, `644` = `rw-r--r--`, `600` = owner read/write only. Change permissions with `chmod`; change owner/group with `chown` and `chgrp`. Prefer least privilege; `777` is almost always wrong.

### 7. Why shouldn't you work as root? What does sudo do?

Root has UID 0 and bypasses ordinary permission checks. A typo, a malicious script or a compromised application running as root can damage the whole system or expose every file. Daily work should use a normal account.

`sudo` runs one command with elevated privileges after checking `/etc/sudoers`. It usually asks for **your** password, not root's, and caches authorisation briefly. This gives an audit trail and limits the blast radius. Grant access with `usermod -aG sudo <user>` on Ubuntu (or the `wheel` group on RHEL-family systems), and remember that group changes apply to new login sessions.

### 8. What is the difference between a process and a service?

A **process** is a running instance of a program, with a PID, memory, an owner and open files. A **service** (daemon) is a long-running background program managed by a service manager, usually **systemd**. A service is started at boot, supervised, restarted according to policy, and controlled through a standard interface.

Inspect processes with `ps aux`, `top` or `htop`; find a PID with `pgrep -a`. Stop gracefully with `kill <pid>` (SIGTERM); use `kill -9` only when necessary. Manage services with `systemctl status|start|stop|restart|reload|enable|disable <unit>`, and read logs with `journalctl -u <unit>`.

`restart` stops and starts the process (brief downtime). `reload` re-reads configuration without dropping the service, when supported — for example `systemctl reload nginx`.

### 9. What is the difference between a shell variable and an environment variable? What is PATH?

A **shell variable** exists only in the current shell. An **environment variable** is exported and inherited by child processes.

```bash
NAME=om                 # shell variable
export FRIEND=rohan     # environment variable
bash -c 'echo "$FRIEND"'  # rohan
```

`PATH` is a colon-separated list of directories the shell searches for a command. If a program is not in `PATH`, you get `command not found`; run it with `./program` or an absolute path. `which`/`command -v` shows which file would run.

Add a directory with `export PATH="$PATH:$HOME/bin"`. To persist it, put the line in `~/.bashrc` and run `source ~/.bashrc`. Do not write `export PATH="/new/path"`, which removes the existing search path.

### 10. How would you install Nginx and serve a static site on Ubuntu?

```bash
sudo apt update
sudo apt install nginx
systemctl status nginx
sudo mkdir -p /var/www/website
sudo chown -R "$USER":"$USER" /var/www/website
# add index.html
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/website
# edit root and server_name
sudo ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
sudo nginx -t
sudo systemctl reload nginx
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

The server block maps `server_name` and port 80 to a document root. `nginx -t` validates syntax before a reload. The firewall must allow 80/443, and the provider may have a second firewall. Test with `curl -I http://localhost` and read `/var/log/nginx/error.log` if the response is wrong.

### 11. How does cron work, and why do cron jobs often fail?

`cron` is a service that runs scheduled commands from per-user crontabs. Each job has five fields: minute, hour, day of month, month, day of week, followed by the command.

```cron
0 8 * * * /home/om/backup.sh >> /home/om/backup.log 2>&1
```

Common failures: cron service is not running; the schedule is wrong; the script is not executable; relative paths do not resolve; the job relies on `~/.bashrc` or an interactive environment; output is discarded and the failure is invisible.

Use full paths, test the script manually, redirect stdout/stderr to a log, and check `systemctl status cron` plus `journalctl -u cron`. Wait for a minute boundary when testing.

### 12. How do you transfer files to a Linux server securely?

Use **SFTP** (FileZilla, port 22) or `scp`/`rsync` over SSH. All are encrypted. Plain FTP sends credentials and data in clear text and should not be used on the public Internet.

```bash
scp index.html om@203.0.113.10:/var/www/website/
rsync -avz website/ om@203.0.113.10:/var/www/website/
```

A common upload failure is **permission denied** because the remote directory is owned by root. Fix it by uploading to a user-owned directory and deploying with privilege, changing ownership to the deploy user, or using a shared group with setgid and group write. Do not use `chmod 777`.

## Optional scenarios

### 13. A site hosted on your server is unreachable. Walk through your diagnosis.

Check layers in order, and stop when a layer fails:

1. **DNS** — does the domain resolve to the right IP? `dig example.com`.
2. **Network/provider firewall** — is the IP reachable, and is the cloud security group allowing 80/443?
3. **Host firewall** — `sudo ufw status verbose`.
4. **Nginx process** — `systemctl status nginx`.
5. **Listening socket** — `ss -tulpn | grep ':80'`.
6. **Local response** — `curl -I http://127.0.0.1`.
7. **Server block** — does `server_name`, `listen` and `root` match? `nginx -T`.
8. **Files and permissions** — does the index file exist and is it readable by the Nginx user? `namei -l`.
9. **Logs** — `/var/log/nginx/error.log`, `journalctl -u nginx`.

A timeout usually means the request never reached Nginx. A `403` means Nginx reached the file layer. A `502` means Nginx is fine but the upstream application is not.

### 14. A teammate fixes an upload problem with `chmod -R 777 /var/www/site`. What is wrong, and what would you do instead?

`777` lets every local user and every compromised process read, modify and execute the site's files. On a web server, a small upload or plugin bug can become code execution. It also hides the real ownership problem.

Instead:

- Identify the user the web server runs as (`www-data` on Ubuntu) and the deploy user.
- Give the deploy user ownership or a shared group with `2775`.
- Keep files `644` and directories `755` where possible.
- Use `sudo` for a privileged deploy step, or deploy to a user-owned release directory and switch a symlink.
- Verify with `namei -l` and test as the actual service user.

### 15. A backup script works when you run it, but the cron job never produces a file. Why?

Cron runs with a minimal environment and no terminal. Likely causes:

- The script uses a relative path; cron's working directory is the user's home (and may differ from your shell).
- A command is in `/usr/local/bin`, which is not in cron's default `PATH`.
- The script is not executable, or the shebang is wrong.
- The schedule is wrong, or cron is not running.
- The script waits for input, or fails and its output is discarded.
- `%` characters in the crontab are interpreted specially.
- You did not wait for the next minute.

Fix by using absolute paths, setting `PATH` or sourcing an environment file, testing the exact command as the cron user, and redirecting `>> log 2>&1`. Check `crontab -l`, `systemctl status cron` and `journalctl -u cron`.

### 16. You start an app with `python3 app.py &` and it dies when you close the terminal. How should it run?

Background jobs are children of your shell and may receive `SIGHUP` when the session ends. Use a service manager so the app is supervised and started at boot.

Create a systemd unit with `User=`, `WorkingDirectory=`, `ExecStart=` (absolute paths), `Restart=on-failure` and `WantedBy=multi-user.target`. Then `daemon-reload`, `enable --now`, and check `systemctl status` and `journalctl -u`. For a quick interactive session, `tmux` or `screen` keeps a terminal alive, but it is not a production supervisor.

### 17. After enabling UFW, you can no longer SSH into the server. What happened, and how do you recover?

You enabled a default-deny firewall without allowing the SSH port (or you allowed the wrong port). Existing connections may survive briefly, but new ones are blocked.

Recover through the provider's web console or recovery mode, then:

```bash
sudo ufw allow OpenSSH
# or, if SSH runs on a custom port:
sudo ufw allow 2222/tcp
sudo ufw status verbose
```

If needed, `sudo ufw disable` restores access, but re-enable it after adding the correct rule. The permanent lesson: allow SSH **before** `ufw enable`, verify from a second terminal, and keep console access available.

### 18. A server is slow and `df -h` shows 100% on `/`. What do you check, and what do you avoid?

Identify what filled the disk:

```bash
df -h
sudo du -h --max-depth=1 / | sort -h
sudo find /var/log -type f -size +100M -ls
sudo lsof +L1          # deleted files still held open
```

Common causes are runaway logs, old backups, package caches, container images and deleted-but-open files. Fix by rotating/compressing logs, removing old archives, clearing caches deliberately, or adding space. Do not blindly delete files under `/var/lib` or `/etc`; a database file or configuration may be inside. Check whether a process is still writing to a deleted file before assuming space will be freed.
