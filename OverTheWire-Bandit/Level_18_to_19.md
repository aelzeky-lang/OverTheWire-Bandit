OverTheWire Bandit: Level 18 ➡️ Level 19
📝 Level Overview
- Objective: Retrieve the password for the next level from a file named readme located in the home directory, while bypassing a system configuration that forcefully logs users out upon login.

- Current Access: bandit18

- Target Access: bandit19

- Key Lessons Learned: SSH Internals, Bypassing Restricted Shells, Non-interactive Sessions, Remote Command Execution (RCE).
### The Technical Challenge & Reconnaissance
This level shifts focus from file manipulation to understanding system and protocol internals. We have the correct password for bandit18, but the system administrator has modified the environment to prevent standard terminal access.
### Step 1: The Initial Login Attempt
Attempting to log in normally using the acquired password:

```Bash
ssh bandit18@bandit.labs.overthewire.org -p 2220
```
- Result: The authentication succeeds, but the server immediately prints Byebye! and closes the connection. No interactive shell (like /bin/bash) is provided.
### Step 2: Bypassing the Shell Initialization
Since the standard interactive login process triggers the trap, we must instruct the SSH client to bypass the default shell allocation and execute our command directly on the remote server.

We append the command we want to run (cat readme) directly to the SSH connection string:
```Bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```
- Result: The SSH client connects, authenticates, executes the cat readme command directly, prints the password for Level 19 to our local terminal, and exits cleanly without ever triggering the Byebye! trap.
## 🚧 Pitfalls, Traps, and Solutions (The Real Learning)
### ❌ Trap 1: The "Byebye!" Forced Logout (Modified Shells)
- The Mechanism: In Linux, when a user logs in interactively, the system allocates a pseudo-terminal (PTY) and runs profile scripts (like `.bashrc` or `.bash_profile`), or it launches the default shell defined in `/etc/passwd`. The administrator for this level configured the shell or profile to immediately execute an exit command or run a script that prints "Byebye!" and terminates the session.
- The Solution: SSH allows for Non-interactive Sessions. By passing a command directly via the SSH syntax, we tell the daemon: "Do not allocate an interactive terminal and do not load the user's interactive profile. Just execute this specific command and return the output." This bypasses the trap entirely.
### 🧠 Key Concepts Learned & Real-World Applications
1. Bypassing Restricted Shells (Penetration Testing)
In real-world hacking and penetration testing, administrators often place users in Restricted Shells (like `rbash` or `lshell`) to limit their capabilities (e.g., preventing directory traversal or blocking certain commands). Executing commands remotely via SSH before the restricted shell environment is fully initialized is a classic bypass technique to gain proper Remote Command Execution (RCE) and escalate privileges.
2. Infrastructure Automation & Scripting (SysAdmin)
System and Network Administrators rarely log into servers interactively to perform routine checks. Understanding non-interactive SSH is crucial for automation. For example, checking the disk space of 100 servers can be automated using a simple loop:
```Bash
for server in server1 server2 server3; do
    ssh user@$server "df -h"
done
```
This executes the command across the fleet instantly without manual logins.
3. Secure Data Piping over SSH
SSH is not just a terminal emulator; it is a secure data transport protocol. You can pipe data from your local machine directly into a remote server's filesystem seamlessly.
For instance, deploying a compressed backup without dropping it on the remote disk first:
```Bash
cat local_backup.tar.gz | ssh admin@remote_server "tar xz -C /var/www/html/"
```
