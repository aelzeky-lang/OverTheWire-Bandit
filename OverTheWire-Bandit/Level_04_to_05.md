# Bandit Level 4 ➔ Level 5 Writeup

## 🎯 Objective
Find the password hidden in the only human-readable file inside the `inhere` directory.

## 🧩 The Challenge
The directory contains multiple files named from `-file00` to `-file09`. Most of them contain binary data which garbles the terminal screen if viewed with `cat`. 

---

## 🛠️ Solutions

### Method 1: The Manual Approach (Brute-force)
Checking files one by one using `cat ./-fileXX` until hitting the right one (`-file07`) which contains plaintext. If a binary file corrupts the terminal viewport, the `reset` command restores it.

### Method 2: The Smart Way (Using the `file` command)
Instead of guessing, we can use the `file` utility combined with a wildcard `*` to check the types of all files instantly:
```bash
file ./*
```
Output Analysis:
The command will reveal that all files are of type data except for ./-file07, which is flagged as ASCII text.

Knowing this, we print its content directly:

```Bash
cat ./-file07
```
🧠 Concepts Learned
The file Command: A powerful utility to determine file types without opening them.

Terminal Corruption & reset: Understanding why binary files mess up the CLI display and how to fix it.

ASCII vs Data Files: Differentiating between human-readable text and non-printable binary streams.
