# OverTheWire Bandit — Levels 0–10

My notes learning Linux fundamentals through the Bandit wargame.

## Level 0 → 1
- **Concept:** Reading a file in your home directory.
- **Command:** `cat readme`
- **Learned:** `ls` to list, `cat` to read. Password was in a file.

## Level 1 → 2
- **Concept:** Filenames that start with `-` confuse commands (look like flags).
- **Command:** `cat ./-`
- **Learned:** `./` forces the shell to treat it as a filename, not an option.

## Level 4 → 5
- **Concept:** Finding the one readable file among binary junk.
- **Command:** `file ./*` then read the ASCII one.
- **Learned:** `file` tells you a file's type. Check before you `cat`. Also: argument injection — filenames that look like flags are a real attack class.

## Level 6 → 7
- **Concept:** Finding a file anywhere on the system by owner/group/size.
- **Command:** `find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null`
- **Learned:** `find` searches by criteria; `2>/dev/null` sends errors (stderr, stream 2) to the void so results stay clean.

## Level 9 → 10
- **Concept:** Base64 is ENCODING, not encryption.
- **Command:** `base64 -d data.txt`
- **Learned:** Base64 is reversible by anyone — no key, no secrecy. Spotting it used "for security" is a real vulnerability finding.

