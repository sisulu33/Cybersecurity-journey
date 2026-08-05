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



