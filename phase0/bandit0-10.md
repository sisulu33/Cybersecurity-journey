# 🐧 OverTheWire Bandit Notes (Levels 0–10)

## Objective
The Bandit wargame is designed to teach Linux command-line fundamentals. Each level introduces a new command or concept that is commonly used in Linux system administration, cybersecurity, and penetration testing.

---

# Level 0 → 1

## Concept
Connecting to a remote Linux machine using SSH.

## Commands

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

## What I Learned
- SSH (Secure Shell) is used to securely connect to remote systems.
- `-p` specifies the port number.
- You authenticate using a username and password.

---

# Level 1 → 2

## Concept
Reading a file named `-`.

## Commands

```bash
cat ./-
```

## What I Learned
- Linux interprets `-` as an option.
- Prefixing it with `./` tells Linux it is a filename in the current directory.

---

# Level 2 → 3

## Concept
Handling filenames with spaces.

## Commands

```bash
cat "spaces in this filename"
```

or

```bash
cat spaces\ in\ this\ filename
```

## What I Learned
- Spaces separate command arguments.
- Use quotation marks or backslashes (`\`) to treat spaces as part of the filename.

---

# Level 3 → 4

## Concept
Working with hidden files.

## Commands

```bash
ls -la
cat ...Hiding-From-You
```

## What I Learned
- Files beginning with `.` are hidden.
- `ls -a` shows hidden files.
- `ls -l` displays detailed file information.
- `ls -la` combines both options.

---

# Level 4 → 5

## Concept
Identifying file types.

## Commands

```bash
file *
```

## What I Learned
- `file` determines what a file actually contains.
- `*` is a wildcard meaning "every file in the current directory."
- File extensions are not always reliable in Linux.

---

# Level 5 → 6

## Concept
Searching for files using `find`.

## Commands

```bash
find . -type f -size 1033c
```

Useful examples:

```bash
find . -type f
find . -size 1033c
find . -perm 400
```

## What I Learned
- `find` searches directories recursively.
- `.` means search from the current directory.
- `-type f` searches only files.
- `-size 1033c` finds files exactly 1033 bytes in size.

---

# Level 6 → 7

## Concept
Searching the filesystem using multiple conditions.

## Commands

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

## What I Learned
- `/` means search the entire filesystem.
- `-user` filters by owner.
- `-group` filters by group.
- `-size` filters by file size.
- `2>/dev/null` hides permission denied errors.

---

# Level 7 → 8

## Concept
Searching text inside files.

## Commands

```bash
grep "millionth" data.txt
```

## What I Learned
- `grep` searches for matching text.
- It prints only the lines containing the search pattern.

---

# Level 8 → 9

## Concept
Finding the unique line.

## Commands

```bash
sort data.txt | uniq -u
```

## What I Learned
- `sort` groups identical lines together.
- `uniq` only compares adjacent lines.
- `uniq -u` prints only lines that appear exactly once.

---

# Level 9 → 10

## Concept
Extracting readable text from binary files.

## Commands

```bash
strings data.txt | grep "="
```

## What I Learned
- `strings` extracts printable text from binary files.
- `grep` filters the extracted text.
- Combining commands with a pipe (`|`) makes searching much easier.

---

# Commands Learned

| Command | Purpose |
|----------|---------|
| `ssh` | Connect to a remote machine securely |
| `ls` | List directory contents |
| `ls -la` | Show hidden files with detailed information |
| `pwd` | Show current directory |
| `cd` | Change directory |
| `cat` | Display file contents |
| `file` | Identify a file's type |
| `find` | Search for files and directories |
| `grep` | Search text inside files |
| `sort` | Sort lines alphabetically |
| `uniq` | Remove or display duplicate lines |
| `strings` | Extract readable text from binary files |

---

# Important Linux Symbols

| Symbol | Meaning |
|---------|---------|
| `.` | Current directory |
| `..` | Parent directory |
| `*` | Wildcard (everything) |
| `|` | Pipe output into another command |
| `>` | Redirect output |
| `>>` | Append output |
| `2>` | Redirect standard error |
| `/dev/null` | Discard unwanted output |

---

# Key Linux Concepts

## Hidden Files
Any filename beginning with a `.` is hidden.

Examples:

```text
.env
.git
...Hiding-From-You
```

---

## Wildcards

```bash
file *
```

`*` matches every file in the current directory.

---

## Pipes

```bash
sort data.txt | uniq -u
```

The output of the first command becomes the input of the second command.

---

## Error Redirection

```bash
find / 2>/dev/null
```

Redirects error messages to `/dev/null`, preventing "Permission denied" messages from appearing.

---

# Biggest Takeaways

- Learn **why** commands work, not just how to type them.
- Read manual pages with `man <command>` instead of memorizing every option.
- Small Linux commands become powerful when combined with pipes (`|`).
- Wildcards (`*`) save time by operating on multiple files.
- The Linux command line is built around combining simple tools to solve complex tasks.
