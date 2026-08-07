---

# Level 21 → 22 — Cron Jobs and Scheduled Tasks

## Concept
Understanding cron jobs and automated task execution.

## What Happened

The password was not stored directly in the user's home directory.

Instead, a cron job periodically executed a script that copied the password into another location.

By inspecting the cron configuration and reading the executed script, I discovered where the password was being written.

## Commands

```bash
ls /etc/cron.d
```

```bash
cat /etc/cron.d/<cron-file>
```

```bash
cat <script>
```

## What I Learned

- Cron automatically executes scheduled tasks.
- Cron jobs often run with elevated privileges.
- Reading automation scripts is often more valuable than searching for passwords directly.

## Security Lesson

Cron jobs should be reviewed carefully because they execute automatically. Misconfigured scheduled tasks are a common source of privilege escalation and persistence in Linux environments.

---

# Level 22 → 23 — Script Logic Tracing

## Concept
Analyzing program logic instead of blindly executing commands.

## What Happened

The cron job executed a shell script.

Instead of guessing where the password was stored, I read the script line by line to understand exactly what it did.

The script:

1. Generated a filename based on the username.
2. Copied the password into that file.
3. Allowed the file to be read.

By understanding the script's logic, I was able to predict the filename and retrieve the password.

## Commands

```bash
cat /usr/bin/<script-name>
```

## What I Learned

- Reading source code is often the fastest way to understand a system.
- Automation scripts frequently reveal hidden behavior.
- Understanding the logic behind a program is more valuable than memorizing commands.

## Why was this a security weakness?

The script copied sensitive data into a **predictable filename** that other users could discover.

It relied on the filename being difficult to guess instead of protecting the file with proper access controls.

## Security Lesson

Security should never depend on secrecy of filenames or implementation details.

Sensitive information should be protected using proper permissions, least privilege, and secure design—not by assuming nobody will find it.

---
# Level 23 → 24 — Cron Jobs, Privilege Escalation, and Debugging

## Objective

Retrieve the password for **bandit24** by taking advantage of a cron job that executes scripts placed in a specific directory.

---

## Concept

This level demonstrates **privilege escalation through automated task execution**.

A cron job running as **bandit24** automatically executes every script placed in:

```text
/var/spool/bandit24/foo
```

Although I cannot directly read:

```text
/etc/bandit_pass/bandit24
```

I can write a script that **bandit24** will execute on my behalf.

---

## Attack Chain

```text
Me (bandit23)
      │
      │ Create script
      ▼
/var/spool/bandit24/foo
      │
      │ Cron detects new script
      ▼
Cron Job (running as bandit24)
      │
      │ Executes my script
      ▼
My script runs with bandit24's permissions
      │
      │ Reads:
      ▼
/etc/bandit_pass/bandit24
      │
      │ Writes password
      ▼
/tmp/my-temp-directory/password
      │
      ▼
I read the password
```

---

## Steps

### 1. Create a temporary directory

```bash
WORKDIR=$(mktemp -d)
chmod 777 "$WORKDIR"
```

The directory must be writable so the cron job (running as **bandit24**) can write the password there.

---

### 2. Create the payload

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/<my-temp-directory>/password
```

---

### 3. Make it executable

```bash
chmod +x payload.sh
```

---

### 4. Copy it into the spool directory

```bash
cp payload.sh /var/spool/bandit24/foo/
```

The cron daemon automatically executes it.

---

### 5. Read the password

```bash
cat /tmp/<my-temp-directory>/password
```

---

# Debugging: Two Silent Failure Bugs

This level taught me that debugging is just as important as knowing the commands.

## Bug #1 — Watching the wrong command

I tried:

```bash
watch -n 5 la -la /tmp/...
```

which produced:

```text
sh: 1: la: Permission denied
```

### Cause

`la` was a shell alias, not a real executable.

`watch` does **not** use shell aliases.

### Fix

Use the actual command:

```bash
watch -n 5 ls -la /tmp/...
```

---

## Bug #2 — Writing to the wrong directory

My payload wrote to:

```text
/tmp/tmp.w7NOObmiHq/pass
```

while I was checking:

```text
/tmp/tmp.woK1VY7A1A/
```

The script worked correctly—but I was looking in the wrong place.

### Lesson

Always verify file paths when debugging automation.

A single incorrect directory can make it appear as though nothing happened.

---

# Why is this Privilege Escalation?

Normally,

```text
bandit23
```

cannot read:

```text
/etc/bandit_pass/bandit24
```

However, the cron job executes my script **as bandit24**.

That means my script temporarily has **bandit24's permissions**.

Instead of reading the file myself,

I convinced a more privileged process to read it for me.

That is the essence of **privilege escalation**.

---

# Real-World Example

Imagine a Linux server where a cron job runs every minute as **root**.

If an attacker can place or modify a script that cron executes:

```text
Attacker
      │
      ▼
Cron (root)
      │
      ▼
Attacker's script executes as root
      │
      ▼
Reads protected files
Creates admin users
Installs malware
Opens a reverse shell
```

The attacker never logs in as root directly.

Instead, they abuse a trusted automated process that already has root privileges.

This is why scheduled tasks, cron jobs, and systemd timers are common targets during privilege-escalation assessments.

---

# Security Lessons

- Never allow untrusted users to place files in directories executed by privileged processes.
- Scheduled tasks should run with the **least privilege** required.
- Validate ownership and permissions before executing scripts automatically.
- Store sensitive data using proper access controls—not in locations writable or predictable by untrusted users.
- Debugging automation often comes down to checking paths, permissions, and execution context.

---

# Key Takeaways

- Learned how cron jobs execute automated tasks.
- Understood how higher-privileged processes can be abused.
- Practiced debugging silent failures caused by incorrect commands and file paths.
- Saw a realistic example of Linux privilege escalation through scheduled task execution.
- Reinforced the principle that understanding *how* a system works is more valuable than memorizing commands.

# Level 24 → 25 — Brute-Forcing a PIN

## Objective

Authenticate to a service that requires the current password and an unknown 4-digit PIN.

---

## Concept

This level demonstrates a **brute-force attack**.

The password was already known, but the PIN was not. Since the PIN was only four digits long (0000–9999), it was feasible to automate every possible attempt.

---

## Steps

### 1. Read the current password

```bash
password=$(cat /etc/bandit_pass/bandit24)
```

### 2. Generate every possible 4-digit PIN

```bash
seq -w 0 9999
```

The `-w` option pads numbers with leading zeros so every PIN is exactly four digits.

Example:

```text
0000
0001
...
9999
```

### 3. Create all password/PIN combinations

```bash
for pin in $(seq -w 0 9999); do
    echo "$password $pin"
done > combinations.txt
```

### 4. Send every attempt to the service

```bash
nc localhost 30002 < combinations.txt
```

The server checks each line until it finds the correct PIN, then returns the next password.

---

## What I Learned

- How shell loops automate repetitive tasks.
- How `seq -w` generates fixed-width numbers.
- How `nc` streams input directly to a TCP service.
- Why short secrets are vulnerable to brute-force attacks.

---

## Security Lesson

A 4-digit PIN provides only 10,000 possible combinations. Without protections such as rate limiting, account lockouts, or multi-factor authentication, brute-force attacks become practical.

---

# Level 25 → 26 — Restricted Login Shells and Shell Escapes

## Objective

Gain access to a shell even though the user's login shell is not Bash.

---

## Concept

Linux users are not required to use `/bin/bash` as their login shell. The login shell can be **any executable**.

In this level, `bandit26` used a custom program instead of Bash. The program displayed a file using `more` and then exited, preventing a normal interactive shell.

The solution was to exploit a **shell escape** feature provided by `more`.

---

## Attack Flow

```text
SSH Login
      │
      ▼
Custom Login Shell
      │
      ▼
more
      │
      ▼
Shell Escape
      │
      ▼
Interactive Shell
```

---

## What I Learned

- Every Linux user has a login shell defined in `/etc/passwd`.
- Programs such as `more`, `less`, and `vim` may provide shell escape functionality.
- Restricted shells are not always secure if the allowed program can launch a shell.

---

## Security Lesson

Replacing Bash with another program is not sufficient security. If the replacement program allows command execution or shell escapes, users may bypass the restriction completely.

---

# Level 26 → 27 — Privilege Escalation with setuid

## Objective

Use the `bandit27-do` binary to execute a command with the permissions of another user.

---

## Concept

The `bandit27-do` binary has the **setuid** bit enabled.

When executed, it runs with the permissions of its owner (`bandit27`) rather than the user who launched it (`bandit26`).

This allows controlled privilege elevation.

---

## Command

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

---

## What I Learned

- The **setuid** bit changes a program's effective user ID.
- Programs can temporarily execute with another user's permissions.
- This mechanism enables legitimate administrative tasks but also presents security risks.

---

## Why is setuid dangerous?

If a privileged setuid program contains a vulnerability, an attacker may exploit it to execute arbitrary commands with elevated privileges.

Misconfigured or vulnerable setuid binaries are a common source of privilege escalation.

---

## Security Lesson

Only trusted and carefully audited programs should use the setuid bit. The principle of least privilege should always be followed.

---

# Level 27 → 28 — Cloning a Git Repository

## Objective

Clone a Git repository over SSH and retrieve the password stored within it.

---

## Concept

This level introduces **Git** as a version control system.

Instead of searching the filesystem, the password is stored in a remote Git repository. The task is to authenticate over SSH, clone the repository, and inspect its contents.

---

## Commands

Clone the repository:

```bash
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
```

Navigate into the repository:

```bash
cd repo
```

View the files:

```bash
ls
cat README
```

---

## What I Learned

- How to clone a Git repository using SSH.
- Git repositories contain a working copy of tracked files.
- SSH keys and Git authentication work together to provide secure access.

---

## Security Lesson

Sensitive information should never be committed to source control. Even if removed later, Git preserves historical versions, making accidental secret exposure a significant security risk.

