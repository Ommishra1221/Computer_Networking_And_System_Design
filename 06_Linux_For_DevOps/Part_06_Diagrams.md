# Part 06 — Visual Companion

Use these diagrams with the [study notes](Part_06_Study_Notes.md). Each one isolates a mechanism that beginners often memorise without understanding. Arrows show control flow, data flow or ownership; they are not all network paths.

## 1. Kernel, operating system and distribution

Read with [What Linux is](Part_06_Study_Notes.md#1-what-linux-is-and-why-it-runs-the-internet).

```text
┌──────────────────────────────────────────────────────────────┐
│                         Your commands                        │
│              terminal → bash → ls / apt / nginx              │
├──────────────────────────────────────────────────────────────┤
│  USER SPACE                                                  │
│  shell · coreutils · package manager · web server · apps    │
│  libraries · desktop environment (optional)                  │
├──────────────────────────────────────────────────────────────┤
│  SYSTEM CALLS: open · read · write · fork · execve · socket  │
├──────────────────────────────────────────────────────────────┤
│  KERNEL                                                      │
│  scheduler · memory manager · filesystems · network stack    │
│  device drivers · permission checks                          │
├──────────────────────────────────────────────────────────────┤
│  HARDWARE: CPU · RAM · disk · NIC · GPU                      │
└──────────────────────────────────────────────────────────────┘

“Linux” strictly = the kernel box.
A distribution  = kernel + user space + package manager + defaults.
```

The kernel does not know what a “website” is. It moves bytes, schedules work and enforces access. Nginx and your application live in user space and use kernel services.

## 2. Three ways to get a Linux machine

Read with [Getting a Linux machine](Part_06_Study_Notes.md#2-getting-a-linux-machine).

```text
OPTION A — VPS                          OPTION B — VM
┌───────────────┐                       ┌────────────────────┐
│ Your laptop   │                       │ Host laptop        │
│  ssh client   │                       │ ┌────────────────┐ │
└───────┬───────┘                       │ │ VirtualBox     │ │
        │ encrypted SSH                 │ │ ┌────────────┐ │ │
        ▼                               │ │ │ Ubuntu VM  │ │ │
┌───────────────┐                       │ │ └────────────┘ │ │
│ Provider      │                       │ └────────────────┘ │
│ physical host │                       └────────────────────┘
│ ┌───────────┐ │                        guest isolated from host
│ │ your VPS  │ │                        but shares real CPU/RAM
│ └───────────┘ │
└───────────────┘                       OPTION C — WSL 2
real public IP, real firewall           ┌────────────────────┐
costs money, closest to production      │ Windows + WSL 2    │
                                        │ ┌────────────────┐ │
                                        │ │ Linux kernel   │ │
                                        │ │ Ubuntu userland│ │
                                        │ └────────────────┘ │
                                        └────────────────────┘
                                        fast, low overhead,
                                        not a full server
```

Whichever route you choose, the shell experience is nearly identical. The differences appear when you test firewalls, public IPs, boot behaviour and `systemd`.

## 3. How one command becomes output

Read with [How the shell reads a command](Part_06_Study_Notes.md#3-how-the-shell-reads-a-command) and [Processes](Part_06_Study_Notes.md#8-processes-and-services).

```text
  you type:  ls -l /var/log | grep nginx > matches.txt
        │
        ▼
┌──────────────────┐
│ shell parses     │  command = ls, args = -l /var/log
│ expansions       │  pipe and redirection recognised
└────────┬─────────┘
         ▼
┌──────────────────┐
│ find executable  │  PATH=/usr/local/bin:/usr/bin:/bin
│                  │  search in order → /usr/bin/ls
└────────┬─────────┘
         ▼
┌──────────────────┐
│ fork + exec      │  shell creates child process
│                  │  child replaces itself with ls
└────────┬─────────┘
         ▼
┌──────────────────┐        ┌──────────────────┐
│ ls asks kernel   │        │ grep child       │
│ to read directory│───────>│ filters lines    │
└──────────────────┘  pipe  └────────┬─────────┘
                                     ▼
                            ┌──────────────────┐
                            │ matches.txt      │
                            │ (shell truncates │
                            │  before running) │
                            └──────────────────┘
```

`>` is handled by the shell, not by `ls`. That is why `sudo echo x > /root/file` can still fail: the shell opens `/root/file` before `sudo` runs. Use `sudo tee /root/file` instead.

## 4. Absolute and relative paths

Read with [Navigating files](Part_06_Study_Notes.md#4-navigating-and-managing-files).

```text
/                         ← root of the filesystem
├── home
│   └── om                ← ~ for user om
│       ├── projects
│       │   └── site
│       └── notes.txt
├── etc
│   └── nginx
└── var
    └── log

From /home/om/projects:

  cd site          relative → /home/om/projects/site
  cd ./site        same, explicitly current directory
  cd ../..         relative → /home
  cd /var/log      absolute → /var/log
  cd ~             home     → /home/om
  cd -             previous directory
```

A leading `/` always means “start at the root”. Everything else is interpreted from the current directory.

## 5. Permission check: one decision, three classes

Read with [Groups and permissions](Part_06_Study_Notes.md#7-groups-and-permissions).

```text
File:  -rwxr-x---  owner: om  group: developers

        ┌── owner? ── yes ──> use rwx  (read, write, execute)
user ───┤
        ├── in group? ─ yes ─> use r-x  (read, execute)
        │
        └── neither ────────> use ---  (no access)

Linux stops at the FIRST matching class.
A group member who is also the owner gets owner permissions.
```

```text
Numeric value:   r = 4    w = 2    x = 1

  7 5 5
  │ │ └── others : 4+1 = r-x
  │ └──── group  : 4+1 = r-x
  └────── owner  : 4+2+1 = rwx

  rwx r-x r-x  =  755
  rw- r-- r--  =  644
  r-- -wx r-x  =  435   (valid, but owner cannot write)
```

| Target | `r` | `w` | `x` |
|---|---|---|---|
| File | Read contents | Change contents | Execute program |
| Directory | List names | Create/delete entries | Enter/traverse |

To delete a file, you need `w` + `x` on its **directory**, not `w` on the file. This is why a read-only file can still be deleted by someone who controls the directory.

## 6. Process and service lifecycle

Read with [Processes and services](Part_06_Study_Notes.md#8-processes-and-services).

```text
LOGIN SHELL (PID 1000)
   │
   ├── fork ──> child PID 1001 ── exec ls ──> exits
   │                                        (short-lived process)
   │
   └── fork ──> child PID 1002 ── exec python app.py
                                            │
                                            ├── running
                                            ├── SIGTERM → cleanup → exit
                                            └── SIGKILL → immediate removal

SYSTEMD (PID 1)
   │
   ├── nginx.service ── master PID 812
   │                      ├── worker 815
   │                      └── worker 816
   ├── cron.service
   └── ssh.service

systemctl start    → ask systemd to start the unit
systemctl stop     → SIGTERM, then SIGKILL after timeout
systemctl restart  → stop + start (brief downtime)
systemctl reload   → re-read config, keep serving
systemctl enable   → create boot-time activation link
```

A process started with `&` belongs to your shell. A service belongs to systemd and survives logout and reboot.

## 7. Shell variables, environment variables and persistence

Read with [Environment variables](Part_06_Study_Notes.md#9-environment-variables-path-and-bashrc).

```text
PARENT SHELL
  NAME=om               shell variable, not inherited
  export FRIEND=rohan   environment variable, inherited

        fork + exec
             │
             ▼
  CHILD PROCESS
    $NAME    → empty
    $FRIEND  → rohan

PERSISTENCE

  ~/.bashrc ── sourced by new interactive shells
      │
      ├── export PATH="$PATH:$HOME/bin"
      ├── export EDITOR=vim
      └── alias ll='ls -alF'

  edit file ──> source ~/.bashrc ──> applies to CURRENT shell
  new login ──> ~/.profile ──> sources ~/.bashrc (Ubuntu)
  service   ──> does NOT read your ~/.bashrc
                use systemd Environment= or EnvironmentFile=
```

```text
PATH search order for `python3`:

/usr/local/sbin → /usr/local/bin → /usr/sbin → /usr/bin → /sbin → /bin
                                                        │
                                                        └── first match wins

which python3  →  /usr/bin/python3
```

## 8. Archive versus compression

Read with [Archives and compression](Part_06_Study_Notes.md#10-archives-and-compression).

```text
BEFORE
python-project/
├── main.py
├── utils.py
├── static/
│   ├── style.css
│   └── logo.png
└── templates/
    └── index.html

ARCHIVING ONLY — tar
┌──────────────────────────────┐
│ python.tar                   │
│ [main.py][utils.py][static/] │
│ [templates/]                 │
└──────────────────────────────┘
one file, roughly same total bytes

COMPRESSION ONLY — gzip
main.py ──> main.py.gz           smaller, still one file

BOTH — tar + gzip, or zip
┌──────────────────────────────┐
│ python-project.tar.gz        │
│ compressed container         │
└──────────────────────────────┘

tar -czf  = create + gzip
tar -xzf  = extract + gunzip
tar -tf   = list without extracting
zip -r    = archive + compress in one step
```

Always list an unknown archive (`tar -tf`, `unzip -l`) before extracting it. Extraction can overwrite files.

## 9. The five cron fields

Read with [Scheduling with cron](Part_06_Study_Notes.md#11-scheduling-with-cron).

```text
         ┌──────── minute       0–59
         │ ┌────── hour         0–23
         │ │ ┌──── day of month 1–31
         │ │ │ ┌── month        1–12
         │ │ │ │ ┌ day of week  0–7 (0 and 7 = Sunday)
         │ │ │ │ │
         * * * * *   command
         │ │ │ │ │
         │ │ │ │ └── */2  → every 2nd weekday value
         │ │ │ └──── 1-5  → Monday to Friday
         │ │ └────── 1    → first day of month
         │ └──────── 8    → 08:00
         └────────── 0    → at minute 0

0 8 * * *  →  daily at 08:00
*/5 * * * * → every 5 minutes
30 2 * * 1 → Monday 02:30
```

```text
TIMELINE (every 5 minutes)

10:00:00  10:05:00  10:10:00  10:15:00
   │         │         │         │
   ▼         ▼         ▼         ▼
 job       job       job       job
   │
   └── runs with a minimal environment:
       short PATH · no ~/.bashrc · no terminal
       → use full paths, redirect output, test manually
```

## 10. Filesystem: one tree, several devices

Read with [The Linux filesystem](Part_06_Study_Notes.md#12-the-linux-filesystem).

```text
/  (root filesystem, mounted from disk A)
├── etc/            configuration
├── home/           user data
│   └── om/
├── usr/            programs and libraries
│   ├── bin/
│   └── local/
├── var/            changing data
│   ├── log/
│   └── www/
├── tmp/            temporary, cleared on boot
├── root/           root user's home (not “/”)
├── proc/           virtual: processes and kernel
├── sys/            virtual: devices and kernel
├── dev/            device files
├── boot/           kernel and bootloader
└── data/  ← mount point for disk B

df -h shows the filesystem behind each mount.
du -sh walks directories and sums file sizes.
```

The tree is logical. `/home` and `/data` can live on different disks, network storage or encrypted volumes; the path does not reveal the device.

## 11. An HTTP request through the server

Read with [Serving sites with Nginx](Part_06_Study_Notes.md#13-serving-sites-with-nginx).

```text
Browser
  │  http://203.0.113.10/
  ▼
┌───────────────────────┐
│ Provider firewall     │  allows 80/443?
│ (security group)      │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ UFW on the server     │  ufw status
│ allows 80/443 + SSH   │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ Nginx :80             │  server_name / default_server
│ server block match    │  root /var/www/website
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ /var/www/website      │  index.html readable by nginx user
└──────────┬────────────┘
           ▼
      200 OK + HTML
```

Failure layers, in order:

```text
DNS → network route → provider firewall → UFW → Nginx listening
    → server block match → document root → file permission → app
```

A `502` means Nginx is reachable but the upstream application failed. A timeout usually means the request never reached Nginx.

## 12. Uploading a file: local, remote and ownership

Read with [Transferring files](Part_06_Study_Notes.md#14-transferring-files).

```text
YOUR LAPTOP                         SERVER
┌──────────────────┐               ┌──────────────────────────┐
│ FileZilla        │  SFTP :22     │ sshd                     │
│ local pane       │──────────────>│                          │
│ ~/Desktop/site   │  encrypted    │ /var/www/website         │
└──────────────────┘               │ owner: root  mode: 755   │
                                   │                          │
                                   │ om cannot write: denied  │
                                   └──────────────────────────┘

FIX OPTIONS
1. upload to /home/om/releases, then deploy with sudo
2. sudo chown -R om:om /var/www/website
3. shared group + setgid:
     groupadd webdeploy
     usermod -aG webdeploy om
     chgrp -R webdeploy /var/www/website
     chmod -R 2775 /var/www/website
4. never: chmod -R 777
```

## 13. The capstone at a glance

Read with [Capstone](Part_06_Study_Notes.md#15-capstone-a-small-production-shaped-server).

```text
1. PROVISION          2. ADMIN USER           3. SOFTWARE
   VPS / VM / WSL        adduser om               apt update
        │                usermod -aG sudo         apt install nginx
        ▼                     │                    │
   ssh root@IP                ▼                    ▼
                         sudo for one-off      systemctl status nginx

4. SITE                5. EXPOSE               6. AUTOMATE
   /var/www/website       ufw allow OpenSSH       backup script
   index.html             ufw allow 'Nginx Full'  crontab -e
   server block           ufw enable              0 8 * * *
   nginx -t                    │                    │
   systemctl reload            ▼                    ▼
        │                 browser test            tar + retention
        └──────────────────────┬──────────────────────┘
                               ▼
                     verify: curl, logs, ls, crontab
```

Every box has a verification command. “It should work” is not verification; a status code, a log line or a file listing is.
