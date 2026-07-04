# 🔄 OverTheWire Bandit: Level 13 ➡️ Level 14

## 📝 Level Overview
* **Objective:** Log into the `bandit14` account using the private SSH key stored on the server, then locate and secure the next level's password.
* **Current Access:** `bandit13`
* **Target Access:** `bandit14`
* **Key Tools & Techniques:** Key-Based SSH Authentication, File Permissions (`chmod`), Data Exfiltration/Pivoting, Troubleshooting Cryptographic Errors.

---

## 🔍 The Constraint & Technical Problem
Initially, the challenge hints at using the private SSH key (`sshkey.private`) located in the home directory to establish a local connection directly on the server to `localhost`. 
However, running the standard local command:
```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```
Reveals an updated server-side restriction:

!!! Connecting from localhost is blocked to conserve resources.
Received disconnect from 127.0.0.1 port 2220:2: no authentication methods enabled

This requires a strategic workaround: Data Exfiltration. Instead of connecting from within the compromised host, the private key must be safely extracted to our local machine to establish an authorized external session.
🛠️ Step-by-Step Methodology
### Step 1: Inspecting & Extracting the Private Key
While logged into bandit13, audit the private key content. This is an OpenSSH private cryptographic key.

```Bash
cat sshkey.private
```
Crucial Note: Ensure the entire block is copied perfectly, including the structural headers:

```Plaintext
-----BEGIN OPENSSH PRIVATE KEY-----
[Cryptographic Payload Data]
-----END OPENSSH PRIVATE KEY-----
```
### Step 2: Creating the Identity File Locally
On the local machine (outside the OverTheWire server), initialize a new file to securely store the extracted key.

```Bash
nano bandit14_key
```
Paste the exact contents copied from the server.

Save and exit the text editor (Ctrl+O -> Enter -> Ctrl+X).

### Step 3: Hardening File Permissions (The Crypto Baseline)
SSH clients strictly enforce safety mechanisms that reject insecure private keys. If a private key is accessible by anyone other than its owner, the authentication process will fail.
```Bash
chmod 600 bandit14_key
```
Why 600? It translates to -rw-------, meaning only the owner has Read & Write access, while group and public permissions are completely stripped.
Step 4: Establishing Remote SSH Session via Identity File
With the hardened private key ready, invoke the external SSH connection specifying the identity file path:

```Bash
ssh -i bandit14_key bandit14@bandit.labs.overthewire.org -p 2220
```
This successfully authenticates the session and bypasses the internal localhost block entirely.

Once inside as bandit14, the password can be retrieved from its designated repository folder.
### 🧠 Key Concepts Learned
1. Key-Based Authentication vs. Password Authentication:

Passwords are vulnerable to brute-force and dictionary attacks. Key-based authentication relies on public-key cryptography (Asymmetric encryption) where a Private Key validates against a corresponding Public Key stored on the server.

2. Strict Cryptographic Security Policies (error in libcrypto):

If a key is poorly formatted or lacks protective permissions, the library throws parsing failures. SSH forces security practices by refusing execution until file permissions are hardened.

3. Pivoting & Network Constraints Workarounds:

Modern security infrastructure frequently blocks internal routing loops (like localhost loops) to conserve resources or prevent pivoting. Extracting credentials safely allows an engineer to shift perspective and find alternative intrusion vectors.
