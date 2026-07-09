# 🎯 OverTheWire Bandit: Level 28 ➡️ Level 29

## 📌 Level Overview
* Objective: Uncover a password that was previously committed to a Git repository and subsequently deleted/redacted by a developer trying to fix an information leak.
* Current Access: bandit28
* Target Access: bandit29
* Key Lessons Learned: Git Commit History Auditing, Source Code Diff Analysis (`git show`), Forensic Version Control, Git Secrets Recovery.

---

## 🕵️ The Technical Challenge & Reconnaissance

### Step 1: Clone and Inspect
We cloned the `bandit28-git` repository to our local workspace specifying the non-standard port `2220`:
```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
```
Upon inspecting the active working directory, the password file or `README.md` contained placeholder data (e.g., `xxxxxxxxx`), meaning the secret was already obscured in the production branch.
### Step 2: Investigating the Version History
To investigate if the secret existed in previous iterations of the repository, we checked the commit history using:
```Bash
git log
```
The commit history revealed three distinct phases:

1. `initial commit of README.md` (The baseline file initialization).

2. `add missing data` (Likely the insertion point of the sensitive flag).

3. `fix info leak` (The mitigation attempt where the developer overwritten the password with masking characters).
## 🧠 Deep Linux & SCM Concepts Investigated
### 1. Git Immutability & Forensics
Git is designed to be an immutable timeline of code modifications. Deleting or modifying a string in the latest version of a file does not purge it from the underlying snapshot metadata (.git directory). Unless the history is forcefully purged/rewritten using advanced garbage collection commands (like `git filter-branch` or `BFG Repo-Cleaner`), any historical secret remains totally retrievable.
### 2. Code Diffing (`git show`)
The `git show` utility behaves as a precise forensics tool. It displays the commit description alongside a patch/diff output. Lines removed from a file during a commit are highlighted in red (preceded by a `-`), while newly introduced lines appear in green (preceded by a `+`).
## ⚔️ The Exploitation & Injection (Step-by-Step)

### Step 1: Dumping the Flawed Commit
We targeted the recent commit hash where the developer explicitly mentioned fixing the information leak (`83d77407b76c9f86ac4e691a47618641c9d527ba`):
```Bash
git show 83d77407b76c9f86ac4e691a47618641c9d527ba
```
### Step 2: Extracting the Redacted Patch
Analyzing the raw diff payload mapping
```Diff
@@ -4,5 +4,5 @@ Some notes for level29 of bandit.
 ## credentials

- username: bandit29
- password: Em7eGtqaMySwNFjCpwzzHhLhospOcdtO
+ password: xxxxxxxxx
```
The red line exposes the exact string sequence deleted during that operation, presenting the target credentials dynamically.
## 🏁 Extraction & Post-Exploitation
By leveraging code differential auditing against the commit history metadata, we successfully reconstructed the redacted environment variable.
- Target Plaintext Password: Em7eGtqaMySwNFjCpwzzHhLhospOcdtO

