# 🎯 OverTheWire Bandit: Level 25 ➡️ Level 26

## 📌 Level Overview
* Objective: Break out of a restricted custom shell environment (Restricted Shell Bypass) to gain interactive CLI access as the target user.
* Current Access: bandit25
* Target Access: bandit26
* Key Lessons Learned: Restricted Shell Bypassing, Interactive Pager Escaping (`more`/`vi`), Living off the Land (GTFOBins), Environment Manipulation (Window Sizing).

---

## 🕵️ The Technical Challenge & Reconnaissance

### Step 1: Discovering the Custom Shell
Before attempting to SSH into `bandit26`, we must identify how the system restricts the user. We parse the `/etc/passwd` file to check the default login shell for the target user:
```bash
cat /etc/passwd | grep "bandit26"
```
Output snippet: `bandit26:x:11026:11026:...:/home/bandit26:/usr/bin/showtext`
Instead of `/bin/bash`, the user is forced into a custom script called `showtext`.
### Step 2: Reverse Engineering the Restricted Shell
We read the contents of the custom shell to understand its logic:
```Bash
cat /usr/bin/showtext
```
Core Logic:
```Bash
#!/bin/sh
export TERM=linux
exec more ~/text.txt
exit 0
```
The script uses `exec` to replace the shell process with the more pager, displaying a banner text file. Once `more` finishes printing the file, the script immediately executes `exit 0`, terminating the SSH connection and effectively trapping the user.
## 🧠 Deep Linux Concepts Investigated
1. Restricted Shell Escapes (GTFOBins)
System administrators often use restricted shells to limit user capabilities. However, if a user can trigger an interactive binary (like `more`, `less`, `vi`, or `man`), they can often utilize built-in escape sequences to spawn a standard system shell, completely bypassing the intended restrictions.
2. The Pager Pause Technique
The `more` command is designed to pause pagination and wait for user interaction only if the text exceeds the terminal's vertical viewing area. If the terminal is large enough to display the entire file at once, `more` exits automatically without yielding control to the user.
## ⚔️ The Exploitation & Injection (Step-by-Step)
### Step 1: Environment Manipulation (Window Shrinking)
To prevent `more` from auto-exiting, we deliberately manipulate our local terminal environment. By shrinking the terminal window's vertical height to just 3-4 lines, we force `more` to pause outputting `~/text.txt` and drop us into its interactive prompt: `--More--(20%)`.
### Step 2: Initiating the SSH Connection
While the terminal is minimized, we initiate the login sequence:

```Bash
ssh -i bandit26.sshkey bandit26@localhost -p 2220
```
### Step 3: Triggering the Internal Text Editor (`vi`)
Once the connection is established and paused at the `--More--` prompt, we press the `v` key. In `more`, this keybinding invokes the `vi` text editor, elevating us from a simple pager to a powerful, highly interactive editor under the context of `bandit26`.
### Step 4: Spawning the Unrestricted Shell
Now inside `vi`'s command mode, we redefine the editor's default shell variable to point to the standard bash binary:

```Vim Script
:set shell=/bin/bash
```
Next, we execute a shell command to spawn the new unrestricted shell directly over the editor interface:
```Vim Script
:sh
```
### 🏁 Extraction & Post-Exploitation
The terminal drops us into a fully interactive bash prompt: `bandit26@bandit:~$`. The restricted jail is successfully bypassed.
We retrieve the flag:

```Bash
cat /etc/bandit_pass/bandit26
```
- Result: The plaintext flag for Level 26 is cleanly retrieved.

- Note: This password changes from one user instance to another.
