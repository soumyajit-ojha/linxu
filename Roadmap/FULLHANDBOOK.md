# Linux + Bash: The Complete Beginner-to-DevOps Handbook

Nov 21, 2026 · @Soumyajit

## How to use this handbook

Your roadmap is solid. This handbook follows it in order and adds the pieces a working DevOps engineer uses every day that were missing. Read one section, then type every command yourself. Reading builds understanding; typing builds memory.

**Set up a practice machine first.** You need a Linux system you are allowed to break. Pick one:

- **Windows:** install WSL2 (run `wsl --install` in PowerShell as Administrator, then reboot). You get Ubuntu inside Windows.
- **macOS or Windows:** install VirtualBox or UTM and create an Ubuntu Server virtual machine.
- **Cloud:** a small free-tier VM on AWS, Google Cloud, Oracle Cloud or DigitalOcean. This also teaches you SSH from day one.

This handbook uses **Ubuntu** (Debian family) for examples, and notes where RHEL/Rocky/Fedora differ.

**The memory method.** For every command, ask yourself three questions: what problem does it solve, what does its name stand for, and what would I type at 3 a.m. when a server is down? Each section gives you a memory hook for exactly that.

**Three habits that will save you:**

1. `man <command>` opens the manual. `<command> --help` gives a short summary. `tldr <command>` (install with `sudo apt install tldr`) gives practical examples.
2. Press **Tab** to autocomplete names. Press it twice to see all options. Press **Up arrow** for previous commands, and **Ctrl+R** to search your history.
3. Read error messages slowly. Ninety percent of the time the answer is in the message.

**What I added to your roadmap:**

| Added topic | Why a DevOps engineer needs it |
| --- | --- |
| What Linux is (kernel, distro, shell) | You can't troubleshoot what you can't picture |
| Absolute vs relative paths, hard and soft links | Used constantly in configs and deployments |
| Text editors (nano, vim) | Servers have no VS Code; you edit configs in the terminal |
| umask, SUID, SGID, sticky bit | Explains "weird" permissions and security findings |
| /etc/shadow, usermod, groupadd, visudo | Real user administration, not just creation |
| Package management (apt, dnf) | Every server setup starts with installing software |
| Signals (SIGTERM vs SIGKILL), nice, jobs | Stopping processes safely |
| Writing your own systemd service | Running your app as a proper service |
| Load average, swap, OOM killer | The real reasons servers slow down and apps die |
| Ports, DNS, /etc/hosts, firewall (ufw) | Most "app is down" issues are network issues |
| SSH keys, \~/.ssh/config, scp, rsync | How you reach and copy to every server |
| cron and systemd timers | Backups and scheduled jobs |
| tar and gzip | Backups, releases, log archives |
| set -euo pipefail, case, arrays, read | Scripts that fail safely instead of silently |
| A troubleshooting playbook | Your Stage 2 exit criterion, as a repeatable method |

## Part 0: What Linux actually is

Linux is a **kernel**: the core program that talks to the hardware and shares it fairly between all running programs. Everything else you touch is built around it.

**The restaurant analogy (remember this):**

| Restaurant | Linux | What it does |
| --- | --- | --- |
| Kitchen equipment | Hardware (CPU, RAM, disk, network card) | Does the actual work |
| Head chef | Kernel | Decides who uses which equipment and when |
| Waiter | Shell (bash) | Takes your order in words and passes it to the kitchen |
| Menu items | Commands and programs (ls, nginx, docker) | Things you can order |
| The restaurant brand | Distribution (Ubuntu, Debian, RHEL, Rocky, Alpine) | Kernel + tools + package manager, packaged together |
| The table you sit at | Terminal | The window where you talk to the waiter |

So when you type `ls` in a terminal, the **terminal** shows your text, the **shell** understands it and starts the `ls` **program**, `ls` asks the **kernel** to read the directory, and the kernel reads the **disk**.

**Distributions you will meet as a DevOps engineer:**

- **Ubuntu / Debian** use `apt` and `.deb` packages. Most common on cloud servers.
- **RHEL / Rocky Linux / AlmaLinux / Fedora / Amazon Linux** use `dnf` (older: `yum`) and `.rpm` packages. Common in enterprises.
- **Alpine** uses `apk`. Tiny; very common inside Docker images.

The commands in this handbook work on all of them. Mainly the package manager differs.

**Two ideas that explain all of Linux:**

1. **Everything is a file.** Regular files, directories, your hard disk (`/dev/sda`), your terminal (`/dev/tty`), even running processes (`/proc/1234`) appear as files. That's why the same tools (`cat`, `ls`, permissions) work on almost everything.
2. **Small tools, combined.** Each command does one thing well. You connect them with pipes (`|`) to solve big problems. `ps aux | grep nginx | wc -l` = list processes, keep nginx lines, count them.

**Reading the prompt.** When you open a terminal you see something like:

```
alex@web01:~/projects$
```

That means user `alex`, on machine `web01`, currently in folder `~/projects` (`~` = your home directory). `$` means a normal user. `#` means you are **root**, the all-powerful administrator, so be careful.

**Anatomy of a command:**

```
ls      -l -a         /var/log
│       │             │
command options/flags argument (what to act on)
```

Short options can be combined: `ls -l -a` = `ls -la`. Long options use two dashes: `ls --all`. Linux is **case-sensitive**: `File.txt` and `file.txt` are two different files.

## Level 1: The filesystem

Linux has **one single tree** that starts at `/` (called "root"). There is no `C:` or `D:` drive like Windows. Extra disks and USB sticks are attached somewhere inside this tree (that's called **mounting**, covered in Disk management).

**Picture it as a house.** `/` is the front door, and every room has one job:

```
/                   the front door: everything starts here
├── bin    → /usr/bin   essential commands (ls, cp, cat)          "binaries"
├── sbin   → /usr/sbin  admin commands (fdisk, reboot)             "system binaries"
├── boot            kernel + bootloader (GRUB) to start the machine
├── dev             hardware as files (/dev/sda disk, /dev/null)   "devices"
├── etc             ALL configuration files (text)                 "edit to configure"
├── home            users' personal folders (/home/alex)
├── root            the root user's home (NOT the same as /)
├── lib    → /usr/lib   shared libraries programs need             like Windows .dll
├── opt             optional third-party software (/opt/google)
├── tmp             temporary files, wiped on reboot, anyone can write
├── usr             installed programs, libraries, docs (read-only-ish)
│   ├── bin         most user commands
│   ├── local       software YOU compiled/installed manually
│   └── share       docs, man pages, icons
├── var             data that VARIES: logs, caches, databases, websites
│   ├── log         logs ← you will live here when troubleshooting
│   ├── lib         app state (/var/lib/docker, /var/lib/mysql)
│   └── www         web content (often for nginx/apache)
├── proc            live info about processes and kernel (virtual, in RAM)
├── sys             live info about hardware/drivers (virtual)
├── run             runtime data since boot (PID files, sockets)
├── mnt             temporary manual mounts
├── media           auto-mounted USB/CD
└── srv             data served by this server (sometimes used)
```

**The DevOps view: where do I look?**

| Question at work | Where to look |
| --- | --- |
| Where is nginx's config? | `/etc/nginx/nginx.conf`, `/etc/nginx/sites-enabled/` |
| Why did the app crash? | `/var/log/`, or `journalctl` |
| Where does Docker store images and volumes? | `/var/lib/docker/` |
| Disk full: what's usually big? | `/var/log`, `/var/lib/docker`, `/tmp`, `/home` |
| Where are user accounts defined? | `/etc/passwd`, `/etc/shadow`, `/etc/group` |
| What's this server's hostname / DNS? | `/etc/hostname`, `/etc/hosts`, `/etc/resolv.conf` |
| Which disks mount at boot? | `/etc/fstab` |
| Scheduled jobs? | `/etc/crontab`, `/etc/cron.d/` |
| How many CPUs / how much RAM? | `/proc/cpuinfo`, `/proc/meminfo` |
| Where do I put my company's app? | `/opt/myapp` or `/srv/myapp` |

**Memory hooks:**

- **etc** = "**E**dit **T**o **C**onfigure". Every config is plain text you can open and read.
- **var** = "**var**ies". Things that grow: logs, caches, databases. Disk-full problems usually live here.
- **usr** = "**U**nix **S**ystem **R**esources", not "user". Users live in `/home`.
- **/root vs /**: `/` is the top of the tree; `/root` is just the root user's home folder.
- **/proc and /sys** don't exist on disk. The kernel invents them live. Try `cat /proc/uptime`.
- **/dev/null** is a black hole: anything written there disappears. You'll use it in scripts.

**Absolute vs relative paths (essential):**

- An **absolute path** starts with `/` and works from anywhere: `/var/log/syslog`.
- A **relative path** starts from where you are now: if you're in `/var`, then `log/syslog` points to the same file.
- Special names: `.` = this directory, `..` = parent directory, `~` = your home, `-` (with `cd`) = the previous directory.

Rule of thumb: **use absolute paths in scripts and configs**, because a script may run from any directory.

**Hidden files:** any name starting with a dot is hidden, like `~/.bashrc` or `~/.ssh`. Show them with `ls -a`.

## Level 1: Navigation and file commands

These commands are your hands and eyes on a server. Each one below has its name meaning, the forms you will actually use, and a trap to avoid.

### Where am I, what's here, go there

**`pwd`** = **p**rint **w**orking **d**irectory. Shows where you are.

```
$ pwd
/home/alex
```

**`ls`** = **l**i**s**t. Shows what's in a directory.

```
ls              # list current directory
ls -l           # long format: permissions, owner, size, date
ls -a           # include hidden files (.bashrc)
ls -lh          # human-readable sizes (4.0K, 12M, 1.5G)
ls -lt          # sort by time, newest first
ls -ltr         # oldest first (newest at the bottom: great for logs)
ls -lS          # sort by size, biggest first
ls -R           # recursive: also list subfolders
ls -ld /etc     # info about the directory itself, not its contents
```

**`cd`** = **c**hange **d**irectory.

```
cd /var/log     # absolute path
cd log          # relative path
cd ..           # up one level
cd ../..        # up two levels
cd ~   or  cd   # go home
cd -            # back to the previous directory (like a TV remote's "last channel")
```

### Create

**`mkdir`** = **m**a**k**e **dir**ectory.

```
mkdir app
mkdir -p app/config/prod      # -p creates parents too, and doesn't fail if it exists
mkdir -p project/{src,tests,docs}   # brace expansion: three folders at once
```

**`touch`** creates an empty file, or updates the timestamp of an existing one (it "touches" it).

```
touch notes.txt
touch file{1..5}.log      # file1.log ... file5.log
```

### Copy, move, delete

**`cp`** = **c**o**p**y.

```
cp a.txt b.txt              # copy file
cp a.txt /tmp/              # copy into a directory
cp -r src/ backup/          # -r = recursive: needed for directories
cp -a /etc/nginx /root/nginx-backup   # -a = archive: keep permissions, owners, timestamps
cp -i a.txt b.txt           # -i = ask before overwriting
```

**`mv`** = **m**o**v**e. There is no rename command: renaming is moving to a new name.

```
mv old.txt new.txt          # rename
mv file.txt /tmp/           # move
mv -i a b                   # ask before overwriting
```

**`rm`** = **r**e**m**ove. **There is no recycle bin. Deleted means gone.**

```
rm file.txt
rm -i file.txt              # ask first
rm -r folder/               # delete a directory and everything in it
rm -rf folder/              # -f = force, never ask: dangerous
rmdir emptyfolder           # only removes EMPTY directories (safe)
```

Trap: `rm -rf /` or `rm -rf $DIR/*` when `$DIR` is empty deletes everything. Before any `rm -rf`, run `ls` on the same path first to see what you will delete.

### Read files

**`cat`** = con**cat**enate. Prints a whole file. Good for short files.

```
cat /etc/hostname
cat -n script.sh            # with line numbers
cat a.txt b.txt > both.txt  # join files (the original meaning of "concatenate")
```

**`less`** opens a file page by page. Use it for anything long. (The joke: "less is more", it replaced the older `more` command.)

| Key in less | Action |
| --- | --- |
| Space / b | Page down / page up |
| g / G | Go to start / end |
| /word | Search forward (n = next, N = previous) |
| F | Follow the file live, like tail -f (Ctrl+C to stop) |
| q | Quit |

**`head`** and **`tail`** show the beginning and the end.

```
head file.log               # first 10 lines
head -n 20 file.log         # first 20 lines
tail file.log               # last 10 lines
tail -n 100 file.log        # last 100 lines
tail -f /var/log/nginx/access.log   # FOLLOW: watch new lines live (Ctrl+C stops)
tail -F app.log             # follow, even if the log is rotated and recreated
```

Memory hook: `tail -f` is the DevOps stethoscope. Open it, reproduce the error in your browser, watch the log line appear.

**`file`** tells you what a file really is, ignoring its extension.

```
$ file mystery
mystery: gzip compressed data
$ file /bin/ls
/bin/ls: ELF 64-bit LSB pie executable, x86-64 ...
```

**`wc`** = **w**ord **c**ount. `wc -l file` counts lines. Very common in pipes.

### Find files

**`find`** searches the real filesystem, live. Slow on huge trees but always accurate. Pattern: `find WHERE WHAT ACTION`.

```
find /etc -name "nginx.conf"           # by exact name
find / -iname "*.log" 2>/dev/null      # -iname = ignore case; hide permission errors
find /var/log -type f -size +100M      # files bigger than 100 MB
find . -type d -name node_modules      # directories only
find /tmp -type f -mtime +7            # modified more than 7 days ago
find /tmp -type f -mtime +7 -delete    # ...and delete them (test without -delete first!)
find . -name "*.sh" -exec chmod +x {} \;   # run a command on each result
find /home -user alex                  # files owned by alex
find / -perm -4000 2>/dev/null         # files with SUID bit (security audits)
```

**`locate`** searches a pre-built database, so it's instant but can be out of date. Install with `sudo apt install plocate`, refresh with `sudo updatedb`.

```
locate nginx.conf
```

**`which`** and **`type`** tell you which program runs when you type a command: `which python3` → `/usr/bin/python3`.

| Tool | Searches | Speed | Up to date? |
| --- | --- | --- | --- |
| find | The disk, live | Slow on big trees | Always |
| locate | A database | Instant | Only after updatedb |
| which | Your PATH (commands only) | Instant | Always |

### Links (added): two names for one file

To understand links, know that a file has two parts: its **data + metadata**, identified by an **inode** number, and a **name** in a directory that points to that inode. See inodes with `ls -i`.

```
ln target.txt hard.txt          # HARD link: a second name for the same inode
ln -s /opt/app/v2 /opt/app/current   # SOFT (symbolic) link: a shortcut pointing to a path
ls -l /opt/app/current
lrwxrwxrwx 1 root root 11 ... current -> /opt/app/v2
```

|  | Hard link | Soft (symbolic) link |
| --- | --- | --- |
| Points to | The inode (the data) | A path (a name) |
| Original deleted | Data still there | Link breaks ("dangling") |
| Works on directories | No | Yes |
| Across disks | No | Yes |
| Like in Windows | No equivalent | Shortcut |

DevOps use: zero-downtime deploys. Put each release in `/opt/app/releases/v1`, `v2`, … and point `/opt/app/current` at the active one. Rolling back = repointing the symlink with `ln -sfn /opt/app/releases/v1 /opt/app/current`.

### Wildcards (globbing)

The shell expands these **before** the command runs:

```
*        any characters        ls *.log
?        exactly one character ls file?.txt
[abc]    one of these           ls file[123].txt
{a,b}    each alternative       cp config.{yml,bak}   → cp config.yml config.bak
```

Tip: test a dangerous wildcard with `echo` first: `echo rm *.log` shows what would run.

## Level 1: Text editors (added)

On a server you edit configs inside the terminal. Learn **nano** to be productive today and **vim** to survive anywhere, because vim (or `vi`) exists on almost every Linux machine, including minimal ones.

**nano: the easy one.** `nano file.txt` opens it. The shortcuts are printed at the bottom; `^` means Ctrl.

| Keys | Action |
| --- | --- |
| Ctrl+O, then Enter | Save ("write Out") |
| Ctrl+X | Exit |
| Ctrl+W | Search ("Where is") |
| Ctrl+K / Ctrl+U | Cut line / paste line |

**vim: the powerful one.** The key idea: vim has **modes**. Keys mean different things in each mode, which is why beginners get "stuck".

- **Normal mode** (the start): keys are commands. Press **Esc** to always come back here.
- **Insert mode**: press `i`, then type text normally.
- **Command mode**: press `:` then a command.

The survival kit:

| Keys (from Normal mode) | Action |
| --- | --- |
| i | Start typing (insert) |
| Esc | Stop typing, back to Normal |
| :w | Save (write) |
| :q | Quit |
| :wq or :x | Save and quit |
| :q! | Quit WITHOUT saving (the panic button) |
| dd / yy / p | Delete line / copy (yank) line / paste |
| u / Ctrl+R | Undo / redo |
| /text then n | Search, next match |
| gg / G | Top / bottom of file |
| :set number | Show line numbers |

Memory hook: **"Esc, colon, w, q"**. If you're ever lost in vim, press Esc twice and type `:q!` to get out safely.

**Safe editing habit for important configs:**

```
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak   # backup first
sudo nano /etc/nginx/nginx.conf                           # edit
sudo nginx -t                                             # test the config
sudo systemctl reload nginx                               # apply only if the test passed
```

Backup, edit, test, apply. Many services have a test command (`nginx -t`, `sshd -t`, `apachectl configtest`).

## Level 1: File permissions

Every file answers one question: **who can do what?** "Who" has three answers (owner, group, others) and "what" has three answers (read, write, execute). That's 3 × 3 = 9 permission bits. Master this and you master half of Linux security.

### Reading `ls -l`, character by character

```
$ ls -l deploy.sh
-rwxr-xr--  1  alex  devops  1024  Nov 21 10:00  deploy.sh
│└┬┘└┬┘└┬┘  │  │     │       │     │             │
│ │  │  │   │  │     │       │     │             └ name
│ │  │  │   │  │     │       │     └ last modified
│ │  │  │   │  │     │       └ size in bytes
│ │  │  │   │  │     └ GROUP owner
│ │  │  │   │  └ USER owner
│ │  │  │   └ number of hard links
│ │  │  └ OTHERS: r-- (read only)
│ │  └ GROUP:  r-x (read, execute)
│ └ USER:   rwx (everything)
└ file type
```

File type, the first character: `-` regular file, `d` directory, `l` symbolic link, `c`/`b` character/block device (in `/dev`), `s` socket, `p` pipe.

### What r, w, x really mean

The meaning differs for files and directories. **This table is the part most people get wrong:**

| Bit | On a FILE | On a DIRECTORY |
| --- | --- | --- |
| r (read) | See the contents (`cat`) | List the names inside (`ls`) |
| w (write) | Change the contents | Create, delete, rename files inside |
| x (execute) | Run it as a program | Enter it (`cd`) and reach files inside |

Two surprises that follow from this table:

- **Deleting a file depends on the directory's `w`, not the file's.** You can delete a read-only file if you can write to its folder.
- **A directory needs `x` to be useful.** `r` without `x` lets you see names but not open anything inside.

Linux checks in order: are you the owner? Use the owner bits. Else, are you in the group? Use group bits. Else, use others. It stops at the first match. (Root ignores almost all of this.)

### Numbers (octal): the 4-2-1 trick

Each letter has a value. Add them per group:

```
r = 4      w = 2      x = 1      - = 0

rwx = 4+2+1 = 7        r-x = 4+0+1 = 5
rw- = 4+2+0 = 6        r-- = 4+0+0 = 4
```

Three digits = owner, group, others. So `chmod 754` = `rwx r-x r--`.

| Number | Letters | Meaning | Typical use |
| --- | --- | --- | --- |
| 755 | rwxr-xr-x | Owner does all; everyone can read and run | Scripts, programs, directories |
| 644 | rw-r--r-- | Owner edits; everyone can read | Normal files, configs, web pages |
| 700 | rwx------ | Only the owner, for everything | Private dirs, `~/.ssh` |
| 600 | rw------- | Only the owner reads/writes | Private keys, `.env` secrets |
| 640 | rw-r----- | Owner edits, group reads, others nothing | Logs, configs with passwords |
| 777 | rwxrwxrwx | Everyone does everything | Almost never. A red flag |

Memory hook: **7 = all, 6 = read+write, 5 = read+run, 4 = read only, 0 = nothing.** Files usually end in **4** (644), directories and scripts in **5** (755), secrets in **0** (600/700).

### chmod: change the mode

Two ways: numbers (set everything at once) or symbols (change one thing).

```
chmod 755 deploy.sh        # numeric
chmod 600 ~/.ssh/id_ed25519

chmod +x deploy.sh         # symbolic: add execute for everyone
chmod u+x deploy.sh        # u=user(owner)  g=group  o=others  a=all
chmod g-w file             # remove write from group
chmod o= file              # others: nothing
chmod u=rw,g=r,o= file     # = 640
chmod -R 755 /var/www/site # recursive (careful: makes files executable too)
```

Better recursive fix for web folders (directories 755, files 644):

```
find /var/www/site -type d -exec chmod 755 {} \;
find /var/www/site -type f -exec chmod 644 {} \;
```

### chown and chgrp: change the owner

```
sudo chown alex file.txt             # new user owner
sudo chown alex:devops file.txt      # user and group at once
sudo chown -R www-data:www-data /var/www/site   # whole tree, for nginx
sudo chgrp devops file.txt           # group only (same as chown :devops)
```

Only root can give a file away to another user, so `chown` usually needs `sudo`.

### umask (added): default permissions for new files

New files start at 666, new directories at 777, then the **umask** removes bits. The usual umask is `022`, which removes write from group and others:

```
files:        666 - 022 = 644
directories:  777 - 022 = 755
```

Check yours with `umask`. That's why new files you create are 644 without you doing anything.

### Special bits (added)

You will see these in `ls -l` output and security audits:

| Bit | Number | Shows as | Effect | Real example |
| --- | --- | --- | --- | --- |
| SUID | 4000 | `rws` in owner | Program runs as the file's owner | `/usr/bin/passwd` runs as root so you can change your password |
| SGID | 2000 | `rws` in group | On a dir: new files inherit the dir's group | Shared team folders |
| Sticky | 1000 | `t` at the end | In a shared dir, only a file's owner can delete it | `/tmp` is `drwxrwxrwt` |

```
chmod 2775 /srv/shared      # SGID shared folder for a team
ls -ld /tmp                 # drwxrwxrwt  ← sticky bit
```

### Classic permission errors and fixes

| Error | Likely cause | Fix |
| --- | --- | --- |
| `bash: ./deploy.sh: Permission denied` | No execute bit | `chmod +x deploy.sh` |
| SSH: `UNPROTECTED PRIVATE KEY FILE` | Key is too open | `chmod 600 ~/.ssh/id_*` and `chmod 700 ~/.ssh` |
| nginx 403 Forbidden | nginx user can't read files or enter dirs | Check with `namei -l /var/www/site/index.html`, fix ownership/755/644 |
| Can't write to a folder | No `w` on the directory | `chown` the folder or add group write |

`namei -l <path>` shows permissions of every directory on the way to a file. It's the best tool for "why can't this user reach this file?"

## Level 1: Users, groups and sudo

Linux is multi-user: every file and every process belongs to a user. Groups let you give permissions to many users at once. **Root** (user ID 0) can do anything, so you work as a normal user and borrow root's power with `sudo` only when needed.

There are three kinds of users:

- **root**, UID 0: the administrator.
- **System/service users**, UID 1–999 (e.g. `www-data`, `mysql`, `nobody`): run services with limited rights, so a hacked web server can't take over the machine. They usually can't log in.
- **Human users**, UID 1000+: you and your teammates.

### Who am I?

```
$ whoami
alex
$ id
uid=1000(alex) gid=1000(alex) groups=1000(alex),27(sudo),998(docker)
$ groups
alex sudo docker
$ id www-data           # check any user
$ who                   # who is logged in right now
$ last                  # login history
```

Read `id` like this: my user number is 1000, my primary group is `alex`, and I'm also in `sudo` (can run admin commands) and `docker` (can use Docker without sudo).

### sudo vs su

|  | sudo | su |
| --- | --- | --- |
| Means | "**s**uper**u**ser **do**" | "**s**witch **u**ser" |
| Does | Runs ONE command as root | Opens a shell as another user |
| Asks for | YOUR password | THAT user's password |
| Logged | Yes, every command | Only the switch |
| Use | Daily admin work | Rarely |

```
sudo apt update                 # one command as root
sudo -i                         # root shell (via sudo, your password)
sudo -u postgres psql           # run as another user, not root
su - alex                       # become alex; "-" loads alex's full environment
sudo !!                         # re-run the last command with sudo
exit                            # leave a root/other-user shell
```

Memory hook: **sudo = borrow the key for one door; su = move into someone else's house.**

### Creating and managing users

```
sudo useradd -m -s /bin/bash -G sudo,docker alex
#            │   │            └ extra groups
#            │   └ login shell
#            └ create /home/alex
sudo passwd alex                 # set the password

sudo adduser alex                # Debian/Ubuntu friendly version: asks questions, does everything
sudo usermod -aG docker alex     # ADD to a group (-a = append!)
sudo usermod -L alex             # lock the account
sudo userdel -r alex             # delete user and home dir
sudo groupadd devops             # create a group
sudo gpasswd -d alex docker      # remove from a group
sudo chage -l alex               # password expiry info
```

Trap: `usermod -G docker alex` without `-a` **replaces** all of alex's groups with just docker, and alex loses sudo. Always `-aG`. Also, new group membership applies only after logging out and in again (or `newgrp docker`).

### The account files

**`/etc/passwd`**: one line per user. Readable by everyone. Despite the name, no passwords are inside anymore.

```
alex:x:1000:1000:Alex Kim:/home/alex:/bin/bash
 │   │  │    │     │        │          └ login shell
 │   │  │    │     │        └ home directory
 │   │  │    │     └ comment (full name)
 │   │  │    └ primary group ID (GID)
 │   │  └ user ID (UID)
 │   └ x = password is in /etc/shadow
 └ username
```

Service users have a shell like `/usr/sbin/nologin`, which blocks logins.

**`/etc/shadow`** (added): the real password hashes. Only root can read it.

```
alex:$y$j9T$Kd...hash...:20048:0:99999:7:::
```

The hash starts with an algorithm marker (`$y$` = yescrypt, `$6$` = SHA-512). `!` or `*` instead of a hash means login with a password is disabled.

**`/etc/group`**: one line per group.

```
docker:x:998:alex,maria
  │    │  │    └ members (supplementary)
  │    │  └ GID
  │    └ password placeholder
  └ group name
```

**`/etc/sudoers`**: who may use sudo and for what. **Never edit it directly.** Use `sudo visudo`, which checks your syntax before saving. A typo in this file can lock everyone out of sudo.

```
root    ALL=(ALL:ALL) ALL
%sudo   ALL=(ALL:ALL) ALL          # % = a group: members of "sudo" can do anything
#  │     │    │   │   └ which commands
#  │     │    │   └ as which group
#  │     │    └ as which user
#  │     └ on which hosts
#  └ who
deploy  ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp   # CI user: one command, no password
```

Best practice: put custom rules in their own file, `sudo visudo -f /etc/sudoers.d/deploy`.

On RHEL/Rocky the admin group is called `wheel` instead of `sudo`.

## Level 2: Package management (added)

On Linux you don't download installers from websites. You ask the **package manager**, which downloads software from trusted **repositories**, installs its dependencies, and later updates everything with one command. Think of it as an app store you drive from the terminal.

| Task | Ubuntu / Debian (apt) | RHEL / Rocky / Fedora (dnf) |
| --- | --- | --- |
| Refresh the list of available packages | `sudo apt update` | `sudo dnf check-update` |
| Upgrade everything installed | `sudo apt upgrade` | `sudo dnf upgrade` |
| Install | `sudo apt install nginx` | `sudo dnf install nginx` |
| Remove | `sudo apt remove nginx` | `sudo dnf remove nginx` |
| Remove + its config files | `sudo apt purge nginx` | (remove, then delete /etc files) |
| Search | `apt search nginx` | `dnf search nginx` |
| Show details | `apt show nginx` | `dnf info nginx` |
| List installed | `apt list --installed` | `dnf list installed` |
| Clean unused deps | `sudo apt autoremove` | `sudo dnf autoremove` |
| Which package owns a file? | `dpkg -S /usr/sbin/nginx` | `rpm -qf /usr/sbin/nginx` |

The classic trap: **`apt update` does not upgrade anything.** It only refreshes the catalogue. Always run `update` before `install` on a fresh server, or you'll get "Unable to locate package".

```
sudo apt update && sudo apt upgrade -y     # the morning routine
sudo apt install -y curl git htop tree     # -y = answer yes (useful in scripts)
```

Repositories are configured in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/` (dnf: `/etc/yum.repos.d/`). Software like Docker is installed by adding the vendor's repository plus its signing key, following the vendor's docs.

Other installers you'll meet: `snap`, Python's `pip`, Node's `npm`, and Alpine's `apk add` inside Dockerfiles.

## Level 2: Processes and signals

A **program** is a file on disk (`/usr/sbin/nginx`). A **process** is that program running in memory. One program can run as many processes: nginx usually runs one master and several workers.

Analogy: a program is a **recipe** in a book; a process is **a cook actually cooking it** right now. Ten cooks can follow the same recipe at once.

### The vocabulary

- **PID** (Process ID): the unique number of a running process. You use it to kill or inspect it.
- **PPID** (Parent PID): the process that started it. Every process has a parent, forming a family tree. See it with `pstree -p`.
- **PID 1**: the first process the kernel starts at boot, usually **systemd**. It is the ancestor of everything and adopts orphans.
- **CPU %**: how much processor time it uses. On a 4-core machine, 400% is the maximum, so one process at 100% is using one full core.
- **Memory**: **RSS** = physical RAM actually used (the number that matters). **VSZ** = virtual memory reserved (often huge; mostly ignore it).
- **Daemon**: a background process with no terminal, running a service. Names often end in **d**: `sshd`, `systemd`, `dockerd`, `crond`. Like a restaurant's dishwasher: always working in the back, nobody talks to it directly.
- **Zombie** (state `Z`): a process that has finished but whose parent hasn't collected its exit status yet. It uses no CPU or RAM, just a slot in the process table. You **can't kill a zombie** (it's already dead); you fix or restart its **parent**. A few are harmless; thousands mean a buggy parent.
- **Orphan**: a running process whose parent died. PID 1 adopts it. Not a problem.

Process states you'll see in the `STAT` column: `R` running, `S` sleeping (waiting, normal), `D` uninterruptible sleep (usually waiting for disk/network; many `D` = storage problem), `T` stopped, `Z` zombie.

### ps: a snapshot

```
ps aux                       # every process, all users
ps aux | grep nginx          # find one (also shows the grep itself)
ps -ef                       # same idea, shows PPID
ps aux --sort=-%cpu | head   # top CPU consumers
ps aux --sort=-%mem | head   # top memory consumers
ps -o pid,ppid,stat,cmd -p 1234   # chosen columns for one PID
```

Reading `ps aux` columns:

```
USER   PID  %CPU %MEM    VSZ   RSS TTY  STAT START  TIME COMMAND
www-data 812  2.1  1.3 55432 26120 ?   S    09:14  0:42 nginx: worker process
```

`?` under TTY = no terminal, so it's a daemon.

### top and htop: live view

`top` is always installed. `htop` (`sudo apt install htop`) is the friendlier, colourful version.

```
top - 10:21:04 up 12 days,  2 users,  load average: 0.52, 0.61, 0.58
Tasks: 213 total,   1 running, 212 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.3 us,  1.2 sy,  0.0 ni, 92.8 id,  0.5 wa, ...
MiB Mem :   3921.4 total,    412.8 free,   1830.2 used,   1678.4 buff/cache
```

**Load average** = the average number of processes running or waiting, over the last 1, 5 and 15 minutes. Compare it to your CPU count (`nproc`): on 4 cores, a load of 4 means fully busy; 8 means work is queuing. The three numbers tell a story: `5.0, 2.0, 0.5` = trouble just started.

**CPU line:** `us` = user programs, `sy` = kernel, `id` = idle, **`wa` = waiting for disk** (high `wa` = slow or overloaded disk, not a CPU problem).

Keys inside top: `P` sort by CPU, `M` sort by memory, `k` kill, `1` show each core, `q` quit. In htop: F6 sort, F9 kill, F5 tree view.

### Signals: how you talk to processes

`kill` doesn't only kill. It **sends a signal**, a message a process can react to.

| Signal | Number | Meaning | Can the process ignore it? |
| --- | --- | --- | --- |
| SIGTERM | 15 | "Please finish up and stop" (the default) | Yes, it can clean up first |
| SIGKILL | 9 | Kernel kills it instantly | No |
| SIGHUP | 1 | "Reload your config" (for many daemons) | Yes |
| SIGINT | 2 | Interrupt; what Ctrl+C sends | Yes |
| SIGSTOP / SIGCONT | 19 / 18 | Pause / resume (Ctrl+Z pauses) | No |

Memory hook: **15 is polite, 9 is a bullet.** Always try 15 first and wait a few seconds; `-9` skips cleanup, so files can be left half-written and locks left behind.

```
kill 1234            # SIGTERM to PID 1234
kill -9 1234         # SIGKILL: last resort
kill -HUP 1234       # ask to reload
pgrep nginx          # print PIDs whose name matches
pgrep -a nginx       # ...with the full command line
pkill nginx          # signal ALL processes matching the name (careful)
pkill -u alex        # all of alex's processes
killall node         # by exact name
```

If a service is managed by systemd, prefer `systemctl stop/restart`, because systemd may simply restart a process you killed.

### Foreground, background and priority (added)

```
long_task &          # start in the background
jobs                 # list background jobs of this shell
Ctrl+Z, then bg      # pause the running command, continue it in the background
fg %1                # bring job 1 back to the foreground
nohup ./run.sh &     # keep running after you log out (better: tmux, or a systemd service)
nice -n 10 ./backup.sh    # start with lower priority (-20 highest ... 19 lowest)
renice 10 -p 1234         # change priority of a running process
```

**tmux** (added, highly recommended): a terminal that survives disconnection. `tmux new -s work`, detach with Ctrl+B then D, reattach later with `tmux attach -t work`. Run long jobs inside it so a dropped SSH connection doesn't kill them.

### Useful inspection tools

```
lsof -p 1234              # files a process has open
lsof -i :80               # which process uses port 80
ls -l /proc/1234/         # everything the kernel knows: cmdline, environ, fd/, status
cat /proc/1234/status     # memory, state, threads
```

## Level 2: Services, systemd and logs

A **service** is a daemon that should always be running (nginx, sshd, docker, your app). **systemd** is PID 1, the manager that starts services at boot, restarts them when they crash, starts them in the right order, and collects their logs.

Analogy: systemd is the **building manager**. Every service has a **contract** (a *unit file*) saying how to start it, what it depends on, and what to do if it fails. The manager follows the contracts and writes everything in a **logbook** (the *journal*).

### systemctl: controlling services

```
sudo systemctl start nginx       # start now
sudo systemctl stop nginx        # stop now
sudo systemctl restart nginx     # stop + start (brief downtime)
sudo systemctl reload nginx      # re-read config without stopping (if supported)
sudo systemctl enable nginx      # start automatically at BOOT
sudo systemctl enable --now nginx   # enable + start in one go
sudo systemctl disable nginx     # don't start at boot
systemctl status nginx           # is it running? why not? last log lines
systemctl is-active nginx        # just "active"/"inactive" (great in scripts)
systemctl list-units --type=service --state=running
systemctl --failed               # every failed unit: check this first on a sick server
sudo systemctl daemon-reload     # after editing ANY unit file
```

Trap: **start ≠ enable**. `start` = now. `enable` = at next boot. A service that's started but not enabled disappears after a reboot.

**Reading `systemctl status`:**

```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Sat 2026-11-21 10:02:11 UTC; 5s ago
    Process: 4120 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
...
nginx[4120]: nginx: [emerg] unknown directive "servr" in /etc/nginx/sites-enabled/default:3
```

- **Loaded** line: where the unit file is, and `enabled` = starts at boot.
- **Active** line: `active (running)` is good; `failed` or `activating (auto-restart)` means trouble.
- Last lines: the most recent logs. Here the answer is right there: a typo on line 3.

`service nginx restart` is the old (SysV) style command. It still works on most systems by forwarding to systemctl. Recognise it, but use `systemctl`.

### Writing your own service (added)

This is how you run your own app properly: it starts at boot, restarts on crash, and logs to the journal. Create `/etc/systemd/system/myapp.service`:

```
[Unit]
Description=My Node.js API
# start after networking is up
After=network.target

[Service]
# never run apps as root
User=myapp
WorkingDirectory=/opt/myapp
EnvironmentFile=/opt/myapp/.env
ExecStart=/usr/bin/node server.js
# auto-restart if it crashes, after 5 seconds
Restart=on-failure
RestartSec=5

[Install]
# "enable" hooks it into normal boot
WantedBy=multi-user.target
```

Note: in unit files, comments must be on their own line starting with `#`.

```
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemctl status myapp
journalctl -u myapp -f
```

Unit files from packages live in `/lib/systemd/system/`. **Yours go in `/etc/systemd/system/`**, which wins over the package ones. To tweak a packaged service safely, use `sudo systemctl edit nginx` (creates an override file).

### journalctl: reading the logbook

```
journalctl -u nginx              # all logs of one service (unit)
journalctl -u nginx -f           # follow live (like tail -f)
journalctl -u nginx -n 50        # last 50 lines
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since "2026-11-21 09:00" --until "2026-11-21 10:00"
journalctl -p err -b             # only errors (-p priority), since this boot (-b)
journalctl -b -1                 # logs from the PREVIOUS boot (why did it crash/reboot?)
journalctl -k                    # kernel messages (like dmesg): OOM kills, disk errors
journalctl -xe                   # jump to the end, with explanations
journalctl --disk-usage          # how much space logs use
sudo journalctl --vacuum-time=7d # delete journal entries older than 7 days
```

Memory hook: **`-u` = which unit, `-f` = follow, `-n` = number, `-p` = priority, `-b` = boot.**

### Classic log files in /var/log

Not everything goes to the journal. Many apps still write text files:

| File | Contains |
| --- | --- |
| `/var/log/syslog` (Ubuntu) or `/var/log/messages` (RHEL) | General system messages |
| `/var/log/auth.log` (Ubuntu) or `/var/log/secure` (RHEL) | Logins, sudo, SSH attempts |
| `/var/log/nginx/access.log`, `error.log` | Every web request / web server errors |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/apt/history.log` | What was installed and when |
| `dmesg` (command) | Kernel ring buffer: hardware, drivers, OOM killer |

Logs are rotated (renamed `.1`, `.2.gz`, then deleted) by **logrotate**, configured in `/etc/logrotate.d/`. That's what stops logs from filling the disk forever.

The log reading workflow: `tail -f` the log in one terminal, reproduce the problem in another, then `grep -i error` around the timestamp.

## Level 2: Disk management

"Disk full" is one of the most common production incidents. A full disk stops databases, breaks log writing, and crashes apps in confusing ways. Learn to diagnose it in under two minutes.

### The four layers (the bookshelf analogy)

| Concept | What it is | Analogy | Example |
| --- | --- | --- | --- |
| **Disk** | The physical or virtual device | An empty room | `/dev/sda`, `/dev/nvme0n1`, `/dev/xvda` (AWS) |
| **Partition** | A slice of the disk | Dividing the room with walls | `/dev/sda1`, `/dev/nvme0n1p1` |
| **Filesystem** | The format that organises files on a partition | The shelving system you install | ext4, xfs (most common), btrfs, vfat |
| **Mount point** | The directory where it appears in the tree | The door label that leads into that room | `/`, `/boot`, `/data` |

The flow is always: disk → partition → format with a filesystem → mount on a directory. Then you can use it.

### Looking at disks

```
lsblk                 # tree of disks, partitions and where they're mounted
lsblk -f              # ...plus filesystem type and UUID
sudo fdisk -l         # detailed partition tables
sudo blkid            # UUIDs and types
```

```
$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   40G  0 disk
├─sda1    8:1    0   39G  0 part /
└─sda15   8:15   0  106M  0 part /boot/efi
sdb       8:16   0  100G  0 disk          ← new disk, not used yet
```

### df vs du: the two questions

- **`df`** = **d**isk **f**ree. "How full is each filesystem?" The bird's-eye view.
- **`du`** = **d**isk **u**sage. "How big is this folder?" The magnifying glass.

```
df -h                 # -h = human readable sizes
df -h /var            # which filesystem holds /var, and how full
df -i                 # INODES: can be 100% even with free space (millions of tiny files)

du -sh /var/log       # -s = summary total for that folder
du -sh /var/* | sort -h            # size of each subfolder, sorted
du -h --max-depth=1 / 2>/dev/null | sort -h   # biggest top-level dirs
sudo ncdu /           # interactive explorer (apt install ncdu): the fastest way
```

**The "disk full" playbook:**

1. `df -h` → which filesystem is at 100%?
2. `sudo du -h --max-depth=1 /that/mount | sort -h` → which folder is big? Repeat one level deeper until you find it.
3. Usual suspects: `/var/log` (huge logs), `/var/lib/docker` (images, containers: `docker system df`, `docker system prune`), old backups, `/tmp`, core dumps.
4. Delete safely, or truncate a live log with `sudo truncate -s 0 /var/log/big.log` (don't `rm` a log a process is writing to).
5. Still full after deleting? A process holds the deleted file open: `sudo lsof +L1` shows them. Restart that process to free the space.
6. Prevent it: logrotate, `journalctl --vacuum-size=500M`, monitoring alerts at 80%.

### Mounting

```
sudo mkfs.ext4 /dev/sdb1          # FORMAT a partition (destroys data on it!)
sudo mkdir /data
sudo mount /dev/sdb1 /data        # attach it to the tree
df -h /data                       # check
sudo umount /data                 # detach (note: umount, not "unmount")
mount | grep sdb                  # what's mounted where
findmnt                           # tree view of all mounts
```

`umount` fails with "target is busy" if anyone is using it, including a shell sitting inside that folder. Find who: `sudo lsof /data`.

Mounts made with `mount` vanish at reboot. To make them permanent, add a line to **`/etc/fstab`**:

```
# device (by UUID)                         mount point  type  options          dump pass
UUID=3f1c...-...-9a2b                      /data        ext4  defaults,nofail  0    2
```

Then test **before rebooting**: `sudo mount -a` (mounts everything in fstab and shows errors). A broken fstab can stop the server from booting; `nofail` makes a missing disk non-fatal.

Worth knowing later: **LVM** (flexible volumes you can grow, very common on servers) and growing cloud disks with `growpart` + `resize2fs` (ext4) or `xfs_growfs` (xfs).

## Level 2: Memory

The most important memory fact for beginners: **Linux uses "free" RAM as disk cache, so low "free" is normal. Look at "available".** Unused RAM is wasted RAM, so Linux fills it with recently read files to speed things up, and gives it back instantly when a program needs it.

Analogy: RAM is your **desk**, disk is the **filing cabinet**. Linux keeps recently used papers on the desk (cache) in case you need them again; when you need space for real work, it clears them off immediately. **Swap** is a box on the floor: extra space, but you have to bend down every time, so it's slow.

### free

```
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.8Gi       0.4Gi        12Mi       1.6Gi       1.8Gi
Swap:          2.0Gi       0.1Gi       1.9Gi
```

| Column | Meaning | Worry when |
| --- | --- | --- |
| used | Used by programs | n/a on its own |
| free | Completely unused | Never alone; low is normal |
| buff/cache | Disk cache, reclaimable | Never, it's a good thing |
| **available** | What a new program can actually get | **Below \~10% of total** |
| Swap used | RAM pages moved to disk | Swap used is growing AND the system is slow |

### vmstat: memory, CPU and disk over time

```
$ vmstat 2 5          # a line every 2 seconds, 5 times
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0  10240 412800  80120 1600000  0    0     5    20  210  400  5  1 93  1  0
```

The columns to watch:

- **r**: processes waiting for CPU. Constantly above your CPU count = CPU bottleneck.
- **b**: processes blocked on I/O (disk). Constantly above 0 = slow disk.
- **si / so** (swap in / out): should be \~0. Constantly non-zero = not enough RAM, the server is "thrashing".
- **wa**: CPU time waiting for disk.

Ignore the first line; it's an average since boot.

### The OOM killer (added)

When RAM and swap are completely full, the kernel's **Out-Of-Memory killer** picks a process (usually the biggest) and kills it with SIGKILL to save the system. Your app "just disappears" with no error in its own log.

```
sudo dmesg -T | grep -i -E "killed process|out of memory"
journalctl -k | grep -i oom
```

If you see your app there, the fix is more RAM, a memory leak fix, or memory limits (e.g. in Docker).

**Finding the memory consumer:**

```
ps aux --sort=-%mem | head -10      # top 10 by memory
top  then press M                   # live, sorted by memory
```

More: `cat /proc/meminfo` (all details), `swapon --show` (swap devices).

## Level 2: Networking

Many "the app is down" incidents are really network problems: wrong DNS, a closed port, an app listening on the wrong address, a firewall. You need a few concepts and a fixed order of checks.

### Concepts in one minute (the postal analogy)

| Concept | Analogy | Example |
| --- | --- | --- |
| **IP address** | A building's street address | `10.0.1.15` (private), `203.0.113.7` (public) |
| **Port** | The apartment number in the building | 22 SSH, 80 HTTP, 443 HTTPS, 5432 PostgreSQL, 3306 MySQL, 6379 Redis |
| **DNS** | The phone book: name → address | `example.com` → `93.184.215.14` |
| **Gateway** | The building's exit to the outside world | Your router / cloud VPC router |
| **TCP** | Registered mail: delivery confirmed, in order | HTTP, SSH, databases |
| **UDP** | A postcard: fast, no guarantee | DNS queries, video, metrics |
| **localhost / 127.0.0.1** | "This same building" | Only reachable from the machine itself |
| **0.0.0.0** (when listening) | "Accept mail at every entrance" | Reachable from outside |

The classic bug: your app listens on `127.0.0.1:3000`, so it works with `curl localhost:3000` on the server but nobody outside can reach it. It must listen on `0.0.0.0:3000` (or sit behind nginx).

### ip: my addresses and routes

```
ip a                  # (ip addr) interfaces and their IP addresses
ip r                  # (ip route) routing table; "default via ..." = the gateway
ip link               # interfaces up/down
hostname -I           # just the IPs
```

`ifconfig` and `route` are the old tools (package net-tools). You'll see them in old tutorials; use `ip`.

### ss: who is listening?

```
sudo ss -tulpn
#        │││││
#        ││││└ n = numeric (don't resolve names)
#        │││└ p = show the process
#        ││└ l = listening sockets only
#        │└ u = UDP
#        └ t = TCP
```

```
Netid State  Local Address:Port  Process
tcp   LISTEN 0.0.0.0:22          users:(("sshd",pid=801))
tcp   LISTEN 127.0.0.1:5432      users:(("postgres",pid=950))   ← local only
tcp   LISTEN 0.0.0.0:80          users:(("nginx",pid=1020))
```

Memory hook: **"ss -tulpn" → "tulip-n"**. `netstat -tulpn` is the old equivalent. Also: `sudo ss -tulpn | grep :80`, `ss -s` (summary), `sudo lsof -i :80`.

### Testing connectivity, layer by layer

```
ping -c 4 8.8.8.8            # can I reach an IP at all? (-c = count; cloud firewalls often block ping)
ping -c 4 google.com         # ...and does DNS work?
traceroute google.com        # every hop on the way: where does it stop?
mtr google.com               # live traceroute + packet loss (apt install mtr)
nc -zv db.internal 5432      # is this TCP port open? (netcat)
```

### curl and wget: talk HTTP

```
curl https://example.com             # print the page
curl -I https://example.com          # headers only: status code, server, redirects
curl -v https://example.com          # verbose: DNS, TLS handshake, headers (great for debugging)
curl -L http://example.com           # follow redirects
curl -o file.zip https://.../file.zip   # save to file
curl -s localhost:3000/health        # silent; app health check
curl -X POST -H "Content-Type: application/json" -d '{"name":"a"}' localhost:3000/api
curl -w "%{http_code} %{time_total}s\n" -o /dev/null -s https://example.com   # just status + timing

wget https://.../file.tar.gz         # download a file (resumes with -c)
```

HTTP status codes to know cold: **200** OK · **301/302** redirect · **400** bad request · **401** not logged in · **403** forbidden (often permissions) · **404** not found · **500** app crashed · **502** Bad Gateway (nginx can't reach your app: is it running?) · **503** unavailable/overloaded · **504** gateway timeout (app too slow).

### DNS

```
dig example.com              # full answer
dig +short example.com       # just the IP
dig example.com MX           # mail records (also: A, AAAA, CNAME, TXT, NS)
dig @8.8.8.8 example.com     # ask a specific DNS server (compare answers)
nslookup example.com         # older, simpler alternative
```

The files involved:

- **`/etc/hosts`**: local overrides, checked BEFORE DNS. `10.0.1.20 db.internal` makes that name work on this machine only. Great for tests, a common source of "works on this server only" bugs.
- **`/etc/resolv.conf`**: which DNS servers this machine asks (on Ubuntu, check with `resolvectl status`).
- **`/etc/hostname`**: this machine's name (`hostnamectl` to change it).

### Firewall (added)

If the app listens on `0.0.0.0` but still isn't reachable from outside, suspect a firewall: on the server, and in the cloud (AWS Security Groups, etc.).

```
sudo ufw status verbose        # Ubuntu's simple firewall
sudo ufw allow 22/tcp          # ALWAYS allow SSH before enabling, or you lock yourself out
sudo ufw allow 80,443/tcp
sudo ufw enable
sudo firewall-cmd --list-all   # RHEL/Rocky equivalent (firewalld)
```

### The network debugging ladder

When "I can't reach the app", climb in this order and stop where it fails:

1. Is the process running? `systemctl status myapp`
2. Is it listening, on the right port and address? `sudo ss -tulpn | grep 3000`
3. Does it answer locally? `curl -v localhost:3000`
4. Does the name resolve? `dig +short app.example.com`
5. Is the port reachable from outside? `nc -zv <server-ip> 3000` from another machine
6. Firewall / security group open? `sudo ufw status`, cloud console
7. Reverse proxy OK? `sudo nginx -t`, `/var/log/nginx/error.log`

## Level 2: SSH, scp and rsync (added)

SSH (**S**ecure **Sh**ell) gives you an encrypted terminal on a remote server, on port 22. It's how you reach every server you'll ever manage, and it's in your Stage 2 exit criterion, so it gets its own section.

### Keys, not passwords

An SSH key comes as a pair:

- **Private key** (`~/.ssh/id_ed25519`): stays on YOUR computer, never shared, permissions 600. It's your physical key.
- **Public key** (`~/.ssh/id_ed25519.pub`): copied to every server you want to access. It's the lock that only your key opens.

Memory hook: **you give out locks (.pub), you keep the key.** The server stores the locks it accepts in `~/.ssh/authorized_keys`.

```
ssh-keygen -t ed25519 -C "alex@laptop"     # 1. create a key pair (set a passphrase)
ssh-copy-id alex@203.0.113.7               # 2. install your public key on the server
ssh alex@203.0.113.7                       # 3. log in, no password needed
```

No `ssh-copy-id` (e.g. on Windows)? Paste the content of your `.pub` file as one line into the server's `~/.ssh/authorized_keys`.

### Connecting

```
ssh user@host                       # default port 22
ssh -p 2222 user@host               # other port
ssh -i ~/.ssh/aws-key.pem ubuntu@host    # specific key (AWS style)
ssh user@host 'df -h; uptime'       # run a command remotely and come back
ssh -v user@host                    # verbose: debug connection problems
ssh -L 8080:localhost:5432 user@host     # tunnel: your localhost:8080 → server's local PostgreSQL
ssh -J bastion user@private-host    # jump through a bastion host
```

**`~/.ssh/config`** saves you typing:

```
Host web01
    HostName 203.0.113.7
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/aws-key.pem
```

Now `ssh web01` is enough, and `scp`/`rsync` understand `web01` too.

### Copying files

```
scp app.tar.gz web01:/tmp/                # local → server
scp web01:/var/log/app.log ./             # server → local
scp -r ./site web01:/var/www/             # a directory

rsync -avz ./site/ web01:/var/www/site/   # sync: copies only what changed
rsync -avz --delete ./site/ web01:/var/www/site/   # ...and delete extras on the server (mirror)
rsync -avzn ...                           # -n = dry run: show what WOULD happen
```

`rsync -avz`: **a**rchive (keep permissions, recursive), **v**erbose, **z** compress. Trap: a trailing `/` on the source means "the contents of", without it you copy the folder itself.

### Securing the SSH server

The server config is `/etc/ssh/sshd_config`. Common hardening:

```
PermitRootLogin no
# keys only: test your key login FIRST
PasswordAuthentication no
```

Then `sudo sshd -t` (test) and `sudo systemctl reload ssh` (Ubuntu; `sshd` on RHEL). **Keep your current session open** while testing in a second terminal, so a mistake can't lock you out.

Failed logins show in `/var/log/auth.log`. Bots try thousands per day; `fail2ban` blocks them.

| SSH error | Usual cause |
| --- | --- |
| `Connection timed out` | Wrong IP, server down, or firewall/security group blocks port 22 |
| `Connection refused` | Server reachable but sshd not running, or wrong port |
| `Permission denied (publickey)` | Wrong user, wrong key, or bad permissions on `~/.ssh` (700) / `authorized_keys` (600) |
| `REMOTE HOST IDENTIFICATION HAS CHANGED` | Server was rebuilt (same IP, new key): `ssh-keygen -R host`. Or a real attack: check first |

## Level 2: Scheduling and archives (added)

Backups, cleanups, certificate renewals and reports run on a schedule. And those backups are usually `.tar.gz` files. These two skills go together.

### cron: the classic scheduler

```
crontab -e          # edit YOUR scheduled jobs
crontab -l          # list them
sudo crontab -e     # root's jobs
```

The five time fields:

```
┌──────── minute       (0-59)
│ ┌────── hour         (0-23)
│ │ ┌──── day of month (1-31)
│ │ │ ┌── month        (1-12)
│ │ │ │ ┌ day of week  (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * *  command
```

| Line | Runs |
| --- | --- |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour, on the hour |
| `30 2 * * *` | Every day at 02:30 |
| `0 9 * * 1-5` | Weekdays at 09:00 |
| `0 3 * * 0` | Sundays at 03:00 |
| `0 0 1 * *` | First day of every month at midnight |
| `@reboot` | Once at every boot |

Memory hook: read it left to right as **"Minute, Hour, Day, Month, Weekday"**, from smallest to biggest. Check expressions at crontab.guru.

A real backup line:

```
30 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

The three classic cron traps:

1. **Cron has an almost empty PATH**, so commands that work in your terminal fail in cron. Use full paths (`/usr/bin/docker`) or set `PATH=...` at the top of the crontab.
2. **Output disappears.** Always redirect to a log: `>> /var/log/job.log 2>&1`.
3. **The server's timezone** decides the time (check with `timedatectl`). Cloud servers are often UTC.

**systemd timers** are the modern alternative: they log to the journal (`journalctl -u backup`), can catch up on missed runs (`Persistent=true`), and are listed with `systemctl list-timers`. You'll meet both at work.

### tar and gzip: archives and compression

`tar` (**t**ape **ar**chive) bundles many files into one. `gzip` compresses. Together: `.tar.gz` (or `.tgz`).

```
tar -czvf backup.tar.gz /etc/nginx      # Create a gZipped archive, Verbose, to File
tar -tzvf backup.tar.gz                 # lisT contents without extracting
tar -xzvf backup.tar.gz                 # eXtract here
tar -xzvf backup.tar.gz -C /tmp/restore # eXtract into a directory
tar -czf app-$(date +%F).tar.gz app/    # dated name: app-2026-11-21.tar.gz
```

Memory hook: **c = create, x = extract, t = table (list), z = gzip, v = verbose, f = file (always last, followed by the name).** "**C**reate e**X**tract **T**able, **Z**ip **V**erbose **F**ile."

Others: `gzip file` / `gunzip file.gz`, `zcat file.gz` and `zgrep error app.log.2.gz` (read rotated logs without extracting), `zip`/`unzip` for Windows people, `-j` instead of `-z` for `.tar.bz2`, `-J` for `.tar.xz`.

## Level 3: Bash scripting

A Bash script is just the commands you'd type, saved in a file so they run the same way every time. If you've done a task by hand twice, script it the third time.

### Your first script, line by line

```
#!/bin/bash
# deploy.sh - pull the latest code and restart containers

echo "Starting deployment"

git pull
docker compose down
docker compose up -d

echo "Deployment completed"
```

- `#!/bin/bash` is the **shebang**: it tells Linux which interpreter runs the file. It must be the very first line. (`#!/usr/bin/env bash` is a more portable variant.)
- Lines starting with `#` are comments.
- Run it: `chmod +x deploy.sh` then `./deploy.sh` (or `bash deploy.sh`). The `./` is needed because the current directory isn't in your `PATH`.

**This roadmap script has a hidden danger:** if `git pull` fails (merge conflict, no network), the script continues, takes the app down, and starts the old code, then prints "Deployment completed". The improved version is at the end of this section.

### Variables

```
name="web01"              # NO spaces around = (name = "x" is an error)
echo "Server: $name"
echo "Server: ${name}_backup"   # braces separate the name from text after it
readonly APP_DIR="/opt/app"    # constant
```

**Quotes matter:**

```
name="world"
echo "Hello $name"    # double quotes: variables expand  → Hello world
echo 'Hello $name'    # single quotes: literal text       → Hello $name
```

Golden rule: **always put variables in double quotes: `"$var"`.** Without quotes, a file name with a space becomes two arguments, and an empty variable vanishes. `rm $file` with `file="my notes.txt"` tries to delete `my` and `notes.txt`.

### Command substitution

Run a command and use its output as a value: `$(command)`.

```
today=$(date +%F)                     # 2026-11-21
backup="db-$today.sql.gz"
files=$(ls /var/log | wc -l)
echo "Running on $(hostname) as $(whoami)"
```

The old form uses backticks: `` `date` ``. Same thing; prefer `$( )` because it nests and reads better.

Arithmetic uses double parentheses: `count=$((count + 1))`, `echo $((7 * 6))`.

### Arguments

```
./deploy.sh production v1.4
```

| Variable | Value | Meaning |
| --- | --- | --- |
| `$0` | ./deploy.sh | The script's name |
| `$1`, `$2` | production, v1.4 | First, second argument |
| `$#` | 2 | Number of arguments |
| `$@` | production v1.4 | All arguments (use as `"$@"`) |
| `$?` | 0 | Exit code of the last command |
| `$$` | 4821 | PID of this script |

```
env=${1:-staging}     # use $1, or "staging" if empty (default value)

if [ $# -lt 1 ]; then
  echo "Usage: $0 <environment>" >&2
  exit 1
fi
```

**Reading input from a user** (added): `read -p "Continue? (y/n) " answer`.

### Exit codes: how commands report success

Every command ends with a number: **0 = success, anything else = failure** (1–255, each command decides what each number means). This is backwards from many languages, so remember: **zero means zero problems.**

```
$ ls /etc > /dev/null ; echo $?
0
$ ls /nope ; echo $?
ls: cannot access '/nope': No such file or directory
2
```

Your scripts should do the same: `exit 0` on success, `exit 1` on error. CI/CD tools (GitHub Actions, Jenkins) decide pass/fail from this number.

### Conditions

```
if [ condition ]; then
  ...
elif [ other ]; then
  ...
else
  ...
fi
```

The spaces inside `[ ]` are required: `[ -f file ]`, never `[-f file]`. In Bash prefer `[[ ]]`, which is safer with empty variables and supports patterns.

| Test | True when |
| --- | --- |
| `-f path` | Regular file exists |
| `-d path` | Directory exists |
| `-e path` | Anything exists |
| `-x path` | Is executable |
| `-z "$s"` / `-n "$s"` | String is empty / not empty |
| `"$a" == "$b"` / `!=` | Strings equal / different |
| `$a -eq $b` | Numbers equal (also -ne, -lt, -le, -gt, -ge) |
| `! test` | Negation |

Memory hook for number tests: **-eq equal, -ne not equal, -lt less than, -gt greater than** (letters, because `<` and `>` mean redirection).

```
if [[ ! -d "$APP_DIR" ]]; then
  echo "App dir missing" >&2
  exit 1
fi

usage=$(df / --output=pcent | tail -1 | tr -dc '0-9')
if (( usage > 80 )); then
  echo "Disk at ${usage}%!"
fi

# You can test any command directly by its exit code:
if systemctl is-active --quiet nginx; then
  echo "nginx is up"
fi
```

**case** (added) for many choices:

```
case "$1" in
  start)   systemctl start myapp ;;
  stop)    systemctl stop myapp ;;
  restart) systemctl restart myapp ;;
  *)       echo "Usage: $0 {start|stop|restart}"; exit 1 ;;
esac
```

### Loops

```
# for: over a list
for server in web01 web02 web03; do
  ssh "$server" uptime
done

# for: over files (never parse ls)
for f in /var/log/*.log; do
  echo "$f: $(wc -l < "$f") lines"
done

# for: numbers
for i in {1..5}; do echo "Attempt $i"; done

# while: until a condition changes (wait for an app to come up)
until curl -sf localhost:3000/health > /dev/null; do
  echo "Waiting for app..."
  sleep 2
done

# while read: process a file line by line
while read -r user; do
  echo "Creating $user"
done < users.txt
```

`break` exits a loop, `continue` skips to the next round.

### Functions

```
log() {
  echo "[$(date '+%F %T')] $*"
}

check_service() {
  local svc="$1"                      # local = only inside this function
  if systemctl is-active --quiet "$svc"; then
    log "$svc OK"
    return 0
  else
    log "$svc DOWN"
    return 1
  fi
}

check_service nginx || systemctl restart nginx
```

Functions get their own `$1`, `$2`; `return` gives an exit code (0–255), not a value. To "return" text, `echo` it and capture with `$(myfunc)`.

### Arrays (added)

```
servers=(web01 web02 db01)
echo "${servers[0]}"        # web01
echo "${#servers[@]}"       # 3 (count)
for s in "${servers[@]}"; do echo "$s"; done
```

### Environment variables

A normal variable lives in the current shell only. An **environment variable** is exported, so programs started from this shell receive it too. That's how apps get config like database URLs.

```
MY_VAR="hello"            # shell only
export DB_HOST="10.0.1.5" # passed to child processes
printenv                  # show all environment variables
echo "$HOME $USER $PATH $PWD $SHELL"
DEBUG=1 ./app.sh          # set for one command only
```

**PATH** is the list of directories searched for commands, separated by `:`. Add your own: `export PATH="$PATH:$HOME/bin"`.

To make settings permanent, put them in **`~/.bashrc`** (loaded for every interactive shell), then run `source ~/.bashrc` to reload now. Aliases go there too: `alias ll='ls -lah'`. System-wide variables: `/etc/environment`.

### Writing scripts that fail safely (added, very important)

Put this at the top of every serious script:

```
set -euo pipefail
```

- `-e`: exit immediately if any command fails.
- `-u`: treat an undefined variable as an error (catches typos like `$APP_DRI`).
- `-o pipefail`: a pipeline fails if any part fails, not just the last.

Debug a script by running `bash -x script.sh`: it prints every command before running it. Check your scripts with **shellcheck** (`sudo apt install shellcheck`), which catches most beginner mistakes.

`trap` runs cleanup when the script exits or fails: `trap 'echo "Failed at line $LINENO" >&2' ERR`.

### The roadmap script, production version

```
#!/usr/bin/env bash
set -euo pipefail

APP_DIR="/opt/myapp"
LOG_FILE="/var/log/myapp-deploy.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
trap 'log "Deployment FAILED at line $LINENO"' ERR

log "Starting deployment on $(hostname)"

cd "$APP_DIR"
git pull --ff-only
docker compose pull
docker compose up -d --remove-orphans

# wait up to 60 s for the health check
for i in {1..30}; do
  if curl -sf http://localhost:3000/health > /dev/null; then
    log "Deployment completed: app is healthy"
    exit 0
  fi
  sleep 2
done

log "App did not become healthy"
docker compose logs --tail=50
exit 1
```

What changed and why: it stops at the first failure, always runs in the right directory, logs with timestamps, avoids taking the app down (`up -d` replaces containers that changed), and only says "completed" after the app actually answers its health check.

## Level 3: Redirection, pipes and text processing

This is where Linux becomes a superpower: small tools joined together answer questions no single tool can.

### The three streams

Every process has three "pipes" attached, each with a number:

| Number | Name | Default | Example |
| --- | --- | --- | --- |
| 0 | stdin (standard input) | Keyboard | What `read` reads |
| 1 | stdout (standard output) | Screen | Normal results |
| 2 | stderr (standard error) | Screen | Error messages |

Analogy: a process is a **kitchen** with one **delivery door** (stdin), one **serving window** (stdout) and one **complaints window** (stderr). Redirection changes where these doors lead.

### Redirection operators

```
command > file        # stdout to file, OVERWRITE
command >> file       # stdout to file, APPEND
command < file        # stdin from file
command 2> err.log    # stderr to file
command 2>&1          # send stderr wherever stdout goes
command > all.log 2>&1   # both to one file (order matters: > first, then 2>&1)
command &> all.log    # bash shortcut for the line above
command > /dev/null 2>&1 # throw away everything (silent)
command | tee out.log    # show on screen AND save to file (tee -a = append)
```

Memory hook: **one arrow `>` wipes, two arrows `>>` add.** The classic disaster is `>` on a file you meant to `>>`. And `2>&1` reads as "**2** goes to where **&1** (stdout) goes"; the `&` means "a stream number, not a file called 1".

**Here-doc** (added): write multi-line text into a file from a script.

```
cat > /etc/myapp/config.env <<EOF
APP_ENV=production
APP_PORT=3000
EOF
```

### Pipes and chaining

```
cmd1 | cmd2         # PIPE: stdout of cmd1 becomes stdin of cmd2
cmd1 && cmd2        # AND: run cmd2 only if cmd1 SUCCEEDED (exit 0)
cmd1 || cmd2        # OR: run cmd2 only if cmd1 FAILED
cmd1 ; cmd2         # run both, no matter what
```

```
ps aux | grep nginx | wc -l
sudo apt update && sudo apt upgrade -y
mkdir -p /data || { echo "cannot create /data"; exit 1; }
systemctl is-active --quiet nginx || sudo systemctl restart nginx
sudo nginx -t && sudo systemctl reload nginx   # reload only if the config is valid
```

Memory hook: **`&&` = "and then, if it worked"; `||` = "or else, if it failed".** A pipe is a conveyor belt between kitchens.

### grep: find lines

**grep** = **g**lobal **r**egular **e**xpression **p**rint. It prints lines that match a pattern.

```
grep "error" app.log              # lines containing error
grep -i "error" app.log           # ignore case (Error, ERROR)
grep -v "DEBUG" app.log           # INVERT: lines NOT matching
grep -n "error" app.log           # with line numbers
grep -c "error" app.log           # count matching lines
grep -r "listen" /etc/nginx/      # recursive through a directory
grep -rl "password" /etc/ 2>/dev/null   # just the file names
grep -A 3 -B 2 "Exception" app.log      # 3 lines After, 2 Before (context)
grep -E "error|warn|fatal" app.log      # Extended regex: OR
grep -w "80" file                 # whole word only (not 8080)
grep -o "[0-9]\{3\}" file         # print only the matching part
```

Regex basics: `^` line start, `$` line end, `.` any character, `*` repeat previous, `[0-9]` a digit. `grep "^#"` = comment lines; `grep -v "^#" nginx.conf | grep -v "^$"` = config without comments and blank lines.

### cut, sort, uniq, wc, tr: shape the data

```
cut -d: -f1 /etc/passwd             # field 1, fields separated by :  → usernames
cut -d, -f2,4 data.csv              # columns 2 and 4 of a CSV
cut -c1-10 file                     # characters 1 to 10

sort file                           # alphabetical
sort -n                             # numeric (so 10 comes after 9)
sort -r                             # reverse
sort -h                             # human sizes (1K < 5M < 2G), pairs with du -h
sort -k2 -n                         # by column 2, numeric
sort -u                             # sort + remove duplicates

uniq                                # remove ADJACENT duplicates: always sort first!
uniq -c                             # count each distinct line

tr 'a-z' 'A-Z'                      # translate characters (uppercase)
tr -d '\r'                          # delete characters (fix Windows line endings)
```

The famous "top N" pattern: **`sort | uniq -c | sort -rn | head`**. Memorise it; you'll use it weekly.

### awk: columns and calculations

awk splits each line into fields: `$1`, `$2`, … (`$0` = whole line, `NF` = number of fields, `NR` = line number). Default separator is whitespace.

```
awk '{print $1}' access.log                     # first column (client IP)
awk '{print $1, $9}' access.log                 # IP and status code
awk -F: '{print $1, $7}' /etc/passwd            # -F sets the separator
awk '$9 == 500' access.log                      # only lines where column 9 is 500
awk '$3 > 50 {print $2}' data.txt               # condition + action
awk '{sum += $10} END {print sum}' access.log   # total bytes sent
awk 'NR==1 || /error/' file                     # header line + error lines
df -h | awk 'NR>1 {print $5, $6}'               # usage % and mount point
```

Memory hook: **awk = "columns"**. If you're thinking "I need the 3rd column", think awk. `cut` is fine for simple single-character separators; awk handles uneven spaces.

### sed: find and replace in streams and files

**sed** = **s**tream **ed**itor.

```
sed 's/old/new/' file              # replace FIRST match per line (prints result)
sed 's/old/new/g' file             # g = global: all matches
sed -i 's/8080/3000/g' config.yml  # -i = edit the file IN PLACE
sed -i.bak 's/8080/3000/g' config.yml   # ...keeping a backup config.yml.bak
sed -n '10,20p' file               # print only lines 10-20
sed '/^#/d' file                   # delete comment lines
sed 's|/var/www|/srv/www|g' file   # any delimiter: handy with paths
```

Memory hook: **`s/find/replace/g`**, "substitute, globally". Always run without `-i` first to preview.

### xargs: turn input into arguments

Some commands don't read stdin; they need arguments. `xargs` builds them.

```
find /tmp -name "*.tmp" | xargs rm               # delete what find found
find . -name "*.log" -print0 | xargs -0 gzip     # safe with spaces in names
cat servers.txt | xargs -I{} ssh {} uptime       # {} = each line
docker ps -q | xargs docker stop                 # stop all containers
xargs -P 4 ...                                   # run 4 in parallel
```

### Put it all together: real DevOps one-liners

```
# Top 10 IP addresses hitting nginx
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# Count of each HTTP status code
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Most requested URLs that returned 404
awk '$9 == 404 {print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# Errors per hour today
grep -i error app.log | awk '{print substr($2,1,2)":00"}' | sort | uniq -c

# Which users can log in (have a real shell)?
grep -v -E "nologin|false" /etc/passwd | cut -d: -f1

# Top 5 processes by memory, just name and %
ps aux --sort=-%mem | awk 'NR>1 && NR<=6 {print $11, $4"%"}'

# Failed SSH login attempts by IP
grep "Failed password" /var/log/auth.log | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" | sort | uniq -c | sort -rn | head

# The 10 biggest files under /var
sudo find /var -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head | awk '{printf "%.1f MB %s\n", $1/1048576, $2}'
```

Build pipelines one step at a time: run the first command, look at the output, add the next `|`, look again. Nobody writes these in one go.

## Troubleshooting playbook: "the application is down"

This is your Stage 2 exit exam turned into a method. Seniors aren't faster because they know magic commands; they check things in a fixed order and don't skip steps. Go from the outside in, and from cheap checks to expensive ones.

### The 10-step method

1. **Define the symptom precisely.** What fails, since when, for everyone or some users? `curl -I https://app.example.com` and note the exact status code or error.
2. **Get on the box and get your bearings.** `ssh web01`, then `uptime` (load, how long it's been up: did it reboot?), `whoami`, `hostname`.
3. **Check the four resources** (most crashes are one of these):
   - Disk: `df -h` and `df -i`
   - Memory: `free -h`, then `dmesg -T | grep -i kill` for OOM kills
   - CPU: `top` (press P), look at load average vs `nproc`
   - I/O wait: `wa` in top, `vmstat 2 5`
4. **Is the service running?** `systemctl status myapp`, `systemctl --failed`. For containers: `docker ps -a` (look for `Exited` or `Restarting`).
5. **Read the logs: the answer is usually here.** `journalctl -u myapp -n 100 --no-pager`, `journalctl -u myapp --since "30 min ago"`, `docker logs --tail 100 <container>`, `tail -n 100 /var/log/nginx/error.log`. Search: `| grep -iE "error|fatal|exception|denied|refused"`.
6. **Is it listening?** `sudo ss -tulpn | grep <port>`. Not listening = app didn't start. Listening on `127.0.0.1` = not reachable from outside.
7. **Does it answer locally?** `curl -v localhost:<port>/health`. Works locally but not from outside → network, firewall, DNS or proxy (go to the network ladder in the Networking section).
8. **Check dependencies.** Database, cache, other APIs: `nc -zv db-host 5432`, `dig +short db-host`. An app often "fails" because something it needs is down.
9. **Check what changed.** Recent deploy (`git log -3`), config edit (`ls -lt /etc/myapp`), package update (`/var/log/apt/history.log`), expired certificate, full disk, rotated password. Most outages follow a change.
10. **Fix, verify, write it down.** Fix the cause (not just restart), confirm with `curl` and the logs, then write 3 lines: what happened, why, how to prevent it.

Memory hook: **"Symptom, Server, Resources, Service, Logs, Listening, Local, Links, Last change, Lesson."**

### Symptom → first command

| Symptom | Most likely | First thing to run |
| --- | --- | --- |
| 502 Bad Gateway from nginx | App behind nginx is down or on a different port | `systemctl status myapp`; `ss -tulpn` |
| 504 Gateway Timeout | App too slow: CPU, DB, or deadlock | `top`; app logs; DB connectivity |
| 403 Forbidden | File permissions or nginx config | `tail /var/log/nginx/error.log`; `namei -l <path>` |
| Service `failed` right after start | Config error, missing env var, port in use | `journalctl -u myapp -n 50` |
| `Address already in use` | Another process holds the port | `sudo ss -tulpn \| grep :3000` |
| App vanishes randomly, no error | OOM killer | `dmesg -T \| grep -i kill` |
| Everything slow, high `wa` | Disk bottleneck | `vmstat 2`; `iostat -x 2` (sysstat) |
| Writes fail, "No space left on device" | Disk or inodes full | `df -h`; `df -i` |
| `Permission denied` | Wrong owner/mode, or wrong user | `ls -l`; `id <service user>` |
| `Connection refused` to DB | DB down or listening on localhost only | `nc -zv host port`; `ss -tulpn` on DB host |
| Works by IP, not by name | DNS or `/etc/hosts` | `dig +short name`; `cat /etc/hosts` |
| Can't SSH in | Firewall, sshd, key permissions | `ssh -v`; cloud console / serial console |

### A practice scenario

You get: "The website shows 502." Here's how it plays out:

```
$ curl -I https://shop.example.com
HTTP/1.1 502 Bad Gateway                    # nginx is up, the app behind it isn't answering

$ systemctl status shop-api
Active: activating (auto-restart) (Result: exit-code)    # it's crash-looping

$ journalctl -u shop-api -n 20 --no-pager
Error: ENOSPC: no space left on device, open '/var/log/shop/app.log'

$ df -h /
/dev/sda1   39G   39G     0 100% /                      # root cause: disk full

$ sudo du -h --max-depth=1 /var | sort -h | tail -3
 31G  /var/log

$ sudo du -sh /var/log/shop/*
 30G  /var/log/shop/debug.log                         # debug logging left on, no rotation

$ sudo truncate -s 0 /var/log/shop/debug.log && sudo systemctl restart shop-api
$ curl -I https://shop.example.com
HTTP/1.1 200 OK
```

Then the lesson: turn off debug logging and add a logrotate rule, plus a disk alert at 80%.

## Practice labs and Stage 2 readiness

You won't remember what you only read. You'll remember what you broke and fixed. Do these labs in order on your practice machine; each one uses the sections above.

### Labs

| # | Lab | Covers |
| --- | --- | --- |
| 1 | Draw the filesystem tree from memory, then check yourself with `ls /` and `tree -L 1 /` | Filesystem |
| 2 | Create `~/lab/{logs,scripts,backup}`, make 20 files with `touch`, copy, rename and delete them; find all files over 1 MB in `/usr` | File commands, find |
| 3 | Create a script without `+x`, hit "Permission denied", fix it. Make a file only you can read (600) and test it as another user | Permissions |
| 4 | Create users `dev1` and `dev2`, a group `devteam`, a shared folder with SGID; give `dev1` sudo for one command only via `visudo` | Users, groups, sudo |
| 5 | Install nginx, then stop, start, enable, break its config on purpose, and diagnose with `systemctl status` and `journalctl` | Packages, systemd, logs |
| 6 | Run `sleep 1000 &`, find it with `pgrep`, inspect with `ps`, kill it with SIGTERM; run `yes > /dev/null &` and find it in `top` | Processes |
| 7 | Fill the disk with `fallocate -l 2G /tmp/big`, find it with `df` and `du`, remove it | Disk |
| 8 | Find which process listens on port 80; `curl -I` your nginx; change it to port 8080 and verify | Networking |
| 9 | Create a cloud VM, set up SSH keys, an `~/.ssh/config` entry, disable password login, `rsync` a folder to it | SSH |
| 10 | Write `backup.sh` that tars `/etc/nginx` with a dated name, keeps the last 7, and logs; schedule it with cron | Scripting, tar, cron |
| 11 | Write `health.sh` that checks disk %, memory, and a list of services, and exits 1 if anything is wrong; run `shellcheck` on it | Scripting, conditions, loops, functions |
| 12 | Run your own small app (any language) as a systemd service with `Restart=on-failure`; kill it with `-9` and watch systemd restart it | Services |
| 13 | Generate traffic to nginx, then answer with one-liners: top IPs, status code counts, top 404 URLs | grep, awk, sort, uniq |
| 14 | Ask a friend (or an AI) to break your server: wrong permissions, full disk, stopped service, wrong port. Fix it using the playbook only | Troubleshooting |

Good free practice: **OverTheWire Bandit** (a game that teaches the shell), **Linux Journey**, and **SadServers** (real broken servers to fix, perfect for lab 14).

### Daily drill (5 minutes)

Pick one command, run `man` on it, learn one new option, and use it in a real task that day. In 3 months that's 60+ options you truly know.

### Stage 2 readiness checklist

Move on when you can do each of these **without looking anything up**:

- [ ] Explain kernel, shell, distro, and what `/etc`, `/var`, `/home`, `/proc` are for
- [ ] Navigate, create, copy, move, delete and find files confidently
- [ ] Edit a config file in vim or nano and save it
- [ ] Read any `ls -l` line and convert permissions between letters and numbers (755, 644, 600)
- [ ] Create users and groups, add a user to a group, and grant sudo safely with `visudo`
- [ ] Install, update and remove packages
- [ ] SSH into a server with keys and copy files with `scp`/`rsync`
- [ ] Find the process using the most CPU and the most memory, and stop it correctly
- [ ] Start, stop, enable and inspect a service; write a basic systemd unit
- [ ] Read service logs with `journalctl` and `/var/log`, filtered by time
- [ ] Diagnose a full disk and find what's filling it
- [ ] Check memory correctly ("available", not "free") and spot an OOM kill
- [ ] Check IP, listening ports, DNS and HTTP response of a service
- [ ] Schedule a job with cron and create/extract a tar.gz backup
- [ ] Write a Bash script with arguments, conditions, loops, functions, exit codes and `set -euo pipefail`
- [ ] Use pipes, redirection, `&&`, `||`, and the text tools to answer a question from a log
- [ ] **Independently troubleshoot a broken application with the 10-step method**

When every box is ticked, you're ready for Stage 2. Tick them in this doc as you go.