# The Machinery of Trust: An Illustrated Journey Into Authentication, Authorization & Security Engineering

Identity is the foundation of execution. Before a system can execute an instruction on behalf of an entity, it must answer two questions: Who are you? and Do you have permission? 

This document reverse engineers the machinery of trust, from physical ancestors to cryptographic implementations, system calls, and network packets.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 1: The Problem of Trust

## Why Does This Exist?
Trust is not a natural property of physical systems. Left alone, physical and virtual environments operate strictly on physical forces and raw data. Trust is an artificial constraint overlaying these mechanisms to restrict access to scarce resources. 

Before digital computers, human societies engineered mechanisms to assert identity and verify claims:

* **House Keys**: A mechanical physical possession token. The system (the lock) does not know *who* inserts the key; it only verifies the possession of a physical object with a cut pattern matching its internal pin configuration.
* **Passports**: A centralized authority claims validation document. The target country trusts the issuing state's cryptographic paper markers (watermarks, seals) to verify that the bearer is the individual pictured.
* **Military Checkpoints**: Shared temporal secrets (passwords). The sentinel demands a word; the traveler speaks the word. Both rely on a shared distribution mechanism occurred prior to the encounter.
* **Bank Signatures**: Behavioral biometric verification. The teller compares the dynamic strokes of a pen on a paper slip to a card kept in a physical drawer, verifying authorship via micro-movements.
* **ATM Cards**: Multi-factor verification combining physical possession (magnetic stripe/chip) with a memorized value (PIN).
* **Employee Badges**: Radio Frequency Identification (RFID) proximity tokens, transmitting a static unique identification number when energized by a reader.
* **Hotel Key Cards**: Magnetic strip or NFC cards containing a transient cryptographic token or sector offset pointing to a room authorization window.

```text
Alice claims:
"I am Alice"

          │
          ▼
System asks:
"Prove it."
          │
          ├─────────────────────────┼─────────────────────────┐
          ▼                         ▼                         ▼
   [ What You Know ]        [ What You Have ]        [ What You Are ]
   - Password               - Hardware Key           - Fingerprint
   - Passphrase             - TOTP Token             - Iris Scan
   - PIN                    - Smart Card             - Face ID
```

---

## Ground Reality
When Alice asserts an identity to a system, the system cannot verify Alice directly. Instead, the system verifies a **proxy** of Alice (a password, a key, a token). 

This introduces a permanent security gap: anyone who steals the proxy *becomes* Alice to the system. Security engineering is the discipline of making the proxy as hard to duplicate, intercept, or forge as physically possible.

---

## Mental Model
Think of trust as a **bridge**. On one side is a claim; on the other is a resource. The gatekeeper will not allow passage until a proof is submitted that spans the bridge. The strength of the bridge depends entirely on the mathematical or physical difficulty of forging that proof.

```text
[ Claim: "I am Alice" ] ══════════ ( The Proof ) ══════════► [ Resource ]
                                         ▲
                                         │
                                 [ Gatekeeper ]
                                 - Validates Proof
                                 - Grants Passage
```

---

## Visual Diagrams

### Physical vs Digital Trust Verification Flow

```text
Physical (Passport Control)
 Traveler           Agent             Passport Database
    │                 │                      │
    │─── Passport ───►│                      │
    │    & Face       │─── Query validity ──►│
    │                 │◄── Active/Valid ─────│
    │◄── Pass/Entry ──│

Digital (Cryptographic Challenge-Response)
 Client             Server            Secret Database
    │                 │                      │
    │─── Claim ID ───►│                      │
    │                 │─── Get Public Key ──►│
    │                 │◄── Public Key/Hash ──│
    │◄── Challenge ───│
    │    (Nonce)      │
    │                 │
    │─── Signature ──►│─── Verify Signature
    │    (Signed)     │    with Public Key
    │◄── Success ─────│
```

---

## Under The Hood
Trust verification at the hardware level is a reduction to zero or one. The CPU core branches based on a register comparison.

```assembly
; Bare-metal pseudo-assembly for checking a PIN
mov eax, [user_input_pin]
cmp eax, [stored_pin]
je  .granted
; Access Denied action
jmp .denied
```

---

## Packet Journey
No packet is exchanged for physical trust; however, the state transition maps directly to electrical signals. For example, a card reader triggers a GPIO interrupt line on a microcontroller when a card is swiped:

```text
Reader Pin:  LOW ─────────┐
                          │ (Insertion Event)
                          ▼
             HIGH ───────────────────────────────────
```

---

## Kernel Perspective
In Linux, trust is tracked via credentials attached to the task execution block (`task_struct->cred`). The kernel does not care about usernames; it checks UID numbers during privileged system calls.

---

## Observe It Yourself
Inspect your current credentials in a Linux terminal:
```bash
id
```
Output:
```text
uid=1000(om) gid=1000(om) groups=1000(om),4(adm),24(cdrom),27(sudo)...
```

---

## Common Misconceptions
> **Myth**: "If a system successfully authenticates you, it trusts you."
> 
> **Reality**: No. Authentication only asserts identity. Trust is transient; the system validates permissions (authorization) on every single transaction, never trusting the client implicitly.

---

## Attack Scenarios
* **The Imposter (Physical)**: An attacker wears a hi-vis vest and carries a clipboard, bypassing a badge reader by tailgating an employee.
* **The Replay (Digital)**: An attacker captures the data packets of a successful badge read or password submission and transmits them again to gain access.

---

## Defenses
* **Multi-Factor Verification**: Requiring multiple distinct categories of proof (possession, knowledge, biometrics).
* **Cryptographic Signatures**: Signing identity claims with private keys that never leave secure hardware enclaves.

---

## Tradeoffs
* **Friction vs Security**: High-security systems (iris scans, hardware tokens) prevent seamless user access, while low-friction systems (persistent cookies, weak passwords) are easily exploited.

---

## Related Concepts
* **AuthN**: Verification of identity.
* **AuthR**: Verification of privileges.

---

## Deep Dive
Read the historical origins of trust tokens:
* Roman *Tesserae*: Small clay tokens used as tickets or identity tokens.
* Watchwords in 18th-century warfare: Sequential shifting codes distributed daily.

---

## Case Studies
* **The Great Walk-In**: In the early era of computing, mainframes were secured only by physical doors. Accessing the terminal room granted root console access by default.

---

## Mini Projects
* **Physical-to-Digital Lock Simulator**: Write a C script that reads input from a mocked serial port buffer representing an RFID reader and validates the card identifier against a local file.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 2: Identity, Claims and Proof

## Why Does This Exist?
In any multi-user network or application, the host must distinguish between different actors. **Identity** is the unique name of an entity. **Claims** are statements the entity makes about itself (e.g., "I am admin," "My email is alice@xyz.com"). **Proof** is the verifiable evidence submitted to substantiate those claims.

---

## Ground Reality
To a computer, a claim is just a string of characters:
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
Without proof, these claims are worthless. Anyone can construct a JSON block claiming to be admin.

---

## Mental Model
Imagine a job applicant submitting a resume:
* **Identity**: The applicant's name on the cover letter.
* **Claims**: The degrees and certifications listed.
* **Proof**: The physical university diploma with an embossed seal.

---

## Visual Diagrams

### The Verification Triangle

```text
             [ Issuer / Authority ]
             (Verifies identity &
              issues signed claims)
                 /          \
                /            \
    Issues     /              \   Trusts
   Signature  /                \  Issuer
             ▼                  ▼
       [ Subject ] ─────────► [ Verifier ]
        (Alice)     Presents   (Server)
                     Claims
```

---

## Under The Hood
How a server validates a claim block using cryptographic signatures:

```text
Received Claims: {"sub": "alice", "role": "admin"}
Received Signature: 0x8f2c...

Verification Math:
Signature Decrypted with Public Key == Hash(Claims JSON)
```

---

## Packet Journey
An HTTP request presenting claims via an Authorization Header:

```http
GET /api/dashboard HTTP/1.1
Host: secure.company.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Kernel Perspective
The kernel tracks identity within processes via UID and GID mappings:

```text
task_struct
  └── cred
        ├── uid (Real UID)
        ├── euid (Effective UID used for access checks)
        └── suid (Saved UID)
```

---

## Observe It Yourself
Check the real vs effective user ID of a process running in Linux:
```bash
grep -i uid /proc/self/status
```
Output:
```text
Uid:	1000	1000	1000	1000
```
*(The four columns represent: Real, Effective, Saved, and Filesystem UIDs)*

---

## Common Misconceptions
> **Myth**: "A claim is secure if it is Base64 encoded."
> 
> **Reality**: Base64 is representation encoding, not encryption or validation. Anyone can decode, modify, and re-encode a Base64 string.

---

## Attack Scenarios
* **Claim Tampering**: An attacker intercepts a session token containing `"admin": false` and changes it to `"admin": true` before forwarding the packet.
* **Identity Spoofing**: Registering a username like `admin` (or using homoglyphs like `admın`) to bypass user checks.

---

## Defenses
* **Cryptographic Signatures**: Protect claims using HMAC or RSA/ECDSA signatures.
* **Unpredictable Identifiers**: Use random UUIDv4 or sequential UUIDv7 rather than sequential auto-incrementing integers (`1`, `2`, `3`) to identify entities.

---

## Tradeoffs
* **Stateless Claims (JWT)**: Fast verification, but difficult to revoke before expiration.
* **Stateful Claims (Database sessions)**: Instant revocation, but requires database lookups on every request.

---

## Related Concepts
* **SAML**: XML-based claim system.
* **OIDC**: JSON-based claim system.

---

## Deep Dive
Learn the details of UUIDv4 generation:
* 122 bits of pure entropy.
* 6 bits of version/variant metadata.
* Total space: $2^{122} \approx 5.3 \times 10^{36}$ combinations, making collisions mathematically impossible under ordinary operations.

---

## Case Studies
* **The Insecure Direct Object Reference (IDOR) exploit**: An attacker changes their profile URL from `example.com/user/102` to `example.com/user/101` and views another user's private claims because the system trusted the URL parameter without validating authorization.

---

## Mini Projects
* **Claims Cryptographic Signer**: Write a Python script that takes a JSON payload, signs it using an HMAC-SHA256 key, and outputs a formatted claims block with signature appended.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 3: Authentication vs Authorization

## Why Does This Exist?
A system needs to partition access control into two phases to ensure separation of concerns:
1. **Who is the user?** (Authentication)
2. **What can they do?** (Authorization)

Conflating these two leads to code vulnerability, where checking if a user exists is treated as equivalent to verifying they have administrative permissions.

---

## Ground Reality
Consider the differences in status codes in HTTP responses:
* **401 Unauthorized**: Misnamed; it means *Unauthenticated*. The server does not know who you are.
* **403 Forbidden**: You are authenticated, but you do not have permission to access the resource.

---

## Mental Model
Think of entering a high-security office building:
* **Authentication**: Showing your driver's license to the security guard at the front desk. They verify your photo matches your face and issue you a badge.
* **Authorization**: Swiping your badge at the door of the server room. The reader determines if your badge is configured to unlock that door.

```text
[ Traveler ] ──► [ Front Desk ] ──► Authenticated! (Issued badge)
                       │
                       ▼
                 [ Server Room ] ──► Swipe Badge ──► Access Denied (Unauthorized)
```

---

## Visual Diagrams

### Authentication and Authorization Separation Flow

```text
User            Auth Server         Resource Gateway       Resource DB
 │                  │                      │                    │
 │─── Credentials ─►│                      │                    │
 │    (Password)    │                      │                    │
 │◄── JWT Token ────│                      │                    │
 │    (Asserts ID)  │                      │                    │
 │                  │                      │                    │
 │─── Request + Token ────────────────────►│                    │
 │                                         │─── Check Signature │
 │                                         │─── Verify Role ────│
 │                                         │◄── Authorized ─────│
 │                                         │───────────────────►│
 │◄── Data Payload ────────────────────────│                    │
```

---

## Under The Hood
Code structure demonstrating separation:

```python
# Authentication Phase
user = authenticate_user(username, password)
if not user:
    return HTTP_401_UNAUTHORIZED

# Authorization Phase
if not user.has_permission("write:reports"):
    return HTTP_403_FORBIDDEN
```

---

## Packet Journey
Tracing the header changes:

```text
1. Credentials POST:
   POST /login
   Body: {"user": "bob", "pass": "secret"}

2. Service response:
   HTTP/2 200 OK
   Set-Cookie: session_id=abc123xyz; Secure; HttpOnly

3. Subsequent authorized request:
   GET /admin/billing
   Cookie: session_id=abc123xyz
```

---

## Kernel Perspective
Linux processes call `setuid()` to elevate or lower privileges. A process starts as root (authenticated) and sets its effective UID (`euid`) to a normal user to run code safely (authorized limit).

---

## Observe It Yourself
See how file permissions enforce authorization:
```bash
ls -l /etc/shadow
```
Output:
```text
-rw-r----- 1 root shadow 1251 May 14 10:22 /etc/shadow
```
*(Only root and members of the shadow group are authorized to read this file)*

---

## Common Misconceptions
> **Myth**: "Using OAuth2 means my API handles authorization."
> 
> **Reality**: No. OAuth2 is a delegation protocol. It handles authorization for client applications, but your API must still authorize user actions locally.

---

## Attack Scenarios
* **Privilege Escalation**: An authenticated user modifies their profile update requests to include a role change parameter (`"role": "admin"`) and elevates their authorization.

---

## Defenses
* **Defense in Depth**: Always verify authorization on the server side, even if the user interface hides restricted options from the user.

---

## Tradeoffs
* **Centralized Authorization**: Clean policy definition, but introduces network latency.
* **Decentralized Authorization**: Fast (tokens have roles embedded), but changes to permissions cannot be revoked instantly.

---

## Related Concepts
* **RBAC**: Role-Based Access Control.
* **ABAC**: Attribute-Based Access Control.

---

## Deep Dive
Read the POSIX capabilities manual (`capabilities(7)`). Linux splits root privileges into distinct capability flags (e.g. `CAP_NET_BIND_SERVICE`, `CAP_SYS_ADMIN`), allowing fine-grained authorization for processes without granting full root access.

---

## Case Studies
* **The Zoom Meeting Hijackings (2020)**: Meeting authentication was successful (users logged in), but missing authorization checks allowed arbitrary authenticated users to join and present in private meetings.

---

## Mini Projects
* **Access Control Gatekeeper**: Write a small middleware class in Python that intercepts requests and evaluates permission annotations against a mocked user role mapping.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 4: Passwords

## Why Does This Exist?
The password is the oldest and most widely used factor of digital authentication. It is a shared secret stored in the system database and memorized by the user. It relies on the assumption that only the genuine user knows this specific secret sequence of characters.

---

## Ground Reality
Humans are poor entropy generators. They choose patterns that are easy to remember, leading to widespread vulnerabilities.
Common patterns like `123456`, `password`, and `qwerty` account for a significant percentage of all credentials globally.

---

## Mental Model
Think of a password as a **secret handshake**. If a guest arrives at the door and mimics the handshake exactly, the door opens. The problem is that anyone who observes the handshake from the window can replicate it later.

---

## Visual Diagrams

### Passwords Transmission Flow (HTTPS Secure Channel)

```text
Client Browser                                    Web Server
┌───────────────────────┐                        ┌───────────────────────┐
│ User enters:          │                        │ Receives plaintext    │
│ "supersecret123"      │                        │ password over TLS:    │
└──────────┬────────────┘                        │ "supersecret123"      │
           │                                     └──────────┬────────────┘
           │ (Encrypted via TLS Tunnel)                     │
           ▼                                                ▼
==================================================   [ Hash Verification ]
  Ciphertext: 0x9f238a8cd8f...                       - Runs bcrypt/Argon2
==================================================   - Compares with DB
```

---

## Under The Hood
When a password is sent, it exists in memory as a character array. Secure systems zero out this memory space immediately after hashing to prevent memory dumps from leaking plaintext.

```c
// Erasing memory trace after validation
char password[64];
// ... read and verify ...
memset_s(password, sizeof(password), 0, sizeof(password));
```

---

## Packet Journey
An HTTP request containing raw credentials:

```http
POST /login HTTP/1.1
Host: api.app.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 37

username=alice&password=supersecret123
```

---

## Kernel Perspective
Linux reads raw console input characters via the TTY driver, disabling echo mode (`ECHO` flag in `termios` struct) to prevent the characters from showing on screen during password entry.

---

## Observe It Yourself
Disable echo mode using python to enter a password silently:
```python
import getpass
passwd = getpass.getpass("Enter secret: ")
print("Password loaded to memory.")
```

---

## Common Misconceptions
> **Myth**: "Hashing passwords on the client side before sending them to the server makes HTTPS unnecessary."
> 
> **Reality**: No. If you hash on the client, the hash itself becomes the new de facto password. An attacker capturing the hash can replay it to log in.

---

## Attack Scenarios
* **Credential Stuffing**: Attackers take credential lists leaked from other site breaches and automatically try them on thousands of other popular sites using bot nets.

---

## Defenses
* **Enforced Minimum Entropy**: Measuring length and character distribution rather than arbitrary character checklist rules.
* **Rate Limiting**: Blocking IP addresses or accounts after repeated failed attempts.

---

## Tradeoffs
* **High Complexity Requirements**: Leads to users writing passwords down on physical notes or reusing them across sites.
* **Low Complexity Requirements**: Easily compromised by basic dictionary attacks.

---

## Related Concepts
* **Passphrases**: Using multiple random words (e.g. `correcthorsebatterystaple`) to increase entropy while maintaining readability.

---

## Deep Dive
Entropy calculation for passwords:
$$H = L \log_2 R$$
Where:
* $L$ = password length
* $R$ = character pool size (e.g. 26 lowercase, 10 digits = 36)
A random 8-character password ($36^8$) yields $\approx 41$ bits of entropy. A 16-character passphrase ($26^{16}$) yields $\approx 75$ bits.

---

## Case Studies
* **The RockYou Leak (2009)**: A social gaming network stored 32 million passwords in plaintext on their database. The resulting leaked list became the standard wordlist used by security tools globally.

---

## Mini Projects
* **Plaintext Input Terminal Masker**: Write a C terminal utility that captures stdin keyboard presses, replacing them with asterisks (`*`) in real time, and stores the buffer in memory.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 5: Password Storage

## Why Does This Exist?
Databases are compromised. If an attacker gains SQL injection access, extracts database backups, or reads storage volumes directly, they must not find user passwords in plaintext. **Password storage design** ensures that even with full database read access, an attacker cannot reverse the stored values to retrieve the original passwords.

---

## Ground Reality
Never write code that saves plaintext passwords to storage:
```sql
-- NEVER DO THIS
INSERT INTO users (username, password) VALUES ('bob', '123456');
```
Instead, save only the non-reversible cryptographic hashes.

---

## Mental Model
Think of password storage as a **museum display locker**. The password is the physical artifact. Instead of keeping the artifact inside the locker, the museum takes a high-detail clay impression (hash). When you present the artifact at the door, they fit it into the impression to see if it matches. If someone steals the locker, they only get clay impressions, from which they cannot reconstruct the original artifact.

---

## Visual Diagrams

### Plaintext vs Cryptographic Storage Architecture

```text
Bad Storage Model (Plaintext Leak)
┌──────────────┐      ┌─────────────┐
│ DB Attacker  │ ──►  │ Users Table │
└──────────────┘      ├─────────────┼──────────┐
                      │ username    │ password │
                      ├─────────────┼──────────┤
                      │ alice       │ p@ssw0rd │ (Leaked!)
                      └─────────────┴──────────┘

Good Storage Model (One-Way Hash Protection)
┌──────────────┐      ┌─────────────┐
│ DB Attacker  │ ──►  │ Users Table │
└──────────────┘      ├─────────────┼────────────────────────────────────┐
                      │ username    │ password_hash                      │
                      ├─────────────┼────────────────────────────────────┤
                      │ alice       │ $2b$12$L7R2x... (bcrypt signature) │
                      └─────────────┴────────────────────────────────────┘
```

---

## Under The Hood
Database storage scheme comparison for Linux shadow entries:

```text
/etc/shadow entry layout:
root:$6$12345abcde$z.qwe987...:18205:0:99999:7:::
  │  │  │          │
  │  │  │          └─── Cryptographic Hash
  │  │  └────────────── Salt value
  │  └───────────────── Hashing Algorithm (6 = SHA-512)
  └──────────────────── Username
```

---

## Packet Journey
When verifying a user's password, the query sent to the database pulls the hash, not the password:

```sql
SELECT password_hash FROM users WHERE username = 'alice';
```

---

## Kernel Perspective
The kernel uses system keyring structures (`keyutils`) to keep temporary authentication keys in non-pageable memory (`mlock()`), preventing them from being written to swap files on disk.

---

## Observe It Yourself
View the hashing configuration of your local machine's pam module:
```bash
grep -v '^#' /etc/pam.d/common-password
```

---

## Common Misconceptions
> **Myth**: "Encrypting the passwords table with AES-256 is safe."
> 
> **Reality**: No. Encryption is symmetric and reversible. If an attacker gains full database access, they will likely find the AES key in the application config files or environment variables.

---

## Attack Scenarios
* **Offline Cracking**: An attacker downloads the database hash column and runs specialized tools (e.g. Hashcat, John the Ripper) to crack the hashes on their own GPU clusters.

---

## Defenses
* **Slow Hash Functions**: Do not use standard fast hash functions like MD5, SHA-1, or SHA-256 for password storage. Use key-stretching algorithms.

---

## Tradeoffs
* **High Hash Cost**: Protects against cracking, but increases CPU utilization on the web server during login processing.

---

## Related Concepts
* **Entropy**: The measurement of unpredictability.

---

## Deep Dive
How the system verifies storage records:
1. User submits plaintext `p`.
2. Retrieve stored row `row = DB.find(username)`.
3. Extract salt `s` from `row.password_hash`.
4. Compute `h' = KeyStretch(p, s)`.
5. Perform constant-time comparison `h' == row.password_hash`.

---

## Case Studies
* **The LinkedIn Leak (2012)**: LinkedIn stored 6.5 million passwords hashed with SHA-1 without salts. Attackers cracked over 90% of the hashes within a few days using standard GPU systems.

---

## Mini Projects
* **Shadow File Parser**: Write a Python parser that reads a mock `/etc/shadow` file format, extracts the salt and algorithm parameters, and checks input passwords against it.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 6: Hash Functions

## Why Does This Exist?
A **hash function** is a mathematical algorithm that maps data of arbitrary size to a bit array of fixed size (a digest). It is designed to be a one-way function: it is computationally easy to compute the output from the input, but mathematically impossible to reverse the output to reconstruct the input.

```text
Input: "hello" ────────────────► [ SHA-256 ] ──► 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
Input: "hello world" ──────────► [ SHA-256 ] ──► b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9
```

---

## Ground Reality
Cryptographic hash functions must satisfy three properties:
1. **Pre-image resistance**: Given hash $H$, it is impossible to find message $M$ such that $Hash(M) = H$.
2. **Second pre-image resistance**: Given message $M_1$, it is impossible to find another message $M_2$ such that $Hash(M_1) = Hash(M_2)$.
3. **Collision resistance**: It is impossible to find *any* two distinct messages $M_1$ and $M_2$ such that $Hash(M_1) = Hash(M_2)$.

---

## Mental Model
Think of a hash function as a **fruit blender**. It is easy to throw in strawberries, bananas, and yogurt to produce a pink smoothie. However, looking at the smoothie, it is impossible to extract the original intact strawberries and bananas.

---

## Visual Diagrams

### Compression Loop of Merkle-Damgård Construction

```text
  Message Chunk 0     Message Chunk 1     Message Chunk 2
        │                   │                   │
        ▼                   ▼                   ▼
IV ──► [f] ───────────────► [f] ───────────────► [f] ──► Final Hash Digest
       (Compression         (Compression         (Compression
        Function)            Function)            Function)
```

---

## Under The Hood
Here is a basic SHA-256 block processing round inside the CPU:

```c
// SHA-256 compression block processing pseudo-code
uint32_t a = state[0], b = state[1], c = state[2], d = state[3];
// Perform 64 rounds of mixing:
for (int i = 0; i < 64; i++) {
    uint32_t T1 = h + Sigma1(e) + Ch(e, f, g) + K[i] + W[i];
    uint32_t T2 = Sigma0(a) + Maj(a, b, c);
    h = g; g = f; f = e; e = d + T1;
    d = c; c = b; b = a; a = T1 + T2;
}
```

---

## Packet Journey
Hash operations are local CPU events; no network packets are sent. However, during file downloads, hashes are transmitted in metadata pages (e.g., `SHA256SUMS` files) over HTTP to verify packet integrity.

---

## Kernel Perspective
The Linux Kernel includes a cryptographic subsystem available via the Netlink interface or standard system modules. This accelerates hash computation at the hardware level using Intel SHA extensions or ARM Cryptography instructions.

---

## Observe It Yourself
Generate a SHA-256 digest of a string in your terminal:
```bash
echo -n "testpassword" | sha256sum
```
Output:
```text
fcd73266cf9b82f0ca3a2c5a089069dfb4e183a65239a7b9736fdfd3744955b2  -
```

---

## Common Misconceptions
> **Myth**: "Cryptographic hashing is just encryption without a key."
> 
> **Reality**: No. Encryption is designed to be reversible (decryption). Hashing is structurally lossy; infinite inputs map to a finite space of outputs, meaning information is permanently discarded during compression.

---

## Attack Scenarios
* **Length Extension Attack**: An attacker takes a hash signature computed from `Hash(Secret || Message)` and appends data to the message, generating a valid signature for the expanded message without knowing the secret. SHA-256 is vulnerable to this; SHA-3 and HMAC-SHA256 are not.

---

## Defenses
* **Use HMAC**: Construct secure keyed-hash message authentication codes instead of simple concatenation layouts:
  $$\text{HMAC}(K, M) = \text{Hash}((K \oplus opad) \mathbin{\Vert} \text{Hash}((K \oplus ipad) \mathbin{\Vert} M))$$

---

## Tradeoffs
* **Fast Hash (SHA-256)**: Ideal for file validation and network packet signatures, but vulnerable to fast brute-forcing if used for storing passwords.

---

## Related Concepts
* **MD5**: Obsolete 128-bit hash function (broken by collision attacks).
* **SHA-3**: Keccak-based hash algorithm resistant to length extension.

---

## Deep Dive
Explore the math of SHA-256 block sizes:
* Message block size: 512 bits (64 bytes).
* If a message is shorter, it is padded with a trailing `1` bit followed by `0` bits, ending with a 64-bit block indicating the length of the message.

---

## Case Studies
* **Flame Malware (2012)**: Used a MD5 collision attack to forge a digital signature from Microsoft, enabling the malware to install itself as a Windows Update file.

---

## Mini Projects
* **Simple MD5 Collision Checker**: Write a Python script that searches for a hash collision in a custom reduced 16-bit hash algorithm using a brute-force memory map.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 7: Salts and Pepper

## Why Does This Exist?
Even if you use a secure cryptographic hash function, two users choosing the same password (e.g. `password123`) will produce the exact same database hash. Furthermore, attackers can compute millions of common password hashes in advance.
**Salts** and **peppers** introduce unique entropy to each password input to defeat precomputation attacks.

---

## Ground Reality
A **Salt** is a unique, cryptographically random value generated for every user during sign-up. It is stored in plaintext next to the user's hash.
A **Pepper** is a global secret string stored in the application code or a Hardware Security Module (HSM), not in the database.

```text
Without Salt (Identical Hashes)
Alice: "hello" ──► SHA-256 ──► 2cf24dba5f...
Bob:   "hello" ──► SHA-256 ──► 2cf24dba5f...

With Salt (Unique Hashes)
Alice: "hello" + "salt_A" ──► SHA-256 ──► 8f9c122b...
Bob:   "hello" + "salt_B" ──► SHA-256 ──► e4a811c0...
```

---

## Mental Model
Think of a salt as a **custom spice mix** added to a soup. Even if two guests order the same basic tomato soup, the chef sprinkles a unique random combination of salt and herbs into each bowl before serving. If a critic tries to analyze the dishes, they must reverse-engineer every bowl individually rather than generalizing from a single recipe.

---

## Visual Diagrams

### Salted and Peppered Hashing Architecture

```text
Plaintext Input: "supersecret"
       │
       ├──────► [ Salt Generator ] ──► (Random: "8f2a1b9d") ──► Saved in DB
       │
       ├──────► [ Pepper File ]    ──► (Secret: "k3yP@pp3r") ──► Stored in code/HSM
       ▼
[ Concatenator ] ──► "supersecret8f2a1b9dk3yP@pp3r"
       │
       ▼
[ Hash Algorithm ] ──► Stored Hash Digest
```

---

## Under The Hood
How a database table stores salted records:

```text
Table schema structure:
┌──────────┬──────────────┬─────────────────────────────────────────────────┐
│ Username │ Salt (Plain) │ Password Hash Digest                            │
├──────────┼──────────────┼─────────────────────────────────────────────────┤
│ alice    │ a7b8c9d0e1f2 │ 3df2a7bc128e0182fc89d0b...                      │
└──────────┴──────────────┴─────────────────────────────────────────────────┘
```

---

## Packet Journey
The salt is read from the database during verification and does not cross the client network boundaries:

```text
1. Client sends POST: {"user": "alice", "pass": "secret"}
2. Server queries DB: "SELECT salt, hash FROM users WHERE user='alice'"
3. Server computes locally: Hash("secret" + salt + pepper)
4. Compare with DB hash.
```

---

## Kernel Perspective
Linux processes request random bytes for salts using the `getrandom()` system call, which extracts entropy directly from the kernel's entropy pool (gathered from CPU noise, hardware events, and keyboard interrupts).

---

## Observe It Yourself
Use openssl to generate a 16-byte random hex string suitable for a salt:
```bash
openssl rand -hex 16
```
Output:
```text
d8e1215b2ef82c9f4305c48b2d184bf3
```

---

## Common Misconceptions
> **Myth**: "The salt must be kept secret."
> 
> **Reality**: No. The salt can be public. Its purpose is to ensure hash uniqueness per user, preventing rainbow table lookups.

---

## Attack Scenarios
* **Rainbow Table Lookup**: Attackers download precomputed tables of 100 billion common passwords mapped to MD5 or SHA-256 hashes. If a database is unsalted, they lookup the leaked hashes instantly. If the database is salted, their precomputed tables are useless, forcing them to re-hash their dictionary for every individual salt.

---

## Defenses
* **CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)**: Always generate salts using system secure entropy, never using deterministic functions like `rand()` or `Math.random()`.

---

## Tradeoffs
* **Pepper Storage**: If the application server is compromised, the attacker may recover the pepper from configuration files, reducing security to salted hashing.

---

## Related Concepts
* **Entropy Pool**: The physical noise collection in the kernel.

---

## Deep Dive
Look at how shadow salts are represented:
The standard prefix `$6$` in `/etc/shadow` indicates SHA-512. The characters following are the salt (e.g. `$12345abcde$`), followed by the resulting hash.

---

## Case Studies
* **The Ashley Madison Breach (2015)**: The site used bcrypt for some accounts but also stored MD5 hashes of passwords without salts for a legacy login system. Attackers cracked millions of passwords by checking the unsalted MD5 column.

---

## Mini Projects
* **Salted Password Manager**: Write a Python program that registers users, generates a secure random salt, computes a salted SHA-256 hash, and saves the credentials to a local JSON file.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 8: Key Stretching

## Why Does This Exist?
Modern graphics cards (GPUs) can compute billions of SHA-256 hashes per second. If an attacker steals a database of salted SHA-256 hashes, they can brute-force passwords of average length in minutes.
**Key Stretching** algorithms solve this by forcing the hashing process to run slow, requiring significantly more CPU time and memory to compute a single hash.

---

## Ground Reality
Standard fast hash functions are designed for high throughput. Key stretching algorithms (e.g. bcrypt, PBKDF2, Argon2) introduce computational barriers:
* **Time Cost**: Forces the CPU to execute thousands of iterations.
* **Memory Cost**: Requires reading/writing to a large memory array, which makes parallel processing on GPUs highly expensive.

```text
Fast Hash (SHA-256)
Plaintext ──► [ SHA-256 ] ──► (Takes 0.0001 milliseconds)

Key Stretching (Argon2id)
Plaintext ──► [ Memory allocation: 64MB ] ──► [ 3 Iterations ] ──► (Takes 200 milliseconds)
```

---

## Mental Model
Imagine a toll booth on a bridge:
* **Fast Hash**: Drivers pay by throwing a coin into an open box as they drive past at 60 mph.
* **Stretched Hash**: The toll collector forces every driver to stop, exit their vehicle, step on a scale, sign a physical ledger, wait for a stamp, and only then return to their car. This prevents a rush of a million cars crossing the bridge simultaneously.

---

## Visual Diagrams

### Argon2id Memory-Hard Matrix Lane Processing

```text
Memory Blocks (e.g., 64MB Array in RAM)
┌───────────────────────────────────────────────┐
│ Block 0   ◄── [Lane 0] ──► Block 1 ──► Block 2│
├───────────────────────────────────────────────┤
│ Block 3   ◄── [Lane 1] ──► Block 4 ──► Block 5│
└───────────────────────────────────────────────┘
      ▲                           │
      │ Random Access Reads       ▼
      └───────────────────────────┘
(Parallel GPU cores are throttled by RAM bus latency bounds)
```

---

## Under The Hood
The Argon2id mixing round architecture structure:

```c
// Argon2 block mixing function structure details
static void mix_blocks(block *next_block, const block *prev_block, const block *ref_block) {
    block tmp;
    for (size_t i = 0; i < ARGON2_QWORDS_IN_BLOCK; i++) {
        tmp.v[i] = prev_block->v[i] ^ ref_block->v[i];
    }
    // Perform column and row mixing transformations:
    for (size_t i = 0; i < 8; i++) {
        GB(tmp.v + 16 * i);
    }
    // ...
}
```

---

## Packet Journey
Key stretching operations are executed entirely on the authentication server. The network impact is a measured latency delay (e.g., HTTP response delay increases by 150-300ms) on login API calls.

---

## Kernel Perspective
The kernel schedules the key stretching thread as a CPU-bound task. High-priority interactive threads might experience latency spikes if multiple users hit the login endpoint simultaneously.

---

## Observe It Yourself
Benchmark bcrypt generation speeds in python to measure execution times:
```python
import bcrypt
import time

start = time.time()
salt = bcrypt.gensalt(rounds=12) # Cost parameter
hashed = bcrypt.hashpw(b"mypassword", salt)
print(f"Computed hash in {time.time() - start:.3f} seconds.")
```

---

## Common Misconceptions
> **Myth**: "If my server is slow, key stretching will crash my app."
> 
> **Reality**: Key stretching requires careful configuration of resources (CPU, RAM). Setting the parameters too high can expose the application server to Denial of Service (DoS) attacks by flooding the server with fake login requests.

---

## Attack Scenarios
* **ASIC/GPU Parallel Cracking**: Attackers compile custom code onto FPGA or ASIC chips designed specifically to compute bcrypt/Argon2. Memory-hard parameters in Argon2 prevent this advantage by forcing the chips to access external RAM, introducing memory bandwidth delays.

---

## Defenses
* **Configurable Costs**: Update work factors (rounds) periodically to keep pace with improvements in hardware computing speeds.

---

## Tradeoffs
* **Security vs Capacity**: Strong stretching increases server cost. If you scale to millions of concurrent logins, you may need dedicated authentication clusters.

---

## Related Concepts
* **Bcrypt**: Based on the Blowfish block cipher's key setup phase.
* **PBKDF2**: Simple loop-based key derivation function (lacks memory hardness).

---

## Deep Dive
Understand the parameters of Argon2id:
* $m$: Memory size (in kilobytes).
* $t$: Time cost (number of iterations).
* $p$: Parallelism factor (number of active processing threads).

---

## Case Studies
* **The LastPass Breach (2022)**: LastPass used PBKDF2 iterations to protect vault keys. Some user vaults had iterations set to 100,100, while legacy vaults had it set as low as 1, allowing attackers to quickly brute-force stolen user vaults on GPU farms.

---

## Mini Projects
* **Key Stretching Benchmarker**: Write a Python script that generates hashes using PBKDF2-HMAC-SHA256, bcrypt, and Argon2, measuring and plotting CPU execution times against variations in cost parameters.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 9: Sessions and Cookies

## Why Does This Exist?
HTTP is a stateless protocol. By default, every request is processed by the server as if it came from a completely new client. If you log in on page 1, the server forgets who you are when you click page 2.
**Sessions** and **Cookies** solve this by creating a stateful relationship over a stateless channel.

---

## Ground Reality
When you log in, the server generates a unique tracking identifier, associates it with your user record in its memory or database, and returns it to the client browser inside a `Set-Cookie` header. The browser automatically stores this value and appends it to every subsequent HTTP request.

---

## Mental Model
Think of a session as a **cloakroom ticket** at a theater:
1. You hand over your coat (credentials).
2. The attendant hangs it up and gives you a small numbered ticket card (session ID/cookie).
3. Whenever you want to buy a drink or retrieve your coat, you simply show the ticket. You don't have to carry your coat around or prove ownership repeatedly.

```text
User Browser                                      Web Server
┌───────────────────────┐                        ┌───────────────────────┐
│ POST /login           │ ──────────────────────►│ Validates credentials, │
│                       │◄───────────────────────│ generates session ID. │
│                       │   Set-Cookie: ID=9982  │                       │
├───────────────────────┤                        ├───────────────────────┤
│ GET /profile          │ ──────────────────────►│ Looks up ID=9982 in   │
│ Cookie: ID=9982       │◄───────────────────────│ DB, confirms "Alice"  │
└───────────────────────┘                        └───────────────────────┘
```

---

## Visual Diagrams

### Stateful Session Lifecycle & Cookie Storage

```text
Client (Browser)                                  Server Engine
┌──────────────────────┐                         ┌──────────────────────┐
│ Memory: [Empty]      │                         │ DB:                  │
├──────────────────────┤                         │ [No active sessions] │
│ POST /login ─────────┼────────────────────────►│                      │
│                      │                         │ - Gen: "xyz789"      │
│                      │◄────────────────────────│ - Save: {alice, 1h}  │
│                      │  Set-Cookie: SID=xyz789 │                      │
├──────────────────────┤  (Secure; HttpOnly)     ├──────────────────────┤
│ Cookie Store:        │                         │                      │
│ SID=xyz789           │                         │                      │
│                      │                         │                      │
│ GET /profile ────────┼────────────────────────►│ Read Cookie: xyz789  │
│ Cookie: SID=xyz789   │◄────────────────────────│ Resolve: User=alice  │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
The structure of a session record in server-side memory (e.g. Redis key-value layout):

```text
Key: session:xyz789
Value:
{
  "user_id": 1001,
  "username": "alice",
  "created_at": 1774902120,
  "expires_at": 1774905720,
  "ip_address": "192.168.1.52",
  "user_agent": "Mozilla/5.0..."
}
```

---

## Packet Journey
HTTP request headers containing session identifiers:

```http
GET /dashboard HTTP/2
Host: example.com
Cookie: session_id=s%3Aabc123456def789.xyz987654321; theme=dark
```

---

## Kernel Perspective
The kernel treats sockets as simple file descriptors (`fd`). When web servers read or write session cookie bytes to sockets, the kernel processes them as raw network packets via the TCP/IP stack (`sys_write` / `sys_read`), completely unaware of the cookies or sessions concept.

---

## Observe It Yourself
Check cookies in your terminal using curl:
```bash
curl -I https://www.google.com
```
Look for lines containing `Set-Cookie:`.

---

## Common Misconceptions
> **Myth**: "Session cookies are safer than JWTs because they store no data on the client."
> 
> **Reality**: The cookie still stores the session ID. If an attacker steals the session ID, they gain full access to the user account, exactly as they would with a stolen JWT.

---

## Attack Scenarios
* **Session Hijacking (XSS)**: An attacker injects malicious JavaScript into a website. The script reads `document.cookie` and sends the user's session identifier to the attacker's server.

---

## Defenses
* **HttpOnly Flag**: Prevents client-side scripts from reading cookie values.
* **Secure Flag**: Forces cookies to be transmitted only over encrypted (HTTPS) connections.
* **SameSite=Strict/Lax**: Prevents cookies from being sent on cross-site requests, mitigating Cross-Site Request Forgery (CSRF).

---

## Tradeoffs
* **Stateful Scaling**: Requires sharing session state across multiple servers (e.g., using a central Redis cluster), introducing latency and a single point of failure.

---

## Related Concepts
* **CSRF (Cross-Site Request Forgery)**: Forcing a logged-in user's browser to execute state-changing actions on a target site.

---

## Deep Dive
Cookie attribute flags:
* `SameSite=Lax`: Cookie is sent when navigating to the origin site (e.g., clicking a link), but blocked on cross-site subrequests (e.g., images, AJAX).
* `SameSite=Strict`: Cookie is never sent on cross-site requests of any kind.

---

## Case Studies
* **The Yahoo Breach (2015-2016)**: Attackers forged session cookies by stealing Yahoo's proprietary user database and signature keys, allowing them to access 32 million accounts without entering passwords.

---

## Mini Projects
* **Cookie-Based Auth Middleware**: Write a simple Python HTTP server from scratch using standard socket libraries that parses `Cookie` headers and restricts a `/secret` path based on a hardcoded active session dictionary.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 10: Session IDs

## Why Does This Exist?
A session identifier is the single proxy representing user authentication. If the session ID is guessable, predictable, or has low entropy, an attacker can construct valid identifiers offline and gain unauthorized access to random user accounts.
**Cryptographically secure session IDs** ensure that the space of possible IDs is large enough to prevent brute-forcing.

---

## Ground Reality
Never generate session IDs using basic random algorithms:
```javascript
// NEVER DO THIS
let sessionId = Math.random().toString(36);
```
These functions are pseudo-random and their internal states can be determined from past output. Always use Cryptographically Secure Pseudo-Random Number Generators (CSPRNG).

---

## Mental Model
Think of session IDs as a **lottery ticket containing a million digits**. If the winning numbers are sequential, anyone can print tickets with the next numbers and win. If the numbers are truly random and the ticket is long enough, the probability of printing a winning ticket is virtually zero.

---

## Visual Diagrams

### PRNG Predictability vs CSPRNG Randomness

```text
Predictable PRNG (Linear Congruential / Math.random)
Seed ──► [ Internal State ] ──► ID 1 ──► ID 2 ──► ID 3 (State can be solved!)

Unpredictable CSPRNG (Kernel Entropy Pool / /dev/urandom)
Entropy ──► [ Chaotic State ] ──► ID 1 ──► ID 2 ──► ID 3 (Non-deterministic)
```

---

## Under The Hood
How Linux generates cryptographically secure random values:

```c
// Kernel read from /dev/urandom pseudo-code
unsigned char buf[32];
getrandom(buf, sizeof(buf), 0);
// Encode buf to base64 or hex to produce session ID
```

---

## Packet Journey
A packet exchanging session cookies:

```http
HTTP/1.1 200 OK
Set-Cookie: session_id=8f9c1e2b3c4d5e6f7a8b9c0d1e2f3a4b; Path=/; Secure; HttpOnly
```

---

## Kernel Perspective
The kernel tracks entropy in a structure called the **entropy pool**. It monitors physical events (disk interrupts, network packet timings, mouse movements) to update a chaotic state matrix used by `/dev/urandom`.

---

## Observe It Yourself
Check the size of the kernel entropy pool in Linux:
```bash
cat /proc/sys/kernel/random/entropy_avail
```

---

## Common Misconceptions
> **Myth**: "Longer session IDs slow down database queries."
> 
> **Reality**: Session IDs are usually indexed keys in key-value stores like Redis or relational databases, resulting in $O(1)$ or $O(\log N)$ lookup speeds regardless of length.

---

## Attack Scenarios
* **Session ID Brute-Forcing**: An attacker generates millions of session IDs sequentially or based on a known random seed and floods the application server with requests until one matches an active session.

---

## Defenses
* **High Entropy**: Use a minimum of 128 bits of entropy (encoded as 32 hex characters or 22 base64 characters).
* **Session Expiry**: Enforce strict session timeouts (e.g. 15-30 minutes of inactivity) to limit the window of exploitation.

---

## Tradeoffs
* **Memory Cost**: Maintaining millions of active high-entropy session structures on the server consumes physical RAM.

---

## Related Concepts
* **UUIDv4**: A standardized 128-bit random identifier.

---

## Deep Dive
Session ID generation entropy requirement calculations:
To prevent collisions, the probability of guessability is governed by the Birthday Paradox:
$$p \approx 1 - e^{-\frac{k^2}{2^N}}$$
Where:
* $k$ = number of active sessions.
* $N$ = number of possible states ($2^{128}$ for a 128-bit key).
With 10 million active sessions ($k = 10^7$) and 128 bits of entropy ($2^{128}$), the probability of a collision is $\approx 10^{-24}$.

---

## Case Studies
* **The Netscape SSL Bug (1995)**: Netscape's browser generated keys using a seed based on the current system microsecond time and process IDs, allowing attackers to calculate the seed value and crack the session keys within minutes.

---

## Mini Projects
* **CSPRNG Session Generator**: Write a Go or Node.js program that outputs 10,000 unique session IDs using a secure system random generator and verifies that no duplicates are produced.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 11: JWT Internals

## Why Does This Exist?
Stateful session models require the server to look up session IDs in a database on every request. When scaling to millions of concurrent requests, this database lookup becomes a performance bottleneck.
**JSON Web Tokens (JWT)** solve this by storing user claims directly in the token itself on the client, verifying authenticity using cryptographic signatures instead of database state.

---

## Ground Reality
A JWT is a string divided into three parts separated by periods (`.`):
`Header.Payload.Signature`
The header and payload are simple JSON structures encoded using Base64URL. The signature is computed over the encoded header and payload using a secret key.

```text
Header:    {"alg":"HS256","typ":"JWT"}  ──► Base64URL ──► eyJhbGciOiJI...
Payload:   {"sub":"123","admin":true}   ──► Base64URL ──► eyJzdWIiOiIx...
Signature: HMACSHA256(header.payload)  ──► Base64URL ──► eXN0ZWFsdGg...
```

---

## Mental Model
Think of a JWT as a **signed medical prescription**:
* **Header**: Specifies the clinic's printing format.
* **Payload**: States the patient's name and medication details (claims).
* **Signature**: The doctor's handwritten ink signature.
The pharmacist (API Gateway) does not need to call the doctor (Session DB) to verify the prescription; they inspect the signature to verify its authenticity.

---

## Visual Diagrams

### JWT Anatomy & Verification Process

```text
  eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiIxMjM0NTY3ODkwIn0 . 4f8a9c...
  └──────┬───────────┘   └──────────┬─────────────┘   └───┬────┘
         ▼                          ▼                     ▼
     [ Header ]                 [ Payload ]          [ Signature ]
     - Algorithm                - User claims        - HMAC verification
     - Token type               - Expiration           digest
```

---

## Under The Hood
HMAC verification logic inside the API gateway:

```python
# JWT Signature Verification
parts = raw_jwt.split(".")
header_payload = parts[0] + "." + parts[1]
expected_signature = base64url_encode(
    hmac_sha256(key, header_payload)
)
if not constant_time_compare(parts[2], expected_signature):
    raise InvalidTokenException("Signature mismatch.")
```

---

## Packet Journey
The JWT is transmitted in HTTP request headers:

```http
GET /api/v1/user/settings HTTP/1.1
Host: microservice.company.local
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIi...
```

---

## Kernel Perspective
JWT signatures are computationally expensive. Performing cryptographic validations on every API call increases user-space CPU utilization, triggering frequent hardware instruction context shifts.

---

## Observe It Yourself
Decode a JWT payload using bash utilities:
```bash
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ" | base64 --decode
```
Output:
```text
{"sub":"1234567890","name":"John Doe","iat":1516239022}
```

---

## Common Misconceptions
> **Myth**: "JWT payload data is encrypted and secret."
> 
> **Reality**: No. JWTs are signed, not encrypted. The payload is easily decoded by anyone who intercepts the token. Never store sensitive secrets like passwords in the payload.

---

## Attack Scenarios
* **The "none" Algorithm Exploit**: Attackers modify the JWT header to specify `"alg": "none"` and delete the signature block. If the server library validates the signature according to the header algorithm, it accepts the modified payload as valid.

---

## Defenses
* **Disable "none" Algorithm**: Configure the JWT verification library to accept only explicit algorithms (e.g. HS256, RS256).
* **Short Lifespans**: Enforce token expiration times (`exp` claim) to limit the window of exploitation.

---

## Tradeoffs
* **Revocation Bottleneck**: Because JWT validation is stateless, you cannot revoke a token easily before it expires without introducing stateful blacklists.

---

## Related Concepts
* **JWE**: Encrypted JSON Web Token format.

---

## Deep Dive
Asymmetric JWT validation (RS256):
The auth server signs the token using its private key. The API servers verify the signature using the matching public key. This allows microservices to validate tokens without sharing the signing secret.

---

## Case Studies
* **The Equifax Breach (2017)**: Missing JWT verification checks on internal microservice pipelines allowed attackers to query backend databases by bypassing API gateway authorization.

---

## Mini Projects
* **JWT Generator & Verifier**: Write a Python module from scratch (using only standard libraries) that generates and verifies HS256 JWTs.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 12: Bearer Tokens

## Why Does This Exist?
When an application delegates API access to a third-party application, the client needs a simple credential token that it can present to the resource server. The resource server accepts the token as proof of authorization, regardless of who presents it.
**Bearer Tokens** provide a standard access delegation credential.

---

## Ground Reality
"Bearer" means: *whoever holds this token has access*.
There are no additional cryptographic challenge steps required from the client.
If a browser, a mobile app, or an attacker sends the token, the API treats them identical.

---

## Mental Model
Think of a bearer token as a **cash bill**. If you walk into a store and hand the cashier a $20 bill, they accept it. They do not check your ID or ask how you acquired the money. Whoever holds the bill owns the value.

```text
[ Bearer Token ] ═════► [ API Gateway ] ═════► Access Granted!
(No signature or ID verification performed on the bearer)
```

---

## Visual Diagrams

### Bearer Token API Access Flow

```text
Client Application                                Resource Server
┌──────────────────────┐                         ┌──────────────────────┐
│ Memory:              │                         │ Verification DB:     │
│ [Token: "t0k3n123"]  │                         │ [Active token list]  │
├──────────────────────┤                         ├──────────────────────┤
│ GET /data            │ ───────────────────────►│ Parses Header.       │
│ Authorization:       │                         │ Reads token value:   │
│ Bearer t0k3n123      │◄────────────────────────│ "t0k3n123"           │
│                      │      Data Payload       │ - Match: Authorized  │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
How a web application extracts and checks bearer tokens:

```python
# Extracting Bearer Token from HTTP Headers
auth_header = request.headers.get("Authorization")
if auth_header and auth_header.startswith("Bearer "):
    token = auth_header.split(" ")[1]
    # Verify token against DB/Redis
```

---

## Packet Journey
The Bearer token passed in the Authorization header:

```http
GET /api/v2/reports HTTP/1.1
Host: gateway.services.net
Authorization: Bearer s2.09c812d38e819b8...
```

---

## Kernel Perspective
Because bearer tokens are static string credentials, they are stored in the memory heap of user-space processes (web servers, proxies). If the process core-dumps due to a crash, the bearer tokens remain readable in the dump file.

---

## Observe It Yourself
Send a mock bearer token request using curl:
```bash
curl -H "Authorization: Bearer test_token_123" https://httpbin.org/headers
```

---

## Common Misconceptions
> **Myth**: "Bearer tokens are secure because they are sent in the Authorization header."
> 
> **Reality**: The Authorization header is just another HTTP header. It is sent in plaintext over TCP unless encapsulated inside a TLS connection.

---

## Attack Scenarios
* **Token Theft via Log Leakage**: Reverse proxies or application load balancers log incoming HTTP headers to diagnostic files, exposing the bearer tokens in plaintext.

---

## Defenses
* **Never Log Headers**: Explicitly strip the `Authorization` header from application logger configurations.
* **Short Expiry**: Expire bearer tokens within minutes to limit exposure.

---

## Tradeoffs
* **Simplicity**: Extremely easy to implement on clients, but high security risk if intercepted.

---

## Related Concepts
* **OAuth2 Access Token**: Typically implemented as a bearer token.

---

## Deep Dive
DPoP (Demonstrating Proof-of-Possession):
An extension to OAuth2 designed to prevent the exploitation of intercepted bearer tokens by forcing the client to sign the HTTP request with a private key, binding the token to the sender.

---

## Case Studies
* **The Slack Token Exposures (2022)**: Public GitHub repositories were found to contain hardcoded Slack app bearer tokens, allowing external attackers to read internal channel messages.

---

## Mini Projects
* **Bearer Token Generator**: Write a script that generates a cryptographically secure, URL-safe access token prefixed with service identifiers (e.g. `acc_1a2b3c...`).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 13: API Keys

## Why Does This Exist?
When building programmatic interfaces (APIs) designed to be accessed by automated scripts or background services, standard user-interactive login flows (sessions, forms) cannot be used.
**API Keys** provide a static credential identifier used by external services to authenticate requests programmatically.

---

## Ground Reality
API keys are long, random, unique strings generated by the API provider. They are typically passed in custom headers or query parameters. Unlike passwords, they are designed to be rotated, revoked, and scoped to specific permissions.

---

## Mental Model
Think of an API key as a **contractor's master key card**. It is issued to an external contractor to allow access to specific rooms in the building. It is configured to unlock only the target doors and can be disabled instantly by security without changing the building's locks.

```text
External Script ──► [ GET /data?key=ak_123 ] ──► [ API Checker ] ──► DB Query
                                                       │
                                                       ▼
                                                Validates scopes,
                                                tracks usage limits.
```

---

## Visual Diagrams

### API Key Generation and Verification Flow

```text
1. Key Generation (Admin Console)
   [ Admin User ] ──► [ Generate Key ] ──► Hash(Key) ──► Saved in DB
                            │
                            ▼
                     [ API Key string ] (Shown once: ak_live_xyz123)

2. Key Verification (API Request)
   [ External App ] ──► GET /api/data ──► Hash(Key) == DB Hash ──► Success
                        X-API-Key: ak_live_xyz123
```

---

## Under The Hood
API keys must not be stored in plaintext on the database. Secure systems hash them before saving, matching the password hashing strategy (though faster algorithms like SHA-256 can be used since the keys themselves contain high entropy).

```python
# API Key Verification
received_key = request.headers.get("X-API-Key")
hashed_key = sha256_hash(received_key)
client_record = DB.find_by_hash(hashed_key)
```

---

## Packet Journey
Passing an API key in custom headers:

```http
GET /v1/charges HTTP/1.1
Host: api.stripe.com
Authorization: Bearer sk_live_51G...
```

---

## Kernel Perspective
API keys are processed in the network buffers of the kernel before being copied to user-space web server memory. If local system monitoring tool like `tcpdump` runs without root constraints, raw interface packets can expose the keys.

---

## Observe It Yourself
View Stripe's open API key format rules: Stripe keys use prefixes like `pk_live_` to distinguish public keys from secret keys.

---

## Common Misconceptions
> **Myth**: "It is safe to include API keys in client-side mobile applications or JavaScript code."
> 
> **Reality**: No. Any client-side application can be decompiled or inspected using browser debugging tools, exposing the API keys instantly.

---

## Attack Scenarios
* **GitHub Scanning Exploits**: Attackers run automated crawlers on public code repositories, searching for regex patterns matching API key layouts. If a key is committed, it is exploited within seconds.

---

## Defenses
* **Hashed Storage**: Store only the SHA-256 hash of the API key, showing the plaintext key to the user only once during creation.
* **IP Whitelisting**: Restrict key usage to specific client IP ranges.

---

## Tradeoffs
* **Static Nature**: API keys do not expire automatically, meaning they remain active until manually rotated.

---

## Related Concepts
* **Key Rotation**: Changing keys periodically to limit exposure.

---

## Deep Dive
Stripe's API Key design principles:
* Prefix: Identifies the key type and system (e.g. `sk_test_`).
* Random bytes: 24-48 bytes of random data.
* Checksum: Dynamic characters at the end of the key to validate structure before hitting database queries.

---

## Case Studies
* **The AWS S3 Key Leaks (Ongoing)**: Hundreds of organizations leak AWS credentials (access keys and secret keys) in public code repositories, leading to unauthorized resource creation and data theft.

---

## Mini Projects
* **API Key Generator & Hasher**: Write a Python script that generates a cryptographically secure key containing a prefix, random string, and signature checksum, returning the key to the user and saving the SHA-256 hash to a CSV file.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 14: Basic Authentication

## Why Does This Exist?
Early web specifications needed a simple, built-in mechanism to restrict access to directories without building custom login forms or session database structures.
**Basic Authentication** is the native HTTP protocol mechanism for transmitting username/password credentials.

---

## Ground Reality
When a server receives a request for a protected path without credentials, it returns an HTTP `401 Unauthorized` response with a `WWW-Authenticate: Basic` header. The browser intercepts this response and displays a native username/password prompt. Once entered, the browser encodes them in Base64 and appends them to every subsequent request in the `Authorization` header.

---

## Mental Model
Think of Basic Auth as a **door lock with a sign asking for your name and passcode**. The visitor writes their credentials on a index card, clips it to the outside of the door, and walks inside. Anyone passing by the door can read the card.

```text
Visitor ──► [ Read-Only Room ] ──► Shows 401 Challenge ──► Visitor inputs credentials
                                                                │
                                                                ▼
                                                       Transmits in Base64
                                                       on every single request
```

---

## Visual Diagrams

### Basic Authentication Protocol Challenge

```text
Client Browser                                    Server Gateway
┌──────────────────────┐                         ┌──────────────────────┐
│ GET /secure          │ ───────────────────────►│ Determines protected │
│                      │◄───────────────────────│ path.                │
│                      │     401 Unauthorized    │                      │
│                      │  WWW-Authenticate: Basic│                      │
├──────────────────────┤                         ├──────────────────────┤
│ Prompts User.        │                         │                      │
│ Base64("user:pass")  │                         │                      │
│                      │                         │                      │
│ GET /secure          │ ───────────────────────►│ Decodes Base64.      │
│ Authorization: Basic │                         │ Checks DB.           │
│ dXNlcjpwYXNz         │◄────────────────────────│ - Success: 200 OK    │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
Basic authentication header parsing:

```python
# Parsing Basic Authentication Header
auth_header = request.headers.get("Authorization")
if auth_header and auth_header.startswith("Basic "):
    encoded_credentials = auth_header.split(" ")[1]
    decoded = base64.b64decode(encoded_credentials).decode("utf-8")
    username, password = decoded.split(":", 1)
```

---

## Packet Journey
HTTP raw request headers showing Basic authentication format:

```http
GET /admin HTTP/1.1
Host: local.router
Authorization: Basic YWRtaW46cGFzc3dvcmQxMjM=
```

---

## Kernel Perspective
Basic authentication credentials exist in cleartext inside the application's user-space heap. Memory analysis tools (`gcore`) running with process access can dump user-space memory and extract passwords.

---

## Observe It Yourself
Base64 encode a credential string to see the Basic Authentication layout:
```bash
echo -n "admin:password123" | base64
```
Output:
```text
YWRtaW46cGFzc3dvcmQxMjM=
```

---

## Common Misconceptions
> **Myth**: "Basic authentication is secure if the credentials are Base64 encoded."
> 
> **Reality**: No. Base64 is not encryption. It can be decoded instantly by anyone.

---

## Attack Scenarios
* **Plaintext Interception**: If Basic authentication is run over unencrypted HTTP (port 80), an attacker on the same local network (e.g. public Wi-Fi) can sniff the network packets and extract the credentials.

---

## Defenses
* **Force TLS**: Never accept Basic Authentication requests over unencrypted connections.
* **Deprecation**: Replace Basic Authentication with session or token models where possible.

---

## Tradeoffs
* **Usability**: Extremely simple to set up, but lacks support for multi-factor authentication or custom login flows.

---

## Related Concepts
* **Digest Authentication**: A challenge-response hashing protocol designed to avoid transmitting plaintext passwords over Basic Auth.

---

## Deep Dive
RFC 7617: The 'Basic' HTTP Authentication Scheme.
Specifies that the credentials format is `username:password` and must be encoded in US-ASCII or UTF-8.

---

## Case Studies
* **Home Router Compromises**: Many legacy consumer network routers expose their administration consoles using Basic Authentication over HTTP. Attackers gain control of these devices using automated credential scans.

---

## Mini Projects
* **Basic Auth Gateway**: Write a C script using the libcurl library that requests a web page and handles basic authentication challenges dynamically.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 15: HMAC Authentication

## Why Does This Exist?
Bearer tokens, API keys, and Basic Authentication headers transmit static credentials over the network on every request. If an attacker intercepts these credentials, they gain unlimited access. Furthermore, they can modify request data (like changing a payment destination) without the server detecting the modification.
**HMAC Authentication** solves this by signing every individual request dynamically using a secret key, ensuring both sender verification and message integrity.

---

## Ground Reality
Under HMAC authentication, the client and server share a secret key.
To send a request, the client constructs a string containing the request details (timestamp, HTTP method, path, and request body) and computes an HMAC signature using the key. The client sends the signature and metadata in the request headers. The server constructs the same string locally, hashes it with the shared key, and compares signatures.

---

## Mental Model
Think of HMAC authentication as a **notarized letter**:
1. You write a letter containing the request details.
2. You place a seal (HMAC) on the envelope using your private signet ring (secret key).
3. The receiver verifies the seal matches your signet ring.
If an attacker intercepts the letter and changes the request details, the seal becomes invalid, and the receiver rejects the message.

```text
Request: GET /api/v1/charge?amount=100
HMAC signature: 9f2c... (computed with secret key)

Attacker modifies request: GET /api/v1/charge?amount=9000
Signature 9f2c... does not match Hash(9000 + secret key)
Server rejects!
```

---

## Visual Diagrams

### HMAC Request Signing and Verification Flow

```text
Client Application                                Server API Gateway
┌──────────────────────┐                         ┌──────────────────────┐
│ Request Data:        │                         │ Receives Request:    │
│ "POST\n/pay\n100"    │                         │ "POST\n/pay\n100"    │
├──────────────────────┤                         ├──────────────────────┤
│ Compute HMAC-SHA256: │                         │ Pulls Shared Key.    │
│ H = HMAC(Key, Data)  │                         │ Computes expected    │
│                      │                         │ signature:           │
│ Send POST /pay       │ ───────────────────────►│ H' = HMAC(Key, Data) │
│ X-Signature: H       │                         │                      │
│                      │◄────────────────────────│ Compare H == H'      │
│                      │      200 OK (Success)   │ - Match: Authorized  │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
HMAC signature generation and timestamp-replay verification:

```python
# HMAC Verification Logic
received_sig = request.headers.get("X-Signature")
timestamp = request.headers.get("X-Timestamp")

# Prevent Replay Attacks
if abs(time.time() - int(timestamp)) > 300: # 5 minute window
    raise InvalidRequestException("Request expired.")

# Reconstruct message signing string
msg = f"{request.method}\n{request.path}\n{timestamp}\n{request.body}"
expected_sig = hmac.new(
    shared_secret_key, msg.encode(), hashlib.sha256
).hexdigest()

if not hmac.compare_digest(received_sig, expected_sig):
    raise InvalidSignatureException("Signature mismatch.")
```

---

## Packet Journey
HMAC headers passed inside a secure API request packet:

```http
POST /v1/transactions HTTP/1.1
Host: payment-gateway.com
X-Signature: 8f2c3a4b9d8e7f6c5b4a3...
X-Timestamp: 1774902120
Content-Type: application/json

{"amount": 100, "currency": "USD"}
```

---

## Kernel Perspective
HMAC validation involves CPU-bound cryptographic operations (SHA-256). Servers handling high volumes of HMAC-signed network packets experience increased CPU core utilization.

---

## Observe It Yourself
Compute an HMAC-SHA256 signature using openssl:
```bash
echo -n "POST/v1/transactions1774902120" | openssl dgst -sha256 -hmac "mysecretkey"
```
Output:
```text
(stdin)= 3d2a7bc128e0182fc89d0b009e25...
```

---

## Common Misconceptions
> **Myth**: "HMAC authentication makes HTTPS unnecessary."
> 
> **Reality**: No. While HMAC protects integrity and validates identity, the request data itself is still transmitted in plaintext. You still need HTTPS to ensure confidentiality.

---

## Attack Scenarios
* **Replay Attacks**: An attacker intercepts a valid signed request and sends it again to the server. The server verifies the signature matches the message and executes the action a second time.

---

## Defenses
* **Timestamps and Nonces**: Include a timestamp inside the signed message and reject requests with timestamps older than a configured limit (e.g. 5 minutes).

---

## Tradeoffs
* **Complexity**: Requires client applications to implement custom cryptographic signing code, making API integration more difficult.

---

## Related Concepts
* **Digital Signatures**: Asymmetric signing mechanisms using public/private key pairs.

---

## Deep Dive
Why `hmac.compare_digest` is used instead of standard string comparison (`==`):
Standard comparison functions exit early when they detect a mismatch, causing execution time variations. Attackers can measure these timing differences down to microseconds to guess the signature byte-by-byte (timing attack). `compare_digest` checks the entire string length regardless of mismatch locations, taking a constant execution time.

---

## Case Studies
* **AWS Signature Version 4**: AWS uses a complex HMAC-based signing protocol (SigV4) to authenticate API requests to services like S3 and EC2, validating both sender identity and request integrity.

---

* **HMAC Request Signer & Verifier**: Write a Python client-server simulator where the client signs requests containing timestamps and body payloads, and the server validates them using constant-time comparisons.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 16: Challenge Response Systems

## Why Does This Exist?
Even if you hash or sign credentials, transmitting any form of a static secret across the network exposes it to sniffing or interception.
**Challenge-Response Systems** verify that a client possesses a secret key *without* ever transmitting the secret itself across the network.

---

## Ground Reality
Under a challenge-response protocol:
1. The client requests authentication.
2. The server generates a unique, single-use random value (a **nonce** or **challenge**) and sends it to the client.
3. The client combines the challenge with their secret key, hashes or signs the combined data, and returns the result (the **response**).
4. The server performs the same computation locally and verifies the response.

---

## Mental Model
Think of a security guard identifying a resident without asking for their password:
1. The guard yells a random phrase: "Blue Monkey!" (Challenge).
2. The resident, knowing the secret handshake (Secret Key), performs a specific rotation of their hands and shouts back: "Yellow Tiger!" (Response).
3. An eavesdropper hears "Blue Monkey" and "Yellow Tiger," but cannot use this information to pass the next day because the guard will yell a different phrase, requiring a different response.

```text
Server (Guard)                                    Client (Resident)
    │                                                     │
    │────────────── Challenge: "Blue Monkey" ────────────►│ (Computes response
    │                                                     │  using secret key)
    │◄───────────── Response: "Yellow Tiger" ─────────────│
```

---

## Visual Diagrams

### Challenge-Response Nonce Loop

```text
Client Application                                Authentication Server
┌──────────────────────┐                         ┌──────────────────────┐
│ Initiate Session ────┼────────────────────────►│ Generates random     │
│                      │◄────────────────────────│ Nonce (Challenge)    │
│                      │    Challenge: c189d     │                      │
├──────────────────────┤                         ├──────────────────────┤
│ Computes Response:   │                         │                      │
│ R = SHA256(c189d + K)│                         │                      │
│                      │                         │                      │
│ Send Response: R ────┼────────────────────────►│ Computes expected:   │
│                      │◄────────────────────────│ R' = SHA256(c189d + K)│
│                      │      Authentication     │ Compare R == R'      │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
A typical challenge-response generation function in pseudo-code:

```python
# Server challenge validation logic
def verify_response(received_response, expected_nonce, user_secret):
    expected_response = hash_function(expected_nonce + user_secret)
    return constant_time_compare(received_response, expected_response)
```

---

## Packet Journey
Tracing challenge-response exchanges over raw socket connections:

```text
1. Client -> Server: AUTH_INIT
2. Server -> Client: CHALLENGE 0x9f8c12a7
3. Client -> Server: RESPONSE 0x2b8e3a... (HMAC(0x9f8c12a7, secret_key))
```

---

## Kernel Perspective
Cryptographic operations involved in generating nonces utilize `/dev/urandom` reads via kernel system calls. The network cards receive packets containing challenge hashes, copying them directly to socket memory space without kernel interference.

---

## Observe It Yourself
See how a challenge-response structure is used in chap authentication files:
```bash
cat /etc/ppp/chap-secrets 2>/dev/null || echo "CHAP secrets requires root access to view."
```

---

## Common Misconceptions
> **Myth**: "Nonces can be sequential numbers like 1, 2, 3."
> 
> **Reality**: No. If nonces are predictable, an attacker can precompute responses for future values. Nonces must be cryptographically random.

---

## Attack Scenarios
* **Reflection Attack**: In symmetric challenge-response networks where both nodes authenticate each other, an attacker opens two connections to a victim and uses the challenge received on connection 2 to answer the challenge on connection 1.

---

## Defenses
* **CSPRNG Nonces**: Always generate challenges using secure system entropy.
* **Mutual Authentication**: Force both client and server to solve challenges to verify each other's identity.

---

## Tradeoffs
* **Network Roundtrips**: Requires multiple packet roundtrips (Initiate -> Challenge -> Response -> Success), introducing latency.

---

## Related Concepts
* **CHAP**: Challenge Handshake Authentication Protocol.

---

## Deep Dive
How CHAP (RFC 1994) operates over PPP links:
* The authenticator sends a Challenge packet containing an ID, a random challenge string, and a directory name.
* The peer responds with a hash computed over the ID, the secret key, and the challenge value.

---

## Case Studies
* **GSM Authentication (Sim Cards)**: Cellular networks use challenge-response authentication. The base station sends a 128-bit random number (RAND) to the SIM card, which computes a response signed with the card's internal secret key (Ki).

---

## Mini Projects
* **Socket Challenge-Response Server**: Write a python program where a client connects via TCP sockets, solves a random cryptographic challenge, and receives a flag.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 17: Public Key Cryptography

## Why Does This Exist?
Symmetric cryptography requires both client and server to share the exact same secret key. If a server authenticates millions of users, it must store millions of secret keys. If the server's database is compromised, all user keys are exposed.
**Public Key Cryptography** solves this by splitting the key into a public key (used to encrypt or verify) and a private key (used to decrypt or sign).

---

## Ground Reality
Public and private keys are mathematically linked pairs.
* **Public Key**: Shared openly. Anyone can use it to encrypt messages or verify signatures.
* **Private Key**: Kept secret by the owner. Only the owner can use it to decrypt messages or generate signatures.
It is computationally impossible to derive the private key from the public key.

---

## Mental Model
Think of public-key cryptography as a **post office mailbox**:
* **Public Key**: The mailbox's physical slot. Anyone walking by can drop a letter inside the slot.
* **Private Key**: The mailbox key held by the postmaster. Only the postmaster can unlock the box and read the letters.

```text
Message ──► [ Encrypt with Public Key ] ──► Ciphertext ──► [ Decrypt with Private Key ] ──► Plaintext
```

---

## Visual Diagrams

### Asymmetric Encryption Architecture

```text
Alice (Sender)                                    Bob (Receiver)
┌──────────────────────┐                         ┌──────────────────────┐
│ Plaintext: "Secret"  │                         │ Holds Private Key    │
├──────────────────────┤                         ├──────────────────────┤
│ Pulls Bob's Public   │                         │                      │
│ Key from network.    │                         │                      │
│ Cipher = RSA_Enc(    │                         │                      │
│   "Secret", Bob_Pub  │                         │                      │
│ )                    │                         │                      │
│ Send Cipher ─────────┼────────────────────────►│ Decrypts:            │
│                      │                         │ Msg = RSA_Dec(       │
│                      │                         │   Cipher, Bob_Priv   │
└──────────────────────┘                         └──────────────────────┘
```

---

## Under The Hood
Basic RSA math operations in code:

```python
# RSA Math Foundation
# Private key: (d, n), Public key: (e, n)
def encrypt(plaintext_int, public_key):
    e, n = public_key
    return pow(plaintext_int, e, n) # C = M^e mod n

def decrypt(ciphertext_int, private_key):
    d, n = private_key
    return pow(ciphertext_int, d, n) # M = C^d mod n
```

---

## Packet Journey
Passing public keys over network layers:

```http
GET /keys/bob.pub HTTP/1.1
Host: keyserver.pgp.net

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz6l2...
-----END PUBLIC KEY-----
```

---

## Kernel Perspective
Asymmetric cryptography is highly CPU intensive. Linux offloads RSA and Elliptic Curve operations to hardware acceleration structures via the kernel cryptographic driver framework (`crypto/asymmetric_keys`).

---

## Observe It Yourself
Generate a new RSA public/private key pair using openssl:
```bash
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl pkey -in private_key.pem -pubout -out public_key.pem
```

---

## Common Misconceptions
> **Myth**: "Public key cryptography makes symmetric encryption obsolete."
> 
> **Reality**: No. Asymmetric encryption is thousands of times slower than symmetric encryption (AES). Secure systems use asymmetric keys only to exchange a symmetric key (hybrid cryptography).

---

## Attack Scenarios
* **Man-In-The-Middle (MITM)**: An attacker intercepts the public key exchange and substitutes their own public key. The sender encrypts messages using the attacker's key, allowing the attacker to decrypt, read, and re-encrypt the data before sending it to the recipient.

---

## Defenses
* **Public Key Infrastructure (PKI)**: Binding public keys to verified identities using digital certificates signed by trusted third parties.

---

## Tradeoffs
* **Performance**: Heavy mathematical overhead (modular exponentiation of large integers).

---

## Related Concepts
* **ECDSA**: Elliptic Curve Digital Signature Algorithm (faster than RSA with smaller keys).

---

## Deep Dive
The math behind RSA:
* Choose two large prime numbers $p$ and $q$.
* Compute $n = p \times q$.
* Compute the totient $\phi(n) = (p - 1)(q - 1)$.
* Choose an integer $e$ such that $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$ (usually $65537$).
* Compute $d$ such that $d \times e \equiv 1 \pmod{\phi(n)}$.

---

## Case Studies
* **The Dual_EC_DRBG Backdoor (2013)**: The NSA allegedly placed a backdoor in an elliptic curve random number generator standard, allowing them to predict generated public/private keys and decrypt traffic.

---

## Mini Projects
* **RSA Math Simulator**: Write a Python script that generates small key pairs (using small prime numbers), encrypts a numerical message, and decrypts it back.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 18: Digital Signatures

## Why Does This Exist?
Public key cryptography allows us to encrypt data securely. However, encrypting a message does not prove *who* sent it.
**Digital Signatures** solve this by providing proof of authorship (non-repudiation) and message integrity.

---

## Ground Reality
A digital signature is computed by hashing the message and encrypting the resulting hash using the sender's **private key**. Anyone can verify the signature by decrypting it with the sender's **public key** and comparing the result to the local hash of the message.

```text
Sign:   Hash(Message) ──► Encrypt with Private Key ──► Signature
Verify: Decrypt Signature with Public Key ──► Original Hash == Hash(Message)
```

---

## Mental Model
Think of a digital signature as a **wax seal stamped with a custom ring**:
* The ring (Private Key) belongs only to the king.
* Anyone can compare the seal to the king's public coin design (Public Key) to verify authenticity.
* If a competitor attempts to modify the document, the wax breaks, invalidating the seal.

---

## Visual Diagrams

### Digital Signature Generation and Verification Pipeline

```text
Signing Message
[ Plaintext Message ] ──► [ Hash Function ] ──► Hash Digest
                                                  │
                                                  ▼
[ Private Key ] ────────► [ Signature Engine ] ──► Digital Signature

Verifying Message
[ Plaintext Message ] ──► [ Hash Function ] ──► Hash Digest (A)
                                                  │
                                                  ▼
                                            [ Compare A == B ] ──► Valid!
                                                  ▲
                                                  │
[ Digital Signature ] ──► [ Decrypt with Pub ] ──► Decrypted Hash (B)
```

---

## Under The Hood
A signature creation utility script block:

```python
# RSA Signature Generation pseudo-code
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding

def sign_message(message_bytes, private_key):
    return private_key.sign(
        message_bytes,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
```

---

## Packet Journey
A JSON payload accompanied by a signature in API headers:

```http
POST /api/v1/webhook HTTP/1.1
Host: receiver.com
X-Signature: MC0CFQCf8a9c...
Content-Type: application/json

{"event": "payment.success", "id": 9912}
```

---

## Kernel Perspective
Linux verifies digital signatures during system boot to enforce kernel module loading constraints (`CONFIG_MODULE_SIG`). The kernel refuses to load modules unless signed by a key built into the kernel binary.

---

## Observe It Yourself
Inspect signature details of a kernel module in Linux:
```bash
modinfo -F sig_key ext4
```

---

## Common Misconceptions
> **Myth**: "Digital signatures encrypt the message payload."
> 
> **Reality**: No. The message is sent in plaintext alongside the signature. The signature only proves authenticity and integrity, not confidentiality.

---

## Attack Scenarios
* **Key Reuse Attacks**: If a sender uses the same key pair for decryption and signature generation, an attacker can trick the sender into signing a hash that represents an encrypted message, leaking the plaintext.

---

## Defenses
* **Key Isolation**: Use separate keys for encryption/decryption and signing/verification.

---

## Tradeoffs
* **Storage Overhead**: Adding signatures to every document or database record increases the storage footprint.

---

## Related Concepts
* **DSA**: Digital Signature Algorithm.

---

## Deep Dive
ECDSA signatures (Elliptic Curve Digital Signature Algorithm):
ECDSA produces two integers, $r$ and $s$, representing the signature. The calculations rely on elliptic curve points over a finite field, yielding smaller signature sizes than RSA with equivalent cryptographic strength.

---

## Case Studies
* **Sony PlayStation 3 Code Signing Bug (2010)**: Sony's implementation of ECDSA used a constant value instead of a random number for $k$ during signature generation. This allowed hackers to calculate the private key and sign custom firmware modules.

---

## Mini Projects
* **ECDSA Document Signer**: Write a Python utility that reads a file, signs its contents using ECDSA, and writes the signature to a separate validation file.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 19: Certificates and PKI

## Why Does This Exist?
If public key cryptography requires verifying identities using public keys, how do you verify that a public key actually belongs to the target website (e.g. `google.com`) and not an attacker?
**Certificates** and **Public Key Infrastructure (PKI)** solve this by binding public keys to identities using trusted third-party verifiers called **Certificate Authorities (CAs)**.

---

## Ground Reality
A **Digital Certificate** (X.509 standard) contains:
* The website's identity (domain name).
* The website's public key.
* The digital signature of the Certificate Authority that verified the website.
Operating systems and web browsers ship with a built-in list of trusted root CAs.

---

## Mental Model
Think of a digital certificate as a **government-issued passport**:
* The passport contains your photo (Public Key) and details (Domain).
* It is stamped by the government passport office (Certificate Authority).
* The border agent (Browser) trusts the stamp because they recognize the government's official seal (Root Certificate).

```text
[ Website Pub Key ] + [ Domain Name ] ──► [ CA signs with Priv Key ] ──► Digital Certificate
```

---

## Visual Diagrams

### Trust Chain Architecture

```text
[ Root CA Certificate ] (Built into browser/OS trust store)
         │
         ▼ (Signs)
[ Intermediate CA Certificate ]
         │
         ▼ (Signs)
[ Leaf / Website Certificate ] (e.g., github.com)
```

---

## Under The Hood
The structure of an X.509 certificate representation:

```text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 04:a8:b9:2f...
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=DigiCert Inc, CN=DigiCert SHA2 Secure Server CA
        Validity:
            Not Before: May 12 00:00:00 2026 GMT
            Not After : May 12 12:00:00 2027 GMT
        Subject: C=US, ST=California, L=San Francisco, O=GitHub, Inc., CN=github.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
            Public-Key: (256 bit)
```

---

## Packet Journey
During a TLS handshake, the server sends its certificate chain to the client:

```text
Handshake protocol packet:
- TLS Record Layer: Handshake Protocol: Certificate
- Certificate Length: 3120 bytes
- Certificates:
    - Certificate: CN=github.com
    - Certificate: CN=DigiCert SHA2 Secure Server CA
```

---

## Kernel Perspective
Web browsers and applications load trusted root certificate files (typically PEM formatted strings stored in `/etc/ssl/certs` or `/etc/pki/tls`) into memory. The kernel exposes these files via standard filesystem APIs.

---

## Observe It Yourself
View the certificate chain of a website using openssl:
```bash
openssl s_client -connect github.com:443 -showcerts
```

---

## Common Misconceptions
> **Myth**: "Private Certificate Authorities are only for local development."
> 
> **Reality**: Large enterprises use private CAs internally to secure millions of microservice connections, bypassing public internet registration systems.

---

## Attack Scenarios
* **CA Compromise**: An attacker hacks a Certificate Authority and issues a fake certificate for `google.com`. They then route user traffic to a fake server that decrypts connections using the forged certificate.

---

## Defenses
* **Certificate Transparency (CT) Logs**: Public, append-only ledgers where all issued certificates must be logged. Browsers reject certificates that are not in CT logs.
* **Certificate Revocation Lists (CRL)** and **OCSP Stapling**: Checking if a certificate has been revoked before its expiration date.

---

## Tradeoffs
* **Centralization**: Relying on a small group of global companies (CAs) to maintain internet trust.

---

## Related Concepts
* **ACME Protocol**: Automated Certificate Management Environment (used by Let's Encrypt to automate renewals).

---

## Deep Dive
Certificate Path Validation algorithm:
The client verifies the signature of the leaf certificate using the intermediate public key. It then verifies the intermediate certificate's signature using the root public key. This process continues until a certificate in the chain matches a trusted certificate in the client's local root store.

---

## Case Studies
* **The DigiNotar Hack (2011)**: A Dutch CA was compromised, allowing attackers to issue hundreds of rogue certificates (including for Google domains). Browsers revoked trust in DigiNotar completely, leading to the company's bankruptcy.

---

## Mini Projects
* **Private CA Signer**: Write a bash script that generates a Root CA certificate, signs a CSR (Certificate Signing Request) for a local domain, and generates a valid leaf certificate.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 20: TLS Handshake

## Why Does This Exist?
Even if we have certificates, we cannot use asymmetric cryptography to encrypt the entire network session because it is too slow.
The **TLS Handshake** protocol solves this by authenticating the server using certificates and negotiating a transient symmetric key used to encrypt all session data.

---

## Ground Reality
The TLS 1.3 handshake completes in a single network roundtrip (1 RTT):
1. **ClientHello**: Client sends supported cryptographic suites and key-share configurations.
2. **ServerHello**: Server chooses the suite, sends its public key share, certificate chain, and signature.
3. Both sides calculate the shared session key locally using Diffie-Hellman, and switch to symmetric encryption (AES-GCM or ChaCha20-Poly1305).

---

## Mental Model
Think of two people negotiating a secure conversation code:
1. Alice shouts: "I speak French and Spanish, here is my mixing base: 5."
2. Bob responds: "Let's speak Spanish. Here is my mixing base: 7. Also, here is my ID card signed by the mayor."
3. Both mix the bases secretly with their own private numbers to generate the exact same shared wordcode. They then speak using the shared wordcode.

```text
Alice (Client)                                    Bob (Server)
    │                                                     │
    │─── ClientHello (Supported suites, Key Share X) ────►│
    │                                                     │─── Verifies identity,
    │◄── ServerHello (Suite choice, Certificate, Key Share Y)│  calculates key.
    │                                                     │
    │ (Verifies certificate, calculates key)              │
```

---

## Visual Diagrams

### TLS 1.3 Handshake Packet Protocol

```text
Client                                              Server
  │                                                   │
  │─── ClientHello ──────────────────────────────────►│
  │    - Supported Cipher Suites                      │
  │    - Key Share (Client DH Share)                  │
  │                                                   │
  │◄── ServerHello ───────────────────────────────────│
  │    - Chosen Cipher Suite                          │
  │    - Key Share (Server DH Share)                  │
  │    - Encrypted Extensions                         │
  │    - Certificate Chain                            │
  │    - Certificate Verify (Signature)               │
  │    - Finished message                             │
  │                                                   │
  ├───────────────────────────────────────────────────┤
  │       All subsequent application data is          │
  │       encrypted using the negotiated symmetric    │
  │       session key.                                │
  └───────────────────────────────────────────────────┘
```

---

## Under The Hood
Key calculation logic using Diffie-Hellman math:

```python
# Diffie-Hellman Key Exchange Foundation
# g (generator), p (prime modulus)
g = 5
p = 23

# Client private key: a
a = 6
A = pow(g, a, p) # Client Share: 5^6 mod 23 = 8

# Server private key: b
b = 15
B = pow(g, b, p) # Server Share: 5^15 mod 23 = 19

# Exchange shares, compute shared secret:
client_secret = pow(B, a, p) # 19^6 mod 23 = 2
server_secret = pow(A, b, p) # 8^15 mod 23 = 2
```

---

## Packet Journey
An actual Wireshark capture analysis snippet of TLS Handshake packets:

```text
No.  Time       Source         Destination    Protocol Info
1    0.0000     192.168.1.10   104.16.249.2   TLSv1.3  Client Hello
2    0.0152     104.16.249.2   192.168.1.10   TLSv1.3  Server Hello, Change Cipher Spec, Handshake Protocol
```

---

## Kernel Perspective
Modern Linux supports Kernel TLS (`kTLS`). Once the user-space application completes the handshake, it hands the symmetric session keys to the kernel using `setsockopt()`, allowing the kernel to encrypt and decrypt packet data directly inside the network stack, avoiding memory copies.

---

## Observe It Yourself
Trace a TLS 1.3 connection handshake in your terminal using curl with verbose output:
```bash
curl -vI https://github.com
```

---

## Common Misconceptions
> **Myth**: "TLS 1.3 and TLS 1.2 handshakes are the same."
> 
> **Reality**: TLS 1.2 requires two network roundtrips (2 RTT) and supports old, weak cryptographic configurations. TLS 1.3 requires only one roundtrip (1 RTT) and deprecates insecure algorithms.

---

## Attack Scenarios
* **Downgrade Attack (Logjam/FREAK)**: An attacker intercepts the handshake and modifies the list of supported ciphers, forcing the client and server to use an older, cryptographically broken protocol suite (like export-grade RSA) to decrypt the session.

---

## Defenses
* **Disable TLS 1.0 and 1.1**: Enforce a minimum TLS version of 1.2, preferably 1.3, in server configurations.
* **HSTS (HTTP Strict Transport Security)**: A header telling browsers to never connect to the site over unencrypted HTTP.

---

## Tradeoffs
* **Handshake Latency**: First connections require network roundtrips before any page content can load.

---

## Related Concepts
* **ALPN**: Application-Layer Protocol Negotiation (negotiating protocols like HTTP/2 during the handshake).

---

## Deep Dive
Diffie-Hellman Ephemeral (DHE):
"Ephemeral" means a new, random key pair is generated for every session. If the server's master private key is stolen in the future, attackers cannot decrypt past sessions (Forward Secrecy).

---

## Case Studies
* **The Heartbleed Bug (2014)**: A vulnerability in OpenSSL's heartbeat extension allowed attackers to read random blocks of server memory, exposing active session keys and private certificates.

---

* **Python TLS Client Socket**: Write a Python client that opens a raw TCP socket, wraps it in a TLS context using standard SSL libraries, and sends a secure HTTP request.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 21: OAuth2

## Why Does This Exist?
Before OAuth2, if you wanted a third-party app (e.g. a photo printer) to access your photos on a service (e.g. Google Photos), you had to give the app your raw username and password. The app could then access *everything* in your account and change your password.
**OAuth2** solves this by providing a delegation framework, allowing you to grant restricted access tokens to third-party applications without sharing your credentials.

---

## Ground Reality
OAuth2 is an authorization delegation protocol.
It defines four roles:
1. **Resource Owner**: The user granting access.
2. **Client**: The third-party application requesting access.
3. **Authorization Server**: The system verifying credentials and issuing tokens.
4. **Resource Server**: The API hosting the user's data.
Instead of credentials, the client uses an **Access Token** scoped to specific permissions (e.g., `read:photos`).

---

## Mental Model
Think of valet parking at a hotel:
* You do not give the valet your master key ring (Username/Password).
* You hand them a restricted **valet key** (Access Token).
* The valet key allows them to start the engine and drive 50 yards, but prevents them from opening the trunk or glove box.

```text
User ──► [ Grants Consent ] ──► [ Auth Server ] ──► Valet Key (Token) ──► [ Valet App ] ──► Accesses Car
```

---

## Visual Diagrams

### OAuth2 Authorization Code Flow

```text
User Browser             Client Web App           Auth Server           API Resource Server
  │                            │                      │                         │
  │─── Click "Login with O" ──►│                      │                         │
  │◄── Redirect to Auth Server─│                      │                         │
  │                            │                      │                         │
  │─── Input Credentials ────────────────────────────►│                         │
  │◄── Consent Screen ───────────────────────────────│                         │
  │─── Grants Consent ──────────────────────────────►│                         │
  │◄── Redirect with Auth Code ───────────────────────│                         │
  │                                                   │                         │
  │─── Send Auth Code ────────►│                      │                         │
  │                            │─── Code + Secret ───►│                         │
  │                            │◄── Access Token ─────│                         │
  │                            │                                                │
  │                            │─── Request Data + Access Token ───────────────►│
  │                            │◄── Private Data ───────────────────────────────│
```

---

## Under The Hood
Exchanging an authorization code for an access token:

```python
# Server token exchange logic
def exchange_code_for_token(client_id, client_secret, code):
    client_record = DB.find_client(client_id)
    if client_record.secret != client_secret:
        raise InvalidClientException("Invalid secret key.")
    
    token_record = DB.find_auth_code(code)
    if token_record.is_expired() or token_record.client_id != client_id:
        raise InvalidCodeException("Authorization code expired or invalid.")
        
    access_token = generate_secure_token()
    DB.save_access_token(access_token, token_record.user_id, token_record.scopes)
    return {"access_token": access_token, "token_type": "Bearer", "expires_in": 3600}
```

---

## Packet Journey
A token exchange POST request packet sent from client to server:

```http
POST /oauth/token HTTP/1.1
Host: identity.provider.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA&redirect_uri=https%3A%2F%2Fclient.example.com%2Fcb&client_id=client123&client_secret=supersecretclientkey
```

---

## Kernel Perspective
OAuth2 exchanges occur entirely within HTTP headers and payloads processed by user-space web services. The kernel handles socket read/write operations for TCP data packets without distinguishing OAuth parameters.

---

## Observe It Yourself
Query an OAuth provider's discovery endpoint to find supported scopes and paths:
```bash
curl -s https://accounts.google.com/.well-known/openid-configuration | grep token_endpoint
```

---

## Common Misconceptions
> **Myth**: "OAuth2 is an authentication protocol."
> 
> **Reality**: No. OAuth2 handles authorization (delegated access). It does not prove the identity of the user to the client application. To handle identity, you must use OpenID Connect (OIDC).

---

## Attack Scenarios
* **Authorization Code Interception**: An attacker intercepts the redirection URL containing the authorization code (e.g. by exploiting custom URL schemes on mobile operating systems) and exchanges the code for an access token.

---

## Defenses
* **PKCE (Proof Key for Code Exchange)**: Enforcing dynamic code challenge verification parameters to prevent authorization code injection and interception attacks.
* **State Parameter**: Utilizing random state values to mitigate Cross-Site Request Forgery (CSRF) on callback endpoints.

---

## Tradeoffs
* **Complexity**: Multiple redirection loops and token exchanges require extensive client/server integration.

---

## Related Concepts
* **OIDC**: Identity overlay on OAuth2.

---

## Deep Dive
PKCE (RFC 7636) mechanics:
* Client generates a secret `code_verifier`.
* Client hashes it: `code_challenge = Base64URL(SHA-256(code_verifier))`.
* Client sends `code_challenge` during authorization request.
* During the token exchange, the client sends the raw `code_verifier`. The server validates that the hashed verifier matches the challenge received earlier.

---

## Case Studies
* **The Facebook Token Leak (2018)**: A vulnerability in Facebook's "View As" feature exposed OAuth access tokens for 50 million users, allowing attackers to hijack accounts.

---

## Mini Projects
* **OAuth2 Code Client**: Write a script that spins up a local web server to catch the OAuth authorization code redirection callback and prints the parameters.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 22: OpenID Connect

## Why Does This Exist?
OAuth2 is designed to delegate authorization, but it does not specify how the identity provider should share user profile data (like email or name) with the client. Early developers hacked OAuth2 to return user profiles, creating fragmented implementations.
**OpenID Connect (OIDC)** overlay solves this by adding an identity layer on top of OAuth2, providing a standardized format for profile sharing.

---

## Ground Reality
OIDC extends OAuth2 by:
1. Adding the `openid` scope to the authorization request.
2. Introducing an **ID Token** (formatted as a signed JWT) containing user identity claims.
3. Defining a standard `/userinfo` endpoint to fetch user metadata.

---

## Mental Model
Think of OAuth2 as a **corporate security pass** that lets you enter the building (access token). Think of OIDC as an **ID badge card** containing your name, employee number, department, and headshot.

```text
[ OAuth2 Access Token ] ──► (Unlocks Doors)
[ OIDC ID Token ]       ──► (Identifies Bearer: "Alice, HR Department")
```

---

## Visual Diagrams

### OIDC Identity Retrieval Chain

```text
Client Application                                Identity Provider (IDP)
┌──────────────────────┐                         ┌──────────────────────┐
│ Request Auth:        │ ───────────────────────►│ Authenticates User.  │
│ scope=openid email   │                         │ Generates tokens.    │
├──────────────────────┤                         ├──────────────────────┤
│ Receives Response:   │◄────────────────────────│ - Access Token       │
│                      │                         │ - ID Token (JWT)     │
├──────────────────────┤                         └──────────────────────┘
│ Parses ID Token:     │
│ sub: "alice123"      │
│ email: "a@corp.com"  │
└──────────────────────┘
```

---

## Under The Hood
An ID Token payload example:

```json
{
  "iss": "https://server.example.com",
  "sub": "248289761001",
  "aud": "s6BhdRkqt3",
  "exp": 1311281970,
  "iat": 1311280970,
  "name": "Jane Doe",
  "email": "janedoe@example.com"
}
```

---

## Packet Journey
Getting user profile information from the OIDC userinfo endpoint:

```http
GET /oauth/userinfo HTTP/1.1
Host: identity.provider.com
Authorization: Bearer access_token_xyz987
```
Response:
```json
{
  "sub": "248289761001",
  "name": "Jane Doe",
  "email": "janedoe@example.com",
  "picture": "https://example.com/jane.jpg"
}
```

---

## Kernel Perspective
Client processes decrypt and parse signed JWT ID tokens in memory. Cryptographic verification utilizes CPU-level optimizations for signature verification, running inside user-space libraries.

---

## Observe It Yourself
Examine an OIDC provider configuration details:
```bash
curl -s https://accounts.google.com/.well-known/openid-configuration | jq .
```

---

## Common Misconceptions
> **Myth**: "The ID Token should be used to call the backend APIs."
> 
> **Reality**: No. The ID token is strictly for the client application to read user profile info. To authenticate requests to your backend APIs, use the Access Token.

---

## Attack Scenarios
* **ID Token Forgery**: An attacker generates a fake ID token with a signature computed using a weak key and presents it to a client application that skips signature verification, logging in as a targeted administrator.

---

## Defenses
* **Always Verify Signatures**: Verify the ID token signature against the IDP's public keys (retrieved from the JWKS endpoint).
* **Audience Check**: Validate that the `aud` (audience) claim in the token matches your application's `client_id`.

---

## Tradeoffs
* **Network Overhead**: Multiple validation steps and key retrievals can add startup latency to new user sessions.

---

## Related Concepts
* **JWKS**: JSON Web Key Set (a list of public keys used to verify tokens).

---

## Deep Dive
The JWKS (JSON Web Key Set) endpoint:
OIDC servers host their public keys at a standardized path (e.g. `/.well-known/jwks.json`). Client applications fetch and cache these keys to verify JWT signatures, allowing key rotation on the IDP side without breaking client code.

---

## Case Studies
* **Sign in with Apple (2020)**: A vulnerability allowed attackers to bypass authentication by forging an ID token payload containing any email address, as Apple failed to verify the signature of the client verification token correctly.

---

## Mini Projects
* **ID Token Decoder & Validator**: Write a script that fetches public keys from a JWKS URL and uses them to validate a local JWT ID Token.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 23: SSO (Single Sign-On)

## Why Does This Exist?
In an enterprise environment with dozens of separate internal applications, forcing users to log in to each application individually creates login fatigue, weak password reuse, and high credential management costs.
**Single Sign-On (SSO)** solves this by centralizing authentication at a single identity provider, allowing users to log in once and gain access to all applications.

---

## Ground Reality
SSO relies on federation protocols (SAML 2.0 or OIDC):
1. User requests access to an internal app (Service Provider / SP).
2. The SP redirects the user's browser to the Central Identity Provider (IdP).
3. The IdP authenticates the user (or recognizes an active session cookie) and generates an assertion token (SAML Assertion or OIDC ID Token).
4. The IdP redirects the user's browser back to the SP with the token.
5. The SP verifies the token and grants access.

---

## Mental Model
Think of SSO as a **theme park wristband**:
* You go to the front ticket booth (IdP), verify your age (Credentials), and buy a wristband.
* When you want to go on a ride (SP), the ride operator checks the wristband.
* You do not need to buy a ticket or show ID at every individual ride.

```text
User ──► [ Ticket Booth (IdP) ] ──► (Wristband) ──► [ Rollercoaster (SP) ] ──► Access Granted
```

---

## Visual Diagrams

### SAML-Based SSO Redirection Flow

```text
User Browser             Service Provider (App)        Identity Provider (IdP)
  │                                │                                │
  │─── Access resource ───────────►│                                │
  │◄── Redirect (SAML Request) ────│                                │
  │                                                                 │
  │─── Forward SAML Request ───────────────────────────────────────►│
  │◄── Prompt Credentials ──────────────────────────────────────────│
  │─── Input Pass + MFA ───────────────────────────────────────────►│
  │◄── Redirect (SAML Response + Signature) ────────────────────────│
  │                                                                 │
  │─── Forward SAML Response ─────►│                                │
  │◄── Grant Access (Session) ─────│                                │
```

---

## Under The Hood
SAML 2.0 XML assertion token block example:

```xml
<saml2p:Response ID="_d8f9..." Version="2.0" IssueInstant="2026-06-19T02:00:00Z">
  <saml2:Issuer>https://idp.corp.com</saml2:Issuer>
  <saml2:Assertion ID="_a7b8...">
    <saml2:Subject>
      <saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">bob@corp.com</saml2:NameID>
    </saml2:Subject>
    <saml2:Conditions NotBefore="2026-06-19T01:55:00Z" NotOnOrAfter="2026-06-19T02:05:00Z"/>
    <ds:Signature>...</ds:Signature>
  </saml2:Assertion>
</saml2p:Response>
```

---

## Packet Journey
A SAML response payload passed via POST request parameter from user browser:

```http
POST /saml/acs HTTP/1.1
Host: application.internal.com
Content-Type: application/x-www-form-urlencoded

SAMLResponse=PHNhbWwycDpSZXNwb25zZSBJRD0iX2Q4ZjkiLi4u
```

---

## Kernel Perspective
SSO operations use cryptographic operations on HTTPS traffic. The kernel processes these SSL connections via socket file descriptors, offloading payload decryption to user-space tools or `kTLS` drivers.

---

## Observe It Yourself
Inspect SAML assertions using browser extension tools (like SAML Tracer) to view decrypted XML payloads during enterprise login loops.

---

## Common Misconceptions
> **Myth**: "SAML is obsolete because of OIDC."
> 
> **Reality**: While OIDC is simpler and used for modern web/mobile apps, SAML remains the dominant standard in legacy corporate enterprise systems due to active Active Directory and Okta integrations.

---

## Attack Scenarios
* **SAML Signature Bypass**: If a service provider parses the SAML assertion payload without validating the `<ds:Signature>` block, or if it validates the signature but fails to check the assertion XML bounds, an attacker can modify the assertion to log in as another user.

---

## Defenses
* **Validate XML Signatures**: Always verify the certificate signature of the entire SAML assertion XML block.
* **Strict Clock Sync**: Use NTP to synchronize clocks and enforce strict validity bounds (`NotOnOrAfter`).

---

## Tradeoffs
* **Single Point of Failure**: If the Identity Provider is down, users cannot access any integrated application.

---

## Related Concepts
* **Federated Identity**: Extending SSO across organization boundaries.

---

## Deep Dive
SAML Signature XML canonicalization (C14N):
XML formatting is flexible (spaces, attributes order). Before checking digital signatures, XML libraries must run canonicalization algorithms to produce identical byte arrays, ensuring signature integrity checks do not fail due to trivial formatting shifts.

---

## Case Studies
* **The Golden SAML Attack (2020/SolarWinds)**: Attackers stole Active Directory Federation Services (ADFS) private token-signing certificates, allowing them to forge valid SAML assertions and gain access to Office 365 and cloud resources bypassing MFA.

---

## Mini Projects
* **SSO Identity Provider Simulator**: Write a simple Flask app that acts as a mock SAML/OIDC Identity Provider, showing a consent page and returning signed tokens to redirect targets.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 24: LDAP (Lightweight Directory Access Protocol)

## Why Does This Exist?
In an enterprise, managers need a centralized catalog to organize network resources: users, groups, devices, printers, and file shares. Storing authentication records in separate application databases makes unified employee onboarding and access control impossible.
**LDAP** provides a protocol and directory service to store, search, and authenticate hierarchical network nodes.

---

## Ground Reality
LDAP is a directory service protocol optimized for heavy reading and searching, structured in a hierarchical tree:
* **DN (Distinguished Name)**: The unique path to a node (e.g. `cn=Bob,ou=HR,dc=corp,dc=com`).
* **Attributes**: Key-value metadata (e.g., `mail: bob@corp.com`).
To authenticate, a client performs an **LDAP Bind** operation, passing a DN and password.

---

## Mental Model
Think of LDAP as a **corporate organizational tree directory**:
* You start at the root (the company name).
* You branch down to divisions (Departments/OUs).
* You branch down to employees (User objects).
Each branch lists contact details and group list tags.

```text
             [ dc=corp,dc=com ] (Root)
                    │
           ┌────────┴────────┐
      [ ou=Sales ]      [ ou=Eng ] (Organizational Units)
           │                 │
      [ cn=Alice ]      [ cn=Bob ] (Common Names / Users)
```

---

## Visual Diagrams

### LDAP Bind Authentication Protocol

```text
Client Application                                  LDAP Server
┌──────────────────────┐                           ┌──────────────────────┐
│ Connect: Port 389/636│ ─────────────────────────>│ Establishes socket.  │
├──────────────────────┤                           ├──────────────────────┤
│ LDAP Bind Request:   │                           │ Matches DN,          │
│ DN: cn=Bob,ou=Eng    │ ─────────────────────────>│ hashes password.     │
│ Pass: supersecret    │◄─────────────────────────│ Validates record.    │
│                      │    LDAP Bind Success     │                      │
├──────────────────────┤                           ├──────────────────────┤
│ LDAP Search:         │                           │                      │
│ filter=(memberOf=HR) │ ─────────────────────────>│ Returns group items. │
│                      │◄─────────────────────────│                      │
└──────────────────────┘                           └────────────────└─────┘
```

---

## Under The Hood
A typical LDAP search filter query execution pattern in Python:

```python
import ldap3

server = ldap3.Server('ldaps://ldap.corp.com:636')
conn = ldap3.Connection(server, 'cn=admin,dc=corp,dc=com', 'adminpassword', auto_bind=True)

# Search for Bob's groups
conn.search('ou=people,dc=corp,dc=com', '(cn=bob)', attributes=['memberOf'])
print(conn.entries)
```

---

## Packet Journey
LDAP packets utilize Basic Encoding Rules (BER) to format binary data fields over TCP port 389 (LDAP) or 636 (LDAPS). A bind packet sequence looks like:

```text
LDAP Message:
  Message ID: 1
  Protocol Op: Bind Request (0)
    Version: 3
    Name: cn=Bob,ou=Eng,dc=corp,dc=com
    Authentication: Simple
      Password: password123
```

---

## Kernel Perspective
The kernel schedules network interrupts for packets incoming on LDAP ports (389/636). Raw BER decoding is processed by user-space LDAP service daemons (e.g. OpenLDAP, Active Directory).

---

## Observe It Yourself
Query a public test LDAP server using `ldapsearch`:
```bash
ldapsearch -x -H ldap://ldap.forumsys.com:389 -b "dc=example,dc=com" "(uid=tesla)"
```

---

## Common Misconceptions
> **Myth**: "LDAP is obsolete in modern network environments."
> 
> **Reality**: While web systems use OIDC, internal enterprise infrastructure (VPNs, printers, local firewalls, linux server administration) still relies on LDAP integration for core directory lookup.

---

## Attack Scenarios
* **LDAP Injection**: An application constructs LDAP filters dynamically using unvalidated user input. An attacker inputs special characters like `*` or `(` to modify the search filter, bypassing authentication or retrieving restricted attributes.

---

## Defenses
* **Escape Input**: Sanitize search query input variables when constructing search filters.
* **Force LDAPS**: Always secure LDAP connections over TLS (port 636) to prevent sniffing passwords.

---

## Tradeoffs
* **Write Overhead**: Directory trees are optimized for fast querying, making write modifications (updating records) slow compared to traditional SQL databases.

---

## Related Concepts
* **Active Directory**: Microsoft's LDAP directory implementation.

---

## Deep Dive
LDAP Distinguished Name (DN) structure parsing:
* `CN`: Common Name (individual user or object).
* `OU`: Organizational Unit (department folder).
* `DC`: Domain Component (domain parts, e.g. `corp.com` splits to `dc=corp,dc=com`).

---

## Case Studies
* **Log4Shell LDAP Exploits (2021)**: The Log4j vulnerability exploited JNDI lookup mechanisms to query rogue LDAP servers, downloading and executing malicious Java code on targeted servers.

---

## Mini Projects
* **Python Mock LDAP Service**: Write a mock authentication backend that validates credentials against a local simulated directory file.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 25: Active Directory

## Why Does This Exist?
Large Windows environments need a centralized directory and security domain controller to handle thousands of machines, policy updates, and identity assertions across computers.
**Active Directory (AD)** extends LDAP by combining it with DNS services, Kerberos authentication, and Group Policy Objects (GPOs) to manage enterprise computer networks.

---

## Ground Reality
Active Directory Domain Services (AD DS) runs on Domain Controllers (DCs).
* **Forest**: The top-level container structure.
* **Domain**: A security boundary partition containing users and computers.
* **Domain Controller**: The server executing the AD service, processing authentication, policy enforcement, and credential validations.

---

## Mental Model
Think of Active Directory as a **city administration department**:
* **LDAP**: The physical catalog folder containing citizens and properties.
* **Kerberos**: The official stamps and identity tickets issued to citizens.
* **Group Policy**: The municipal rules and building codes enforced on properties.

```text
                  [ Active Directory Controller ]
                  ┌──────────────┼──────────────┐
                  ▼              ▼              ▼
                LDAP          Kerberos        Group
              (Directory)   (Auth Engine)    Policies
```

---

## Visual Diagrams

### Active Directory Domain Joining & Auth Architecture

```text
Windows Client Machine                              Domain Controller (DC)
┌──────────────────────┐                           ┌──────────────────────┐
│ Request DNS Resolver │ ─────────────────────────>│ Resolves DC SRV record│
│                      │◄─────────────────────────│                      │
├──────────────────────┤                           ├──────────────────────┤
│ Connect via Kerberos │                           │                      │
│ (Port 88)            │ ─────────────────────────>│ Verifies Computer.   │
│                      │◄─────────────────────────│ Issues Ticket (TGT)  │
├──────────────────────┤                           ├──────────────────────┤
│ Apply Group Policies │                           │                      │
│ (GPO)                │◄─────────────────────────│ Pushes GPO settings. │
└──────────────────────┘                           └──────────────────────┘
```

---

## Under The Hood
Active Directory uses standard LDAP interfaces. Searching the AD Global Catalog in PowerShell:

```powershell
# Querying Active Directory users
Get-ADUser -Filter 'Name -like "*Bob*"' -Properties MemberOf
```

---

## Packet Journey
AD communications leverage DNS SRV record lookups to locate Domain Controllers over network links:

```text
DNS SRV Request: _kerberos._tcp.corp.com
DNS SRV Response: dc1.corp.com:88
```

---

## Kernel Perspective
Windows operating system kernels intercept local logon credential submissions, routing them via the Local Security Authority Subsystem Service (`lsass.exe`) to perform remote DC Kerberos operations.

---

## Observe It Yourself
Check domain connection details from a Windows CMD prompt:
```cmd
gpresult /R
```

---

## Common Misconceptions
> **Myth**: "Active Directory is just LDAP."
> 
> **Reality**: AD uses LDAP as its query interface, but relies on Kerberos for authentication, DNS for service locator records, and SMB protocols to distribute Group Policy files.

---

## Attack Scenarios
* **Golden Ticket Attack**: Attackers compromise the Domain Controller's Key Distribution Center service account (`KRBTGT`) and extract its password hash, enabling them to forge active Kerberos tickets that grant permanent domain administrator access.

---

## Defenses
* **LAPS (Local Administrator Password Solution)**: Automatically rotate local computer administrator passwords in AD.
* **Tiered Administrative Access**: Keep domain admin accounts separated from workstation endpoints to prevent credential theft.

---

## Tradeoffs
* **Windows Lock-In**: Heavy reliance on proprietary Microsoft architectures makes Linux integration difficult.

---

## Related Concepts
* **Azure Active Directory**: A separate, cloud-native identity service (now Entra ID) utilizing OIDC instead of Kerberos.

---

## Deep Dive
The `lsass.exe` process:
The Local Security Authority Subsystem Service manages security policy, audits, and user authentications on Windows systems. Attackers target this process memory space (e.g. using Mimikatz) to dump plaintext credentials or Kerberos session keys.

---

## Case Studies
* **NotPetya Cyberattack (2017)**: The malware harvested credentials from system memory using Mimikatz and spread automatically across corporate networks by exploiting Active Directory trust connections.

---

## Mini Projects
* **AD Query Tool**: Write a Python script using LDAP libraries to connect to a mocked AD LDAP database, searching for users belonging to specific security groups.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 26: Kerberos

## Why Does This Exist?
When authenticating in a corporate network over unsecure local links, sending password hashes or session tokens to multiple internal file servers and services exposes them to theft.
**Kerberos** solves this by using a trusted third-party ticket system, allowing a client to prove their identity to network services without ever sending passwords or credentials over the wires.

---

## Ground Reality
Kerberos relies on a **Key Distribution Center (KDC)** containing:
1. **Authentication Service (AS)**: Verifies user identity and issues a Ticket Granting Ticket (TGT).
2. **Ticket Granting Service (TGS)**: Verifies the TGT and issues a Service Ticket.
Clients present the Service Ticket to access target services.
Everything is symmetric key cryptography based.

---

## Mental Model
Think of visiting an amusement park with separate ticket checkpoints:
1. You go to the front gate cash desk (AS), show ID and pay (Log in). They stamp your hand with a pass (TGT).
2. When you want to ride the roller coaster, you go to the ticket booth (TGS) and show your hand stamp. They issue you a ride coupon (Service Ticket).
3. You hand the coupon to the ride operator (Service) to enter. The ride operator does not look at your ID card or cash.

```text
User ──► [ Auth Service (AS) ] ──► (TGT) ──► [ Ticket Service (TGS) ] ──► (Service Ticket) ──► [ Ride Operator ]
```

---

## Visual Diagrams

### Kerberos Triple Handshake Flow

```text
Client Computer               Key Distribution Center (KDC)            Target Service
  │                                     │                                    │
  │─── 1. AS Request (ID, Timestamp) ──►│                                    │
  │◄── 2. AS Reply (TGT + Session Key) ─│                                    │
  │                                     │                                    │
  │─── 3. TGS Request (TGT + Auth) ────►│                                    │
  │◄── 4. TGS Reply (Service Ticket) ───│                                    │
  │                                                                          │
  │─── 5. Service Request (Service Ticket) ─────────────────────────────────►│
  │◄── 6. Service Reply (Mutual verification) ───────────────────────────────│
```

---

## Under The Hood
Kerberos ticket validation math structure:

```text
TGT contents (encrypted with KDC private key):
{
  "client_id": "alice",
  "client_ip": "192.168.1.10",
  "session_key": "K_C_KDC",
  "expiry": 1774905720
}
```

---

## Packet Journey
Kerberos communications operate over UDP/TCP port 88. A Ticket Granting Service request packet looks like:

```text
Kerberos AP-REQ
  Protocol Version: 5
  Message Type: Ticket Granting Service Request (12)
  Realm: CORP.COM
  Sname: cifs/fileserver.corp.com
  Ticket: (Encrypted payload containing ticket metadata)
```

---

## Kernel Perspective
Linux processes can authenticate via Kerberos by mounting local ticket caches stored in `/tmp/krb5cc_*` files. System authentication modules read these cache files to complete API validations.

---

## Observe It Yourself
List your active Kerberos tickets in a Linux/Mac terminal:
```bash
klist
```

---

## Common Misconceptions
> **Myth**: "Kerberos requires asymmetric public key cryptography."
> 
> **Reality**: Classic Kerberos relies entirely on symmetric key cryptography. The KDC shares symmetric secrets with every user and service, using them to construct nested encryption envelopes.

---

## Attack Scenarios
* **Kerberoasting**: Attackers compromise a normal domain user account, query the Domain Controller for Kerberos service tickets for accounts running database services, and crack the ticket hashes offline to recover service account passwords.

---

## Defenses
* **Strong Service Passwords**: Use long, complex passwords for service accounts to prevent offline cracking.
* **AES Encryption**: Enforce AES-256 encryption for Kerberos tickets, disabling weak legacy DES/RC4 ciphers.

---

## Tradeoffs
* **Clock Sync Bounds**: Kerberos requires strict time synchronization (usually within 5 minutes) across the network. If client/server clocks drift, authentication fails.

---

## Related Concepts
* **SPNEGO**: A negotiation mechanism used by web browsers to establish Kerberos connections.

---

## Deep Dive
Kerberos ticket lifecycle:
Tickets have an expiration window (usually 10 hours) and a renewal window (usually 7 days). This restricts the usability window of intercepted tickets without forcing users to re-enter passwords continually.

---

## Case Studies
* **The SolarWinds Orion Hack (2020)**: Cybercriminals bypassed federated authentication controls by utilizing compromised active Kerberos ticket generation accounts to pivot laterally within private networks.

---

* **Kerberos Key Generator**: Write a Python script that takes a username, password, and realm, and generates a standard Kerberos Keytab file containing hashed secrets.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 27: MFA & TOTP (Multi-Factor Authentication & Time-Based One-Time Passwords)

## 1. Why Does This Exist?
Passwords are static credentials. If an attacker intercepts, sniffs, guesses, or leaks a password from a database, they gain full, permanent access to the account. 
**Multi-Factor Authentication (MFA)** introduces secondary dynamic verification factors. **Time-Based One-Time Passwords (TOTP)** provide a dynamic possession factor that changes automatically every 30 seconds, ensuring that intercepted credentials become useless almost immediately.

---

## 2. Historical Context
Early multi-factor systems used physical hardware tokens (like RSA SecurID key fobs) that displayed rotating numbers. These tokens were expensive to manufacture, required manual distribution to employees, and suffered from clock-drift over several years, rendering them unusable. TOTP (RFC 6238) standardized this process, allowing software applications on smartphones to emulate hardware tokens.

---

## 3. Problems Before It Existed
* **Static Credentials**: Reusable passwords meant single-point-of-failure security architectures.
* **Costly Hardware**: Proprietary security hardware restricted MFA adoption to large financial and military organizations.
* **Proprietary Protocols**: Lack of interoperability forced vendors into expensive single-supplier lock-ins.

---

## 4. Ground Reality
TOTP is a symmetric-key protocol that derives a transient 6-digit or 8-digit numeric code from:
1. A shared secret key ($K$), established during setup via QR code scan.
2. The current epoch time, divided into step windows (usually 30 seconds) to form a counter ($T$).
Both client and server calculate this code independently. The client submits the code, and the server validates it.

---

## 5. Mental Models
Think of two synchronized clocks with matching gears that turn a combination lock. Even if an outsider sees the combination numbers for the current minute, the gears will shift at the next mark, locking them out and requiring a new combination.

---

## 6. Analogies
A TOTP token behaves like a **bank vault containing a mechanical calendar wheel**:
* The calendar wheel rotates to a new day every 30 seconds.
* Both you and the bank teller have the exact same instruction book (Shared Secret) to translate the calendar wheel's daily position into a numeric code.
* Even if someone steals yesterday's code, the bank teller rejects it because the calendar wheel has already advanced.

---

## 7. Internal Machinery
The TOTP algorithm is built upon HTOP (HMAC-Based One-Time Password, RFC 4226).
1. Calculate the counter value: $T = \lfloor (\text{Current Unix Time} - \text{T0}) / T_{\text{interval}} \rfloor$.
2. Calculate the HMAC hash: $HS = \text{HMAC-SHA1}(K, T)$.
3. Perform **Dynamic Truncation** on $HS$ to extract a 4-byte segment.
4. Convert the segment to an integer and compute modulo $10^{\text{digits}}$ (usually $10^6$) to produce the final token.

---

## 8. State Machines

```text
       ┌────────────────────────┐
       │   Setup / Scan QR      │ ──► Store Shared Secret (K)
       └───────────┬────────────┘
                   │
                   ▼
       ┌────────────────────────┐
  ┌──► │ Calculate Counter T    │ ──► T = (UnixTime - 0) / 30
  │    └───────────┬────────────┘
  │                │
  │                ▼
  │    ┌────────────────────────┐
  │    │ Compute HMAC-SHA1(K,T) │ ──► Get 20-byte Hash (HS)
  │    └───────────┬────────────┘
  │                │
  │                ▼
  │    ┌────────────────────────┐
  │    │   Dynamic Truncation   │ ──► Extract 4 bytes using offset
  │    └───────────┬────────────┘
  │                │
  │                ▼
  │    ┌────────────────────────┐
  │    │  Modulo 10^6 and Emit  │ ──► Print/Submit Code
  │    └───────────┬────────────┘
  │                │
  └────────────────┴── 30-Second Boundary Reached
```

---

## 9. Sequence Diagrams

```text
Client App (Authenticator)                User Browser             Server Backend
    │                                          │                          │
    │─── Scan Setup QR (Shared Secret K) ─────►│                          │
    │                                          │─── Submit Username/Pass ─►│
    │                                          │◄── Prompt TOTP Code ─────│
    │                                          │                          │
    │─── Read local system time                │                          │
    │─── Calculate TOTP(K, Time) -> 482910     │                          │
    │─── Display 482910                        │                          │
    │                                          │                          │
    │◄── User reads and types 482910 ──────────│                          │
    │                                          │─── Submit Code 482910 ──►│
    │                                          │                          │ (Calculates expected
    │                                          │                          │  TOTP for current time
    │                                          │                          │  window. Compare match)
    │                                          │◄── Authentication OK ────│
```

---

## 10. Layer Diagrams

```text
+-----------------------------------------------------------+
|               User Interface / Authenticator App          |
+-----------------------------------------------------------+
|         TOTP Engine (RFC 6238) / Step counter logic       |
+-----------------------------------------------------------+
|         HOTP Core (RFC 4226) / Dynamic Truncation         |
+-----------------------------------------------------------+
|         HMAC Cryptographic Engine (SHA-1 / SHA-256)       |
+-----------------------------------------------------------+
|         System Real-Time Clock (RTC) / Epoch Unix Time    |
+-----------------------------------------------------------+
```

---

## 11. Packet Journey
A TOTP payload submitted inside a standard POST login request:

```http
POST /api/v1/auth/login-mfa HTTP/1.1
Host: secure-vault.corp.com
Content-Type: application/json

{
  "username": "alice",
  "password": "supersecretpassword",
  "totp_code": "582019"
}
```

---

## 12. Memory Journey
1. The shared secret $K$ is decrypted from client app secure storage (e.g. Android Keystore / iOS Keychain) into memory space.
2. The current epoch timestamp is read via system libraries into a 64-bit integer variable.
3. The HMAC buffer performs mathematical iterations, caching intermediate state variables.
4. The memory blocks containing the raw secret $K$ and intermediate hash states are zeroized immediately after code derivation to prevent extraction via core dumps or debugging processes.

---

## 13. CPU Perspective
The CPU executes the TOTP calculations within a fraction of a millisecond.
* Retrieves current time using hardware timers (TSC / HPET).
* Performs bitwise operations (XOR, shifting) to implement SHA-1 compression rounds.
* Executes bitmasking operations to extract the offset index: `offset = hash[19] & 0xf`.
* Generates a constant-time comparison path to check client inputs, avoiding branching hazards.

---

## 14. Kernel Perspective
The kernel services user-space time requests using the `vdso` (virtual dynamic shared object) framework. This executes the `clock_gettime()` system call directly in user space, avoiding expensive context switches.

---

## 15. Cryptographic Perspective
HMAC-SHA1 hashing properties secure the core TOTP generation loop. While SHA-1 is deprecated for collision resistance (digital signatures), its preimage resistance properties remain secure, making it impossible for an observer to reverse the output codes back to discover the master secret key $K$.

---

## 16. Performance Characteristics
* **Computational Cost**: Extremely low. Uses basic bitwise ops and a single HMAC loop.
* **Server Verification Scaling**: $O(1)$ lookup and math overhead, easily processing thousands of requests per second.

---

## 17. Tradeoffs
* **Clock Sync Dependency**: Requires both client and server clocks to match. If clocks drift beyond $T_{\text{interval}}$, codes fail.
* **Storage Footprint**: The server must securely store the shared secret key $K$ for every user, creating a database targeting surface.

---

## 18. Failure Modes
* **Epoch Reset / Clock Drift**: The client's clock slips by 2 minutes, causing all generated codes to belong to expired or future server validation windows.
* **Database Leak**: An attacker dumps the user table containing plaintext TOTP secrets, enabling them to generate valid tokens indefinitely.

---

## 19. Attack Surface
* **Phishing/Proxy Attacks (AitM)**: Attackers set up a reverse-proxy login portal. The user inputs their credentials and TOTP code. The proxy forwards these instantly to the real server, establishing an active session for the attacker before the 30-second token expires.
* **Sim-Swap**: Not directly applicable to TOTP, but affects SMS-based MFA bypass techniques.

---

## 20. Defenses
* **Time Drift Tolerance Windows**: Servers allow a buffer window of $\pm 1$ steps (checking the previous, current, and next code) to accommodate slight user clock drifts.
* **Secret Encryption at Rest**: Encrypt user TOTP secret keys in the database using a master key stored in a hardware security module (HSM).

---

## 21. Real-World Examples
Google Authenticator, Microsoft Authenticator, and Authy use TOTP to generate 2FA tokens.

---

## 22. Production Architectures

```text
                       ┌─────────────────────────┐
                       │   Reverse Proxy / API   │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │   Auth Service Pods     │
                       └────────────┬────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌──────────────────────┐                          ┌──────────────────────┐
│  Primary Database    │                          │      HSM Cluster     │
│  (Encrypted User K)  │ ◄──────────────────────► │  (Decrypts User K    │
└──────────────────────┘                          │   for Validation)    │
                                                  └──────────────────────┘
```

---

## 23. Reverse Engineering Perspective
Decompiling mobile authenticators reveals the base32 decoding logic of the configuration URI schemes:
`otpauth://totp/Issuer:user@domain.com?secret=JBSWY3DPEHPK3PXP&issuer=Issuer`
The token generator parses this secret parameter into a byte array before running the SHA-1 HMAC loop.

---

## 24. Observe It Yourself
Calculate a TOTP code locally using a Python script:
```python
import time, hmac, hashlib, struct, base64

def get_totp(secret):
    key = base64.b32decode(secret, casefold=True)
    # Calculate time counter (30s steps)
    counter = int(time.time() / 30)
    # Pack counter as 8-byte big-endian binary struct
    msg = struct.pack(">Q", counter)
    # HMAC-SHA1 signature
    hs = hmac.new(key, msg, hashlib.sha1).digest()
    # Dynamic Truncation
    offset = hs[19] & 0xf
    binary = struct.unpack(">I", hs[offset:offset+4])[0] & 0x7fffffff
    # Modulo to get 6 digit code
    code = binary % 1000000
    return f"{code:06d}"

# Secret must be base32 format
print("Your TOTP Code:", get_totp("JBSWY3DPEHPK3PXP"))
```

---

## 25. Common Misconceptions
> **Myth**: "The authenticator app communicates with the server over the internet to get the code."
> 
> **Reality**: No. The app runs completely offline. It calculates the code mathematically using only its local clock and the stored secret key $K$.

---

## 26. Related Concepts
* **HOTP**: Counter-Based One-Time Passwords (changes only when requested, rather than over time steps).
* **Backup Codes**: Static, single-use keys to bypass TOTP if the authenticator device is lost.

---

## 27. Deep Dives
Why SHA-1 is still used in TOTP despite collision vulnerabilities:
Symmetric signatures (HMAC) do not rely on hash collision resistance. They require preimage resistance (meaning you cannot reconstruct $K$ from $HS$). Because SHA-1 preimage resistance remains secure, there is no threat to the integrity of the generated codes.

---

## 28. Case Studies
* **The RSA SecurID Compromise (2011)**: Attackers hacked RSA Security and stole the seed files containing the master secrets for millions of SecurID physical tokens. Using this data, they duplicated tokens and breached corporate networks.

---

## 29. Interview Questions
* **Q**: "What happens if a user submits a TOTP code twice within the same 30-second window?"
* **A**: The server must keep a local cache of recently used codes and reject duplicate submissions within the same window to prevent token reuse replay attacks.

---

## 30. Mini Projects
* **TOTP CLI Utility**: Write a CLI program that decodes an `otpauth://` QR-code string, stores the configuration securely, and prints a CLI progress bar showing the remaining seconds of the current code.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 28: Hardware Keys & U2F

## 1. Why Does This Exist?
Software-based authentication methods (like SMS, push notifications, and TOTP) are vulnerable to phishing. An attacker can set up a fake login page that intercepts both the password and the OTP code, forwarding them immediately to log in.
**Hardware Keys** and the **Universal 2nd Factor (U2F)** protocol solve this by linking authentication directly to the specific cryptographic domain origin, ensuring that codes generated for a fake domain (e.g. `google-login.com`) are rejected by the target domain (`google.com`).

---

## 2. Historical Context
Yubico, Google, and NXP founded the FIDO (Fast IDentity Online) Alliance in 2012 to eliminate phishing attacks. U2F was their first protocol, utilizing public key cryptography embedded inside secure USB/NFC hardware tokens to establish origin-bound identity challenges.

---

## 3. Problems Before It Existed
* **Adversary-in-the-Middle (AiTM)**: Hackers easily bypassed TOTP/SMS validation loops by tricking users into inputting codes on proxy domains.
* **Physical Device Compromise**: Phone malware could read stored TOTP database keys.
* **Credential Stuffing**: Reusable key material allowed attackers to compromise credentials offline.

---

## 4. Ground Reality
U2F operates over USB, NFC, or Bluetooth. During registration, the U2F hardware key generates a brand-new cryptographic key pair bound to the specific domain origin (e.g. `https://github.com`).
* **Registration**: The key returns its public key and a **Key Handle** to the server.
* **Authentication**: The server sends a challenge. The hardware key uses the key handle to locate or recreate the private key, signs the challenge along with the verified domain origin, and returns the signature.

---

## 5. Mental Models
Think of a lock system that refuses to unlock unless the key has been physically laser-etched with the exact coordinate address of the building. Even if someone duplicates the key shape, if they insert it in a door at a different address, the key detects the location mismatch and refuses to rotate.

---

## 6. Analogies
A U2F token acts like a **notary who has a photographic memory for shapes**:
* The notary sits inside a secure vault (Secure Element).
* When a website asks for a signature, the notary looks out the window to verify the street name (Browser Origin).
* If the street name matches the document register, the notary stamps it. If the street name is off by one character, the notary shreds the document.

---

## 7. Internal Machinery
A secure element inside the hardware token contains a master key ($MK$). When registering:
1. The token receives the `AppID` (domain hash).
2. It combines the `AppID` and a random nonce ($N$) using HMAC-SHA256 and the master key $MK$ to derive a unique private key ($PrK$).
3. The token exports the public key ($PuK$) along with the nonce $N$ (serving as the Key Handle) back to the server. This design avoids storing keys on the hardware device itself, allowing infinite key registrations on restricted hardware.

---

## 8. State Machines

```text
                 [ Registration Flow ]
Client Browser                               Hardware Key
      │                                            │
      │─── Register Request (AppID, Challenge) ───►│ (Prompt user check:
      │                                            │  touch button)
      │                                            │
      │                                            ▼
      │                                     Derive Key pair:
      │                                     PrK = HMAC(MK, AppID + Nonce)
      │                                            │
      │◄── Send Public Key & Key Handle (Nonce) ───│
      │
      ▼
                 [ Authentication Flow ]
Client Browser                               Hardware Key
      │                                            │
      │─── Auth Request (AppID, Key Handle) ──────►│ (Verify AppID matches
      │                                            │  origin domain)
      │                                            │
      │                                            ▼
      │                                     Recreate Private Key:
      │                                     PrK = HMAC(MK, AppID + Nonce)
      │                                            │
      │                                            ▼
      │                                     Sign Challenge + Origin
      │                                            │
      │◄── Return Signature & Counter ─────────────│
```

---

## 9. Sequence Diagrams

```text
User                      Browser App                 Hardware Key                Server Database
 │                            │                            │                             │
 │─── Input credentials ─────►│                            │                             │
 │                            │─── Request login ───────────────────────────────────────►│
 │                            │◄── Send challenge, Key Handle, AppID ────────────────────│
 │                            │                            │                             │
 │                            │─── Send U2F Auth Request ─►│                             │
 │                            │◄── Flash LED / Wait Touch ─│                             │
 │─── Touch Key button ───────────────────────────────────►│                             │
 │                            │                            │─── Verify Origin == AppID   │
 │                            │                            │─── Re-derive Private Key    │
 │                            │                            │─── Increment monotonic CTR  │
 │                            │                            │─── Sign (Challenge + Origin)│
 │                            │◄── Signature, Counter ─────│                             │
 │                            │─── Forward Response ────────────────────────────────────►│
 │                            │                                                          │ (Verify signature
 │                            │                                                          │  using Public Key,
 │                            │                                                          │  verify Counter >
 │                            │                                                          │  last recorded)
 │                            │◄── Authentication Success ───────────────────────────────│
```

---

## 10. Layer Diagrams

```text
+-----------------------------------------------------------+
|          Web Application Interface (Javascript API)       |
+-----------------------------------------------------------+
|          Browser Client Platform (FIDO / U2F API Engine)  |
+-----------------------------------------------------------+
|          Hardware Interface Layer (HID Driver / USB / NFC)|
+-----------------------------------------------------------+
|          Token Controller / USB Controller Chip           |
+-----------------------------------------------------------+
|          Secure Element (ECC calculations, Master Key)   |
+-----------------------------------------------------------+
```

---

## 11. Packet Journey
A FIDO U2F raw execution frame representation over USB HID (Human Interface Device):

```text
Frame Format:
- Report ID: 0x00
- Channel ID: 0x12345678 (Virtual bus channel)
- Command: MSG (0x83)
- Payload Length: 0x0040 (64 bytes)
- Payload Data:
    - Parameter: Control Byte (0x02 - Authenticate)
    - Hash: Client Data Hash (32 bytes)
    - Hash: Application Parameter / AppID Hash (32 bytes)
```

---

## 12. Memory Journey
The hardware key's Secure Element executes in isolated memory domains. The Master Key ($MK$) remains trapped in flash ROM blocks that are physically inaccessible to the USB controller CPU. Intermediate ECC signing calculations store dynamic values on internal registers that zero out instantly on power-loss.

---

## 13. CPU Perspective
The hardware key's internal microprocessor spends most of its time in low-power sleep mode. Upon receiving a USB interrupt, it wakes, loads the inputs into registers, requests the secure element to run the modular elliptic curve math for ECDSA (secp256r1) signing, increments a hardware-backed monotonic counter, and schedules the return packets on the USB transmit queue.

---

## 14. Kernel Perspective
The host Linux kernel communicates with hardware keys via the `uinput` or generic `hidraw` drivers. It does not require specific kernel module setups, exposing the hardware key to user-space browsers through raw read/write access nodes under `/dev/hidraw*`.

---

## 15. Cryptographic Perspective
U2F uses asymmetric Elliptic Curve Cryptography, specifically the **NIST P-256 curve (secp256r1)**. The signature guarantees:
* **Integrity**: Challenge and origin cannot be modified.
* **Origin Assertion**: The signature is generated over the browser's assertion of the domain, blocking phishing relays.
* **Monotonic Counter Verification**: The key includes an incrementing counter value, detecting and blocking duplicated clones of physical key data.

---

## 16. Performance Characteristics
* **Symmetric Key Derivation**: $< 1$ ms on hardware token.
* **Asymmetric ECDSA Signing**: $10 - 50$ ms inside the secure element.
* **No Network Latency**: The token computes everything locally without contacting external networks.

---

## 17. Tradeoffs
* **Physical Device Dependency**: If you lose the physical key, you are locked out unless backup methods exist.
* **Device Cost**: Unlike software TOTP, physical hardware keys require capital investment.

---

## 18. Failure Modes
* **Counter Desynchronization**: If the server's database tracks a counter value higher than the value presented by the key (due to database corruptions or key rollbacks), the server flags the key as cloned and blocks access.
* **Physical Damage**: Electrostatic discharge destroys the secure element's memory blocks.

---

## 19. Attack Surface
* **Supply Chain Compromise**: Attackers intercept hardware key shipments and inject custom firmware backdoors before delivering them to users.
* **Browser Sandbox Escapes**: Malware running on the local OS bypasses browser restrictions, communicating directly with the USB `/dev/hidraw` nodes to sign forged challenges.

---

## 20. Defenses
* **Hardware Attestation Certificates**: During registration, the key returns a certificate signed by the manufacturer's root key (e.g. Yubico Root CA). The server verifies this signature to guarantee the key is genuine hardware.
* **Multiple Key Registrations**: Register at least two hardware keys (a primary and a backup stored in a safe location).

---

## 21. Real-World Examples
Yubikey (Yubico), Google Titan Security Key, and OnlyKey are U2F-compatible security keys.

---

## 22. Production Architectures

```text
                       ┌─────────────────────────┐
                       │    Web Application      │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │   Identity Service API  │
                       └────────────┬────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌──────────────────────┐                          ┌──────────────────────┐
│  Users DB            │                          │  Attestation Store   │
│  (Saves WebAuthn     │                          │  (Validates Key      │
│   Credentials)       │                          │   Manufacturer ID)   │
└──────────────────────┘                          └──────────────────────┘
```

---

## 23. Reverse Engineering Perspective
Using USB monitoring tools (like USBPcap / Wireshark), you can sniff raw HID transactions. You will observe structured APDU (Application Protocol Data Unit) blocks matching the ISO 7816 smart card standards, wrapping the cryptographic operations inside byte structures.

---

## 24. Observe It Yourself
Interact with local USB raw devices using Python to list connected FIDO tokens:
```bash
python3 -c "import fido2.hid; print(list(fido2.hid.CtapHidDevice.list_devices()))"
```

---

## 25. Common Misconceptions
> **Myth**: "Hardware keys track all the sites you visit, threatening user privacy."
> 
> **Reality**: No. The key does not store a list of registered domains. The key handles are stored on the server's database. Additionally, each domain gets a completely separate key pair, preventing cross-domain user tracking.

---

## 26. Related Concepts
* **CTAP1**: Client-to-Authenticator Protocol 1 (U2F).
* **CTAP2**: Extension allowing passwordless authentication.

---

## 27. Deep Dives
How the Key Handle recovery loop works:
Instead of allocating expensive EEPROM flash storage on the key, the key handle is actually an encrypted bundle containing the private key. When authenticating, the server sends this handle back to the token. The token uses its internal master key ($MK$) to decrypt the handle, recovering the transient private key.

---

## 28. Case Studies
* **Google's Phishing Elimination (2017)**: Google mandated physical U2F security keys for all 85,000+ employees. Since completing the deployment, Google reported zero successful work account phishing takeovers.

---

## 29. Interview Questions
* **Q**: "How does a U2F token protect against a Man-in-the-Middle attacker who sets up a proxy site at `secure-github.com`?"
* **A**: The browser sends the real origin domain (`secure-github.com`) to the token. The token signs this origin. When the server validates the signature, it detects that the signed origin does not match its actual domain (`github.com`) and rejects the response.

---

* **Software Token Simulator**: Write a Python tool that simulates a hardware U2F device in software, handling registration challenges, deriving keys using an internal master key, and signing payloads.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 29: WebAuthn (Web Authentication)

## 1. Why Does This Exist?
U2F was designed as a secondary authentication factor (MFA), meaning users still needed to enter their usernames and passwords first. This did not eliminate password databases, which remained major targets for leaks and brute-forcing.
**WebAuthn** expands this framework to allow **passwordless authentication**, letting users authenticate securely using local biometric devices (like fingerprint readers, face ID, or device PINs) without passing passwords across the network.

---

## 2. Historical Context
WebAuthn is a joint standard by the W3C and the FIDO Alliance, finalized in 2019. It standardizes browser APIs to allow websites to directly communicate with authenticators (built-in security hardware or external security keys) using CTAP2 protocols, replacing password forms.

---

## 3. Problems Before It Existed
* **Password Leaks**: Massive server databases storing password hashes were routinely breached.
* **Bad UX**: Users struggled to remember long, complex passwords across hundreds of websites.
* **MFA Friction**: Requiring users to enter a password and then locate a 2FA code slowed down logins.

---

## 4. Ground Reality
WebAuthn is a web standard built around asymmetric cryptography.
* **Client Credentials**: The user's device (phone, laptop) generates a unique key pair for the target site. The private key remains locked inside the device's hardware TPM or Secure Enclave.
* **Server Enrollment**: The public key is sent to the server.
* **Local Verification**: The user unlocks the local private key using biometrics (fingerprint/face) or a device PIN. The device then signs the server's challenge to prove identity.

---

## 5. Mental Models
Think of a website login as an automated lockbox that requires an official cryptographic signature. Instead of checking your ID card, the lockbox sends a document (Challenge) to your smartphone. You unlock your phone's signature stamp (Biometrics/PIN) to sign the document. The lockbox verifies the signature using the public key it recorded during registration.

---

## 6. Analogies
WebAuthn is like a **personal notary living on your phone**:
* When a website asks: "Who are you?" it sends a blank claim check (Challenge).
* The phone notary asks you to prove you are there by touching the fingerprint sensor (User Verification).
* Once you touch it, the notary signs the claim check with their official stamp (Private Key) and sends it back.
* The website checks the stamp against its records.

---

## 7. Internal Machinery
During login, the browser invokes the WebAuthn API:
1. `navigator.credentials.create()` or `navigator.credentials.get()` is called.
2. The browser validates the origin domain to prevent domain spoofing.
3. The browser communicates with the device's authenticator using the CTAP2 protocol.
4. The authenticator prompts the user for local verification (biometrics or PIN).
5. The authenticator returns an assertion object containing signed client data and an authenticator data payload.

---

## 8. State Machines

```text
                  [ WebAuthn Verification Loop ]
    Application JS                                   Browser Core
          │                                               │
          │─── navigator.credentials.get(options) ───────►│
          │                                               │─── Verify Origin
          │                                               │    (e.g., github.com)
          │                                               │
          │                                               ▼
          │                                      [ Query Authenticator ]
          │                                               │
          │                                               ▼
          │                                       Verify User Presence
          │                                       (Biometrics / PIN Check)
          │                                               │
          │                                               ▼
          │                                       Generate Signature:
          │                                       ECDSA(Challenge + Origin)
          │                                               │
          │◄── Return credential assertion object ────────│
```

---

## 9. Sequence Diagrams

```text
User                      Browser API                 Local TPM / Key             Server Backend
 │                            │                             │                            │
 │─── Click "Login" ─────────►│                             │                            │
 │                            │─── Fetch Challenge ─────────────────────────────────────►│
 │                            │◄── Return challenge & credential ID ─────────────────────│
 │                            │                             │                            │
 │                            │─── Request Assertion ──────►│                            │
 │                            │◄── Prompt Fingerprint Check │                            │
 │─── Place Fingerprint ───────────────────────────────────►│                            │
 │                            │                             │─── Validate Fingerprint    │
 │                            │                             │─── Sign (Challenge + Origin│
 │                            │                             │    using private key)      │
 │                            │◄── Return Signature payload │                            │
 │                            │─── Submit Assertion to Server ──────────────────────────►│
 │                            │                                                          │ (Verify signature
 │                            │                                                          │  using Public Key,
 │                            │                                                          │  validate challenge)
 │                            │◄── Login Success ────────────────────────────────────────│
```

---

## 10. Layer Diagrams

```text
+-----------------------------------------------------------+
|          Web Application Frontend (Javascript)            |
+-----------------------------------------------------------+
|          W3C WebAuthn API (navigator.credentials)         |
+-----------------------------------------------------------+
|          Browser Security Engine (Origin Checking)        |
+-----------------------------------------------------------+
|          CTAP2 Protocol Translation Layer                 |
+-----------------------------------------------------------+
|          Platform Authenticator (TPM 2.0 / Secure Enclave)|
+-----------------------------------------------------------+
```

---

## 11. Packet Journey
The JSON payload generated by `navigator.credentials.get()` and returned to user JavaScript:

```json
{
  "id": "ARu8...",
  "rawId": "ARu8...",
  "type": "public-key",
  "response": {
    "authenticatorData": "SZYN5Yg...",
    "clientDataJSON": "eyJ0eXBlIjoid2ViYXV0aG4uZ2V0IiwiY2hhbGxlbmdlIjoi...",
    "signature": "MEYCIQ...",
    "userHandle": "dXNlcjEyMw=="
  }
}
```

---

## 12. Memory Journey
1. The browser parses the challenge string from the HTTP response into a buffer in heap memory.
2. The browser forwards the challenge buffer to the OS security service (e.g. Windows Hello or Apple LocalAuthentication daemon).
3. The security daemon coordinates with the hardware cryptoprocessor, loading the targeted private key into the Secure Enclave's protected memory bounds.
4. Once biometrics pass, the hardware registers complete the ECDSA operation, and the memory structures containing the private key are zeroized.

---

## 13. CPU Perspective
The CPU acts as a traffic controller, delegating cryptographic operations to hardware security chips (TPMs/Secure Enclaves) using memory-mapped I/O (MMIO). This ensures the CPU cores never see or process the raw private keys directly.

---

## 14. Kernel Perspective
The kernel intercepts biometric scans (fingerprint/camera) using dedicated device drivers. It routes this data to isolated secure hardware channels, preventing user-space malware from sniffing or hijacking raw biometric images.

---

## 15. Cryptographic Perspective
WebAuthn relies on standard asymmetric algorithms:
* **Ed25519** (Edwards-curve Digital Signature Algorithm) or **secp256r1** (ECDSA).
The public key is stored on the server's database, while the private key is generated and stored locally in hardware, rendering database leaks harmless.

---

## 16. Performance Characteristics
* **Local Biometric Match**: $< 100$ ms.
* **Server Verification Math**: $< 5$ ms (highly efficient ECDSA signature validation).

---

## 17. Tradeoffs
* **Device Portability**: If you register your laptop's built-in fingerprint reader, you cannot log in from a desktop PC without setting up a secondary roaming key (USB key or phone proxy).

---

## 18. Failure Modes
* **TPM Corruption**: If the device's hardware TPM malfunctions or is cleared during an OS reinstall, the local private keys are permanently destroyed.
* **Server Challenge Expiry**: If the server's challenge times out before the user touches their biometric sensor, authentication fails.

---

## 19. Attack Surface
* **Credential Sharing Vulnerabilities**: A malware utility on the user's host machine could display a fake browser page to trick the user into authenticating a session request initiated by an attacker.
* **Biometric Forgery**: High-resolution copies of fingerprints or 3D face masks could bypass biometric sensors on weak devices.

---

## 20. Defenses
* **User Verification Requirement**: Force the `userVerification` option to `required` on the server, enforcing a PIN or biometric check instead of a simple button tap.
* **Origin Checking**: The server must strictly validate that the `origin` parameter inside the signed `clientDataJSON` matches its official scheme and domain.

---

## 21. Real-World Examples
GitHub, Microsoft accounts, and Okta support passwordless logins using WebAuthn (Windows Hello / FaceID).

---

## 22. Production Architectures

```text
                       ┌─────────────────────────┐
                       │   Reverse Proxy Server  │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │   Auth Service Backend  │
                       └────────────┬────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌──────────────────────┐                          ┌──────────────────────┐
│  WebAuthn Creds DB   │                          │  Redis Cache         │
│  (Saves Public Keys  │                          │  (Stores Active      │
│   and Credential IDs)│                          │   Challenges)        │
└──────────────────────┘                          └──────────────────────┘
```

---

## 23. Reverse Engineering Perspective
By hooking the WebAuthn API functions in the browser console (e.g. overriding `navigator.credentials.get`), you can intercept raw challenge data and modify options dynamically to see how the target web application handles validation errors.

---

## 24. Observe It Yourself
Inspect WebAuthn credentials in Google Chrome:
1. Open Developer Tools (F12).
2. Click the three dots -> More Tools -> **WebAuthn**.
3. You can set up a virtual authenticator to test registrations and authentications offline without using physical hardware.

---

## 25. Common Misconceptions
> **Myth**: "WebAuthn sends your fingerprint data to the server."
> 
> **Reality**: No. The biometrics are processed completely locally. The server only receives a digital signature proving that the local biometric check succeeded.

---

## 26. Related Concepts
* **Passkeys**: A synchronization mechanism built on top of WebAuthn.
* **FIDO2**: The umbrella standard combining WebAuthn and CTAP2.

---

## 27. Deep Dives
Why WebAuthn prevents user tracking:
During registration, the authenticator generates a unique credential ID and key pair. Since the key is generated dynamically using domain hashes, two different sites (e.g. `siteA.com` and `siteB.com`) receive completely separate public keys, preventing them from linking accounts back to the same device.

---

## 28. Case Studies
* **Cloudflare's WebAuthn Transition (2021)**: Cloudflare migrated all employees from physical U2F tokens and soft MFA to WebAuthn-based passwordless credentials, reducing login friction and eliminating SMS and application-based phishing attacks.

---

## 29. Interview Questions
* **Q**: "What is the difference between User Presence (UP) and User Verification (UV) in WebAuthn?"
* **A**: User Presence (UP) verifies that a human is physically present (e.g., a simple button touch on a USB key). User Verification (UV) validates the *specific* user's identity (e.g., a fingerprint scan, face recognition, or entering a local device PIN).

---

## 30. Mini Projects
* **Passwordless WebAuthn Server**: Write a python Flask application using the `py_webauthn` library to register and log in users using their browser's built-in WebAuthn APIs.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Chapter 30: Passkeys

## 1. Why Does This Exist?
Traditional WebAuthn keys are tied to a single physical device (like a specific laptop's TPM). If a user upgraded their laptop or lost their smartphone, they lost all their credentials, making recovery difficult for non-technical users.
**Passkeys** solve this by enabling secure synchronization of FIDO credentials across a user's cloud accounts (like Apple iCloud, Google Password Manager, or 1Password), making passwordless credentials portable while maintaining protection against phishing.

---

## 2. Historical Context
The FIDO Alliance introduced the concept of "synced credentials" in 2022 under the marketing term **Passkeys**. Operating system vendors (Apple, Google, Microsoft) integrated this directly into their platforms, enabling passkey synchronization across ecosystems.

---

## 3. Problems Before It Existed
* **Device Lock-In**: WebAuthn keys locked users to a single device, creating friction when upgrading hardware.
* **Account Recovery**: Losing a device meant relying on insecure recovery fallbacks like email links or SMS codes.
* **Lack of Portability**: Logging in to a website on a Smart TV or public computer using a laptop key was difficult.

---

## 4. Ground Reality
A Passkey is a synchronized FIDO credential.
* **Storage**: The private keys are encrypted and stored in the user's cloud keychain.
* **Cross-Device authentication**: The protocol uses Bluetooth and QR codes to allow a phone to act as a roaming authenticator for other nearby devices (e.g., logging in to a desktop computer using a smartphone).

---

## 5. Mental Models
Think of a passkey as a **virtual signature stamp stored in a secure cloud safe**:
* You can access the safe from any of your personal devices (Phone, Tablet, Laptop) by verifying your biometrics.
* The signature stamp is origin-bound, refusing to stamp anything that doesn't match the target website.
* If you lose a device, you do not lose the stamp, because you can retrieve it by logging into your cloud account on a new device.

---

## 6. Analogies
A Passkey is like a **notary passport that syncs to your personal cloud**:
* When you buy a new phone, the cloud securely downloads the notary registry.
* When you want to log in on a computer, the computer displays a QR code.
* You scan the QR code with your phone. The devices use Bluetooth to confirm they are physically close, and the phone signs the login request.

---

## 7. Internal Machinery
Cross-device authentication (hybrid flow) steps:
1. The client browser displays a QR code containing a WebSocket URL and encryption keys.
2. The user scans the QR code with their mobile device.
3. The phone and the computer advertise local Bluetooth parameters to verify proximity.
4. They establish an encrypted tunnel over WebSockets.
5. The phone prompts for biometrics, signs the challenge, and returns the signature to the computer.

---

## 8. State Machines

```text
                  [ Cross-Device Passkey Flow ]
Client Desktop PC                                  User Smartphone
        │                                                 │
        │─── Show QR Code (WS URL + Key)                  │
        │                                                 │
        │◄── Scan QR Code with camera ────────────────────│
        │                                                 │
        │◄── Local proximity check (Bluetooth) ──────────►│
        │                                                 │
        │─── Send auth challenge via WS tunnel ──────────►│
        │                                                 │
        │                                                 ▼
        │                                          Verify Biometrics
        │                                          Sign Challenge
        │                                                 │
        │◄── Send cryptographic signature ────────────────│
        │
        ▼
   Log In Client
```

---

## 9. Sequence Diagrams

```text
User                      Desktop Browser             User Phone               Server Backend
 │                            │                            │                         │
 │─── Click "Login with P" ──►│                            │                         │
 │                            │─── Request challenge ───────────────────────────────►│
 │                            │◄── Return challenge ─────────────────────────────────│
 │                            │                            │                         │
 │                            │─── Display QR Code ────────│                         │
 │─── Scans QR Code with phone ───────────────────────────►│                         │
 │                            │◄── Bluetooth Proximity Ping│                         │
 │                            │─── Bluetooth Pong ─────────│                         │
 │                            │                            │─── Authenticate User    │
 │                            │                            │─── Sign Challenge       │
 │                            │◄── Forward Signature ──────│                         │
 │                            │─── Submit Signature to Server ──────────────────────►│
 │                            │◄── Auth Success ─────────────────────────────────────│
```

---

## 10. Layer Diagrams

```text
+-----------------------------------------------------------+
|               Web Application Client (HTML5 / JS)         |
+-----------------------------------------------------------+
|               OS Passkey Manager (iCloud / Google / MS)   |
+-----------------------------------------------------------+
|               Bluetooth Core Driver / WebSocket Tunnel    |
+-----------------------------------------------------------+
|               Hardware Secure Enclave / Secure Element    |
+-----------------------------------------------------------+
```

---

## 11. Packet Journey
The ClientDataJSON structure signed by the smartphone passkey during hybrid flow:

```json
{
  "type": "webauthn.get",
  "challenge": "dGVzdC1jaGFsbGVuZ2U...",
  "origin": "https://identity.vault.com",
  "androidPackageName": null
}
```

---

## 12. Memory Journey
1. The browser initiates key-sync verification routines, loading encrypted cloud blobs into RAM.
2. The user unlocks their cloud keychain password, decrypting the key blobs in isolated memory pages marked as non-swappable.
3. The cryptographic engine uses the decrypted private key to sign the challenge, immediately releasing the raw key memory allocation back to the system allocator.

---

## 13. CPU Perspective
The CPU manages encryption and decryption of passkey synchronization synchronization loops, executing AES-GCM decryption operations over encrypted cloud sync payloads before utilizing local hardware acceleration interfaces.

---

## 14. Kernel Perspective
During proximity checks, the kernel's Bluetooth stack starts advertising and scanning for specific local Bluetooth Low Energy (BLE) service UUIDs, checking signal strength (RSSI) parameters to ensure physical proximity before completing key sharing actions.

---

## 15. Cryptographic Perspective
Passkeys use public-key cryptography (typically **secp256r1** or **Ed25519**). Cloud synchronization uses end-to-end encryption (E2EE), meaning the cloud provider (Apple, Google) cannot read the private keys because the decryption key is derived from the user's local device passcode.

---

## 16. Performance Characteristics
* **Local Handshake Latency**: 2 - 5 seconds for QR scanning and Bluetooth proximity checks.
* **Network Overhead**: Minimal. Encrypted assertions are less than 1KB.

---

## 17. Tradeoffs
* **Trust in Sync Provider**: Relies on the security of the user's Apple/Google account and cloud keychains.
* **Proximity Requirement**: Bluetooth must be enabled on both devices for cross-device authentication.

---

## 18. Failure Modes
* **Bluetooth Disabled**: If Bluetooth is turned off on either the phone or the computer, cross-device authentication fails.
* **Cloud Sync Outage**: If the cloud synchronization service is down, newly created passkeys are not available on other devices.

---

## 19. Attack Surface
* **Cloud Account Takeover**: If an attacker gains access to the user's Google or Apple account and bypasses recovery security, they can download the synced keychain and access the decrypted passkeys.
* **Phishing QR Code Attacks**: Attackers display a phishing login page with their own QR code to hijack a session when the victim scans it.

---

## 20. Defenses
* **Hardware-Bound Passkeys**: For high-security environments, servers can set the `attestation` parameter to reject synced credentials and require hardware-bound tokens.
* **Proximity Checks (BLE)**: The protocol requires Bluetooth proximity verification to prevent remote attackers from using QR codes to hijack credentials.

---

## 21. Real-World Examples
Apple iCloud Keychain, Google Password Manager, and Dashlane support Passkey creation and synchronization.

---

## 22. Production Architectures

```text
  Desktop Browser                 Cloud Directory                 User Smartphone
         │                               │                               │
         │                               │◄─── Sync Encrypted Keys ──────│
         │                               │                               │
         │◄─── Download Encrypted Keys ──│                               │
         │                                                               │
  [ Unlocks locally via PIN/Face ]                                [ Unlocks via Fingerprint ]
```

---

## 23. Reverse Engineering Perspective
Analyzing the `navigator.credentials.create` configuration parameters reveals the key request options:
* `authenticatorAttachment: "platform"` forces the use of a built-in device passkey manager.
* `residentKey: "required"` ensures the credential is saved to the device database so it can be looked up automatically by username.

---

## 24. Observe It Yourself
Inspect passkeys saved on a local macOS device:
1. Open **System Settings** -> **Passwords**.
2. Search for registered websites. You can view, share, or delete saved passkeys.

---

## 25. Common Misconceptions
> **Myth**: "Passkeys are less secure than normal WebAuthn because they are stored in the cloud."
> 
> **Reality**: Passkeys are encrypted end-to-end using keys derived from local device passcodes. The cloud provider cannot decrypt them, ensuring they remain protected against database breaches.

---

## 26. Related Concepts
* **Passkey Providers**: Third-party managers like 1Password and Bitwarden.
* **Hybrid Flow**: The cross-device authentication protocol.

---

## 27. Deep Dives
FIDO Hybrid protocol proximity checking:
The hybrid flow uses BLE (Bluetooth Low Energy) advertising. The computer and phone do not pair; instead, they broadcast random values encrypted with a key derived from the scanned QR code. This checks that both devices are in the same room without exposing identifying information.

---

## 28. Case Studies
* **PayPal's Passkey Deployment (2022)**: PayPal became one of the first payment providers to deploy Passkey login options, reporting lower login times and reduced support requests compared to traditional password/OTP configurations.

---

## 29. silly Questions
* **Q**: "Why does a Passkey require Bluetooth when logging in to a desktop computer from a mobile phone?"
* **A**: Bluetooth is used as a proximity check to verify the phone is in the same physical room as the computer, preventing remote attackers from using phished QR codes to hijack sessions.

---

## 30. Mini Projects
* **Passkey Registration Simulator**: Build a mock dashboard page in JavaScript and Python that implements the registration and login assertions for synced passkeys.




