# 🎯 OverTheWire Bandit: Level 26 ➡️ Level 27

## 📌 Level Overview
* Objective: Escalate privileges to the next user by exploiting a custom SUID (Set Owner User ID) executable.
* Current Access: bandit26
* Target Access: bandit27
* Key Lessons Learned: Privilege Escalation, SUID/SGID Permissions, Execution Wrappers, Linux File Permissions.

---

## 🕵️ The Technical Challenge & Reconnaissance

### Step 1: Local Enumeration
Upon spawning an interactive shell as `bandit26`, the first step in any post-exploitation phase is local enumeration. We list the contents of the current home directory, including hidden files and detailed permissions:
```bash
ls -al
```
- Output observation:
Among standard hidden files, an executable binary named `bandit27-do` stands out.
### Step 2: Permission Analysis
Analyzing the file permissions of `bandit27-do`:
`-rwsr-x--- 1 bandit27 bandit26 7360 May  7  2020 bandit27-do`
- The owner of the file is `bandit27`.
- The group owner is `bandit26` (which allows us to execute it).
- Crucially, the permission string contains an `s` instead of an `x` in the owner's execution slot (`rws`). This indicates the SUID bit is set.
## 🧠 Deep Linux Concepts Investigated
### The SUID (SetUID) Bit
SUID is a special type of file permission given to a file. Normally in Linux, when a user executes a program, the program runs with the permissions of the user who started it. However, if an executable has the SUID bit set, it will run with the privileges of the file's owner, regardless of who executes it.
In this scenario, executing `bandit27-do` temporarily grants us the privileges of user `bandit27`.
### Execution Wrappers (run-as)
The name `bandit27-do` implies it functions similarly to `sudo` or `su`, acting as a wrapper to execute subsequent arguments as another user.
## ⚔️ The Exploitation & Injection (Step-by-Step)
### Step 1: Verifying the Execution Context
Before attacking the password file, it is best practice to verify our effective user ID when running the binary:
```Bash
./bandit27-do whoami
```
- Output: `bandit27`
This confirms that any command appended as an argument to this binary will be executed under the context of our target user.
### Step 2: The Privilege Escalation Attack
Since the binary blindly executes commands passed to it as `bandit27`, we can instruct it to read the protected password file that only `bandit27` has access to:
```Bash
./bandit27-do cat /etc/bandit_pass/bandit27
```
## 🏁 Extraction & Post-Exploitation
By leveraging the SUID binary, the system executes `cat /etc/bandit_pass/bandit27` as the target user.
- Output:

```Bash
YnQpBuifNMas1hcUFk70Zmqfc1030MOU
```
- Result: The plaintext flag for Level 27 is cleanly retrieved.

- Note: This password changes from one user instance to another.

