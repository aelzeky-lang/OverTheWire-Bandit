# 🎯 OverTheWire Bandit: Level 29 ➡️ Level 30

## 📌 Level Overview
* Objective: Extract a hidden password stored inside a non-default remote Git branch that is separate from the primary execution branch.
* Current Access: bandit29
* Target Access: bandit30
* Key Lessons Learned: Git Branch Enumeration (`git branch -a`), Detached HEAD States, Multi-branch Auditing, Dev vs. Production Secret Analysis.

---

## 🕵️ The Technical Challenge & Reconnaissance

### Step 1: Clone and Local Setup
We initialized by cloning the target repository for `bandit29-git` via the standard port configuration (`2220`):
```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
```
Upon inspecting the default `master` branch, no explicit production credentials or leaks were identified within the active tracking logs.
### Step 2: Investigating Isolated Branches
To audit whether alternative tracking workspaces existed on the remote server, we enumerated all operational branches:
```bash
git branch -a
```
The metadata layout revealed hidden upstream tracking references:

- `remotes/origin/dev` (Development testing branch)

- `remotes/origin/sploits-dev` (Exploit testing framework branch)
## 🧠 Deep Linux & SCM Concepts Investigated
### 1. Remote vs Local Tracking Branches
When cloning a repository, Git defaults to downloading the history but only initializes a local working branch for `master`/`main`. Production secrets are frequently left behind in development branches (`dev`, `staging`, `feature/test`) that are pushed to the remote server but never merged into the deployment branch.
### 2. Detached HEAD State
When running a checkout directly against a remote branch descriptor (e.g., `remotes/origin/dev`), Git enters a Detached HEAD state. This means the file tracking mechanism points directly to a specific commit snapshot instead of a local branch reference, allowing a temporary read-only view of historical artifacts.
## ⚔️ The Exploitation & Injection (Step-by-Step)
### Step 1: Shifting Working Workspaces
We jumped environments by checking out the remote development instance dynamically:
```bash
git checkout remotes/origin/dev
```
### Step 2: Extracting the Target Flag
Once the workspace synchronized with the development profile, we listed the active root variables and dumped the target configuration markdown file:
```Bash
ls -la
cat README.md
```
The output exposed the configuration descriptors for the subsequent challenge tier:
```Plaintext
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: jq9Dfg2rXsfYsWMgFuKlXhphjdH7USgX
```
##🏁 Extraction & Post-Exploitation
By auditing hidden branch indicators rather than relying solely on the primary timeline, we bypassed the initial redacting strategy.

Target Plaintext Password: `jq9Dfg2rXsfYsWMgFuKlXhphjdH7USgX`
