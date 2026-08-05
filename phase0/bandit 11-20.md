# Level 11 → 12

## Concept
Decoding text encoded with **ROT13**.

## Commands

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

or

```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt
```

## What I Learned
- **ROT13** is a simple substitution cipher that rotates each letter by 13 places.
- The `tr` command translates one set of characters into another.
- Running ROT13 twice returns the original text.

## Security Principle: Kerckhoffs's Principle
A secure system should remain secure even if **everything about it is public except the secret key**.

ROT13 is **not secure encryption** because:
- The algorithm is public.
- There is no secret key.
- Anyone who knows ROT13 can instantly reverse it.

**Security should depend on the secrecy of the key, not the secrecy of the algorithm.**

---

# Level 12 → 13

## Concept
Working with files that have been compressed multiple times.

## Commands

```bash
mktemp -d
cd <temporary_directory>
cp ~/data.txt .
xxd -r data.txt > data
file data.bin
```

Continue checking the file type after each extraction:

```bash
file <filename>
```

Possible extraction commands:

```bash
gunzip
bunzip2
unxz
tar -xf
mv
```

## Methodology (Compression Onion)

The most important lesson from this level is the **process**, not the commands.

```
Hexdump
     │
     ▼
Convert back to binary (xxd -r)
     │
     ▼
Use file to identify the format
     │
     ▼
Extract using the correct tool
     │
     ▼
Run file again
     │
     ▼
Repeat until you reach plain text
```
## The collision I hit (and how I solved it)
bzip2 -d refused to run because a file named "data" already existed 
from an earlier tar layer. Fix: decompress to a fresh name instead:
    bzip2 -dc data.bz2 > layer7
Lesson: keep a clean workspace; leftover files cause name collisions.

### What I Learned
- Never guess a file's format.
- Always use `file` to identify what comes next.
- Different compression formats require different extraction tools.
- Complex problems become manageable when solved one layer at a time.

---

# Level 13 → 14

## Concept
Authenticating with an SSH private key.

## Commands

```bash
ls -la
```

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```

```bash
cat /etc/bandit_pass/bandit14
```

## The catch: localhost login is blocked
Connecting bandit13 → bandit14 via localhost is disabled. 
Real solution: copy the key to my own machine, then connect:
```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private ~/key 
```
```bash
chmod 600 ~/key                     # SSH refuses loose key permissions
```
```bash
ssh -i ~/key bandit14@bandit.labs.overthewire.org -p 2220
```
Gotcha: scp uses -P (capital), ssh uses -p (lowercase).

## What I Learned
- SSH supports authentication using cryptographic keys instead of passwords.
- The private key proves your identity without sending the key itself over the network.
- Only someone possessing the correct private key can authenticate successfully.

## SSH Key Authentication

```
Private Key (Your Machine)
        │
Signs a challenge
        │
        ▼
SSH Server
        │
Verifies signature using your public key
        │
        ▼
Access Granted
```

### Key Principle
**The private key never leaves your computer.**

Only a cryptographic signature is transmitted, making SSH key authentication much more secure than password-based authentication.

---

# Level 14 → 15

## Concept
Communicating with a network service using Netcat.

## Commands

```bash
nc localhost 30000
```

Then paste the Bandit 14 password and press **Enter**.

## What I Learned
- `nc` (Netcat) is a tool for opening raw TCP or UDP network connections.
- Many network services simply listen on a port and wait for incoming data.
- After sending the correct password, the service responds with the next password.

## Why I Had to Become `bandit14`

The service on port **30000** only accepts the **Bandit 14 password**.

```
bandit13
      │
No valid password
      │
Access denied
      │
Become bandit14
      │
Have correct credentials
      │
Connect to port 30000
      │
Receive Bandit 15 password
```

This demonstrates **access control**: possessing the correct credentials determines whether access is granted.

---

# Level 15 → 16

## Concept
Establishing an encrypted connection using TLS.

## Commands

```bash
openssl s_client -connect localhost:30001
```

After the TLS connection is established, paste the Bandit 15 password.

## What I Learned
- Some services require encrypted communication instead of plain TCP.
- `openssl s_client` creates a secure TLS connection.
- After the secure channel is established, data is encrypted while in transit.

## Plain TCP vs TLS

### Netcat (Plain Text)

```
You
 │
 ▼
Password ─────────────► Server
```

Anyone monitoring the network could read the password.

### TLS (Encrypted)

```
You
 │
 ▼
Encrypted Data ═══════► Server
```

Anyone intercepting the traffic sees only encrypted data.

### Key Takeaways
- **Netcat (`nc`)** is useful for plain TCP communication.
- **OpenSSL (`openssl s_client`)** is used when the service requires TLS encryption.
- Encryption protects data while it travels across the network but does not replace authentication.

  # Level 16 → 17 — Finding the Correct TLS Service

## Concept
Scanning a range of ports to discover which services are available and identifying the one that uses SSL/TLS.

## What Happened

The challenge required finding a service listening on one of several ports.

Instead of guessing, I scanned the port range and identified which ports were open and which one was running SSL/TLS.

After finding the correct service, I connected securely using OpenSSL.

## Commands

Scan the port range:

```bash
nmap -sV -p 31000-32000 localhost
```

Connect to the SSL/TLS service:

```bash
openssl s_client -connect localhost:<PORT>
```

Send the current password:

```bash
<password>
```

## What I Learned

- `nmap` discovers open ports and identifies running services.
- `-sV` performs service version detection.
- `openssl s_client` is used to communicate with encrypted services.
- Not every open port speaks the same protocol.

## Security Lesson

Always identify the service before interacting with it. Port numbers are conventions—not guarantees. Service detection prevents using the wrong protocol and helps accurately understand a system's exposed services.

---

# Level 17 → 18 — Comparing Files with diff

## Concept
Finding differences between two files.

## What Happened

Two files contained almost identical data.

The password was the **only line that differed** between them.

Instead of manually comparing thousands of lines, I used `diff` to identify the single change.

## Command

```bash
diff passwords.old passwords.new
```

## Example Output

```text
42c42
< old_password
---
> new_password
```

The line beginning with `>` is the new password.

## What I Learned

- `diff` compares two files line by line.
- It highlights additions, deletions, and modifications.
- It is an essential tool for comparing configuration files, source code, backups, and logs.

## Understanding the Output

```
<  Line from the first file
>  Line from the second file
```

The symbols indicate which file each line came from.

## Security Lesson

Comparing versions of files is a common forensic technique. Attackers, administrators, and security analysts often use `diff` to identify configuration changes, unauthorized modifications, or newly introduced data without reading entire files manually.

# Level 18 → 19 — Escaping a Restricted Shell

## Concept
Bypassing a restricted shell to execute a command before the shell logs you out.

## What Happened
When logging into `bandit18`, the shell immediately logged the user out because the shell startup files were modified to prevent interactive access.

Instead of requesting an interactive shell, SSH was used to execute a command directly on the remote machine.

## Command

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

## What I Learned

- SSH can execute a command directly on the remote host without starting an interactive shell.
- Shell startup scripts (such as `.bashrc`) only affect interactive shells.
- Executing a command directly bypasses shell restrictions.

## Security Lesson

Restricting interactive shells alone is not enough to secure a system. If users can still execute remote commands through SSH, they may bypass those restrictions. Access control should be enforced at the SSH server level, not only in shell configuration files.

---

# Level 19 → 20 — Understanding setuid

## Concept
Running a program with the permissions of its owner.

## What Happened

A binary named `bandit20-do` had the **setuid** bit enabled.

Instead of running with my permissions, it executed with the permissions of its owner (`bandit20`).

This allowed me to read the password file that my own account could not normally access.

## Commands

```bash
ls -l
```

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

## What I Learned

- The **setuid** bit changes the effective user ID of a process.
- The program executes with the permissions of the file owner.
- This mechanism allows carefully controlled privilege elevation.

## Why is setuid dangerous?

If a root-owned setuid program contains a vulnerability, an attacker may exploit it to execute arbitrary code as root.

This is one of the most common privilege-escalation techniques on Linux systems.

## Security Lesson

The setuid bit is powerful and should only be used when absolutely necessary. Every privileged program must be carefully audited because any vulnerability can lead to privilege escalation.

---

# Level 20 → 21 — Netcat Listener and Authentication

## Concept
Communicating with another process using TCP sockets.

## What Happened

A program waited for a TCP connection.

When it received the correct password, it returned the password for the next level.

To complete the challenge, I had to start my own Netcat listener and let the program connect back to me.

## Commands

Terminal 1

```bash
nc -l -p 12345
```

Terminal 2

```bash
./suconnect 12345
```

## Client vs Listener

**Client**

- Initiates a connection.

```text
Client ─────────► Server
```

**Listener**

- Waits for incoming connections.

```text
Server
 ▲
 │
Client
```

## What I Learned

- TCP communication always has one side initiating the connection.
- Netcat can act as either a client or a server.
- Many penetration testing tools build on these networking fundamentals.

## Security Lesson

Many remote access tools and reverse shells rely on outbound connections instead of inbound ones because outbound traffic is often allowed through firewalls. Understanding listeners and client connections is fundamental to understanding network security.

