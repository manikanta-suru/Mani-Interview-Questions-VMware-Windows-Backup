# Command Reference Guide

## File and Directory Operations Commands

| Command | Description                     | Options       | Examples                  |
|---------|---------------------------------|---------------|---------------------------|
| `ls`    | List files and directories.     | `-l`, `-a`, `-h` | `ls -l`                   |
| `mkdir` | Create a new directory.         | `-p`          | `mkdir -p dir1/dir2`      |
| `rm`    | Delete files and directories.   | `-r`, `-f`    | `rm -r dir1`              |
| `cp`    | Copy files and directories.     | `-r`          | `cp -r dir1 dir2`         |
| `mv`    | Move or rename files and directories. |           | `mv file1 file2`          |

## File Permission Commands

| Command | Description                     | Options       | Examples                  |
|---------|---------------------------------|---------------|---------------------------|
| `chmod` | Change file permissions.        | `u`, `g`, `o`, `+`, `-`, `=` | `chmod u+rwx file.txt` |
| `chown` | Change file ownership.          |               | `chown user file.txt`     |
| `chgrp` | Change group ownership.         |               | `chgrp group file.txt`    |
| `umask` | Set default file permissions.   |               | `umask 022`               |

## Process Management Commands

| Command | Description                     | Options       | Examples                  |
|---------|---------------------------------|---------------|---------------------------|
| `ps`    | Display running processes.      | `-aux`        | `ps aux`                  |
| `top`   | Monitor system processes in real-time. |         | `top`                     |
| `kill`  | Terminate a process.            | `-9`          | `kill -9 PID`             |
| `pkill` | Terminate processes based on their name. |       | `pkill process_name`      |
| `pgrep` | List processes based on their name. |           | `pgrep process_name`      |

## System Information Commands

| Command   | Description                     | Options       | Examples                  |
|-----------|---------------------------------|---------------|---------------------------|
| `uname`   | Display system information.     | `-a`          | `uname -a`                |
| `hostname`| Display the system's hostname.  |               | `hostname`                |
| `uptime`  | Display system uptime.          |               | `uptime`                  |
| `lscpu`   | Display CPU information.        |               | `lscpu`                   |
| `lsusb`   | List USB devices.               |               | `lsusb`                   |

## Networking Commands

| Command   | Description                     | Options       | Examples                  |
|-----------|---------------------------------|---------------|---------------------------|
| `ifconfig`| Display network interface information. |         | `ifconfig`                |
| `ping`    | Send ICMP echo requests to a host. |           | `ping google.com`         |
| `netstat` | Display network connections and statistics. | `-tuln` | `netstat -tuln`           |
| `ss`      | Display network socket information. | `-tuln`   | `ss -tuln`                |
| `ssh`     | Securely connect to a remote server. |         | `ssh user@hostname`       |
| `scp`     | Securely copy files between hosts. |           | `scp file.txt user@hostname:/path/to/destination` |
| `wget`    | Download files from the web.    |               | `wget http://example.com/file.txt` |
| `curl`    | Transfer data to or from a server. |           | `curl http://example.com` |


Excellent idea 👍 — putting this Linux commands cheatsheet on GitHub as a Markdown (`.md`) file makes it both readable and shareable.

Here’s a **ready-to-use Markdown version** of your content, properly formatted for GitHub with clean headings, code blocks, and bullet points.

---

## 🐧 **Linux Commands Cheatsheet**

### 🧭 Core Linux Commands

| Command          | Description                                              |
| ---------------- | -------------------------------------------------------- |
| `pwd`            | Shows the current working directory.                     |
| `ls`             | Lists files and directories in the current location.     |
| `cd`             | Changes the current directory.                           |
| `tree`           | Displays directories and files in a tree-like format.    |
| `stat`           | Shows detailed information about a file.                 |
| `touch`          | Creates an empty file or updates file timestamps.        |
| `file`           | Determines the type of a file.                           |
| `basename`       | Extracts the filename from a path.                       |
| `dirname`        | Extracts the directory path from a full path.            |
| `cat`            | Displays the contents of a file.                         |
| `tac`            | Displays a file’s contents in reverse order.             |
| `less`           | Views a file one page at a time (scrollable).            |
| `more`           | Views a file one page at a time (simpler than less).     |
| `head`           | Shows the first few lines of a file.                     |
| `tail`           | Shows the last few lines of a file.                      |
| `nl`             | Displays a file with line numbers.                       |
| `strings`        | Extracts readable text from binary files.                |
| `od`             | Displays files in octal or other formats.                |
| `nano`           | Simple text editor in terminal.                          |
| `vi` / `vim`     | Advanced terminal text editor.                           |
| `emacs`          | Feature-rich terminal/GUI text editor.                   |
| `hexdump` / `hd` | Displays file contents in hexadecimal format.            |
| `xxd`            | Creates a hex dump or converts hex back to binary.       |
| `col` / `colrm`  | Formats or removes specific columns from text.           |
| `clear`          | Clears the terminal screen.                              |
| `reset`          | Resets the terminal.                                     |
| `sleep`          | Pauses execution for a specified time.                   |
| `yes`            | Repeatedly outputs a string (default “y”) until stopped. |
| `rev`            | Reverses the content of each line.                       |
| `cmp`            | Compares two files byte by byte.                         |
| `comm`           | Compares two sorted files line by line.                  |

---

### 🔍 Searching & Text Processing Commands

| Command                                                                  | Description                                                |
| ------------------------------------------------------------------------ | ---------------------------------------------------------- |
| `grep`                                                                   | Searches for patterns in files.                            |
| `egrep`                                                                  | Extended grep; supports regex patterns.                    |
| `fgrep`                                                                  | Searches for fixed strings (no regex).                     |
| `zgrep`                                                                  | Searches compressed files (.gz) using grep.                |
| `find`                                                                   | Searches files and directories by name, type, size, etc.   |
| `locate`                                                                 | Quickly finds files using a prebuilt database.             |
| `updatedb`                                                               | Updates the database used by `locate`.                     |
| `which`                                                                  | Shows the path of an executable command.                   |
| `whereis`                                                                | Shows binary, source, and man page locations of a command. |
| `type`                                                                   | Displays command type (builtin, alias, or file).           |
| `cut`, `sort`, `uniq`, `join`, `paste`, `wc`, `tr`, `awk`, `sed`, `diff` | Various text and file manipulation tools.                  |

---

### 👥 User & Group Management

(Examples)

```bash
useradd user1       # Creates a new user
passwd user1        # Sets password for user
usermod -aG sudo user1  # Adds user to sudo group
id user1            # Displays UID and GID info
whoami              # Shows the current logged-in username
```

---

### 🔐 Permissions & Ownership

| Command               | Description                         |
| --------------------- | ----------------------------------- |
| `chmod`               | Changes file/directory permissions. |
| `chown`               | Changes owner of file/directory.    |
| `chgrp`               | Changes group ownership.            |
| `umask`               | Sets default permissions mask.      |
| `getfacl` / `setfacl` | Manage Access Control Lists.        |
| `lsattr` / `chattr`   | Show or change file attributes.     |

---

### ⚙️ Process & Job Management

| Command                             | Description                                |
| ----------------------------------- | ------------------------------------------ |
| `ps aux`                            | Shows all running processes.               |
| `top` / `htop` / `atop`             | Real-time process and resource monitoring. |
| `pgrep`, `pkill`, `kill`, `killall` | Manage processes by PID or name.           |
| `jobs`, `fg`, `bg`, `disown`        | Manage background and foreground jobs.     |
| `nohup`, `setsid`                   | Run processes detached from terminal.      |

---

### 💽 Disk & Filesystem

| Command                           | Description                     |
| --------------------------------- | ------------------------------- |
| `lsblk`, `blkid`                  | Show block device info.         |
| `fdisk`, `parted`                 | Manage disk partitions.         |
| `mkfs`, `fsck`, `mount`, `umount` | Manage filesystems.             |
| `du`, `df`                        | Show disk usage and free space. |
| `swapon`, `swapoff`, `mkswap`     | Manage swap space.              |

---

### 📦 Package Management

| Command                               | Description                       |
| ------------------------------------- | --------------------------------- |
| `apt`, `apt-get`, `apt-cache`, `dpkg` | Debian/Ubuntu package tools.      |
| `yum`, `dnf`, `rpm`                   | RHEL/CentOS/Fedora package tools. |
| `zypper`                              | openSUSE package manager.         |
| `snap`, `flatpak`                     | Universal app systems.            |

---

### 🌐 Networking & Remote Access

| Command                                       | Description                        |
| --------------------------------------------- | ---------------------------------- |
| `ip a`, `ifconfig`                            | Show network interfaces.           |
| `ping`, `traceroute`, `mtr`                   | Network connectivity tools.        |
| `curl`, `wget`, `scp`, `rsync`, `ssh`, `sftp` | File transfer & remote access.     |
| `ss`, `netstat`, `nmap`, `tcpdump`            | Network inspection and monitoring. |

---

### 🧰 System Information

| Command                                 | Description                       |
| --------------------------------------- | --------------------------------- |
| `uname -a`                              | Show system and kernel info.      |
| `lsb_release -a`, `cat /etc/os-release` | Show OS details.                  |
| `lscpu`, `lspci`, `lsusb`, `lshw`       | Show hardware info.               |
| `uptime`, `free -h`, `vmstat`, `iostat` | Show performance and system load. |

---

### 🔒 Security & Access Control

| Command                                  | Description                  |
| ---------------------------------------- | ---------------------------- |
| `getenforce`, `setenforce`, `sestatus`   | Manage SELinux mode.         |
| `ufw`, `firewall-cmd`, `iptables`, `nft` | Firewall management.         |
| `gpg`, `openssl`                         | Encryption and certificates. |
| `sha256sum`, `md5sum`                    | File integrity checks.       |

---

### 🧩 Fun / Miscellaneous

| Command                           | Description                           |
| --------------------------------- | ------------------------------------- |
| `cowsay`, `fortune`, `sl`         | Fun terminal tools.                   |
| `cal`, `date`, `figlet`, `toilet` | Display calendar, time, or ASCII art. |

---

### 🧠 Practice Recommendations

* 🖥 **Cloud:** Use AWS / Azure / GCP free-tier VMs.
* 💻 **Local VM:** Try VirtualBox with Ubuntu or CentOS.
* 📚 **Learn by Doing:**
  Use `man <command>` or `<command> --help` to explore.
* 🔗 **Combine Commands:**
  Example:

  ```bash
  ps aux | grep nginx
  du -sh * | sort -h
  ```

---

### 🏁 Summary

This cheatsheet is a comprehensive guide to Linux command-line essentials for:

* System administration
* DevOps / Cloud engineering
* Troubleshooting and automation

---

## 📘 How to Upload to GitHub

1. Create a new repo (e.g., `linux-commands-cheatsheet`).
2. Add a new file named `README.md`.
3. Paste the Markdown content above.
4. Commit and push it.

Example Git commands:

```bash
git init
git add README.md
git commit -m "Added Linux Commands Cheatsheet"
git branch -M main
git remote add origin https://github.com/<yourusername>/linux-commands-cheatsheet.git
git push -u origin main
```

---

Would you like me to **generate this Markdown as a downloadable `.md` file** for direct upload to GitHub?


```
