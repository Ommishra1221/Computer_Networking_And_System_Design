# Part 06 — Revision Quiz

Thirty questions across recall, command construction, output reading and troubleshooting. Attempt it without notes first; then mark it with the [answer key](#answer-key) and revisit only the sections you missed.

**Suggested time:** 35 minutes. **Pass mark:** 24/30. A score below 20 means reread the [study notes](Part_06_Study_Notes.md) before attempting the capstone.

## Section A — Multiple choice (10 questions)

**1. Strictly speaking, Linux is:**

- A. A complete desktop operating system
- B. A kernel
- C. A package manager
- D. A distribution

**2. Which command prints the absolute path of your current directory?**

- A. `ls`
- B. `cd`
- C. `pwd`
- D. `whoami`

**3. A file has the mode `-rw-r-----`. Which statement is correct?**

- A. Owner can read/write; group can read; others have no access
- B. Owner can read only; group can write; others can execute
- C. Everyone can read; only the owner can write
- D. Owner can execute; group can read/write; others can read

**4. `chmod 755 website` gives a directory:**

- A. `rwxrwxrwx`
- B. `rwxr-xr-x`
- C. `rw-r--r--`
- D. `r-xr-x---`

**5. Which command adds `om` to the `sudo` group without removing his other supplementary groups?**

- A. `sudo usermod -G sudo om`
- B. `sudo usermod -aG sudo om`
- C. `sudo groupadd sudo om`
- D. `sudo chgrp sudo om`

**6. Which command refreshes the package index without upgrading installed software?**

- A. `sudo apt upgrade`
- B. `sudo apt update`
- C. `sudo apt install --refresh`
- D. `sudo apt autoremove`

**7. A service is `enabled`. What does that mean?**

- A. It is currently running
- B. It will start automatically at boot
- C. It has no configuration errors
- D. It is listening on all interfaces

**8. What does `tar -xzf site.tar.gz` do?**

- A. Create a gzip-compressed archive
- B. List the archive without extracting
- C. Extract a gzip-compressed archive
- D. Compress an existing `.tar` file

**9. The cron expression `30 2 * * 1` runs:**

- A. Every 30 minutes on the first day of the month
- B. At 02:30 every Monday
- C. At 01:30 every day
- D. Every 2 hours and 30 minutes

**10. Which directory primarily holds system-wide configuration files?**

- A. `/home`
- B. `/var`
- C. `/etc`
- D. `/tmp`

## Section B — Write the command (8 questions)

Write one command for each task. Full paths and options matter.

**11.** Create the nested directory `app/static/css` in your home directory, creating any missing parents.

**12.** List all entries in `/var/log`, including hidden files, in long format with human-readable sizes.

**13.** Recursively make `om` the owner and `developers` the group of `/var/www/website`.

**14.** Show the PID and full command line of every process whose name contains `nginx`.

**15.** Follow new lines as they are appended to `/var/log/nginx/error.log`.

**16.** Permanently add `$HOME/bin` to the end of `PATH` for your interactive shells.

**17.** Create a gzip-compressed archive named `site-backup.tar.gz` containing `/var/www/website`.

**18.** Schedule `/home/om/backup-site.sh` to run every day at 08:00, appending its output and errors to `/home/om/backup.log`.

## Section C — Short answers (6 questions)

**19.** What is the difference between `>` and `>>` in the shell?

**20.** Why should a cron job use absolute paths for the program and the script?

**21.** What happens if you run `export PATH="/home/om/bin"` in a normal shell?

**22.** What is the difference between `kill <pid>` and `kill -9 <pid>`?

**23.** What is a shebang, and why does a script need one?

**24.** Why is `/tmp` a bad place for important files?

## Section D — Practical reasoning (6 questions)

**25.** A web directory is owned by `root:root` with mode `755`. User `om` uploads a file with FileZilla and gets “permission denied”. Give two safe ways to solve this, and one unsafe “solution” to avoid.

**26.** Nginx returns `502 Bad Gateway` for a proxied application. Nginx itself is running. What are the first three things you check?

**27.** A cron job is listed in `crontab -l` and scheduled every minute, but no output file appears. List three likely causes and the check for each.

**28.** A site works at `http://203.0.113.10` but not at `http://example.com`. Which layer is most likely wrong, and what would you check?

**29.** Three developers need to edit files in `/var/www/site`. Design a group-and-permissions setup that lets them write without making the directory public. Include the commands.

**30.** You accidentally run `rm -rf` on a project directory on a server. What recovery options exist, and what habits would have prevented the loss?

## Answer key

### Section A

**1. B — kernel.** A distribution combines the kernel with user-space tools, a package manager and defaults. Ubuntu is a distribution. → [Section 1.2](Part_06_Study_Notes.md#12-linux-technically-means-the-kernel)

**2. C — `pwd`.** “Print working directory” returns the absolute path. `ls` lists, `cd` changes, `whoami` shows the user. → [Section 4.1](Part_06_Study_Notes.md#41-where-am-i)

**3. A.** `rw-` owner, `r--` group, `---` others. The first `-` is the file type, not a permission. → [Section 7.2](Part_06_Study_Notes.md#72-read-the-ten-character-permission-string)

**4. B — `rwxr-xr-x`.** 7 = 4+2+1, 5 = 4+1, 5 = 4+1. → [Section 7.6](Part_06_Study_Notes.md#76-numeric-chmod)

**5. B — `usermod -aG`.** `-a` appends; omitting it replaces the supplementary group list. → [Section 5.4](Part_06_Study_Notes.md#54-grant-administrative-access)

**6. B — `apt update`.** It refreshes the catalogue only. `upgrade` installs updates. → [Section 6.2](Part_06_Study_Notes.md#62-refresh-the-index-before-installing)

**7. B — starts at boot.** `active` means running now; `enabled` is a boot-time property. → [Section 8.6](Part_06_Study_Notes.md#86-a-service-is-a-managed-background-program)

**8. C — extract.** `-x` extract, `-z` gzip, `-f` file. `-t` lists, `-c` creates. → [Section 10.2](Part_06_Study_Notes.md#102-tar-basics)

**9. B — Monday 02:30.** Minute 30, hour 2, any day of month, any month, weekday 1 = Monday. → [Section 11.4](Part_06_Study_Notes.md#114-the-five-fields)

**10. C — `/etc`.** Configuration files for the system and services. → [Section 12.2](Part_06_Study_Notes.md#122-the-directories-you-will-actually-use)

### Section B

**11.**

```bash
mkdir -p ~/app/static/css
```

`-p` creates missing parents and does not fail if the final directory exists.

**12.**

```bash
ls -lah /var/log
```

`-l` long, `-a` hidden, `-h` human-readable.

**13.**

```bash
sudo chown -R om:developers /var/www/website
```

`-R` recursive; `owner:group` sets both in one operation. `chown -R om` plus `chgrp -R developers` also works.

**14.**

```bash
pgrep -a nginx
```

`-a` prints the full command line. `ps aux | grep nginx` also works, but may match the `grep` process itself.

**15.**

```bash
tail -f /var/log/nginx/error.log
```

`-f` follows appended data; `Ctrl+C` stops.

**16.**

```bash
echo 'export PATH="$PATH:$HOME/bin"' >> ~/.bashrc
source ~/.bashrc
```

Appending with `>>` preserves the file. `source` applies it to the current shell; new interactive shells read it automatically.

**17.**

```bash
tar -czf site-backup.tar.gz /var/www/website
```

Run with `sudo` if your user cannot read all files. Prefer running from a safe directory; review archive paths before extraction.

**18.**

```cron
0 8 * * * /home/om/backup-site.sh >> /home/om/backup.log 2>&1
```

Minute 0, hour 8, every day. `2>&1` captures errors too. Test the script manually first.

### Section C

**19.** `>` truncates the target and writes stdout from the start; `>>` appends. Both create the file if missing. `>` destroys existing content without asking, so it is a common cause of accidental data loss. → [Section 3.3](Part_06_Study_Notes.md#33-wildcards-and-pipes)

**20.** Cron starts jobs with a minimal environment and a short `PATH`; it does not read your interactive `~/.bashrc`. A program or script found in your shell may not be found by cron. Absolute paths remove that ambiguity. → [Section 11.6](Part_06_Study_Notes.md#116-environment-and-logging)

**21.** You replace the entire search path, so commands such as `ls`, `apt` and `python3` may no longer be found in that shell. Fix it by restoring a standard `PATH` (`export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"`) or opening a new shell, then use `export PATH="$PATH:/home/om/bin"`. → [Section 9.5](Part_06_Study_Notes.md#95-add-a-directory-to-path)

**22.** `kill <pid>` sends SIGTERM (15), which asks the process to terminate and allows cleanup. `kill -9 <pid>` sends SIGKILL, which cannot be caught and removes the process immediately, potentially leaving locks or partial writes. Use `-9` only when graceful termination fails. → [Section 8.4](Part_06_Study_Notes.md#84-stop-a-process)

**23.** A shebang is the first line of a script, beginning `#!`, that names the interpreter, e.g. `#!/usr/bin/env bash`. When the file is executed directly, the kernel uses that interpreter. Without it, the shell may still run the file, but behaviour depends on the caller; with it, execution is predictable. → [Section 9.6](Part_06_Study_Notes.md#96-scripts-shebangs-and-execute-permission)

**24.** `/tmp` is shared, often world-writable with the sticky bit, and may be cleared at boot or by cleanup jobs. It is also a common target for temporary files and can fill up. Important data belongs in a backed-up location such as `/home`, `/srv` or `/var` with appropriate permissions. → [Section 12.2](Part_06_Study_Notes.md#122-the-directories-you-will-actually-use)

### Section D

**25. Safe options:**

- Upload to a directory `om` owns (e.g. `/home/om/releases`), then deploy with `sudo` or a symlink switch.
- Give `om` ownership of the site directory if that is the intended deploy model: `sudo chown -R om:om /var/www/website`.
- Use a shared group with group write and setgid: `groupadd webdeploy`, `usermod -aG webdeploy om`, `chgrp -R webdeploy /var/www/website`, `chmod -R 2775 /var/www/website`.

**Unsafe:** `chmod -R 777 /var/www/website`. It lets every local user and compromised process modify the site. → [Section 14.2](Part_06_Study_Notes.md#142-why-an-upload-can-fail-with-permission-denied)

**26.**

1. Is the application running? `systemctl status <app>` or `pgrep -a`.
2. Is it listening on the address/port used by `proxy_pass`? `ss -tulpn | grep <port>` and `curl http://127.0.0.1:<port>`.
3. Read the app logs and Nginx `error.log`; check the upstream host/port, startup errors, environment variables and whether the app listens on `127.0.0.1` vs `0.0.0.0`.

`502` means Nginx could not get a valid response from the upstream, not that Nginx is down. → [Section 13.7](Part_06_Study_Notes.md#137-common-nginx-errors)

**27.** Any three of:

- Cron service not running → `systemctl status cron`.
- Wrong schedule/time zone or did not wait for a minute boundary → `crontab -l`, `date`.
- Relative path or missing command in cron's `PATH` → use absolute paths, set `PATH`, test the exact command.
- Script not executable / bad shebang → `ls -l`, run it manually.
- Script fails and output is discarded → add `>> log 2>&1`.
- Script waits for input or needs an interactive environment → remove prompts, source a specific env file.
- `%` in the crontab interpreted specially → move the logic into a script or escape it.
- Crontab missing a final newline → end the file with a newline.

→ [Section 11.6](Part_06_Study_Notes.md#116-environment-and-logging)

**28.** DNS (or the server block's `server_name`/default matching). Check `dig example.com` / `dig +short example.com`; confirm the A/AAAA record points to the server IP. Then confirm Nginx has a server block for that name (`nginx -T`) and that you are not hitting a default block. DNS caching can delay a recent change; test with `curl --resolve example.com:80:203.0.113.10 http://example.com/` to separate DNS from Nginx. → [Section 13.5](Part_06_Study_Notes.md#135-add-a-second-site-with-a-server-block)

**29.**

```bash
sudo groupadd webteam
sudo usermod -aG webteam alice
sudo usermod -aG webteam bob
sudo usermod -aG webteam carol
sudo chgrp -R webteam /var/www/site
sudo chmod -R 2775 /var/www/site
```

`2775` gives owner and group `rwx`, others `r-x`; the leading `2` is setgid, so new files and directories inherit the `webteam` group. Ensure the web server can still read the files (often it runs as `www-data`), and have users log out/in for group membership to apply. → [Section 14.2](Part_06_Study_Notes.md#142-why-an-upload-can-fail-with-permission-denied)

**30.** `rm` has no recycle bin. Recovery options: restore from a backup, restore a VM/volume snapshot, or check whether a running process still holds deleted files (`lsof +L1`) — but a directory tree is usually gone. Stop writing to the filesystem to avoid overwriting recoverable blocks, and use file-recovery tools only as a last resort on a mounted filesystem.

Prevention: take a snapshot before risky work, keep automated backups (and test restores), print and inspect the path before `rm -rf`, quote variables, use `--`, prefer an explicit path over a variable, and avoid recursive deletes on production without a change window. → [Section 4.6](Part_06_Study_Notes.md#46-deleting-safely)

## Score and next step

| Score | Interpretation | Next action |
|---|---|---|
| 27–30 | Strong beginner operational grasp | Do the capstone from memory, then add HTTPS or a reverse proxy |
| 24–26 | Passing; a few gaps | Review missed sections and redo the matching lab |
| 20–23 | Fragile | Reread Sections 3–9, then retake the quiz |
| Below 20 | Foundations need another pass | Work through the study notes with a live terminal, one section at a time |

The goal is not the score. It is being able to open a terminal on an unfamiliar server, find the relevant evidence, and make one safe change at a time.
