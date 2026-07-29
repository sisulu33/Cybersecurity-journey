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
