# Day 2 Report — Linux User Management, Permissions & Process Management

**Task:** DEVOPS-002
**Date:** [Insert Date]
**Prepared by:** [Insert Name]

---

## 1. Topics Learned

On Day 2, the focus was on three core areas of Linux system administration:

1. **User Management** — creating, modifying, and deleting user accounts and groups, and understanding how Linux tracks user identity through `/etc/passwd`, `/etc/shadow`, and `/etc/group`.
2. **File Permissions** — understanding the Linux permission model (read, write, execute) for the owner, group, and others, and how to view and change permissions and ownership.
3. **Process Management** — understanding how Linux runs and manages processes, how to view running processes, and how to control or terminate them.
4. **Package Management** — installing, updating, and removing software packages using the system's package manager.

Together, these topics form the foundation of day-to-day Linux server administration, which is essential for any DevOps role — managing who can access a system, what they can do on it, what is currently running, and what software is installed.

---

## 2. Commands Used

### User & Group Management
| Command | Purpose |
|---|---|
| `whoami` | Displays the current logged-in user |
| `id` | Shows user ID (UID), group ID (GID), and group memberships |
| `useradd <username>` | Creates a new user account |
| `passwd <username>` | Sets or changes a user's password |
| `usermod -aG <group> <username>` | Adds a user to an additional group |
| `userdel <username>` | Deletes a user account |
| `groupadd <groupname>` | Creates a new group |
| `groupdel <groupname>` | Deletes a group |
| `cat /etc/passwd` | Lists all user accounts on the system |
| `cat /etc/group` | Lists all groups on the system |

### Permissions & Ownership
| Command | Purpose |
|---|---|
| `ls -l` | Lists files with detailed permission, owner, and group info |
| `chmod` | Changes file/directory permissions (e.g., `chmod 755 file.sh`) |
| `chown <user>:<group> <file>` | Changes the owner and group of a file |
| `chmod u+x <file>` | Adds execute permission for the file owner |

### Process Management
| Command | Purpose |
|---|---|
| `ps` | Shows currently running processes for the current shell |
| `ps aux` | Shows all running processes on the system in detail |
| `top` | Displays a live, updating view of system processes and resource usage |
| `kill <PID>` | Terminates a process using its Process ID |
| `kill -9 <PID>` | Forcefully terminates a process |
| `bg` / `fg` | Moves a process to the background or foreground |
| `jobs` | Lists background jobs in the current shell |

### Package Management
| Command | Purpose |
|---|---|
| `sudo apt update` | Refreshes the local package index (Debian/Ubuntu) |
| `sudo apt install <package>` | Installs a new package |
| `sudo apt remove <package>` | Removes an installed package |
| `sudo apt upgrade` | Upgrades installed packages to their latest versions |
| `dpkg -l` | Lists installed packages |

---

## 3. Problems Faced

*[Fill this section in with the specific issues you personally ran into. Common examples are listed below as a starting point — replace or expand with what actually happened during your session.]*

- Permission denied errors when running `useradd` or `chmod` without `sudo`, resolved by prefixing commands with `sudo`.
- Confusion initially between numeric (`chmod 755`) and symbolic (`chmod u+x`) permission notation.
- A process that did not terminate with a normal `kill` command and required `kill -9` to force-stop it.
- Package installation failing until `sudo apt update` was run first to refresh the package index.

---

## 4. Screenshots

*[Insert screenshots here for each command run during the session, e.g.:]*

- Screenshot of `id` and `cat /etc/passwd` output
- Screenshot of `useradd` and `passwd` in action
- Screenshot of `ls -l` showing file permissions before and after `chmod`
- Screenshot of `ps aux` and `top` output
- Screenshot of a process being killed
- Screenshot of a package being installed via `apt install`

---

## 5. Permission Table

Linux file permissions are divided into three categories (Owner, Group, Others), each with three possible permissions (Read, Write, Execute).

| Symbol | Numeric Value | Meaning |
|---|---|---|
| `r` | 4 | Read — view file contents / list directory contents |
| `w` | 2 | Write — modify file contents / add-remove files in directory |
| `x` | 1 | Execute — run the file as a program / enter the directory |
| `-` | 0 | No permission |

**Example breakdown of `chmod 754 file.sh`:**

| Category | Numeric | Symbolic | Meaning |
|---|---|---|---|
| Owner | 7 | `rwx` | Read, write, and execute |
| Group | 5 | `r-x` | Read and execute only |
| Others | 4 | `r--` | Read only |

**Example `ls -l` output interpretation:**

```
-rwxr-xr--  1 devuser devgroup  1024 Aug 08 10:00 script.sh
```

| Field | Value | Meaning |
|---|---|---|
| File type | `-` | Regular file (`d` would mean directory) |
| Owner permissions | `rwx` | Owner can read, write, execute |
| Group permissions | `r-x` | Group can read, execute |
| Others permissions | `r--` | Others can only read |
| Owner | `devuser` | User who owns the file |
| Group | `devgroup` | Group that owns the file |

---

## 6. User and Group Summary

*[Fill in with the actual users/groups you created. Example table below.]*

| User Created | Groups Assigned | Purpose |
|---|---|---|
| `devuser1` | `developers` | Example application/deployment user |
| `devuser2` | `developers`, `sudo` | Example admin-level user |

**Key concepts covered:**
- Every user has a unique UID and belongs to a primary group, plus optional secondary groups.
- `/etc/passwd` stores user account information (username, UID, GID, home directory, shell).
- `/etc/shadow` stores encrypted password data and is readable only by root.
- `/etc/group` stores group definitions and their members.

---

## 7. Process Management Summary

- Every running program on Linux is a **process**, identified by a unique **Process ID (PID)**.
- `ps aux` and `top` are the primary tools for viewing currently running processes, their resource usage (CPU, memory), and their state.
- Processes can run in the **foreground** (tying up the terminal) or **background** (freeing the terminal for other tasks), controlled with `&`, `bg`, `fg`, and `jobs`.
- The `kill` command sends signals to processes — a normal `kill` requests graceful termination, while `kill -9` forces immediate termination.
- Understanding process management is critical for diagnosing performance issues and stopping unresponsive or unwanted programs on a server.

---

## 8. Package Management Summary

- Package managers (like `apt` on Debian/Ubuntu or `yum`/`dnf` on RHEL/CentOS) handle installing, updating, and removing software along with their dependencies.
- `apt update` refreshes the local index of available packages and their versions — it does not install or upgrade anything by itself.
- `apt install <package>` installs new software; `apt remove <package>` uninstalls it.
- `apt upgrade` updates all installed packages to their latest available versions.
- Package management ensures software on a Linux server stays consistent, up to date, and free of manual dependency conflicts.

---

## Conclusion

Day 2 covered the essential administrative skills needed to manage users, control access through permissions, monitor and manage running processes, and maintain installed software on a Linux system. These fundamentals directly support common DevOps responsibilities such as provisioning servers, securing file access, and maintaining system health.
