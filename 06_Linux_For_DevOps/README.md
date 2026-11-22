# Part 06 — Linux for DevOps

This part takes you from “I have never opened a Linux terminal” to operating a small Linux server with confidence. It follows **CodeWithHarry's Linux Tutorial For Beginners | Complete Linux Course for DevOps** (2:27:42), then adds the basic concepts, commands, safety notes and practice that a first-time learner needs.

**Lecture:** [Linux Tutorial For Beginners | Complete Linux Course for DevOps](https://youtu.be/ldgl9Bd3lLg) — 00:00–2:27:42.

This module is a companion to the five-part networking and system-design course. It is **not** a segment of the original 4:19:58 lecture, so it is numbered separately. Read it before, after or alongside Parts 1–5; nothing here depends on the earlier chapters, although Part 3's TCP/ports material and Part 4's TLS material become more useful once you host a real site.

## Choose the file that serves your next step

| File | Use it for |
|---|---|
| [Study Notes](Part_06_Study_Notes.md) | Connected explanations, worked examples, command walkthroughs and small labs |
| [Diagrams](Part_06_Diagrams.md) | Visual explanations of the terminal, filesystem, permissions, processes, boot flow and request path |
| [Command Reference and Resources](Part_06_Additional_Resources.md) | Copy-ready command tables, cron/permission cheat sheets, practice labs and curated reading |
| [Interview Questions](Part_06_Interview_Questions.md) | Twelve essential answers followed by six optional reasoning scenarios |
| [Revision Quiz](Part_06_Revision_Quiz.md) | 30 questions across recall, commands, troubleshooting and practical tasks, with a full answer key |
| **PDF** | [Part_06_Linux_For_DevOps_Complete_Notes.pdf](Part_06_Linux_For_DevOps_Complete_Notes.pdf) — the study notes, diagrams, reference, interview questions and quiz in one print-ready book |

## The learning sequence

The lecture has a practical shape: understand what Linux is, obtain a machine, learn to move around it, then learn to administer it.

| Chapter | Time | What it establishes |
|---|---|---|
| Introduction | 05:00 | Why Linux is a baseline DevOps skill |
| What is Linux | 05:58 | Kernel, operating system and distribution |
| History of Linux | 08:31 | Unix → GNU/MINIX → Linux → distributions and Android |
| Getting an Online Linux Server | 12:06 | VPS, SSH and the cloud terminal |
| Installing Linux Through VirtualBox on Windows | 18:44 | Virtualisation, ISO, VM resources and installation |
| Installing Linux Through WSL in Windows | 26:58 | A real Linux user space inside Windows |
| Installing Linux Through VirtualBox on Mac | 32:23 | The same VM workflow on macOS |
| Basic Linux Commands | 36:23 | Navigation, files, editing and history |
| Creating Users | 54:35 | Root risk, accounts, `sudo` and home directories |
| Package Management | 1:00:55 | `apt`, repositories, install/remove/purge |
| Groups and Permissions | 1:07:22 | Groups, `ls -l`, `chown`, `chgrp`, symbolic and numeric `chmod` |
| Processes and Services | 1:19:17 | `ps`, `top`, `kill`, `systemctl`, enable/reload |
| Environment Variables, Path and bashrc | 1:33:30 | Shell vs environment variables, `PATH`, persistence |
| Archives and Compression | 1:53:31 | `tar`, `gzip`, `zip` and safe extraction |
| Cronjobs | 2:01:44 | Scheduling, five cron fields, full paths and logs |
| Understanding Linux Filesystem | 2:12:24 | `/`, `/etc`, `/var`, `/usr`, `/tmp` and friends |
| Understanding Nginx | 2:17:05 | Serving a site, ports, firewall and server blocks |
| Using FileZilla to transfer files | 2:22:58 | SFTP, drag-and-drop and permission failures |
| Conclusion | 2:26:33 | Recap, the limits of the course and what to learn next |

## Ready-to-explain outcomes

- [ ] Explain the difference between the Linux kernel, an operating system and a distribution.
- [ ] Obtain a Linux environment through a VPS, a virtual machine or WSL, and explain the trade-offs.
- [ ] Navigate the filesystem with absolute and relative paths and edit a file in `vim`.
- [ ] Create a user, grant administrative access deliberately, and explain why daily work should not use `root`.
- [ ] Install, inspect and remove software with `apt`.
- [ ] Read `ls -l`, then set ownership and permissions with `chown`, `chgrp` and `chmod`.
- [ ] Find and stop a process, then manage a long-running service with `systemctl`.
- [ ] Explain `PATH`, persist an environment variable, and avoid breaking the shell.
- [ ] Create and extract `.tar`, `.tar.gz` and `.zip` archives.
- [ ] Schedule a job with `cron`, using full paths and a log file.
- [ ] Explain the purpose of the main top-level directories.
- [ ] Serve a page with Nginx, open the required port safely, and add a second site.
- [ ] Transfer a file with FileZilla or `scp` and fix a permission-denied upload.

## How to study this module

1. **Get a machine first.** A cheap VPS is the closest to real work; VirtualBox is best for a completely disposable machine; WSL is the fastest route on Windows. Every exercise can be done on any of the three.
2. **Type the commands.** Reading commands creates recognition; typing them creates memory. Break things in a throwaway VM on purpose.
3. **Keep a lab log.** For each command, note what you expected, what happened, and how you verified it.
4. **Use the diagrams when a concept feels abstract.** The [visual companion](Part_06_Diagrams.md) is designed to be read beside the notes.
5. **Finish with the quiz.** Anything you miss points to one specific section, not to “Linux being hard”.

## A note on the source and corrections

The lecture is in Hindi/Hinglish. These notes are an explanatory adaptation, not a verbatim transcript. The sequence, examples and commands follow the lecture; explanations, corrections and additional basics are marked as supplements where they go beyond it.

Places where this module deliberately adds precision:

- “Linux is secure” is too broad. Linux has a strong permission model and open review, but security still depends on updates, configuration and access control.
- `apt upgrade` is not inherently bad. It is a real system change that belongs in a maintenance window with testing and a recovery plan, not an automatic habit on a production server.
- `chmod 777` and `kill -9` “work”, but they remove protection or skip graceful cleanup. The notes explain safer defaults.
- `sites-available` and `sites-enabled` are Debian/Ubuntu Nginx conventions, and the enabling symlink is part of the complete workflow.
- A VPS is a virtual server on shared physical hardware. “Cloud” changes who manages the hardware, not whether hardware exists.

**Course home:** [Computer Networking & System Design](../README.md)  
**Next:** [Command Reference](Part_06_Additional_Resources.md) · [Interview Questions](Part_06_Interview_Questions.md) · [Revision Quiz](Part_06_Revision_Quiz.md)
