# Part 06 — Linux for DevOps

**Lecture:** 00:00–2:27:42 — what Linux is → obtaining a machine → basic commands → users → packages → permissions → processes/services → environment → archives → cron → filesystem → Nginx → file transfer.

Linux is not one program you memorise. It is an operating system family you learn to *operate*: find where you are, understand who is allowed to do what, start and stop work, and keep a service running after you close the terminal. This part builds that operating model in the same order as the lecture, with extra basics wherever a beginner would otherwise have to guess.

> **How to read the command examples.** A line beginning with `$` is a normal-user shell prompt; `#` means a root shell. The text after `#` in a command block is a comment, not something to type. Outputs are illustrative and shortened; versions, dates and addresses on your machine will differ.

## 1. What Linux is, and why it runs the Internet

### 1.1 A computer needs a manager

The hardware in front of you can store bytes and execute instructions, but it cannot decide *which* program runs next, *which* memory belongs to which process, or *who* may read a file. An **operating system (OS)** provides those decisions and the interfaces that programs use.

The core of the operating system is the **kernel**. It sits between software and hardware and manages:

| Resource | What the kernel decides |
|---|---|
| CPU time | Which process runs, for how long, on which core |
| Memory | Which process gets which pages of RAM, and what happens on a page fault |
| Storage | How files and directories map to blocks on a device |
| Devices | How drivers talk to disks, network cards, keyboards and GPUs |
| Processes | How programs are created, scheduled, signalled and terminated |
| Security | Which user or process may access a resource |

Above the kernel sits **user space**: the shell, utilities, package manager, web servers and your applications. A program requests kernel work through **system calls** such as `open`, `read`, `write`, `fork` and `execve`. You rarely call these directly, but every command you type eventually becomes one.

```text
┌─────────────────────────────────────────────────────┐
│  You: terminal → shell → commands and applications  │
├─────────────────────────────────────────────────────┤
│  User space: bash, ls, apt, nginx, python, your app │
├─────────────────────────────────────────────────────┤
│  System-call interface (open, read, write, fork…)   │
├─────────────────────────────────────────────────────┤
│  Kernel: scheduler, memory, filesystems, drivers,   │
│          networking, permission checks              │
├─────────────────────────────────────────────────────┤
│  Hardware: CPU, RAM, disk, NIC, GPU                  │
└─────────────────────────────────────────────────────┘
```

### 1.2 “Linux” technically means the kernel

Strictly speaking, **Linux is a kernel**, not a complete operating system. When people say “I installed Linux”, they normally mean a **distribution**: the Linux kernel plus a large collection of software that makes a usable system.

A distribution usually contains:

- the Linux kernel;
- the **GNU** user-space tools (`bash`, `coreutils`, `grep`, and so on);
- a package manager and software repositories;
- an init/service manager (usually **systemd**);
- libraries and compilers;
- optionally a desktop environment, browser and office software.

That is why the same kernel can power a tiny server with no screen and a full desktop with a graphical installer. The kernel is shared; the surrounding choices are not.

**Common distributions and families**

| Distribution | Family / package tool | Typical use |
|---|---|---|
| Ubuntu | Debian / `apt` | Beginner servers, cloud, desktops |
| Debian | Debian / `apt` | Stable servers, base for Ubuntu |
| Linux Mint | Ubuntu / `apt` | Desktop users coming from Windows |
| Fedora | Red Hat / `dnf` | Recent software, developer desktops |
| RHEL, Rocky, AlmaLinux | Red Hat / `dnf` | Enterprise servers |
| Arch Linux | Independent / `pacman` | Advanced users who want control |
| Kali Linux | Debian / `apt` | Security testing (not a general server choice) |
| Alpine | Independent / `apk` | Very small containers |

The lecture uses **Ubuntu**, so the examples use `apt`, `sudo`, `systemctl` and the Debian-style Nginx layout. Most concepts transfer; package commands and configuration paths differ on other families.

### 1.3 The shell is your control surface

A **shell** is a program that reads commands, expands them, runs programs and shows output. `bash` (Bourne Again SHell) is the default on most Linux systems. A **terminal** is the window or connection that carries text to the shell. The terminal is the display; the shell is the interpreter.

This distinction matters when you troubleshoot: a “terminal not working” problem may be a connection issue, while a “command not found” problem is usually the shell's `PATH`.

### 1.4 A short history that explains today's design

| Year | Event | Why it still matters |
|---|---|---|
| 1969 | Ken Thompson and Dennis Ritchie begin **Unix** at Bell Labs | Files, processes, pipes and “small tools” became the model Linux inherited |
| 1983 | Richard Stallman starts the **GNU** project | Provided the free compiler, shell and utilities that Linux needed to be usable |
| 1987 | Andrew Tanenbaum releases **MINIX** for teaching | Linus Torvalds used and studied it |
| 1991 | Linus Torvalds releases the first Linux kernel, University of Helsinki | A free kernel that could run on ordinary PCs, with source available |
| 1990s | Distributions form (Slackware, Debian, Red Hat, later Ubuntu) | Kernel + tools + package management became a complete system |
| 2003–2005 | Android Inc. builds on the Linux kernel; Google acquires it in 2005 | Most phones run a Linux-based kernel |
| 2000s–2020s | Linux dominates servers, cloud and supercomputers | Most cloud instances, containers and AI infrastructure run Linux |

Two corrections to the popular telling:

- **Unix did not disappear.** macOS is a Unix-derived system, and Linux is “Unix-like” rather than a direct descendant. The shared ideas — files, processes, permissions, pipes — are why terminal skills transfer.
- **Open source is not automatically secure.** Anyone can inspect the code, but bugs still exist, distributions still ship updates, and a badly configured server is insecure regardless of licence. Linux's security reputation comes from its permission model *plus* disciplined patching and configuration.

### 1.5 Why DevOps, cloud and AI work all assume Linux

- Most cloud virtual machines and container base images are Linux.
- Kubernetes runs containers on Linux nodes; Docker's core features depend on Linux kernel facilities.
- Web servers, databases, message queues and CI runners are commonly Linux services.
- AI training and inference clusters usually run Linux, often with NVIDIA drivers and container toolkits.
- Server access is often **SSH only** — no desktop, no mouse, just a shell.

You do not need to be a kernel developer. You do need to answer four operational questions quickly:

1. Where am I, and what is here?
2. Who am I, and what am I allowed to do?
3. What is running, and how do I control it?
4. How do I make this repeatable and survive a reboot?

Every section below builds one of those answers.

## 2. Getting a Linux machine

You have three sensible options. All three give you a shell; they differ in cost, isolation and how close they are to production.

### 2.1 Option A — Rent a VPS (closest to real work)

A **VPS (Virtual Private Server)** is a virtual machine rented from a hosting provider. It runs on shared physical hardware, but you receive your own operating-system instance, IP address and root access. “Cloud server” is a similar idea with provider-specific management features.

A typical setup flow, as in the lecture:

1. Create an account with a provider and choose a plan. For practice, 1–2 vCPUs and 2–8 GB RAM is plenty. The lecture recommends a mid-tier plan so the machine feels responsive.
2. Choose **Ubuntu LTS**. **LTS** means Long Term Support: security and maintenance updates for a longer period than an interim release. Prefer it for servers.
3. Choose the data-centre region closest to you or your users.
4. Set a **root password** or, better, add an SSH key during creation. Save the credentials in a password manager.
5. Wait for provisioning, then copy the provider's SSH command.

The command looks like this:

```bash
ssh root@203.0.113.10
```

- `ssh` — Secure Shell, an encrypted remote login protocol.
- `root` — the remote username.
- `203.0.113.10` — the remote IP address. (`203.0.113.0/24` is reserved for documentation, so it is safe in notes.)

The first connection asks you to confirm the host's fingerprint:

```text
The authenticity of host '203.0.113.10' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` only if this is the first connection to a server you just created, or if the fingerprint matches one shown in the provider's console. On later connections, a changed fingerprint is a warning worth investigating, not a prompt to dismiss.

Then enter the password. Password characters are not displayed — not even asterisks. That is normal.

A successful login shows a prompt like:

```text
root@ubuntu-s-1vcpu-1gb:~#
```

This tells you: user `root`, hostname `ubuntu-s-1vcpu-1gb`, current directory `~` (root's home, `/root`), and `#` meaning an administrative shell.

**VPS advantages:** reachable from anywhere, realistic remote administration, can host a real site, survives your laptop closing.
**VPS cautions:** it costs money, it is exposed to the Internet, and a mistake can delete a real server. Use a strong password or SSH key, keep it patched, and do not store irreplaceable data without backups.

### 2.2 Option B — VirtualBox on Windows or macOS (best for safe practice)

**Virtualisation** uses software to create a virtual computer that borrows CPU, RAM and disk from a **host** machine. The virtual machine (**guest**) has its own operating system and believes it is a separate computer. A **hypervisor** manages this isolation; VirtualBox is a Type-2 hypervisor that runs on top of your desktop OS.

```text
Host laptop (Windows/macOS)
├── VirtualBox application
│   └── Ubuntu guest VM
│       ├── virtual CPU cores
│       ├── virtual RAM
│       ├── virtual disk (a large file on the host)
│       └── virtual network adapter
└── Your normal desktop, files and browser
```

The guest is isolated: deleting its disk file does not delete host files, and experiments stay contained. It is not magic — the guest still consumes real RAM and CPU, and cannot exceed the host's capacity.

**Step-by-step (Windows or Mac):**

1. Download VirtualBox from `virtualbox.org` and install it. On Windows, accept the network-interface warning; VirtualBox installs a virtual network driver.
2. Download an Ubuntu **Desktop ISO** from `ubuntu.com`. An **ISO** is a disk-image file containing the installer and live system.
3. Open VirtualBox → **New**.
4. Name the VM (for example `ubuntu-lab`), select the Ubuntu ISO, and choose **Skip Unattended Installation** if you want to see the normal installer.
5. Allocate memory. Give the VM **at least 4 GB**, ideally 8 GB if your host has 16 GB. Do not give it more than half your host RAM.
6. Allocate CPU cores. Two is enough; four is comfortable. Do not assign all host cores.
7. Create a virtual disk. **20–25 GB** is enough for this course; dynamically allocated disks grow as needed.
8. Start the VM. Select the ISO if prompted, choose **Try or Install Ubuntu**, and follow the installer.
9. Choose language, keyboard, normal installation, and **Erase disk and install Ubuntu**. This erases the *virtual* disk only — not your host disk — but read the screen carefully to be sure you are inside the VM.
10. Create a username and password. Use a practice password you can remember; this is a lab machine.
11. Reboot, remove/eject the ISO when prompted, and log in.
12. Open the terminal: press `Ctrl+Alt+T`, or search “Terminal” in Activities.

**Mac notes:** the workflow is identical. On Apple Silicon, use an ARM64 Ubuntu image and a VirtualBox version that supports it; if VirtualBox is unreliable, UTM or VMware Fusion are alternatives. On Intel Macs, the standard AMD64 image works.

**Make the VM pleasant:** install **Guest Additions** (Devices → Insert Guest Additions CD) for better display scaling, shared clipboard and shared folders. If graphics are slow, enable 3D acceleration and give the VM more video memory. First boot is often slow because the installer is still finishing background work.

**Snapshots** are the real superpower of a VM. Take one after installation and before risky experiments. If you break the system, restore the snapshot instead of reinstalling. This is why a VM is the best place to practise `rm -rf`, permissions and firewall changes.

### 2.3 Option C — WSL 2 on Windows (fastest, with caveats)

**WSL (Windows Subsystem for Linux)** runs a real Linux kernel in a lightweight virtualised environment inside Windows. You get a Linux user space and terminal without installing a full desktop VM.

Requirements: Windows 10 version 2004+ or Windows 11, virtualisation enabled in firmware, and administrator access.

```powershell
# In an Administrator PowerShell or Windows Terminal
wsl --install
```

This installs WSL and a default Ubuntu distribution. Reboot when asked. On first launch, create a Linux username and password; this account is separate from your Windows account.

Useful commands (run in PowerShell/CMD, not inside Linux):

```powershell
wsl --list --online          # show installable distributions
wsl --install -d Ubuntu      # install a specific distribution
wsl --list --verbose         # show installed distributions and WSL version
wsl --set-default-version 2  # make WSL 2 the default
wsl --shutdown               # stop all WSL instances
```

Inside the Ubuntu terminal, normal Linux commands work. Your Windows drives are mounted under `/mnt/c`, `/mnt/d`, and so on.

```bash
cd /mnt/c/Users/YourName/Desktop
ls
cd ~
```

**WSL cautions:**

- File access across `/mnt/c` is slower than inside the Linux filesystem. Keep projects in `~/projects` for performance.
- Services and networking behave differently from a real server. A web server started in WSL may be reachable from Windows via `localhost`, but firewall and boot behaviour are not the same as a VPS.
- `systemd` support depends on the WSL version and configuration. On current WSL 2 it can be enabled in `/etc/wsl.conf`; if `systemctl` says the system has not been booted with systemd, that is why.
- WSL is excellent for learning commands, Git, Python and Node. It is not a substitute for practising SSH, firewalls and a public web server.

### 2.4 Choosing between them

| | VPS | VirtualBox VM | WSL 2 |
|---|---|---|---|
| Realistic server | Highest | Medium | Low–medium |
| Safe to break | No (costs money) | Yes (snapshot) | Mostly yes |
| Setup time | Minutes | 30–60 minutes | Minutes |
| Works offline | No | Yes | Yes |
| Resource cost | None locally | High (RAM/CPU/disk) | Low |
| Public IP / real firewall | Yes | No (NAT by default) | No |
| Best for | Deployment practice | Full Linux + desktop | Quick command practice |

A good combination is WSL or a VM for daily practice and a cheap VPS for the deployment capstone.

### 2.5 First five checks on any new machine

```bash
whoami        # which user am I?
pwd           # which directory am I in?
hostnamectl   # hostname and OS information
df -h         # disk space
free -h       # memory
```

If these work, the shell and basic system tools are alive. If `sudo` is missing, install it or use the provider's root console.

## 3. How the shell reads a command

Before memorising commands, learn the grammar. Nearly every command is:

```text
command [options] [arguments]
```

```bash
ls -l /var/log
│  │   └── argument: what to operate on
│  └────── option: modifies behaviour
└───────── command: the program to run
```

Options often have a short form (`-l`) and a long form (`--long`). They can usually be combined: `ls -lah` means `-l -a -h`.

### 3.1 Absolute and relative paths

- An **absolute path** starts at the root: `/home/om/projects`.
- A **relative path** starts from your current directory: `projects`, `./projects`, `../logs`.

Special path pieces:

| Symbol | Meaning |
|---|---|
| `/` | The root of the filesystem (also the separator between directories) |
| `~` | Your home directory (`/home/om`, or `/root` for root) |
| `.` | The current directory |
| `..` | The parent directory |
| `-` | The previous directory, when used with `cd -` |

The same slash means two things depending on position. In `/home/om`, the first `/` is the root; the others separate names. In `cd /`, the argument is the root itself.

### 3.2 Case, spaces and quoting

Linux filenames are **case-sensitive**: `Notes.txt`, `notes.txt` and `NOTES.TXT` are three different files. Windows is usually case-insensitive; do not carry that habit over.

Spaces separate arguments, so a filename with a space must be quoted or escaped:

```bash
mkdir "my first website"
cd "my first website"
cd my\ first\ website      # equivalent
```

Use single quotes when you want the literal text, and double quotes when you still want variable expansion:

```bash
echo '$HOME'    # prints $HOME
echo "$HOME"    # prints /home/om
```

### 3.3 Wildcards and pipes

| Pattern | Matches |
|---|---|
| `*` | Any number of characters (within a directory level) |
| `?` | Exactly one character |
| `[abc]` | One character from the set |
| `[0-9]` | One digit |

```bash
ls *.log          # every .log file in this directory
rm report-?.txt   # report-1.txt, report-a.txt, but not report-10.txt
```

The **pipe** `|` connects one command's output to the next command's input:

```bash
ps aux | grep nginx
```

`ps aux` produces a table; `grep nginx` keeps only lines containing `nginx`. This “small tools, connected” pattern is central to Linux.

**Redirection** sends output to a file instead of the screen:

```bash
command > file.txt      # overwrite file with stdout
command >> file.txt     # append stdout
command 2> error.txt    # redirect stderr only
command > all.txt 2>&1  # stdout and stderr to the same file
command < input.txt     # read stdin from a file
```

`>` overwrites without asking. There is no recycle bin. Be careful with `>` when you meant `>>`.

### 3.4 Getting help without memorising everything

```bash
man ls            # full manual page; q to quit
ls --help         # short usage summary
apropos permission # search manual descriptions
```

Read the **synopsis** first, then examples. Manual pages are dense but authoritative. `man man` explains how to navigate them.

### 3.5 Exit status: did the command succeed?

Every command returns a number from 0 to 255. `0` means success; non-zero means some kind of failure.

```bash
ls /does-not-exist
echo $?           # prints a non-zero value
```

This is how scripts make decisions. In a chain, `&&` runs the next command only on success, while `;` runs it regardless:

```bash
mkdir backup && cd backup      # cd only if mkdir worked
```

## 4. Navigating and managing files

### 4.1 Where am I?

```bash
pwd
```

`pwd` = **print working directory**. It prints the absolute path of the directory you are in.

```text
om@ubuntu:~$ pwd
/home/om
```

### 4.2 What is here?

```bash
ls            # names in the current directory
ls -l         # long format: type, permissions, owner, group, size, date, name
ls -a         # include hidden entries (names beginning with .)
ls -lah       # long + hidden + human-readable sizes
ls /var/log   # list a different directory
```

Example long listing:

```text
drwxr-xr-x 2 om developers 4096 Nov 20 10:14 projects
-rw-r--r-- 1 om om          214 Nov 20 10:10 notes.txt
```

The first character tells you the type: `d` directory, `-` regular file, `l` symbolic link. The next nine characters are permissions, explained in Section 7.

On Ubuntu, `ls` may colour directories blue, executables green and archives red. Colour is a convenience, not data; do not rely on it over SSH or in scripts.

### 4.3 Moving around

```bash
cd projects           # relative: enter projects inside the current directory
cd /var/www/html      # absolute: go exactly here from root
cd ..                 # parent directory
cd ../..              # two levels up
cd ~                  # home directory
cd                    # also home
cd -                  # previous directory
cd /                  # filesystem root
```

From the root, `cd ..` keeps you at `/`; there is nothing above it.

**Tab completion** is the single biggest time-saver in the shell. Type the first few characters and press `Tab`:

```bash
cd /ho<Tab>           # expands to /home/
cd /home/o<Tab>       # expands to /home/om/ if unique
```

If nothing happens, the prefix is ambiguous; press `Tab` twice to see the candidates. Tab completion also reduces typos in long paths.

### 4.4 Creating directories and files

```bash
mkdir website                   # one directory
mkdir -p website/css/images     # create parents as needed
mkdir -p "my first website"     # quoted space
touch index.html                # create an empty file (or update its timestamp)
touch style.css app.js          # several files at once
```

`mkdir -p` is idempotent: if the directory already exists, it does not fail. That makes it useful in scripts.

### 4.5 Copying, moving and renaming

```bash
cp notes.txt notes.backup.txt           # copy a file
cp notes.txt /tmp/                      # copy into a directory
cp -r website website-copy              # copy a directory recursively
cp -i notes.txt notes.backup.txt        # ask before overwriting
mv notes.txt archive/                   # move into a directory
mv old-name.txt new-name.txt            # rename
mv website /var/www/                    # move a directory
```

`cp` creates a second copy; `mv` changes the location or name and leaves one entry. `mv` also overwrites by default; `-i` asks first.

### 4.6 Deleting safely

```bash
rm notes.txt            # delete a file
rm -i notes.txt         # ask first
rm -r old-folder        # recursively delete a directory and its contents
rmdir empty-folder      # remove an empty directory only (safer)
```

`rm` is permanent. `rm -rf` means “recursive, force, no questions”:

```bash
rm -rf /home/om/projects   # deletes the whole tree
```

Never run `rm -rf` with a variable you have not checked, a trailing space before a slash, or a path you have not read twice. A classic disaster is `rm -rf $DIR/` when `DIR` is empty, which becomes `rm -rf /`. Defensive habits:

```bash
echo "$DIR"                 # print first
ls -la "$DIR"               # inspect first
rm -rf -- "$DIR"            # -- ends option parsing
```

There is no undo and no recycle bin on a server. Backups and snapshots are the real safety net.

### 4.7 Viewing files

```bash
cat notes.txt             # print the whole file
cat -n notes.txt          # number the lines
less /var/log/syslog      # page through a large file
head -n 20 notes.txt      # first 20 lines
tail -n 20 notes.txt      # last 20 lines
tail -f /var/log/syslog   # follow new lines as they are written (Ctrl+C to stop)
wc -l notes.txt           # count lines
```

`less` navigation: `Space` next page, `b` previous page, `/pattern` search forward, `n` next match, `q` quit.

`cat` is fine for small files. Using `cat` on a 500 MB log floods your terminal; use `less`, `head`, `tail` or `grep`.

### 4.8 Editing with vim

`vim` is a terminal text editor with modes. The two you need first:

- **Normal mode** — navigation and commands (the starting mode).
- **Insert mode** — typing text.

```bash
vim index.html
```

Inside `vim`:

1. Press `i` to enter insert mode. The bottom line shows `-- INSERT --`.
2. Type your text.
3. Press `Esc` to return to normal mode.
4. Type `:wq` and press `Enter` to write and quit.
5. To leave without saving: `Esc`, then `:q!`, `Enter`.

Useful normal-mode commands:

| Key | Action |
|---|---|
| `i` | Insert before the cursor |
| `a` | Insert after the cursor |
| `o` | Open a new line below and insert |
| `Esc` | Leave insert mode |
| `:w` | Save |
| `:q` | Quit (fails if unsaved changes) |
| `:wq` or `ZZ` | Save and quit |
| `:q!` | Quit without saving |
| `dd` | Delete the current line |
| `yy` / `p` | Copy the current line / paste below |
| `/word` | Search forward |
| `:set nu` | Show line numbers |

The mode system feels strange for ten minutes and then becomes fast. If you get stuck, press `Esc` twice and type `:q!`.

**Beginner alternative:** `nano` is a simpler editor. Use `nano file`, edit, then `Ctrl+O`, `Enter` to save and `Ctrl+X` to exit. It is fine to use `nano` while learning; `vim` is worth learning because it is present on almost every server.

### 4.9 Clearing and recalling

```bash
clear        # clear the screen (Ctrl+L also works)
history      # numbered list of previous commands
history 20   # last 20 commands
!!           # repeat the previous command
!42          # repeat history item 42
```

The `Up` and `Down` arrows walk through history. `Ctrl+R` searches history interactively; press `Ctrl+R` again to cycle matches, `Enter` to run, or `Ctrl+G` to cancel.

### 4.10 Lab — build a small workspace

```bash
cd ~
mkdir -p lab/site/css lab/backup
cd lab/site
touch index.html
vim index.html          # add: <h1>Hello Linux</h1>, save with :wq
cat -n index.html
cp index.html ../backup/index.html
mv ../backup/index.html ../backup/index-v1.html
ls -l ../backup
cd ..
pwd
```

You should now be able to explain every path in those commands. If not, reread Sections 3.1 and 4.3 before continuing; everything later depends on this.

## 5. Users, root and sudo

### 5.1 Why not work as root?

Linux is a **multi-user** system. Every process runs as a user, and every file has an owner. The **root** user has UID 0 and bypasses ordinary permission checks. Root can delete the kernel, rewrite firewall rules, reset any password and read every file.

That power is necessary for administration but dangerous for daily work:

- A typo as root can destroy the system.
- A compromised application running as root gives an attacker the whole machine.
- Malware or a malicious script inherits whatever privileges you used to run it.

The safer pattern is: log in as a normal user, use **`sudo`** (“superuser do”) for individual administrative commands, and use root only through a controlled path (provider console, recovery mode, or a dedicated break-glass account).

```bash
sudo apt update
```

`sudo` checks whether your user is authorised in `/etc/sudoers` (or a file in `/etc/sudoers.d/`), asks for **your** password, and runs that one command as root. The authorisation usually lasts a few minutes per terminal.

### 5.2 Create a normal user

On Ubuntu/Debian, the friendly command is:

```bash
sudo adduser om
```

It asks for a password, then some optional “Full Name”, “Room Number” and phone fields. You can press `Enter` through the optional fields and confirm with `Y`. It creates:

- the user account;
- a home directory `/home/om`;
- a matching primary group `om`;
- default shell and skeleton files (`.bashrc`, `.profile`).

The lower-level command is `useradd`. It does less by default, so you must ask for a home directory:

```bash
sudo useradd -m -s /bin/bash rohan
sudo passwd rohan
```

`-m` creates the home directory; `-s` sets the login shell. Without `-m`, the user may have no home and login can behave oddly. For beginners on Ubuntu, `adduser` is usually the better choice.

Set or change a password later with:

```bash
sudo passwd om
```

The password is not echoed while typing. Use a password manager; do not reuse the VPS root password.

### 5.3 Check identity and switch users

```bash
whoami                 # current username
id                     # UID, GID and group memberships
groups                 # group names
who                    # logged-in users
w                      # who is logged in and what they are doing
last                   # recent logins
```

Switch to another account:

```bash
su - om                # start a login shell as om (loads om's environment)
su om                  # switch user but keep much of the current environment
exit                   # return to the previous shell
```

As root, `su - om` does not ask for om's password; as a normal user, it does. The `-` matters: a login shell loads the target user's profile and `PATH`, which avoids confusing “works for root but not for om” problems.

### 5.4 Grant administrative access

Add the user to the `sudo` group:

```bash
sudo usermod -aG sudo om
```

- `-a` **append** to the supplementary group list.
- `-G` specifies supplementary groups.
- Omitting `-a` replaces the user's entire supplementary group list. That is a common and painful mistake.

Verify:

```bash
groups om
# om : om sudo
```

The user must log out and back in (or start a new login shell) for the new group membership to apply to their session:

```bash
su - om
sudo whoami        # should print root after authentication
```

The group name differs by distribution: `sudo` on Ubuntu/Debian, often `wheel` on RHEL-family systems. Check `/etc/sudoers` or the distribution documentation.

### 5.5 Where user information lives

| File | Contains | Notes |
|---|---|---|
| `/etc/passwd` | One line per user: name, UID, GID, home, shell | World-readable; no password hashes in modern systems |
| `/etc/shadow` | Password hashes and aging | Readable only by root |
| `/etc/group` | Group name, GID, members | Supplementary memberships |
| `/etc/sudoers` | Who may run what as root | Edit with `sudo visudo`, never with a plain editor |

A `/etc/passwd` line looks like:

```text
om:x:1001:1001:Om,,,:/home/om:/bin/bash
```

Fields: username, placeholder `x`, UID, GID, comment/GECOS, home directory, login shell.

Edit sudo rules with:

```bash
sudo visudo
```

`visudo` locks the file and validates syntax before saving. A broken sudoers file can lock you out of `sudo`, so this matters.

### 5.6 Deleting or disabling an account

```bash
sudo userdel -r olduser     # remove account and home directory
sudo usermod -L olduser     # lock the password
sudo usermod -U olduser     # unlock
```

`userdel -r` deletes the home directory and mail spool. Confirm that no important files live there and that running services do not depend on the account.

### 5.7 SSH keys are better than passwords

Passwords can be guessed or reused. A key pair has a private key (stays on your computer, never shared) and a public key (installed on the server in `~/.ssh/authorized_keys`).

```bash
# On your computer
ssh-keygen -t ed25519 -C "om@laptop"
ssh-copy-id om@203.0.113.10
ssh om@203.0.113.10
```

Once key login works, disable password authentication only after confirming you can reconnect with the key. Keep a provider console available as a fallback. This is an optional supplement to the lecture but standard practice for any public server.

## 6. Installing software with apt

### 6.1 Packages and repositories

A **package** is a bundle containing a program, its libraries, documentation and metadata. A **package manager** resolves dependencies, downloads packages from configured **repositories**, verifies signatures, installs files and records what is installed.

Ubuntu and Debian use **APT** (Advanced Package Tool). The command-line front end is `apt`; `apt-get` is the older, script-friendly interface. Both still work, but interactive use should prefer `apt`.

Ubuntu is built **on top of Debian**. Debian is an independent, long-running community project; Canonical produces Ubuntu by customising Debian with its own release cycle, installer, defaults and support. This is why both use `.deb` packages and `apt`.

### 6.2 Refresh the index before installing

```bash
sudo apt update
```

This downloads the **package index** — the list of available packages and versions — from the repositories. It does **not** upgrade installed software. Think of it as updating the shop's catalogue, not buying anything.

```bash
sudo apt upgrade
```

This installs available updates for already-installed packages. It can restart services, change configuration behaviour or introduce a regression. On a real server:

- Read what will change before confirming.
- Prefer a maintenance window for major updates.
- Have backups and a rollback plan.
- Use `apt list --upgradable` to preview.
- Consider `unattended-upgrades` for security patches, with testing and monitoring.

The lecture's warning — “don't blindly run upgrade” — is about production risk, not about avoiding updates forever. Unpatched systems are a far more common security problem than a bad update that was never tested.

Other useful forms:

```bash
sudo apt full-upgrade    # may add/remove packages to resolve dependencies
sudo apt dist-upgrade    # older name for similar behaviour
sudo apt list --upgradable
```

### 6.3 Install, inspect and remove

```bash
sudo apt install nginx
sudo apt install curl git python3 nginx    # several packages in one command
sudo apt install -y nginx                  # assume yes (non-interactive)
apt show nginx                             # package metadata, version, dependencies
apt list --installed                       # all installed packages
apt list --installed | grep nginx          # filter the list
apt search "web server"                    # search names/descriptions
```

Remove:

```bash
sudo apt remove apache2         # remove program files, keep configuration
sudo apt purge apache2          # remove program files and configuration
sudo apt autoremove             # remove packages no longer required
```

`remove` is reversible-ish: configuration remains, so reinstalling may restore your settings. `purge` is a clean slate. `autoremove` cleans dependencies that were installed automatically and are no longer needed, but review the list before confirming.

### 6.4 Other package tools you will meet

| Tool | Use |
|---|---|
| `dpkg -i package.deb` | Install a downloaded Debian package directly; does not resolve dependencies |
| `snap install` | Install a sandboxed Snap package |
| `flatpak` | Desktop application packaging |
| `dnf` / `yum` | RHEL, Fedora, Rocky, AlmaLinux |
| `pacman` | Arch Linux |
| `apk` | Alpine Linux |
| `pip`, `npm`, `cargo` | Language-specific package managers, separate from the OS |

Mixing sources without understanding them can create conflicting versions. Prefer the distribution repository for system services, and language-specific tools for application dependencies.

### 6.5 Lab — install and verify a web server

```bash
sudo apt update
sudo apt install nginx
systemctl status nginx
curl -I http://localhost
```

`curl -I` sends a HEAD request and prints response headers. A `200 OK` or `301`/`302` means Nginx is answering. If the service is not running, read the status output and logs before reinstalling.

## 7. Groups and permissions

### 7.1 Why groups exist

Suppose a server has 200 developers. Giving each developer permission on each project directory one by one is unmanageable. A **group** is a named collection of users. Set permissions once on the group and every member inherits them.

Each user has:

- a **primary group** (usually a group with the same name as the user), recorded in `/etc/passwd`;
- zero or more **supplementary groups**, recorded in `/etc/group`.

```bash
sudo groupadd developers
sudo useradd -m rohan
sudo useradd -m lavish
sudo passwd rohan
sudo passwd lavish
sudo usermod -aG developers rohan
sudo usermod -aG developers lavish
groups rohan
# rohan : rohan developers
```

Other group commands:

```bash
sudo groupdel developers        # delete a group (remove memberships first in practice)
sudo gpasswd -a rohan developers
sudo gpasswd -d rohan developers
getent group developers         # show the group entry
```

### 7.2 Read the ten-character permission string

```bash
ls -l
```

```text
drwxr-xr-x 2 om developers 4096 Nov 20 10:14 projects
-rw-r--r-- 1 om om          214 Nov 20 10:10 notes.txt
```

Break down `drwxr-xr-x`:

```text
d   rwx   r-x   r-x
│   │     │     └── others: read + execute
│   │     └──────── group: read + execute
│   └────────────── owner: read + write + execute
└────────────────── type: d = directory, - = regular file, l = symlink
```

The next columns are: hard-link count, owner, group, size, modification date, name.

The **owner** is a single user. The **group** is a single group. **Others** means everyone else. Permission checks use the most specific matching class: if you are the owner, owner permissions apply even if the group would allow more.

### 7.3 What r, w and x mean

For a **regular file**:

| Permission | Meaning |
|---|---|
| `r` | Read the contents |
| `w` | Modify the contents |
| `x` | Execute it as a program/script |

For a **directory**, the meaning changes:

| Permission | Meaning |
|---|---|
| `r` | List the names inside (`ls`) |
| `w` | Create, delete or rename entries inside |
| `x` | Enter/traverse the directory (`cd`) and access entries by name |

A directory with `r` but no `x` lets you see names but not open them. A directory with `x` but no `r` lets you `cd` into it and open a file if you already know its exact name, but not list it. To create or delete files inside a directory, you need `w` **and** `x` on the directory — file permissions alone are not enough.

### 7.4 Change ownership and group

```bash
sudo chown lavish devs                 # owner → lavish
sudo chown lavish:developers devs      # owner and group in one step
sudo chgrp developers devs             # group only
sudo chown -R om:om /var/www/website   # recursive
```

`chown` changes the owner; only root can give a file to another user. `chgrp` changes the group; the owner can change a file's group to a group they belong to, subject to system policy.

Common deployment fix:

```bash
sudo chown -R www-data:www-data /var/www/website
```

This lets the web server user read the site. But it may prevent your deploy user from writing. A better pattern is a shared group with group write and setgid, or a deploy process that runs as the correct user. Do not solve ownership problems by opening permissions to everyone.

### 7.5 Symbolic chmod

```bash
chmod u+w file        # add write for owner
chmod g+r file        # add read for group
chmod o-rwx file      # remove all permissions for others
chmod a-w file        # remove write for all (u+g+o)
chmod u+x script.sh   # make executable by owner
```

Who: `u` owner (user), `g` group, `o` others, `a` all.
Operator: `+` add, `-` remove, `=` set exactly.
Permission: `r`, `w`, `x`.

Multiple changes can be combined:

```bash
chmod u+x,g+rx,o-rwx deploy.sh
```

Note that the owner symbol is `u`, not `o`; `o` means others.

### 7.6 Numeric chmod

Each triad is a sum:

| Value | Permission |
|---|---|
| 4 | read |
| 2 | write |
| 1 | execute |

Add them for each class:

```text
rwx = 4+2+1 = 7
rw- = 4+2   = 6
r-x = 4+1   = 5
r-- = 4     = 4
-wx = 2+1   = 3
-w- = 2     = 2
--x = 1     = 1
--- = 0     = 0
```

```bash
chmod 755 website     # owner rwx, group r-x, others r-x
chmod 644 notes.txt   # owner rw-, group r--, others r--
chmod 600 id_rsa      # owner rw-, nobody else
chmod 700 private/    # owner only
```

The lecture's `chmod 435 devs` is a useful arithmetic exercise:

```text
4 = r--   (owner: read only)
3 = -wx   (group: write + execute)
5 = r-x   (others: read + execute)

Result: r---wxr-x
```

It is valid, but unusual: the owner cannot write to their own directory. A more practical shared-project setting might be `775` (owner and group read/write/execute) or `2775` with setgid so new files inherit the directory's group.

### 7.7 Dangerous defaults and safer choices

```bash
chmod -R 777 /var/www/website     # avoid
```

`777` lets every user on the system read, write and execute everything in the tree. On a web server, that can turn a small file-upload bug into remote code execution. Prefer:

- `755` for directories that only need to be readable/executable;
- `644` for normal files;
- `600` for secrets and private keys;
- `775`/`2775` for a shared group-owned deployment directory;
- least privilege for the service account that actually needs access.

**umask** controls default permissions for new files and directories. A typical `umask 022` gives new directories `755` and files `644`. `umask` masks bits out; it does not add them.

### 7.8 Special bits (optional supplement)

| Bit | Numeric | Effect |
|---|---|---|
| setuid | 4000 | Program runs with the file owner's effective UID |
| setgid | 2000 | Program runs with the file's group; on a directory, new files inherit the group |
| sticky | 1000 | On a directory, only the owner of a file (or root) can delete it |

`/tmp` usually has the sticky bit (`drwxrwxrwt`), which is why all users can create files there but cannot delete each other's files. Setuid is powerful and should be rare; audit it rather than adding it casually.

## 8. Processes and services

### 8.1 A process is a running program

A **program** is a file on disk. A **process** is an executing instance with its own memory, PID, user, open files and state. Run the same program twice and you get two processes with different PIDs.

Every process has:

- a **PID** (process ID);
- a **PPID** (parent process ID);
- an owner (UID/GID);
- a current working directory and environment;
- file descriptors for input, output, error and open files;
- a state (running, sleeping, stopped, zombie).

When you type `ls`, the shell forks a child process, the child executes `ls`, and the shell waits for it to finish. When you type `nginx`, a long-running process (or a service managed by systemd) starts.

### 8.2 Inspect processes

```bash
ps                 # processes in the current terminal/session
ps aux             # all processes, all users, BSD-style columns
ps -ef             # all processes, System V-style columns
ps aux | grep nginx
pgrep -a nginx     # PIDs and command names matching nginx (cleaner than grep)
pstree -p          # process tree with PIDs
```

`ps aux` columns:

| Column | Meaning |
|---|---|
| `USER` | Owner of the process |
| `PID` | Process ID |
| `%CPU` | CPU percentage used |
| `%MEM` | Resident memory percentage |
| `VSZ` | Virtual memory size |
| `RSS` | Resident set size (physical memory in use) |
| `TTY` | Terminal associated with it |
| `STAT` | State (`R` running, `S` sleeping, `Z` zombie, and modifiers) |
| `START` | Start time/date |
| `TIME` | Cumulative CPU time |
| `COMMAND` | Command line |

`grep nginx` can match the `grep` process itself. `pgrep -a` avoids that confusion.

### 8.3 Watch resource use in real time

```bash
top
```

`top` refreshes periodically. Key columns: PID, USER, PR/NI priority, VIRT/RES/SHR memory, S state, %CPU, %MEM, TIME+, COMMAND. Press `q` to quit. Press `M` to sort by memory, `P` by CPU, `k` to send a signal to a PID (use with care).

```bash
htop
```

`htop` is a friendlier viewer with colour, scrolling and function keys. If it is missing:

```bash
sudo apt install htop
```

In `htop`: `F6` chooses the sort column (CPU%, MEM%), `F3` searches, `F9` sends a signal, `F10` quits. The lecture demonstrates sorting by CPU and memory.

Also useful:

```bash
uptime             # load average for 1, 5 and 15 minutes
free -h            # RAM and swap
vmstat 1 5         # run queue, memory, swap, I/O every second
```

**Load average** is the average number of runnable or uninterruptible processes. On a 2-core machine, a load of 2 is roughly “fully busy”; 8 means heavy queueing. It is not a percentage and not meaningful without core count and workload type.

### 8.4 Stop a process

```bash
kill 30414          # send SIGTERM (15): ask the process to terminate gracefully
kill -9 30414       # send SIGKILL: force immediate termination
pkill python3       # signal every process matching the name
pkill -f "python3 main.py"   # match the full command line
```

`SIGTERM` lets a process flush buffers, close files and clean up. `SIGKILL` cannot be caught or ignored; the kernel removes the process immediately, potentially leaving temporary files, locks or partial writes. Use `-9` only when graceful termination has failed and you understand the consequence.

Find the PID first:

```bash
pgrep -a python3
ps -fp 30414
```

Signals worth knowing:

| Signal | Number | Meaning |
|---|---|---|
| `SIGHUP` | 1 | Hangup; many daemons reload configuration |
| `SIGINT` | 2 | Interrupt (Ctrl+C) |
| `SIGTERM` | 15 | Graceful termination (default for `kill`) |
| `SIGKILL` | 9 | Forced kill, cannot be caught |
| `SIGSTOP` | 19 | Pause a process (cannot be caught) |
| `SIGCONT` | 18 | Resume a paused process |

### 8.5 Foreground, background and persistence

```bash
python3 main.py          # runs in the foreground; Ctrl+C interrupts
python3 main.py &        # runs in the background; shell prints job number and PID
jobs                     # list background jobs in this shell
fg %1                    # bring job 1 to the foreground
bg %1                    # resume a stopped job in the background
```

Background jobs are children of your shell. Close the terminal and they may receive `SIGHUP` and die. For work that must survive logout, use a service manager (systemd) or a terminal multiplexer such as `tmux`/`screen`; `nohup command &` is a quick but limited alternative.

### 8.6 A service is a managed background program

A **service** (often a **daemon**) is a long-running program that provides a function without an interactive user: a web server, database, SSH daemon, cron scheduler. You do not start it manually every time; a **service manager** starts it at boot, restarts it on failure where configured, and gives you a standard way to query and control it.

Most modern distributions use **systemd**. PID 1 is `systemd`, which reads **unit files** (usually in `/lib/systemd/system/` or `/etc/systemd/system/`) describing how to start, stop and supervise a service.

```bash
systemctl status nginx
```

Typical output:

```text
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-11-19 10:12:03 UTC; 2h 4min ago
       Docs: man:nginx(8)
   Main PID: 812 (nginx)
      Tasks: 3 (limit: 4652)
     Memory: 6.2M
        CPU: 41ms
     CGroup: /system.slice/nginx.service
             ├─812 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─815 "nginx: worker process"
```

The important lines: `Loaded` (unit found; `enabled` means start at boot), `Active` (running/stopped/failed), and the main PID.

Control it:

```bash
sudo systemctl start nginx      # start now
sudo systemctl stop nginx       # stop now
sudo systemctl restart nginx    # stop + start (brief downtime)
sudo systemctl reload nginx     # re-read configuration without dropping the process
sudo systemctl enable nginx     # start automatically at boot
sudo systemctl disable nginx    # do not start at boot
sudo systemctl enable --now nginx   # enable and start in one step
systemctl is-active nginx
systemctl is-enabled nginx
```

`restart` vs `reload`:

- **restart** stops the process and starts a new one. Connections may drop, and in-flight requests can fail. Use it when the program itself changed or reload is unsupported.
- **reload** asks the running process to re-read its configuration. For Nginx this is graceful: old workers finish current requests while new workers use the new config. Not every service supports reload.

After changing a systemd unit file, reload the manager's view:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

Read logs with journald:

```bash
journalctl -u nginx                 # all logs for the unit
journalctl -u nginx -f              # follow live
journalctl -u nginx --since "10 min ago"
journalctl -p err -b                # errors since boot
```

Older systems and some services write to `/var/log/`. `service nginx status` is a compatibility wrapper on many distributions, but `systemctl` is the primary interface.

### 8.7 A minimal custom service (optional supplement)

If you write an app at `/home/om/app/server.py`, create `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=My Python app
After=network.target

[Service]
Type=simple
User=om
WorkingDirectory=/home/om/app
ExecStart=/usr/bin/python3 /home/om/app/server.py
Restart=on-failure
RestartSec=3
Environment=APP_ENV=production

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemctl status myapp
journalctl -u myapp -f
```

This is the standard way to make an application survive logout and reboot. Running it with `&` is not a production strategy.

## 9. Environment variables, PATH and bashrc

### 9.1 Variables

A **variable** is a named reference to a value held by the shell.

```bash
NAME=om             # no spaces around =
echo "$NAME"        # om
echo $NAME          # also om
```

Rules:

- No spaces around `=`: `NAME = om` is a command named `NAME`, not an assignment.
- Use `$NAME` or `${NAME}` to expand the value.
- Single quotes prevent expansion; double quotes allow it.
- Unquoted values undergo word splitting and wildcard expansion, which is why `"$VAR"` is usually safer.

```bash
GREETING="hello world"
echo $GREETING      # hello world (two arguments to echo, same output here)
printf '%s\n' "$GREETING"   # one argument, precise
```

### 9.2 Shell variables vs environment variables

A **shell variable** exists only in the current shell. An **environment variable** is exported to child processes.

```bash
COURSE=linux              # shell variable
bash -c 'echo "$COURSE"'  # prints nothing: child did not inherit it

export COURSE=linux       # environment variable
bash -c 'echo "$COURSE"'  # prints linux
```

`export` marks a variable for inheritance. Every process receives a copy of the environment at creation; changing a variable later does not retroactively change existing children.

List them:

```bash
printenv              # environment variables
env                   # same idea, also used to run a command with a modified environment
set                   # all shell variables and functions (much longer)
```

Common variables:

| Variable | Meaning |
|---|---|
| `HOME` | Home directory |
| `USER` | Current username |
| `SHELL` | Login shell path |
| `PWD` | Current working directory |
| `PATH` | Directories searched for commands |
| `LANG` / `LC_*` | Locale and character encoding |
| `TERM` | Terminal type |
| `EDITOR` | Preferred editor for tools that ask |

```bash
echo "$HOME"
echo "$USER"
echo "$SHELL"
```

### 9.3 Why environment variables matter in DevOps

Applications should not hard-code values that differ between development, staging and production. Environment variables let the same code run with different configuration:

```bash
export APP_ENV=staging
export DATABASE_URL="postgres://app:secret@127.0.0.1:5432/app"
export LOG_LEVEL=debug
```

Your application reads them at startup. The container, CI runner or server injects the right values. This is the basis of twelve-factor configuration and of Kubernetes ConfigMaps/Secrets.

**Security warning:** environment variables are visible to the process owner (and root) via `/proc/<pid>/environ`. Do not put long-lived secrets in a shell history or a world-readable file. Use a secret manager or a restricted environment file with `600` permissions, and avoid `export SECRET=...` directly on a shared terminal.

### 9.4 PATH: how the shell finds commands

```bash
echo "$PATH"
```

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

`PATH` is a colon-separated list of directories. When you type `ls`, the shell searches those directories in order and runs the first match.

```bash
which ls          # /usr/bin/ls
which python3     # /usr/bin/python3
type ls           # shell builtin, alias, or file
command -v nginx  # portable lookup
```

If a command is not in any `PATH` directory, you get:

```text
bash: mytool: command not found
```

You can always run it by explicit path:

```bash
/usr/local/bin/mytool
./mytool            # current directory, explicitly
```

The current directory is usually **not** in `PATH` for security reasons. If it were, an attacker's `ls` in a shared directory could run when you meant `/usr/bin/ls`. This is why `./script.sh` is required.

### 9.5 Add a directory to PATH

```bash
mkdir -p ~/bin
mv myscript.sh ~/bin/
chmod +x ~/bin/myscript.sh
export PATH="$PATH:$HOME/bin"
myscript.sh
```

Use `"$PATH:..."` so the existing entries remain. A common mistake is:

```bash
export PATH="/home/om/bin"      # wipes the rest of PATH
```

That makes `ls`, `apt` and almost everything else disappear from the current shell until you fix it (often by opening a new shell or using an absolute path to a shell). To recover in the same session:

```bash
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

Then fix your startup file.

### 9.6 Scripts, shebangs and execute permission

A shell script is a text file containing commands:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Hello, $USER"
echo "Today is $(date)"
```

- `#!/usr/bin/env bash` is the **shebang**; it tells the kernel which interpreter to use.
- `set -e` stops on errors, `-u` errors on unset variables, `-o pipefail` propagates pipeline failures. These are good defaults for automation.

Make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh
```

Without execute permission, you can still run it through the interpreter:

```bash
bash hello.sh
```

### 9.7 Persist configuration in `~/.bashrc`

Changes made with `export` last only for the current shell session. To apply them to every new interactive shell, add them to `~/.bashrc`:

```bash
vim ~/.bashrc
```

At the end of the file:

```bash
export PATH="$PATH:$HOME/bin"
export EDITOR=vim
export APP_ENV=development
```

Save, then load it into the current shell:

```bash
source ~/.bashrc
# or: . ~/.bashrc
```

`source` executes the file in the **current** shell, so its variables affect your session. Running `bash ~/.bashrc` starts a child shell; the variables disappear when it exits.

Which file loads when?

- **Interactive non-login shell** (new terminal tab): `~/.bashrc`.
- **Login shell** (SSH login, `su -`, console login): `/etc/profile`, then `~/.profile` (or `~/.bash_profile`), which on Ubuntu usually sources `~/.bashrc`.
- **Non-interactive shell** (script): generally neither, unless `BASH_ENV` is set.

So `~/.bashrc` is the right place for interactive aliases and PATH additions on Ubuntu. For environment variables needed by services, prefer systemd `Environment=`/`EnvironmentFile=`, container configuration or your orchestrator — not a user's `.bashrc`, which services do not read.

Useful aliases:

```bash
alias ll='ls -alF'
alias ..='cd ..'
alias ports='ss -tulpn'
```

Aliases work only in interactive shells and are not a substitute for scripts.

## 10. Archives and compression

### 10.1 Two different jobs

- **Archiving** combines many files and directories into one container file. It does not necessarily reduce size.
- **Compression** encodes data to use fewer bytes. It does not necessarily combine files.

A `.tar` file is an archive. A `.tar.gz` is an archive that was then compressed with gzip. A `.zip` does both.

```text
python-project/
├── main.py
├── utils.py
├── static/
└── templates/

tar -cf python.tar python-project     → one container, roughly same total bytes
gzip python.tar                      → python.tar.gz, smaller
zip -r python.zip python-project     → one compressed container
```

### 10.2 tar basics

```bash
tar -cf python.tar python-project     # create archive
tar -tf python.tar                    # list contents (peek, don't extract)
tar -xf python.tar                    # extract
tar -xvf python.tar                   # extract verbosely
tar -xvf python.tar -C /tmp/          # extract into /tmp
```

Meaning of the common flags:

| Flag | Meaning |
|---|---|
| `-c` | create |
| `-x` | extract |
| `-t` | list/table of contents |
| `-f` | file name follows (must come before the archive name) |
| `-v` | verbose |
| `-z` | gzip compression/decompression |
| `-j` | bzip2 |
| `-J` | xz |
| `-C` | change to directory before operating |

Create and extract a compressed archive directly:

```bash
tar -czf python-project.tar.gz python-project
tar -xzf python-project.tar.gz
tar -tzf python-project.tar.gz        # list a .tar.gz without extracting
```

`tar` preserves ownership and permissions by default when run as root, and stores relative paths as given. Extracting an archive created with absolute paths can overwrite system files. Before extracting an untrusted archive, list it first:

```bash
tar -tf suspicious.tar
tar -xvf suspicious.tar -C /tmp/safe-dir
```

Watch for entries beginning with `/` or containing `..`; these are path-traversal risks. Modern GNU tar strips leading `/` by default, but do not rely on that alone.

### 10.3 gzip and gunzip

`gzip` compresses a single file and **replaces** it with a `.gz` version by default:

```bash
gzip python.tar          # creates python.tar.gz, removes python.tar
gunzip python.tar.gz     # restores python.tar
gzip -k python.tar       # keep the original as well
gzip -l python.tar.gz    # compression ratio
```

`gzip` does not archive directories. That is why `tar` and `gzip` are used together: `tar` makes one file, then `gzip` shrinks it.

### 10.4 zip and unzip

ZIP is convenient when exchanging files with Windows or macOS because it is widely supported and combines archiving and compression:

```bash
zip -r python.zip python-project      # -r = recurse into directories
unzip python.zip                      # extract here
unzip -l python.zip                   # list contents
unzip python.zip -d /tmp/extracted    # extract elsewhere
```

On minimal Ubuntu installs, `zip`/`unzip` may not be present:

```bash
sudo apt install zip unzip
```

`zip` is not ideal for preserving Linux ownership and special permissions; use `tar` for system backups.

### 10.5 Practical backup example

```bash
mkdir -p ~/backups
tar -czf ~/backups/site-$(date +%F).tar.gz /var/www/website
ls -lh ~/backups
```

`$(date +%F)` expands to a date such as `2026-11-22`, giving a unique filename per day. Verify an archive before trusting it:

```bash
tar -tzf ~/backups/site-2026-11-22.tar.gz | head
sha256sum ~/backups/site-2026-11-22.tar.gz
```

A checksum is a fixed-length fingerprint. Compare it after copying the file to another machine to detect corruption or tampering. It is not encryption.

**Encryption note:** `tar` and `zip` do not encrypt data by themselves. Use `age`, `gpg` or an encrypted filesystem for sensitive backups, and protect the key separately.

## 11. Scheduling with cron

### 11.1 What cron is

**cron** is a background service that runs scheduled commands. Each user has a **crontab** (cron table). A single scheduled command is a **cron job**.

Typical uses:

- nightly database backups;
- clearing temporary files every few minutes;
- rotating logs;
- sending a health report every morning;
- renewing certificates;
- syncing files between directories.

The alternative is doing it manually, which works until you forget, are on holiday, or have 200 servers.

### 11.2 Check the service first

```bash
systemctl status cron
```

On Ubuntu the service is `cron`; on some distributions it is `crond`. If it is inactive:

```bash
sudo systemctl start cron
sudo systemctl enable cron
```

If cron is stopped, your jobs will never run. Check this before debugging the schedule.

### 11.3 Manage your crontab

```bash
crontab -l          # list your jobs
crontab -e          # edit your jobs
crontab -r          # remove your entire crontab (dangerous, no confirmation on some systems)
```

`crontab -e` opens an editor (the first time, it may ask you to choose). A new crontab is empty. Add one line per job. Lines beginning with `#` are comments.

Run as root with `sudo crontab -e` only for jobs that truly need root. Prefer a dedicated user with narrowly scoped permissions.

### 11.4 The five fields

```text
┌───────────── minute       0–59
│ ┌─────────── hour         0–23
│ │ ┌───────── day of month 1–31
│ │ │ ┌─────── month        1–12 (or JAN–DEC)
│ │ │ │ ┌───── day of week  0–7 (0 and 7 = Sunday, or SUN–SAT)
│ │ │ │ │
* * * * *  command to run
```

Examples:

| Schedule | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | At minute 0 of every hour |
| `0 8 * * *` | Every day at 08:00 |
| `30 2 * * 1` | Every Monday at 02:30 |
| `0 0 1 * *` | First day of every month at midnight |
| `0 9 * * 1-5` | Weekdays at 09:00 |
| `0 0 1 1 *` | January 1 at midnight |
| `15 14 1 * *` | 14:15 on the first of every month |

Special operators:

| Symbol | Meaning | Example |
|---|---|---|
| `*` | Every value | `*` in minute = every minute |
| `,` | List | `0,15,30,45 * * * *` |
| `-` | Range | `0 9-17 * * *` |
| `/` | Step | `*/10 * * * *` |

Day-of-month and day-of-week are **ORed** when both are restricted (not AND). For example, `0 0 1 * 1` runs on the first of the month **or** on Mondays. This surprises people; test with `crontab.guru` or a safe schedule.

### 11.5 A first job, done correctly

Create a script rather than cramming a complex pipeline into a crontab line:

```bash
#!/usr/bin/env bash
set -euo pipefail

STAMP=$(date +%F-%H%M)
DEST="$HOME/backups"
mkdir -p "$DEST"
tar -czf "$DEST/site-$STAMP.tar.gz" /var/www/website
find "$DEST" -name 'site-*.tar.gz' -mtime +7 -delete
```

```bash
chmod +x ~/backup-site.sh
~/backup-site.sh          # test manually first
```

Then:

```cron
0 8 * * * /home/om/backup-site.sh >> /home/om/backup.log 2>&1
```

Why this form:

- **Full paths** for the script and for any program it calls. Cron's `PATH` is minimal and may not include `/usr/local/bin`.
- **Redirect output** so failures are visible. Without redirection, cron may try to email output and it disappears if mail is not configured.
- **Test manually** before scheduling. A job that fails every minute is worse than no job.
- **No interactive prompts.** Cron has no terminal; `apt install` without `-y` or a script waiting for input will hang or fail.
- **Quote paths** with spaces.
- **End the file with a newline.** Some cron implementations ignore a final line without one.

Check it:

```bash
crontab -l
sleep 60
cat ~/backup.log
ls -lh ~/backups
```

Cron runs at minute boundaries. If you schedule `* * * * *` at 10:00:30, the first run is at 10:01:00. Wait at least a full minute.

### 11.6 Environment and logging

Cron starts jobs with a minimal environment. `HOME`, `SHELL`, `LOGNAME` and a short `PATH` are set; your `.bashrc` is not loaded. This is the most common reason “it works when I run it, but not from cron”.

Fix it by:

- using absolute paths;
- setting `PATH` at the top of the crontab if needed:

```cron
PATH=/usr/local/bin:/usr/bin:/bin
```

- sourcing a specific environment file inside the script;
- defining needed variables in the script.

Logs:

```bash
journalctl -u cron --since today
grep CRON /var/log/syslog        # Debian/Ubuntu
```

`/var/log/syslog` location varies by distribution. If `journalctl` is available, prefer it.

### 11.7 Cron safety practices

- **Avoid overlapping runs.** A backup that takes 20 minutes scheduled every 5 minutes creates a pile of processes. Use `flock`:

```cron
*/5 * * * * /usr/bin/flock -n /tmp/myjob.lock /home/om/job.sh >> /home/om/job.log 2>&1
```

- **Do not schedule `reboot` or destructive cleanup casually.**
- **Monitor success**, not just execution. A log line or a heartbeat file is better than silence.
- **Keep secrets out of the crontab**; crontabs are readable by the owner and root, and may be included in backups.
- **Prefer scripts** for anything longer than a simple command.
- **Consider systemd timers** for new automation: they support dependencies, `Persistent=true` (catch up after downtime), random delays, resource limits and integrated logging. Cron is still ubiquitous and worth knowing.

### 11.8 Crontab.guru

`crontab.guru` translates a five-field expression into plain English and provides examples. It is an excellent learning and verification tool. Do not paste secrets into any website; the schedule itself is not sensitive, but the command line might be.

## 12. The Linux filesystem

### 12.1 One tree, many mount points

Linux presents all files in a single hierarchical tree beginning at `/`. Disks and partitions are **mounted** at directories inside that tree. Unlike Windows drive letters (`C:\`, `D:\`), a second disk might appear as `/data` or `/var/lib/mysql`.

```text
/
├── bin → usr/bin        essential command binaries (symlink on modern Ubuntu)
├── boot                 kernel and bootloader files
├── dev                  device files
├── etc                  system-wide configuration
├── home                 regular users' home directories
│   ├── om
│   ├── lavish
│   └── rohan
├── lib → usr/lib        shared libraries
├── media                removable media mount points
├── mnt                  temporary manual mount points
├── opt                  optional third-party software
├── proc                 virtual filesystem for process/kernel info
├── root                 root user's home directory
├── run                  runtime state (PIDs, sockets)
├── sbin → usr/sbin      system administration binaries
├── srv                  service data (web, FTP)
├── sys                  virtual filesystem for devices/kernel
├── tmp                  temporary files, often cleared at boot
├── usr                  Unix System Resources: programs, libraries, docs
│   ├── bin
│   ├── lib
│   ├── local
│   └── share
└── var                  variable data: logs, caches, spools, websites
    ├── log
    ├── www
    └── lib
```

This layout is guided by the **Filesystem Hierarchy Standard (FHS)**, though distributions add their own directories and symlinks.

### 12.2 The directories you will actually use

| Path | Purpose | Beginner note |
|---|---|---|
| `/` | Root of everything | You cannot go “above” it |
| `/root` | Home of the root user | Not the same as `/`; normal users usually cannot enter |
| `/home/name` | User's home | Your files, dotfiles, `.ssh`, projects |
| `/etc` | Configuration | Nginx, SSH, cron, package sources |
| `/var` | Changing data | Logs grow here; backups and databases too |
| `/var/log` | Log files | `syslog`, `auth.log`, service logs |
| `/var/www` | Web content (Debian/Ubuntu convention) | Nginx/Apache document roots |
| `/usr` | System programs and libraries | “Unix System Resources”, not “user” |
| `/usr/bin` | Most user commands | `ls`, `python3`, `vim` |
| `/usr/sbin` | Admin commands | Often need root |
| `/usr/local` | Locally installed software | Compiled tools, custom binaries |
| `/tmp` | Temporary files | Shared, sticky bit; may be cleared; never store important data |
| `/opt` | Optional add-on software | Some commercial or self-contained apps |
| `/srv` | Service data | Alternative to `/var/www` for some setups |
| `/boot` | Kernel and bootloader | Do not delete; can make system unbootable |
| `/dev` | Device nodes | `/dev/sda`, `/dev/null`, `/dev/random` |
| `/proc` | Kernel/process info as files | `/proc/cpuinfo`, `/proc/<pid>/` |
| `/sys` | Device and kernel state | Mostly for tools, not hand-editing |
| `/run` | Runtime state since boot | PID files, sockets; cleared at boot |
| `/mnt`, `/media` | Mount points | Manual mounts vs removable media |

### 12.3 “Root” means two different things

This confuses almost every beginner:

- **`/`** is the root of the filesystem — the outermost directory.
- **`/root`** is the home directory of the **root user**.

`cd /` goes to the top of the tree. `cd /root` goes to the administrator's private home. They are different paths with different permissions.

Similarly, “root privileges” means UID 0, not “the root folder”.

### 12.4 Find out what is using space

```bash
df -h                 # filesystem usage by mount
du -sh /var/log       # total size of a directory
du -h --max-depth=1 /var | sort -h
lsblk                 # block devices and mount points
mount | column -t     # current mounts
find /var/log -type f -size +100M -ls   # large files
```

`df` reports filesystem-level usage; `du` walks directories and sums file sizes. They can disagree because of deleted-but-open files, reserved blocks, compression or different mount boundaries.

A full disk causes confusing failures: services cannot write logs, databases stop, package installs fail. Check `df -h` early when “nothing works”.

### 12.5 Paths, links and mounts (optional supplement)

- A **hard link** is another name for the same inode; deleting one name does not delete the data until all links are gone. Hard links cannot span filesystems and usually cannot point to directories.
- A **symbolic link** (symlink) is a small file containing a path. `ln -s target linkname` creates one. If the target moves, the link breaks.
- A **mount** attaches a filesystem at a directory. `/etc/fstab` makes mounts persistent across boot; a mistake there can prevent booting, so edit with care and keep a recovery console available.

```bash
ln -s /var/www/website/current /var/www/html
readlink -f /var/www/html
```

Symlinks are used heavily by Nginx's `sites-enabled` directory.

## 13. Serving sites with Nginx

### 13.1 What Nginx does

**Nginx** is a high-performance web server and reverse proxy. It can:

- serve static files (HTML, CSS, JavaScript, images, PDFs);
- terminate TLS/HTTPS;
- proxy requests to application servers (Node, Python, Go, Java);
- load-balance across several backends;
- cache responses;
- compress responses;
- rewrite URLs and enforce redirects.

It is event-driven, so a small number of worker processes can handle many concurrent connections. That is why it is common in front of application servers and in containers.

```text
Browser ──HTTP/HTTPS──> Nginx ──static file──> /var/www/website
                          │
                          └──proxy_pass──> 127.0.0.1:3000 (your app)
```

### 13.2 Install and check

```bash
sudo apt update
sudo apt install nginx
systemctl status nginx
```

Ubuntu starts Nginx automatically after installation. Visit `http://YOUR_SERVER_IP` and you should see the default “Welcome to nginx!” page.

If the browser cannot connect:

1. Is Nginx running? `systemctl status nginx`
2. Is it listening on port 80? `sudo ss -tulpn | grep :80`
3. Is a firewall blocking it? `sudo ufw status`
4. Is the provider's external firewall/security group allowing HTTP?
5. Are you using the right IP and `http://`, not `https://`?

### 13.3 Ports and firewalls

HTTP uses **port 80**; HTTPS uses **port 443**. A URL without a scheme may be upgraded by the browser to HTTPS (especially with HSTS), which fails if TLS is not configured yet. Type `http://` explicitly while setting up.

**UFW** (Uncomplicated Firewall) is Ubuntu's front end for netfilter. Check it:

```bash
sudo ufw status
```

If it says `Status: inactive`, it is not filtering traffic. Before enabling it, allow SSH or you may lock yourself out:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'     # ports 80 and 443
sudo ufw enable
sudo ufw status verbose
```

`Nginx Full` is an application profile installed by the Nginx package. `Nginx HTTP` opens only 80. If you changed SSH to a custom port, allow that port instead of `OpenSSH`.

Cloud providers often have a second firewall (security group). Opening UFW alone may not be enough; the provider's console must also allow the port.

### 13.4 The default site

Ubuntu's Nginx default document root is:

```text
/var/www/html
```

Its default page is `index.nginx-debian.html` (or `index.html`). The server block is in `/etc/nginx/sites-available/default`, enabled by a symlink in `/etc/nginx/sites-enabled/`.

Edit the page:

```bash
sudo vim /var/www/html/index.nginx-debian.html
```

Change the `<h1>` or title, save, and reload:

```bash
sudo systemctl reload nginx
```

Reload is enough for content changes; Nginx reads files from disk per request. Configuration changes also need a reload or restart.

You cannot edit files owned by root as a normal user. Use `sudo`, or change ownership of a directory you are meant to deploy into. Do not `chmod 777` the web root.

### 13.5 Add a second site with a server block

A **server block** is Nginx's virtual host: it matches requests by `server_name`, `listen` port and other criteria, then decides the document root or proxy target.

```bash
sudo mkdir -p /var/www/website
sudo chown -R "$USER":"$USER" /var/www/website

cat > /var/www/website/index.html <<'HTML'
<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My second site</title></head>
  <body><h1>Hello from my second Nginx site</h1></body>
</html>
HTML

sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/website
sudo vim /etc/nginx/sites-available/website
```

A minimal server block:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;

    root /var/www/website;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable it by linking into `sites-enabled`, test the configuration, then reload:

```bash
sudo ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
sudo nginx -t
sudo systemctl reload nginx
```

- `sites-available` holds definitions; `sites-enabled` holds symlinks to active ones. This is a Debian/Ubuntu convention, not an Nginx core requirement.
- `nginx -t` parses the configuration and reports syntax errors. **Always run it before reloading.** A bad config plus a restart can take the site down.
- If both the default site and your site listen on port 80 with no matching `server_name`, the default (or first) server block wins. Use `default_server` deliberately, or remove the default symlink:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

DNS: a domain's **A record** must point to the server's IPv4 address (AAAA for IPv6) before `server_name` can match public traffic. For local testing, add an entry to `/etc/hosts` on your own machine:

```text
203.0.113.10 example.com
```

HTTPS is a separate step. Use Certbot with Let's Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot obtains a certificate, edits the server block, and installs a renewal timer. Ports 80 and 443 must be reachable for validation. This is an optional supplement to the lecture but expected for any public site.

### 13.6 Proxying an application (optional supplement)

A static site is served directly. A Node/Python/Go app usually listens on a local port and Nginx forwards to it:

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The app should listen on `127.0.0.1`, not `0.0.0.0`, so it is not directly exposed. Nginx becomes the only public entry point and can handle TLS, compression and caching.

### 13.7 Common Nginx errors

| Status | Likely cause | Check |
|---|---|---|
| Connection refused/timeout | Nginx down, port blocked, wrong IP | `systemctl status nginx`, `ss -tulpn`, `ufw status`, cloud firewall |
| `403 Forbidden` | No index file, permissions, directory listing disabled | `ls -l`, `namei -l /var/www/...`, Nginx error log |
| `404 Not Found` | Wrong `root`, missing file, `try_files` mismatch | `nginx -T`, request path, file existence |
| `502 Bad Gateway` | App behind `proxy_pass` is down or wrong port | `systemctl status app`, `curl 127.0.0.1:3000` |
| `504 Gateway Timeout` | Backend too slow | App logs, timeouts, database |
| `nginx: [emerg]` on reload | Syntax/config error | `nginx -t` output, fix before reload |

Logs:

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

`access.log` shows requests and status codes; `error.log` explains why Nginx refused or failed. Read them before changing configuration randomly.

## 14. Transferring files

### 14.1 FileZilla over SFTP

**FileZilla** is a graphical file-transfer client. Use **SFTP** (SSH File Transfer Protocol), not plain FTP:

- SFTP runs over SSH, usually port **22**, and is encrypted.
- Plain FTP sends credentials and data in clear text and is unsuitable for the Internet.
- FTPS is FTP over TLS, but SFTP is the simpler choice on a Linux server that already has SSH.

Connect:

1. Open FileZilla → **File → Site Manager → New site**.
2. Protocol: **SFTP**.
3. Host: your server IP or domain (no `http://`, no trailing path).
4. Port: `22` (or your custom SSH port).
5. Logon type: Normal; username and password (or key file).
6. Connect and accept the host key only if it matches your server.
7. The left pane is your local machine; the right pane is the server.

Transfer:

- Drag a file/folder from left to right to upload; right to left to download.
- Right-click → Download for a selected remote file.
- Refresh both panes after a transfer to confirm.
- The transfer queue shows failures and their reasons.

### 14.2 Why an upload can fail with “permission denied”

The remote directory is owned by another user and is not writable by your login. This is normal for `/var/www/html` and `/etc`.

```bash
ls -ld /var/www/website
# drwxr-xr-x 2 root root 4096 ...
```

Fixes, in order of preference:

1. **Upload to a directory your user owns**, then deploy with a privileged step. For example, keep releases in `/home/om/releases` and point Nginx there.
2. **Give your deploy user ownership of the site directory** if that user is meant to manage it:

```bash
sudo chown -R om:om /var/www/website
```

3. **Use a shared group** for multiple deployers:

```bash
sudo groupadd webdeploy
sudo usermod -aG webdeploy om
sudo chgrp -R webdeploy /var/www/website
sudo chmod -R 2775 /var/www/website
```

The setgid bit (`2`) makes new files inherit the `webdeploy` group. Users must log out/in for group membership to apply.

4. **Use `sudo` with a command-line copy** (`scp` to `/tmp`, then `sudo mv`), or a deployment tool that handles privilege properly.

Do **not** fix it with `chmod -R 777`. That lets any process on the server modify your site.

### 14.3 SCP and rsync from the terminal

```bash
# Upload a file
scp index.html om@203.0.113.10:/var/www/website/

# Upload a directory recursively
scp -r website om@203.0.113.10:/home/om/

# Download a file
scp om@203.0.113.10:/var/log/nginx/error.log ./

# Rsync: efficient repeated sync (only changed data)
rsync -avz website/ om@203.0.113.10:/var/www/website/
```

- `scp` is simple and encrypted; it is fine for one-off copies.
- `rsync -a` preserves permissions/timestamps, `-v` is verbose, `-z` compresses in transit. A trailing slash on the source means “contents of this directory”; without it, the directory itself is copied inside the destination. This is a classic mistake.
- For repeated deployments, `rsync --delete` can mirror exactly but will delete remote files not present locally. Test with `--dry-run` first.

SSH keys make both tools passwordless and scriptable. Use a dedicated deploy key with restricted permissions, not your personal root key.

### 14.4 Which transfer method?

| Method | Encrypted | Best for |
|---|---|---|
| SFTP (FileZilla) | Yes | Interactive browsing and drag-and-drop |
| `scp` | Yes | One-off file/directory copy |
| `rsync` | Yes (over SSH) | Repeated sync, large trees, resumable-ish transfers |
| FTP | No | Avoid on the public Internet |
| Git | Yes (over HTTPS/SSH) | Versioned code deployment, not large binary assets |
| Object storage (S3-compatible) | Yes | Large assets, backups, CDN origin |

## 15. Capstone: a small production-shaped server

This lab combines the whole lecture. Use a disposable VM or a cheap VPS. Replace `om`, the IP and the domain with your own.

### 15.1 Build

```bash
# 1. Connect as the provider's initial user
ssh root@203.0.113.10

# 2. Update the package index and install basics
apt update
apt install -y nginx ufw

# 3. Create your daily admin user
adduser om
usermod -aG sudo om

# 4. Switch to the admin user
su - om
sudo whoami            # root

# 5. Confirm the web server
systemctl status nginx
curl -I http://localhost
```

Create the site:

```bash
sudo mkdir -p /var/www/website
sudo chown -R om:om /var/www/website

cat > /var/www/website/index.html <<'HTML'
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Linux capstone</title>
  </head>
  <body>
    <h1>Deployed on Ubuntu with Nginx</h1>
    <p>Built during the Linux for DevOps module.</p>
  </body>
</html>
HTML
```

Create and enable a server block:

```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/website
sudo vim /etc/nginx/sites-available/website
```

Use:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /var/www/website;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

`server_name _;` is a catch-all placeholder for testing by IP. For a real domain, use the domain name.

```bash
sudo ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

Firewall:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status verbose
```

Open `http://203.0.113.10` in a browser. You should see the page.

Add a backup job:

```bash
mkdir -p /home/om/backups
cat > /home/om/backup-site.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

STAMP=$(date +%F-%H%M)
DEST="$HOME/backups"
mkdir -p "$DEST"
tar -czf "$DEST/site-$STAMP.tar.gz" /var/www/website
find "$DEST" -name 'site-*.tar.gz' -mtime +7 -delete
SH
chmod +x /home/om/backup-site.sh
/home/om/backup-site.sh
ls -lh /home/om/backups
```

Schedule it:

```bash
crontab -e
```

```cron
0 8 * * * /home/om/backup-site.sh >> /home/om/backup.log 2>&1
```

Verify:

```bash
crontab -l
sleep 60          # only if testing with * * * * *
cat /home/om/backup.log
```

### 15.2 Verify the whole chain

```bash
whoami                          # om
sudo whoami                     # root
systemctl is-active nginx       # active
systemctl is-enabled nginx      # enabled
curl -I http://localhost        # 200 OK
ss -tulpn | grep ':80'          # nginx listening
sudo ufw status                 # 80/443 allowed, 22 allowed
ls -l /var/www/website          # om owns the site
crontab -l                      # backup job present
tar -tzf ~/backups/site-*.tar.gz | head   # archive readable
```

### 15.3 Break it deliberately, then fix it

On a VM or VPS you own, try one at a time:

1. Stop Nginx. Observe the browser error, then `systemctl start nginx`.
2. Change the site's port to 8080 in the server block. Run `nginx -t`, reload, and observe. Change it back.
3. `chmod 000 /var/www/website/index.html`. Request the page, read the 403 in `error.log`, then restore with `chmod 644`.
4. Rename `index.html`. Observe 403/404 depending on `try_files`, then restore.
5. Add a cron job that writes a timestamp every minute. Watch the log, then remove the job.
6. Add a directory to `PATH`, run a script from another directory, then persist the change in `.bashrc` and verify in a new shell.

Each failure teaches you which layer to inspect: DNS → network/firewall → Nginx → file permissions → application. That diagnostic order is the real skill.

## 16. Troubleshooting guide

| Symptom | First checks | Likely fix |
|---|---|---|
| `command not found` | `which cmd`, `echo $PATH`, spelling | Install package; use full path; fix `PATH` |
| `Permission denied` | `ls -l`, `id`, `namei -l path` | Correct owner/group/mode; use `sudo` for the specific command; do not use 777 |
| `No such file or directory` | `pwd`, `ls`, absolute path | Fix path, create file, check case |
| SSH `Connection refused` | Server running? Port 22 open? Correct IP? | Start SSH, fix firewall/security group, confirm IP |
| SSH `Connection timed out` | Network route, provider firewall, wrong IP | Check console, security group, local network |
| SSH host key changed | Reinstalled server or possible attack | Verify fingerprint in provider console before removing the old key |
| Website not loading | `systemctl status nginx`, `ss -tulpn`, `ufw status` | Start Nginx; allow 80/443; check cloud firewall |
| Nginx `403` | Index file, directory permissions, `root` path | Fix ownership/mode; ensure `index.html` exists |
| Nginx `404` | `root`, request path, `try_files` | Correct document root or URL |
| Nginx `502` | Backend app, `proxy_pass` port | Start app; verify with `curl 127.0.0.1:PORT` |
| `nginx -t` fails | Syntax, missing `;`/`}`, bad path | Read the reported file/line; fix before reload |
| Service starts then dies | App error, port conflict, permissions, env | `journalctl -u service -n 100`; run command manually as the service user |
| Cron job not running | Service active? Schedule? Full paths? Logs? | `systemctl status cron`, `crontab -l`, redirect output, test script manually |
| Cron works manually, not scheduled | Minimal environment, relative paths | Use absolute paths, set `PATH`, source env file |
| Disk full | `df -h`, `du -sh *`, large logs | Clean old logs/backups; rotate logs; add space |
| High load / slow | `top`/`htop`, `free -h`, `iostat`, `df -h` | Identify CPU, memory or I/O bottleneck; stop runaway process |
| `sudo: user is not in the sudoers file` | `groups`, `/etc/sudoers` | Add to `sudo`/`wheel`; re-login |
| Locked out after firewall change | Provider console | Re-enable SSH access; always allow SSH before `ufw enable` |
| Accidentally overwrote a file with `>` | Shell history, backups, snapshots | Restore from backup; use `>>` or `tee` next time |

**A diagnostic order that works**

```text
1. Identity and location     whoami, pwd, hostname
2. Process/service state     systemctl status, ps, ss
3. Configuration            nginx -t, config files, env
4. Permissions              ls -l, namei -l, id
5. Logs                     journalctl, /var/log/*
6. Network path             ping, curl, dig, traceroute
7. Resources                df -h, free -h, top
```

Change one thing at a time, and write down what you changed. Randomly restarting services destroys evidence.

## 17. What to memorise, and what to look up

Memorise the operating model, not every flag:

- `pwd`, `ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `cat`, `less`, `grep`
- `chmod`, `chown`, `ls -l`, the 4/2/1 values
- `ps aux`, `kill`, `systemctl status/start/stop/restart/reload/enable`
- `apt update/install/remove/purge`
- `echo $PATH`, `export`, `source ~/.bashrc`
- `tar -czf` / `tar -xzf`
- the five cron fields and “use full paths”
- `/etc`, `/var/log`, `/home`, `/tmp`, `/usr/bin`

Look up exact flags, Nginx directives, cron edge cases and distribution-specific paths. Knowing *what to check* is more valuable than reciting `man` pages. The [command reference](Part_06_Additional_Resources.md) exists for that lookup.

## A final pass through the story

You rent or create a Linux machine and reach it through SSH. A shell reads your commands and searches `PATH` for programs. You navigate a single filesystem tree, create files, and edit them. Users and groups decide who may read, write or execute; `sudo` grants temporary administrative power. `apt` installs software from signed repositories. Processes run programs; systemd keeps services alive across logout and reboot. Environment variables configure behaviour without changing code. Archives move many files as one; cron repeats work on a schedule. Nginx listens on a port, serves a directory, and can proxy to an application. FileZilla or `scp` moves files in and out. Logs tell you what actually happened.

Use the [visual companion](Part_06_Diagrams.md) to reconstruct the flows, the [command reference](Part_06_Additional_Resources.md) while practising, and the [interview questions](Part_06_Interview_Questions.md) and [quiz](Part_06_Revision_Quiz.md) to test recall.
