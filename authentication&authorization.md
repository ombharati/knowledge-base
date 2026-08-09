# Authentication and Authorization

## Learning Path

This topic will progress through:

1. Prerequisites and vocabulary
2. Core mental model
3. Trust boundaries and architecture
4. End-to-end login and request flow
5. Credentials and authentication factors
6. Password authentication internals
7. Sessions, cookies, and tokens
8. Authorization models
9. OAuth 2.0, OpenID Connect, and SAML
10. Operating-system authentication and authorization
11. Infrastructure identities and service-to-service authentication
12. Practical tools and debugging
13. Security and performance
14. Advanced internals

This part covers sections 1–4.

---

# 1. Prerequisites

Authentication and authorization sit at the intersection of several systems concepts.

You do not need to master all of these first, but you should understand their role.

## 1.1 Client-server communication

A **client** sends a request to a **server**.

Examples:

* A browser sends an HTTP request to a web application.
* `ssh` connects to an SSH server.
* A Kubernetes workload calls another service.
* A process asks the Linux kernel to open a file.

```mermaid
flowchart LR
    C[Client] -->|Request| S[Server]
    S -->|Response| C
```

Authentication answers:

> Which identity is making this request?

Authorization answers:

> Is that identity permitted to perform this operation?

The server may answer both questions itself, or delegate some of the work to another component.

---

## 1.2 HTTP requests and responses

Many authentication systems operate through HTTP.

A simplified HTTP request:

```http
GET /admin/users HTTP/1.1
Host: app.example.com
Cookie: session_id=abc123
```

A simplified response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"users": [...]}
```

Authentication information may appear in:

* Cookies
* The `Authorization` header
* Client certificates
* Request signatures
* Application-specific headers

Example bearer token:

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

> A bearer token is usually usable by whoever possesses it. “Bearer” effectively means “the holder may present this token.”

---

## 1.3 Cryptographic building blocks

Authentication systems frequently depend on cryptography, but different mechanisms solve different problems.

| Mechanism                | Main purpose                            |       Reversible? | Typical use                            |
| ------------------------ | --------------------------------------- | ----------------: | -------------------------------------- |
| Hash function            | Produce a fixed-size digest             |                No | Integrity checks                       |
| Password hash/KDF        | Make password guessing expensive        |                No | Password storage                       |
| Symmetric encryption     | Hide data using one shared key          |               Yes | Encrypting stored or transmitted data  |
| Asymmetric encryption    | Encrypt using a public/private key pair |               Yes | Key exchange and specialized protocols |
| Digital signature        | Prove data was signed by a private key  | Verification only | Signed tokens and certificates         |
| MAC/HMAC                 | Prove integrity using a shared secret   | Verification only | Cookie and request signing             |
| Random number generation | Produce unpredictable values            |    Not applicable | Sessions, nonces, keys                 |

Important distinction:

* **Encryption** provides confidentiality.
* **Hashing** produces a digest.
* **Signing** provides integrity and origin authentication.
* **Password hashing** deliberately consumes time and memory to slow brute-force attacks.

---

## 1.4 TLS

TLS protects data moving between two endpoints.

For HTTPS, TLS usually provides:

* Encryption
* Message integrity
* Server authentication through certificates
* Optional client authentication

```mermaid
flowchart LR
    B[Browser] == "Encrypted TLS connection" ==> W[Web server]
```

TLS does **not** automatically authenticate the human user.

It normally authenticates the server to the client:

> “This endpoint controls the private key corresponding to the certificate for `app.example.com`.”

The application still needs a mechanism such as:

* Password login
* Passkey
* Client certificate
* Federated identity
* Session cookie

---

## 1.5 Databases and state

Authentication systems usually maintain state somewhere.

Examples:

* User records
* Password hashes
* MFA configuration
* Active sessions
* Revoked tokens
* Roles and permissions
* Audit events

A system might store this state in:

* SQL database
* LDAP directory
* Redis
* Operating-system account database
* Identity provider
* Hardware security module
* Cloud identity service

Understanding where the authoritative state lives is critical for debugging.

---

## 1.6 Time

Time affects authentication more than it first appears.

It is used for:

* Token expiry
* Certificate validity
* Time-based one-time passwords
* Session timeout
* Replay prevention
* Audit ordering
* Temporary credentials

If clocks disagree, authentication can fail even when credentials are otherwise valid.

Examples:

* “Token is not valid yet.”
* “Token expired.”
* TOTP code rejected.
* Certificate considered expired.
* Signed request timestamp outside the permitted window.

Infrastructure commonly synchronizes time using NTP or another time service.

---

## 1.7 Processes and operating-system identity

A process normally executes under an operating-system identity.

On Linux, important identifiers include:

* Real user ID
* Effective user ID
* Real group ID
* Effective group ID
* Supplementary groups

The kernel uses these identities when deciding whether a process may:

* Open a file
* Bind to a privileged port
* Send a signal
* Trace another process
* Mount a filesystem
* Access a device

Application authentication and operating-system authentication are related, but they are not the same system.

A web application may authenticate a user named `alice`, while the application process itself runs as Linux user `www-data`.

```mermaid
flowchart TD
    H[Human user: Alice]
    B[Browser process]
    W[Web application process: www-data]
    K[Kernel]
    F[Protected file]

    H --> B
    B -->|HTTP request authenticated as Alice| W
    W -->|open system call as UID www-data| K
    K -->|OS authorization decision| F
```

Two authorization decisions may therefore occur:

1. The web application decides whether Alice may request the file.
2. The kernel decides whether the `www-data` process may open the file.

---

## Questions: Prerequisites

1. Why does HTTPS server authentication not automatically authenticate the user sitting behind the browser?
2. How could incorrect system time break both TOTP and signed access tokens?
3. A web application authorizes Alice to download a report, but the kernel returns `EACCES` when the application opens the file. Which two identity systems are involved?
4. Why is encrypting passwords in a database generally the wrong storage model?
5. Which security property is missing if a message is encrypted but not protected against modification?
6. Why must session identifiers come from a cryptographically secure random-number generator?

---

# 2. Core Mental Model

## 2.1 The simplest model

A protected operation can be represented as:

```text
Subject attempts Action on Resource
```

Examples:

```text
Alice attempts READ on payroll-report.pdf
Service A attempts WRITE on queue orders
Process 4182 attempts OPEN on /etc/shadow
Administrator attempts DELETE on virtual-machine-17
```

The system must determine:

1. Who or what is the subject?
2. What evidence supports that identity?
3. What action is being requested?
4. Which resource is targeted?
5. What policy applies?
6. What decision should be enforced?

---

## 2.2 Authentication

**Authentication** is the process of establishing or verifying an identity.

It answers:

> Who is this subject?

Examples:

* Verifying a password
* Validating an SSH signature
* Checking a client TLS certificate
* Validating an identity token
* Verifying a passkey assertion
* Accepting a Kerberos service ticket

Authentication does not inherently grant access.

A user can be successfully authenticated but still be prohibited from accessing a resource.

---

## 2.3 Authorization

**Authorization** determines whether a subject may perform an action on a resource.

It answers:

> Is this subject permitted to do this?

Authorization may consider:

* User identity
* Group membership
* Role
* Resource ownership
* Requested action
* Device state
* Network location
* Time
* Authentication strength
* Risk score
* Tenant or organization
* Data classification

Example:

```text
Subject: Alice
Action: DELETE
Resource: Project 827
Context:
  role = project_editor
  project_owner = Bob
  authentication_method = password
Decision: DENY
Reason: project_editor cannot delete projects
```

---

## 2.4 Authentication and authorization are separate decisions

```mermaid
flowchart LR
    R[Incoming request]
    A{Authenticated?}
    Z{Authorized?}
    P[Perform operation]
    U[Return 401 or login challenge]
    F[Return 403 or access denied]

    R --> A
    A -->|No| U
    A -->|Yes| Z
    Z -->|No| F
    Z -->|Yes| P
```

The HTTP distinction is commonly:

| Status             | Typical meaning                        |
| ------------------ | -------------------------------------- |
| `401 Unauthorized` | Authentication is missing or invalid   |
| `403 Forbidden`    | Identity is known but access is denied |

The name `401 Unauthorized` is historically confusing. In practice, it normally indicates an **authentication** problem.

Implementations do not always follow this distinction consistently. Some applications return:

* `401` for every access failure
* `403` for unauthenticated users
* `404` to avoid revealing whether a resource exists
* `302` redirects to a login page

Therefore, status codes provide clues, not complete proof.

---

## 2.5 Identity, principal, subject, and account

These words are related but not perfectly interchangeable.

### Identity

An **identity** is a representation of an entity within a system.

Examples:

* `alice@example.com`
* Linux UID `1001`
* Kubernetes service account `payments/api`
* AWS IAM role `ProductionBackupRole`
* Certificate subject or SPIFFE ID

### Principal

A **principal** is an entity that can be authenticated or assigned permissions.

Examples:

* Human user
* Service account
* Workload
* Device
* Process
* API client

### Subject

The **subject** is the principal currently participating in a security decision.

In a request:

```text
Principal represented by the request = subject
```

The term appears frequently in:

* X.509 certificates
* OAuth/OIDC claims
* Access-control systems
* Operating-system security

### Account

An **account** is a system record representing an identity.

It might contain:

* Identifier
* Profile information
* Password hash
* Group membership
* Account state
* MFA configuration
* Recovery methods

An identity may exist without a conventional account. For example, an external identity provider may authenticate a user who is represented locally only by a generated mapping.

---

## 2.6 Credentials

A **credential** is evidence used to support an identity claim.

Examples:

* Password
* Private key
* Client certificate
* Session cookie
* Access token
* Kerberos ticket
* Passkey
* One-time password

A credential is not always entered directly by a human.

For example:

1. The user enters a password.
2. The server verifies it.
3. The server issues a session identifier.
4. The browser sends the session identifier on later requests.
5. The session identifier becomes the credential for those requests.

This distinction matters:

* The password authenticates the login attempt.
* The session cookie authenticates subsequent requests.

---

## 2.7 Claims and attributes

A **claim** is a statement about a subject.

Examples:

```text
subject_id = 4f91a
email = alice@example.com
department = finance
role = manager
authentication_time = 2026-07-31T09:15:00Z
authentication_method = passkey
```

An **attribute** is a property associated with an identity or resource.

Claims are often carried in tokens, while attributes may be retrieved from directories or databases.

Do not assume a claim is trustworthy merely because it exists.

The system must know:

* Who issued it
* Whether it was modified
* Whether the issuer is trusted
* Whether it is intended for this recipient
* Whether it is still valid
* Whether it is relevant to the decision

---

## 2.8 Authentication event versus authenticated session

A login is usually an event.

A session is continuing state derived from that event.

```mermaid
flowchart LR
    L[Login authentication]
    S[Session established]
    R1[Request 1]
    R2[Request 2]
    R3[Request 3]

    L --> S
    S --> R1
    S --> R2
    S --> R3
```

Authentication may happen once, while authorization happens many times.

Example:

1. Alice authenticates at 09:00.
2. A session is created.
3. At 09:10, Alice reads a document.
4. At 09:20, Alice attempts to delete it.
5. At 09:30, Alice changes her password.

Each operation may require a new authorization decision.

Some operations may also require **reauthentication** or **step-up authentication**.

Example:

```text
Reading profile: existing session is sufficient
Changing email: password confirmation required
Transferring money: passkey or MFA required
```

---

## 2.9 Authentication, authorization, and accounting

A common infrastructure model is **AAA**:

| Component      | Question         |
| -------------- | ---------------- |
| Authentication | Who are you?     |
| Authorization  | What may you do? |
| Accounting     | What did you do? |

Accounting includes:

* Audit logs
* Session start and end
* Commands executed
* Data accessed
* Policy decisions
* Administrative changes
* Authentication failures

Accounting is not financial accounting in this context. It means security-relevant recording and traceability.

---

## 2.10 Policy decision and policy enforcement

A useful architectural distinction is:

* **Policy Decision Point**, or PDP
* **Policy Enforcement Point**, or PEP
* **Policy Information Point**, or PIP
* **Policy Administration Point**, or PAP

### Policy Decision Point

Evaluates the policy and returns a decision.

Example:

```text
ALLOW
DENY
ALLOW WITH CONDITIONS
```

### Policy Enforcement Point

Intercepts the operation and enforces the decision.

Examples:

* API gateway
* Web application middleware
* Linux kernel
* Database engine
* Kubernetes API server
* Reverse proxy

### Policy Information Point

Provides data needed for the decision.

Examples:

* Identity directory
* Group database
* Device inventory
* Resource metadata
* Risk engine

### Policy Administration Point

Where policies are created and managed.

Examples:

* IAM management console
* Configuration repository
* Kubernetes RBAC objects
* Policy-as-code repository

```mermaid
flowchart LR
    S[Subject]
    PEP[Policy Enforcement Point]
    PDP[Policy Decision Point]
    PIP[Policy Information Point]
    PAP[Policy Administration Point]
    R[Resource]

    S -->|Requested operation| PEP
    PEP -->|Decision request| PDP
    PDP -->|Fetch attributes| PIP
    PIP -->|Attributes| PDP
    PAP -->|Policies| PDP
    PDP -->|Allow or deny| PEP
    PEP -->|Allowed operation| R
```

In a small application, these may all be implemented in the same process.

In a large system, they may be separate services.

---

## 2.11 Authentication is a chain of trust

A server rarely learns a user’s real-world identity directly.

Instead, it trusts a chain of evidence.

Example:

```text
Application trusts identity provider
Identity provider trusts passkey verification
Passkey verifier trusts a registered public key
Public key was enrolled under an account
Application maps account identifier to local permissions
```

The final authorization decision is only as reliable as the chain supporting it.

Questions to ask:

1. Which component originally authenticated the subject?
2. Which component issued the credential?
3. Which component validated it?
4. Why does the application trust that issuer?
5. How is the external identity mapped to a local principal?
6. Where are permissions obtained?
7. What happens when identity state changes?

---

## 2.12 Simplified model versus exact implementation

### Simplified mental model

```text
User proves identity
Server creates session
User sends session
Server checks permissions
Server performs operation
```

### More exact implementation

```text
Human
  ↓ enters evidence
User agent
  ↓ sends protocol message
Authentication endpoint
  ↓ reads identity record
Credential verifier
  ↓ evaluates password/key/factor
Session or token issuer
  ↓ creates security context
Client stores credential
  ↓ sends credential with request
Gateway or middleware
  ↓ validates credential
Application resolves local subject
  ↓ loads policy inputs
Authorization engine
  ↓ returns decision
Application and operating system
  ↓ access downstream resources
Audit system records event
```

The simplified model is useful for reasoning.

The exact model is necessary for:

* Security design
* Troubleshooting
* Incident response
* Performance analysis
* Correct logout and revocation
* Distributed-system architecture

---

## Questions: Core Mental Model

1. Why must authorization be evaluated after authentication rather than inferred from successful authentication?
2. A user’s role is removed while they have an active session. Which parts of the architecture determine whether the change takes effect immediately?
3. Why can a session cookie be considered a credential even though the user never types it?
4. What is the difference between trusting a claim and merely decoding a claim?
5. Where should enforcement occur if an application has both an API gateway and direct internal network paths to the backend?
6. Why might returning `404` instead of `403` reduce information leakage?
7. In a distributed system, what could happen if the policy decision point is unavailable but the enforcement point is still accepting traffic?
8. Which system component should record the policy reason for a denial: the PDP, the PEP, or both?

---

# 3. Trust Boundaries and Architecture

## 3.1 What is a trust boundary?

A **trust boundary** is a point where data moves between components with different trust assumptions.

Examples:

* Browser to web server
* Public network to reverse proxy
* Application to identity provider
* Container to host kernel
* Workload to cloud metadata service
* API gateway to backend service
* Employee device to corporate network
* One tenant to another tenant

Crossing a trust boundary should trigger questions such as:

* Is the sender authenticated?
* Is the channel protected?
* Can the message be replayed?
* Can the data be modified?
* Is the receiver correct?
* Is input being mistaken for trusted metadata?
* Does the recipient have excessive authority?

---

## 3.2 Authentication does not remove the trust boundary

Suppose a request contains:

```http
X-User: alice
X-Role: admin
```

These headers are not trustworthy merely because they contain identity information.

A client could send them directly.

They become trustworthy only when architecture ensures that:

1. A trusted proxy authenticates the user.
2. The proxy removes client-supplied copies of those headers.
3. The proxy writes its own verified identity headers.
4. The backend only accepts traffic from the proxy.
5. The channel from proxy to backend is protected appropriately.

```mermaid
flowchart LR
    C[Untrusted client]
    P[Trusted authentication proxy]
    A[Backend application]

    C -->|Untrusted headers| P
    P -->|Strip client identity headers| P
    P -->|Add verified identity headers| A
```

If the backend is directly reachable, an attacker may bypass the proxy and forge the headers.

This is a common architectural authorization failure.

---

## 3.3 Identity propagation

After authenticating a subject, systems often need to carry the identity through multiple components.

Example path:

```text
Browser
→ CDN
→ Load balancer
→ API gateway
→ Frontend service
→ Backend service
→ Database
```

Possible propagation methods include:

* Session cookie
* Bearer access token
* Signed internal token
* mTLS identity
* Request header added by a trusted proxy
* Kerberos delegation
* Workload identity
* Database connection identity

The core question is:

> At each hop, how does the receiving component know which identity is speaking?

---

## 3.4 End-user identity and service identity

A backend request may involve at least two identities:

1. The service making the network connection
2. The end user on whose behalf the service is acting

Example:

```text
Alice uses frontend-service
frontend-service calls payroll-service
```

The payroll service may need to know:

```text
Caller workload = frontend-service
End user = Alice
```

These identities answer different questions.

| Question                                             | Relevant identity      |
| ---------------------------------------------------- | ---------------------- |
| Is this connection from an approved service?         | Service identity       |
| May Alice view payroll record 71?                    | End-user identity      |
| Is frontend-service permitted to call this API?      | Service identity       |
| Is the request acting on Alice’s behalf?             | Delegated user context |
| Which component should appear in network audit logs? | Service identity       |
| Which user initiated the business operation?         | End-user identity      |

Do not collapse these identities into a single generic “authenticated” state.

---

## 3.5 Direct authentication versus delegated authentication

### Direct authentication

The target service verifies the credential itself.

Example:

```text
Client → SSH server
SSH server verifies client key
```

### Delegated authentication

The target service trusts another system to authenticate the subject.

Example:

```text
User → Identity provider
Identity provider → signed identity assertion
User → application with assertion
Application validates assertion
```

```mermaid
sequenceDiagram
    participant U as User
    participant I as Identity Provider
    participant A as Application

    U->>I: Authenticate
    I-->>U: Signed identity assertion
    U->>A: Present assertion
    A->>A: Validate issuer, signature, audience, time
    A-->>U: Application session
```

Delegation reduces the need for every application to handle passwords directly, but introduces dependencies:

* Issuer availability
* Signing-key distribution
* Identity mapping
* Claim interpretation
* Token expiry
* Revocation behavior
* Trust configuration

---

## 3.6 Centralized versus decentralized decisions

### Centralized authentication

A shared identity provider authenticates users.

Advantages:

* Consistent MFA policy
* Central account disabling
* Fewer applications handling passwords
* Central audit trail
* Easier federation

Risks and costs:

* High-value target
* Outage affects many applications
* Integration complexity
* Incorrect claims affect many services

### Decentralized authentication

Each application maintains its own users and credentials.

Advantages:

* Fewer runtime dependencies
* Application-specific control
* Potentially simpler for isolated systems

Risks and costs:

* Password duplication
* Inconsistent security
* Difficult account lifecycle management
* More recovery systems
* Fragmented auditing

Most large environments use some combination:

* Central human identity
* Distributed workload identities
* Local authorization policies
* Central policy or directory data

---

## 3.7 Reference architecture

```mermaid
flowchart TD
    U[User]
    B[Browser]
    RP[Reverse Proxy / Gateway]
    IDP[Identity Provider]
    APP[Application]
    AUTHZ[Authorization Engine]
    DIR[Identity Directory]
    DB[(Application Database)]
    AUDIT[(Audit System)]

    U --> B
    B -->|HTTPS request| RP
    RP -->|Authentication redirect or validation| IDP
    IDP -->|Identity assertion| RP
    RP -->|Authenticated request| APP
    APP -->|Policy query| AUTHZ
    AUTHZ -->|Identity attributes| DIR
    AUTHZ -->|Allow or deny| APP
    APP -->|Authorized data operation| DB

    IDP --> AUDIT
    RP --> AUDIT
    APP --> AUDIT
    AUTHZ --> AUDIT
```

Possible responsibilities:

| Component            | Responsibility                                         |
| -------------------- | ------------------------------------------------------ |
| Browser              | Collects credentials and stores session material       |
| Reverse proxy        | Terminates TLS, routes requests, may validate identity |
| Identity provider    | Authenticates users and issues assertions              |
| Application          | Implements business operations                         |
| Authorization engine | Evaluates policy                                       |
| Directory            | Stores users, groups, and attributes                   |
| Database             | Stores application resources and possibly permissions  |
| Audit system         | Records security-relevant events                       |

Real architectures vary. A component may perform several roles.

---

## 3.8 Control plane and data plane

A useful infrastructure distinction:

### Control plane

Configures identity and policy.

Examples:

* Create user
* Register public key
* Assign role
* Create access policy
* Rotate signing key
* Disable account
* Configure MFA requirements

### Data plane

Processes actual operations under those policies.

Examples:

* Validate an access token
* Check a permission
* Open a protected file
* Serve an API response
* Accept an SSH connection

```mermaid
flowchart LR
    ADMIN[Administrator]
    CP[Control Plane]
    POLICY[(Identity and Policy State)]
    DP[Data Plane]
    REQUEST[Request]

    ADMIN -->|Configure| CP
    CP --> POLICY
    REQUEST --> DP
    POLICY -->|Policy data| DP
```

Many propagation problems arise because control-plane changes do not reach the data plane immediately.

Example:

1. An administrator removes Alice from the `admin` group.
2. The directory updates immediately.
3. A gateway caches Alice’s groups for ten minutes.
4. An application trusts the gateway’s cached claim.
5. Alice remains effectively authorized until the cache expires.

---

## 3.9 Fail-open and fail-closed behavior

When an authentication or authorization dependency fails, the system must decide what to do.

### Fail closed

Deny the operation when a security decision cannot be made.

```text
Authorization service unavailable
→ deny request
```

Security benefit:

* Prevents unverified access.

Availability cost:

* Dependency failures can block legitimate users.

### Fail open

Allow the operation when the security decision cannot be made.

```text
Authorization service unavailable
→ allow request
```

Availability benefit:

* Service continues operating.

Security cost:

* Attackers may exploit decision-system failures.

Some systems use contextual strategies:

* Fail closed for writes
* Use cached decisions for low-risk reads
* Allow existing sessions but reject new logins
* Permit emergency administrative access through a separate mechanism
* Degrade to a restricted read-only mode

Fail-open behavior should be explicit, narrow, and audited.

---

## 3.10 Authentication state is not always binary

A subject may be:

* Unauthenticated
* Partially authenticated
* Authenticated with a weak method
* Authenticated with MFA
* Authenticated but required to reauthenticate
* Authenticated through a recovery method
* Authenticated from an unmanaged device
* Authenticated with elevated privileges
* Authenticated but suspended from sensitive operations

Therefore, a richer security context might contain:

```json
{
  "subject": "user-1937",
  "authenticated": true,
  "authentication_methods": ["password", "totp"],
  "authentication_time": "2026-07-31T09:15:00Z",
  "device_managed": false,
  "risk_level": "medium",
  "session_id": "s_71bb...",
  "privilege_level": "standard"
}
```

Authorization can then require more than `authenticated == true`.

Example:

```text
ALLOW wire_transfer
IF:
  authentication includes passkey or phishing-resistant MFA
  AND authentication age < 5 minutes
  AND device risk != high
```

---

## Questions: Trust Boundaries and Architecture

1. Why is a signed user token insufficient to authenticate the service forwarding that token?
2. What must a backend verify before trusting an `X-User` header from a reverse proxy?
3. How could an attacker bypass authentication if the backend is reachable directly?
4. Why might removing a user from a group fail to revoke access immediately?
5. When should a system distinguish the end-user identity from the calling workload identity?
6. What are the security and availability consequences of caching authorization decisions?
7. Which operations might reasonably use cached policy during a PDP outage, and which should always fail closed?
8. Why can centralized authentication simplify account management while increasing systemic risk?
9. How could a control-plane operation succeed while the data-plane behavior remains unchanged?
10. What additional context might an authorization policy require beyond the user identifier?

---

# 4. Concrete End-to-End Example

We will follow one example through the system:

> Alice signs in to an internal web application and attempts to download a financial report.

The system uses:

* HTTPS
* Username and password for initial authentication
* TOTP as a second factor
* Server-side session storage
* A browser session cookie
* Role- and resource-based authorization
* A reverse proxy
* An application server
* PostgreSQL
* Centralized audit logs

This is one possible architecture, not a universal design.

---

## 4.1 Components

```mermaid
flowchart LR
    A[Alice]
    B[Browser]
    P[Reverse Proxy]
    APP[Application]
    S[(Session Store)]
    U[(User Database)]
    R[(Report Database)]
    L[(Audit Logs)]

    A --> B
    B -->|HTTPS| P
    P --> APP
    APP --> U
    APP --> S
    APP --> R

    P --> L
    APP --> L
```

### Responsibility map

| Component       | Responsibility                                  |
| --------------- | ----------------------------------------------- |
| Alice           | Supplies authentication factors                 |
| Browser         | Sends requests and stores cookies               |
| Reverse proxy   | Terminates TLS and routes HTTP                  |
| Application     | Verifies login, creates sessions, checks access |
| User database   | Stores account and password-verification data   |
| Session store   | Maps session identifiers to security context    |
| Report database | Stores report metadata and data                 |
| Audit system    | Records authentication and authorization events |

---

# 4.2 Stage 1: Alice requests the application

Alice enters:

```text
https://finance.example.internal/reports
```

The browser performs several operations before authentication begins:

1. Resolves the hostname.
2. Establishes a TCP connection.
3. Performs a TLS handshake.
4. Validates the server certificate.
5. Sends the HTTP request.

Simplified request:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
User-Agent: ExampleBrowser/1.0
Accept: text/html
```

No session cookie exists yet.

---

## Component handling

| Stage                | Component                 | Data involved                         |
| -------------------- | ------------------------- | ------------------------------------- |
| Name resolution      | DNS resolver              | Hostname → IP address                 |
| Connection           | OS networking stack       | Source and destination socket         |
| TLS                  | Browser and reverse proxy | Certificates, keys, encrypted channel |
| HTTP routing         | Reverse proxy             | Host and path                         |
| Authentication check | Application               | Cookie or token state                 |

---

## Application decision

The application checks for authentication material.

Pseudocode:

```python
session_id = request.cookies.get("session_id")

if session_id is None:
    return redirect("/login")
```

The application does not yet know that the user is Alice.

It only knows:

```text
Request has no acceptable session credential
```

Simplified response:

```http
HTTP/1.1 302 Found
Location: /login
Cache-Control: no-store
```

The browser follows the redirect.

---

## 4.3 Stage 2: The login form is delivered

Request:

```http
GET /login HTTP/1.1
Host: finance.example.internal
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Cache-Control: no-store
Set-Cookie: login_csrf=RANDOM_VALUE; Secure; HttpOnly; SameSite=Lax

<form method="post" action="/login">
  <input name="username">
  <input name="password" type="password">
  <input type="hidden" name="csrf_token" value="RANDOM_VALUE">
  <button type="submit">Sign in</button>
</form>
```

The application creates a CSRF token.

**CSRF**, or Cross-Site Request Forgery, is an attack where another site causes a browser to send an unintended authenticated request.

At this stage, the user is not authenticated, but protecting the login flow from CSRF can still matter. Login CSRF may cause a victim’s browser to become logged into an attacker-controlled account.

---

## Data state after this stage

Browser:

```text
login_csrf cookie = random value
rendered form contains matching CSRF value
session_id = absent
```

Server:

```text
May store CSRF state, depending on implementation
No authenticated user session exists
```

---

# 4.4 Stage 3: Alice submits username and password

Alice enters:

```text
Username: alice@example.com
Password: [secret value]
```

The browser sends:

```http
POST /login HTTP/1.1
Host: finance.example.internal
Content-Type: application/x-www-form-urlencoded
Cookie: login_csrf=RANDOM_VALUE
Origin: https://finance.example.internal

username=alice%40example.com&password=...&csrf_token=RANDOM_VALUE
```

TLS encrypts this request while it travels between the browser and the TLS endpoint.

TLS does not prevent:

* Malicious browser extensions from reading the page
* Malware on the endpoint
* Application logging of the password
* Server-side memory exposure
* Phishing on another domain
* Compromise after TLS termination

---

## 4.5 Stage 4: Request validation

Before checking the password, the application should validate the request itself.

Possible checks:

1. Is the HTTP method allowed?
2. Is the request body within the size limit?
3. Is the content type expected?
4. Does the CSRF token match?
5. Is the `Origin` header acceptable?
6. Is the source subject to rate limiting?
7. Is the account identifier syntactically valid?
8. Has the request exceeded login-attempt limits?

Pseudocode:

```python
if not csrf_valid(request):
    audit("login_rejected", reason="invalid_csrf")
    return error(400)

if not login_rate_limit.allow(request.client_ip, request.username):
    audit("login_rejected", reason="rate_limit")
    return error(429)
```

The order matters.

For example, querying the user database before applying basic request limits may allow an attacker to consume database resources cheaply.

---

## 4.6 Stage 5: Account lookup

The application normalizes and looks up the username.

Conceptually:

```sql
SELECT
    user_id,
    username,
    password_hash,
    totp_enabled,
    account_status
FROM users
WHERE normalized_username = $1;
```

Parameterization is essential.

The application should not construct SQL by concatenating user input.

Unsafe conceptual pattern:

```python
query = "SELECT * FROM users WHERE username = '" + username + "'"
```

Safer pattern:

```python
row = database.execute(
    "SELECT user_id, password_hash FROM users WHERE normalized_username = %s",
    (normalized_username,)
)
```

Actual placeholder syntax varies by database client.

---

## Account lookup result

Illustrative internal record:

```text
user_id: u_1048
normalized_username: alice@example.com
password_hash: $argon2id$v=19$m=65536,t=3,p=1$...
totp_enabled: true
account_status: active
```

The password itself is not stored.

The record contains a password hash generated using a password hashing algorithm such as Argon2id, bcrypt, scrypt, or PBKDF2.

Their exact suitability depends on implementation, environment, policy, and configuration. Password hashing will be covered later.

---

## 4.7 Handling unknown users

Suppose the username does not exist.

A naïve application might respond:

```text
Unknown username
```

For an existing user with a wrong password:

```text
Incorrect password
```

This lets attackers enumerate valid accounts.

A more uniform external response is:

```text
Invalid username or password
```

Internally, the application may still record a specific reason.

It may also perform a dummy password-hash computation for unknown users to reduce timing differences.

This does not make timing perfectly identical, but it avoids an obvious fast path.

---

# 4.8 Stage 6: Password verification

The application passes:

* The password entered by Alice
* The stored encoded password hash

to a password-verification function.

Conceptually:

```python
valid = password_hasher.verify(
    stored_hash,
    submitted_password
)
```

The verifier:

1. Parses the stored algorithm identifier.
2. Extracts parameters.
3. Extracts the salt.
4. Runs the password-hashing algorithm on the submitted password.
5. Compares the computed result with the stored result.
6. Uses a comparison method designed to avoid obvious timing leakage.

The application should not manually reimplement this process.

---

## Password-verification result

Possible internal state:

```text
account_exists = true
password_valid = true
account_active = true
second_factor_required = true
```

Alice is not yet fully authenticated because the policy requires TOTP.

This is a **partial authentication state**.

The application must securely bind the completed password check to the upcoming MFA step.

---

# 4.9 Stage 7: Temporary authentication transaction

The server creates a temporary transaction:

```text
transaction_id = tx_73c1...
user_id = u_1048
password_verified = true
created_at = 2026-07-31T09:15:00Z
expires_at = 2026-07-31T09:20:00Z
mfa_attempts = 0
status = pending_totp
```

The browser receives a temporary cookie:

```http
HTTP/1.1 302 Found
Location: /login/totp
Set-Cookie: auth_tx=tx_73c1...; Secure; HttpOnly; SameSite=Lax; Max-Age=300
Cache-Control: no-store
```

This is not the final application session.

The temporary credential should have narrow authority:

```text
Allowed:
- submit TOTP for transaction tx_73c1

Not allowed:
- access reports
- change profile
- invoke normal APIs
```

This is an example of **least privilege** applied to authentication state.

---

# 4.10 Stage 8: TOTP submission

The browser requests the TOTP page and Alice enters a six-digit code generated by her authenticator.

Request:

```http
POST /login/totp HTTP/1.1
Host: finance.example.internal
Cookie: auth_tx=tx_73c1...
Content-Type: application/x-www-form-urlencoded

code=123456
```

The code above is only illustrative.

The server:

1. Resolves `auth_tx`.
2. Confirms the transaction exists.
3. Confirms it has not expired.
4. Confirms password verification was completed.
5. Loads Alice’s TOTP secret.
6. Computes valid codes for the accepted time window.
7. Compares the submitted code.
8. Checks whether the code was already used, if replay tracking is implemented.
9. Increments attempt counters.
10. Records the result.

TOTP is time-based, so server clock accuracy matters.

---

## Simplified TOTP mental model

```text
shared secret + current time window
→ HMAC computation
→ truncated numeric code
```

Both the authenticator and server know the same secret.

That means TOTP is not phishing-resistant. A phishing site can collect the current code and relay it quickly to the real service.

Passkeys and properly designed hardware-backed authentication can provide stronger phishing resistance.

---

# 4.11 Stage 9: Authentication succeeds

The application now considers the authentication policy satisfied.

Security context:

```json
{
  "subject_id": "u_1048",
  "authentication_methods": ["password", "totp"],
  "authentication_time": "2026-07-31T09:16:12Z",
  "authentication_strength": "mfa",
  "account_status": "active"
}
```

The application creates a session identifier using a cryptographically secure random-number generator.

Example conceptual value:

```text
s_E5f8m4...
```

The session identifier should be:

* Unpredictable
* Long enough to resist guessing
* Unique with overwhelming probability
* Free of embedded secrets that need client interpretation
* Stored and transmitted securely

---

# 4.12 Stage 10: Session storage

The server stores:

```text
session_id_hash: HASH(s_E5f8m4...)
subject_id: u_1048
created_at: 2026-07-31T09:16:12Z
last_seen_at: 2026-07-31T09:16:12Z
absolute_expiry: 2026-07-31T17:16:12Z
idle_expiry: 2026-07-31T09:46:12Z
auth_methods: [password, totp]
csrf_secret: ...
status: active
```

Some systems store a hash of the session identifier rather than the raw value. This can reduce the effect of certain session-store disclosures, though details matter.

The browser receives the raw session identifier:

```http
HTTP/1.1 302 Found
Location: /reports
Set-Cookie: session_id=s_E5f8m4...; Secure; HttpOnly; SameSite=Lax; Path=/
Set-Cookie: auth_tx=; Max-Age=0; Secure; HttpOnly; SameSite=Lax
Cache-Control: no-store
```

Cookie attributes:

| Attribute              | Purpose                                 |
| ---------------------- | --------------------------------------- |
| `Secure`               | Send cookie only over HTTPS             |
| `HttpOnly`             | Prevent normal JavaScript access        |
| `SameSite`             | Restrict some cross-site cookie sending |
| `Path=/`               | Defines URL path scope                  |
| `Max-Age` or `Expires` | Defines browser-side persistence        |

These attributes reduce risk but do not make cookie theft impossible.

---

## Data transformation so far

```text
Human knowledge:
  password
        ↓ verified

Shared MFA secret:
  used to verify TOTP
        ↓ authentication result

Authentication result:
  user u_1048 passed password + TOTP
        ↓ session creation

Random session identifier:
  s_E5f8m4...
        ↓ browser credential

Server-side session:
  maps s_E5f8m4... to user u_1048 and authentication context
```

The browser does not need to resend the password or TOTP on every request.

---

# 4.13 Stage 11: Browser requests the reports page

The browser follows the redirect:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
Cookie: session_id=s_E5f8m4...
```

The reverse proxy:

1. Accepts the TLS connection.
2. Parses the HTTP request.
3. Applies request-size and protocol rules.
4. Routes `/reports` to the application.
5. Adds infrastructure metadata as configured.

Possible internal forwarding:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
Cookie: session_id=s_E5f8m4...
X-Request-ID: req_8ae1...
X-Forwarded-Proto: https
```

Identity has not necessarily been validated by the proxy in this architecture. The application handles the session.

---

# 4.14 Stage 12: Session authentication

The application extracts:

```text
session_id = s_E5f8m4...
```

It queries the session store using the appropriate lookup representation.

Conceptually:

```text
HASH(s_E5f8m4...) → session record
```

It checks:

* Session exists
* Session status is active
* Absolute expiry has not passed
* Idle expiry has not passed
* Account is not disabled, depending on architecture
* Session has required authentication strength
* Session is not revoked
* Session metadata is internally consistent

If valid, the application creates a request-local security context.

Example:

```python
request.identity = {
    "subject_id": "u_1048",
    "session_id": "s_E5f8m4...",
    "auth_methods": ["password", "totp"]
}
```

This state should usually be scoped to the request.

It must not leak across requests handled by reused threads, workers, or asynchronous tasks.

---

# 4.15 Stage 13: Reports list authorization

The application now knows:

```text
Subject = u_1048
Action = LIST
Resource type = financial_report
```

It loads authorization information.

Possible data:

```text
User u_1048:
  department = finance
  roles = [financial_analyst]

Policy:
  financial_analyst may list reports in own department
```

Decision:

```text
ALLOW LIST financial_report
WHERE report.department = finance
```

A strong design does not merely authorize the route and then fetch every report.

Instead, authorization constraints should affect data retrieval.

Conceptual query:

```sql
SELECT report_id, title, reporting_period
FROM reports
WHERE department_id = $1
ORDER BY reporting_period DESC;
```

The department identifier must come from trusted server-side identity or policy data, not directly from a client-controlled field.

---

## Dangerous pattern

```sql
SELECT *
FROM reports
WHERE department_id = $client_supplied_department;
```

If Alice can change the request from:

```text
department=finance
```

to:

```text
department=executive
```

the application may expose another department’s data.

This is an authorization failure, not an authentication failure.

Alice is still correctly authenticated.

---

# 4.16 Stage 14: Alice selects a report

Alice chooses:

```text
Report ID: report-2026-Q2-finance
```

Browser request:

```http
GET /reports/report-2026-Q2-finance/download HTTP/1.1
Host: finance.example.internal
Cookie: session_id=s_E5f8m4...
```

The application must not assume that access to the reports page grants access to every report URL.

It performs authorization again.

Security tuple:

```text
Subject: u_1048
Action: DOWNLOAD
Resource: report-2026-Q2-finance
Context:
  department = finance
  role = financial_analyst
  report classification = confidential
  report department = finance
  authentication methods = password + totp
```

---

## Possible authorization policy

```text
ALLOW download
IF:
  subject.department == resource.department
  AND subject.role IN ["financial_analyst", "finance_manager"]
  AND authentication includes MFA
  AND account.status == active
```

Result:

```text
ALLOW
```

The policy engine may return additional obligations:

```text
ALLOW
OBLIGATIONS:
  record audit event
  apply confidential watermark
  disable shared-cache storage
```

---

# 4.17 Stage 15: Resource retrieval

After authorization succeeds, the application retrieves the resource.

Possible query:

```sql
SELECT
    storage_key,
    title,
    classification
FROM reports
WHERE report_id = $1
  AND department_id = $2;
```

Using the authorized department in the query provides defense in depth.

The query should return exactly one authorized resource.

If no row is returned, the application may answer:

```http
HTTP/1.1 404 Not Found
```

This can avoid revealing whether an inaccessible report exists.

---

## Downstream identity issue

Suppose the report file is stored at:

```text
/srv/reports/finance/2026-q2.pdf
```

The application process runs as Linux user:

```text
finance-app
```

When it calls:

```text
open("/srv/reports/finance/2026-q2.pdf", O_RDONLY)
```

the Linux kernel evaluates filesystem permissions for `finance-app`, not Alice.

Therefore:

```text
Application authorization:
  May Alice download this report?

Kernel authorization:
  May process UID finance-app open this file?
```

Both must succeed.

---

# 4.18 Stage 16: Response construction

The application returns the file:

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="finance-2026-q2.pdf"
Cache-Control: private, no-store
X-Content-Type-Options: nosniff
```

The exact headers depend on requirements.

Sensitive data may require:

* No shared caching
* Content-disposition controls
* Download auditing
* Watermarking
* Data-loss prevention inspection
* Response-size limits
* Range-request handling
* Malware scanning

Authentication alone does not determine these controls.

They depend on resource sensitivity and system policy.

---

# 4.19 Stage 17: Audit events

The system records relevant events.

Illustrative structured login event:

```json
{
  "timestamp": "2026-07-31T09:16:12Z",
  "event_type": "authentication_success",
  "subject_id": "u_1048",
  "methods": ["password", "totp"],
  "session_id_hash": "sha256:...",
  "source_ip": "192.0.2.51",
  "request_id": "req_71a2...",
  "result": "success"
}
```

Illustrative authorization event:

```json
{
  "timestamp": "2026-07-31T09:18:44Z",
  "event_type": "authorization_decision",
  "subject_id": "u_1048",
  "action": "download",
  "resource_id": "report-2026-Q2-finance",
  "policy": "financial-report-download-v3",
  "decision": "allow",
  "request_id": "req_8ae1..."
}
```

Security logs should generally avoid including:

* Passwords
* TOTP secrets
* Session identifiers in reusable form
* Full access tokens
* Private keys
* Sensitive report contents
* Unnecessary personal data

Logging full credentials can turn the logging system into a credential database.

---

# 4.20 Complete sequence diagram

```mermaid
sequenceDiagram
    actor Alice
    participant Browser
    participant Proxy as Reverse Proxy
    participant App as Application
    participant Users as User Database
    participant Sessions as Session Store
    participant Reports as Report Database
    participant Audit as Audit System

    Alice->>Browser: Open /reports
    Browser->>Proxy: GET /reports
    Proxy->>App: Forward request
    App->>App: No session found
    App-->>Browser: 302 /login

    Browser->>Proxy: GET /login
    Proxy->>App: Forward request
    App-->>Browser: Login form + CSRF state

    Alice->>Browser: Enter username and password
    Browser->>Proxy: POST /login
    Proxy->>App: Forward encrypted request after TLS termination
    App->>App: Validate request and rate limits
    App->>Users: Lookup normalized username
    Users-->>App: Account + password hash
    App->>App: Verify password hash
    App->>Audit: Record password-stage result
    App-->>Browser: Temporary auth transaction

    Alice->>Browser: Enter TOTP
    Browser->>Proxy: POST /login/totp
    Proxy->>App: Forward request
    App->>Users: Load MFA configuration
    Users-->>App: TOTP verification data
    App->>App: Verify TOTP
    App->>Sessions: Create authenticated session
    App->>Audit: Record authentication success
    App-->>Browser: Set session cookie; redirect /reports

    Browser->>Proxy: GET /reports + session cookie
    Proxy->>App: Forward request
    App->>Sessions: Resolve and validate session
    Sessions-->>App: Subject and authentication context
    App->>App: Authorize report listing
    App->>Reports: Query authorized reports
    Reports-->>App: Finance report metadata
    App-->>Browser: Reports page

    Alice->>Browser: Select report
    Browser->>Proxy: GET /reports/{id}/download + cookie
    Proxy->>App: Forward request
    App->>Sessions: Validate session
    Sessions-->>App: Subject u_1048
    App->>Reports: Load resource attributes
    Reports-->>App: Department and classification
    App->>App: Authorize subject/action/resource
    App->>Audit: Record authorization decision
    App->>Reports: Retrieve authorized report
    Reports-->>App: Report data
    App-->>Browser: 200 PDF response
```

---

# 4.21 What data changes at each stage?

| Stage                 | Input                                 | Output                                  |
| --------------------- | ------------------------------------- | --------------------------------------- |
| Initial request       | No credential                         | Login redirect                          |
| Password submission   | Username + password                   | Password verification result            |
| MFA transaction       | Password-verified partial state       | Temporary transaction                   |
| TOTP submission       | Temporary transaction + code          | Full authentication result              |
| Session creation      | Authentication result                 | Random session ID + server-side session |
| Authenticated request | Session cookie                        | Request-local subject context           |
| Authorization         | Subject + action + resource + context | Allow or deny                           |
| Resource retrieval    | Authorized resource identifier        | Protected data                          |
| Audit                 | Security event fields                 | Persistent event record                 |

A major mental-model distinction is:

```text
Credentials are not permissions.
Identity is not a decision.
A valid session is not permanent authorization.
A route match is not object-level authorization.
```

---

# 4.22 Failure walkthroughs

## Failure A: Wrong password

Flow:

```text
Account lookup succeeds
→ password verification fails
→ no partial authentication transaction
→ no session created
→ generic login failure returned
→ failure event recorded
```

Possible response:

```http
HTTP/1.1 401 Unauthorized
Cache-Control: no-store
```

Browser-based applications may instead return `200` with the login form and an error message.

---

## Failure B: Correct password, wrong TOTP

Flow:

```text
Password verification succeeds
→ temporary transaction exists
→ TOTP verification fails
→ attempt count increments
→ final session is not created
```

The application must not accidentally treat the password stage as a complete login.

---

## Failure C: Stolen session cookie

If an attacker obtains the active cookie:

```text
session_id=s_E5f8m4...
```

the attacker may be able to act as Alice without knowing:

* Alice’s password
* Alice’s TOTP secret
* Alice’s current TOTP code

The session is now the active credential.

This is why session protection is critical.

---

## Failure D: Session exists but has expired

Flow:

```text
Cookie is present
→ session lookup succeeds
→ expiry check fails
→ session rejected
→ user must authenticate again
```

The mere presence of a cookie is not proof of a valid session.

---

## Failure E: Alice requests another department’s report

Flow:

```text
Session valid
→ Alice authenticated
→ report exists
→ department policy does not match
→ authorization denied
```

Possible external responses:

```http
HTTP/1.1 403 Forbidden
```

or:

```http
HTTP/1.1 404 Not Found
```

depending on disclosure policy.

---

## Failure F: Application allows access, kernel denies file open

Flow:

```text
Application policy = ALLOW
→ process calls open()
→ kernel checks filesystem permissions
→ kernel returns EACCES
→ application returns server error or controlled denial
```

This is not necessarily an application authorization failure.

It may be:

* Incorrect file ownership
* Incorrect Unix mode bits
* Missing group membership
* SELinux denial
* AppArmor denial
* Filesystem mount restriction
* Container volume permission issue

---

## Failure G: Account disabled after session creation

Behavior depends on architecture.

### Design 1: Check account state on every request

```text
Disable account
→ next request queries current account state
→ session rejected quickly
```

Cost:

* Additional database or directory dependency.

### Design 2: Trust cached session state

```text
Disable account
→ active session remains usable until expiry or revocation
```

Benefit:

* Faster requests and fewer dependencies.

Risk:

* Delayed revocation.

### Design 3: Revocation event

```text
Disable account
→ publish revocation event
→ session store marks sessions inactive
→ subsequent requests fail
```

This is more complex but can combine good performance with faster revocation.

---

# 4.23 Practical observation with `curl`

## Inspect response headers

```bash
curl -i https://finance.example.internal/reports
```

### What it does

Requests the resource and prints response headers together with the body.

### Why it is useful

It can reveal:

* Redirects
* Status codes
* `Set-Cookie`
* Cache-control headers
* Authentication challenges

### Important flag

* `-i`: include response headers in output

### Output interpretation

Illustrative output:

```http
HTTP/1.1 302 Found
Location: /login
Set-Cookie: login_csrf=...; Secure; HttpOnly; SameSite=Lax
Cache-Control: no-store
```

This indicates that the request was not associated with an accepted authenticated session.

### Safety

Do not paste session cookies or tokens into shared terminals, tickets, or chat systems.

---

## Follow redirects

```bash
curl -i -L https://finance.example.internal/reports
```

### What it does

Follows HTTP redirects.

### Important flag

* `-L`: follow the URL in each `Location` header

### Why it is useful

It shows the eventual login page instead of stopping at the initial `302`.

### Limitation

It does not automatically reproduce browser JavaScript behavior or complex interactive login flows.

---

## Store and reuse cookies

```bash
curl -i \
  --cookie-jar cookies.txt \
  --cookie cookies.txt \
  https://finance.example.internal/login
```

### What it does

* Writes received cookies to `cookies.txt`
* Sends matching cookies from `cookies.txt`

### Important flags

* `--cookie-jar FILE`: save response cookies
* `--cookie FILE`: read and send cookies

### Why it is useful

It helps inspect multi-request session flows.

### Safety

The cookie file may contain live credentials.

Protect it:

```bash
chmod 600 cookies.txt
```

Delete it when finished:

```bash
rm -f cookies.txt
```

On highly sensitive systems, secure deletion is filesystem- and storage-dependent; ordinary `rm` only removes the directory entry.

---

## Show verbose connection details

```bash
curl -v https://finance.example.internal/reports
```

### What it does

Shows connection, TLS, request-header, and response-header details.

### Important flag

* `-v`: verbose mode

### Why it is useful

It helps diagnose:

* TLS certificate problems
* Hostname mismatch
* Redirect behavior
* Proxy behavior
* Which headers were sent

### Safety

Verbose output may expose:

* Cookies
* Authorization headers
* Internal hostnames
* Tokens

Redact before sharing.

---

## Send a bearer token

```bash
curl -i \
  -H 'Authorization: Bearer REPLACE_WITH_TOKEN' \
  https://api.example.internal/reports
```

### What it does

Adds an HTTP `Authorization` header.

### Why it is useful

It tests token-protected APIs without a browser.

### Important flag

* `-H`: add a request header

### Safety

Putting secrets directly on a command line may expose them through:

* Shell history
* Process inspection
* Terminal recording
* CI logs

A temporary environment variable can reduce accidental duplication, although it is not a complete secret-management solution:

```bash
read -r -s ACCESS_TOKEN
curl -i \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  https://api.example.internal/reports
unset ACCESS_TOKEN
```

`read -s` suppresses terminal echo in common shells such as Bash.

---

# 4.24 Practical log correlation

A request ID allows events from several components to be connected.

Example:

```text
Browser request
  request_id=req_8ae1

Reverse proxy log
  request_id=req_8ae1
  status=200
  upstream=finance-app

Application log
  request_id=req_8ae1
  subject=u_1048
  action=download
  resource=report-2026-Q2-finance

Authorization log
  request_id=req_8ae1
  decision=allow
  policy=financial-report-download-v3

Database log
  application_name=finance-app
  request_id=req_8ae1
  query_duration_ms=12
```

Without correlation identifiers, debugging becomes guesswork across distributed logs.

Avoid trusting a client-provided request ID without normalization or validation. A proxy may replace it or generate a new internal trace identifier.

---

# 4.25 Troubleshooting decision tree

```mermaid
flowchart TD
    A[Protected request failed]
    B{Was authentication material sent?}
    C[Inspect browser/curl cookie or Authorization header]
    D{Was credential accepted?}
    E[Check expiry, signature, session lookup, issuer, audience]
    F{Was subject resolved?}
    G[Check identity mapping and account status]
    H{Was authorization denied?}
    I[Inspect action, resource, policy, roles, attributes]
    J{Did downstream access fail?}
    K[Check database, filesystem, network, service identity]
    L[Inspect application error and response construction]

    A --> B
    B -->|No| C
    B -->|Yes| D
    D -->|No| E
    D -->|Yes| F
    F -->|No| G
    F -->|Yes| H
    H -->|Yes| I
    H -->|No| J
    J -->|Yes| K
    J -->|No| L
```

The key debugging rule is:

> Identify the exact stage where the request stopped progressing.

Do not treat every `403`, redirect, or login prompt as the same problem.

---

## Questions: End-to-End Flow

1. At which exact stage does Alice’s browser first possess a reusable authenticated credential?
2. Why should the temporary MFA transaction have less authority than the final session?
3. What could happen if the application creates the final session immediately after password verification but merely records MFA as “pending”?
4. Why must the application authorize the individual report instead of only protecting the `/reports` page?
5. How can adding an authorized department predicate to the SQL query provide defense in depth?
6. Which data should be stable within a session, and which data should be reloaded on every authorization decision?
7. Why does `HttpOnly` reduce some cookie-theft risks without preventing all session hijacking?
8. How would you distinguish a session-store outage from an expired session using logs and metrics?
9. What is the difference between the browser-to-proxy identity context and the application-process identity seen by the kernel?
10. Why could account disabling fail to terminate existing sessions immediately?
11. What information should be included in an authorization audit event to make a future investigation possible?
12. A user receives `404` for a report they believe exists. Which authentication, authorization, and data-layer conditions could produce that result?
13. What new security issue appears if the reverse proxy logs the complete `Cookie` header?
14. How could a valid session be accepted by one application instance but rejected by another?
15. Why should the server generate a new session identifier after successful authentication rather than continue using a pre-login identifier?

# 5. Credentials and Authentication Factors

A credential is evidence used to support an identity claim.

The important security question is not merely:

> “Does the user know a secret?”

It is:

> “What evidence was presented, how was it verified, and what attacks can defeat that evidence?”

---

## 5.1 Credential lifecycle

Credentials have a lifecycle:

```mermaid
flowchart LR
    E[Enrollment]
    I[Issuance]
    S[Storage]
    P[Presentation]
    V[Verification]
    R[Rotation or recovery]
    X[Revocation]
    D[Deletion]

    E --> I --> S --> P --> V
    V --> R
    R --> S
    V --> X --> D
```

### Enrollment

The system connects a credential to an identity.

Examples:

* A user creates a password.
* An administrator installs an SSH public key.
* A user registers a passkey.
* A certificate authority issues a certificate.
* A workload receives a cloud identity.
* An authenticator app scans a TOTP secret.

Enrollment is often more security-sensitive than everyday verification.

An attacker who can register their own credential under Alice’s account may not need to steal Alice’s existing credential.

---

### Issuance

A system creates or distributes credential material.

Examples:

* Server creates an API key.
* Certificate authority signs a certificate.
* Identity provider issues an access token.
* Application creates a session cookie.
* Kerberos KDC issues a ticket.

The issuer must decide:

* Which subject receives the credential?
* What permissions or claims are attached?
* How long is it valid?
* Which audience may accept it?
* Can it be delegated?
* How can it be revoked?

---

### Storage

Credential storage occurs on both client and server sides.

Examples:

| Credential  | Client-side storage               | Server-side storage               |
| ----------- | --------------------------------- | --------------------------------- |
| Password    | Human memory or password manager  | Password hash                     |
| Session ID  | Browser cookie store              | Session record                    |
| SSH key     | Private-key file or agent         | Public key                        |
| TOTP        | Authenticator secret              | Same shared secret                |
| Passkey     | Authenticator or password manager | Public key                        |
| API key     | Secret manager or configuration   | Hash or lookup record             |
| Certificate | Certificate plus private key      | Trusted CA or certificate mapping |

A recurring rule:

> Store only what the verifier actually needs.

For public-key authentication, the verifier generally needs the public key, not the private key.

---

### Presentation

The subject presents proof through a protocol.

Examples:

* Password sent over TLS
* Signature over an SSH handshake
* WebAuthn assertion
* Bearer token in an HTTP header
* Client certificate during TLS
* Kerberos ticket in a protocol exchange

Presentation may expose reusable credential material.

For example:

* A password is reusable.
* A bearer token is reusable until expiry or revocation.
* A digital signature over a challenge is normally not reusable for another challenge.

---

### Verification

The verifier checks whether the evidence satisfies its policy.

Verification may include:

* Password hash comparison
* Signature verification
* Certificate-chain validation
* Token signature validation
* Issuer and audience validation
* Expiry checks
* Nonce or challenge checks
* Device attestation
* Session lookup
* Revocation status

Successful cryptographic verification is only part of the decision.

A certificate signature may be valid while the certificate is:

* Expired
* Revoked
* Issued for another identity
* Intended for another usage
* Chained to an untrusted authority

---

### Rotation

Rotation replaces credential material.

Reasons include:

* Scheduled key rotation
* Suspected compromise
* Personnel changes
* Cryptographic-policy changes
* Certificate expiry
* Secret leakage
* Algorithm migration

Rotation can be difficult in distributed systems because old and new credentials may need to coexist temporarily.

---

### Revocation

Revocation makes a previously valid credential unacceptable.

Examples:

* Delete an active session.
* Disable an API key.
* Remove an SSH public key.
* Add a certificate to a revocation system.
* Mark a refresh token revoked.
* Disable the underlying account.

Revocation latency depends on:

* Caches
* Token lifetime
* Offline verification
* Replication delay
* Component availability
* Whether each verifier checks revocation state

---

## 5.2 Authentication factors

Authentication factors are categories of evidence.

The common categories are:

| Factor     | Meaning                     | Examples                              |
| ---------- | --------------------------- | ------------------------------------- |
| Knowledge  | Something the subject knows | Password, PIN                         |
| Possession | Something the subject has   | Security key, phone, smart card       |
| Inherence  | Something the subject is    | Fingerprint, face, iris               |
| Location   | Somewhere the subject is    | Approved network or physical location |
| Behavior   | Something the subject does  | Typing or interaction patterns        |

The first three are the traditional factor categories.

Location and behavior are often used as contextual or risk signals rather than primary factors.

---

## 5.3 Multifactor authentication

**Multifactor authentication**, or MFA, requires evidence from more than one factor category.

Examples:

```text
Password + TOTP device
Knowledge + Possession
```

```text
PIN + smart card
Knowledge + Possession
```

```text
Passkey unlocked with fingerprint
Possession + local user verification
```

Two passwords are not two factors.

Two knowledge-based questions are not two factors.

```text
Password + security question
= two pieces of knowledge
≠ strong multifactor authentication
```

---

## 5.4 Two-step verification versus two-factor authentication

These terms are often used loosely.

### Two-step verification

Two separate verification stages occur.

Example:

```text
Password
then
another password
```

This is two-step verification but not necessarily two-factor authentication.

### Two-factor authentication

The evidence comes from two distinct factor categories.

Example:

```text
Password
then
hardware security key
```

In product documentation, “2FA” is frequently used for both concepts. Architecture and security reviews should identify the actual mechanisms rather than rely only on labels.

---

## 5.5 Authentication strength

Not all credentials within the same factor category have equal strength.

For example, possession factors include:

* SMS code
* TOTP authenticator
* Push approval
* Hardware security key
* Smart card
* Device-bound private key
* Passkey synchronized through a credential provider

These differ in resistance to:

* Phishing
* Malware
* SIM swapping
* Replay
* Credential export
* Social engineering
* Device theft
* Server compromise

A useful model is:

```text
Authentication strength
=
factor diversity
+ protocol properties
+ credential protection
+ enrollment security
+ recovery security
+ verifier implementation
```

---

## 5.6 Phishing resistance

A phishing-resistant authenticator prevents or strongly limits authentication to an attacker-controlled site.

### Password flow

```text
Alice enters password at fake.example
→ attacker learns password
→ attacker sends password to real application
```

### TOTP relay

```text
Alice enters password and TOTP at fake.example
→ attacker immediately forwards both
→ real application accepts them
```

TOTP reduces some account-takeover risks, but a real-time phishing proxy can relay it.

### Origin-bound public-key flow

A WebAuthn or passkey credential is associated with a relying-party identifier.

The authenticator signs a challenge for the expected site identity.

A phishing site on another domain cannot normally request an assertion valid for the legitimate site.

```mermaid
sequenceDiagram
    actor Alice
    participant Fake as fake.example
    participant Auth as Authenticator
    participant Real as bank.example

    Alice->>Fake: Open phishing page
    Fake->>Auth: Request credential for fake.example
    Auth-->>Fake: No bank.example credential available
    Fake->>Real: Cannot produce valid bank.example assertion
```

This is a protocol property, not merely a user-interface warning.

---

## 5.7 Knowledge factors

### What they are

Secrets the subject can reproduce.

Examples:

* Password
* PIN
* Recovery phrase
* Answers to security questions

### Why they exist

They are easy to deploy because they do not require specialized hardware or certificate infrastructure.

### How they work

The verifier stores a representation that allows checking whether the submitted value matches.

For passwords, this should be a password hash rather than plaintext or reversible encryption.

### Common failures

* Weak passwords
* Password reuse
* Credential stuffing
* Phishing
* Keylogging
* Password database theft
* Logging submitted passwords
* Predictable recovery questions
* Insecure reset process

### Where they fit

Knowledge factors are frequently used for:

* Initial account login
* Device unlock
* Recovery
* Step-up authentication
* Local key decryption

They are increasingly paired with stronger mechanisms.

---

## 5.8 Possession factors

### What they are

Proof that the subject controls a device or cryptographic object.

Examples:

* Security key
* Smart card
* Mobile authenticator
* Device private key
* SIM-associated phone number
* Hardware token

### How possession is proven

Possession may be demonstrated by:

* Producing a one-time code
* Approving a push request
* Signing a challenge
* Completing a client-certificate handshake
* Decrypting a value
* Responding through a registered channel

### Important distinction

Receiving an SMS does not prove possession of a specific physical phone.

It more closely proves current control over delivery to a phone number.

That control may be transferred through:

* SIM swapping
* Carrier-account compromise
* Number reassignment
* Call forwarding
* Mobile malware

---

## 5.9 Inherence factors

Biometric characteristics are commonly used to unlock a device or private key.

Examples:

* Fingerprint
* Face recognition
* Iris recognition
* Voice characteristics

### Important mental model

In many modern systems, the server does not receive the raw biometric.

Instead:

```text
Biometric match on local device
→ device unlocks cryptographic credential
→ credential signs server challenge
→ server verifies signature
```

The server authenticates the cryptographic response.

The biometric is part of local user verification.

---

### Biometric limitations

Biometrics are not ordinary secrets.

A fingerprint:

* Cannot easily be changed after compromise
* May be left on physical objects
* Is probabilistically matched
* May produce false accepts or false rejects
* May have accessibility limitations
* May be subject to coercion concerns

Therefore, biometric systems usually require fallback methods such as a PIN.

The fallback method becomes part of the overall security boundary.

---

## 5.10 One-time passwords

A one-time password is intended to be accepted once or for a limited period.

Common categories:

* HOTP: counter-based
* TOTP: time-based
* Server-generated email code
* Server-generated SMS code

---

## 5.11 HOTP mental model

HOTP uses:

```text
Shared secret + counter
→ HMAC
→ truncation
→ numeric code
```

Both sides must maintain compatible counter state.

A simplified flow:

```mermaid
sequenceDiagram
    participant Token
    participant User
    participant Server

    Token->>Token: Compute code from secret and counter
    Token-->>User: Display code
    User->>Server: Submit code
    Server->>Server: Compute expected code
    Server->>Server: Advance accepted counter
```

### Failure mode: counter drift

If the token advances its counter but the server does not observe those codes, their counters diverge.

The server may search a limited future window to resynchronize.

A large search window increases the number of acceptable codes and can weaken guessing resistance.

---

## 5.12 TOTP mental model

TOTP derives a moving counter from time.

Conceptually:

```text
time_step = floor(current_unix_time / interval)
code = HOTP(secret, time_step)
```

A common interval is 30 seconds, though implementations and policies vary.

The verifier may accept:

```text
previous interval
current interval
next interval
```

to tolerate clock skew and network delay.

This expands the set of valid codes.

---

### TOTP data flow

```mermaid
flowchart LR
    S[Shared secret]
    C[Current time step]
    H[HMAC]
    T[Dynamic truncation]
    N[Numeric code]

    S --> H
    C --> H
    H --> T --> N
```

### Security properties

TOTP provides:

* A short-lived code
* Protection against reuse of a captured password alone
* Offline generation without network connectivity

TOTP does not inherently provide:

* Phishing resistance
* Server authentication
* Strong replay prevention within the accepted window
* Proof of a particular device
* Protection if the shared secret is stolen

---

## 5.13 Push authentication

A login request triggers a notification on a registered device.

The user approves or rejects it.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant Server
    participant Push as Push Service
    participant Phone

    Browser->>Server: Password accepted
    Server->>Push: Send approval request
    Push->>Phone: Display notification
    User->>Phone: Approve
    Phone->>Server: Signed or authenticated response
    Server-->>Browser: Authentication complete
```

### Common failure: push fatigue

An attacker repeatedly attempts login, causing many approval prompts.

The user may approve one to stop the interruptions.

Defenses include:

* Number matching
* Displaying origin and location
* Rate limiting
* User-initiated authentication flows
* Phishing-resistant authenticators
* Clear rejection and reporting controls

---

## 5.14 Challenge-response authentication

A challenge-response protocol proves possession of a secret or private key without sending that credential directly.

### Public-key example

1. Server generates unpredictable challenge.
2. Client signs challenge using private key.
3. Server verifies signature using public key.
4. Server confirms the challenge belongs to the current transaction.
5. Server rejects reuse.

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: Begin authentication
    Server-->>Client: Random challenge
    Client->>Client: Sign challenge with private key
    Client->>Server: Signature + credential identifier
    Server->>Server: Verify signature with public key
    Server-->>Client: Authentication result
```

The private key does not cross the network.

---

### Why the challenge must be unpredictable

Suppose the server always asks the client to sign:

```text
LOGIN
```

An attacker could capture one valid signature and replay it later.

A random challenge binds the response to a fresh authentication transaction.

The challenge should also be:

* Associated with the intended user or transaction
* Short-lived
* Single-use
* Generated using a secure random source

---

## 5.15 Replay attacks

A replay attack reuses a previously valid authentication message.

Example:

```text
Attacker records:
Authorization: Bearer abc123

Attacker later sends:
Authorization: Bearer abc123
```

If the bearer token remains valid, the replay succeeds.

Defenses depend on the protocol:

| Defense                              | Mechanism                                     |
| ------------------------------------ | --------------------------------------------- |
| Nonce                                | Require a fresh random value                  |
| Timestamp                            | Reject old messages                           |
| Sequence number                      | Reject repeated or out-of-order values        |
| Challenge-response                   | Sign a fresh server challenge                 |
| Short expiry                         | Reduce replay window                          |
| Channel binding                      | Bind proof to a specific connection           |
| Token binding or proof-of-possession | Require a key in addition to token possession |
| Replay cache                         | Remember previously used values               |

Bearer credentials are intentionally simple, but their simplicity creates replay risk.

---

## 5.16 Bearer versus proof-of-possession credentials

### Bearer credential

```text
Possession of token string
→ sufficient to use token
```

Examples:

* Many session cookies
* Many OAuth access tokens
* API keys

### Proof-of-possession credential

The client must prove control of a cryptographic key associated with the credential.

```text
Token + valid signature from associated private key
→ credential accepted
```

This can reduce the usefulness of a stolen token.

However, it introduces complexity:

* Key generation
* Key storage
* Request signing
* Replay prevention
* Clock handling
* Canonicalization of signed data
* Key rotation

---

## 5.17 API keys

An API key is usually a high-entropy secret identifying an application, client, or integration.

Example:

```http
Authorization: ApiKey ak_live_7f8...
```

### What it is

A machine-oriented credential, often represented as a random string.

### Why it exists

It provides a simple way to authenticate automated clients.

### How it works

1. Server issues a random key.
2. Client stores the key.
3. Client sends it with requests.
4. Server resolves or hashes the key.
5. Server checks status, scope, and policy.

### Where it fits

API keys are common for:

* Developer APIs
* CI/CD integrations
* Monitoring agents
* Webhooks
* Service integrations
* Low-complexity internal systems

### Common misconceptions

An API key is not automatically:

* A user identity
* Short-lived
* Least privilege
* Protected from replay
* Suitable for browser code
* Safe to embed in mobile applications
* Safe merely because it appears in a custom header

---

## 5.18 Structured API-key design

A key may be divided into:

```text
public identifier.secret material
```

Example conceptual format:

```text
ak_prod_93ad7f.K7Lr9...secret...
```

The public prefix can help identify the database record without storing the full secret.

Server-side record:

```text
key_id = ak_prod_93ad7f
secret_hash = HASH(K7Lr9...secret...)
owner = inventory-importer
scopes = [inventory:read, inventory:write]
status = active
created_at = ...
expires_at = ...
```

Verification:

1. Parse key identifier.
2. Load key record.
3. Hash submitted secret.
4. Compare with stored hash.
5. Check status and expiry.
6. Check scope.
7. Record use.

This resembles password verification but uses a high-entropy machine-generated secret.

Because strong API keys have high entropy, a fast cryptographic hash may be suitable for lookup or storage in some designs. Human passwords require deliberately expensive password hashing because their entropy is much lower.

---

## 5.19 Credential scope

A credential should be limited along several dimensions.

| Scope dimension | Example                            |
| --------------- | ---------------------------------- |
| Resource        | Only storage bucket `reports-prod` |
| Action          | Read but not delete                |
| Environment     | Development only                   |
| Network         | Requests from approved subnet      |
| Audience        | Only `billing-api`                 |
| Time            | Valid for 15 minutes               |
| Tenant          | Organization `tenant-42`           |
| Delegation      | Cannot create further credentials  |

A credential with excessive scope increases the effect of compromise.

---

## 5.20 Long-lived versus short-lived credentials

### Long-lived credentials

Examples:

* Static API key
* SSH private key without expiry
* Password
* Permanent cloud access key

Advantages:

* Operational simplicity
* No frequent renewal dependency

Risks:

* Larger compromise window
* Harder inventory
* Forgotten credentials
* Difficult revocation
* Secret accumulation

### Short-lived credentials

Examples:

* Fifteen-minute access token
* Temporary cloud role credential
* Short-lived certificate
* Kerberos service ticket

Advantages:

* Reduced exposure window
* Natural expiry
* Better support for dynamic identity

Costs:

* Issuance dependency
* Renewal logic
* Clock sensitivity
* More protocol complexity
* Failure when renewal systems are unavailable

---

## 5.21 Credential bootstrap problem

A system that issues credentials must authenticate the requester somehow.

Example:

```text
Workload needs a short-lived certificate.
How does certificate issuer know which workload it is?
```

Possible bootstrap identities:

* Cloud instance identity
* Kubernetes service-account token
* TPM key
* Preinstalled certificate
* Node identity
* Hardware serial and enrollment secret
* Human administrator approval

This creates a chain:

```text
Bootstrap identity
→ credential issuer
→ short-lived credential
→ target service
```

The strength of the final credential cannot fully compensate for an insecure bootstrap process.

---

## 5.22 Recovery is an authentication mechanism

Account recovery is often treated as a secondary feature.

Security-wise, recovery is an alternative authentication path.

Example:

```text
Strong passkey login
but
password reset through weak email account
```

An attacker will target the weakest accepted path.

Recovery mechanisms include:

* Email reset link
* SMS reset code
* Backup code
* Recovery key
* Help-desk identity verification
* Trusted contact
* Existing authenticated device
* Administrator reset

Questions to ask:

* Who may initiate recovery?
* What evidence is required?
* How long are recovery links valid?
* Are links single-use?
* Does recovery revoke existing sessions?
* Is the user notified?
* Can support personnel bypass normal controls?
* Are recovery actions audited?

---

## 5.23 Device trust is not user authentication

A system may know:

```text
Device certificate belongs to laptop-173
```

That does not necessarily prove:

```text
Alice is currently using laptop-173
```

Similarly:

```text
Alice authenticated successfully
```

does not prove:

```text
Alice is using a managed or uncompromised device
```

A policy may combine both:

```text
ALLOW sensitive download
IF:
  user authenticated with MFA
  AND device certificate is valid
  AND device compliance state is healthy
```

These are separate trust signals.

---

## 5.24 Credential comparison table

| Credential                    | Secret crosses network? |                      Replay risk |              Phishing resistance | Server storage                |
| ----------------------------- | ----------------------: | -------------------------------: | -------------------------------: | ----------------------------- |
| Password over TLS             |         Yes, inside TLS |                              Yes |                              Low | Password hash                 |
| TOTP                          |    Code crosses network |           Within validity window |                              Low | Shared secret                 |
| Session cookie                |   Token crosses network |                      Usually yes |            Depends on login flow | Session record or signing key |
| API key                       |                     Yes |                              Yes |                Not user-oriented | Hash or secret record         |
| SSH public-key authentication |         Private key: no | Challenge prevents simple replay | Strong against password phishing | Public key                    |
| Client certificate            |         Private key: no |    Protocol-dependent protection |  Strong when correctly validated | Trusted CA or mapping         |
| Passkey/WebAuthn              |         Private key: no |                  Fresh challenge |                             High | Public key                    |
| SMS code                      |    Code crosses network |                   Limited window |                              Low | Delivery-channel mapping      |

---

## Questions: Credentials and Factors

1. Why is credential enrollment often as sensitive as credential verification?
2. Why does a password plus a security question not provide true multifactor authentication?
3. How can TOTP be successfully relayed by a phishing proxy even though each code expires quickly?
4. Why should a server challenge be both unpredictable and single-use?
5. What changes in the threat model when a bearer token becomes proof-of-possession-bound?
6. Why may a fast hash be acceptable for a high-entropy API key while being unsuitable for a human password?
7. How can a secure login mechanism be undermined by a weak account-recovery process?
8. Why does possession of a valid device certificate not prove which human is operating the device?
9. What operational dependency is introduced when permanent API keys are replaced with 15-minute credentials?
10. Why can a wide TOTP acceptance window increase guessing and replay risk?
11. Which component should detect reuse of a one-time challenge in a horizontally scaled authentication service?
12. How would you rotate an API key without causing downtime for a client that cannot update instantly?
13. Why should a temporary authentication transaction not be accepted as a normal application session?
14. What information must a service validate besides the cryptographic correctness of a client certificate?
15. How could a compromised credential issuer affect every downstream service even when those services validate signatures correctly?

---

# 6. Password Authentication Internals

Passwords remain widely used because they are simple to understand and deploy.

Their simplicity for users hides significant server-side complexity.

A secure password system must address:

* Password capture
* Transport protection
* Storage
* Offline cracking
* Online guessing
* Credential stuffing
* Account enumeration
* Reset and recovery
* Hash migration
* Logging and observability
* Session creation
* Compromise response

---

## 6.1 Password authentication data flow

```mermaid
sequenceDiagram
    actor User
    participant Client
    participant TLS as TLS Channel
    participant App as Authentication Service
    participant DB as User Database
    participant Hash as Password Hasher
    participant Session as Session Store

    User->>Client: Enter username and password
    Client->>TLS: Submit login request
    TLS->>App: Deliver decrypted request
    App->>DB: Lookup account
    DB-->>App: Stored password hash and account state
    App->>Hash: Verify submitted password
    Hash-->>App: Match or mismatch
    App->>App: Apply MFA and policy
    App->>Session: Create session if successful
    App-->>Client: Return session credential
```

The password is normally present in plaintext at several points:

* User input field
* Browser or client memory
* Decrypted request in application memory
* Password-verification function input

It should not be:

* Written to logs
* Stored in the database
* Included in metrics
* Added to tracing attributes
* Sent to unrelated services
* Retained longer than needed

TLS protects transport, not application memory.

---

## 6.2 Why plaintext storage fails

Suppose a database contains:

```text
alice@example.com | Summer2026!
bob@example.com   | correct-horse-...
```

Any database read exposes every password immediately.

Consequences extend beyond the current application because users may reuse passwords elsewhere.

A database administrator, SQL injection, backup leak, or accidental export could expose all credentials.

Plaintext storage provides no meaningful containment after database compromise.

---

## 6.3 Why reversible encryption is usually wrong

A system might encrypt passwords:

```text
ciphertext = Encrypt(database_key, password)
```

To verify login, it decrypts the stored value.

This requires the application to possess a decryption key.

If an attacker compromises both:

* The password database
* The application configuration or key-access path

the attacker can recover every password directly.

Password verification does not require recovering the original password.

Therefore, one-way password hashing is the correct model.

> Password encryption may be necessary only in unusual systems that must authenticate onward to a legacy service requiring the original password. Such designs introduce substantial risk.

---

## 6.4 Ordinary hash functions are insufficient

A naïve design:

```text
stored_hash = SHA256(password)
```

At login:

```text
SHA256(submitted_password) == stored_hash
```

This is one-way, but still insecure for password storage.

Why?

General-purpose hash functions are intentionally fast.

An attacker with the database can test large numbers of candidate passwords quickly.

```text
candidate password
→ fast hash
→ compare with stolen hash
```

Human-chosen passwords often have limited effective entropy because they follow predictable patterns.

Examples:

* Common words
* Names and dates
* Keyboard patterns
* Reused passwords
* Appended digits
* Known leaked passwords

The attacker does not reverse the hash mathematically.

The attacker guesses candidates and checks them.

---

## 6.5 Online versus offline guessing

### Online guessing

The attacker sends login attempts to the real server.

```mermaid
flowchart LR
    A[Attacker]
    S[Login server]
    DB[(User database)]

    A -->|Guess 1| S
    A -->|Guess 2| S
    A -->|Guess 3| S
    S --> DB
```

The defender can observe and limit attempts using:

* Rate limits
* Delays
* MFA
* Account lockout
* IP reputation
* Risk detection
* CAPTCHA or proof-of-work in limited contexts
* User notifications

---

### Offline guessing

The attacker steals password hashes and tests guesses on their own hardware.

```mermaid
flowchart LR
    H[(Stolen hashes)]
    G[Candidate generator]
    C[Attacker CPUs or GPUs]
    M[Matches]

    G --> C
    H --> C
    C --> M
```

The real service does not receive these attempts.

Therefore, online rate limiting cannot stop offline cracking.

The defense is to make each guess expensive through a password hashing or key-derivation function.

---

## 6.6 Password hashing functions

A password hashing function is deliberately expensive.

Modern choices commonly include:

* Argon2id
* scrypt
* bcrypt
* PBKDF2

The correct selection depends on:

* Platform support
* Security policy
* Compliance requirements
* Available libraries
* Memory limits
* Performance targets
* Hardware environment

Do not implement these algorithms manually.

Use maintained, well-reviewed libraries.

---

## 6.7 Password hash components

A stored password hash often encodes:

* Algorithm
* Version
* Cost parameters
* Salt
* Derived output

Illustrative Argon2id format:

```text
$argon2id$v=19$m=65536,t=3,p=1$BASE64_SALT$BASE64_HASH
```

Conceptual interpretation:

```text
algorithm = argon2id
version = 19
memory cost = 65536 KiB
time cost = 3
parallelism = 1
salt = unique random value
derived hash = verification output
```

Exact encoding and supported parameters depend on the library.

---

## 6.8 Salt

A **salt** is a random value generated independently for each password record.

Hashing becomes:

```text
password_hash = KDF(password, salt, cost_parameters)
```

The salt is not normally secret.

It is stored alongside the password hash.

---

### Why salts exist

Without salts:

```text
Alice password = example123
Bob password   = example123

SHA256(example123) for Alice
=
SHA256(example123) for Bob
```

An attacker can see that both users share a password.

With unique salts:

```text
KDF(example123, salt_A)
≠
KDF(example123, salt_B)
```

The attacker must compute guesses separately for each salt.

---

### What salts prevent

Salts help prevent:

* Identical passwords producing identical stored hashes
* Efficient use of precomputed rainbow tables
* One computation checking the same candidate across all accounts

### What salts do not prevent

Salts do not prevent:

* Guessing common passwords
* Offline attacks against individual hashes
* Keylogging
* Phishing
* Credential stuffing
* Weak password selection
* Database tampering

---

## 6.9 Salt generation

A salt should be generated using a cryptographically secure random-number generator.

Conceptual Python example:

```python
import secrets

salt = secrets.token_bytes(16)
```

### What it does

`secrets.token_bytes(16)` requests 16 bytes from Python’s security-oriented randomness interface.

### Why it is useful

It generates unpredictable material appropriate for salts, tokens, and keys.

### Important detail

A password-hashing library normally generates and encodes the salt automatically.

Prefer the library’s high-level API.

### Safety concern

Do not replace secure randomness with predictable functions such as ordinary simulation-oriented pseudorandom generators.

---

## 6.10 Pepper

A **pepper** is an additional secret value stored separately from the password database.

Conceptually:

```text
password_hash = KDF(password || pepper, salt, parameters)
```

or:

```text
stored_value = HMAC(pepper, password_KDF_output)
```

The exact construction should follow a reviewed library or architecture rather than an improvised design.

---

### Why a pepper exists

If an attacker steals only the database, the pepper remains unavailable.

This can make offline verification more difficult.

The pepper may be stored in:

* Hardware security module
* Dedicated secrets manager
* Protected application environment
* Separate key-management system

---

### Pepper tradeoffs

Advantages:

* Adds protection against database-only compromise

Costs:

* Introduces secret-key management
* May create a runtime dependency
* Makes rotation difficult
* A lost pepper can make all passwords unverifiable
* Compromise of the application runtime may expose pepper access

Salt and pepper are not interchangeable:

| Property            | Salt                                        | Pepper                                     |
| ------------------- | ------------------------------------------- | ------------------------------------------ |
| Unique per password | Usually yes                                 | Usually shared or versioned                |
| Secret              | No                                          | Yes                                        |
| Stored with hash    | Yes                                         | No                                         |
| Main purpose        | Prevent precomputation and cross-user reuse | Add protection against database-only theft |

---

## 6.11 Argon2id mental model

Argon2 is designed to consume both memory and computation.

Argon2id combines properties intended to resist several implementation and attack concerns.

Simplified model:

```text
Password + salt + parameters
→ allocate memory
→ perform repeated memory-dependent transformations
→ produce derived output
```

Why memory matters:

An attacker may use GPUs, FPGAs, or specialized hardware.

If each password guess requires substantial memory, parallel guessing becomes more expensive.

---

## 6.12 Password-hash cost parameters

A password-hash configuration may include:

* Memory cost
* Time or iteration cost
* Parallelism
* Output length

Increasing costs generally improves resistance to offline guessing but also affects:

* Login latency
* CPU utilization
* Memory usage
* Authentication-service capacity
* Denial-of-service exposure
* Serverless execution cost
* Mobile or embedded compatibility

The goal is not “maximum possible cost.”

The goal is:

> The highest safe cost the production system can sustain under expected and adversarial load.

Parameters should be benchmarked on actual deployment hardware.

---

## 6.13 Simplified versus exact Argon2 behavior

### Simplified mental model

```text
Argon2id makes every password guess consume time and memory.
```

### More exact reality

Security depends on:

* Argon2 variant
* Version
* Memory cost
* Time cost
* Parallelism
* Library implementation
* Hardware
* Available attacker hardware
* Password distribution
* Server-side controls

Merely seeing the string `argon2id` does not prove that parameters are strong.

---

## 6.14 bcrypt

bcrypt is a password hashing function with an adjustable work factor.

A stored value commonly contains:

* Version marker
* Cost
* Salt
* Hash output

Conceptual form:

```text
$2b$12$...
```

The cost controls repeated work exponentially.

A cost of 12 does not mean 12 iterations; it represents a logarithmic work factor.

### Important implementation issue

bcrypt has input-length behavior that applications must understand.

Libraries and systems may handle long passwords differently.

Applications should:

* Follow library documentation
* Avoid silent truncation
* Test maximum accepted length
* Apply consistent normalization and encoding
* Avoid inventing unreviewed preprocessing

---

## 6.15 scrypt

scrypt is a memory-hard password-based key-derivation function.

It uses parameters controlling:

* CPU and memory cost
* Block size
* Parallelization

It is used for password storage and key derivation in various systems.

As with Argon2id, safe configuration requires benchmarking and maintained implementations.

---

## 6.16 PBKDF2

PBKDF2 repeatedly applies a pseudorandom function such as HMAC.

Conceptually:

```text
password + salt
→ repeated HMAC computations
→ derived output
```

It is widely supported and may be required by certain platforms or compliance environments.

Compared with memory-hard alternatives, PBKDF2 primarily increases computation rather than requiring substantial memory.

Its safety depends heavily on:

* High enough iteration count
* Appropriate pseudorandom function
* Salt quality
* Correct implementation
* Periodic parameter review

---

## 6.17 Password verification step by step

Given:

```text
submitted_password
stored_encoded_hash
```

The verifier performs:

1. Parse the encoded hash.
2. Identify algorithm and version.
3. Extract salt.
4. Extract cost parameters.
5. Run the same password hashing function.
6. Compare derived output with stored output.
7. Return match or mismatch.
8. Determine whether rehashing is needed.

Pseudocode:

```python
def verify_login(username: str, password: str) -> bool:
    user = load_user_by_normalized_username(username)

    if user is None:
        perform_dummy_password_hash(password)
        return False

    if user.status != "active":
        return False

    if not password_hasher.verify(user.password_hash, password):
        return False

    if password_hasher.needs_rehash(user.password_hash):
        update_password_hash(
            user.id,
            password_hasher.hash(password),
        )

    return True
```

This is illustrative. Production code also needs:

* Rate limiting
* Audit logging
* Error handling
* MFA state
* Transaction safety
* Secret redaction
* Session creation
* Monitoring
* Race-condition handling

---

## 6.18 Rehash on login

Password-hash parameters become outdated as hardware improves.

A practical migration pattern is:

```text
User successfully logs in
→ verifier notices stored parameters are old
→ hash submitted password using current parameters
→ replace stored hash
```

This is called **rehashing on login**.

Advantages:

* Gradual migration
* No plaintext password storage
* No forced reset for every active user

Limitations:

* Inactive accounts never migrate
* Update failures need handling
* Multiple concurrent logins may race
* Old hashes remain until next successful login
* Compromised old hashes are still vulnerable

---

### Algorithm migration example

Stored record:

```text
algorithm = bcrypt
cost = 10
```

Current policy:

```text
algorithm = argon2id
memory = approved current value
time = approved current value
```

Flow:

```mermaid
sequenceDiagram
    participant User
    participant App
    participant DB
    participant Old as bcrypt verifier
    participant New as Argon2id hasher

    User->>App: Submit password
    App->>DB: Load bcrypt hash
    DB-->>App: Existing hash
    App->>Old: Verify password
    Old-->>App: Valid
    App->>New: Hash same submitted password
    New-->>App: New Argon2id hash
    App->>DB: Replace old hash
    App-->>User: Login succeeds
```

---

## 6.19 Dummy verification for unknown accounts

Suppose account lookup returns immediately for unknown usernames, but password verification takes 150 milliseconds for known users.

An attacker could measure response times:

```text
10 ms → account probably absent
160 ms → account probably present
```

A mitigation is to run a dummy password-hash verification for missing accounts.

Conceptually:

```python
DUMMY_HASH = "$argon2id$...valid encoded dummy hash..."

if user is None:
    password_hasher.verify(DUMMY_HASH, submitted_password)
    return generic_failure()
```

This reduces an obvious timing difference.

It does not guarantee perfect indistinguishability because other code paths may still differ.

---

## 6.20 Constant-time comparison

A naïve byte comparison may return at the first mismatch.

Conceptually:

```text
compare byte 1
if different, return
compare byte 2
if different, return
...
```

Execution time may reveal how much of a value matched.

Cryptographic libraries use comparison functions designed to avoid obvious data-dependent early exit.

For password hashing, use the library’s complete verification API.

Do not manually compare encoded password hashes.

---

## 6.21 Username normalization

Before account lookup, the application must define how usernames are compared.

Questions include:

* Are usernames case-sensitive?
* Are leading or trailing spaces allowed?
* Are Unicode characters normalized?
* Are email addresses treated as usernames?
* Can the username change?
* Are aliases allowed?
* Is the identity key actually a stable internal ID?

Example:

```text
Alice@example.com
alice@example.com
alice@example.com<space>
```

Whether these represent the same account depends on explicit system rules.

Inconsistent normalization can cause:

* Duplicate accounts
* Account confusion
* Authentication bypass
* Recovery misrouting
* Authorization mismatches
* Audit ambiguity

A safer architecture uses an immutable internal identifier after lookup:

```text
submitted login name
→ normalized lookup key
→ immutable user ID u_1048
```

Subsequent authorization should use the immutable ID.

---

## 6.22 Unicode and password handling

Passwords should generally be treated as opaque user-entered strings or byte sequences according to a clearly defined policy.

Potential problems include:

* Different Unicode representations
* Inconsistent encoding between clients and servers
* Silent trimming
* Lowercasing
* Maximum-length truncation
* Null-character handling
* Library-specific conversion

Changing password normalization after users have enrolled can make existing passwords unverifiable.

Do not silently transform passwords unless the system has a deliberate, documented design.

---

## 6.23 Maximum password length

A login endpoint needs request-size limits to prevent resource exhaustion.

However, extremely small password maximums can:

* Prevent use of password-manager-generated values
* Encourage weaker choices
* Conflict with passphrases
* Reveal legacy algorithm constraints

The application should choose a generous but bounded maximum and reject oversized input before expensive hashing.

Conceptual flow:

```text
Request body limit
→ parse input
→ password length validation
→ rate limiting
→ expensive password verification
```

The exact limit should reflect:

* Library behavior
* Encoding
* Protocol constraints
* Denial-of-service analysis
* User experience

---

## 6.24 Password policy

Password policy attempts to reduce predictable password selection.

Possible controls:

* Minimum length
* Block known compromised passwords
* Permit password-manager-generated values
* Avoid arbitrary composition rules
* Prevent reuse of recent passwords where required
* Require MFA for sensitive accounts
* Apply stronger protection to administrative access

Rigid rules such as:

```text
one uppercase
one lowercase
one digit
one symbol
```

often produce predictable patterns:

```text
Password1!
Summer2026!
CompanyName7#
```

Length and resistance to known-compromised-password reuse are generally more meaningful than forcing superficial complexity patterns.

---

## 6.25 Credential stuffing

Credential stuffing uses username-password pairs stolen from other services.

Flow:

```mermaid
flowchart LR
    L[(Leaked credential list)]
    B[Automated login bot]
    A[Target application]
    M[Matching reused accounts]

    L --> B
    B --> A
    A --> M
```

The target application’s password database does not need to be compromised.

The attack succeeds because users reuse credentials.

Defenses include:

* MFA
* Passkeys
* Compromised-password screening
* Rate limits
* Bot detection
* Login anomaly detection
* User notifications
* Password-manager adoption
* Device and risk signals

---

## 6.26 Password spraying

Password spraying tries a small number of common passwords against many accounts.

Example:

```text
Try Welcome2026! against 10,000 usernames
```

This avoids triggering per-account lockout thresholds quickly.

Defenses must consider multiple dimensions:

* Attempts per account
* Attempts per IP
* Attempts per network
* Attempts per device fingerprint
* One password tried across many accounts
* Organization-wide failure rates
* Time-window patterns

---

## 6.27 Brute-force attacks

A brute-force attack systematically explores candidate values.

For online authentication, the attacker is constrained by:

* Network latency
* Rate limits
* Lockouts
* Detection
* Server processing

For stolen hashes, the attacker is constrained by:

* Hash cost
* Hardware
* Password entropy
* Number of hashes
* Salt uniqueness
* Pepper availability

The same word “brute force” may refer to very different operational situations.

---

## 6.28 Rate limiting

Rate limiting restricts authentication attempts over time.

Possible keys:

```text
source IP
username
account ID
device identifier
network prefix
organization
endpoint
global service
```

No single key is sufficient.

### Per-IP only

Attackers can distribute attempts across many addresses.

### Per-account only

Attackers can cause denial of service by intentionally locking victims.

### Global only

A large attack may block all legitimate authentication.

A layered design combines several signals.

---

## 6.29 Token-bucket mental model

A token bucket can model allowed login attempts.

```text
Bucket capacity = 5
Refill rate = 1 token per minute
Each attempt consumes 1 token
No token → reject or delay
```

```mermaid
flowchart LR
    T[Incoming login attempt]
    B{Token available?}
    A[Allow verification]
    R[Reject or delay]
    F[Periodic refill]

    T --> B
    B -->|Yes| A
    B -->|No| R
    F --> B
```

Rate limits should be observable through metrics and structured events.

---

## 6.30 Account lockout

Account lockout disables login after repeated failures.

Example:

```text
Five failed attempts
→ lock account for fifteen minutes
```

Benefits:

* Slows online guessing
* Creates visible security signals

Risks:

* Attackers can lock other users out
* Help-desk load increases
* Distributed counters may be inconsistent
* Attackers can rotate attempts below thresholds

Alternatives or complements:

* Progressive delay
* Risk-based challenges
* Temporary throttling
* MFA
* Notification
* Device verification
* Passwordless mechanisms

---

## 6.31 Progressive delay

Instead of hard lockout:

```text
Failure 1 → immediate
Failure 2 → 1 second
Failure 3 → 2 seconds
Failure 4 → 4 seconds
Failure 5 → 8 seconds
```

This slows automated attempts while avoiding permanent account denial.

The delay should not consume an application worker synchronously if that creates a denial-of-service vulnerability.

A system can record a future permitted-attempt time and reject early until then.

---

## 6.32 Authentication denial of service

Password hashing is intentionally expensive.

An attacker can exploit this by submitting many login attempts.

```text
Cheap HTTP request
→ expensive Argon2 computation
→ CPU and memory exhaustion
```

Defenses include:

* Request-size limits
* Network-level controls
* Rate limiting before hashing
* Bounded authentication worker pools
* Queues
* Circuit breakers
* Autoscaling with cost controls
* Per-account and per-source limits
* Capacity monitoring
* Separate authentication infrastructure

Do not reduce password-hash cost as the first response to attack traffic without analyzing the security tradeoff.

---

## 6.33 Password reset flow

A typical email-reset flow:

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant App
    participant DB
    participant Mail
    participant Inbox

    User->>Browser: Request password reset
    Browser->>App: Submit account identifier
    App->>DB: Resolve account
    App->>App: Generate random reset token
    App->>DB: Store token hash, account, expiry
    App->>Mail: Send reset link
    Mail->>Inbox: Deliver link
    User->>Browser: Open link
    Browser->>App: Present reset token
    App->>DB: Validate token hash and expiry
    App->>App: Accept new password
    App->>DB: Store new password hash
    App->>DB: Mark token used
    App->>App: Revoke sessions as policy requires
```

---

## 6.34 Reset-token storage

A password reset token is a bearer credential.

Anyone who obtains it may be able to reset the password.

A strong design stores:

```text
token identifier
token hash
user ID
created time
expiry time
used status
request metadata
```

The email contains the raw token.

The database can store only a hash of that token.

This resembles session or API-key storage.

---

## 6.35 Reset-token requirements

A reset token should be:

* Random
* Unpredictable
* Single-use
* Short-lived
* Bound to the intended account
* Invalidated after use
* Protected from logging
* Sent only over HTTPS in the final link
* Invalidated when superseded if policy requires

The reset page should avoid leaking the token through:

* Referrer headers
* Analytics scripts
* Third-party assets
* Browser history where practical
* Proxy logs
* Application logs
* Error-reporting systems

---

## 6.36 Generic reset response

Requesting reset for an unknown account should not obviously reveal whether the account exists.

External response:

```text
If an account matches that identifier, reset instructions will be sent.
```

Internally:

```text
event=reset_requested
account_found=false
normalized_identifier_hash=...
```

Uniform messaging reduces straightforward enumeration, though timing and delivery behavior may still leak information.

---

## 6.37 Session handling after password reset

Possible policies:

### Revoke all sessions

```text
Password changed
→ all existing sessions invalidated
```

Security benefit:

* Removes an attacker who already has a session.

Usability cost:

* Logs the user out everywhere.

### Preserve current session only

```text
Password changed from authenticated settings page
→ preserve current verified session
→ revoke other sessions
```

This can balance usability and compromise response.

### Preserve all sessions

This is simpler but may leave stolen sessions active.

The correct choice depends on whether the change was:

* Routine
* Recovery-driven
* Administrator-forced
* Compromise-related

---

## 6.38 Password change versus password reset

### Password change

User is already authenticated and provides:

* Current password or step-up authentication
* New password

### Password reset

User cannot use the normal credential and follows a recovery path.

Reset should be treated as higher risk because the normal proof is unavailable.

Additional controls may include:

* Stronger notification
* Session revocation
* Delay for high-risk accounts
* Support review
* Recovery-key requirement
* Restriction on sensitive actions for a period

---

## 6.39 Password history

Some systems prohibit recent password reuse.

A naïve implementation stores old plaintext passwords, which is unacceptable.

A safer design stores historical password hashes and verifies the proposed new password against each relevant old hash.

However, this creates:

* Additional expensive hash operations
* More sensitive stored material
* Migration complexity
* Potential denial-of-service cost

Password history should be used only when justified by policy and threat model.

---

## 6.40 Logs for password authentication

A useful failure event might contain:

```json
{
  "timestamp": "2026-07-31T10:14:52Z",
  "event_type": "authentication_failure",
  "account_id": "u_1048",
  "submitted_identifier_hash": "sha256:...",
  "source_ip": "192.0.2.51",
  "user_agent_family": "ExampleBrowser",
  "reason": "password_mismatch",
  "rate_limit_state": "allowed",
  "request_id": "req_91ac..."
}
```

The external response may remain generic while the internal reason is specific.

Never log:

```text
password=Summer2026!
submitted_totp=123456
reset_token=...
session_id=...
```

---

## 6.41 Illustrative login logs

```text
2026-07-31T10:14:52Z level=INFO
event=login_attempt
request_id=req_91ac
account_id=u_1048
source_ip=192.0.2.51

2026-07-31T10:14:52Z level=WARN
event=password_verification_failed
request_id=req_91ac
account_id=u_1048
hash_algorithm=argon2id
rate_limit_remaining=3

2026-07-31T10:14:52Z level=INFO
event=login_response
request_id=req_91ac
status=401
duration_ms=184
```

Observations:

* The password is absent.
* The algorithm may be logged for migration analysis.
* The internal reason is visible to operators.
* A shared request ID correlates the stages.
* Duration supports latency and anomaly analysis.

---

## 6.42 Practical password-hash testing with Python

The exact library API depends on the installed package.

An illustrative example using a maintained Argon2 library:

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

hasher = PasswordHasher()

stored_hash = hasher.hash("example password")

try:
    valid = hasher.verify(stored_hash, "example password")
except VerifyMismatchError:
    valid = False

print(valid)
```

### What it demonstrates

* Hash generation creates a salted encoded value.
* Verifying the correct password returns success.
* Verification uses parameters encoded in the stored hash.

### Safety

Do not use real passwords in:

* Source code
* Shell history
* Shared notebooks
* Screenshots
* Demonstration logs

This example demonstrates library behavior, not a complete authentication service.

---

## 6.43 Inspecting shadow password records on Linux

On Linux systems using local account files, password hash information is commonly stored in `/etc/shadow`.

A privileged inspection command:

```bash
sudo getent shadow alice
```

### What it does

Queries the configured system databases for the shadow entry of user `alice`.

`getent` may consult sources configured through the Name Service Switch rather than reading only one local file.

### Why it is useful

It helps determine:

* Whether a local shadow entry exists
* Which hash identifier is stored
* Password-age fields
* Account-expiry state

### Important considerations

* Root privileges are generally required to view shadow data.
* The result is security-sensitive.
* Do not paste the full hash into tickets or chat systems.
* Behavior varies with NSS configuration and operating system.

Illustrative redacted output:

```text
alice:$y$j9T$REDACTED_HASH:20665:0:99999:7:::
```

Field meanings depend on the shadow-file format and system implementation.

The second field contains password state or encoded hash information.

Special markers may indicate:

* Locked password
* No usable password
* Implementation-specific account state

Do not edit `/etc/shadow` manually unless performing controlled system recovery with full understanding of the format and locking requirements.

---

## 6.44 Inspecting password aging

```bash
sudo chage -l alice
```

### What it does

Displays password aging and account-expiry information for `alice`.

### Why it is useful

It helps diagnose:

* Forced password change
* Password expiry
* Minimum change interval
* Warning period
* Account expiry

### Important flag

* `-l`: list aging information

Illustrative output:

```text
Last password change                                    : Jul 20, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

Exact formatting varies.

### Safety

Changing password aging can lock users out or weaken policy. Listing information is safer than modifying it.

---

## 6.45 Locking a Linux password

```bash
sudo passwd -l alice
```

### What it does

Locks the password authentication path for the account by modifying its password field according to the implementation.

### Important flag

* `-l`: lock the password

### What it does not necessarily do

It may not prevent all access.

The user might still authenticate through:

* SSH public key
* Existing sessions
* Kerberos
* Sudo configuration
* Other PAM modules
* Scheduled jobs
* Service credentials

This illustrates a core principle:

> Disabling one credential is not necessarily equivalent to disabling the identity.

### Safety

This is a state-changing administrative command and may disrupt access.

Verify that an emergency or alternate administrative path exists before changing privileged accounts.

---

## 6.46 Checking password status

```bash
sudo passwd -S alice
```

### What it does

Displays password status information.

### Important flag

* `-S`: report password status

Illustrative output:

```text
alice P 2026-07-20 0 99999 7 -1
```

The status marker and date fields are implementation-dependent.

Common status concepts include:

* Usable password
* Locked password
* No password

Consult the operating system’s manual for exact semantics.

---

## 6.47 PAM’s role

On many Linux systems, applications do not verify local passwords directly.

They use **PAM**, the Pluggable Authentication Modules framework.

Applications such as:

* `login`
* `sudo`
* `sshd`
* Display managers
* Screen lockers

can invoke a configured PAM service stack.

Simplified flow:

```mermaid
flowchart LR
    A[Application]
    P[PAM framework]
    M1[pam_unix]
    M2[pam_faillock]
    M3[pam_sss or directory module]
    D[(Local or remote identity source)]

    A --> P
    P --> M1
    P --> M2
    P --> M3
    M1 --> D
    M3 --> D
```

PAM can coordinate:

* Authentication
* Account validity
* Password changes
* Session setup

PAM behavior depends heavily on distribution and configuration.

The application receives success or failure from the PAM stack rather than reading `/etc/shadow` itself.

---

## 6.48 PAM stages

PAM separates several management groups:

| PAM group  | Purpose                                   |
| ---------- | ----------------------------------------- |
| `auth`     | Verify authentication evidence            |
| `account`  | Check whether account access is allowed   |
| `password` | Change authentication tokens              |
| `session`  | Set up or tear down session-related state |

This distinction is important.

A password can be correct while account access is denied because:

* Account expired
* Login time restricted
* Host policy denies access
* User not permitted through SSH configuration
* Required group membership absent

Therefore:

```text
Password verification success
≠
Complete login success
```

---

## 6.49 Observing PAM logs

Depending on distribution and logging configuration, authentication events may appear in:

```bash
sudo journalctl -u ssh
```

or:

```bash
sudo journalctl -u sshd
```

or traditional log files such as:

```text
/var/log/auth.log
/var/log/secure
```

### `journalctl -u`

Shows logs for a named systemd unit.

Example:

```bash
sudo journalctl -u sshd --since "10 minutes ago"
```

### Important options

* `-u sshd`: select the service unit
* `--since`: restrict the time range

### Why it is useful

It may reveal:

* Invalid user
* Password failure
* PAM account rejection
* Public-key rejection
* Session creation
* Connection source

### Safety

Authentication logs contain:

* Usernames
* Source addresses
* Internal hostnames
* Timing information

Share only the necessary redacted lines.

---

## 6.50 Illustrative SSH password failure

```text
Jul 31 15:21:18 server sshd[28173]:
pam_unix(sshd:auth): authentication failure;
user=alice rhost=192.0.2.51

Jul 31 15:21:20 server sshd[28173]:
Failed password for alice from 192.0.2.51 port 54218 ssh2
```

Possible interpretation:

1. SSH transport and protocol connection succeeded.
2. User `alice` was selected.
3. Password authentication was attempted.
4. PAM password verification failed.
5. No authenticated SSH session was created.

This does not prove whether the cause was:

* Incorrect password
* Locked password
* PAM module policy
* Directory outage
* Account state
* Authentication-method configuration

More logs and configuration context may be required.

---

## 6.51 Debugging password login systematically

### Stage 1: Request reached the service

Check:

* Network connectivity
* TLS termination
* Reverse proxy logs
* Correct hostname
* Correct login endpoint

### Stage 2: Identifier resolution

Check:

* Username normalization
* Directory lookup
* Tenant selection
* Account duplication
* NSS or LDAP availability

### Stage 3: Credential verification

Check:

* Hash format
* Password input encoding
* Account password state
* PAM module result
* Hash-library errors
* Pepper access

### Stage 4: Account policy

Check:

* Disabled or expired account
* Allowed login time
* Required group
* Host restrictions
* MFA requirement

### Stage 5: Session creation

Check:

* Session-store availability
* Secure randomness
* Cookie construction
* Database transaction
* Signing-key access

### Stage 6: Session return

Check:

* `Set-Cookie`
* HTTPS
* Cookie domain and path
* `Secure`
* `SameSite`
* Browser policy
* Proxy header rewriting

---

## 6.52 Common password-system misconceptions

### “The database contains hashes, so compromise is harmless”

False.

Attackers can perform offline guessing.

Security depends on:

* Password quality
* Hash algorithm
* Cost parameters
* Salt
* Pepper
* Attacker hardware
* Database size

---

### “Hashing twice is automatically more secure”

Not necessarily.

Improvised constructions can:

* Break migration
* Reduce compatibility
* Add no meaningful security
* Interact badly with length limits
* Make verification error-prone

Use a standard password-hashing construction with deliberate parameters.

---

### “Changing passwords regularly always improves security”

Frequent forced changes may produce predictable variations and user workarounds.

Password changes are most valuable when:

* Compromise is suspected
* Credential exposure occurred
* A weak password is detected
* Policy or role changes require it
* The user requests a change

Exact organizational requirements may differ.

---

### “Account lockout stops brute force”

It slows some online attacks but can enable denial of service and does nothing against offline cracking.

---

### “MFA makes password storage unimportant”

False.

Passwords may still be:

* Reused elsewhere
* Used in recovery
* Used where MFA is bypassed
* Valuable to attackers
* Exposed in logs
* Used against legacy protocols

---

### “A successful password check means login succeeded”

False.

The system may still reject access because of:

* MFA failure
* Account policy
* device policy
* disabled state
* session-store failure
* authorization policy

---

## 6.53 Password authentication performance

Password hashing creates intentional latency.

Suppose verification takes:

```text
150 ms
```

A single worker can theoretically process only a limited number of sequential verifications per second.

With concurrency, capacity depends on:

* CPU cores
* Memory per hash
* Parallelism settings
* Worker count
* Runtime overhead
* Database latency
* MFA and directory calls
* Rate-limit architecture

A benchmark should measure:

* Median verification time
* High-percentile latency
* Memory usage
* Throughput
* Behavior under concurrent login bursts
* Behavior under attack traffic
* Impact on unrelated requests

Authentication endpoints are often separated from ordinary application workers to contain resource pressure.

---

## 6.54 Authentication metrics

Useful metrics include:

```text
login_attempts_total
login_success_total
login_failure_total
password_hash_duration_seconds
account_lookup_duration_seconds
rate_limit_rejections_total
session_creation_failures_total
mfa_challenges_total
password_rehash_total
directory_errors_total
```

Dimensions should be chosen carefully.

Avoid unbounded high-cardinality labels such as:

```text
username
session_id
raw IP address in every metric label
request ID
```

These can overwhelm metrics systems and leak sensitive information.

Detailed identifiers belong in controlled logs or traces, not general metric labels.

---

## 6.55 Example troubleshooting scenario

### Symptom

Users report:

```text
Correct password returns login page again.
```

### Possible causes

1. Password verification fails.
2. MFA transaction is not stored.
3. Session creation fails.
4. Session cookie is not returned.
5. Browser rejects the cookie.
6. Cookie path does not include the target route.
7. Cookie has `Secure`, but test environment uses HTTP.
8. Reverse proxy removes `Set-Cookie`.
9. Load-balanced instances use inconsistent signing keys.
10. Session is created on one node but read from another node without shared storage.
11. Application immediately marks session expired because clocks differ.
12. Account policy rejects the login after password success.

### Debugging path

```text
Check authentication event
→ check session-creation event
→ inspect Set-Cookie response
→ inspect browser cookie store
→ inspect next request Cookie header
→ inspect session lookup
→ inspect expiry and account policy
```

Do not start by resetting the password unless evidence points to password verification.

---

## Questions: Password Authentication Internals

1. Why does a unique salt prevent one candidate computation from being reused against every account?
2. Why does a salt not protect a weak password from individual offline guessing?
3. What security benefit does a pepper provide, and what operational failure can a lost pepper cause?
4. Why should password-hash parameters be selected through production benchmarking rather than copied blindly?
5. How can an attacker turn expensive password hashing into a denial-of-service mechanism?
6. Why should request limits and rate limits be applied before expensive password verification?
7. How does rehash-on-login migrate algorithms without storing plaintext passwords?
8. What happens to inactive accounts during a rehash-on-login migration?
9. Why can account-enumeration timing persist even when the application returns identical error messages?
10. How could inconsistent username normalization create two identities that appear identical to humans?
11. Why is password reset an alternative authentication protocol rather than merely an account-management feature?
12. What should happen to active sessions after a recovery-driven password reset?
13. Why may locking a Linux password fail to prevent SSH public-key access?
14. Which PAM stage can reject a user even after the password has been verified successfully?
15. How would you distinguish a failed password check from a failed session-cookie write?
16. Why are full usernames and request IDs often inappropriate as metric labels but useful in controlled logs?
17. How could a pepper stored in the same application environment as the database credentials still provide limited defense?
18. Why is a fast general-purpose hash unsuitable for human passwords even when every account has a unique salt?
19. What race conditions could occur when two successful logins both attempt to upgrade the same stored password hash?
20. Why might reducing Argon2 memory cost improve availability while decreasing protection against offline cracking?

# 7. Sessions, Cookies, and Tokens

Initial authentication answers:

```text
Did the subject successfully prove an identity?
```

A session or token allows later requests to reuse that result without resending the original password, passkey assertion, or MFA code.

The central transition is:

```text
Authentication event
→ authenticated security context
→ reusable request credential
```

---

## 7.1 Why sessions exist

HTTP is fundamentally request-response oriented.

A request does not automatically inherit identity from a previous request.

Without a session mechanism:

```text
Request 1: Alice logs in
Request 2: server sees an unrelated request
Request 3: server sees another unrelated request
```

The server needs a way to associate later requests with Alice’s successful login.

A session provides that continuity.

```mermaid
sequenceDiagram
    actor Alice
    participant Browser
    participant App
    participant SessionStore

    Alice->>Browser: Enter credentials
    Browser->>App: Login request
    App->>App: Verify credentials
    App->>SessionStore: Create session for Alice
    App-->>Browser: Return session credential

    Browser->>App: Request + session credential
    App->>SessionStore: Resolve session
    SessionStore-->>App: Alice's security context
    App-->>Browser: Authenticated response
```

---

## 7.2 Session versus session credential

These are related but distinct.

### Session

Server-side or logically maintained authentication state.

Example:

```json
{
  "session_id_hash": "sha256:...",
  "subject_id": "u_1048",
  "created_at": "2026-07-31T09:16:12Z",
  "idle_expiry": "2026-07-31T09:46:12Z",
  "absolute_expiry": "2026-07-31T17:16:12Z",
  "authentication_methods": ["password", "totp"],
  "status": "active"
}
```

### Session credential

The value presented by the client to identify or prove possession of the session.

Example:

```text
session_id=s_E5f8m4...
```

The client usually holds the session credential.

The server holds the associated session state.

> Stealing the session credential may be enough to impersonate the user even if the attacker does not know the original password.

---

## 7.3 Opaque session identifiers

An opaque session identifier is a random value with no client-visible meaning.

Example:

```text
s_W7G6vH2qfC8n...
```

The client cannot decode the identity or permissions from it.

The server uses it as a lookup key.

```mermaid
flowchart LR
    C[Client]
    ID[Opaque session ID]
    S[(Session Store)]
    CTX[Security Context]

    C -->|Cookie| ID
    ID -->|Lookup| S
    S --> CTX
```

### Advantages

* Easy central revocation
* Session data can change immediately
* Small client credential
* Sensitive metadata stays server-side
* Straightforward inactivity tracking

### Costs

* Requires session-store access
* Shared storage may be needed across application instances
* Store failure may affect all authenticated requests
* Replication and consistency must be designed

---

## 7.4 Generating a session identifier

A session ID must be:

* Unpredictable
* Generated using a cryptographically secure random source
* Long enough to resist guessing
* Independent of user-controlled data
* Rotated at security-sensitive transitions

A practical local generation command:

```bash
openssl rand -base64 32
```

### What it does

Generates 32 cryptographically secure random bytes and encodes them using Base64.

### Why it is useful

It demonstrates the type of randomness required for:

* Session identifiers
* Reset tokens
* API keys
* Nonces

### Important arguments

* `rand`: generate random bytes
* `-base64`: encode output as Base64
* `32`: number of random bytes before encoding

Illustrative output:

```text
8Zi2M8xH5B+zPru31fDeVZz8hTvz4TfR0f5BmKwfRhA=
```

### Safety

Do not manually generate production session IDs in shell scripts unless the architecture specifically requires it.

Application frameworks usually provide safer session APIs.

Standard Base64 includes characters such as `/`, `+`, and `=`. URL-safe encodings may be more convenient for cookies and URLs.

---

## 7.5 Session lookup

A request arrives with:

```http
Cookie: session_id=s_W7G6vH2qfC8n...
```

The application may compute:

```text
lookup_key = SHA-256(session_id)
```

and query:

```text
lookup_key
→ session record
```

Storing a hash rather than the raw session ID can reduce the usefulness of a session-store leak.

However:

* A weak or guessable session ID remains unsafe.
* The application still receives the raw ID.
* Logs can still leak the raw credential.
* A compromised runtime can still capture it.
* Hashing does not provide expiry or revocation by itself.

---

## 7.6 Session creation step by step

After successful authentication:

1. Generate a new random session ID.
2. Create server-side session state.
3. Bind it to the authenticated subject.
4. Record authentication method and time.
5. Set expiry values.
6. Store status as active.
7. Send the ID to the client in a protected cookie.
8. Remove temporary login-state credentials.
9. Record the session creation event.

Conceptual pseudocode:

```python
def create_session(user, authentication_context):
    raw_id = secure_random_token()

    session = {
        "id_hash": sha256(raw_id),
        "subject_id": user.id,
        "created_at": now(),
        "last_seen_at": now(),
        "absolute_expiry": now() + HOURS_8,
        "idle_expiry": now() + MINUTES_30,
        "auth_methods": authentication_context.methods,
        "status": "active",
    }

    session_store.insert(session)

    return make_cookie(
        name="session_id",
        value=raw_id,
        secure=True,
        http_only=True,
        same_site="Lax",
    )
```

Real code must also address:

* Transaction failures
* Duplicate identifiers
* Key rotation
* Logging redaction
* Concurrent sessions
* Device metadata
* Revocation
* Session limits

---

## 7.7 Session fixation

A session fixation attack causes a victim to authenticate using a session identifier already known to the attacker.

### Vulnerable flow

```text
1. Attacker obtains session ID X.
2. Attacker causes Alice's browser to use X.
3. Alice logs in.
4. Server upgrades session X to authenticated.
5. Attacker uses X as Alice.
```

```mermaid
sequenceDiagram
    actor Attacker
    actor Alice
    participant Browser
    participant App

    Attacker->>Browser: Cause browser to use session X
    Alice->>App: Log in using session X
    App->>App: Mark session X authenticated
    Attacker->>App: Reuse session X
    App-->>Attacker: Alice's authenticated access
```

### Defense

Generate a new session identifier after authentication.

```text
Pre-login session X
→ authentication succeeds
→ invalidate X
→ create authenticated session Y
```

Session IDs should also be rotated after:

* Privilege elevation
* Account recovery
* MFA completion
* Sensitive identity changes
* Administrative impersonation start or end

---

## 7.8 Cookie fundamentals

A cookie is a small name-value item stored by a user agent and sent with matching HTTP requests.

Server response:

```http
Set-Cookie: session_id=s_W7G6v...; Secure; HttpOnly; SameSite=Lax; Path=/
```

Later request:

```http
Cookie: session_id=s_W7G6v...
```

The browser applies cookie matching rules based on:

* Domain
* Path
* Scheme security
* Expiry
* SameSite policy
* Browser rules

---

## 7.9 Important cookie attributes

| Attribute     | Main effect                                                 |
| ------------- | ----------------------------------------------------------- |
| `Secure`      | Cookie is sent only over secure HTTPS connections           |
| `HttpOnly`    | Normal client-side JavaScript cannot read the cookie        |
| `SameSite`    | Restricts some cross-site sending behavior                  |
| `Domain`      | Controls which hosts receive the cookie                     |
| `Path`        | Restricts URL paths receiving the cookie                    |
| `Max-Age`     | Defines lifetime in seconds                                 |
| `Expires`     | Defines expiry at a specific time                           |
| `Partitioned` | Supports partitioned cookie behavior in compatible browsers |

Browser behavior evolves, so production design must be tested against supported browsers.

---

## 7.10 `Secure`

Example:

```http
Set-Cookie: session_id=...; Secure
```

The browser sends the cookie only over HTTPS.

Without `Secure`, a browser might send the cookie over an unencrypted HTTP request.

`Secure` does not protect against:

* Server-side logging
* Browser compromise
* Malicious extensions
* XSS-driven authenticated actions
* Application bugs
* Theft from unprotected backups

---

## 7.11 `HttpOnly`

Example:

```http
Set-Cookie: session_id=...; HttpOnly
```

This prevents normal page JavaScript from reading:

```javascript
document.cookie
```

for that cookie.

It reduces direct cookie theft through many XSS attacks.

It does not stop malicious JavaScript from causing the browser to send authenticated requests.

Example:

```javascript
fetch("/account/delete", {
  method: "POST"
});
```

The browser may attach the `HttpOnly` cookie automatically.

Therefore:

```text
HttpOnly reduces credential extraction
but
does not remove XSS risk
```

---

## 7.12 `SameSite`

Typical values:

* `Strict`
* `Lax`
* `None`

### `Strict`

The browser strongly restricts cookie sending on cross-site navigation.

Security benefit:

* Stronger CSRF resistance

Usability concern:

* Legitimate cross-site entry flows may lose session context

### `Lax`

Allows cookies in some top-level navigation cases but restricts many cross-site subrequests.

Often a practical default for ordinary web sessions.

### `None`

Allows cross-site cookie use.

It normally requires:

```text
SameSite=None; Secure
```

This may be needed for:

* Embedded applications
* Cross-site identity flows
* Third-party integrations

It increases the need for deliberate CSRF protection.

---

## 7.13 Cookie domain scope

Example:

```http
Set-Cookie: session_id=...; Domain=example.com
```

This may make the cookie available to multiple subdomains.

Possible recipients include:

```text
app.example.com
admin.example.com
legacy.example.com
```

A compromise of a less-trusted subdomain may affect broader session security.

Prefer the narrowest domain scope that supports the application.

A host-only cookie omits `Domain`:

```http
Set-Cookie: session_id=...; Path=/; Secure; HttpOnly
```

It is restricted to the host that set it.

---

## 7.14 Cookie path scope

Example:

```http
Set-Cookie: admin_session=...; Path=/admin
```

The browser sends it only for matching paths.

Path scoping can reduce unnecessary exposure, but it is not a strong isolation mechanism between mutually distrustful applications on the same origin.

Applications sharing one origin also share important browser security boundaries such as:

* JavaScript origin
* Storage context
* Service workers
* Some cookie interactions

Use separate origins when strong isolation is required.

---

## 7.15 Cookie prefixes

Some browsers support cookie name prefixes with additional requirements.

Examples:

```http
Set-Cookie: __Secure-session=...; Secure
```

```http
Set-Cookie: __Host-session=...; Secure; Path=/
```

A `__Host-` cookie is intended to have stricter host scoping and should not include a `Domain` attribute.

These prefixes can help prevent certain cookie-overwrite and scope mistakes.

They are defense in depth, not a replacement for secure application design.

---

## 7.16 Session cookies versus persistent cookies

### Session cookie

No explicit long-term expiry is set.

The browser treats it as a browsing-session cookie.

Behavior across browser restart may vary due to session-restoration features.

### Persistent cookie

Contains `Max-Age` or `Expires`.

Example:

```http
Set-Cookie: session_id=...; Max-Age=28800
```

The browser may retain it across restarts until expiry.

Persistent sessions improve convenience but increase the time a stolen browser profile or device may retain access.

---

## 7.17 Idle and absolute expiry

A robust session often has two time limits.

### Idle expiry

Session expires after inactivity.

Example:

```text
No accepted request for 30 minutes
→ expire session
```

### Absolute expiry

Session expires after a maximum total lifetime.

Example:

```text
Session older than 8 hours
→ expire regardless of activity
```

Combined policy:

```text
valid if:
  now < absolute_expiry
  AND now < idle_expiry
```

Without an absolute limit, continuous activity could keep a stolen session alive indefinitely.

---

## 7.18 Updating idle expiry

The server may update `last_seen_at` on each request.

Naïve design:

```text
Every request
→ write session record
```

This can produce high write load.

Possible optimization:

```text
Update last_seen_at only if more than 5 minutes old
```

Tradeoff:

* Fewer writes
* Less precise inactivity timing

Another approach stores activity in a fast cache and persists less frequently.

---

## 7.19 Concurrent sessions

A user may have sessions on:

* Laptop
* Phone
* Tablet
* Multiple browsers
* API client
* Administrative workstation

A session record should be independently identifiable.

Example:

```text
session A → laptop
session B → phone
session C → browser profile
```

Useful user controls:

* List active sessions
* Show approximate creation time
* Show device or client information
* Revoke one session
* Revoke all other sessions
* Detect unusual activity

Device labels are hints, not reliable proof.

User-agent strings and IP addresses can change or be forged.

---

## 7.20 Session revocation

Revocation can occur because of:

* Logout
* Password reset
* Account disablement
* Suspected compromise
* Administrator action
* Excessive risk
* Role change
* Device loss
* Session-limit enforcement

For server-side sessions:

```text
session.status = revoked
```

or:

```text
delete session record
```

The next lookup fails.

Revocation speed depends on:

* Cache TTL
* Replication delay
* Local session caches
* Read consistency
* Whether every service checks the same store

---

## 7.21 Logout

A complete logout flow commonly:

1. Invalidates the server-side session.
2. Sends an expired replacement cookie.
3. Clears related browser state where appropriate.
4. Records the logout event.
5. Redirects to a non-sensitive page.

Example response:

```http
Set-Cookie: session_id=; Max-Age=0; Path=/; Secure; HttpOnly; SameSite=Lax
```

Deleting only the browser cookie is insufficient if the server-side session remains valid.

An attacker who copied the cookie earlier could continue using it.

Deleting only the server-side session is secure for server enforcement, but stale client cookies may create confusing behavior.

---

## 7.22 Logout from all devices

A simple implementation deletes all sessions for the subject.

```sql
DELETE FROM sessions
WHERE subject_id = $1;
```

A scalable implementation might use a subject-level version.

Example:

```text
User session generation = 18
Session issued with generation = 18
```

On global logout:

```text
User session generation = 19
```

Any session carrying generation `18` is rejected.

This can revoke many sessions without individually updating each one.

---

## 7.23 CSRF

Cross-Site Request Forgery abuses the browser’s automatic credential sending.

Suppose Alice is logged in to:

```text
bank.example
```

A malicious page causes her browser to submit:

```http
POST https://bank.example/transfer
Cookie: session_id=AliceSession
```

The attacker may not know the cookie.

The browser attaches it automatically.

```mermaid
sequenceDiagram
    actor Alice
    participant Evil as evil.example
    participant Browser
    participant Bank as bank.example

    Alice->>Evil: Visit malicious page
    Evil->>Browser: Submit hidden request to bank.example
    Browser->>Bank: Request + Alice's cookie
    Bank->>Bank: Mistakes request for Alice's intent
```

---

## 7.24 CSRF defenses

Common defenses include:

* CSRF tokens
* `SameSite` cookies
* Origin checking
* Referer checking as supplementary evidence
* Reauthentication for sensitive operations
* Avoiding state changes through safe HTTP methods
* Custom request headers for API requests
* User-interaction confirmation

No single mechanism is universally sufficient for every architecture.

---

## 7.25 Synchronizer-token pattern

1. Server creates a random CSRF token bound to the session.
2. Token is embedded in the form.
3. Browser submits the form and token.
4. Server compares submitted token with session state.
5. Request is rejected if missing or incorrect.

Form:

```html
<form method="post" action="/profile/email">
  <input type="hidden" name="csrf_token" value="RANDOM_TOKEN">
  <input name="new_email">
  <button type="submit">Change email</button>
</form>
```

The attacker’s site cannot normally read the legitimate page to obtain the token because of browser same-origin protections.

An XSS vulnerability in the legitimate application may bypass CSRF protections.

---

## 7.26 Double-submit cookie pattern

The server or client creates a CSRF value stored in a cookie and also submits it separately.

Example:

```text
Cookie:
  csrf=RANDOM

Request body or header:
  csrf=RANDOM
```

The server checks that they match.

A robust design should consider:

* Cookie injection
* Subdomain scope
* Signing or binding
* Origin validation
* Token predictability

The exact construction matters; copying the pattern superficially can create bypasses.

---

## 7.27 XSS and sessions

Cross-Site Scripting allows attacker-controlled JavaScript to run in the trusted application origin.

Potential effects:

* Read non-`HttpOnly` tokens
* Make authenticated requests
* Read sensitive page content
* Alter transaction details
* Capture user input
* Trigger password or MFA prompts
* Persist through stored content

`HttpOnly` helps protect the session identifier from direct JavaScript extraction, but XSS may still operate through the victim’s active session.

Major defenses include:

* Correct output encoding
* Safe templating
* Input handling
* Content Security Policy
* Avoiding dangerous DOM APIs
* Dependency management
* Trusted Types where supported
* Isolation of untrusted content

---

## 7.28 Stateful and stateless authentication

### Stateful session

The server stores per-session state.

```text
Client token
→ server lookup
→ session context
```

### Stateless token

The token contains claims needed by the verifier.

```text
Client token
→ cryptographic verification
→ claims
```

“Stateless” usually means:

```text
No central per-request session lookup required
```

It does not mean the overall system has no state.

State still exists in:

* Signing keys
* User accounts
* Authorization data
* Revocation systems
* Token issuance records
* Refresh-token stores
* Audit logs

---

## 7.29 Signed tokens

A signed token contains data and a cryptographic signature.

Conceptually:

```text
claims + metadata
→ sign with issuer key
→ token
```

Verifier:

```text
token
→ verify signature
→ parse claims
→ validate protocol rules
```

The signature provides:

* Integrity
* Evidence of issuance by a trusted key holder

It does not provide confidentiality.

Anyone holding a readable signed token may be able to decode its claims.

---

## 7.30 JWT structure

A JSON Web Token commonly contains three Base64URL-encoded parts:

```text
header.payload.signature
```

Example shape:

```text
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiJ1XzEwNDgiLCJhdWQiOiJyZXBvcnRzLWFwaSJ9
.
SIGNATURE_BYTES
```

### Header

May contain:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key-2026-07"
}
```

### Payload

May contain:

```json
{
  "iss": "https://identity.example.internal",
  "sub": "u_1048",
  "aud": "reports-api",
  "iat": 1785489372,
  "exp": 1785490272,
  "scope": "reports:read"
}
```

### Signature

Protects the encoded header and payload against modification.

---

## 7.31 JWT is a format, not an authentication system

JWT specifies a token representation.

It does not by itself define:

* How the user logged in
* Why the issuer should be trusted
* Where the token is stored
* Which transport sends it
* How authorization is evaluated
* How revocation works
* Whether the token is an ID token or access token
* Which claims are required
* Whether encryption is used

A valid JWT can still be unusable or unsafe if validation rules are wrong.

---

## 7.32 Decoding a JWT locally

The following command decodes the payload for inspection without verifying the signature:

```bash
TOKEN='HEADER.PAYLOAD.SIGNATURE'

python3 - "$TOKEN" <<'PY'
import base64
import json
import sys

parts = sys.argv[1].split(".")
if len(parts) != 3:
    raise SystemExit("Expected a three-part JWT")

payload = parts[1]
payload += "=" * (-len(payload) % 4)

decoded = base64.urlsafe_b64decode(payload)
print(json.dumps(json.loads(decoded), indent=2))
PY
```

### What it does

* Splits a three-part JWT
* Base64URL-decodes its payload
* Parses the JSON
* Prints formatted claims

### Why it is useful

It helps inspect:

* Issuer
* Subject
* Audience
* Expiry
* Scopes
* Roles
* Token identifiers

### Critical limitation

This does **not** verify the signature.

Decoded claims are untrusted until cryptographic and semantic validation succeeds.

### Safety

Tokens may contain sensitive identifiers and may be active bearer credentials.

Avoid pasting production tokens into:

* Public websites
* Shared terminals
* Tickets
* Chat messages
* Screenshots

---

## 7.33 JWT validation step by step

A verifier should generally:

1. Parse token structure safely.
2. Reject unsupported algorithms.
3. Select an expected verification key.
4. Verify the signature.
5. Validate trusted issuer.
6. Validate intended audience.
7. Validate expiry.
8. Validate not-before time if present.
9. Validate token type or purpose.
10. Validate required claims.
11. Validate subject mapping.
12. Apply authorization policy.
13. Check revocation or token status where required.

```mermaid
flowchart TD
    T[Incoming token]
    P[Parse]
    A{Expected algorithm?}
    K[Select trusted key]
    S{Signature valid?}
    I{Issuer valid?}
    U{Audience valid?}
    E{Time valid?}
    Y{Correct token type?}
    M[Map subject]
    Z[Authorization]

    T --> P --> A
    A -->|No| X1[Reject]
    A -->|Yes| K --> S
    S -->|No| X2[Reject]
    S -->|Yes| I
    I -->|No| X3[Reject]
    I -->|Yes| U
    U -->|No| X4[Reject]
    U -->|Yes| E
    E -->|No| X5[Reject]
    E -->|Yes| Y
    Y -->|No| X6[Reject]
    Y -->|Yes| M --> Z
```

---

## 7.34 Signature validation is not enough

Suppose a token is correctly signed:

```json
{
  "iss": "trusted-issuer",
  "sub": "u_1048",
  "aud": "another-service"
}
```

The signature is valid, but the token was not intended for the current service.

If the current service ignores `aud`, it may accept a token issued for another application.

Similarly, a verifier must distinguish token purposes such as:

* Access token
* ID token
* Logout token
* Email-verification token
* Password-reset token

A signed value must be validated for its intended context.

---

## 7.35 Algorithm confusion

A verifier must not blindly trust the token’s declared algorithm.

Unsafe mental model:

```text
Token says alg=X
→ verifier automatically accepts X
```

Safer model:

```text
Application configuration expects algorithm X
AND token declares compatible algorithm X
→ verify using trusted key and implementation
```

The verifier should maintain an explicit allowlist.

The token header is attacker-controlled input until verification succeeds.

---

## 7.36 Key identifiers

A token header may contain:

```json
{
  "kid": "key-2026-07"
}
```

The verifier uses `kid` to select a verification key.

Potential failure modes:

* Treating `kid` as a filesystem path
* Using it directly in SQL
* Fetching arbitrary URLs based on header values
* Accepting unknown keys without restriction
* Cache poisoning
* Incorrect key rotation
* Falling back to an unsafe default key

The key identifier is not itself proof of trust.

It is a selector within an already trusted key-management process.

---

## 7.37 Claims

Common JWT claim names include:

| Claim | Meaning           |
| ----- | ----------------- |
| `iss` | Issuer            |
| `sub` | Subject           |
| `aud` | Intended audience |
| `exp` | Expiry time       |
| `nbf` | Not valid before  |
| `iat` | Issued at         |
| `jti` | Token identifier  |

Protocol-specific tokens may define additional claims.

Custom claims might include:

```json
{
  "tenant_id": "tenant-42",
  "scope": "reports:read",
  "roles": ["financial_analyst"]
}
```

A claim should have:

* Defined meaning
* Trusted issuer
* Expected data type
* Valid range
* Clear update behavior
* Clear authorization relevance

---

## 7.38 Clock skew

Distributed systems rarely have perfectly identical clocks.

A verifier may allow a small time tolerance.

Example:

```text
Accept token up to 60 seconds around boundary conditions
```

Excessive tolerance increases the effective token-validity window.

Clock skew should be:

* Small
* Explicit
* Monitored
* Supported by time synchronization
* Consistent across services

Do not “fix” widespread token expiry failures by adding large validation tolerances without correcting clock synchronization.

---

## 7.39 Access token versus session cookie

These concepts can overlap, but common usage differs.

| Property                    | Browser session cookie       | API access token                          |
| --------------------------- | ---------------------------- | ----------------------------------------- |
| Typical client              | Browser                      | API client or service                     |
| Typical transport           | `Cookie` header              | `Authorization` header                    |
| Browser sends automatically | Yes                          | Usually application-controlled            |
| Common storage              | Cookie store                 | Memory, secret store, application storage |
| CSRF concern                | High when automatically sent | Lower if not automatically attached       |
| XSS theft concern           | Reduced with `HttpOnly`      | High if exposed to JavaScript             |
| Common format               | Opaque ID                    | Opaque or structured                      |
| Revocation                  | Easy with server-side store  | Depends on design                         |

An access token stored in a cookie gains cookie-related behavior, including automatic transmission and CSRF considerations.

A token’s security depends on both format and transport.

---

## 7.40 Token expiry

Short token expiry reduces the time a stolen token remains useful.

Example:

```text
Issued at: 10:00
Expires at: 10:15
```

After expiry, the client needs:

* Reauthentication
* Token refresh
* A new delegated token
* A new service credential

Short lifetimes improve containment but increase issuance traffic and renewal dependency.

---

## 7.41 Refresh tokens

A refresh token obtains new access tokens without repeating full user authentication.

Typical flow:

```mermaid
sequenceDiagram
    participant Client
    participant Auth as Authorization Server
    participant API

    Client->>Auth: Present refresh token
    Auth->>Auth: Validate client, token, grant, and policy
    Auth-->>Client: New access token
    Client->>API: Request with access token
    API-->>Client: Protected response
```

Refresh tokens are often:

* Longer-lived
* More sensitive
* Restricted to the authorization server
* Stored more carefully
* Revocable
* Rotated after use

A refresh token should not normally be sent to resource APIs.

---

## 7.42 Refresh-token rotation

With rotation:

1. Client presents refresh token A.
2. Server invalidates A.
3. Server issues refresh token B.
4. Later reuse of A is suspicious.

```text
A used once
→ A invalid
→ B active
```

If both A and B appear later, the server may infer:

* Token theft
* Client concurrency bug
* Retry race
* Storage rollback

The system needs a deliberate response policy.

Possible response:

```text
Revoke the entire refresh-token family
→ require reauthentication
```

---

## 7.43 Stateless-token revocation problem

A self-contained signed access token may be accepted without a database lookup.

If it is valid for one hour:

```text
Token issued at 10:00
Account disabled at 10:05
Token expires at 11:00
```

Without extra controls, the token may remain usable until 11:00.

Possible approaches:

* Short token lifetime
* Denylist by token ID
* Subject-level revocation timestamp
* Token introspection
* Key rotation
* Audience-specific keys
* Real-time account-state checks
* Gateway validation
* Continuous access evaluation

Each approach reintroduces some state, latency, or complexity.

---

## 7.44 Token introspection

Instead of validating a token entirely locally, a service asks the issuer or authorization server whether it is active.

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Auth as Authorization Server

    Client->>API: Request + access token
    API->>Auth: Introspect token
    Auth-->>API: Active, subject, scope, expiry
    API->>API: Apply authorization
    API-->>Client: Response
```

Advantages:

* Faster revocation response
* Central status control
* Opaque tokens supported
* Claims need not be exposed to clients

Costs:

* Network dependency
* Additional latency
* Central load
* Availability coupling
* Need for caching strategy

---

## 7.45 Token exchange and delegation

A frontend service may receive a user token but should not always forward it unchanged to every backend.

A safer architecture may exchange it for a narrower token.

Example:

```text
User token:
  audience = frontend
  broad user context

Exchanged token:
  audience = reports-api
  scope = reports:read
  actor = frontend-service
  subject = Alice
  lifetime = 5 minutes
```

This limits:

* Audience
* Scope
* Lifetime
* Delegation path

It also makes the acting service visible.

---

## 7.46 Token audience

A token should identify its intended recipient.

Example:

```json
{
  "aud": "reports-api"
}
```

The billing API should reject it.

Without audience restriction:

```text
Token issued for low-risk service
→ replayed to high-risk service
```

Audience validation limits cross-service token reuse.

---

## 7.47 Token scope

Scopes describe delegated permissions.

Example:

```text
reports:read
reports:download
reports:write
```

A token containing:

```text
reports:read
```

should not permit:

```text
reports:delete
```

Scopes are authorization inputs, not necessarily complete authorization decisions.

The service may still need to check:

* Which report
* Which tenant
* Resource ownership
* User status
* Data classification
* Context

---

## 7.48 Token storage in browsers

Possible locations include:

* `HttpOnly` cookie
* JavaScript memory
* `sessionStorage`
* `localStorage`
* IndexedDB
* Browser extension storage

Each has different exposure.

### `HttpOnly` cookie

Pros:

* Not directly readable by normal JavaScript
* Automatic browser handling

Cons:

* Automatically sent
* CSRF considerations
* Cookie-scope complexity

### JavaScript memory

Pros:

* Lost on reload or process end
* Not persisted by default

Cons:

* Accessible to executing application JavaScript
* XSS can access it
* Refresh handling is harder

### `localStorage`

Pros:

* Easy persistence
* Simple API

Cons:

* Accessible to JavaScript
* Long-lived
* XSS can extract tokens
* Shared across tabs for the origin

There is no universally safe storage location in a compromised browser origin.

Preventing XSS and limiting token power remain essential.

---

## 7.49 Signed cookies

Instead of storing session data centrally, an application may place data in a signed cookie.

Example conceptual value:

```text
base64(session_data).signature
```

The server verifies the signature before trusting the data.

Advantages:

* No central session lookup
* Easy horizontal scaling
* Fewer storage operations

Risks and limitations:

* Revocation difficulty
* Cookie-size limits
* Claims may become stale
* Signing-key compromise affects all cookies
* Sensitive data is readable unless separately encrypted
* Key rotation requires careful support
* Client resends all data on every request

A signature prevents modification, not reading.

---

## 7.50 Encrypted tokens

Encrypted tokens hide their contents from clients and intermediaries that lack the decryption key.

They may also be signed or use authenticated encryption.

Encryption can protect confidentiality but does not solve:

* Expiry policy
* Revocation
* Audience validation
* Key rotation
* Replay
* Authorization correctness
* Secure client storage

Encryption should be added for a defined confidentiality requirement, not merely because a token contains JSON.

---

## 7.51 Distributed session storage

A load-balanced application may have instances:

```text
app-1
app-2
app-3
```

If each stores sessions only in local memory:

```text
Login handled by app-1
Next request handled by app-2
→ session not found
```

Possible solutions:

* Shared session store
* Sticky sessions
* Replicated local session state
* Stateless signed tokens

---

### Shared store

```mermaid
flowchart TD
    LB[Load Balancer]
    A1[App 1]
    A2[App 2]
    A3[App 3]
    S[(Shared Session Store)]

    LB --> A1
    LB --> A2
    LB --> A3
    A1 --> S
    A2 --> S
    A3 --> S
```

Advantages:

* Any instance can resolve any session
* Central revocation
* Consistent state

Costs:

* Shared dependency
* Network latency
* Capacity planning
* Store security
* Replication behavior

---

### Sticky sessions

The load balancer routes one client to the same application instance.

Advantages:

* Simpler local session storage

Costs:

* Uneven load
* Failover complexity
* Session loss when instance dies
* Proxy dependence
* Harder autoscaling
* Sticky mapping may be imperfect

Sticky sessions do not replace secure session design.

---

## 7.52 Session-store consistency

Suppose:

```text
Region A revokes session
Region B still has stale replica
```

The user may remain authenticated in Region B.

Consistency decisions include:

* Strongly consistent reads
* Primary-region validation
* Eventual replication
* Local caches
* Revocation event propagation
* Short cache TTL
* Risk-based regional restrictions

Security-critical revocation may require stronger guarantees than ordinary session reads.

---

## 7.53 Session caching

An application might cache:

```text
session ID hash → security context
```

Benefits:

* Lower session-store latency
* Reduced central load

Risks:

* Revocation delay
* Stale roles
* Stale account status
* Cache-key leakage
* Cross-tenant mistakes
* Memory pressure

Cache entries should have:

* Bounded lifetime
* Correct tenant and issuer context
* Secure key handling
* Explicit invalidation where possible

---

## 7.54 Session binding signals

Some systems associate a session with:

* IP address
* User-agent family
* Device cookie
* Client certificate
* Geographic region
* TLS characteristics

Strict IP binding often causes false failures because addresses change through:

* Mobile networks
* VPNs
* Corporate proxies
* IPv6 privacy addressing
* Carrier-grade NAT

These signals are often better used for risk detection than exact equality enforcement.

---

## 7.55 Step-up authentication

A valid session may be insufficient for a sensitive action.

Example:

```text
View dashboard:
  existing session allowed

Change bank account:
  require recent passkey verification
```

Security context may include:

```json
{
  "authenticated_at": "2026-07-31T09:16:12Z",
  "methods": ["password", "totp"],
  "last_step_up_at": "2026-07-31T11:03:14Z",
  "step_up_method": "passkey"
}
```

Policy:

```text
ALLOW change_payment_destination
IF:
  passkey verified
  AND last_step_up_at within 5 minutes
```

---

## 7.56 Impersonation sessions

Support or administrators may need to impersonate a user.

A safe impersonation context should retain both identities:

```text
actor = support-agent-17
effective_subject = user-1048
reason = support-case-7182
```

Do not overwrite the actor identity with the target user.

Audit logs should show:

* Who initiated impersonation
* Which user was impersonated
* Why
* When it began and ended
* Which actions occurred
* Whether sensitive actions were blocked

---

## 7.57 Practical cookie inspection with `curl`

Store cookies from a response:

```bash
curl -sS \
  -D response-headers.txt \
  -c cookies.txt \
  https://app.example.internal/login \
  -o response-body.html
```

### What it does

* Sends an HTTPS request
* Writes response headers to a file
* Saves cookies to a cookie jar
* Saves the response body separately

### Important flags

* `-sS`: quiet progress output but show errors
* `-D FILE`: write response headers
* `-c FILE`: write cookies
* `-o FILE`: write response body

### Why it is useful

It separates:

* Headers
* Cookies
* Body

This makes redirect and cookie debugging easier.

### Safety

`cookies.txt` may contain active credentials.

Use restrictive permissions:

```bash
chmod 600 cookies.txt response-headers.txt
```

Delete the files when no longer needed.

---

## 7.58 Send stored cookies

```bash
curl -sS \
  -b cookies.txt \
  -D - \
  https://app.example.internal/reports \
  -o /dev/null
```

### What it does

Sends applicable cookies from the cookie jar and prints response headers.

### Important flags

* `-b FILE`: read cookies
* `-D -`: print response headers to standard output
* `-o /dev/null`: discard response body

### Output interpretation

```http
HTTP/1.1 200 OK
```

may indicate the cookie was accepted.

```http
HTTP/1.1 302 Found
Location: /login
```

may indicate:

* Cookie not sent
* Cookie rejected
* Session expired
* Session lookup failed
* Account policy rejected the session

---

## 7.59 Inspect cookie-jar contents

```bash
cat cookies.txt
```

An illustrative Netscape-format entry may look like:

```text
app.example.internal	FALSE	/	TRUE	1785523200	session_id	REDACTED
```

Fields commonly represent:

* Domain
* Subdomain matching
* Path
* Secure flag
* Expiry
* Name
* Value

### Safety

Do not display production cookie files in screen shares or shared terminal sessions.

Prefer redacted inspection:

```bash
awk 'BEGIN { OFS="\t" }
     /^#/ { next }
     NF >= 7 {
       $7 = "[REDACTED]"
       print
     }' cookies.txt
```

---

## 7.60 Practical session debugging checklist

### Client side

Check:

* Was `Set-Cookie` returned?
* Did the browser accept it?
* Does domain match?
* Does path match?
* Is `Secure` used over HTTPS?
* Is SameSite behavior blocking the flow?
* Is the cookie expired?
* Is another cookie with the same name interfering?

### Proxy side

Check:

* Is `Set-Cookie` preserved?
* Are headers rewritten?
* Is HTTPS correctly communicated upstream?
* Are multiple cookie headers handled correctly?
* Does cache behavior expose or suppress cookies?

### Application side

Check:

* Was session creation committed?
* Is the session ID generated securely?
* Is lookup hashing consistent?
* Are signing keys consistent across instances?
* Is the session already expired?
* Is account state checked?
* Is session rotation invalidating the wrong record?

### Store side

Check:

* Record exists
* Replication is current
* TTL is correct
* Eviction occurred
* Store capacity is sufficient
* Key namespace is correct
* Tenant context is included

---

## Questions: Sessions, Cookies, and Tokens

1. Why is rotating the session ID after login necessary even when the pre-login session ID was randomly generated?
2. How can `HttpOnly` prevent one form of session theft while still allowing malicious JavaScript to perform authenticated actions?
3. Why can an application with valid cookie settings still be vulnerable to CSRF?
4. How do idle expiry and absolute expiry address different risks?
5. What consistency problem appears when session revocation is replicated asynchronously across regions?
6. Why does a valid JWT signature not prove that the token is intended for the current API?
7. Which JWT fields should be considered attacker-controlled before signature verification?
8. Why does decoding a JWT reveal claims without proving their authenticity?
9. How does refresh-token rotation help detect token reuse?
10. Why are short-lived access tokens not equivalent to immediate revocation?
11. When is an opaque token plus introspection preferable to a self-contained signed token?
12. How can a shared session store become both a performance bottleneck and a security-critical dependency?
13. Why can strict IP-address session binding create failures for legitimate mobile users?
14. How should audit logs represent administrator impersonation without losing the initiating actor?
15. What failure would occur if one application instance signs cookies with key A while another verifies only with key B?
16. Why is a signed cookie readable even when it cannot be modified?
17. How can a browser reject a correctly formed cookie because of its surrounding request context?
18. Which revocation strategy would you choose for a globally distributed service requiring near-immediate administrator lockout?
19. Why should an access token intended for `reports-api` be rejected by `billing-api` even when both trust the same issuer?
20. How can authorization claims inside a long-lived token become stale without any cryptographic failure?

---

# 8. Authorization Models

Authentication establishes a subject.

Authorization determines whether that subject may perform an action on a resource.

The basic decision tuple is:

```text
Subject
Action
Resource
Context
→ Decision
```

Example:

```text
Subject:
  Alice

Action:
  download

Resource:
  report-2026-Q2-finance

Context:
  current time
  device trust
  tenant
  authentication method
  resource classification

Decision:
  allow
```

---

## 8.1 Why authorization exists

Without authorization, every authenticated identity would have equivalent access.

Authentication alone can distinguish:

```text
Alice
Bob
reporting-service
backup-service
```

It does not define what each may do.

Authorization enforces:

* Least privilege
* Separation of duties
* Tenant isolation
* Ownership
* Administrative boundaries
* Data classification
* Business rules
* Compliance controls
* Safety constraints

---

## 8.2 Authorization data flow

```mermaid
flowchart LR
    R[Request]
    I[Authenticated Identity]
    A[Action]
    O[Resource Attributes]
    C[Context]
    P[Policy Engine]
    D{Decision}
    E[Enforcement]

    R --> A
    R --> O
    I --> P
    A --> P
    O --> P
    C --> P
    P --> D
    D --> E
```

The decision is only as correct as its inputs.

A policy can be logically perfect but still fail if:

* Identity mapping is wrong
* Resource attributes are stale
* Tenant is derived from user input
* Action is misclassified
* Device state is forged
* Enforcement is bypassed

---

## 8.3 Authentication middleware versus authorization logic

Authentication middleware might produce:

```python
request.identity = {
    "subject_id": "u_1048",
    "roles": ["financial_analyst"]
}
```

Authorization then asks:

```python
can_download(
    subject=request.identity,
    report=report,
)
```

A dangerous pattern:

```python
if request.identity:
    return download_report(report_id)
```

This checks only whether the user is authenticated.

It does not verify access to the selected report.

---

## 8.4 Coarse-grained and fine-grained authorization

### Coarse-grained

Controls access to broad areas.

Examples:

* May access `/admin`
* May call reports API
* May enter production environment

### Fine-grained

Controls specific actions and objects.

Examples:

* May edit report 172
* May view invoices for tenant 42
* May approve payments below ₹500,000
* May restart only services owned by the platform team

Both are useful.

A gateway may perform coarse checks while the application performs object-level checks.

---

## 8.5 Default deny

A default-deny system rejects operations unless a policy explicitly permits them.

```text
No matching allow rule
→ deny
```

This is safer than:

```text
No matching deny rule
→ allow
```

Default deny helps contain:

* New endpoints
* New actions
* Missing roles
* Unexpected resources
* Policy gaps

It does not guarantee correct authorization. An overly broad allow rule can still expose data.

---

## 8.6 Explicit deny

Some policy systems support explicit deny rules.

Example:

```text
ALLOW finance_manager to download finance reports

DENY any subject from downloading legal_hold reports
```

When allow and deny rules conflict, the system needs a defined combination strategy.

Common strategies:

* Deny overrides
* Allow overrides
* First applicable
* Most specific rule
* Ordered evaluation
* Consensus or threshold

Unclear rule precedence creates dangerous surprises.

---

## 8.7 Discretionary Access Control

**Discretionary Access Control**, or DAC, allows an owner or authorized controller to assign access.

Classic Unix file permissions are partly discretionary.

Example:

```text
Alice owns file
→ Alice can grant group read access
```

Typical characteristics:

* Resource ownership
* Owner-controlled permissions
* User and group identity
* Access-control lists

Advantages:

* Flexible
* Familiar
* Decentralized management

Risks:

* Owners may overshare
* Permissions accumulate
* Group relationships become complex
* Data may move to less protected resources

---

## 8.8 Mandatory Access Control

**Mandatory Access Control**, or MAC, applies centrally defined labels and rules that ordinary users cannot freely override.

Examples include systems based on:

* Security classifications
* Integrity levels
* SELinux labels
* Mandatory information-flow rules

Conceptual policy:

```text
Subject clearance = Secret
Resource classification = Top Secret
→ deny read
```

or:

```text
Process type = web_server_t
File type = shadow_t
→ deny open
```

MAC complements ordinary ownership and mode permissions.

---

## 8.9 Role-Based Access Control

**Role-Based Access Control**, or RBAC, assigns permissions to roles and roles to subjects.

```mermaid
flowchart LR
    U[Users]
    R[Roles]
    P[Permissions]
    O[Resources]

    U --> R
    R --> P
    P --> O
```

Example:

```text
Alice
→ financial_analyst role
→ reports:read permission
```

---

## 8.10 Why RBAC exists

Assigning permissions directly to every user scales poorly.

Without roles:

```text
Alice → permission A
Alice → permission B
Bob → permission A
Bob → permission B
Carol → permission A
Carol → permission B
```

With roles:

```text
financial_analyst → permission A, permission B

Alice → financial_analyst
Bob → financial_analyst
Carol → financial_analyst
```

RBAC groups permissions around job functions.

---

## 8.11 RBAC components

| Component       | Meaning                                   |
| --------------- | ----------------------------------------- |
| User or subject | Entity receiving roles                    |
| Role            | Named job function or permission grouping |
| Permission      | Allowed operation                         |
| Resource        | Target object or class                    |
| Role assignment | Subject-to-role mapping                   |
| Role hierarchy  | Inheritance between roles                 |
| Constraint      | Rule restricting combinations             |

Example:

```text
Role:
  financial_analyst

Permissions:
  report:list
  report:read
  report:download
```

---

## 8.12 Role hierarchy

A role may inherit another role.

```text
finance_manager
→ inherits financial_analyst
→ adds report:approve
```

```mermaid
flowchart TD
    FM[Finance Manager]
    FA[Financial Analyst]
    READ[Read Reports]
    DOWNLOAD[Download Reports]
    APPROVE[Approve Reports]

    FM --> FA
    FA --> READ
    FA --> DOWNLOAD
    FM --> APPROVE
```

Role hierarchies simplify assignment but can make effective permissions hard to understand.

Deep inheritance increases debugging complexity.

---

## 8.13 Role explosion

RBAC struggles when access depends on many dimensions.

Example requirements:

```text
Finance analyst for tenant A
Finance analyst for tenant B
Finance manager for tenant A
Finance manager for tenant B
Finance analyst with export permission
Finance analyst without export permission
```

This may produce many roles:

```text
tenant-a-finance-analyst-export
tenant-a-finance-analyst-no-export
tenant-b-finance-manager-export
...
```

This is role explosion.

It often indicates that some conditions should be represented as attributes or resource relationships instead.

---

## 8.14 Separation of duties

Some permissions should not be held or exercised by the same subject.

Example:

```text
Person who creates payment
must not
approve the same payment
```

This can be enforced statically:

```text
User cannot hold both requester and approver roles
```

or dynamically:

```text
User may hold both roles
but cannot approve a payment they created
```

Dynamic separation requires transaction context and resource history.

---

## 8.15 Attribute-Based Access Control

**Attribute-Based Access Control**, or ABAC, evaluates attributes of:

* Subject
* Resource
* Action
* Environment

Example policy:

```text
ALLOW download
IF:
  subject.department == resource.department
  AND subject.employment_status == "active"
  AND resource.classification <= subject.clearance
  AND current_time within business_hours
```

---

## 8.16 ABAC data model

### Subject attributes

```json
{
  "subject_id": "u_1048",
  "department": "finance",
  "clearance": "confidential",
  "employment_status": "active"
}
```

### Resource attributes

```json
{
  "resource_id": "report-2026-Q2",
  "department": "finance",
  "classification": "confidential",
  "tenant_id": "tenant-42"
}
```

### Environment attributes

```json
{
  "current_time": "2026-07-31T11:14:00Z",
  "network_zone": "corporate",
  "device_managed": true,
  "risk_level": "low"
}
```

---

## 8.17 Why ABAC exists

ABAC handles decisions that are awkward to encode as roles.

Examples:

* Same department
* Same tenant
* Resource classification
* Device compliance
* Time-based restrictions
* Geographic restrictions
* Transaction amount
* Employment state

It offers flexibility but increases dependency on accurate attribute data.

---

## 8.18 ABAC failure modes

### Stale attributes

```text
Alice transferred out of finance
but
cached department attribute remains finance
```

### Untrusted attributes

```text
department taken from request body
instead of
identity directory
```

### Type confusion

```text
clearance = "10"
classification = 2
```

String and numeric comparison may produce unexpected results.

### Missing attributes

What does policy do when:

```text
device_managed = unknown
```

Possible choices:

* Deny
* Treat as false
* Use fallback
* Request more information

Missing-data behavior must be explicit.

---

## 8.19 Relationship-Based Access Control

**Relationship-Based Access Control**, or ReBAC, bases decisions on relationships between entities.

Examples:

```text
Alice is owner of Project 17
Bob is member of Team A
Team A has editor access to Project 17
Carol manages Bob
```

A decision may traverse relationships:

```text
Alice
→ member of Team A
→ Team A editor of Folder X
→ Folder X contains Document Y
→ Alice may edit Document Y
```

```mermaid
flowchart LR
    A[Alice]
    T[Team A]
    F[Folder X]
    D[Document Y]

    A -->|member of| T
    T -->|editor of| F
    F -->|contains| D
```

---

## 8.20 Why ReBAC exists

Many application permissions are naturally relational.

Examples:

* Document sharing
* Social-network visibility
* Organization hierarchy
* Project membership
* Parent-child resource inheritance
* Repository collaboration
* Group-based sharing

Encoding every relationship as a role can become unmanageable.

---

## 8.21 ReBAC challenges

* Graph traversal cost
* Cycles
* Inheritance ambiguity
* Relationship freshness
* Bulk listing
* Revocation propagation
* Tenant isolation
* Debugging effective access
* Very large group membership
* Nested groups

A point lookup such as:

```text
Can Alice read document Y?
```

may be fast.

A list query such as:

```text
Return every document Alice can read
```

can be substantially harder.

---

## 8.22 Access-Control Lists

An **Access-Control List**, or ACL, is attached to a resource and lists who may do what.

Example:

```json
{
  "resource": "report-172",
  "entries": [
    {
      "principal": "u_1048",
      "permissions": ["read", "download"]
    },
    {
      "principal": "group:finance-managers",
      "permissions": ["read", "download", "approve"]
    }
  ]
}
```

Mental model:

```text
Resource
→ list of principals and permissions
```

---

## 8.23 ACL advantages and limitations

### Advantages

* Easy to answer “who can access this resource?”
* Natural for sharing
* Supports user- and group-specific exceptions
* Familiar from filesystems and cloud storage

### Limitations

* Difficult to answer “what can this user access?” at scale
* ACLs may become large
* Inheritance can become confusing
* Orphaned entries may remain after identity deletion
* Bulk revocation may require many updates

---

## 8.24 Capability-based authorization

A capability is an unforgeable reference or token that grants authority to perform an operation.

Mental model:

```text
Possession of capability
→ authority to access specific resource
```

Example:

```text
Capability grants:
  read object 71
  until 12:00
```

A signed download URL is a capability-like mechanism.

Example conceptual URL:

```text
https://storage.example/object/71
?expires=...
&signature=...
```

Anyone possessing the URL during its validity may be able to use it.

---

## 8.25 Capability properties

Capabilities can be restricted by:

* Resource
* Action
* Expiry
* Audience
* Network
* Number of uses
* Cryptographic key
* Delegation depth

Benefits:

* Fine-grained delegation
* No broad identity lookup required
* Useful for temporary sharing
* Natural least-privilege token

Risks:

* Leakage grants access
* Revocation can be difficult
* URLs may appear in logs or referrers
* Forwarding may be hard to control
* Possession may not identify the user

---

## 8.26 ACL versus capability

| Question                        | ACL                    | Capability                    |
| ------------------------------- | ---------------------- | ----------------------------- |
| Where is authority represented? | On or near resource    | In possessed reference/token  |
| Typical check                   | Is subject listed?     | Is capability valid?          |
| Easy revocation                 | Usually central update | Depends on design             |
| Easy delegation                 | Requires ACL update    | Can pass capability           |
| Leakage risk                    | Identity still checked | Possession may be enough      |
| Audit identity                  | Usually explicit       | May be anonymous or delegated |

---

## 8.27 Policy-based authorization

A policy engine evaluates declarative rules.

Example policy:

```text
allow if:
  input.subject.tenant_id == input.resource.tenant_id
  and "report:read" in input.subject.permissions
  and input.resource.status != "deleted"
```

Benefits:

* Centralized logic
* Reusable policies
* Testing
* Version control
* Auditability
* Separation from business code

Risks:

* Network dependency
* Policy-language complexity
* Input-contract drift
* Difficult data fetching
* Performance overhead
* Partial enforcement if some paths bypass the engine

---

## 8.28 Policy Decision Point and Policy Enforcement Point

### Policy Decision Point

Returns a decision.

```json
{
  "decision": "allow",
  "policy_version": "reports-v17",
  "reason": "same_department_and_scope"
}
```

### Policy Enforcement Point

Blocks or permits the real operation.

```text
Policy says deny
→ application does not query or return protected data
```

A correct decision is useless if enforcement is missing.

---

## 8.29 Enforcement locations

Authorization may be enforced at:

* Browser UI
* API gateway
* Reverse proxy
* Application route
* Service method
* Database query
* Database row-level security
* Filesystem
* Kernel
* Storage service
* Message broker
* Cloud API

Client-side UI restrictions are not sufficient.

Hiding a button does not prevent a crafted API request.

---

## 8.30 Defense in depth

Suppose Alice may access only tenant 42.

Controls may exist at several layers:

```text
Gateway:
  token tenant must equal URL tenant

Application:
  policy verifies subject tenant

Database query:
  WHERE tenant_id = 42

Database RLS:
  connection context restricted to tenant 42
```

A failure in one layer may still be contained by another.

Defense in depth should not produce contradictory policies that are impossible to debug.

---

## 8.31 Object-level authorization

An endpoint may be protected while individual objects remain exposed.

Example vulnerable route:

```http
GET /api/invoices/172
```

Application:

```python
require_authenticated_user()
return load_invoice(172)
```

An attacker changes:

```text
172 → 173
```

If invoice 173 belongs to another user and no object check occurs, access succeeds.

This class of failure is often called an insecure direct object reference or broken object-level authorization.

---

## 8.32 Safe object lookup

Instead of:

```sql
SELECT *
FROM invoices
WHERE invoice_id = $1;
```

use an authorized lookup:

```sql
SELECT *
FROM invoices
WHERE invoice_id = $1
  AND tenant_id = $2;
```

or:

```sql
SELECT i.*
FROM invoices AS i
JOIN account_memberships AS m
  ON m.account_id = i.account_id
WHERE i.invoice_id = $1
  AND m.user_id = $2
  AND m.can_read = TRUE;
```

The correct structure depends on the data model.

The important principle is:

> The resource lookup and authorization constraint should be difficult to separate accidentally.

---

## 8.33 Tenant isolation

Multi-tenant systems serve multiple organizations or customers.

Every relevant object may need a tenant identifier:

```text
tenant_id = tenant-42
```

Authorization must ensure:

```text
subject tenant
matches
resource tenant
```

Common failure:

```text
Tenant ID accepted from request header
without binding it to authenticated identity
```

An attacker changes:

```http
X-Tenant-ID: tenant-42
```

to:

```http
X-Tenant-ID: tenant-43
```

Tenant selection must be derived from trusted membership and routing rules.

---

## 8.34 Tenant context propagation

A request may move through:

```text
Gateway
→ application
→ queue
→ worker
→ database
```

Tenant context must remain bound to the operation.

Example queued message:

```json
{
  "job_id": "job-718",
  "tenant_id": "tenant-42",
  "requested_by": "u_1048",
  "operation": "generate_report"
}
```

The worker must not trust an arbitrary tenant identifier merely because it appears in a queue message.

It should trust:

* The queue producer identity
* Message integrity
* Schema validation
* Authorized relationship between user and tenant
* Restricted queue permissions

---

## 8.35 Authorization in asynchronous systems

A user starts a long-running job.

At submission:

```text
Alice is authorized
```

At execution ten minutes later:

```text
Alice's access has been revoked
```

Possible models:

### Check at submission only

The accepted job retains authority to complete.

Useful when the job represents a committed action.

### Check at execution

Worker reevaluates current permissions.

Useful when access must reflect current state.

### Delegated capability

Submission creates a narrowly scoped capability:

```text
Generate report 71 once
before 12:00
```

The design must define when authorization is evaluated.

---

## 8.36 Time-of-check to time-of-use

A **TOCTOU** problem occurs when authorization is checked and the relevant state changes before use.

Example:

```text
1. Check Alice owns file.
2. File ownership changes.
3. Delete file.
```

Or:

```text
1. Check account balance.
2. Concurrent transaction changes balance.
3. Approve withdrawal using stale state.
```

Defenses include:

* Database transactions
* Row locks
* Atomic conditional updates
* Version checks
* Performing check and use in one trusted component
* Capability generation tied to immutable resource state

---

## 8.37 Atomic authorization with SQL

Suppose Alice may update a document only if she is the owner.

Instead of:

```text
SELECT owner
→ compare in application
→ UPDATE document
```

use:

```sql
UPDATE documents
SET title = $1
WHERE document_id = $2
  AND owner_user_id = $3;
```

Then inspect affected-row count.

```text
1 row changed → authorized resource updated
0 rows changed → missing resource or unauthorized
```

This reduces the gap between check and use.

It may intentionally avoid revealing whether the object exists.

---

## 8.38 Database Row-Level Security

Some databases support row-level security policies.

Conceptual PostgreSQL example:

```sql
ALTER TABLE reports ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation
ON reports
USING (
    tenant_id = current_setting('app.tenant_id')::uuid
);
```

### What it does

Restricts visible rows according to a session or transaction setting.

### Why it is useful

It adds a database-level tenant boundary.

### Critical requirements

The application must securely set `app.tenant_id`.

It must prevent untrusted users from changing that setting.

Connection pooling must reset context correctly between requests.

### Safety

Incorrect RLS policy or connection-context leakage can expose or hide large amounts of data.

Test with separate database roles and realistic pooling behavior.

---

## 8.39 Connection-pool identity leakage

Suppose one database connection processes:

```text
Request A:
  tenant_id = tenant-42
```

The connection returns to the pool without resetting state.

Next:

```text
Request B:
  tenant_id should be tenant-43
```

If the old value persists, Request B may operate under tenant 42.

Mitigations include:

* Transaction-local settings
* Mandatory reset hooks
* Separate pools
* Database roles
* Automated cross-tenant tests
* Failing closed when context is absent

---

## 8.40 Authorization caching

A service may cache:

```text
Can Alice read report 71? → allow
```

Benefits:

* Lower policy latency
* Lower directory load
* Better availability

Risks:

* Stale role assignment
* Delayed revocation
* Incorrect cache key
* Missing tenant dimension
* Missing action dimension
* Policy-version mismatch

A safe cache key may need:

```text
subject ID
tenant ID
action
resource ID or resource version
policy version
relevant attribute version
```

Incorrectly caching only by subject can authorize unrelated actions.

---

## 8.41 Filtering versus post-filtering

### Post-filtering

```text
Load 10,000 records
→ authorization-check each one
→ return 17
```

Problems:

* High data exposure inside application
* Poor performance
* Pagination errors
* Count leakage
* Accidental unfiltered response
* Expensive policy calls

### Authorization-aware query

```text
Build query constrained to authorized records
→ database returns 17
```

Usually better for listing and search.

The difficulty is translating policy into query constraints without changing semantics.

---

## 8.42 Pagination failure

Suppose the database returns 20 records, then the application removes unauthorized rows.

Page 1 may contain only 3 visible rows.

The user may infer hidden records from:

* Missing rows
* Page counts
* Total counts
* Gaps in identifiers
* Timing

Correct pagination should operate over the authorized dataset where practical.

---

## 8.43 Field-level authorization

A user may access a resource but not every field.

Example employee record:

```json
{
  "name": "Alice",
  "department": "Finance",
  "salary": 1800000,
  "medical_notes": "..."
}
```

Possible policy:

```text
Manager:
  name, department

HR:
  name, department, salary

Occupational health:
  medical_notes
```

Returning the entire object and hiding fields in the UI is insufficient.

The server must filter or separately authorize sensitive fields.

---

## 8.44 Action semantics

Authorization depends on correctly identifying the action.

Actions may include:

* Create
* Read
* List
* Update
* Delete
* Approve
* Export
* Share
* Impersonate
* Execute
* Delegate
* Restore
* View metadata
* View secret value

`read` and `list` are not always equivalent.

A user may know a resource exists without being allowed to read its contents.

Similarly:

```text
May update profile
```

does not necessarily imply:

```text
May change role
May change tenant
May disable MFA
```

Sensitive sub-actions should be modeled explicitly.

---

## 8.45 Authorization and business logic

Some authorization rules are inseparable from business state.

Example:

```text
Payment may be approved only if:
  subject is an approver
  amount is within approval limit
  subject did not create payment
  payment is pending
  tenant matches
```

This is more than a simple role check.

Authorization and business invariants must be coordinated.

A policy engine may decide identity-related conditions, while the application transaction enforces current business state.

---

## 8.46 Denial responses

Possible responses:

### `401 Unauthorized`

Authentication missing or invalid.

### `403 Forbidden`

Authenticated identity lacks permission.

### `404 Not Found`

Resource absence or intentionally hidden existence.

### `409 Conflict`

Operation conflicts with current resource state.

### `429 Too Many Requests`

Rate or quota policy denies operation temporarily.

Status choice should reflect:

* Protocol semantics
* Information-disclosure policy
* Client behavior
* Consistency across APIs

---

## 8.47 Authorization error detail

External response:

```json
{
  "error": "forbidden"
}
```

Internal decision log:

```json
{
  "subject_id": "u_1048",
  "action": "download",
  "resource_id": "report-71",
  "decision": "deny",
  "reason": "resource_department_mismatch",
  "policy_version": "reports-v17"
}
```

Detailed internal reasons help debugging.

Detailed external reasons may leak:

* Resource existence
* Group names
* Policy structure
* Sensitive classifications
* Account status

---

## 8.48 Explainability

Operators need to answer:

```text
Why was this allowed?
Why was this denied?
```

A decision system should be able to identify:

* Subject
* Resource
* Action
* Relevant attributes
* Applied policy
* Policy version
* Decision
* Reason
* Missing data
* Source of attributes

Avoid logging sensitive values unnecessarily.

---

## 8.49 Authorization audit log

Illustrative event:

```json
{
  "timestamp": "2026-07-31T12:18:44Z",
  "request_id": "req_8ae1",
  "subject_id": "u_1048",
  "actor_id": "u_1048",
  "tenant_id": "tenant-42",
  "action": "report.download",
  "resource_type": "financial_report",
  "resource_id": "report-2026-Q2",
  "decision": "allow",
  "policy_id": "report-access",
  "policy_version": "17",
  "authentication_methods": ["password", "totp"],
  "reason": "same_tenant_same_department"
}
```

For impersonation:

```text
actor_id != subject_id
```

---

## 8.50 Testing authorization

Authorization tests should include more than allowed cases.

A useful matrix:

| Subject           | Resource                | Action  | Expected                |
| ----------------- | ----------------------- | ------- | ----------------------- |
| Finance analyst   | Own department report   | Read    | Allow                   |
| Finance analyst   | Other department report | Read    | Deny                    |
| Finance analyst   | Own department report   | Delete  | Deny                    |
| Finance manager   | Own department report   | Approve | Allow                   |
| Suspended manager | Own department report   | Approve | Deny                    |
| Tenant 42 user    | Tenant 43 report        | Read    | Deny                    |
| Unauthenticated   | Any report              | Read    | Authentication required |

Test edge cases:

* Missing role
* Duplicate role
* Stale cache
* Deleted resource
* Null tenant
* Unknown action
* Malformed identifier
* Concurrent ownership change
* Revocation during request
* Nested group cycle

---

## 8.51 Negative authorization testing

A strong test strategy asks:

```text
How could this request become unauthorized?
```

Examples:

* Change resource ID
* Change tenant ID
* Remove role
* Use expired session
* Reuse another user’s URL
* Call hidden endpoint directly
* Modify JSON fields not shown in UI
* Change HTTP method
* Add administrator-only field
* Bypass gateway
* Replay queued job
* Use stale token after role removal

Authorization testing should deliberately cross trust boundaries.

---

## 8.52 Practical HTTP authorization testing

Assume two test accounts with separate cookie jars:

```bash
curl -sS \
  -b alice-cookies.txt \
  -D - \
  https://app.example.internal/reports/report-71 \
  -o alice-response.json
```

Then test the same resource as Bob:

```bash
curl -sS \
  -b bob-cookies.txt \
  -D - \
  https://app.example.internal/reports/report-71 \
  -o bob-response.json
```

### What it does

Sends the same object request under different authenticated sessions.

### Why it is useful

It tests object-level authorization rather than only route authentication.

### What to inspect

* Status code
* Response body
* Response size
* Timing
* Headers
* Audit log
* Whether the resource exists for both users
* Whether the server leaks metadata

### Safety

Use dedicated test accounts and non-production data.

Do not attempt cross-user authorization tests on systems without explicit permission.

---

## 8.53 Testing HTTP methods

An application may protect:

```http
GET /reports/71
```

but forget:

```http
DELETE /reports/71
```

Test each supported method deliberately:

```bash
curl -i \
  -X DELETE \
  -b alice-cookies.txt \
  https://app.example.internal/reports/report-71
```

### Important flag

* `-X DELETE`: explicitly chooses the HTTP method

### Safety

This can alter or destroy data.

Use a disposable test environment and test resource.

For read-only observation, use application logs, mocks, or a staging system.

---

## 8.54 Inspecting Unix authorization

Display identity and group membership:

```bash
id
```

Example:

```text
uid=1001(alice) gid=1001(alice) groups=1001(alice),27(sudo),2001(finance)
```

### What it does

Shows the current process user and group identity.

### Why it is useful

Linux file authorization often depends on:

* Effective UID
* Primary GID
* Supplementary groups

### Output meaning

* `uid`: current user ID
* `gid`: primary group ID
* `groups`: supplementary groups

Group changes may not affect already-running login sessions until group membership is refreshed.

---

## 8.55 Inspecting file permissions

```bash
ls -l /srv/reports/finance/2026-q2.pdf
```

Illustrative output:

```text
-rw-r----- 1 finance-app finance 482137 Jul 31 10:20 2026-q2.pdf
```

Interpretation:

```text
-          regular file
rw-        owner may read and write
r--        group may read
---        others have no access
finance-app owner
finance     group
```

This does not show all possible controls.

Access may also depend on:

* ACLs
* SELinux
* AppArmor
* Mount options
* Capabilities
* Container mappings
* Network filesystem policy

---

## 8.56 Inspecting file ACLs

```bash
getfacl /srv/reports/finance/2026-q2.pdf
```

### What it does

Displays extended POSIX ACL entries where supported.

Illustrative output:

```text
# file: srv/reports/finance/2026-q2.pdf
# owner: finance-app
# group: finance
user::rw-
user:auditor:r--
group::r--
mask::r--
other::---
```

### Why it is useful

It reveals permissions not visible in basic mode bits.

### Important concept: mask

The ACL mask can limit effective permissions of named users and groups.

An entry may appear to grant access but be reduced by the mask.

### Safety

Reading ACLs is generally non-destructive.

Changing them can expose or block sensitive files.

---

## 8.57 Testing file access as another user

A controlled test may use:

```bash
sudo -u alice -- test -r /srv/reports/finance/2026-q2.pdf
echo $?
```

### What it does

Runs `test -r` under user `alice`.

### Output meaning

* Exit status `0`: test reports readable
* Nonzero: test reports not readable

### Important parts

* `sudo -u alice`: execute as Alice
* `--`: end option parsing
* `test -r`: test read access
* `echo $?`: print previous command’s exit status

### Limitations

Effective access can still differ for the real application because of:

* Different groups
* SELinux domain
* Container namespace
* Capabilities
* Network filesystem mapping
* Application sandboxing

### Safety

Use only with administrative authorization.

Do not substitute arbitrary usernames or paths on systems you do not control.

---

## 8.58 Linux permission walkthrough

Suppose:

```text
Process:
  UID = finance-app
  groups = finance

File:
  owner = root
  group = finance
  mode = 0640
```

Kernel evaluation, simplified:

1. Is process UID the owner?

   * No.
2. Is process in file group?

   * Yes.
3. Apply group bits:

   * `r--`
4. Requested operation:

   * Read.
5. Decision:

   * Allow.

If process requests write:

```text
group bits lack w
→ deny with EACCES
```

Exact access may still be modified by ACLs or mandatory controls.

---

## 8.59 Authorization failure troubleshooting

### Step 1: Confirm authentication

Determine:

* Was a credential presented?
* Was it valid?
* Which subject was resolved?
* Which authentication method was used?

### Step 2: Identify exact action

Do not use vague labels such as “access.”

Determine:

```text
list
read
download
edit
approve
delete
share
```

### Step 3: Identify exact resource

Capture:

* Resource type
* Resource ID
* Tenant
* Owner
* Classification
* Current state

### Step 4: Inspect policy inputs

Check:

* Roles
* Groups
* Attributes
* Relationships
* Device state
* Time
* Token scope
* Authentication strength

### Step 5: Inspect decision

Check:

* Allow or deny
* Applied policy
* Policy version
* Missing attributes
* Cache result
* Reason

### Step 6: Inspect enforcement

Confirm the denied path did not continue.

For an allowed decision, verify downstream systems also permitted the operation.

---

## 8.60 Common authorization failures

### Missing object check

Route requires login but any object ID is accepted.

### Client-controlled role

```json
{
  "role": "admin"
}
```

is trusted from request input.

### Tenant identifier trust

Client chooses another tenant without membership validation.

### Hidden UI mistaken for enforcement

Button is hidden, but endpoint remains callable.

### Stale token claims

Removed role remains in long-lived token.

### Cache-key omission

Decision cached by subject but not resource or action.

### Inconsistent service checks

Gateway enforces policy, but backend is directly reachable.

### Batch endpoint bypass

Single-object endpoint checks each object, batch endpoint does not.

### Export path bypass

UI view filters sensitive fields, CSV export returns all fields.

### Background-worker bypass

API checks user permissions, queued worker trusts arbitrary job data.

---

## 8.61 Authorization performance

Authorization latency may include:

```text
Identity parsing
+ group lookup
+ resource lookup
+ relationship traversal
+ policy evaluation
+ cache access
+ audit write
```

Performance strategies include:

* Local policy evaluation
* Cached attributes
* Precomputed group expansion
* Indexed relationship tuples
* Authorization-aware database queries
* Batch policy checks
* Short-lived decision caches
* Asynchronous audit transport

Each optimization affects freshness and revocation.

---

## 8.62 Batch authorization

Listing 100 resources should not always make 100 network policy calls.

Possible batch request:

```json
{
  "subject": "u_1048",
  "action": "report.read",
  "resources": [
    "report-1",
    "report-2",
    "report-3"
  ]
}
```

Response:

```json
{
  "report-1": "allow",
  "report-2": "deny",
  "report-3": "allow"
}
```

Batching reduces overhead but must preserve:

* Individual resource attributes
* Tenant boundaries
* Decision reasons
* Correct partial failure handling

---

## 8.63 Policy versioning

Policies change over time.

A decision log should record:

```text
policy_id = report-access
policy_version = 17
```

Without versioning, an operator may be unable to reproduce why a request was allowed yesterday but denied today.

Policy deployment should support:

* Review
* Tests
* Staging
* Rollback
* Change history
* Impact analysis
* Auditability

---

## 8.64 Policy shadow evaluation

A new policy may run in observation mode.

```text
Current policy decision:
  allow

Candidate policy decision:
  deny
```

The system enforces the current policy but records differences.

This helps detect:

* Unexpected access loss
* New privilege expansion
* Missing attributes
* Performance regressions
* Tenant-specific anomalies

Shadow evaluation must not expose sensitive policy data to clients.

---

## 8.65 Authorization model comparison

| Model        | Main organizing idea               | Strong fit                       | Common weakness                  |
| ------------ | ---------------------------------- | -------------------------------- | -------------------------------- |
| DAC          | Owner controls access              | Files and user-managed sharing   | Oversharing and permission drift |
| MAC          | Central labels and mandatory rules | High-assurance isolation         | Complexity and rigidity          |
| RBAC         | Roles contain permissions          | Job-function access              | Role explosion                   |
| ABAC         | Attributes drive rules             | Context-rich decisions           | Attribute trust and complexity   |
| ReBAC        | Relationships drive access         | Sharing and graph-based products | Traversal and listing cost       |
| ACL          | Resource lists allowed principals  | Per-object sharing               | Hard user-centric queries        |
| Capabilities | Possession grants authority        | Delegation and temporary access  | Leakage and revocation           |

Real systems commonly combine several models.

Example:

```text
RBAC:
  user is financial_analyst

ABAC:
  user's department matches report department

ReBAC:
  user is member of project owning the report

Capability:
  temporary signed download link

MAC:
  process label may access storage label
```

---

## 8.66 Choosing an authorization model

Ask:

1. Are permissions primarily based on job function?
2. Are resources owned or shared individually?
3. Are tenant and resource attributes central?
4. Are relationships deeply nested?
5. Is temporary delegation important?
6. Must access be centrally mandatory?
7. How quickly must revocation propagate?
8. Must users understand and manage sharing?
9. Are list queries as important as point checks?
10. Where will enforcement occur?
11. What is the expected policy scale?
12. How will decisions be explained and audited?

Choose the simplest model that accurately represents the real access relationships.

Do not force complex business relationships into a single role name.

---

## Questions: Authorization Models

1. Why does protecting an API route not guarantee object-level authorization?
2. How can an authorization-aware SQL query reduce both TOCTOU risk and accidental data exposure?
3. What problem does RBAC solve, and what conditions commonly cause role explosion?
4. Why is ABAC only as trustworthy as the sources of its attributes?
5. How does ReBAC naturally represent shared folders better than flat roles?
6. In what sense is a signed download URL a capability?
7. Why can a valid capability provide access without proving the current user’s identity?
8. What must a database connection pool do when tenant context is stored in connection-local state?
9. How can authorization post-filtering break pagination semantics?
10. Why should `report:list` and `report:read` sometimes be separate permissions?
11. How would you enforce “a payment creator cannot approve the same payment”?
12. What decision should a policy engine return when a required attribute is missing?
13. Why can caching authorization only by subject ID expose unrelated resources?
14. How should a system authorize a queued operation when the user’s permissions may change before execution?
15. Which layers could enforce tenant isolation, and what failure does each layer contain?
16. How can explicit deny interact unexpectedly with role inheritance?
17. Why must the actor and effective subject remain distinct during support impersonation?
18. What information is needed to reproduce an authorization decision after a policy update?
19. Why might a gateway-level authorization check be bypassed even though every normal client uses the gateway?
20. How would you test for authorization failures in batch, export, background-job, and alternate HTTP-method paths?
21. What is the difference between a resource ACL and a role assignment?
22. Why can nested group membership make both authorization latency and revocation difficult?
23. When is a mandatory access-control layer useful even if the application already has RBAC?
24. How can a policy be logically correct while enforcement remains insecure?
25. Why should tenant identity never be trusted solely because it appears in a client-supplied header?

# 9. OAuth 2.0, OpenID Connect, and SAML

These protocols are frequently grouped together because they support centralized identity and cross-application access.

They solve different problems:

| Technology     | Primary purpose                                                                      |
| -------------- | ------------------------------------------------------------------------------------ |
| OAuth 2.0      | Delegated authorization to protected resources                                       |
| OpenID Connect | Authentication and identity information built on OAuth 2.0                           |
| SAML 2.0       | Federated authentication and assertion exchange, commonly for enterprise browser SSO |

The most important distinction is:

```text
OAuth 2.0:
  May this client access a resource?

OpenID Connect:
  Who authenticated, and what was the authentication result?

SAML:
  What identity assertion did a trusted identity provider issue to a service provider?
```

OAuth 2.0 was defined as a framework that allows a client to obtain limited access to an HTTP service, either on behalf of a resource owner or on its own behalf. OpenID Connect adds an authentication layer and communicates the result in an ID Token.

---

## 9.1 The problem: credential sharing does not scale

Suppose a reporting application needs access to Alice’s files in a storage service.

A naïve design asks Alice for her storage-service password:

```text
Alice
→ gives storage password to reporting application
→ reporting application logs in as Alice
```

This creates several problems:

* The reporting application receives Alice’s full credential.
* The application may receive more authority than it needs.
* Alice cannot revoke only the reporting application.
* Password changes break the integration.
* MFA and passkeys are difficult to support.
* The storage service cannot distinguish Alice from the third-party application.
* Credential compromise affects every use of the password.

OAuth introduces a delegated credential:

```text
Alice authorizes reporting application
→ authorization server issues limited access token
→ reporting application accesses only approved resources
```

The reporting application receives a scoped credential rather than Alice’s password.

---

## 9.2 OAuth is not primarily a login protocol

A common statement is:

```text
Log in with OAuth
```

This is incomplete.

OAuth itself focuses on authorization to protected resources.

A client may receive an access token without receiving a standardized, authenticated statement about the human user’s identity.

For standardized federated login, OAuth is commonly combined with OpenID Connect.

```text
OAuth access token:
  intended for a resource server

OIDC ID token:
  intended for the client
  describes an authentication event
```

Using an arbitrary OAuth access token as proof of user login can create token-substitution and audience-confusion problems.

---

# 9.3 OAuth roles

OAuth defines several logical roles.

## Resource owner

The entity capable of granting access to a protected resource.

In common user-delegation flows:

```text
Resource owner = end user
```

But not every flow involves a human.

---

## Client

The application requesting access.

Examples:

* Server-rendered web application
* Browser-based application
* Mobile application
* CLI tool
* Backend service
* Automation platform

The word **client** refers to its OAuth role, not necessarily to a physical client machine.

---

## Authorization server

The component that:

* Authenticates or otherwise interacts with the resource owner
* Obtains authorization
* Issues access tokens
* May issue refresh tokens
* May expose authorization, token, introspection, revocation, and metadata endpoints

---

## Resource server

The API or service hosting protected resources.

It receives an access token and decides whether the represented authority permits the requested operation.

Examples:

* Reports API
* File-storage API
* Calendar API
* Cloud management API
* UserInfo endpoint

---

## User agent

Usually the browser that carries front-channel messages between the client and authorization server.

The user agent is not automatically trusted merely because the user operates it.

URLs, query parameters, browser history, extensions, scripts, and redirects can all expose data.

---

## Logical architecture

```mermaid
flowchart LR
    U[Resource Owner / End User]
    B[Browser / User Agent]
    C[OAuth Client]
    AS[Authorization Server]
    RS[Resource Server]

    U --> B
    B --> C
    C -->|Authorization request through browser| AS
    AS -->|Authorization response through browser| C
    C -->|Token request| AS
    AS -->|Access token| C
    C -->|Access token + API request| RS
```

---

# 9.4 Concrete example architecture

We will follow one example:

> Alice opens an internal finance web application. The application uses the corporate identity provider for login and calls a separate reports API.

Components:

```text
Browser:
  finance.example.internal

Backend for Frontend:
  finance-web-backend

OpenID Provider / Authorization Server:
  identity.example.internal

Resource Server:
  reports-api.example.internal
```

The architecture uses:

* OpenID Connect Authorization Code flow
* PKCE
* A confidential server-side client
* A browser session cookie
* Access tokens kept in the backend
* An ID Token validated by the backend
* A Reports API access token
* Local application authorization after login

```mermaid
flowchart LR
    A[Alice]
    B[Browser]
    F[Finance BFF]
    I[Identity Provider]
    R[Reports API]
    S[(Session Store)]

    A --> B
    B -->|Session cookie| F
    F -->|OIDC authorization request| I
    I -->|Authorization code| F
    F -->|Code exchange| I
    I -->|ID token + access token| F
    F --> S
    F -->|Bearer access token| R
```

For browser applications, the current IETF best-practice document RFC 10017, published in July 2026, describes three main patterns: a Backend for Frontend, a token-mediating backend, and a browser-based OAuth client. It presents those patterns in decreasing order of security, with the BFF keeping access and refresh tokens out of browser JavaScript.

---

# 9.5 Front channel and back channel

Understanding the data path is essential.

## Front channel

Messages travel through the browser.

Examples:

* Redirect to authorization endpoint
* Authorization request
* Authorization response
* Authorization code
* OAuth errors
* SAML Response through an auto-submitted form

```text
Authorization server
→ browser
→ client redirect endpoint
```

Front-channel data may be exposed to:

* Browser history
* Extensions
* Scripts
* Referrer headers
* Proxy logs
* Screenshots
* User copying
* Open redirects

Therefore, long-lived access tokens should not ordinarily be exposed through front-channel URLs.

---

## Back channel

Messages travel directly between server components.

Examples:

* Authorization-code exchange
* Token refresh
* Token introspection
* UserInfo request
* SAML artifact resolution
* Client authentication

```text
Client backend
→ direct HTTPS request
→ authorization server
```

The browser does not normally see this exchange.

---

## Why separate channels?

The browser is useful for:

* User interaction
* Login
* Consent
* Redirecting between trust domains

The back channel is preferable for:

* Client authentication
* Token delivery
* Secret-bearing operations
* Direct protocol validation

---

# 9.6 Client registration

Before the flow begins, the authorization server usually has a client record.

Example:

```json
{
  "client_id": "finance-web",
  "client_type": "confidential",
  "redirect_uris": [
    "https://finance.example.internal/oidc/callback"
  ],
  "allowed_scopes": [
    "openid",
    "profile",
    "reports.read"
  ],
  "token_endpoint_auth_method": "private_key_jwt"
}
```

The client registration answers:

* Which application is this?
* Which redirect URIs are allowed?
* Which response types or grants are allowed?
* Which scopes may it request?
* How does it authenticate at the token endpoint?
* Which signing or encryption algorithms are expected?
* Which logout endpoints are allowed?

---

## Client identifier

`client_id` identifies the OAuth client.

It is normally not secret.

Example:

```text
client_id=finance-web
```

An attacker knowing the client ID should not gain client authority.

---

## Client secret

A confidential client may authenticate using a secret.

Example conceptual configuration:

```text
client_id = finance-web
client_secret = high-entropy-secret
```

A client secret must not be embedded in:

* Browser JavaScript
* Public source code
* Distributed desktop binaries
* Mobile application packages
* Public container images

Software delivered to user-controlled devices cannot reliably keep a shared secret confidential.

---

## Public client

A public client cannot reliably protect a static client credential.

Examples:

* Browser-only JavaScript application
* Installed mobile application
* Desktop application

Public does not mean untrusted or unauthenticated user.

It means:

```text
The client instance cannot securely hold a shared client secret.
```

---

## Confidential client

A confidential client can protect authentication credentials in a controlled server environment.

Examples:

* Traditional backend web application
* Backend for Frontend
* Server-side integration
* Protected automation service

A confidential client may authenticate using:

* Client secret
* Private-key JWT
* Mutual TLS
* Other deployment-specific mechanisms

Current OAuth security guidance recommends asymmetric client authentication where feasible because the authorization server need not store a reusable symmetric secret.

---

# 9.7 Authorization Code flow with OpenID Connect

The complete flow has two broad phases:

```text
Front channel:
  obtain authorization code

Back channel:
  exchange code for tokens
```

The authorization code is:

* Short-lived
* Single-use
* Bound to the client
* Bound to a redirect URI
* Bound to a PKCE challenge when PKCE is used

It is not the access token.

---

# 9.8 Step 1: Alice requests the finance application

Request:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
```

The finance backend checks for a valid local session.

```text
Session cookie absent
→ user is not locally authenticated
```

The backend begins an OIDC authorization transaction.

---

# 9.9 Step 2: Generate transaction state

The client generates several values:

```text
state
nonce
PKCE code_verifier
PKCE code_challenge
```

Example transaction record:

```json
{
  "transaction_id": "tx_93af...",
  "state_hash": "sha256:...",
  "nonce_hash": "sha256:...",
  "code_verifier_encrypted": "...",
  "original_url": "/reports",
  "created_at": "2026-07-31T10:30:00Z",
  "expires_at": "2026-07-31T10:35:00Z"
}
```

These values solve different problems.

---

## `state`

`state` associates the authorization response with the browser session and transaction that initiated it.

Simplified use:

```text
Client sends state=X
Authorization server returns state=X
Client checks state=X belongs to pending transaction
```

It can help prevent:

* Login CSRF
* Authorization-response injection
* Transaction confusion

`state` may also reference the originally requested application location.

Do not place an arbitrary return URL in unsigned, unvalidated state and redirect to it blindly.

That can create an open redirect.

---

## `nonce`

In OpenID Connect, `nonce` associates an ID Token with the client’s authentication request.

The client sends:

```text
nonce=N
```

The OpenID Provider includes:

```json
{
  "nonce": "N"
}
```

in the ID Token where required or requested.

The client verifies that it matches the pending transaction.

This helps prevent ID Token replay and transaction substitution.

---

## PKCE code verifier

The client generates a high-entropy random value:

```text
code_verifier = RANDOM_VALUE
```

This value remains private to the client transaction.

---

## PKCE code challenge

For the recommended `S256` method:

```text
code_challenge =
  BASE64URL(SHA-256(code_verifier))
```

The authorization request contains the challenge.

The token request later contains the verifier.

Current OAuth security best practice applies PKCE to all kinds of OAuth clients, not only native applications, and identifies `S256` as the method that does not expose the verifier in the authorization request. Authorization servers must support PKCE under that BCP.

---

## PKCE mental model

```mermaid
flowchart LR
    V[Random code_verifier]
    H[SHA-256]
    C[code_challenge]
    A[Authorization Request]
    T[Token Request]

    V --> H --> C --> A
    V --> T
```

The authorization server remembers:

```text
Authorization code C1
is bound to
code_challenge H(V)
```

At token exchange, it verifies:

```text
H(submitted code_verifier)
==
stored code_challenge
```

---

# 9.10 Generate PKCE values practically

The following Bash and Python example generates a verifier and `S256` challenge:

```bash
python3 <<'PY'
import base64
import hashlib
import secrets

verifier = base64.urlsafe_b64encode(
    secrets.token_bytes(32)
).rstrip(b"=").decode("ascii")

challenge = base64.urlsafe_b64encode(
    hashlib.sha256(verifier.encode("ascii")).digest()
).rstrip(b"=").decode("ascii")

print(f"code_verifier={verifier}")
print(f"code_challenge={challenge}")
PY
```

## What it does

1. Generates 32 secure random bytes.
2. Encodes them using URL-safe Base64 without padding.
3. Hashes the ASCII verifier with SHA-256.
4. Encodes the digest as the challenge.

## Why it is useful

It demonstrates the exact relationship between:

```text
code_verifier
and
code_challenge
```

## Output interpretation

Illustrative output:

```text
code_verifier=0kYl...REDACTED
code_challenge=kQ2B...REDACTED
```

The values change every time.

## Safety

The verifier is transaction-sensitive.

Do not log real production verifiers or reuse one value across transactions.

Production applications should use a maintained OAuth/OIDC library rather than shell-generated values.

---

# 9.11 Step 3: Construct authorization request

The finance backend redirects the browser to the authorization endpoint.

Conceptual request:

```http
GET /authorize?
    response_type=code&
    client_id=finance-web&
    redirect_uri=https%3A%2F%2Ffinance.example.internal%2Foidc%2Fcallback&
    scope=openid%20profile%20reports.read&
    state=STATE_VALUE&
    nonce=NONCE_VALUE&
    code_challenge=CHALLENGE_VALUE&
    code_challenge_method=S256
Host: identity.example.internal
```

Parameters:

| Parameter                    | Purpose                                     |
| ---------------------------- | ------------------------------------------- |
| `response_type=code`         | Request an authorization code               |
| `client_id`                  | Identify the client                         |
| `redirect_uri`               | Define where the response must be delivered |
| `scope`                      | Request OAuth permissions and OIDC behavior |
| `state`                      | Bind browser transaction                    |
| `nonce`                      | Bind resulting ID Token                     |
| `code_challenge`             | Bind code to PKCE verifier                  |
| `code_challenge_method=S256` | Define challenge transformation             |

---

## `openid` scope

The presence of:

```text
openid
```

makes the request an OpenID Connect authentication request rather than plain OAuth authorization.

Additional scopes may request user claims:

```text
profile
email
```

Application/API scopes may request delegated authority:

```text
reports.read
```

Do not assume every authorization server uses identical scope names or semantics.

---

# 9.12 Step 4: Validate redirect URI

The authorization server resolves the client record and compares the supplied redirect URI against registered values.

Expected:

```text
https://finance.example.internal/oidc/callback
```

Received:

```text
https://finance.example.internal/oidc/callback
```

The current OAuth Security BCP requires exact string matching for redirect URIs, except for the specifically defined localhost-port behavior of native applications. It also warns against open redirectors.

---

## Dangerous wildcard registration

```text
https://*.example.internal/*
```

This may allow responses to be sent to:

* Untrusted subdomains
* Compromised applications
* Attacker-controlled paths
* Open redirect endpoints

---

## Dangerous prefix matching

Registered:

```text
https://finance.example.internal/callback
```

Attacker supplies:

```text
https://finance.example.internal/callback-attacker
```

A naïve prefix check may incorrectly accept it.

---

# 9.13 Step 5: Identity provider authenticates Alice

The authorization server or OpenID Provider determines whether Alice already has a login session.

Possible paths:

```text
Existing IdP session satisfies policy
→ no new credential prompt
```

or:

```text
No suitable IdP session
→ request password/passkey/MFA
```

The finance application does not receive Alice’s password.

Alice enters credentials only at the identity provider.

This reduces the number of applications handling primary authentication factors.

---

## Authentication context

The provider may record:

```json
{
  "subject": "idp-user-731",
  "auth_time": "2026-07-31T10:31:02Z",
  "methods": ["pwd", "otp"],
  "assurance_level": "corporate-mfa"
}
```

OpenID Connect can communicate authentication-related claims such as:

* `auth_time`
* `acr`
* `amr`

The relying application must define which values it trusts and how they map to local assurance policy.

---

# 9.14 Step 6: Authorization and consent

The authorization server evaluates whether the client may receive the requested authority.

Inputs may include:

* Client registration
* Requested scopes
* User identity
* Organizational policy
* Existing consent
* Resource ownership
* Administrator approval
* Risk
* Device state

Possible result:

```text
Approved:
  openid
  profile
  reports.read

Rejected:
  reports.write
```

Consent and authorization are not identical.

A user clicking “Allow” does not override every organizational policy.

Similarly, an administrator may pre-authorize access without showing an interactive consent screen.

---

# 9.15 Step 7: Authorization code issuance

The authorization server creates a short-lived code record:

```json
{
  "code_hash": "sha256:...",
  "client_id": "finance-web",
  "redirect_uri": "https://finance.example.internal/oidc/callback",
  "subject_id": "idp-user-731",
  "scopes": ["openid", "profile", "reports.read"],
  "code_challenge": "CHALLENGE_VALUE",
  "code_challenge_method": "S256",
  "nonce": "NONCE_VALUE",
  "expires_at": "2026-07-31T10:33:02Z",
  "used": false
}
```

The code is returned through the browser:

```http
HTTP/1.1 302 Found
Location: https://finance.example.internal/oidc/callback?
          code=AUTHORIZATION_CODE&
          state=STATE_VALUE
```

The browser follows the redirect.

---

## Why use a code?

The front channel receives a short-lived intermediate value rather than the final access token.

The backend can then:

* Authenticate itself
* Present the PKCE verifier
* Perform direct TLS communication
* Receive tokens outside browser URLs
* Detect code replay
* Keep refresh tokens outside browser JavaScript

Current OAuth security guidance recommends the authorization code response rather than delivering access tokens in the authorization response. It advises against the implicit grant because of token leakage and replay risks.

---

# 9.16 Step 8: Validate callback transaction

The finance backend receives:

```http
GET /oidc/callback?code=AUTHORIZATION_CODE&state=STATE_VALUE
```

It must:

1. Locate the pending transaction.
2. Validate `state`.
3. Confirm the transaction is not expired.
4. Confirm it has not already completed.
5. Confirm the browser session is bound to the transaction.
6. Recover the PKCE verifier.
7. Recover the expected nonce.
8. Avoid logging the full callback URL.
9. Reject unsolicited or duplicate responses.

Pseudocode:

```python
transaction = lookup_transaction(hash(request.query["state"]))

if transaction is None:
    reject("unknown_state")

if transaction.expires_at < now():
    reject("expired_transaction")

if transaction.completed:
    reject("replayed_callback")
```

Validation should occur before the application treats the code as meaningful.

---

# 9.17 Step 9: Back-channel token request

The backend sends a direct HTTPS request to the token endpoint:

```http
POST /token HTTP/1.1
Host: identity.example.internal
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
redirect_uri=https%3A%2F%2Ffinance.example.internal%2Foidc%2Fcallback&
client_id=finance-web&
code_verifier=CODE_VERIFIER
```

A confidential client also authenticates using its configured method.

Examples:

* HTTP Basic client secret
* Client secret in request body
* Private-key JWT
* Mutual TLS

The exact client-authentication method depends on registration and provider support.

---

# 9.18 Step 10: Authorization server validates code exchange

The authorization server checks:

* Code exists
* Code is unexpired
* Code is unused
* Client ID matches
* Redirect URI matches
* Client authentication succeeds where required
* PKCE verifier matches the challenge
* Grant remains valid
* Subject and client are allowed
* Requested token policy is satisfied

PKCE check:

```text
BASE64URL(SHA-256(submitted_verifier))
==
stored_challenge
```

If any check fails:

```text
No token issued
```

After successful exchange:

```text
Authorization code marked used
```

A second exchange attempt should fail.

---

## What PKCE prevents

Suppose an attacker steals the authorization code from the browser redirect.

Without PKCE:

```text
Stolen code
→ attacker exchanges it
→ attacker receives tokens
```

With PKCE:

```text
Stolen code
+ no verifier
→ token endpoint rejects exchange
```

PKCE helps ensure that the client instance redeeming the code is the one that initiated the transaction. RFC 10017 requires browser-based public clients using the code grant to use PKCE and requires authorization servers to support and enforce it for those clients.

---

## What PKCE does not prevent

PKCE does not by itself prevent:

* Malicious code running inside the legitimate client
* Theft of both code and verifier
* Access-token theft after issuance
* Incorrect issuer validation
* Incorrect redirect URI registration
* Compromised authorization server
* XSS controlling a browser-based client
* Overly broad scopes
* API authorization bugs

---

# 9.19 Step 11: Token response

Illustrative response:

```json
{
  "token_type": "Bearer",
  "access_token": "ACCESS_TOKEN",
  "expires_in": 900,
  "refresh_token": "REFRESH_TOKEN",
  "scope": "openid profile reports.read",
  "id_token": "ID_TOKEN"
}
```

Each item serves a different purpose.

---

## Access token

Presented to the resource server.

```http
Authorization: Bearer ACCESS_TOKEN
```

It represents delegated authority such as:

```text
reports.read
```

A bearer token can generally be used by whoever possesses it without separately proving control of a cryptographic key, so it must be protected in storage and transport.

---

## ID Token

Presented to and validated by the OpenID Connect client.

It communicates the authentication result.

It is not normally sent to the Reports API.

---

## Refresh token

Presented to the authorization server to obtain new access tokens.

It is not normally sent to the resource server.

---

## Token type

Commonly:

```text
Bearer
```

Other token-binding or proof-of-possession mechanisms may require additional proof.

---

## Expiry

`expires_in` describes access-token lifetime in seconds.

Example:

```text
900 seconds = 15 minutes
```

The client should track expiry conservatively rather than waiting for the exact final millisecond.

---

# 9.20 Step 12: Validate the ID Token

A simplified ID Token payload:

```json
{
  "iss": "https://identity.example.internal",
  "sub": "idp-user-731",
  "aud": "finance-web",
  "exp": 1785495662,
  "iat": 1785494762,
  "nonce": "NONCE_VALUE",
  "auth_time": 1785494662,
  "acr": "corporate-mfa",
  "amr": ["pwd", "otp"]
}
```

The client must not trust these claims merely because the JWT can be decoded.

---

## Core validation steps

1. Parse the token safely.
2. Select a trusted verification key.
3. Enforce expected signing algorithms.
4. Verify signature.
5. Require exact expected issuer.
6. Require client ID in audience.
7. Validate expiry.
8. Validate not-before if present.
9. Validate issued-at according to policy.
10. Validate nonce against transaction.
11. Validate authorized party where applicable.
12. Map subject using issuer and subject together.
13. Apply authentication-strength policy.

OpenID Connect requires the issuer to exactly match the expected OpenID Provider and requires the audience to contain the client’s registered `client_id`.

---

## Subject identity

The stable identity key should normally be treated as:

```text
(issuer, subject)
```

Example:

```text
issuer  = https://identity.example.internal
subject = idp-user-731
```

Do not assume:

```text
sub alone
```

is globally unique across unrelated issuers.

---

## Email is not a reliable primary identity key

An email claim may:

* Change
* Be reassigned
* Differ in case handling
* Be absent
* Be unverified
* Collide across issuers
* Represent a contact address rather than account identity

Prefer a stable issuer-subject mapping.

Example local identity mapping:

```text
external_issuer = https://identity.example.internal
external_subject = idp-user-731
local_user_id = u_1048
```

---

# 9.21 Step 13: Create local application session

After successful ID Token validation, the finance backend creates its own local session:

```json
{
  "local_session_id": "s_71bb...",
  "local_user_id": "u_1048",
  "issuer": "https://identity.example.internal",
  "external_subject": "idp-user-731",
  "auth_time": "2026-07-31T10:31:02Z",
  "auth_methods": ["pwd", "otp"],
  "token_reference": "token-set-172",
  "status": "active"
}
```

The browser receives:

```http
Set-Cookie: __Host-session=s_71bb...; Secure; HttpOnly; SameSite=Lax; Path=/
```

The access and refresh tokens remain in backend-controlled storage.

---

## Why create a local session?

The application needs local control over:

* Logout
* Idle expiry
* Application roles
* User mapping
* Token storage
* CSRF protection
* Step-up authentication
* Impersonation
* Audit context
* Session revocation

Federated login does not eliminate local session management.

---

# 9.22 Step 14: Call the Reports API

The browser calls the finance backend:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
Cookie: __Host-session=s_71bb...
```

The backend resolves the local session and retrieves the access token.

It calls the resource server:

```http
GET /v1/reports HTTP/1.1
Host: reports-api.example.internal
Authorization: Bearer ACCESS_TOKEN
```

The browser does not receive the access token in this BFF architecture.

---

# 9.23 Step 15: Resource server validates access token

The Reports API validates:

* Token format
* Signature or introspection result
* Trusted issuer
* Intended audience
* Expiry
* Not-before
* Token type
* Required scopes
* Subject/client distinction
* Tenant
* Revocation where required
* Proof-of-possession information where applicable

It then performs its own authorization.

Example:

```text
Token scope:
  reports.read

Subject:
  u_1048

Requested resource:
  report-71

Application policy:
  subject.department == report.department
```

Both must succeed:

```text
Token grants broad API permission
AND
resource-level policy allows this report
```

---

## Scope is not complete authorization

A scope such as:

```text
reports.read
```

may mean:

```text
The client is permitted to request report reads.
```

It does not necessarily mean:

```text
The subject may read every report.
```

The Reports API still needs:

* Object-level authorization
* Tenant isolation
* Resource ownership checks
* Field filtering
* Business rules

---

# 9.24 End-to-end OIDC sequence

```mermaid
sequenceDiagram
    actor Alice
    participant Browser
    participant BFF as Finance BFF
    participant OP as OpenID Provider
    participant API as Reports API
    participant Store as Session/Token Store

    Alice->>Browser: Open /reports
    Browser->>BFF: GET /reports
    BFF->>BFF: No local session
    BFF->>BFF: Generate state, nonce, verifier, challenge
    BFF->>Store: Save pending transaction
    BFF-->>Browser: Redirect to authorization endpoint

    Browser->>OP: Authorization request
    OP->>OP: Validate client and redirect URI
    OP->>Alice: Authenticate and apply policy
    Alice->>OP: Complete authentication
    OP->>OP: Create authorization code bound to PKCE
    OP-->>Browser: Redirect with code and state

    Browser->>BFF: Callback with code and state
    BFF->>Store: Load pending transaction
    BFF->>BFF: Validate state
    BFF->>OP: Code + verifier + client authentication
    OP->>OP: Validate code, client, redirect URI, PKCE
    OP-->>BFF: ID token + access token + refresh token

    BFF->>BFF: Validate ID token and nonce
    BFF->>Store: Create local session and store token set
    BFF-->>Browser: Set session cookie; redirect /reports

    Browser->>BFF: GET /reports + session cookie
    BFF->>Store: Resolve local session and access token
    BFF->>API: GET /reports + bearer token
    API->>API: Validate token and authorize resources
    API-->>BFF: Authorized reports
    BFF-->>Browser: Render response
```

---

# 9.25 Data transformation table

| Stage                  | Input                                      | Output                                |
| ---------------------- | ------------------------------------------ | ------------------------------------- |
| Transaction start      | Unauthenticated browser request            | State, nonce, PKCE verifier/challenge |
| Authorization request  | Client ID, redirect URI, scopes, challenge | User interaction at provider          |
| Authentication         | User factors                               | Provider authentication context       |
| Authorization          | Client, user, scopes, policy               | Authorization code                    |
| Callback               | Code and state                             | Validated pending transaction         |
| Token exchange         | Code, verifier, client authentication      | ID/access/refresh tokens              |
| ID validation          | Signed ID Token                            | Local external identity               |
| Session creation       | External identity and token set            | Local application session             |
| API call               | Access token                               | Resource-server security context      |
| Resource authorization | Subject, scope, resource, policy           | Allow or deny                         |

---

# 9.26 `state`, `nonce`, and PKCE comparison

| Mechanism | Main binding                                                  |
| --------- | ------------------------------------------------------------- |
| `state`   | Authorization response to browser/client transaction          |
| `nonce`   | ID Token to authentication request                            |
| PKCE      | Authorization code to client instance that initiated the flow |

They overlap in some attack defenses but are not interchangeable in every deployment.

A secure library may use multiple mechanisms.

Do not remove one merely because another value appears to “look random.”

---

# 9.27 Authorization code versus access token

| Property                | Authorization code  | Access token                          |
| ----------------------- | ------------------- | ------------------------------------- |
| Purpose                 | Exchange for tokens | Access protected resource             |
| Typical lifetime        | Very short          | Short                                 |
| Typical use count       | Once                | Many requests until expiry            |
| Intended recipient      | Token endpoint      | Resource server                       |
| Travels through browser | Usually yes         | Preferably not in code flow           |
| Bound to redirect URI   | Yes                 | Not normally in same way              |
| Bound to PKCE           | Yes when used       | Indirect result of validated exchange |

Using an authorization code directly at an API is incorrect.

Using an access token at the callback endpoint as if it were a code is also incorrect.

---

# 9.28 ID Token versus access token

| Property             | ID Token              | Access token          |
| -------------------- | --------------------- | --------------------- |
| Intended recipient   | OpenID Connect client | Resource server       |
| Main purpose         | Authentication result | API authorization     |
| Audience             | Client ID             | API/resource audience |
| Contains user claims | Often                 | Format-dependent      |
| Used to call API     | Normally no           | Yes                   |
| Creates local login  | After validation      | Should not be assumed |
| Standard format      | JWT                   | Opaque or structured  |

---

## Common failure: sending ID Token to API

```http
Authorization: Bearer ID_TOKEN
```

Why it fails conceptually:

* The audience may be the client, not the API.
* The token represents authentication, not delegated API authority.
* The API may incorrectly accept claims not intended for it.
* Scope semantics may be absent.

---

## Common failure: using access token as login proof

A client receives an access token intended for:

```text
contacts-api
```

and uses it as proof that the browser user logged into the client.

Problems:

* Audience belongs to the API.
* Token may represent a client rather than user.
* Token structure may be opaque.
* Token may have been obtained by another client.
* Identity claims may have undefined semantics.

OIDC solves this by defining an ID Token intended for the relying client.

---

# 9.29 OIDC discovery

An OpenID Provider commonly publishes metadata at a well-known endpoint.

Example command:

```bash
curl -sS \
  https://identity.example.internal/.well-known/openid-configuration \
  | jq
```

## What it does

* Requests the provider’s discovery document.
* Formats the JSON using `jq`.

## Why it is useful

It can reveal:

* Issuer
* Authorization endpoint
* Token endpoint
* UserInfo endpoint
* JWKS URI
* Supported scopes
* Supported response types
* Supported signing algorithms
* PKCE support metadata

## Illustrative output

```json
{
  "issuer": "https://identity.example.internal",
  "authorization_endpoint": "https://identity.example.internal/authorize",
  "token_endpoint": "https://identity.example.internal/token",
  "userinfo_endpoint": "https://identity.example.internal/userinfo",
  "jwks_uri": "https://identity.example.internal/jwks",
  "response_types_supported": ["code"],
  "code_challenge_methods_supported": ["S256"]
}
```

## Safety

Do not replace the configured trusted issuer with a value supplied by an untrusted request.

Dynamic discovery based on arbitrary user-controlled domains can create server-side request forgery or issuer-confusion risks.

---

# 9.30 JWKS

A JSON Web Key Set publishes public verification keys.

Fetch it:

```bash
curl -sS \
  https://identity.example.internal/jwks \
  | jq
```

Illustrative response:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "signing-2026-07",
      "use": "sig",
      "alg": "RS256",
      "n": "BASE64URL_MODULUS",
      "e": "AQAB"
    }
  ]
}
```

## Important fields

| Field    | Meaning                   |
| -------- | ------------------------- |
| `kty`    | Key type                  |
| `kid`    | Key identifier            |
| `use`    | Intended use              |
| `alg`    | Declared algorithm        |
| `n`, `e` | RSA public-key parameters |

## Why it is useful

Clients use public keys to verify ID Token signatures.

Resource servers may use the same mechanism for JWT access tokens.

## Common failure

A new signing key is used before clients refresh the JWKS cache.

Result:

```text
unknown kid
→ token rejected
```

A safe rotation period overlaps old and new keys.

---

# 9.31 OIDC UserInfo endpoint

The UserInfo endpoint is an OAuth-protected resource that returns claims about the authenticated end user.

Request:

```bash
curl -sS \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  https://identity.example.internal/userinfo \
  | jq
```

## What it does

Presents the access token to the provider’s UserInfo endpoint.

## Why it is useful

It can retrieve claims such as:

* Subject identifier
* Display name
* Email
* Profile information

## Illustrative response

```json
{
  "sub": "idp-user-731",
  "name": "Alice Example",
  "email": "alice@example.internal",
  "email_verified": true
}
```

## Critical validation

The `sub` returned by UserInfo should correspond to the authenticated subject represented by the ID Token.

Do not merge identity information from mismatched subjects.

OpenID Connect defines UserInfo as an OAuth-protected resource and requires TLS for communication with it.

## Safety

Access tokens are credentials.

Avoid shell history and verbose traces.

A safer interactive pattern:

```bash
read -r -s ACCESS_TOKEN
curl -sS \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  https://identity.example.internal/userinfo \
  | jq
unset ACCESS_TOKEN
```

---

# 9.32 Testing token endpoint manually

A controlled test request might use:

```bash
curl -sS \
  -X POST \
  https://identity.example.internal/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=authorization_code' \
  --data-urlencode "code=${AUTHORIZATION_CODE}" \
  --data-urlencode 'redirect_uri=https://finance.example.internal/oidc/callback' \
  --data-urlencode 'client_id=finance-web' \
  --data-urlencode "code_verifier=${CODE_VERIFIER}"
```

## What it does

Submits an authorization-code token request.

## Important flags

* `-X POST`: use POST
* `-H`: set content type
* `--data-urlencode`: safely form-encode each parameter

## Why it is useful

It helps diagnose:

* Code validity
* Redirect URI mismatch
* PKCE mismatch
* Client authentication
* Token endpoint errors

## Safety

Authorization codes and PKCE verifiers are temporary credentials.

Manual redemption may consume the code, causing the real application callback to fail.

Use only in an isolated test flow.

---

# 9.33 Token endpoint errors

Illustrative response:

```json
{
  "error": "invalid_grant",
  "error_description": "Authorization code is invalid or expired"
}
```

Possible causes include:

* Code already used
* Code expired
* Wrong client
* Wrong redirect URI
* Wrong PKCE verifier
* Code issued by another authorization server
* Transaction state deleted
* Clock or replication issue

Do not expose excessive internal detail in user-facing responses.

Internal logs should identify the exact validation stage.

---

# 9.34 Refresh flow

When the access token expires, the backend may present its refresh token:

```http
POST /token HTTP/1.1
Host: identity.example.internal
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token=REFRESH_TOKEN&
client_id=finance-web
```

The authorization server validates:

* Refresh token exists
* Token belongs to client
* Token is not expired
* Token is not revoked
* Grant remains valid
* User/account policy remains acceptable
* Client authentication succeeds
* Requested scopes do not exceed allowed authority

It returns a new access token and may rotate the refresh token.

---

## Refresh-token rotation

```text
Refresh token A used
→ A invalidated
→ refresh token B issued
```

If A later appears again:

```text
Possible theft or replay
```

The server may revoke the token family and require reauthentication.

Concurrency must be handled carefully because two legitimate refresh attempts can resemble replay.

---

# 9.35 OAuth client-credentials flow

The client-credentials grant represents the client acting on its own behalf.

Example:

```text
backup-service
→ obtains token
→ calls storage API
```

No human user is involved.

```mermaid
sequenceDiagram
    participant Client as Backup Service
    participant AS as Authorization Server
    participant API as Storage API

    Client->>AS: Client authentication + client_credentials grant
    AS-->>Client: Access token
    Client->>API: Request + access token
    API->>API: Authorize service identity
    API-->>Client: Response
```

Token request:

```bash
curl -sS \
  -X POST \
  https://identity.example.internal/token \
  -u "${CLIENT_ID}:${CLIENT_SECRET}" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=client_credentials' \
  --data-urlencode 'scope=reports.generate'
```

## What it does

Authenticates a confidential client and requests an access token for its own service identity.

## Important flag

* `-u`: sends HTTP Basic credentials

## Safety

Command-line secrets can leak through:

* Shell history
* Process listings
* CI output
* Debug traces

Prefer private-key client authentication or a secrets-management integration in production.

---

## User token versus service token

A resource server must distinguish:

```text
User-delegated token:
  subject represents resource owner
  client represents acting application

Service token:
  subject may represent client/service itself
```

Confusing these forms can cause a service identity to be mistaken for a human user.

Current OAuth security guidance explicitly warns resource servers to distinguish tokens representing clients from tokens representing resource owners.

---

# 9.36 Resource Owner Password Credentials grant

This legacy flow asks the client to collect the user’s username and password and send them to the authorization server.

```text
User
→ gives password to OAuth client
→ client sends password to authorization server
```

This defeats a central design advantage of federation:

```text
The client should not receive the user's primary credential.
```

The current OAuth Security BCP states that this grant must not be used because it exposes resource-owner credentials to the client and does not fit modern multi-step or cryptographic authentication methods.

---

# 9.37 Implicit grant

The implicit grant delivers an access token through the authorization response.

Conceptually:

```text
Authorization server
→ browser URL
→ access token
```

This exposes the token to additional front-channel risks.

Modern deployments should use authorization code flow with PKCE rather than adopting the implicit grant merely because the client runs in a browser. Current OAuth guidance recommends against response types that issue access tokens directly through the authorization response.

---

# 9.38 Browser application architecture patterns

## Pattern 1: Backend for Frontend

```mermaid
flowchart LR
    B[Browser]
    F[BFF]
    AS[Authorization Server]
    API[Resource API]

    B -->|Session cookie| F
    F -->|OAuth| AS
    F -->|Access token| API
```

Tokens remain server-side.

### Advantages

* Refresh tokens not exposed to browser JavaScript
* Access tokens not directly readable by XSS
* Client can authenticate confidentially
* Central token management
* Browser uses familiar secure session cookie

### Remaining risks

* XSS can still make authenticated BFF requests
* CSRF must be addressed
* BFF becomes a sensitive proxy
* Token and session stores require protection
* Backend compromise exposes tokens

---

## Pattern 2: Token-mediating backend

```mermaid
flowchart LR
    B[Browser]
    T[Token Backend]
    AS[Authorization Server]
    API[Resource API]

    B --> T
    T --> AS
    T -->|Access token delivered| B
    B -->|Access token| API
```

The backend handles OAuth, but the browser receives access tokens for direct API calls.

### Tradeoff

* Backend manages refresh and protocol complexity
* Browser still has token-extraction exposure

---

## Pattern 3: Browser-based OAuth client

```mermaid
flowchart LR
    B[Browser OAuth Client]
    AS[Authorization Server]
    API[Resource API]

    B -->|Authorization Code + PKCE| AS
    AS -->|Tokens| B
    B -->|Access token| API
```

The browser handles OAuth directly.

### Tradeoff

* Simpler deployment
* No confidential client secret
* Access and possibly refresh tokens exist in browser execution context
* XSS may extract tokens
* CORS and storage design become important

RFC 10017 documents these patterns and their security tradeoffs for browser-based applications.

---

# 9.39 OAuth and OIDC common failures

## Failure: redirect URI too broad

```text
Attacker-controlled redirect accepted
→ code delivered to attacker
```

---

## Failure: missing PKCE validation

```text
Attacker steals code
→ exchanges it before legitimate client
```

---

## Failure: constant PKCE verifier

```text
Every transaction uses same verifier
→ theft of one value weakens all flows
```

PKCE values must be transaction-specific.

---

## Failure: `state` accepted but not bound to browser

```text
Attacker obtains their own state/code pair
→ injects response into victim browser
```

Randomness alone is insufficient if the server does not bind the value to the initiating transaction.

---

## Failure: nonce ignored

```text
Old valid ID Token replayed
→ client creates new session
```

---

## Failure: wrong issuer accepted

```text
Token signed by another trusted provider
→ accepted in incorrect tenant or provider context
```

This is especially dangerous in multi-provider clients.

---

## Failure: audience ignored

```text
ID Token intended for application A
→ replayed at application B
```

or:

```text
Access token intended for API A
→ accepted by API B
```

---

## Failure: email used as universal identity

```text
Same email from different issuer
→ accounts merged incorrectly
```

---

## Failure: refresh token stored in browser local storage

```text
XSS executes
→ extracts long-lived refresh token
→ attacker obtains continuing access
```

---

## Failure: token logged

Possible leakage points:

* Reverse proxy
* Application logger
* Exception tracker
* Browser console
* Request trace
* Analytics system
* CI output
* Shell history

---

## Failure: client secret embedded in SPA

```text
Secret shipped to every browser
→ secret is public
```

The authorization server cannot treat it as confidential client authentication.

---

## Failure: scope treated as resource ownership

```text
scope=reports.read
→ application returns every report
```

Scope must be combined with object authorization.

---

## Failure: consent mistaken for authentication

A consent event answers:

```text
Did the resource owner approve requested access?
```

It does not necessarily answer:

```text
Which authentication assurance was used?
```

---

## Failure: local account status ignored

The IdP authenticates Alice successfully, but the finance application’s local account is suspended.

The relying application must still apply local policy.

---

# 9.40 OAuth troubleshooting by stage

## Stage 1: Authorization request construction

Check:

* Correct authorization endpoint
* Correct client ID
* Exact redirect URI
* Correct scopes
* `response_type=code`
* Unique state
* Unique nonce
* PKCE challenge and method

---

## Stage 2: Identity-provider interaction

Check:

* User authentication succeeded
* MFA policy satisfied
* Client allowed
* Consent or administrator policy allowed
* User assigned to application
* Tenant selection correct

---

## Stage 3: Authorization response

Check:

* Callback route reached
* Code present
* State present
* Error parameters
* Browser did not block navigation
* Proxy preserved query parameters
* Transaction still exists

---

## Stage 4: Token exchange

Check:

* Code unused
* Code unexpired
* Same redirect URI
* Same client ID
* Correct client authentication
* Correct PKCE verifier
* Correct token endpoint
* TLS validation

---

## Stage 5: ID Token validation

Check:

* Expected issuer
* Known `kid`
* Signature
* Algorithm
* Audience
* Expiry
* Nonce
* Subject mapping
* Clock synchronization

---

## Stage 6: Local session creation

Check:

* User mapping
* Account state
* Session-store write
* Cookie attributes
* Proxy header behavior
* Session signing keys
* Redirect after callback

---

## Stage 7: API request

Check:

* Access token selected
* Token not expired
* Correct audience
* Required scope
* API trust configuration
* Object-level authorization
* Tenant context

---

# 9.41 Illustrative OIDC logs

```text
2026-07-31T10:30:00Z
event=oidc_authorization_started
request_id=req_701
transaction_id_hash=sha256:...
client_id=finance-web
issuer=https://identity.example.internal

2026-07-31T10:31:06Z
event=oidc_callback_received
request_id=req_702
state_valid=true
code_present=true

2026-07-31T10:31:06Z
event=oidc_token_exchange
request_id=req_702
result=success
token_endpoint_duration_ms=142

2026-07-31T10:31:06Z
event=id_token_validation
request_id=req_702
issuer_valid=true
audience_valid=true
signature_valid=true
nonce_valid=true
subject_hash=sha256:...

2026-07-31T10:31:06Z
event=local_session_created
request_id=req_702
local_user_id=u_1048
session_id_hash=sha256:...
```

Do not log:

```text
authorization_code
code_verifier
access_token
refresh_token
raw ID Token
client_secret
```

---

# 9.42 OpenID Connect mental model

OpenID Connect adds an identity layer to OAuth.

Roles:

| OpenID Connect term    | Meaning                                      |
| ---------------------- | -------------------------------------------- |
| OpenID Provider, or OP | Performs authentication and issues ID Tokens |
| Relying Party, or RP   | Client that relies on authentication result  |
| End User               | Human identity being authenticated           |
| ID Token               | Signed statement about authentication        |
| UserInfo endpoint      | Protected resource returning user claims     |

```mermaid
flowchart LR
    U[End User]
    RP[Relying Party]
    OP[OpenID Provider]
    UI[UserInfo Endpoint]

    U --> RP
    RP --> OP
    OP -->|ID Token| RP
    RP -->|Access Token| UI
    UI -->|User Claims| RP
```

OpenID Connect defines authentication on top of OAuth 2.0 and uses claims to communicate information about the end user.

---

# 9.43 Authentication result versus user profile

The ID Token primarily communicates the authentication result.

Example:

```json
{
  "iss": "https://identity.example.internal",
  "sub": "idp-user-731",
  "aud": "finance-web",
  "exp": 1785495662,
  "iat": 1785494762,
  "auth_time": 1785494662
}
```

Profile information may appear:

* In the ID Token
* At UserInfo
* In a directory
* In the application’s local user database

Do not require all mutable profile data to be embedded in every ID Token.

---

# 9.44 `acr`, `amr`, and `auth_time`

## `auth_time`

When the user authentication occurred.

Example policy:

```text
Sensitive action requires authentication age < 5 minutes
```

---

## `amr`

Authentication Method References.

Possible values may indicate methods such as:

```text
password
one-time password
hardware key
biometric-assisted device unlock
```

Meaning and exact values depend on the provider and profile.

---

## `acr`

Authentication Context Class Reference.

It may represent an assurance class such as:

```text
corporate-mfa
phishing-resistant
high-assurance
```

The relying party must have an agreed interpretation.

A string containing “high” is not secure merely because of its name.

---

# 9.45 OIDC session layers

A federated login may create several independent sessions:

```text
1. Identity-provider session
2. OAuth authorization grant
3. Local application session
4. API token set
```

Logging out of one does not automatically remove all others.

Example:

```text
Local finance session deleted
but
IdP session remains active
```

The next login attempt may immediately succeed without credential prompts because the IdP still has a valid session.

This is normal unless the application requests or coordinates provider logout.

---

# 9.46 Single Sign-On

Single Sign-On means one authentication context can be reused across multiple applications.

Example:

```text
Alice authenticates at corporate IdP
→ opens finance application
→ opens HR application
→ IdP session satisfies both
```

Each application may still create its own local session.

```mermaid
flowchart TD
    I[Identity Provider Session]
    F[Finance Local Session]
    H[HR Local Session]
    T[Ticketing Local Session]

    I --> F
    I --> H
    I --> T
```

SSO reduces repeated credential entry.

It also increases the impact of:

* IdP compromise
* Stolen IdP session
* Incorrect federation configuration
* Weak recovery
* Excessively long IdP session lifetime

---

# 9.47 Single Logout is harder than Single Sign-On

A user may have:

* IdP session
* Several application sessions
* Refresh tokens
* Access tokens
* Mobile sessions
* Offline jobs

A logout message may fail to reach one application.

Therefore:

```text
Logout request sent
≠
every downstream credential is certainly invalid
```

Systems should distinguish:

* Local logout
* Provider logout
* Global logout attempt
* Token revocation
* Account disablement
* Session expiry

---

# 9.48 SAML core mental model

SAML exchanges XML assertions between trusted parties.

Typical browser SSO roles:

| Role                               | Meaning                                                |
| ---------------------------------- | ------------------------------------------------------ |
| Identity Provider, or IdP          | Authenticates user and issues assertion                |
| Service Provider, or SP            | Application relying on assertion                       |
| Principal                          | User represented by assertion                          |
| Assertion Consumer Service, or ACS | SP endpoint receiving SAML Response                    |
| SSO Service                        | IdP endpoint handling authentication requests          |
| Metadata                           | Configuration describing entities, endpoints, and keys |

```mermaid
flowchart LR
    U[User]
    B[Browser]
    SP[Service Provider]
    IDP[Identity Provider]

    U --> B
    B --> SP
    SP -->|AuthnRequest through browser| IDP
    IDP -->|SAML Response through browser| SP
```

The OASIS Web Browser SSO profile defines both SP-initiated and IdP-initiated patterns and several HTTP bindings.

---

# 9.49 SAML assertion

A SAML assertion can contain statements about a subject.

Common statement types include:

* Authentication statement
* Attribute statement
* Authorization decision statement

Simplified structure:

```xml
<saml:Assertion>
  <saml:Issuer>https://idp.example.internal</saml:Issuer>

  <saml:Subject>
    <saml:NameID>alice-federated-id</saml:NameID>
  </saml:Subject>

  <saml:Conditions>
    <saml:AudienceRestriction>
      <saml:Audience>https://finance.example.internal/saml</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>

  <saml:AuthnStatement>
    ...
  </saml:AuthnStatement>

  <saml:AttributeStatement>
    ...
  </saml:AttributeStatement>
</saml:Assertion>
```

This is simplified and incomplete.

Production SAML validation must use a maintained implementation.

---

# 9.50 SAML Response versus assertion

A SAML protocol Response may contain one or more assertions.

Conceptually:

```xml
<samlp:Response>
  <saml:Issuer>...</saml:Issuer>
  <samlp:Status>...</samlp:Status>
  <saml:Assertion>
    ...
  </saml:Assertion>
</samlp:Response>
```

The Response and assertion can have separate:

* IDs
* Issuers
* Signatures
* Time conditions
* Destinations
* Validation rules

Do not assume that validating one arbitrary XML signature automatically validates the exact assertion consumed by application logic.

---

# 9.51 SAML metadata

SAML participants exchange metadata describing:

* Entity ID
* SSO endpoints
* ACS endpoints
* Supported bindings
* Signing certificates
* Encryption certificates
* Logout endpoints
* Requested attributes
* Organization information

SAML metadata standardizes information needed for federation relationships, including endpoints, identifiers, certificates, and keys.

Simplified SP metadata:

```xml
<md:EntityDescriptor entityID="https://finance.example.internal/saml">
  <md:SPSSODescriptor>
    <md:KeyDescriptor use="signing">
      ...
    </md:KeyDescriptor>

    <md:AssertionConsumerService
      Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
      Location="https://finance.example.internal/saml/acs"
      index="0"/>
  </md:SPSSODescriptor>
</md:EntityDescriptor>
```

Metadata is part of the trust configuration.

Obtaining metadata over HTTPS does not automatically prove that the metadata belongs to the intended partner.

The trust process may include:

* Manual verification
* Signed metadata
* Trusted federation aggregate
* Out-of-band fingerprint confirmation
* Controlled configuration management

---

# 9.52 SP-initiated SAML SSO flow

Concrete example:

> Alice opens the finance application. The finance SP redirects her to the corporate IdP. The IdP authenticates her and posts a SAML Response to the SP’s ACS endpoint.

```mermaid
sequenceDiagram
    actor Alice
    participant Browser
    participant SP as Finance Service Provider
    participant IdP as Corporate Identity Provider

    Alice->>Browser: Open protected finance page
    Browser->>SP: GET /reports
    SP->>SP: No local session
    SP-->>Browser: Redirect with SAMLRequest and RelayState
    Browser->>IdP: AuthnRequest
    IdP->>Alice: Authenticate if needed
    Alice->>IdP: Complete login
    IdP->>IdP: Create signed SAML Response
    IdP-->>Browser: HTML form containing SAMLResponse
    Browser->>SP: POST SAMLResponse to ACS
    SP->>SP: Validate response and assertion
    SP->>SP: Map federated subject to local user
    SP-->>Browser: Set local session cookie
    Browser->>SP: Request protected page
```

OASIS describes this common profile as an HTTP Redirect binding for the `AuthnRequest` followed by an HTTP POST binding carrying the Response to the SP’s Assertion Consumer Service.

---

# 9.53 Step 1: Service Provider saves local transaction state

Alice requests:

```http
GET /reports HTTP/1.1
Host: finance.example.internal
```

The SP has no local session.

It creates:

```json
{
  "request_id": "_saml_req_8172",
  "relay_state": "relay_71ab...",
  "original_url": "/reports",
  "created_at": "2026-07-31T10:30:00Z",
  "expires_at": "2026-07-31T10:35:00Z"
}
```

The SAML request ID will later be compared with:

```xml
InResponseTo="_saml_req_8172"
```

---

# 9.54 Step 2: Construct `AuthnRequest`

Simplified request:

```xml
<samlp:AuthnRequest
  ID="_saml_req_8172"
  Version="2.0"
  IssueInstant="2026-07-31T10:30:00Z"
  Destination="https://identity.example.internal/saml/sso"
  AssertionConsumerServiceURL="https://finance.example.internal/saml/acs">

  <saml:Issuer>
    https://finance.example.internal/saml
  </saml:Issuer>
</samlp:AuthnRequest>
```

Important fields:

| Field                         | Purpose                  |
| ----------------------------- | ------------------------ |
| `ID`                          | Correlate response       |
| `IssueInstant`                | Request time             |
| `Destination`                 | Intended IdP endpoint    |
| `AssertionConsumerServiceURL` | Intended return endpoint |
| `Issuer`                      | SP entity identifier     |

The request may be signed depending on deployment policy.

---

# 9.55 Step 3: Redirect binding

The XML request is:

1. Serialized.
2. Deflated according to binding rules.
3. Base64-encoded.
4. URL-encoded.
5. placed in `SAMLRequest`.

Conceptual redirect:

```http
HTTP/1.1 302 Found
Location: https://identity.example.internal/saml/sso?
          SAMLRequest=ENCODED_REQUEST&
          RelayState=relay_71ab...
```

`RelayState` commonly references SP-local transaction state.

It should not be treated as a trusted arbitrary redirect URL.

---

# 9.56 Step 4: IdP validates request

The IdP checks:

* SP issuer is known
* Destination is correct
* ACS is allowed
* Binding is allowed
* Request signature where required
* Request time
* Requested authentication context
* NameID policy
* User assignment
* Organizational policy

If the user lacks a suitable IdP session, the IdP authenticates them.

---

# 9.57 Step 5: IdP creates assertion

Simplified assertion:

```xml
<saml:Assertion
  ID="_assertion_91"
  Version="2.0"
  IssueInstant="2026-07-31T10:31:05Z">

  <saml:Issuer>
    https://identity.example.internal/saml
  </saml:Issuer>

  <saml:Subject>
    <saml:NameID
      Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
      federated-alice-731
    </saml:NameID>

    <saml:SubjectConfirmation
      Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">

      <saml:SubjectConfirmationData
        InResponseTo="_saml_req_8172"
        Recipient="https://finance.example.internal/saml/acs"
        NotOnOrAfter="2026-07-31T10:36:05Z"/>
    </saml:SubjectConfirmation>
  </saml:Subject>

  <saml:Conditions
    NotBefore="2026-07-31T10:30:05Z"
    NotOnOrAfter="2026-07-31T10:36:05Z">

    <saml:AudienceRestriction>
      <saml:Audience>
        https://finance.example.internal/saml
      </saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>

  <saml:AuthnStatement
    AuthnInstant="2026-07-31T10:30:55Z"
    SessionIndex="_idp_session_718">

    <saml:AuthnContext>
      <saml:AuthnContextClassRef>
        corporate-mfa
      </saml:AuthnContextClassRef>
    </saml:AuthnContext>
  </saml:AuthnStatement>

  <saml:AttributeStatement>
    <saml:Attribute Name="department">
      <saml:AttributeValue>finance</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

---

# 9.58 Step 6: Sign response or assertion

The IdP applies XML Signature according to federation configuration.

A browser-delivered SAML response requires integrity protection; the OASIS technical overview states that browser-delivered response messages containing assertions must be digitally signed for message integrity.

The SP must validate against a trusted IdP key obtained through its federation trust configuration.

It should not blindly trust a certificate embedded inside the untrusted message.

---

# 9.59 Step 7: HTTP POST binding

The IdP returns an HTML form:

```html
<form method="post"
      action="https://finance.example.internal/saml/acs">
  <input type="hidden"
         name="SAMLResponse"
         value="BASE64_ENCODED_RESPONSE">
  <input type="hidden"
         name="RelayState"
         value="relay_71ab...">
</form>

<script>
  document.forms[0].submit();
</script>
```

The browser posts:

```http
POST /saml/acs HTTP/1.1
Host: finance.example.internal
Content-Type: application/x-www-form-urlencoded

SAMLResponse=BASE64_ENCODED_RESPONSE&
RelayState=relay_71ab...
```

The browser carries the assertion but does not need to understand it.

---

# 9.60 Step 8: ACS validation

The Assertion Consumer Service must validate the exact object that the application will consume.

Checks commonly include:

1. Parse XML using secure settings.
2. Reject unexpected structure.
3. Validate Response status.
4. Validate trusted issuer.
5. Validate XML signature.
6. Confirm signed element is the consumed Response or assertion.
7. Validate `Destination`.
8. Validate `InResponseTo`.
9. Validate `Recipient`.
10. Validate `AudienceRestriction`.
11. Validate `NotBefore`.
12. Validate `NotOnOrAfter`.
13. Check assertion ID replay cache.
14. Validate authentication context where required.
15. Map NameID or attributes to local identity.
16. Apply local account policy.
17. Create a local session.

The OASIS flow explicitly places signature and assertion validation at the ACS before creation of a local login context.

---

# 9.61 `InResponseTo`

For SP-initiated SSO:

```xml
InResponseTo="_saml_req_8172"
```

binds the response to a pending authentication request.

The SP should confirm:

```text
Request ID exists
AND
belongs to current transaction
AND
has not expired
AND
has not already completed
```

IdP-initiated SSO usually lacks an SP-created request to correlate, which changes the threat model.

---

# 9.62 `Destination`

Example:

```xml
Destination="https://finance.example.internal/saml/acs"
```

The SP verifies that the message was intended for its actual receiving endpoint.

Ignoring Destination can allow assertions to be replayed or redirected into an unintended endpoint.

---

# 9.63 `Recipient`

Inside SubjectConfirmationData:

```xml
Recipient="https://finance.example.internal/saml/acs"
```

The SP verifies that the bearer assertion is being presented to its intended recipient.

---

# 9.64 `AudienceRestriction`

Example:

```xml
<saml:Audience>
  https://finance.example.internal/saml
</saml:Audience>
```

The SP must determine that it is an intended audience.

A valid IdP signature does not make an assertion intended for another SP acceptable.

This is analogous to OAuth/OIDC audience validation.

---

# 9.65 Time conditions

Example:

```xml
NotBefore="2026-07-31T10:30:05Z"
NotOnOrAfter="2026-07-31T10:36:05Z"
```

The SP checks current time with a small, deliberate skew tolerance.

Potential failures:

* SP clock ahead
* IdP clock behind
* Excessive network delay
* Timezone conversion bug
* Incorrect inclusive/exclusive boundary handling
* Excessive skew allowance

OASIS errata notes that SAML entities should allow reasonable clock skew when interpreting time-based conditions.

---

# 9.66 Replay cache

An assertion may be valid for several minutes.

Without replay detection:

```text
Attacker captures valid SAML Response
→ submits same response repeatedly
→ multiple sessions created
```

The SP can store used identifiers:

```text
Response ID
Assertion ID
expiry
```

Example:

```text
_assertion_91 used
→ reject second use
```

The replay-cache lifetime should cover the assertion validity period plus reasonable processing delay.

---

# 9.67 NameID

`NameID` represents the subject identifier.

Possible styles include:

* Persistent
* Transient
* Email-address format
* Unspecified
* Deployment-specific identifiers

A persistent pseudonymous identifier may be stable for one IdP-SP relationship without revealing the user’s internal username.

A transient identifier may last only for the session.

OASIS describes both persistent and transient federated identifiers, as well as attribute-based identity linking.

---

## Do not assume NameID is an email

Example:

```text
federated-alice-731
```

may be an opaque persistent identifier.

Hard-coding email assumptions causes:

* Failed mappings
* Account duplication
* Privacy leakage
* Broken provider migration

---

# 9.68 Attribute statements

An assertion may carry attributes:

```xml
<saml:Attribute Name="department">
  <saml:AttributeValue>finance</saml:AttributeValue>
</saml:Attribute>
```

Possible uses:

* Profile population
* Group mapping
* Role mapping
* Tenant assignment
* Authorization input
* Just-in-time provisioning

The IdP and SP must agree on:

* Attribute name
* Namespace
* Value format
* Multiplicity
* Case rules
* Source authority
* Update behavior
* Missing-value behavior

OASIS notes that attributes can support authorization, but the IdP and SP need prior agreement about attribute names and values.

---

# 9.69 Local authorization after SAML login

A signed assertion might contain:

```text
department=finance
group=financial-analysts
```

The application may map this to:

```text
local role = financial_analyst
```

The SP still needs object-level policy:

```text
May Alice read report 71?
```

SAML authentication does not automatically authorize every application resource.

---

# 9.70 IdP-initiated SSO

In IdP-initiated SSO:

```text
User starts at IdP portal
→ selects application
→ IdP sends SAML Response to SP
```

No SP `AuthnRequest` exists.

Advantages:

* Convenient application portal
* Simple launch experience

Security and operational differences:

* No `InResponseTo` correlation with an SP request
* SP must carefully restrict accepted IdP-initiated responses
* RelayState handling becomes important
* Unsolicited responses may increase login-CSRF risk
* Application cannot express request-specific authentication requirements in the same way

Prefer SP-initiated flows where request correlation and application-driven policy are important.

---

# 9.71 SAML bindings

A binding defines how SAML messages travel.

Common bindings include:

| Binding       | Data path                                                                    |
| ------------- | ---------------------------------------------------------------------------- |
| HTTP Redirect | Message encoded in URL query                                                 |
| HTTP POST     | Message in submitted form                                                    |
| HTTP Artifact | Browser carries short reference; SP retrieves assertion through back channel |
| SOAP          | Direct XML message exchange                                                  |
| PAOS          | Enhanced client/proxy reverse-SOAP pattern                                   |

The same SAML logical message can be transported using different bindings.

---

## HTTP Artifact mental model

Instead of carrying the full assertion through the browser:

```text
IdP
→ browser receives short artifact
→ browser sends artifact to SP
→ SP resolves artifact directly with IdP
```

```mermaid
sequenceDiagram
    participant Browser
    participant SP
    participant IdP

    IdP-->>Browser: Redirect with artifact
    Browser->>SP: Present artifact
    SP->>IdP: ArtifactResolve through back channel
    IdP-->>SP: SAML Response
```

Advantages:

* Full assertion stays off browser front channel
* Direct SP-IdP exchange

Costs:

* Back-channel dependency
* Additional endpoint and trust configuration
* More protocol complexity

---

# 9.72 SAML signature versus encryption

## Signature

Provides:

* Integrity
* Evidence of signing-key possession
* Authentication of message origin within configured trust

It does not hide assertion contents.

---

## Encryption

Provides confidentiality to the holder of the corresponding decryption key.

A SAML assertion may be:

* Signed
* Encrypted
* Signed then encrypted
* Protected at both message and transport layers

Encryption does not replace signature validation or semantic validation.

TLS protects each transport hop.

XML encryption can protect the assertion from the browser and intermediaries carrying the message.

---

# 9.73 XML signature complexity

XML signatures are more structurally complex than signing a flat byte string.

The signature references specific XML nodes by ID and applies transformations and canonicalization.

A secure verifier must ensure:

```text
The signed node
is the same node
the application later consumes.
```

A dangerous implementation may:

1. Find one valid signed assertion.
2. Parse user data from a different unsigned assertion.
3. Treat the overall message as valid.

This general class is associated with XML signature-wrapping attacks.

Use hardened SAML libraries that:

* Enforce expected schema and structure
* Resolve IDs safely
* Limit accepted signatures
* Return the validated assertion object directly
* Reject duplicates and ambiguity
* Disable unsafe XML features

---

# 9.74 XML parser security

Untrusted XML must be parsed defensively.

Risks include:

* External entity resolution
* Entity expansion
* Network fetches
* Excessive nesting
* Huge documents
* Duplicate IDs
* Namespace confusion
* Schema ambiguity

The SAML implementation should:

* Disable external entities
* Disable arbitrary DTD processing
* Apply size limits
* Use secure parser settings
* Avoid manual DOM traversal after separate signature validation

---

# 9.75 Inspecting a SAML Response

Assume a Base64-encoded POST-binding response is stored in:

```text
saml-response.b64
```

Decode it:

```bash
base64 --decode saml-response.b64 > saml-response.xml
```

On systems where the flag differs, consult the local `base64` manual.

## What it does

Decodes Base64 text into XML.

## Why it is useful

It allows inspection of:

* Issuer
* Subject
* Audience
* Time conditions
* Attributes
* Destination
* `InResponseTo`
* Signature placement

## Safety

A SAML Response may be an active bearer assertion.

Treat the files as credentials:

```bash
chmod 600 saml-response.b64 saml-response.xml
```

Do not upload production assertions to public decoder websites.

---

## Format XML for inspection

```bash
xmllint --format saml-response.xml
```

## What it does

Parses and formats XML for readability.

## Why it is useful

It helps inspect structure and nested elements.

## Critical limitation

Formatting does not verify:

* Signature
* Issuer
* Audience
* Time
* Recipient
* Replay
* Subject mapping

Do not treat readable XML as trusted XML.

## Safety

Some XML parsers have features that can access external resources.

Use secure tooling and untrusted-input precautions.

---

## Extract selected values

Example:

```bash
xmllint \
  --xpath 'string(//*[local-name()="Issuer"][1])' \
  saml-response.xml
```

### What it does

Prints the first element whose local name is `Issuer`.

### Why it is useful

It can assist manual inspection.

### Limitation

Using `local-name()` ignores namespaces and may select an unintended node in adversarial XML.

It must not be used as production security validation.

---

# 9.76 SAML certificate inspection

If an IdP signing certificate has been obtained through trusted metadata and stored as:

```text
idp-signing.crt
```

inspect it:

```bash
openssl x509 \
  -in idp-signing.crt \
  -noout \
  -subject \
  -issuer \
  -serial \
  -fingerprint \
  -sha256 \
  -dates
```

## What it does

Displays certificate metadata without exposing a private key.

## Important flags

* `-in`: certificate file
* `-noout`: suppress encoded certificate output
* `-subject`: display subject
* `-issuer`: display issuer
* `-serial`: display serial number
* `-fingerprint -sha256`: display SHA-256 fingerprint
* `-dates`: display validity dates

## Why it is useful

It helps compare:

* Configured certificate
* Metadata certificate
* Expected fingerprint
* Rotation dates

## Important misconception

SAML federation trust may be anchored directly in exchanged metadata rather than ordinary public web PKI semantics.

A certificate’s subject name alone does not establish that it is the expected IdP signing key.

---

# 9.77 SAML certificate rotation

The IdP may publish:

```text
Old signing key
New signing key
```

during an overlap period.

Safe rotation:

1. SP imports new verification key.
2. IdP metadata contains both keys.
3. IdP begins signing with new key.
4. SP accepts new signatures.
5. Old assertions expire.
6. Old key is removed later.

Failure pattern:

```text
IdP starts new key immediately
→ SP still trusts only old key
→ all logins fail
```

Metadata distribution and cache timing are part of authentication availability.

---

# 9.78 SAML replay and bearer assertions

Many browser SSO assertions use bearer subject confirmation.

Conceptually:

```text
Possession of valid assertion
→ may be sufficient at intended ACS
```

Therefore, protect assertions from:

* Logs
* Browser extensions
* Referrer leakage
* Debug tooling
* Packet capture outside TLS
* Shared support tickets
* Replay inside validity window

Controls include:

* TLS
* Short assertion lifetime
* Recipient validation
* Audience validation
* `InResponseTo`
* Replay cache
* Signed messages

---

# 9.79 SAML and local sessions

After successful assertion validation:

```text
SAML Response
→ federated subject
→ local user mapping
→ local session cookie
```

Later requests normally use the SP’s own session cookie.

The browser does not submit the SAML assertion on every application request.

This resembles OIDC:

```text
Federated authentication result
→ local application session
```

---

# 9.80 SAML Just-in-Time provisioning

An SP may create a local account the first time it receives a valid assertion.

Example:

```text
No local account for federated subject
→ create local user
→ map attributes
→ assign baseline access
```

Risks:

* Unexpected user population
* Incorrect tenant assignment
* Attribute spoofing through misconfigured IdP
* Excessive default role
* Duplicate account creation
* Email-based account takeover
* No deprovisioning when employment ends

JIT provisioning must define:

* Trusted issuers
* Allowed domains or organizations
* Stable identity key
* Default permissions
* Attribute mapping
* Account-disable process
* Reconciliation with directory lifecycle

---

# 9.81 SAML common failures

## Failure: signature checked against untrusted embedded key

```text
Attacker signs assertion with attacker's key
→ embeds attacker's certificate
→ SP trusts embedded key
```

The SP must anchor verification in configured federation trust.

---

## Failure: valid signature, wrong audience

```text
Assertion issued for HR application
→ replayed to finance application
```

---

## Failure: `InResponseTo` ignored

```text
Old or attacker-initiated response
→ accepted into unrelated browser transaction
```

---

## Failure: assertion replay

```text
Same Response posted repeatedly
→ multiple local sessions
```

---

## Failure: RelayState open redirect

```text
RelayState=https://attacker.example
→ SP redirects after login
```

Use opaque server-side state or strict allowlisting.

---

## Failure: attributes directly become administrator roles

```text
role=admin
→ application assigns full local administration
```

Role mapping should be:

* Issuer-specific
* Explicit
* Narrow
* Audited
* Tested

---

## Failure: email-based account linking

```text
Assertion email matches existing local administrator
→ federated account linked automatically
```

An attacker controlling a different trusted IdP or unverified email claim may take over the account.

---

## Failure: only Response signature checked, unsigned assertion consumed

The application must validate that the exact consumed assertion is protected by the accepted signature structure.

---

## Failure: excessive clock skew

```text
Accept assertions 30 minutes after expiry
→ large replay window
```

---

## Failure: IdP logout assumed to revoke local sessions

Some SP sessions may remain active after IdP logout failure or logout-message loss.

---

# 9.82 OAuth/OIDC and SAML comparison

| Property                 | OAuth 2.0                       | OpenID Connect                | SAML 2.0                               |
| ------------------------ | ------------------------------- | ----------------------------- | -------------------------------------- |
| Primary goal             | Delegated API access            | Federated authentication      | Federated assertion exchange           |
| Common data format       | Parameters, JSON, tokens        | JSON/JWT                      | XML                                    |
| Common browser flow      | Authorization Code              | Authorization Code + ID Token | Web Browser SSO                        |
| Authentication result    | Not standardized by OAuth alone | ID Token                      | Authentication assertion               |
| API access token         | Core concept                    | Inherited from OAuth          | Not primary browser-SSO model          |
| Client/application term  | Client                          | Relying Party                 | Service Provider                       |
| Identity service term    | Authorization Server            | OpenID Provider               | Identity Provider                      |
| Response recipient check | Redirect URI, audience          | Client ID audience            | ACS, Recipient, Audience               |
| Transaction binding      | State, PKCE                     | State, PKCE, nonce            | Request ID, `InResponseTo`, RelayState |
| Typical enterprise use   | API delegation                  | Modern web/mobile SSO         | Enterprise SaaS/browser SSO            |

---

# 9.83 Similar concepts across protocols

| Concept                     | OAuth/OIDC                | SAML                        |
| --------------------------- | ------------------------- | --------------------------- |
| Identity issuer             | Authorization Server / OP | IdP                         |
| Application                 | Client / RP               | SP                          |
| Federated identity result   | ID Token                  | Assertion                   |
| Intended recipient          | `aud` claim               | AudienceRestriction         |
| Transaction correlation     | `state`, `nonce`, code    | `InResponseTo`, RelayState  |
| Return endpoint             | Redirect URI              | ACS                         |
| Key discovery/configuration | Discovery + JWKS          | Metadata                    |
| Local continuity            | Application session       | SP session                  |
| Time validity               | `exp`, `nbf`, `iat`       | `NotBefore`, `NotOnOrAfter` |

The concepts are similar, but the protocols are not wire-compatible.

---

# 9.84 Where authorization occurs in federated systems

A complete access decision may involve:

```text
Identity provider:
  Is Alice allowed to authenticate?
  Is Alice assigned to the application?

Authorization server:
  May finance-web obtain reports.read?

Finance application:
  Is Alice's local account active?

Reports API:
  Does token allow reports.read?

Object policy:
  May Alice read report 71?

Database:
  Does tenant-constrained query return report 71?

Operating system:
  May reports-api process open backing file?
```

No single token replaces all these checks.

---

# 9.85 Trust-chain walkthrough

For the finance example:

```text
Finance BFF trusts:
  configured IdP issuer
  IdP signing keys
  token endpoint TLS
  OIDC library validation
  local identity mapping

Reports API trusts:
  configured authorization server
  token verification keys or introspection
  access-token audience and scope
  application authorization policy

Database trusts:
  reports-api database role
  parameterized query
  tenant context
```

A failure at any link can undermine the final result.

---

# 9.86 Federation outage behavior

Suppose the identity provider becomes unavailable.

Possible effects:

```text
New logins:
  fail

Existing local application sessions:
  may continue

Access-token refresh:
  may fail

Already valid access tokens:
  may continue until expiry

SAML local SP sessions:
  may continue

Global logout:
  may fail
```

This depends on token lifetime, session design, and runtime dependencies.

A centralized IdP outage does not necessarily terminate every existing application session immediately.

---

# 9.87 Federation performance

A login may involve:

```text
Browser redirects
+ IdP authentication
+ directory lookup
+ MFA
+ token exchange
+ signing-key lookup
+ local account mapping
+ session creation
```

Normal authenticated requests should not necessarily repeat the complete flow.

Performance strategies include:

* Local sessions
* Cached provider metadata
* Cached JWKS with bounded refresh
* Short network paths
* Reused TLS connections
* Efficient token verification
* Controlled introspection caching
* Reliable session stores

Security-sensitive caches must handle key rotation and revocation.

---

# 9.88 Federation metrics

Useful metrics include:

```text
oidc_authorization_started_total
oidc_callback_total
oidc_state_validation_failure_total
oidc_token_exchange_duration_seconds
oidc_token_exchange_failure_total
id_token_signature_failure_total
id_token_audience_failure_total
jwks_refresh_total
jwks_unknown_kid_total
saml_response_received_total
saml_signature_failure_total
saml_audience_failure_total
saml_replay_rejection_total
federated_account_mapping_failure_total
```

Avoid labels containing:

* Raw subject identifiers
* Tokens
* Authorization codes
* Session IDs
* Full email addresses
* SAML assertions

Use logs for detailed controlled investigation.

---

# 9.89 Practical troubleshooting scenarios

## Scenario A: Infinite OIDC redirect loop

Symptoms:

```text
Application redirects to IdP
IdP redirects back
Application redirects to IdP again
```

Possible causes:

* Local session cookie not set
* Browser rejects cookie
* Session store loses record
* Callback creates session on one node only
* Redirect URI differs between proxy and application
* Account mapping fails after token validation
* Cookie uses `Secure` in HTTP test environment
* SameSite behavior blocks required request
* Application sees wrong external scheme
* Session encryption/signing keys differ across instances

Debug path:

```text
Validate callback success
→ verify local session creation
→ inspect Set-Cookie
→ inspect browser cookie store
→ inspect next request Cookie
→ inspect session lookup
```

---

## Scenario B: `invalid_grant` at token endpoint

Possible causes:

* Authorization code expired
* Code already redeemed
* Redirect URI mismatch
* PKCE verifier mismatch
* Client mismatch
* Code issued by another provider
* Clock/replication issue
* Callback processed twice

---

## Scenario C: `unknown kid`

Possible causes:

* IdP rotated key
* Client JWKS cache stale
* Wrong issuer selected
* Token forged with unknown key ID
* Metadata points to wrong JWKS URI
* Multi-tenant key set mixed up
* Key not yet replicated

Do not solve by accepting any embedded key.

---

## Scenario D: ID Token valid but user mapping fails

Possible causes:

* Unknown issuer-subject pair
* Email changed
* Account not provisioned
* Tenant not assigned
* Local account disabled
* Duplicate account
* Required claim absent
* Claim type differs from expectation

---

## Scenario E: SAML response expired immediately

Possible causes:

* SP clock ahead
* IdP clock behind
* Incorrect timezone handling
* Very narrow validity window
* Queue or proxy delay
* Parsing boundary bug
* `NotOnOrAfter` treated incorrectly

---

## Scenario F: SAML signature valid but login denied

Possible causes:

* Wrong audience
* Wrong Recipient
* Wrong Destination
* `InResponseTo` not found
* Assertion replay detected
* NameID mapping absent
* Local account suspended
* Authentication context too weak
* Required attribute missing

Cryptographic validity is only one stage.

---

## Scenario G: User has wrong role after federated login

Trace:

```text
IdP directory group
→ IdP claim/attribute mapping
→ token/assertion
→ application mapping
→ local role
→ policy decision
```

Inspect each transformation.

Do not begin by manually adding permissions before finding where mapping diverged.

---

# 9.90 Common misconceptions

## “OAuth authenticates the user”

OAuth standardizes delegated authorization.

OpenID Connect supplies a standardized authentication layer.

---

## “An ID Token is an API token”

An ID Token is intended for the relying client.

An access token is intended for a resource server.

---

## “JWT validation means signature validation”

Complete validation also includes:

* Issuer
* Audience
* Time
* Token type
* Nonce
* Expected algorithm
* Subject mapping

---

## “PKCE replaces every other validation”

PKCE binds the authorization code to a verifier.

It does not replace issuer, audience, redirect URI, token, or application authorization checks.

---

## “A client secret secures a browser SPA”

A secret shipped to the browser cannot remain confidential.

---

## “SSO means all applications share one session cookie”

Usually each application has its own local session.

The shared IdP session allows them to authenticate without asking for credentials again.

---

## “A signed SAML assertion can be accepted by any SP”

The SP must verify audience, recipient, destination, time, trust, and transaction binding.

---

## “SAML attributes are automatically trustworthy permissions”

They are trusted only according to:

* Issuer trust
* Signature validation
* Attribute contract
* Local mapping
* Application policy

---

## “Logging out of the IdP terminates every application session”

Logout propagation can fail, and applications may retain local sessions or tokens.

---

## “Consent means authorization is safe”

A user may approve excessive scope or misunderstand the request.

The system must still enforce least privilege and resource-level authorization.

---

# 9.91 Questions: OAuth 2.0

1. Why is an authorization code safer to expose through the browser than a long-lived access token?
2. What exact relationship does PKCE create between the authorization request and token request?
3. Why should the redirect URI used at the token endpoint match the authorization transaction?
4. How can an authorization code be stolen but remain unusable?
5. Why is a client ID not treated as a secret?
6. Why can a server-side BFF be a confidential client while a browser SPA cannot?
7. What failure occurs if an authorization server accepts prefix-matched redirect URIs?
8. How could an open redirect on an otherwise trusted client domain leak an authorization code?
9. Why is `state` insufficient if it is random but not bound to a browser transaction?
10. What security property does `nonce` add to OpenID Connect?
11. Why should a resource server validate `aud` even when the access-token signature is valid?
12. Why can a token with `reports.read` still be denied access to report 71?
13. How does the client-credentials grant differ from a user-delegated grant?
14. Why must a resource server distinguish service subjects from human subjects?
15. What is the availability tradeoff between locally validated JWT access tokens and introspected opaque tokens?
16. How can refresh-token rotation distinguish theft from ordinary reuse, and where can concurrency create false alarms?
17. Why does a short-lived access token reduce but not eliminate revocation delay?
18. What information might leak if the callback URL is written to reverse-proxy logs?
19. Why does placing a client secret in minified JavaScript provide no meaningful protection?
20. How would you debug an authorization code that succeeds on one application instance but fails on another?

---

# 9.92 Questions: OpenID Connect

1. Why should local identity be keyed by the pair `(issuer, subject)`?
2. What account-takeover risk appears when applications automatically link users by email?
3. Why is decoding an ID Token not equivalent to validating it?
4. What does the ID Token audience represent?
5. Why should an API normally reject an ID Token in its `Authorization` header?
6. How can an application use `auth_time` to implement step-up authentication?
7. What must happen when the provider rotates to a new signing key?
8. How could accepting a token from the wrong issuer cause cross-tenant identity confusion?
9. Why might the UserInfo `sub` need to be compared with the ID Token `sub`?
10. What remains locally controlled after authentication is delegated to an OpenID Provider?
11. Why can deleting the application session fail to log the user out of the identity provider?
12. Which failures could create an infinite redirect loop after an apparently successful ID Token validation?
13. How should an application behave when the ID Token is valid but the local account is suspended?
14. Why should authentication methods such as `amr` not be interpreted without an agreed provider contract?
15. What new risk appears when discovery is performed using an attacker-controlled issuer URL?

---

# 9.93 Questions: SAML

1. Why must the Service Provider validate both the signature and the assertion’s semantic conditions?
2. What does `InResponseTo` bind, and why is it usually absent in IdP-initiated SSO?
3. How do Destination, Recipient, and Audience represent different checks?
4. Why can a valid assertion intended for the HR application be unsafe at the finance application?
5. How does a replay cache reduce risk even when assertions have short expiry times?
6. Why should the SP not trust a certificate merely because it is embedded in the SAML Response?
7. What problem occurs when application logic consumes a different assertion node from the one covered by the validated XML signature?
8. Why can RelayState become an open-redirect vulnerability?
9. How does HTTP Artifact binding change which components see the full assertion?
10. What are the operational consequences of an IdP signing-key rotation performed without overlap?
11. How can a valid SAML login still result in a local authorization denial?
12. Why is a persistent NameID generally safer as an account key than a mutable email attribute?
13. What can happen if connection or system clocks differ by several minutes?
14. Why does XML parsing itself belong inside the SAML security boundary?
15. How could Just-in-Time provisioning assign a new user to the wrong tenant?
16. Why may IdP logout fail to terminate an already active SP session?
17. How would you trace an incorrect role from the source directory through the final application policy?
18. What differences in transaction assurance exist between SP-initiated and IdP-initiated SSO?
19. Why is SAML encryption not a substitute for signature and audience validation?
20. What should an SP do when a required authorization attribute is absent from an otherwise valid assertion?

# 10. Operating-System Authentication and Authorization

Application authentication answers questions such as:

```text
Which application user sent this HTTP request?
```

Operating-system authentication and authorization answer questions such as:

```text
Which local account started this process?
May this process open this file?
May it bind this port?
May it signal another process?
May it become another user?
```

The kernel does not normally understand application identities such as:

```text
alice@example.com
tenant-42
financial_analyst
```

It works with operating-system security identities such as:

* User IDs
* Group IDs
* Security tokens
* Capabilities
* Access-control lists
* Mandatory security labels
* Namespaces
* Credentials attached to processes

---

## 10.1 Simplified mental model

On a Unix-like system:

```text
Login name
→ account lookup
→ credential verification
→ account policy
→ user and group IDs
→ process credentials
→ kernel access checks
```

```mermaid
flowchart LR
    U[User enters login]
    A[Account lookup]
    V[Credential verification]
    P[Account policy]
    C[Process credentials]
    K[Kernel authorization]
    R[File, process, socket, device]

    U --> A --> V --> P --> C --> K --> R
```

The important transition is:

```text
Human proves account identity
→ operating system creates a process carrying security credentials
```

After login, most filesystem and process authorization does not repeatedly verify the password.

The kernel evaluates the credentials attached to the calling process.

---

## 10.2 Exact implementation varies

The exact flow differs across:

* Linux distributions
* BSD systems
* macOS
* Windows
* Local versus directory-backed accounts
* Console login versus SSH
* Graphical login versus service startup
* Containers versus host processes
* Filesystem implementations
* Mandatory access-control frameworks

Examples:

| Environment           | Common identity mechanisms                                        |
| --------------------- | ----------------------------------------------------------------- |
| Linux local login     | NSS, PAM, `/etc/passwd`, `/etc/shadow`                            |
| Linux directory login | PAM plus LDAP, SSSD, Kerberos, or another provider                |
| SSH                   | SSH protocol plus host/user authentication and optional PAM       |
| Windows domain        | Active Directory, Kerberos, NTLM, access tokens, ACLs             |
| macOS                 | OpenDirectory, authorization services, keychain, sandbox controls |
| Containers            | Host kernel credentials plus namespaces, capabilities, LSM rules  |

The shared mental model is more stable than any one configuration file.

---

# 10.3 Accounts, users, and numeric IDs

Unix-like systems generally use numeric identifiers internally.

Important values include:

* UID: user identifier
* GID: group identifier
* Supplementary group IDs
* Process capabilities
* Security labels

Example:

```text
Login name: alice
UID: 1001
Primary GID: 1001
Supplementary groups: finance, developers
```

The name `alice` is mainly a human-readable mapping.

The kernel commonly reasons about:

```text
UID 1001
GID 1001
GID 2001
GID 2007
```

---

## 10.4 Why numeric IDs exist

Names can:

* Change
* Be duplicated across identity sources
* Be unavailable during directory outages
* Have different normalization rules
* Be expensive to resolve repeatedly

Numeric IDs provide compact values suitable for process credentials and filesystem metadata.

A file may store:

```text
owner UID = 1001
group GID = 2001
```

A command such as `ls` resolves those IDs back to names for display.

---

## 10.5 Identity mapping failure

Suppose a file is owned by UID `1001`.

Later:

1. Account `alice` is deleted.
2. UID `1001` becomes unassigned.
3. A new account `bob` is created with UID `1001`.

The file now appears owned by Bob.

The filesystem stored the numeric ID, not the historical identity meaning.

This is why UID and GID reuse can create security problems.

---

## 10.6 Inspect current process identity

```bash
id
```

### What it does

Displays the user and group identities associated with the current process.

### Why it is useful

It helps diagnose authorization involving:

* File permissions
* Group access
* `sudo`
* Network filesystems
* Container user mappings
* Service processes

### Illustrative output

```text
uid=1001(alice) gid=1001(alice) groups=1001(alice),2001(finance),2007(developers)
```

### Output meaning

* `uid=1001(alice)`: current user identity
* `gid=1001(alice)`: primary group
* `groups=...`: primary and supplementary groups

### Safety

This command is read-only.

It may reveal internal account and group names when shared externally.

---

## 10.7 Inspect another account

```bash
id alice
```

### What it does

Resolves account information for `alice`.

### Why it is useful

It verifies whether:

* The account exists
* NSS can resolve it
* Group membership is visible
* Directory-backed identity lookup is working

### Important limitation

The output describes current identity-source data.

An already-running process belonging to Alice may still carry an older supplementary-group list.

---

# 10.8 Name Service Switch

On many Linux systems, the **Name Service Switch**, or NSS, determines where account and group information comes from.

Possible sources include:

* Local files
* LDAP
* SSSD
* DNS
* System databases
* Container-specific mechanisms
* Other configured modules

Conceptual lookup:

```text
get user alice
→ check NSS configuration
→ query configured sources in order
→ return account record
```

---

## 10.9 `/etc/passwd`

A local account record may appear in `/etc/passwd`.

Inspect safely:

```bash
getent passwd alice
```

### What it does

Queries the system’s configured account databases for `alice`.

### Why `getent` is useful

It follows NSS configuration.

This is usually more informative than reading only:

```text
/etc/passwd
```

because the account may come from a directory service.

### Illustrative output

```text
alice:x:1001:1001:Alice Example:/home/alice:/bin/bash
```

### Common fields

```text
login name
password placeholder
UID
primary GID
comment or GECOS field
home directory
login shell
```

### Important misconception

The `x` field does not mean the password is literally `x`.

It usually indicates that password-verification data is stored elsewhere, commonly in a shadow database.

---

## 10.10 Inspect all visible identity sources carefully

```bash
getent passwd
```

### What it does

Lists accounts visible through configured NSS sources.

### Why it is useful

It can show whether:

* Directory accounts are visible
* Local and remote identities overlap
* Unexpected service accounts exist

### Safety and performance

On systems with large or remote directories, enumerating all accounts may be slow, restricted, or unsupported.

Prefer a targeted lookup when possible:

```bash
getent passwd alice
```

---

# 10.11 NSS is not authentication

NSS answers questions such as:

```text
Does account alice exist?
What UID maps to alice?
What groups contain alice?
```

It does not necessarily verify credentials.

A common separation is:

```text
NSS:
  identity lookup

PAM:
  authentication and account/session policy
```

A system may successfully resolve Alice through NSS but still reject her login through PAM.

Conversely, a specialized authentication system might verify a user but fail to map them to a usable local UID.

---

# 10.12 PAM mental model

PAM allows an application to request authentication-related services through a configured module stack.

Applications might include:

* `login`
* `sshd`
* `sudo`
* Graphical display managers
* Screen lockers
* FTP servers
* VPN software
* Administrative tools

```mermaid
flowchart LR
    A[Application]
    L[libpam]
    S[PAM service stack]
    M1[Authentication module]
    M2[Account module]
    M3[Session module]
    D[Local or remote identity source]

    A --> L --> S
    S --> M1
    S --> M2
    S --> M3
    M1 --> D
    M2 --> D
```

The application commonly asks PAM:

```text
Authenticate this user
Check whether this account may log in
Change authentication token
Open or close a session
```

---

## 10.13 PAM service configuration

PAM configuration is commonly selected by service name.

Examples:

```text
/etc/pam.d/sshd
/etc/pam.d/sudo
/etc/pam.d/login
```

The exact location and included configuration vary by distribution.

A stack contains module rules for management groups such as:

```text
auth
account
password
session
```

---

## 10.14 PAM management groups

| Group      | Main responsibility                                                       |
| ---------- | ------------------------------------------------------------------------- |
| `auth`     | Verify authentication evidence and establish authentication-related state |
| `account`  | Check whether the account may access the service                          |
| `password` | Change authentication tokens                                              |
| `session`  | Perform session setup and teardown                                        |

A successful `auth` stage does not guarantee a successful login.

Example:

```text
Password correct
but
account expired
→ account stage denies access
```

---

## 10.15 PAM control logic

A PAM stack may include control rules such as:

* Required
* Requisite
* Sufficient
* Optional
* Extended control syntax

These determine how module results affect the final result.

Simplified interpretation:

### Required

The module must succeed, but later modules may still run before failure is returned.

### Requisite

The module must succeed; failure may stop the stack immediately.

### Sufficient

Success may complete the stack if no earlier required module has failed.

### Optional

Result usually matters only when no stronger module determines the outcome.

Exact behavior can become complex when modules are included from other stacks or extended control syntax is used.

Do not edit PAM rules without a recovery path.

A single configuration mistake can block all administrative login.

---

# 10.16 Concrete Linux SSH login flow

We will trace:

> Alice connects to a Linux server using SSH public-key authentication, then runs a command with `sudo`.

Components:

```text
Alice's SSH client
Network stack
OpenSSH server
NSS
Optional PAM stack
Kernel
User shell
sudo
sudo policy
Target command
Filesystem and process authorization
```

```mermaid
flowchart LR
    C[SSH Client]
    N[Network and TLS-like SSH transport]
    S[sshd]
    NSS[NSS account lookup]
    PAM[PAM account/session checks]
    K[Kernel process creation]
    SH[Alice shell]
    SU[sudo]
    CMD[Privileged command]

    C --> N --> S
    S --> NSS
    S --> PAM
    S --> K --> SH --> SU --> CMD
```

SSH does not use TLS.

It uses its own transport, key exchange, encryption, integrity, host authentication, and user-authentication protocols.

---

# 10.17 Stage 1: TCP connection

Alice runs:

```bash
ssh alice@server.example.internal
```

### What it does

Starts an SSH client and requests a connection to the remote host.

### Command components

* `ssh`: OpenSSH client
* `alice@`: requested remote account name
* `server.example.internal`: destination host

### Default port

SSH normally uses TCP port 22 unless configured otherwise.

### Safety

The remote hostname must be trusted.

A successful TCP connection does not prove the server identity.

---

## 10.18 Observe connection setup

```bash
ssh -v alice@server.example.internal
```

### What it does

Enables verbose SSH diagnostic output.

### Why it is useful

It shows stages such as:

* Configuration selection
* Hostname resolution
* TCP connection
* Host-key negotiation
* Key exchange
* Authentication methods
* Public keys attempted

### Important flags

* `-v`: verbose
* `-vv`: more verbose
* `-vvv`: maximum routine diagnostic verbosity

### Safety

Verbose output may reveal:

* Usernames
* Hostnames
* Local key paths
* Authentication methods
* Network information

It should not normally print private-key contents, but redact diagnostic logs before sharing.

---

# 10.19 Stage 2: SSH host authentication

Before authenticating Alice, the client authenticates the server.

The server proves possession of a private host key.

The client checks the corresponding host key against:

* A previously stored known-host entry
* A host-certificate authority
* Administrator-provided configuration
* A manual fingerprint decision

Simplified flow:

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: Begin SSH transport
    Server-->>Client: Host public key and signed handshake data
    Client->>Client: Verify signature
    Client->>Client: Compare host key with trust configuration
```

This protects against connecting securely to the wrong server.

---

## 10.20 Known-host verification

The client may consult:

```text
~/.ssh/known_hosts
```

Inspect matching entries:

```bash
ssh-keygen -F server.example.internal
```

### What it does

Searches known-host files for entries matching the hostname.

### Why it is useful

It helps diagnose:

* Missing host key
* Changed host key
* Multiple key types
* Hashed hostname entries

### Important flag

* `-F`: find a host in known-hosts data

### Safety

Do not remove a changed host-key warning merely to make the connection work.

A host-key change may indicate:

* Legitimate server rebuild
* Host-key rotation
* DNS error
* Connecting to a different server
* Man-in-the-middle attack

Verify the new fingerprint through a trusted channel.

---

## 10.21 Host-key warning

Illustrative diagnostic:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

This means:

```text
The host key presented now
does not match
the trusted key stored earlier
```

It does not automatically mean an attack.

It does mean the trust relationship has changed and needs investigation.

---

# 10.22 Stage 3: SSH key exchange and encrypted channel

The client and server negotiate:

* Key-exchange algorithm
* Host-key algorithm
* Encryption algorithm
* Integrity protection
* Compression behavior where used

They derive session keys.

After key exchange:

```text
SSH protocol messages
→ encrypted and integrity-protected
```

Server host authentication and user authentication are separate stages.

---

# 10.23 Stage 4: Account lookup

Alice requests account:

```text
alice
```

`sshd` needs to determine:

* Does this account exist?
* What UID and GID does it map to?
* What is the home directory?
* Which shell is configured?
* Is the account permitted by SSH policy?

It may use NSS for account lookup.

Conceptual result:

```text
username = alice
UID = 1001
primary GID = 1001
home = /home/alice
shell = /bin/bash
```

If account lookup fails, SSH cannot create a normal local user session even if some external key identity was otherwise valid.

---

# 10.24 Stage 5: SSH public-key authentication

Alice’s client holds a private key.

The server has or can obtain the corresponding authorized public key.

Common local file:

```text
/home/alice/.ssh/authorized_keys
```

Simplified flow:

1. Client offers a public-key identity.
2. Server checks whether that key is authorized for Alice.
3. Server sends or verifies authentication data tied to the current SSH exchange.
4. Client signs using the private key.
5. Server verifies the signature using the public key.
6. Private key never crosses the network.

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Keys as Authorized Key Source

    Client->>Server: Offer public key
    Server->>Keys: Is key authorized for Alice?
    Keys-->>Server: Authorized
    Client->>Client: Sign SSH authentication data
    Client->>Server: Signature
    Server->>Server: Verify signature with public key
```

---

## 10.25 SSH private key versus public key

| Material         | Storage                     | Security role        |
| ---------------- | --------------------------- | -------------------- |
| Private key      | Alice’s device or agent     | Produces signatures  |
| Public key       | Server authorization source | Verifies signatures  |
| Host private key | Server                      | Authenticates server |
| Host public key  | Client trust store          | Verifies server      |

The user private key must remain secret.

The public key is not a password substitute that needs secrecy.

---

# 10.26 Inspect an SSH public-key fingerprint

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

### What it does

Displays the fingerprint of a public key.

### Important flags

* `-l`: show fingerprint
* `-f FILE`: read key from file

### Why it is useful

Fingerprints provide a shorter identifier for comparing keys.

### Illustrative output

```text
256 SHA256:REDACTED alice@laptop (ED25519)
```

### Safety

A public-key fingerprint is not normally secret.

Still, it can reveal account or device metadata.

Do not run this command on a private-key path when a public-key path is available.

---

# 10.27 `authorized_keys` restrictions

An authorized-key entry can include restrictions.

Conceptual example:

```text
from="192.0.2.0/24",no-agent-forwarding,no-port-forwarding ssh-ed25519 AAAA...
```

Possible restrictions include:

* Source network
* Forced command
* Agent forwarding
* Port forwarding
* PTY allocation
* Environment options
* User-presence or certificate constraints depending on mechanism

This allows a single key to have less than full interactive-shell authority.

Example:

```text
Backup key
→ may execute only backup command
→ no shell
→ no port forwarding
```

---

## 10.28 Common `authorized_keys` failures

* Incorrect file owner
* File or directory permissions too broad
* Wrong home directory
* Key line wrapped or damaged
* Wrong public key
* Account lookup failure
* Mandatory access-control denial
* `sshd_config` disables public-key authentication
* Different authorized-key source configured
* Certificate authority expected instead of raw key
* Source restriction does not match
* Forced command fails
* Network-mounted home unavailable

---

# 10.29 Stage 6: PAM account and session checks

Depending on server configuration, `sshd` may call PAM even when public-key authentication succeeds.

Possible PAM responsibilities:

* Account expiry
* Login-time restrictions
* Access-control rules
* MFA
* Session limits
* Home-directory setup
* Resource limits
* Audit session
* Environment setup

Therefore:

```text
SSH signature valid
≠
login necessarily allowed
```

Possible flow:

```text
Public-key authentication succeeds
→ PAM account check fails
→ no shell session
```

---

# 10.30 Stage 7: Determine user and group credentials

Before creating Alice’s shell, the system establishes credentials such as:

```text
UID = 1001
effective UID = 1001
primary GID = 1001
supplementary groups = [2001, 2007]
```

The group list may come from:

* Local group files
* Directory service
* SSSD
* Cached identity data
* Login-time group initialization

Already-running processes generally keep their existing credentials until explicitly changed.

---

## 10.31 Why group changes may appear delayed

Suppose an administrator adds Alice to group `finance`.

Directory state becomes:

```text
alice ∈ finance
```

Alice’s existing shell still carries:

```text
supplementary groups = old list
```

A new login may be required so the new process receives an updated group list.

Check current shell:

```bash
id
```

Check current directory view:

```bash
id alice
```

If they differ, the running process may have stale credentials.

---

# 10.32 Stage 8: Process creation

After successful authentication and account policy:

1. `sshd` creates or forks a session process.
2. It sets group credentials.
3. It sets the target UID.
4. It applies limits and session setup.
5. It changes directory where appropriate.
6. It starts the configured shell or command.
7. The kernel attaches credentials to the new process.

Simplified process state:

```text
PID = 21871
real UID = 1001
effective UID = 1001
saved UID = 1001
real GID = 1001
effective GID = 1001
supplementary groups = [2001, 2007]
```

---

# 10.33 Real, effective, and saved user IDs

Unix-like processes may carry multiple user identifiers.

## Real UID

Usually identifies the account that started the process.

## Effective UID

Commonly used for permission checks.

## Saved set-user-ID

May allow a privileged executable to temporarily drop and later regain an effective identity.

Exact semantics vary by system call and operating system.

---

## 10.34 Why several IDs exist

Consider a privileged utility that needs to:

1. Start with elevated authority.
2. Perform a restricted setup operation.
3. Drop privileges.
4. Perform most work as the calling user.
5. Regain limited authority only when necessary.

Multiple IDs support controlled transitions.

Improper identity transitions can create privilege-escalation vulnerabilities.

---

# 10.35 Inspect process credentials

```bash
grep -E '^(Uid|Gid|Groups):' /proc/self/status
```

### What it does

Reads credential-related fields for the current process on Linux.

### Why it is useful

It shows numeric:

* Real IDs
* Effective IDs
* Saved IDs
* Filesystem IDs
* Supplementary groups

### Illustrative output

```text
Uid:	1001	1001	1001	1001
Gid:	1001	1001	1001	1001
Groups:	1001 2001 2007
```

### Linux-specific behavior

`/proc` is a Linux interface.

Other operating systems expose process credentials differently.

### Safety

This is read-only.

Process information visibility may be restricted by system policy.

---

# 10.36 File access mental model

A process requests:

```text
open("/srv/reports/q2.pdf", O_RDONLY)
```

The kernel evaluates whether the process credentials permit the operation.

Simplified decision order for traditional mode bits:

1. If process effective UID matches file owner, use owner bits.
2. Otherwise, if process belongs to file group, use group bits.
3. Otherwise, use other bits.
4. Apply additional ACL, capability, mandatory-control, mount, and filesystem rules.
5. Allow or deny.

The kernel does not combine owner, group, and other mode bits to find the most permissive set.

It selects the applicable class.

---

# 10.37 Traditional permission bits

Example:

```text
-rw-r-----
```

Breakdown:

```text
-    file type
rw-  owner permissions
r--  group permissions
---  other permissions
```

Numeric form:

```text
0640
```

| Bit     | Value |
| ------- | ----: |
| Read    |     4 |
| Write   |     2 |
| Execute |     1 |

Thus:

```text
owner = 6 = read + write
group = 4 = read
other = 0
```

---

## 10.38 Directory permissions differ from file permissions

For a regular file:

| Permission | Meaning                      |
| ---------- | ---------------------------- |
| Read       | Read file data               |
| Write      | Modify file data             |
| Execute    | Execute file where supported |

For a directory:

| Permission | Meaning                                                   |
| ---------- | --------------------------------------------------------- |
| Read       | List directory entries                                    |
| Write      | Create, remove, or rename entries, subject to other rules |
| Execute    | Traverse/search the directory                             |

A user may know a filename but be unable to access it because a parent directory lacks execute permission.

---

## 10.39 Path traversal authorization

Opening:

```text
/srv/reports/finance/q2.pdf
```

requires search permission on each directory component:

```text
/
→ /srv
→ /srv/reports
→ /srv/reports/finance
→ q2.pdf
```

A readable file can remain inaccessible because one directory in the path blocks traversal.

---

# 10.40 Inspect every path component

```bash
namei -l /srv/reports/finance/q2.pdf
```

### What it does

Displays each component of the pathname with ownership and permission information.

### Why it is useful

It detects failures where:

* Final file permissions look correct
* A parent directory denies traversal
* A symbolic link changes the path
* Ownership differs unexpectedly

### Illustrative output

```text
f: /srv/reports/finance/q2.pdf
drwxr-xr-x root        root     /
drwxr-xr-x root        root     srv
drwxr-x--- finance-app finance  reports
drwxr-x--- finance-app finance  finance
-rw-r----- finance-app finance  q2.pdf
```

### Safety

This is read-only.

It may expose sensitive filesystem structure.

`namei` availability varies across Unix-like systems.

---

# 10.41 File ownership and group

Inspect:

```bash
stat /srv/reports/finance/q2.pdf
```

### What it does

Displays detailed filesystem metadata.

### Why it is useful

It can show:

* Numeric and named owner
* Group
* Mode
* File type
* Size
* Timestamps
* Inode
* Filesystem metadata

### Illustrative fields

```text
Access: (0640/-rw-r-----)
Uid: ( 1005/finance-app)
Gid: ( 2001/finance)
```

### Important limitation

`stat` does not necessarily show:

* Extended ACL details
* SELinux policy outcome
* AppArmor policy
* Network filesystem server decisions
* Encryption policy
* All mount restrictions

---

# 10.42 Changing mode bits

```bash
chmod 640 /srv/reports/finance/q2.pdf
```

### What it does

Sets traditional mode permissions to:

```text
owner: read/write
group: read
other: none
```

### Why it is useful

It adjusts basic discretionary access.

### Safety

This changes authorization.

An incorrect mode can:

* Expose sensitive data
* Block the application
* Break deployment
* Interact unexpectedly with ACL masks

Inspect the current state and intended ownership first.

---

# 10.43 Changing ownership

```bash
sudo chown finance-app:finance /srv/reports/finance/q2.pdf
```

### What it does

Sets:

```text
owner = finance-app
group = finance
```

### Why it is useful

Ownership determines which permission class applies.

### Safety

Changing ownership may:

* Change access immediately
* Break service expectations
* Affect backup behavior
* Interact with set-user-ID or set-group-ID bits
* Be restricted on network filesystems

Use only when the intended security model is known.

---

# 10.44 Creation permissions and `umask`

A process requests a creation mode, and the `umask` removes permissions.

Conceptually:

```text
requested mode
AND NOT umask
→ initial mode
```

Example:

```text
requested file mode = 0666
umask = 0027
result = 0640
```

Inspect current mask:

```bash
umask
```

Illustrative output:

```text
0027
```

### Why it is useful

It explains why newly created files do not receive the full requested permissions.

### Important misconception

`umask` does not add permissions.

It removes selected permission bits from the requested creation mode.

---

## 10.45 Test creation behavior

```bash
umask 0027
touch test-file
stat -c '%A %a %n' test-file
```

### What it does

* Sets the shell’s mask
* Creates an empty file
* Shows symbolic and numeric mode

### Illustrative output

```text
-rw-r----- 640 test-file
```

### Safety

Changing `umask` affects files subsequently created by that shell and child processes.

Use a temporary directory for testing.

The `stat -c` syntax shown is common on GNU/Linux and differs on some other systems.

---

# 10.46 Special mode bits

Traditional Unix mode bits also include:

* Set-user-ID
* Set-group-ID
* Sticky bit

---

## 10.47 Set-user-ID

On an executable file, set-user-ID can cause the process effective UID to become the file owner’s UID.

Conceptual example:

```text
Executable owner = root
set-user-ID set
ordinary user executes it
→ effective UID may become root
```

This is highly security-sensitive.

A vulnerability in such a program may become a privilege-escalation vulnerability.

Interpreters and filesystems may apply restrictions, and exact behavior varies.

---

## 10.48 Set-group-ID

On an executable, it can affect effective group identity.

On a directory, it commonly causes newly created entries to inherit the directory’s group.

Example:

```text
/srv/team-project owned by group developers
set-group-ID on directory
→ new files inherit developers group
```

Inspect:

```bash
ls -ld /srv/team-project
```

Illustrative output:

```text
drwxrws--- 4 root developers 4096 Jul 31 12:00 /srv/team-project
```

The `s` in the group execute position indicates set-group-ID.

---

## 10.49 Sticky bit

On a shared writable directory, the sticky bit restricts removal or renaming of entries.

Typical example:

```text
/tmp
```

Illustrative mode:

```text
drwxrwxrwt
```

Without the sticky bit, users with directory write access could remove each other’s files more freely.

---

# 10.50 POSIX ACLs

Traditional mode bits provide one owner, one group class, and one other class.

ACLs allow additional entries.

Example:

```text
owner finance-app: read/write
group finance: read
user auditor: read
others: none
```

Inspect:

```bash
getfacl /srv/reports/finance/q2.pdf
```

Illustrative output:

```text
# file: srv/reports/finance/q2.pdf
# owner: finance-app
# group: finance
user::rw-
user:auditor:r--
group::r--
mask::r--
other::---
```

---

## 10.51 ACL mask

The ACL mask limits effective permissions for:

* Named users other than the owner
* Named groups
* Owning group entry

Example:

```text
user:auditor:rw-
mask::r--
```

Effective permission for `auditor` becomes:

```text
r--
```

not:

```text
rw-
```

This is a common source of confusion.

---

## 10.52 Set a file ACL

```bash
sudo setfacl -m u:auditor:r-- /srv/reports/finance/q2.pdf
```

### What it does

Adds or modifies a named-user ACL entry.

### Important arguments

* `-m`: modify ACL
* `u:auditor:r--`: grant user `auditor` read permission

### Why it is useful

It grants access without:

* Changing file owner
* Adding the user to the main file group
* Exposing access to all others

### Safety

This changes authorization immediately.

Inspect the resulting mask:

```bash
getfacl /srv/reports/finance/q2.pdf
```

---

## 10.53 Default directory ACLs

A directory can have default ACL entries inherited by new children.

Inspect:

```bash
getfacl /srv/reports/finance
```

Possible entries:

```text
default:user::rwx
default:group::r-x
default:group:auditors:r-x
default:mask::r-x
default:other::---
```

Default ACLs affect newly created entries.

They do not necessarily retroactively update existing files.

---

# 10.54 Kernel file-access walkthrough

Suppose:

```text
Process:
  effective UID = 1001
  supplementary groups = [2001]

File:
  owner UID = 1005
  group GID = 2001
  mode = 0640
```

Alice calls:

```c
open("/srv/reports/finance/q2.pdf", O_RDONLY)
```

Simplified kernel reasoning:

1. Resolve pathname.
2. Verify search permission on each parent directory.
3. Locate final inode.
4. Determine requested access: read.
5. Effective UID does not equal owner UID.
6. Process belongs to file group `2001`.
7. Apply group permission bits: `r--`.
8. Evaluate ACLs if present.
9. Evaluate mandatory security controls.
10. Evaluate filesystem and mount restrictions.
11. Allow file descriptor creation.

Result:

```text
open() returns a non-negative file descriptor
```

---

## 10.55 Failed access

If group bits are:

```text
---
```

and no ACL grants access:

```text
open() = -1
errno = EACCES
```

The application may see:

```text
Permission denied
```

The exact denial might come from:

* Directory traversal
* File mode
* ACL
* Mandatory access control
* Network filesystem server
* Filesystem policy

The final error alone may not identify the rejecting layer.

---

# 10.56 Observe failed system calls with `strace`

```bash
strace -e trace=openat,read,write \
  command-that-reads-report
```

### What it does

On Linux, traces selected system calls made by a process.

### Important arguments

* `-e trace=...`: limit tracing to selected calls
* `openat`: commonly used to open files
* `read`, `write`: data-transfer calls

### Illustrative output

```text
openat(AT_FDCWD, "/srv/reports/finance/q2.pdf", O_RDONLY) = -1 EACCES (Permission denied)
```

### Why it is useful

It confirms:

* Exact pathname
* Requested flags
* Kernel error
* Whether the application reached the filesystem operation

### Safety

Tracing may expose:

* File paths
* Arguments
* Secrets
* Data
* Internal implementation details

Tracing another process may require elevated privileges and can affect timing.

Use only on systems and processes you are authorized to inspect.

---

## 10.57 Trace a running process

```bash
sudo strace -p 21871 -e trace=file
```

### What it does

Attaches to process `21871` and traces file-related system calls.

### Important flags

* `-p PID`: attach to existing process
* `-e trace=file`: select file-related calls

### Safety

Attaching may:

* Pause the target briefly
* Affect performance
* Expose sensitive paths
* Be blocked by security policy
* Require elevated privileges

Avoid using it casually on latency-sensitive production services.

---

# 10.58 `sudo` mental model

`sudo` allows an authorized user to execute a command under another identity, commonly root.

It performs several distinct steps:

```text
Caller identity
→ sudo policy lookup
→ optional authentication
→ environment and command validation
→ credential transition
→ execute target command
→ audit
```

Authentication and authorization are separate:

```text
Password correct
but
sudo policy denies command
→ command does not run
```

---

# 10.59 Concrete `sudo` flow

Alice runs:

```bash
sudo systemctl restart reports-api
```

Possible steps:

1. Shell starts `/usr/bin/sudo`.
2. `sudo` identifies Alice from process credentials.
3. `sudo` reads configured policy.
4. Policy determines whether Alice may run the command.
5. PAM may authenticate Alice.
6. `sudo` validates command path and arguments.
7. It prepares a controlled environment.
8. It changes process credentials.
9. It executes `systemctl`.
10. Logs record the attempt and result.

```mermaid
sequenceDiagram
    participant Shell
    participant Sudo
    participant Policy
    participant PAM
    participant Kernel
    participant Command

    Shell->>Sudo: sudo systemctl restart reports-api
    Sudo->>Policy: May Alice run this command?
    Policy-->>Sudo: Allowed, authentication required
    Sudo->>PAM: Authenticate Alice
    PAM-->>Sudo: Success
    Sudo->>Kernel: Change credentials and execute
    Kernel->>Command: Start privileged command
```

---

## 10.60 `sudo` commonly authenticates the caller

When `sudo` prompts:

```text
[sudo] password for alice:
```

it generally verifies Alice’s credential.

It does not usually ask for the target account’s password.

This answers:

```text
Is the person controlling Alice's session still able to authenticate as Alice?
```

Authorization still comes from sudo policy.

---

# 10.61 Inspect allowed sudo operations

```bash
sudo -l
```

### What it does

Lists commands the current user may run through `sudo`.

### Why it is useful

It reveals:

* Allowed target users
* Allowed commands
* Password requirements
* Environment restrictions
* Policy aliases as displayed
* Host-specific rules

### Important flag

* `-l`: list privileges

### Safety

This is primarily an inspection operation, though policy may require authentication.

Its output exposes privilege structure and should not be shared unnecessarily.

---

## 10.62 Inspect another user’s sudo policy

```bash
sudo -l -U alice
```

### What it does

Requests policy information for user `alice`.

### Important flags

* `-l`: list
* `-U alice`: inspect specified user

### Safety

This normally requires administrative privilege.

Policy output may reveal sensitive administrative paths.

---

# 10.63 Edit sudo policy safely

Use:

```bash
sudo visudo
```

### What it does

Edits sudo policy using locking and syntax validation.

### Why it is useful

Directly editing the main policy file risks introducing syntax errors that can block administrative access.

### Safety

This is a highly sensitive operation.

Before changing policy:

* Maintain another administrative session.
* Verify an emergency access path.
* Prefer a test host.
* Grant the narrowest necessary command.
* Avoid broad wildcard rules.
* Validate included files.

---

## 10.64 Dangerous sudo rule

Conceptual rule:

```text
alice ALL=(root) NOPASSWD: ALL
```

This grants Alice unrestricted root command execution without reauthentication.

A narrower rule might allow one controlled operation.

However, command-level restrictions are difficult when the allowed command can:

* Execute a shell
* Load arbitrary configuration
* Read attacker-controlled files
* Invoke plugins
* Set environment variables
* Write executable code
* Escape to another command
* Control a service definition

The full behavior of the permitted program matters.

---

# 10.65 Command arguments matter

Suppose policy allows:

```text
/usr/bin/systemctl restart reports-api
```

A broad pattern such as:

```text
/usr/bin/systemctl *
```

may allow operations far beyond the intended service.

Possible unintended actions:

* Start another service
* Edit or link units indirectly
* Trigger privileged helpers
* Access environment or status data
* Operate on attacker-controlled unit names

Authorization should consider both executable and permitted arguments.

---

# 10.66 Environment variables and privilege

A privileged command may change behavior based on variables such as:

* `PATH`
* Library loading variables
* Language-runtime module paths
* Editor configuration
* Configuration-file locations
* Proxy settings
* Temporary-directory paths

`sudo` normally applies environment controls.

Dangerous configuration can allow an attacker to influence which code a privileged process loads or executes.

A privileged program should use:

* Absolute paths where appropriate
* Controlled search paths
* Trusted configuration
* Secure temporary-file handling
* Minimal inherited environment

---

# 10.67 `sudo` timestamp

After successful authentication, `sudo` may cache the authentication result temporarily.

Conceptual flow:

```text
First sudo command
→ password prompt
→ timestamp established

Second sudo command shortly later
→ no password prompt
```

This is not a new operating-system login.

It is cached authorization-related authentication state managed by `sudo`.

---

## 10.68 Invalidate cached `sudo` authentication

```bash
sudo -k
```

### What it does

Invalidates the current user’s cached `sudo` authentication timestamp.

### Why it is useful

The next `sudo` operation should require authentication again where policy requires it.

### Important flag

* `-k`: invalidate cached credentials

### Safety

This does not remove the user’s sudo authorization.

It invalidates cached authentication state.

---

# 10.69 Effective identity after `sudo`

A privileged command may run with:

```text
real UID = 0 or caller-dependent
effective UID = 0
```

Exact identity transitions depend on implementation.

Inspect inside a root command:

```bash
sudo id
```

Illustrative output:

```text
uid=0(root) gid=0(root) groups=0(root)
```

This shows the command’s operating-system identity, not merely the identity of the terminal user.

---

# 10.70 Set-user-ID versus `sudo`

| Mechanism                     | Authority decision                                      |
| ----------------------------- | ------------------------------------------------------- |
| Set-user-ID executable        | Kernel applies executable file mode and owner           |
| `sudo`                        | Privileged sudo program applies configurable policy     |
| Capability-enabled executable | Kernel grants specified capability set under rules      |
| Service manager               | Manager starts configured process as specified identity |

`sudo` generally provides more explicit command policy and auditing than granting arbitrary set-user-ID executables.

---

# 10.71 Linux capabilities

Traditional Unix privilege treats UID 0 as broadly privileged.

Linux capabilities divide some root authority into separate units.

Examples include capabilities related to:

* Binding privileged ports
* Changing file ownership
* Sending signals across identity boundaries
* Loading kernel modules
* Changing process IDs
* Network administration
* Raw sockets

Conceptual model:

```text
Instead of full root authority
→ process receives selected privilege bits
```

---

## 10.72 Why capabilities exist

A network server may need to bind port 80.

Traditional approach:

```text
Run entire process as root
```

Capability-oriented approach:

```text
Grant only authority needed to bind a privileged port
→ run remaining process without full root identity
```

This supports least privilege.

It does not guarantee safety because some individual capabilities are themselves very powerful.

---

# 10.73 Process capability sets

Linux processes may have capability sets such as:

* Permitted
* Effective
* Inheritable
* Bounding
* Ambient

A simplified model:

| Set         | Role                                                         |
| ----------- | ------------------------------------------------------------ |
| Permitted   | Capabilities process may make effective                      |
| Effective   | Capabilities currently used for checks                       |
| Inheritable | Capabilities considered across executable transition         |
| Bounding    | Upper limit on capabilities obtainable                       |
| Ambient     | Capabilities preserved across some non-privileged executions |

Exact transition rules are complex.

Do not design security based only on a simplified capability summary.

---

## 10.74 Inspect process capabilities

```bash
grep '^Cap' /proc/self/status
```

Illustrative output:

```text
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	000001ffffffffff
CapAmb:	0000000000000000
```

These values are hexadecimal bitmasks.

Decode with:

```bash
capsh --decode=0000000000000000
```

### What `capsh` does

Provides capability-related inspection and testing functions.

### Important flag

* `--decode=MASK`: translate hexadecimal capability mask

### Availability

`capsh` is not installed on every Linux system.

---

## 10.75 Inspect file capabilities

```bash
getcap /usr/bin/ping
```

### What it does

Displays file capabilities attached to the executable.

### Illustrative output

```text
/usr/bin/ping cap_net_raw=ep
```

Meaning, simplified:

```text
Executable may obtain CAP_NET_RAW
in permitted and effective sets
```

Exact output varies by distribution and packaging.

### Safety

Inspection is read-only.

File capabilities are security-sensitive metadata.

---

## 10.76 Set a file capability

```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/example-server
```

### What it does

Allows the executable to gain the capability used for binding low-numbered network ports.

### Safety

This changes executable privilege.

Risks include:

* Any exploitable code path may use the capability.
* Replacing the executable may alter or remove metadata.
* Backup and deployment tools may not preserve capability attributes.
* Scripts do not behave like ordinary native executables for file capabilities.
* The program may execute other code with unexpected privilege transitions.

Prefer service-manager capability controls where available because they are easier to review and bind to a service configuration.

---

# 10.77 Service-manager identity

A service manager such as systemd can start a service under a specific account.

Illustrative unit fragment:

```ini
[Service]
User=reports-api
Group=reports-api
ExecStart=/usr/local/bin/reports-api
```

This means the service process is intended to run with that user and group identity.

Additional controls may restrict:

* Capabilities
* Filesystem paths
* Devices
* Network families
* Namespaces
* System calls
* Privilege escalation

---

## 10.78 Inspect service identity

```bash
systemctl show reports-api \
  -p User \
  -p Group \
  -p DynamicUser \
  -p NoNewPrivileges
```

### What it does

Displays selected service properties.

### Important options

* `show`: display unit properties
* `-p`: select properties

### Why it is useful

It verifies the intended service identity and selected hardening controls.

### Safety

This is read-only.

Unit configuration and property names vary by service manager and system version.

---

## 10.79 Inspect running service process

```bash
systemctl status reports-api
```

Then inspect its PID:

```bash
ps -o pid,user,group,comm -p 21871
```

### What `ps` does

Displays selected process fields.

### Important options

* `-o`: choose output columns
* `-p PID`: select process

### Illustrative output

```text
PID   USER         GROUP        COMMAND
21871 reports-api  reports-api  reports-api
```

### Why it is useful

Configured identity and actual running identity can differ if:

* Service was started manually
* Unit overrides exist
* Privilege transitions occur after startup
* Wrapper scripts alter execution
* An old process predates configuration changes

---

# 10.80 `NoNewPrivileges`

On Linux, the no-new-privileges process attribute prevents certain executable transitions from granting additional privilege.

Conceptual effect:

```text
Process executes set-user-ID or capability-marked program
→ no additional privilege gained through that execution
```

It is useful for containing child processes.

It does not remove authority already held by the process.

---

# 10.81 Mandatory Access Control

Traditional Unix permissions are largely discretionary.

A user with sufficient ownership or privilege may change them.

Mandatory Access Control adds centrally enforced policy.

Linux commonly supports mandatory controls through Linux Security Modules such as:

* SELinux
* AppArmor
* Other configured security modules

These can deny an operation even when:

```text
UID/GID/mode checks would allow it
```

---

## 10.82 Combined authorization

A process may need all of the following to succeed:

```text
Traditional mode or ACL allows
AND
mandatory policy allows
AND
mount/filesystem policy allows
AND
capability or privilege rules allow
```

A denial at any layer blocks the operation.

```mermaid
flowchart TD
    R[System call]
    D{DAC and ACL allow?}
    M{Mandatory policy allow?}
    F{Filesystem and mount allow?}
    P{Privilege constraints satisfied?}
    A[Allow]
    X[Deny]

    R --> D
    D -->|No| X
    D -->|Yes| M
    M -->|No| X
    M -->|Yes| F
    F -->|No| X
    F -->|Yes| P
    P -->|No| X
    P -->|Yes| A
```

---

# 10.83 SELinux mental model

SELinux assigns security contexts to:

* Processes
* Files
* Sockets
* Ports
* Other kernel objects

Conceptual process context:

```text
system_u:system_r:httpd_t:s0
```

Conceptual file context:

```text
system_u:object_r:httpd_sys_content_t:s0
```

Policy evaluates:

```text
May process type httpd_t
read
file type httpd_sys_content_t?
```

The decision is based on labels and policy, not only path names.

---

## 10.84 Inspect SELinux mode

```bash
getenforce
```

### What it does

Shows the current SELinux enforcement state where SELinux is installed.

Possible outputs include:

```text
Enforcing
Permissive
Disabled
```

### Meaning

* `Enforcing`: policy denials are enforced.
* `Permissive`: denials are logged but generally not enforced.
* `Disabled`: SELinux is not active.

### Safety

Inspection is read-only.

Do not switch a production system to permissive merely to make an error disappear.

That removes an important security layer and hides the real policy mismatch.

---

## 10.85 Inspect SELinux context

```bash
ls -Z /srv/reports/finance/q2.pdf
```

### What it does

Displays the file’s SELinux context where supported.

Illustrative output:

```text
system_u:object_r:reports_data_t:s0 /srv/reports/finance/q2.pdf
```

Inspect a process:

```bash
ps -eZ | grep reports-api
```

### Why it is useful

A process may have correct Unix ownership but the wrong SELinux type relationship.

---

## 10.86 SELinux denial symptoms

Application log:

```text
open failed: Permission denied
```

Traditional permissions:

```text
appear correct
```

SELinux audit log:

```text
avc: denied { read } ...
```

The denial indicates mandatory policy blocked the operation.

The exact audit format varies.

---

## 10.87 Search recent SELinux denials

A commonly available inspection command is:

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
```

### What it does

Searches audit records for recent SELinux access-vector-cache denials.

### Important arguments

* `-m AVC,USER_AVC`: select denial-related message types
* `-ts recent`: limit to recent records

### Why it is useful

It identifies:

* Source process context
* Target context
* Requested operation
* Object class
* Process name
* File or resource information

### Safety

Audit logs contain sensitive process and path data.

Do not automatically convert every denial into a new allow rule.

A denial may reveal:

* Incorrect file labeling
* Compromised process behavior
* Wrong deployment path
* Overly broad application access attempt

---

# 10.88 AppArmor mental model

AppArmor commonly applies profiles associated with executable paths and process confinement.

A profile may restrict:

* File access
* Network access
* Capabilities
* Signal operations
* Execution transitions
* Mount operations

Simplified policy question:

```text
May process under profile reports-api
read /srv/reports/**?
```

AppArmor and SELinux use different policy models and tooling.

Do not assume commands or logs from one apply to the other.

---

## 10.89 Inspect AppArmor status

```bash
sudo aa-status
```

### What it does

Displays loaded AppArmor profiles and enforcement state where the tooling is installed.

### Why it is useful

It shows:

* Whether AppArmor is active
* Which profiles are loaded
* Which are enforcing
* Which are in complain mode
* Which processes are confined

### Safety

Inspection is read-only.

Complain mode logs violations without enforcing them and should not be used as a permanent substitute for correct policy.

---

# 10.90 Seccomp

Seccomp restricts which system calls a process may make.

Conceptual policy:

```text
reports-api may call:
  read
  write
  openat
  accept
  epoll_wait

reports-api may not call:
  mount
  ptrace
  reboot
```

Seccomp is not user authentication.

It is process confinement and attack-surface reduction.

A process can be correctly authenticated and authorized at the application level but terminated because it attempts a forbidden system call.

---

# 10.91 Namespaces and identity

Linux namespaces isolate views of system resources.

Relevant examples include:

* User namespaces
* Mount namespaces
* PID namespaces
* Network namespaces
* IPC namespaces
* UTS namespaces

A user namespace can map IDs differently inside and outside a container.

Example:

```text
Inside container UID 0
→ mapped to host UID 100000
```

Thus:

```text
root inside container
≠
necessarily host root
```

The exact risk depends on namespace configuration, kernel security, capabilities, mounts, and runtime policy.

---

## 10.92 Inspect namespace-related identity

Inside a Linux process:

```bash
cat /proc/self/uid_map
```

Illustrative output:

```text
         0     100000      65536
```

Simplified interpretation:

```text
Container UID range beginning at 0
maps to
host UID range beginning at 100000
```

### Safety

This is read-only.

Mappings are Linux- and namespace-specific.

---

# 10.93 Container process identity

A container process still executes under the host kernel.

The kernel evaluates:

* Host-side mapped UID/GID
* Namespace membership
* Capabilities
* Seccomp
* LSM policy
* Mount permissions
* Cgroup restrictions
* Device permissions

The container image’s `/etc/passwd` supplies names inside the container, but kernel enforcement ultimately uses numeric credentials and namespace mappings.

---

## 10.94 Common container permission failure

Container process:

```text
inside UID = 1000
```

Mounted host directory:

```text
owner host UID = 2000
mode = 0700
```

No matching ID mapping exists.

Result:

```text
container process cannot access mounted directory
```

Changing a username inside the image does not necessarily fix the numeric ownership mismatch.

---

# 10.95 Windows access-token mental model

Windows uses a different security model.

After authentication, a process commonly receives an access token containing information such as:

* User security identifier, or SID
* Group SIDs
* Privileges
* Integrity level
* Restricted SIDs
* Token type
* Elevation information

Objects may have security descriptors containing:

* Owner
* Discretionary ACL
* System ACL
* Integrity information

The access check conceptually evaluates:

```text
Process or thread access token
against
object security descriptor
```

This is analogous in purpose to Unix process credentials and file authorization, but not identical in implementation.

---

## 10.96 Windows authentication packages

Depending on environment, Windows authentication may involve:

* Kerberos
* NTLM
* Local account authentication
* Smart cards
* Certificates
* Windows Hello
* Cloud identity integration

Domain authentication and local authorization can be separate.

Example:

```text
Domain authenticates Alice
→ Windows creates local logon session and access token
→ file server evaluates ACL
```

---

## 10.97 User Account Control

User Account Control, or UAC, helps separate ordinary administrative activity from elevated execution.

An administrator account may operate with a filtered token until elevation occurs.

A consent or credential prompt may create a process with a higher-privilege token.

UAC is not equivalent to Unix `sudo`, though both address controlled privilege elevation.

---

# 10.98 Kerberos preview

Kerberos commonly supports centralized network authentication.

Simplified model:

```text
User authenticates to Key Distribution Center
→ receives Ticket-Granting Ticket
→ requests service ticket
→ presents service ticket to target service
```

The target service does not need to receive the user’s password.

```mermaid
sequenceDiagram
    participant U as User Client
    participant KDC as Kerberos KDC
    participant S as Service

    U->>KDC: Initial authentication
    KDC-->>U: Ticket-Granting Ticket
    U->>KDC: Request service ticket
    KDC-->>U: Service ticket
    U->>S: Present service ticket
    S->>S: Verify ticket
```

Kerberos will be examined further with infrastructure and service identities.

---

# 10.99 Authentication versus process authorization

Suppose Alice logs into Linux successfully.

That authentication result creates a user process.

Later, Alice’s process tries to read:

```text
/etc/shadow
```

The kernel does not ask Alice for her password again.

It evaluates:

```text
process credentials
against
file security policy
```

Possible decision:

```text
authenticated Alice
but
UID 1001 lacks access
→ deny
```

This is the operating-system equivalent of:

```text
Valid web session
but
user lacks permission for report
```

---

# 10.100 Complete SSH-to-file flow

Concrete operation:

> Alice connects through SSH and reads a finance report.

```mermaid
sequenceDiagram
    actor Alice
    participant Client as SSH Client
    participant SSHD as SSH Server
    participant NSS
    participant PAM
    participant Kernel
    participant Shell
    participant File as Report File

    Alice->>Client: ssh alice@server
    Client->>SSHD: Establish SSH transport
    SSHD-->>Client: Prove host identity
    Client->>Client: Validate host key

    Client->>SSHD: Request user alice
    SSHD->>NSS: Resolve alice
    NSS-->>SSHD: UID, GID, home, shell

    Client->>SSHD: Public-key proof
    SSHD->>SSHD: Verify authorized key and signature
    SSHD->>PAM: Account and session checks
    PAM-->>SSHD: Allow

    SSHD->>Kernel: Create process with Alice's credentials
    Kernel-->>Shell: Start shell as UID 1001

    Alice->>Shell: cat /srv/reports/finance/q2.pdf
    Shell->>Kernel: execve cat
    Kernel-->>Shell: Start cat as Alice
    Shell->>Kernel: openat report file
    Kernel->>Kernel: Check path, UID/GID, ACL, LSM, mount policy
    Kernel-->>Shell: File descriptor or EACCES
    Shell->>File: read if allowed
```

---

# 10.101 Data transformations

| Stage               | Input                        | Output                                 |
| ------------------- | ---------------------------- | -------------------------------------- |
| Host authentication | Server host-key proof        | Trusted encrypted SSH connection       |
| Account lookup      | Login name `alice`           | UID, GID, home, shell                  |
| User authentication | SSH public-key signature     | Authenticated SSH identity             |
| Account policy      | Account and service context  | Allow or deny login                    |
| Session creation    | Account identity             | Process credentials                    |
| Command execution   | Shell command                | New process with inherited credentials |
| File open           | Process credentials and path | File descriptor or denial              |
| Read                | File descriptor              | File bytes                             |

---

# 10.102 Troubleshooting scenario: SSH key accepted but login closes

Symptoms:

```text
Public-key authentication appears successful
Connection closes before shell appears
```

Possible causes:

* PAM account denial
* Expired account
* Invalid login shell
* Missing home directory
* Session module failure
* Resource limit
* Forced command exits
* Shell startup file exits
* SELinux or AppArmor denial
* Filesystem unavailable
* User process limit reached
* SSH configuration restricts the user

Debug order:

```text
Client verbose output
→ sshd logs
→ account lookup
→ shell validity
→ PAM account/session logs
→ mandatory-access logs
→ home-directory access
```

---

# 10.103 Troubleshooting scenario: group added but access still denied

State:

```text
Administrator adds Alice to finance group
Directory lookup shows membership
Alice's existing shell cannot read finance file
```

Likely explanation:

```text
Existing process credentials still contain old supplementary groups
```

Check:

```bash
id
```

and compare:

```bash
id alice
```

Resolution commonly requires a new login session.

Avoid using commands that create confusing nested shells without understanding how they initialize groups.

---

# 10.104 Troubleshooting scenario: root can read, service cannot

Symptoms:

```text
sudo cat file succeeds
reports-api gets Permission denied
```

Possible causes:

* File owner/group mismatch
* Parent directory traversal denial
* ACL mask
* SELinux type mismatch
* AppArmor profile
* Container UID mapping
* Read-only mount
* Network filesystem identity mapping
* Service running as unexpected user
* Service manager sandbox restriction

Debug path:

```text
Identify actual service PID
→ inspect UID/GID/groups
→ inspect path components
→ inspect ACL
→ inspect mandatory-control logs
→ inspect mount and namespace context
```

Do not conclude that the application is broken merely because root can access the resource.

Root bypasses or satisfies many controls that ordinary service processes do not.

---

# 10.105 Troubleshooting scenario: `sudo` password succeeds but command denied

Possible output:

```text
alice is not allowed to run '/usr/bin/systemctl restart reports-api' as root
```

Interpretation:

```text
Authentication may have succeeded
but
sudo authorization policy denied the operation
```

Inspect:

```bash
sudo -l
```

Check:

* Exact command path
* Exact arguments
* Target user
* Host rule
* Included policy files
* Group membership
* Whether policy change reached this host

---

# 10.106 Troubleshooting scenario: permission denied despite mode `0777`

World-readable and writable mode does not guarantee access.

Possible remaining controls:

* Parent directory traversal
* SELinux
* AppArmor
* Read-only mount
* Network filesystem export policy
* Immutable file attribute
* Container namespace mismatch
* Seccomp for related operations
* Service-manager path restrictions
* Filesystem-specific policy

Setting `0777` is not a valid general debugging solution.

It may expose data without fixing the real cause.

---

# 10.107 Observe process tree and identities

```bash
ps -eo pid,ppid,user,group,comm --forest
```

### What it does

Displays a process tree with identity columns on systems supporting these options.

### Why it is useful

It helps trace:

```text
service manager
→ wrapper
→ application
→ worker
→ helper command
```

A helper may run under a different identity from the main service.

### Safety

Process listings reveal internal software and account names.

---

# 10.108 Audit authentication and authorization

Useful events include:

```text
SSH connection accepted
SSH user authentication succeeded
PAM account allowed
Session opened
sudo command requested
sudo policy allowed or denied
Process executed
File access denied
Mandatory policy denied
Session closed
```

A single user action may create records across:

* SSH logs
* PAM logs
* sudo logs
* Audit subsystem
* Service manager
* Kernel audit
* Application logs

Correlation requires:

* Time synchronization
* Process IDs
* Session identifiers
* User IDs
* Source address
* Executable path
* Command
* Audit session ID

---

# 10.109 Linux audit inspection

Where Linux Audit is configured, a targeted search may use:

```bash
sudo ausearch -ua alice -ts today
```

### What it does

Searches audit events associated with user `alice` from today.

### Important arguments

* `-ua alice`: filter by user account
* `-ts today`: start time

### Why it is useful

It may correlate:

* Authentication
* Process execution
* Privilege changes
* Access denials
* Session events

### Safety

Audit records can contain sensitive command and path information.

Exact event coverage depends on configured audit rules.

---

# 10.110 Security considerations

## Minimize interactive root use

Prefer:

* Specific service identities
* Narrow sudo rules
* Audited automation
* Capability restriction
* Mandatory confinement
* Separate administrative accounts

---

## Avoid shared accounts

Shared accounts reduce accountability.

If five administrators all log in as:

```text
admin
```

logs cannot easily identify the human actor.

Prefer:

```text
individual identity
→ controlled elevation
→ audited privileged action
```

---

## Protect service accounts

Service accounts should generally have:

* No unnecessary interactive shell
* Restricted home directory
* Minimal groups
* Narrow filesystem permissions
* No reusable human password where avoidable
* Managed credentials
* Explicit service-manager configuration

---

## Separate authentication from elevation

Human login:

```text
Alice authenticates as Alice
```

Administrative operation:

```text
Alice receives narrowly controlled elevated authority
```

Logging directly in as root removes useful identity continuity.

---

## Preserve actor identity

An elevated command should ideally retain audit context:

```text
actor = Alice
effective UID = root
command = systemctl restart reports-api
```

Effective identity alone is insufficient for accountability.

---

## Use least privilege across layers

An application should have only the authority required at each layer:

```text
Linux user:
  reports-api

Filesystem:
  read reports, write temporary output

Database:
  read report metadata

Network:
  connect only to required services

Capabilities:
  none unless required

Sudo:
  no access

Mandatory policy:
  narrow application profile
```

---

# 10.111 Performance considerations

Operating-system authorization is usually designed to be fast, but several operations can add latency.

Examples:

* Remote NSS lookup
* Large nested group expansion
* Network-mounted home directory
* Directory-service timeout
* PAM module network call
* Mandatory-policy audit pressure
* Excessive filesystem ACL complexity
* Network filesystem authorization
* DNS reverse lookup during login
* Home-directory auto-mount

---

## 10.112 Identity lookup caching

Tools such as directory clients may cache:

* User records
* Group records
* Authentication state
* Offline credentials
* Negative lookup results

Benefits:

* Faster lookup
* Continued login during directory outage
* Lower directory load

Risks:

* Stale group membership
* Delayed account disablement
* UID mapping inconsistency
* Offline access after central revocation
* Difficult cache invalidation

The acceptable cache lifetime depends on the required revocation speed.

---

# 10.113 Simplified model versus exact kernel reality

## Simplified model

```text
User logs in
→ process gets UID and groups
→ kernel checks file mode
```

## More exact reality

```text
Application protocol authenticates user
→ NSS resolves account
→ PAM modules run
→ supplementary groups initialized
→ process credentials created
→ namespaces may remap IDs
→ capability sets applied
→ seccomp may restrict system calls
→ path components resolved
→ mount namespace selects filesystem
→ DAC and ACL evaluated
→ mandatory security module evaluated
→ filesystem-specific and network-server policy evaluated
→ operation allowed or denied
```

The simplified model is useful for learning.

The exact model is required when:

* Permissions appear correct but access fails
* Containers are involved
* Directory identities are involved
* Mandatory access control is active
* Privilege transitions occur
* Network filesystems are involved
* Auditability matters

---

# 10.114 Questions: Operating-System Authentication and Authorization

1. Why does the kernel generally use numeric UIDs instead of account names for filesystem ownership?
2. How can reusing a deleted user’s UID give a new account access to old files?
3. What is the difference between NSS identity lookup and PAM authentication?
4. Why can an account resolve successfully through `getent` while login still fails?
5. How can public-key authentication succeed while PAM still denies the SSH session?
6. Why may adding a user to a group not affect an already-running shell?
7. Which process credential normally matters most for traditional Unix permission checks?
8. Why does a readable file remain inaccessible when a parent directory lacks execute permission?
9. How does the ACL mask change the effective permissions of a named-user entry?
10. What security problem occurs when an application runs as root only to bind a privileged port?
11. How do Linux capabilities reduce privilege, and why can an individual capability still be dangerous?
12. What is the difference between authenticating through `sudo` and being authorized by sudo policy?
13. Why can a broad sudo command wildcard produce privilege escalation even when the executable path is fixed?
14. What information should an audit record preserve when Alice runs a command with effective UID 0?
15. Why can mode `0777` fail to overcome a SELinux or AppArmor denial?
16. What should you inspect when root can read a file but a service process cannot?
17. How can a container process running as UID 0 remain unprivileged relative to the host?
18. Why does changing the username inside a container not necessarily fix mounted-volume permissions?
19. How can a stale NSS or directory cache delay account revocation?
20. What is the difference between discretionary file permissions and mandatory access-control policy?
21. Why must the exact process label and target object label be inspected in an SELinux denial?
22. How can a service-manager configuration say `User=reports-api` while the running process has another identity?
23. What does `NoNewPrivileges` prevent, and what existing authority does it leave unchanged?
24. Why does a host-key warning occur before user authentication begins?
25. How would you distinguish an SSH host-authentication failure from a user-authentication failure?
26. Why should a changed SSH host key be verified rather than automatically deleted from `known_hosts`?
27. What data does a public-key SSH signature bind to the current connection?
28. Why is the private key never required on the SSH server?
29. How can an `authorized_keys` forced command reduce the authority of a deployment key?
30. Why might a forced command still be unsafe if it accepts attacker-controlled arguments?
31. How can a set-user-ID executable regain privilege after temporarily dropping it?
32. What TOCTOU problems can occur when a privileged program checks a path and later opens it?
33. Why can network filesystem authorization differ from local mode-bit expectations?
34. What evidence would show that `openat()` reached the kernel and failed with `EACCES`?
35. How would you investigate a login that succeeds through password verification but immediately closes during session setup?
36. What is the relationship between a Windows access token and a Unix process credential?
37. Why is UAC elevation not identical to authenticating a new Windows user?
38. Which operating-system controls reduce damage after an application-level authorization bypass?
39. Why is a service account with no interactive password still an identity that requires authorization controls?
40. How can centralized identity improve account management while creating login latency and outage dependencies?

# 11. Infrastructure Identities and Service-to-Service Authentication

Human authentication begins with questions such as:

```text
Which employee is using this browser?
Which administrator opened this SSH session?
```

Infrastructure authentication begins with questions such as:

```text
Which workload opened this network connection?
Which service submitted this queue message?
Which deployment is requesting a cloud credential?
Which process is connecting to this database?
```

The subject is no longer necessarily a human.

It may be:

* A process
* Virtual machine
* Container
* Kubernetes Pod
* Serverless function
* CI/CD job
* Hardware device
* Service account
* Application instance
* Cluster node
* Control-plane component

---

## 11.1 Core mental model

A workload identity system performs this transformation:

```text
Runtime properties
→ verified workload identity
→ short-lived credential
→ authenticated connection or request
→ authorization decision
```

```mermaid
flowchart LR
    W[Workload]
    B[Bootstrap Evidence]
    I[Identity Issuer]
    C[Credential]
    S[Target Service]
    P[Authorization Policy]

    W -->|Presents runtime evidence| I
    I -->|Issues credential| C
    W -->|Credential + request| S
    S -->|Identity and request| P
    P -->|Allow or deny| S
```

The difficult part is usually not producing a certificate or token.

The difficult part is answering:

> Why should the issuer believe that this process is the intended workload?

This is the **workload identity bootstrap problem**.

---

## 11.2 Workload identity versus service account

A **workload identity** is the identity assigned to a running computational workload.

Example:

```text
production payments API
```

A **service account** is usually a named, non-human principal used by software.

Example:

```text
payments-api-service-account
```

The terms overlap, but they are not identical.

A service account may exist when no workload is running.

A workload may receive a dynamically generated identity without using a traditional long-lived service-account password.

---

## 11.3 Identity is not the credential

Consider:

```text
Identity:
  orders-service

Credential:
  X.509 certificate valid for 30 minutes
```

or:

```text
Identity:
  Kubernetes ServiceAccount shop/orders

Credential:
  projected signed bearer token
```

or:

```text
Identity:
  cloud role reports-reader

Credential:
  temporary access key, secret, and session token
```

Credentials can rotate while the logical identity stays stable.

```mermaid
flowchart TD
    I[Stable workload identity]
    C1[Credential generation 1]
    C2[Credential generation 2]
    C3[Credential generation 3]

    I --> C1
    I --> C2
    I --> C3
```

This separation allows:

* Short credential lifetimes
* Rotation without changing policy
* Better compromise containment
* Central workload authorization
* Elimination of many permanent secrets

---

# 11.4 Why static service credentials are difficult

A traditional deployment might store:

```text
DATABASE_PASSWORD=...
API_KEY=...
CLOUD_SECRET_KEY=...
```

The application reads these values at startup.

This is operationally simple but creates significant lifecycle problems.

---

## Static credential lifecycle

```text
Generate secret
→ copy into secret store
→ inject into workload
→ use for months or years
→ manually rotate
→ find every remaining copy
```

Copies may remain in:

* Environment variables
* Deployment manifests
* CI/CD logs
* Container layers
* Shell history
* Crash dumps
* Process memory
* Backups
* Developer machines
* Configuration repositories
* Old runtime instances

---

## Shared secret ambiguity

Suppose every instance of `orders-service` uses the same API key.

The target service learns:

```text
Credential belongs to orders-service
```

It may not learn:

```text
Which cluster?
Which Pod?
Which deployment version?
Which runtime instance?
Was the credential copied elsewhere?
```

A shared credential weakens instance-level accountability.

---

## Rotation problem

If credential A is replaced by credential B:

```text
Issue B
→ update all clients
→ allow A and B temporarily
→ verify all clients use B
→ revoke A
```

Failures include:

* One client still uses A
* A stopped client restarts with old configuration
* New deployment receives B while rollback receives A
* Server revokes A too early
* Both credentials remain accepted indefinitely

---

## Secret zero

A workload that retrieves secrets from a secret manager must first authenticate to the secret manager.

This first credential is often called:

```text
secret zero
```

Example:

```text
Application needs database password
→ must authenticate to secret manager
→ what credential authenticates that first request?
```

Workload identity systems try to bootstrap from runtime properties rather than another manually distributed permanent secret.

---

## Questions: Static Credentials

1. Why does moving a password from a configuration file to a secret manager not fully solve the bootstrap problem?
2. How does sharing one API key across 500 workload instances reduce audit quality?
3. Why can overlapping credentials simplify rotation while accidentally creating permanent privilege?
4. What should happen to a credential injected into an old deployment after a rollback?
5. Which copies of a secret might survive after the primary secret-store entry is deleted?
6. Why is a long random service password still operationally risky despite being resistant to guessing?

---

# 11.5 Main workload-authentication mechanisms

| Mechanism                 | Evidence presented                               | Common use                      |
| ------------------------- | ------------------------------------------------ | ------------------------------- |
| Static secret             | Password or API key                              | Legacy integrations             |
| Signed bearer token       | JWT or opaque token                              | APIs and control planes         |
| X.509 certificate         | Certificate plus private-key proof               | mTLS                            |
| Cloud instance identity   | Provider-issued signed evidence                  | VM and workload federation      |
| Kubernetes ServiceAccount | Signed ServiceAccount token                      | Kubernetes API and federation   |
| Kerberos service ticket   | Ticket issued by KDC                             | Enterprise network services     |
| SSH host or user key      | Public-key signature                             | Administration and automation   |
| Hardware-backed key       | TPM/HSM signature                                | Device and node identity        |
| Unix peer credentials     | Kernel-provided process identity                 | Local sockets                   |
| SPIFFE SVID               | X.509, JWT, or another defined identity document | Heterogeneous workload identity |

A real system may use several mechanisms.

Example:

```text
Kubernetes ServiceAccount token
→ exchanged for cloud temporary credential
→ used to request database certificate
→ certificate used for mTLS
```

---

# 11.6 Authentication versus transport encryption

Two services can communicate over encrypted TLS without authenticating the client workload.

Typical HTTPS:

```text
Client verifies server
Server does not necessarily verify client identity
```

Mutual TLS:

```text
Client verifies server
Server verifies client
```

TLS 1.3 supports certificate-based client authentication: the server can send a `CertificateRequest`, and the client can return a certificate and proof of private-key possession. The server can reject the connection if acceptable client authentication is required but not supplied.

---

## One-way TLS

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: Begin TLS handshake
    Server-->>Client: Server certificate
    Client->>Client: Validate server identity
    Client->>Server: Establish encrypted channel
```

The server knows:

```text
A network client established this connection
```

It does not necessarily know which workload it is.

The client may later authenticate using:

* Bearer token
* API key
* Password
* Signed request
* Application protocol

---

## Mutual TLS

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: Begin TLS handshake
    Server-->>Client: Server certificate + client certificate request
    Client->>Client: Validate server certificate
    Client->>Server: Client certificate + signature proof
    Server->>Server: Validate client certificate and proof
    Client->>Server: Establish mutually authenticated channel
```

The TLS layer can produce:

```text
Peer certificate identity
```

The application must still map that identity to permissions.

---

# 11.7 X.509 certificate mental model

An X.509 certificate is a signed binding between:

```text
Identity information
+
Public key
+
Validity and usage constraints
```

A simplified certificate:

```text
Subject identity:
  payments.internal

Public key:
  PK_payments

Issuer:
  Internal Workload CA

Validity:
  10:00 to 10:30

Extensions:
  server authentication
  DNS name payments.internal

Signature:
  created by Internal Workload CA
```

The certificate is public.

The private key corresponding to the public key must remain secret.

---

## Certificate proof

A peer does not authenticate merely by sending a certificate.

It must prove possession of the corresponding private key by signing handshake data.

```text
Certificate says:
  public key belongs to orders-service

Handshake signature proves:
  current peer possesses matching private key
```

---

## Certificate anatomy

Important certificate fields and extensions include:

| Field                    | Purpose                                |
| ------------------------ | -------------------------------------- |
| Serial number            | Issuer-specific certificate identifier |
| Issuer                   | Entity that signed certificate         |
| Subject                  | Named certificate subject              |
| Subject Alternative Name | Identities bound to key                |
| Public key               | Key used to verify peer signatures     |
| Not Before               | Earliest validity time                 |
| Not After                | Expiry time                            |
| Basic Constraints        | Whether certificate can act as a CA    |
| Key Usage                | Allowed cryptographic operations       |
| Extended Key Usage       | Intended high-level uses               |
| Authority Key Identifier | Helps identify issuer key              |
| Subject Key Identifier   | Identifies subject public key          |
| Signature                | Protects certificate contents          |

---

## Subject Alternative Name

For modern TLS service-identity verification, the application compares its expected reference identity with identities presented in the certificate’s Subject Alternative Name extension. RFC 9525 states that service identity belongs in `subjectAltName` rather than relying on the certificate Common Name.

Examples:

```text
DNS:payments.internal
DNS:payments.shop.svc.cluster.local
IP:192.0.2.15
URI:spiffe://production.example/ns/shop/sa/payments
```

The correct identifier type depends on the identity model.

---

## Certificate validity interval

```text
Not Before <= current time < Not After
```

A certificate can fail because:

* Client clock is behind
* Server clock is ahead
* Certificate has expired
* Certificate was issued with incorrect dates
* Rotation delivered a future-dated certificate
* Time synchronization failed

Short-lived certificates reduce compromise duration but increase dependence on reliable renewal.

---

## Key Usage and Extended Key Usage

A certificate may constrain use of its key.

Examples:

```text
Key Usage:
  Digital Signature

Extended Key Usage:
  TLS Web Server Authentication
```

or:

```text
Extended Key Usage:
  TLS Web Client Authentication
```

A certificate with a valid signature may still be inappropriate for the requested purpose.

Certificate validation includes more than verifying the issuer signature; implementations also evaluate certificate purpose and relevant usage constraints. OpenSSL’s verification interface distinguishes purposes such as `sslserver` and `sslclient`.

---

# 11.8 Certification authority and trust chain

A certificate authority, or CA, signs certificates.

Example chain:

```text
Root CA
  signs
Intermediate Workload CA
  signs
payments-service certificate
```

```mermaid
flowchart TD
    R[Root CA]
    I[Intermediate Workload CA]
    L[payments-service leaf certificate]

    R -->|Signs| I
    I -->|Signs| L
```

The verifier starts from a configured trust anchor.

It does not trust a chain merely because each certificate contains the next issuer’s name.

---

## Path validation mental model

The verifier checks:

1. Can a path be built from leaf certificate to a trusted root?
2. Are signatures valid?
3. Are certificates within their validity intervals?
4. Are CA constraints valid?
5. Are key usages appropriate?
6. Are critical extensions understood?
7. Does the leaf identity match the expected peer?
8. Is revocation required and available?
9. Does policy permit this trust domain and certificate purpose?

Certification-path validation is defined separately from the TLS handshake itself; TLS delegates detailed certificate validation to PKIX certificate-processing rules.

---

## Trust anchor

A root certificate is trusted because local configuration declares it trusted.

Not because:

* It is self-signed
* It has “Root CA” in its name
* Its signature verifies using its own public key
* The peer sent it during the handshake

A self-signed attacker certificate can verify its own signature.

It remains untrusted unless deliberately installed as a trust anchor.

---

## Intermediate certificates

Servers usually send:

```text
Leaf certificate
Intermediate certificate or certificates
```

They generally do not need to send the root trust anchor.

The client already has the trusted root.

A missing intermediate may cause:

```text
unable to build certificate chain
```

even when the leaf certificate itself is correctly signed.

---

# 11.9 Concrete example: orders-service calls payments-service

Architecture:

```text
orders-service:
  client workload

payments-service:
  target server workload

workload CA:
  issues short-lived certificates

authorization engine:
  decides which workload may call which API
```

```mermaid
flowchart LR
    O[orders-service]
    CA[Workload CA]
    P[payments-service]
    AZ[Authorization Policy]

    O -->|Obtain client identity| CA
    P -->|Obtain server identity| CA
    O -->|mTLS request| P
    P -->|Peer identity + action| AZ
    AZ -->|Allow or deny| P
```

Policy:

```text
ALLOW:
  subject = orders-service
  action = create-payment
  resource = payments API

DENY:
  all other callers
```

---

# 11.10 Stage 1: Workloads bootstrap identity

When `orders-service` starts, an identity agent or issuer examines runtime evidence.

Possible evidence:

* Kubernetes namespace and ServiceAccount
* Pod UID
* Container runtime metadata
* Node identity
* Executable hash
* Unix UID
* Cloud instance identity document
* TPM attestation
* VM image
* Deployment labels
* Signed scheduler metadata

Conceptual result:

```text
Runtime instance:
  cluster = production
  namespace = shop
  service account = orders

Mapped identity:
  orders-service
```

This mapping is security-critical.

If an attacker can modify trusted labels or run under the same runtime selector, they may receive the same identity.

---

# 11.11 Stage 2: Certificate issuance

The issuer creates or accepts a public key and signs a short-lived certificate.

Client identity:

```text
URI:
  spiffe://production.example/ns/shop/sa/orders

Public key:
  PK_orders

Validity:
  30 minutes

Purpose:
  client authentication
```

Server identity:

```text
DNS:
  payments.shop.svc.cluster.local

URI:
  spiffe://production.example/ns/shop/sa/payments

Purpose:
  server authentication
```

Private keys may be:

* Generated by the workload
* Generated by a local identity agent
* Stored in memory
* Held in a hardware module
* Exposed through a local signing API

The preferred design minimizes copying and long-term storage of private keys.

---

# 11.12 Stage 3: Network connection

`orders-service` opens a TCP connection to:

```text
payments.shop.svc.cluster.local:443
```

Before authentication:

```text
Network source address
does not reliably equal
workload identity
```

IP addresses may represent:

* NAT gateway
* Sidecar proxy
* Load balancer
* Node
* Pod
* Service virtual IP
* Reused address

Network location may be a policy input, but it is not automatically a cryptographic identity.

---

# 11.13 Stage 4: Server certificate presentation

`payments-service` presents its certificate chain.

`orders-service` checks:

```text
Trusted issuer?
Valid signatures?
Valid time?
Correct certificate purpose?
Expected service identity?
```

The expected identity should come from trusted application configuration or service discovery.

It should not be copied blindly from an attacker-controlled certificate.

---

## Expected identity versus presented identity

```text
Expected:
  payments.shop.svc.cluster.local

Presented:
  DNS:payments.shop.svc.cluster.local
```

Result:

```text
match
```

Wrong certificate:

```text
Presented:
  DNS:inventory.shop.svc.cluster.local
```

Result:

```text
reject
```

A certificate may chain to a trusted CA while identifying the wrong service.

---

# 11.14 Stage 5: Client certificate request

The server requests a client certificate.

It may indicate:

* Acceptable certificate authorities
* Supported signature algorithms
* Authentication context

The client selects an appropriate certificate and signs the relevant handshake transcript.

TLS 1.3 includes a `CertificateRequest`, a client `Certificate`, a `CertificateVerify` proof, and a `Finished` message when certificate-based client authentication is used.

---

# 11.15 Stage 6: Server validates client identity

`payments-service` validates the client chain and proof.

Conceptual security context:

```json
{
  "authenticated": true,
  "peer_identity": "spiffe://production.example/ns/shop/sa/orders",
  "issuer": "production-workload-ca",
  "certificate_serial": "71af...",
  "certificate_expiry": "2026-07-31T11:00:00Z"
}
```

At this point:

```text
TLS authentication succeeded
```

This does not yet mean:

```text
orders-service may create payments
```

---

# 11.16 Stage 7: Application authorization

Request:

```http
POST /v1/payments HTTP/1.1
Host: payments.shop.svc.cluster.local
Content-Type: application/json

{
  "order_id": "order-781",
  "amount": 4200
}
```

Authorization input:

```text
Subject:
  spiffe://production.example/ns/shop/sa/orders

Action:
  payment.create

Resource:
  payment for order-781

Context:
  production trust domain
  certificate valid
  request amount = 4200
  end-user context = Alice
```

Policy might require:

```text
ALLOW if:
  workload == orders-service
  AND end user may purchase order-781
  AND order is pending
  AND amount matches authorized order total
```

Service identity and end-user identity solve different problems.

---

# 11.17 Stage 8: Downstream operation

The payments service may then connect to:

* Database
* Payment processor
* Queue
* Audit service

Each connection introduces another identity and authorization decision.

```text
orders-service
→ payments-service
→ payment database
→ external payment provider
```

Do not assume authentication automatically propagates through all hops.

---

# 11.18 Complete mTLS sequence

```mermaid
sequenceDiagram
    participant Orders as orders-service
    participant Agent as Workload Identity Agent
    participant CA as Identity Issuer
    participant Payments as payments-service
    participant Policy as Authorization Engine
    participant DB as Payment Database

    Orders->>Agent: Request workload identity
    Agent->>Agent: Inspect runtime metadata
    Agent->>CA: Request certificate for authorized identity
    CA-->>Agent: Short-lived certificate
    Agent-->>Orders: Client certificate and key access

    Payments->>Agent: Request workload identity
    Agent->>CA: Request server certificate
    CA-->>Agent: Short-lived certificate
    Agent-->>Payments: Server certificate and key access

    Orders->>Payments: Begin TLS
    Payments-->>Orders: Server certificate + request client certificate
    Orders->>Orders: Validate chain, identity, time, purpose
    Orders->>Payments: Client certificate + signature proof
    Payments->>Payments: Validate chain, identity, time, purpose

    Orders->>Payments: POST /v1/payments
    Payments->>Policy: Subject + action + resource + context
    Policy-->>Payments: Allow
    Payments->>DB: Write payment
    DB-->>Payments: Success
    Payments-->>Orders: 201 Created
```

---

# 11.19 Data changes through the flow

| Stage                 | Input                       | Output                        |
| --------------------- | --------------------------- | ----------------------------- |
| Runtime attestation   | Pod/process/node metadata   | Workload identity             |
| Issuance              | Identity + public key       | Signed certificate            |
| Server TLS validation | Server certificate chain    | Authenticated server identity |
| Client TLS validation | Client certificate chain    | Authenticated client identity |
| HTTP parsing          | Method, route, body         | Requested application action  |
| Authorization         | Workload + user + resource  | Allow or deny                 |
| Database operation    | Authorized business request | Persistent state              |

---

## Questions: mTLS Flow

1. Why is the source IP insufficient as the primary workload identity?
2. What does the client prove when it sends `CertificateVerify`?
3. Why can a certificate chain be valid while the service identity is wrong?
4. Which component decides whether `orders-service` may create a payment?
5. Why must the server distinguish TLS authentication from application authorization?
6. What happens if the workload CA issues the `orders-service` identity to an attacker-controlled Pod?
7. Why should the expected server identity come from trusted configuration rather than the peer certificate?
8. Which identities exist when orders-service acts on Alice’s behalf?
9. How could a sidecar proxy change which process directly holds the private key?
10. What downstream identities exist when payments-service writes to its database?

---

# 11.20 Inspect an X.509 certificate

```bash
openssl x509 \
  -in workload.crt \
  -noout \
  -subject \
  -issuer \
  -serial \
  -dates \
  -fingerprint \
  -sha256
```

### What it does

Displays selected certificate metadata without printing the complete encoded certificate.

### Important flags

* `-in workload.crt`: input certificate
* `-noout`: suppress PEM output
* `-subject`: show subject
* `-issuer`: show issuer
* `-serial`: show serial number
* `-dates`: show validity interval
* `-fingerprint -sha256`: show SHA-256 fingerprint

OpenSSL’s `x509` utility supports inspecting subject, issuer, serial number, validity dates, fingerprints, extensions, and other certificate fields.

### Why it is useful

It quickly identifies:

* Wrong issuer
* Expired certificate
* Unexpected subject
* Different certificate generation
* Rotation mismatch

### Safety

Certificates are normally public, but they may reveal:

* Internal hostnames
* Organization names
* Workload topology
* Identity naming conventions

Do not accidentally point the command at a private-key file.

---

## Inspect Subject Alternative Names

```bash
openssl x509 \
  -in workload.crt \
  -noout \
  -ext subjectAltName
```

Illustrative output:

```text
X509v3 Subject Alternative Name:
    DNS:payments.internal,
    URI:spiffe://production.example/ns/shop/sa/payments
```

The exact output depends on the certificate.

---

## Inspect key usage

```bash
openssl x509 \
  -in workload.crt \
  -noout \
  -ext keyUsage \
  -ext extendedKeyUsage
```

Possible output:

```text
X509v3 Key Usage: critical
    Digital Signature
X509v3 Extended Key Usage:
    TLS Web Server Authentication
```

Check that the certificate purpose matches its actual use.

---

## Inspect complete details

```bash
openssl x509 \
  -in workload.crt \
  -noout \
  -text
```

### Why it is useful

It displays:

* Public-key algorithm
* Signature algorithm
* Extensions
* Constraints
* SANs
* Validity
* Issuer and subject

### Safety

The output is lengthy and may expose internal identity details.

Use targeted flags when sharing diagnostics.

---

# 11.21 Verify a certificate chain

```bash
openssl verify \
  -CAfile workload-root-ca.crt \
  -untrusted workload-intermediate-ca.crt \
  -purpose sslserver \
  payments.crt
```

### What it does

Builds and verifies a certificate path for `payments.crt`.

### Important flags

* `-CAfile`: trusted root certificates
* `-untrusted`: intermediate certificates used to construct the path
* `-purpose sslserver`: check suitability for server authentication
* `payments.crt`: leaf certificate to verify

OpenSSL’s `verify` command performs certificate-chain verification and supports purpose checks such as `sslserver` and `sslclient`.

### Successful illustrative output

```text
payments.crt: OK
```

### Important limitation

This does not necessarily verify the exact application service identity unless an appropriate hostname, IP, email, or other identity check is requested separately.

---

## Verify hostname

```bash
openssl verify \
  -CAfile workload-root-ca.crt \
  -untrusted workload-intermediate-ca.crt \
  -purpose sslserver \
  -verify_hostname payments.internal \
  payments.crt
```

This checks whether the certificate is appropriate for the expected DNS hostname in addition to chain and purpose validation.

---

# 11.22 Inspect a live TLS server

```bash
openssl s_client \
  -connect payments.internal:443 \
  -servername payments.internal \
  -showcerts \
  -verify_return_error \
  -verify_hostname payments.internal
```

### What it does

Connects as a TLS client and displays handshake and certificate information.

### Important flags

* `-connect HOST:PORT`: target
* `-servername`: send the TLS server name
* `-showcerts`: display certificates sent by the server
* `-verify_return_error`: stop on certificate verification errors
* `-verify_hostname`: verify expected DNS identity

OpenSSL documents that `s_client` supports certificate display, hostname checks, custom CA files, client certificates, private keys, and explicit failure on verification errors.

### Important misconception

Without `-verify_return_error`, some `s_client` invocations may continue after displaying verification errors.

Do not interpret a completed connection as proof that all validation succeeded.

### Safety

The output may reveal:

* Internal certificate chains
* Server software behavior
* Supported protocols
* Internal names

---

## Connect using a private CA

```bash
openssl s_client \
  -connect payments.internal:443 \
  -servername payments.internal \
  -CAfile workload-root-ca.crt \
  -verify_return_error \
  -verify_hostname payments.internal
```

Use this when the service uses an internal CA that is not in the system trust store.

Do not use an insecure “skip verification” option merely to hide a trust configuration problem.

---

# 11.23 Test mTLS with `openssl s_client`

```bash
openssl s_client \
  -connect payments.internal:443 \
  -servername payments.internal \
  -CAfile workload-root-ca.crt \
  -cert orders.crt \
  -key orders.key \
  -verify_return_error \
  -verify_hostname payments.internal
```

### Additional flags

* `-cert orders.crt`: client certificate
* `-key orders.key`: corresponding private key

### What it tests

* Server certificate trust
* Server hostname
* Client certificate presentation
* Client private-key possession
* Basic TLS negotiation

### What it does not fully test

* Application authorization
* HTTP route permissions
* End-user context
* Business logic
* Database access
* Whether the server maps the certificate identity correctly

### Safety

`orders.key` is a private key.

Protect it:

```bash
chmod 600 orders.key
```

Do not place private keys in shared diagnostic archives.

---

# 11.24 Test an HTTP mTLS endpoint with `curl`

```bash
curl -i \
  --cacert workload-root-ca.crt \
  --cert orders.crt \
  --key orders.key \
  https://payments.internal/health
```

### What it does

Sends an HTTPS request using:

* A custom trusted CA
* A client certificate
* The client private key

### Important flags

* `--cacert`: CA used to validate server
* `--cert`: client certificate
* `--key`: client private key
* `-i`: include response headers

The current curl command-line interface supports separate client certificate and private-key options for TLS client authentication.

### Output interpretation

```http
HTTP/1.1 200 OK
```

may show that TLS and the health route succeeded.

```http
HTTP/1.1 403 Forbidden
```

may mean:

```text
mTLS authentication succeeded
but
application authorization denied
```

A TLS alert before an HTTP response indicates failure earlier in the connection.

---

# 11.25 Certificate rotation

Short-lived workload certificates require automatic renewal.

Typical lifecycle:

```text
Certificate lifetime: 30 minutes
Renewal begins: after 15–20 minutes
Old certificate remains valid during overlap
New connections use new certificate
Old connections may continue according to TLS/session policy
Expired certificate removed
```

```mermaid
timeline
    title Certificate Rotation
    10:00 : Certificate A issued
    10:15 : Certificate B requested
    10:16 : Certificate B installed
    10:16 : New connections use B
    10:30 : Certificate A expires
```

---

## Rotation implementation choices

### Restart workload

```text
New certificate written
→ process restarted
```

Simple but disruptive.

### Reload certificate

```text
Identity agent updates files
→ application reloads without restart
```

Requires correct file-watching and atomic replacement.

### Local identity API

```text
Application receives updated identity through stream
```

Avoids relying on periodic file polling.

### Proxy termination

```text
Sidecar or local proxy holds certificates
→ application uses local plaintext or protected connection
```

Reduces application integration effort but moves trust to the proxy boundary.

---

## Rotation failure modes

* Application reads certificate once at startup
* Certificate file updates but private key does not
* Key and certificate are replaced non-atomically
* New chain lacks intermediate
* Issuer changes but trust bundle is stale
* Workload clock is wrong
* Renewal service unavailable
* File permissions prevent reading new key
* Existing connection lives beyond intended identity revocation
* Load balancer caches old certificate

---

# 11.26 Revocation versus short lifetimes

Certificate revocation mechanisms can include:

* Certificate Revocation Lists
* Online status checking
* Issuer-side deny state
* Trust-bundle removal
* Identity-registration removal
* Workload termination
* Very short certificate lifetimes

Short lifetimes reduce the maximum time a certificate remains valid after issuance stops.

They do not instantly revoke:

* Existing TLS connections
* Copied private keys with still-valid certificates
* Cached authorization decisions
* Session tickets or application sessions
* Credentials issued downstream

A rapid response may require:

```text
stop renewal
+ terminate workload
+ close active connections
+ update policy
+ revoke downstream credentials
```

---

# 11.27 Long-lived connections

Services may keep:

* HTTP/2 connections
* gRPC channels
* Database pools
* Message-broker sessions
* TLS sessions

A certificate can expire after the connection is established.

The exact application behavior may be:

```text
Existing connection continues
New connection fails
```

Therefore, certificate lifetime does not necessarily limit the lifetime of an already authenticated transport connection.

Systems requiring rapid revocation may need:

* Maximum connection age
* Periodic reauthentication
* Connection draining
* Server-side session termination
* Authorization checks per request

---

# 11.28 Kubernetes ServiceAccounts

A Kubernetes `ServiceAccount` is an API object representing an identity for processes running in Pods.

A typical use flow is:

```text
Create ServiceAccount
→ grant permissions
→ assign ServiceAccount to Pod
→ Pod receives credentials
→ workload authenticates to API server
```

Kubernetes documentation describes this exact three-part model: create the ServiceAccount, grant it permissions using an authorizer such as RBAC, and assign it to Pods.

---

## Kubernetes identity name

A ServiceAccount is namespaced.

Conceptual identity:

```text
system:serviceaccount:shop:orders
```

Components:

```text
namespace = shop
service account = orders
```

The same ServiceAccount name in another namespace represents another principal:

```text
system:serviceaccount:staging:orders
≠
system:serviceaccount:shop:orders
```

---

## ServiceAccount is not the Pod

Several Pods can use the same ServiceAccount.

```text
orders-pod-1
orders-pod-2
orders-pod-3
→ ServiceAccount shop/orders
```

The API server can authenticate the ServiceAccount identity.

Application-level audit may need additional metadata to identify the individual Pod instance.

---

# 11.29 Create a Kubernetes ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: orders
  namespace: shop
```

Apply:

```bash
kubectl apply -f orders-serviceaccount.yaml
```

### What it does

Creates or updates the declared ServiceAccount object.

### Why it is useful

It creates a distinct principal for the workload rather than using the namespace’s default ServiceAccount.

### Safety

Creating the ServiceAccount alone does not grant meaningful access.

Permissions are granted separately through authorization policy.

---

## Inspect it

```bash
kubectl get serviceaccount orders \
  --namespace shop \
  -o yaml
```

### Important flags

* `--namespace shop`: select namespace
* `-o yaml`: print complete API representation

### Why it is useful

It shows:

* Name
* Namespace
* UID
* Metadata
* Image pull secrets
* Automount configuration

Kubernetes documents `kubectl get serviceaccounts` and setting `spec.serviceAccountName` to assign a non-default ServiceAccount.

---

# 11.30 Assign a ServiceAccount to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: orders-api
  namespace: shop
spec:
  serviceAccountName: orders
  containers:
    - name: orders-api
      image: registry.example.internal/orders-api:1.8.0
```

Important field:

```yaml
serviceAccountName: orders
```

If no ServiceAccount is specified, the namespace’s `default` ServiceAccount is normally assigned.

---

## Why not grant permissions to `default`?

Any Pod that does not declare another ServiceAccount may use the default identity.

Granting broad permissions to it can unintentionally grant the same authority to many unrelated workloads.

An application-specific ServiceAccount creates a narrower policy boundary. Kubernetes RBAC guidance recommends application-specific ServiceAccounts as the most secure of the standard ServiceAccount-grant patterns.

---

# 11.31 ServiceAccount token data flow

Modern Kubernetes deployments typically use the TokenRequest mechanism and a projected volume to provide a short-lived, automatically rotating ServiceAccount token to a Pod. Older clusters used long-lived static Secret-backed tokens more extensively.

```mermaid
sequenceDiagram
    participant Pod
    participant Kubelet
    participant API as Kubernetes API Server
    participant Authz as Kubernetes Authorizer

    Kubelet->>API: Request bounded ServiceAccount token
    API-->>Kubelet: Signed short-lived token
    Kubelet->>Pod: Project token into filesystem
    Pod->>API: HTTPS request + bearer token
    API->>API: Validate token and audience
    API->>Authz: Subject + verb + resource
    Authz-->>API: Allow or deny
```

---

## Typical mounted files

A Pod may receive files under:

```text
/var/run/secrets/kubernetes.io/serviceaccount/
```

Common entries include:

```text
token
ca.crt
namespace
```

Exact projection and file paths can vary with Pod configuration and Kubernetes implementation.

---

## Token is a bearer credential

A ServiceAccount token can generally be used by whoever possesses it within its valid conditions.

If an attacker reads the token from a compromised container:

```text
Attacker may call API
as
system:serviceaccount:shop:orders
```

The impact depends on the ServiceAccount’s RBAC permissions.

---

# 11.32 Opt out of automatic token mounting

A workload that does not call the Kubernetes API may not need an API credential.

ServiceAccount-level configuration:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: static-web
  namespace: shop
automountServiceAccountToken: false
```

Pod-level configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  namespace: shop
spec:
  serviceAccountName: static-web
  automountServiceAccountToken: false
  containers:
    - name: web
      image: registry.example.internal/static-web:2.1.0
```

Kubernetes supports disabling automatic ServiceAccount credential mounting at either the ServiceAccount or Pod level; the Pod-level setting takes precedence when both are present.

### Why it matters

An unnecessary credential turns an application compromise into potential control-plane access.

---

# 11.33 Kubernetes RBAC objects

Kubernetes RBAC uses:

* `Role`
* `ClusterRole`
* `RoleBinding`
* `ClusterRoleBinding`

A Role contains additive permission rules within a namespace.

A ClusterRole is not namespaced and can represent cluster-wide or reusable permissions.

Kubernetes RBAC permissions are additive; there are no explicit deny rules in the RBAC model itself.

---

## Example Role

Allow reading ConfigMaps in namespace `shop`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: config-reader
  namespace: shop
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
```

### Fields

* `apiGroups: [""]`: core API group
* `resources: ["configmaps"]`: target resource type
* `verbs`: allowed API operations

---

## Example RoleBinding

Bind that Role to the `orders` ServiceAccount:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: orders-config-reader
  namespace: shop
subjects:
  - kind: ServiceAccount
    name: orders
    namespace: shop
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: config-reader
```

The RoleBinding exists in the namespace where the permissions apply.

---

## Authorization tuple

A request might become:

```text
Subject:
  system:serviceaccount:shop:orders

Verb:
  get

API group:
  core

Resource:
  configmaps

Namespace:
  shop

Name:
  order-settings
```

RBAC returns:

```text
allow
```

---

# 11.34 Inspect effective Kubernetes authorization

```bash
kubectl auth can-i get configmaps \
  --namespace shop \
  --as=system:serviceaccount:shop:orders
```

### What it does

Asks the Kubernetes authorization layer whether the specified principal can perform the action.

### Important parts

* `get`: API verb
* `configmaps`: resource
* `--namespace shop`: target namespace
* `--as=...`: impersonate the ServiceAccount for the authorization check

### Output

```text
yes
```

or:

```text
no
```

`kubectl auth can-i` is specifically designed to query whether an action is allowed and supports impersonating a ServiceAccount using `--as`. The caller must itself be authorized to impersonate that identity.

### Safety

Impersonation permission is powerful.

This command tests authorization but does not automatically reproduce every admission-control or application condition affecting a real request.

---

## List allowed actions

```bash
kubectl auth can-i --list \
  --namespace shop \
  --as=system:serviceaccount:shop:orders
```

### Why it is useful

It helps identify unexpectedly broad permissions.

### Limitation

The output may not fully express:

* Admission policy
* Resource-name restrictions in an intuitive way
* External authorization webhooks
* Runtime network restrictions
* Application-specific authorization

---

# 11.35 Request a temporary ServiceAccount token

```bash
kubectl create token orders \
  --namespace shop \
  --duration 10m
```

### What it does

Requests a ServiceAccount token through the Kubernetes API.

### Important flags

* `orders`: ServiceAccount name
* `--namespace shop`: ServiceAccount namespace
* `--duration 10m`: requested lifetime

The server may return a different lifetime according to cluster policy. `kubectl create token` can also request a custom audience and can bind tokens to supported Kubernetes objects.

### Safety

The command prints a live bearer token.

Do not:

* Run it during a screen share
* Paste output into tickets
* Save it in shell history
* Put it in source code
* Use it as a permanent credential

---

## Request a custom audience

```bash
kubectl create token orders \
  --namespace shop \
  --duration 10m \
  --audience https://inventory.example.internal
```

The target verifier should validate that its expected audience is present.

A token intended for the Kubernetes API should not automatically be accepted by an unrelated service.

---

# 11.36 ServiceAccount token structure

A projected ServiceAccount token is commonly a signed token containing claims such as:

```json
{
  "iss": "https://kubernetes.default.svc",
  "sub": "system:serviceaccount:shop:orders",
  "aud": ["https://kubernetes.default.svc"],
  "exp": 1785498000,
  "iat": 1785494400
}
```

Additional Kubernetes-specific claims may bind the token to:

* Namespace
* ServiceAccount
* Pod
* Node
* Object UID

Exact claims depend on Kubernetes version and token-request context.

---

## Validation requirements

A receiving service should validate:

* Signature
* Trusted issuer
* Intended audience
* Expiry
* Not-before time
* ServiceAccount subject
* Bound object where required
* Token-review or revocation state where required
* Local authorization policy

Do not merely decode the JWT and trust `sub`.

---

# 11.37 TokenReview

A service integrated with the Kubernetes control plane may ask the API server to review a token.

Conceptual request:

```yaml
apiVersion: authentication.k8s.io/v1
kind: TokenReview
spec:
  token: REDACTED
  audiences:
    - https://inventory.example.internal
```

Conceptual result:

```yaml
status:
  authenticated: true
  user:
    username: system:serviceaccount:shop:orders
  audiences:
    - https://inventory.example.internal
```

Tradeoffs:

* Central validation and current cluster knowledge
* Runtime dependency on API server
* Additional latency and load
* Need to authorize the reviewing service
* Credential exposure to the review endpoint

---

# 11.38 Kubernetes authorization is not Pod isolation

A ServiceAccount and RBAC control Kubernetes API access.

They do not automatically restrict:

* Pod-to-Pod network traffic
* Linux filesystem access
* Node access
* Cloud API access
* Application endpoints
* Secret values already mounted
* Container escape
* Database permissions

Other layers include:

* NetworkPolicy
* Pod security controls
* Linux namespaces
* Seccomp
* AppArmor or SELinux
* Cloud identity
* Application authorization
* Secret-store policy

---

# 11.39 Kubernetes privilege-escalation relationships

Some Kubernetes permissions indirectly grant more authority than their names suggest.

Examples:

```text
create pods
→ run workload under another ServiceAccount

read secrets
→ steal credentials

create rolebindings
→ grant permissions

create tokens
→ obtain ServiceAccount credentials

exec into pods
→ access mounted credentials and process environment

update workloads
→ replace image with attacker-controlled code
```

Authorization review must consider what a permission enables indirectly.

---

# 11.40 Kubernetes troubleshooting flow

Symptom:

```text
Pod receives:
403 Forbidden
from Kubernetes API
```

Debug stages:

1. Confirm the Pod’s ServiceAccount.
2. Confirm token is mounted.
3. Confirm token audience.
4. Confirm token has not expired.
5. Confirm API server trusts issuer.
6. Resolve authenticated username.
7. Test RBAC with `kubectl auth can-i`.
8. Inspect RoleBindings and ClusterRoleBindings.
9. Confirm namespace.
10. Confirm API group and subresource.
11. Inspect admission-control rejection.
12. Inspect audit logs.

---

## Inspect Pod ServiceAccount

```bash
kubectl get pod orders-api \
  --namespace shop \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

### What it does

Prints the ServiceAccount assigned to the Pod.

### Why it is useful

It detects:

* Accidental use of `default`
* Wrong deployment template
* Incorrect namespace assumption

---

## Inspect bindings

```bash
kubectl get rolebindings,clusterrolebindings \
  --all-namespaces
```

### Safety and output size

This can expose cluster-wide authorization structure and may produce a large output.

Prefer targeted inspection when possible:

```bash
kubectl describe rolebinding orders-config-reader \
  --namespace shop
```

---

# 11.41 Cloud workload identity federation

Cloud platforms commonly support temporary credentials derived from another trusted identity.

Generic flow:

```text
Kubernetes or VM identity
→ signed token or attestation
→ cloud security-token service
→ temporary cloud role credential
→ cloud API request
```

```mermaid
sequenceDiagram
    participant W as Workload
    participant L as Local Identity Provider
    participant STS as Cloud Token Service
    participant API as Cloud Resource API

    W->>L: Obtain signed workload assertion
    L-->>W: Short-lived assertion
    W->>STS: Exchange assertion for cloud role
    STS->>STS: Validate issuer, audience, subject, policy
    STS-->>W: Temporary cloud credential
    W->>API: Signed request using temporary credential
    API->>API: Authenticate role and authorize action
```

---

## Why federation is preferable to static cloud keys

Static model:

```text
Create permanent cloud key
→ store in Kubernetes Secret
→ mount into Pod
```

Federated model:

```text
Pod identity
→ temporary cloud credential
```

Benefits:

* No permanent cloud secret stored in the cluster
* Short credential lifetime
* Role policy based on workload identity
* Automatic renewal
* Better issuer and audience restrictions
* Easier per-workload separation

---

## Federation trust policy

The cloud token service may require:

```text
Trusted issuer:
  cluster identity issuer

Audience:
  cloud token service

Subject:
  system:serviceaccount:shop:orders
```

Mapped role:

```text
orders-object-reader
```

A broad condition such as:

```text
any ServiceAccount in any namespace
```

can allow unintended workloads to assume the cloud role.

---

## Confused deputy risk

Suppose one intermediary service can obtain cloud credentials for many workloads.

An attacker asks it to obtain:

```text
administrator role
```

The intermediary must verify:

* Which caller is requesting the exchange
* Which role the caller is allowed to obtain
* Which audience is intended
* Which tenant applies
* Whether delegation is permitted

Otherwise, the credential broker becomes a confused deputy.

---

# 11.42 Node identity versus workload identity

A node may host many workloads.

```text
Node identity:
  worker-node-17

Workload identity:
  orders-service
```

If every Pod inherits node-level cloud authority:

```text
Compromise one Pod
→ obtain node credential
→ access authority intended for all node workloads
```

Workload-level identity provides finer isolation.

Node identity may still bootstrap workload identity.

```text
Trusted node
→ attests Pod properties
→ workload issuer creates Pod identity
```

---

# 11.43 Instance metadata services

Cloud environments may expose identity or credential APIs through a link-local or local endpoint.

A workload might request:

```text
temporary credentials for attached role
```

Security risks include:

* Server-side request forgery reaching metadata
* Containers inheriting node authority
* Untrusted processes accessing local endpoint
* Credential theft from compromised workload
* Excessive role permissions

Defenses vary by platform but may include:

* Session-oriented metadata protocols
* Network filtering
* Workload-specific identity binding
* Hop restrictions
* Proxy controls
* Minimal role policy

---

# 11.44 SPIFFE mental model

SPIFFE defines a framework for workload identity across heterogeneous environments.

Its central concepts include:

* SPIFFE ID
* SPIFFE Verifiable Identity Document, or SVID
* Workload API
* Trust domain
* Trust bundle

The SPIFFE specification defines a structured identity namespace and an SVID as the cryptographically verifiable mechanism by which a workload communicates its SPIFFE identity.

---

## SPIFFE ID

Example:

```text
spiffe://production.example/ns/shop/sa/orders
```

Breakdown:

```text
scheme:
  spiffe

trust domain:
  production.example

path:
  /ns/shop/sa/orders
```

The path structure is operator-defined.

It might encode:

* Environment
* Cluster
* Namespace
* Service account
* Application
* Region
* Deployment

Do not infer authorization semantics from the text unless the organization defines that contract.

---

## Trust domain

A trust domain is an identity namespace with associated trust material.

Examples:

```text
spiffe://production.example/...
spiffe://staging.example/...
spiffe://partner.example/...
```

A verifier needs the appropriate trust bundle for the presented trust domain.

Foreign trust domains require an explicit federation relationship rather than automatic trust. The SPIFFE Workload API specification describes using trust-domain-specific bundles and rejecting peers where no matching trusted bundle is available.

---

## SVID

An SVID carries:

* SPIFFE ID
* Valid signature
* Optional public key, depending on representation

Defined representations include X.509-based and token-based identity documents.

---

## X.509-SVID

Conceptual identity:

```text
URI SAN:
  spiffe://production.example/ns/shop/sa/orders
```

Used for:

* Mutual TLS
* Peer authentication
* Signed communication
* Secure channels

---

## JWT-SVID

Conceptual claims:

```json
{
  "sub": "spiffe://production.example/ns/shop/sa/orders",
  "aud": ["inventory-api"],
  "exp": 1785498000
}
```

Useful where bearer tokens integrate more easily than mTLS.

It introduces bearer-token replay and storage concerns.

---

# 11.45 SPIFFE Workload API

A local workload obtains identity material through a Workload API endpoint.

Conceptual flow:

```text
Process connects to local Unix socket
→ local identity agent determines caller
→ agent returns identity material authorized for that process
```

```mermaid
sequenceDiagram
    participant W as Workload Process
    participant A as Local Identity Agent
    participant I as Identity Issuer

    W->>A: Connect to local Workload API
    A->>A: Inspect caller runtime properties
    A->>I: Obtain or renew authorized identity
    I-->>A: Signed identity document
    A-->>W: SVID + trust bundle
```

The SPIFFE Workload API intentionally relies on the endpoint implementation to identify the local caller out-of-band rather than requiring the workload to present another bootstrap token. It also supports streaming updated identity and trust material to connected workloads.

---

## Why a local Unix socket helps

On a local Unix-domain socket, the agent may observe:

* Peer process UID
* Process ID
* Cgroup
* Container metadata
* Executable path
* Pod identity
* Namespace membership

The exact selectors and attestation mechanism depend on implementation.

The socket itself must be protected so that unrelated workloads cannot receive each other’s identity.

---

## Identity delivery

Possible delivery patterns:

* Workload directly calls API
* Sidecar proxy calls API
* Node agent writes files
* Secret volume is updated
* Local proxy terminates mTLS
* Application receives certificate through library callback

Each pattern changes:

* Which process holds private key
* Rotation behavior
* Trust boundary
* Application complexity
* Incident-response evidence

---

# 11.46 Service mesh identity

A service mesh may run a proxy alongside each workload.

```text
Application
→ local proxy
→ mTLS
→ remote proxy
→ remote application
```

```mermaid
flowchart LR
    A1[orders application]
    P1[orders proxy]
    P2[payments proxy]
    A2[payments application]

    A1 -->|Local connection| P1
    P1 == mTLS ==> P2
    P2 -->|Local connection| A2
```

The proxies may:

* Obtain workload certificates
* Establish mTLS
* Authenticate peer identity
* Enforce network policy
* Produce telemetry
* Rotate certificates

---

## Important trust boundary

If the application-to-proxy connection is unprotected:

```text
Proxy authenticates remote workload
but
local proxy must still know which local application sent the request
```

If several processes can reach the local proxy, one may impersonate another at the application layer.

Local isolation may depend on:

* Loopback binding
* Unix sockets
* Network namespaces
* Pod boundaries
* Peer credentials
* Proxy metadata
* Sidecar configuration

---

## Mesh identity does not replace application authorization

A mesh may prove:

```text
Caller is orders-service
```

The application must still decide:

```text
May orders-service modify order-781?
May it call the refund endpoint?
May Alice approve this transaction?
```

Transport policy is generally coarser than business authorization.

---

# 11.47 Service identity and end-user identity

A backend request may contain two independent principals.

```text
Transport caller:
  orders-service

Delegated user:
  Alice
```

Authorization example:

```text
ALLOW refund if:
  transport caller == orders-service
  AND user == Alice
  AND Alice owns order
  AND order is refundable
```

Do not accept end-user identity from an unsigned header merely because mTLS authenticated the service.

---

## Identity propagation pattern

```text
Frontend authenticates Alice
→ frontend obtains downstream token
→ token identifies Alice and frontend
→ backend validates token
→ mTLS identifies frontend workload
```

Possible downstream security context:

```json
{
  "workload_subject": "orders-service",
  "end_user_subject": "u_1048",
  "delegation_chain": ["finance-web", "orders-service"],
  "scopes": ["orders.refund"],
  "authentication_time": "2026-07-31T10:31:02Z"
}
```

---

## Actor and subject

Useful distinction:

```text
actor:
  workload making request

subject:
  user on whose behalf action occurs
```

For service-only operations:

```text
actor = backup-service
subject = backup-service
```

For delegated operations:

```text
actor = orders-service
subject = Alice
```

Audit systems should preserve both.

---

# 11.48 Token forwarding versus token exchange

## Direct forwarding

```text
Frontend receives user token
→ sends same token to every backend
```

Problems:

* Token audience may be wrong
* Scope may be too broad
* Every backend sees same credential
* Delegation path is hidden
* Token theft has broad impact

---

## Token exchange

```text
Frontend presents upstream token
→ issuer creates narrower downstream token
```

Example:

```text
Upstream token:
  audience = frontend
  scope = broad user session

Downstream token:
  audience = payments-api
  scope = payment.create
  actor = orders-service
  subject = Alice
  expiry = 5 minutes
```

This gives each backend a credential intended for it.

---

# 11.49 Message queues and asynchronous identity

A producer may send:

```json
{
  "event_type": "order.created",
  "order_id": "order-781",
  "requested_by": "u_1048"
}
```

The queue authenticates the producer connection.

The consumer must decide whether it trusts the message contents.

Questions:

* Which producer identity published it?
* Is the topic restricted?
* Can another producer forge `requested_by`?
* Is the message signed?
* Is integrity protected in storage?
* Can it be replayed?
* Is authorization checked again at consumption?
* What happens after user access is revoked?

---

## Broker-level authentication

The producer may authenticate using:

* mTLS certificate
* SASL credential
* OAuth token
* Cloud workload identity
* Kerberos
* Static username and password

Broker authorization may permit:

```text
orders-service:
  publish to orders.created

payments-service:
  consume from orders.created
```

This does not prove every field in the message is semantically valid.

---

## Message signing

A producer can sign selected message data.

Conceptual envelope:

```json
{
  "payload": {
    "order_id": "order-781",
    "amount": 4200
  },
  "producer": "orders-service",
  "timestamp": "2026-07-31T11:00:00Z",
  "message_id": "msg-9182",
  "signature": "..."
}
```

The consumer verifies:

* Signature
* Producer key
* Message ID
* Timestamp
* Expected topic
* Schema
* Replay state

Encryption of the broker connection does not protect a message after it is stored or forwarded by other components unless those components remain trusted.

---

# 11.50 Database identities

An application can authenticate to a database using:

* Static password
* Client certificate
* Kerberos
* Operating-system peer identity
* Cloud temporary token
* Managed identity
* Secret-manager-issued dynamic credential

The database principal should be application-specific.

Example:

```text
reports-api database role:
  SELECT reports
  INSERT audit rows
  no schema modification
  no user administration
```

---

## Shared database account failure

```text
frontend
worker
reporting job
administrator script
→ all connect as app_user
```

The database cannot distinguish callers.

A compromised low-risk worker inherits all authority of the shared principal.

---

## Connection pools

A pool may retain an authenticated database session.

If application-level users share one database principal:

```text
Database sees reports-api
not Alice
```

The application must enforce user-level authorization.

Alternatively, some systems use per-user database identities or secure session context, but this introduces:

* Pooling complexity
* Context-reset requirements
* Credential issuance load
* Database role explosion
* Auditing tradeoffs

---

## Dynamic database credentials

A credential broker may issue:

```text
Username:
  reports-api-session-781

Password:
  random

Lifetime:
  15 minutes

Permissions:
  read-only reports schema
```

Benefits:

* Short-lived secrets
* Automatic expiry
* Per-instance audit
* Reduced permanent password exposure

Operational concerns:

* Existing connections after expiry
* Renewal
* Pool draining
* Broker outages
* Role cleanup
* Clock synchronization

---

# 11.51 Local Unix-socket peer authentication

Processes communicating over a local Unix-domain socket may use kernel-provided peer credentials.

The server can learn properties such as:

* Peer UID
* Peer GID
* Peer PID

Exact interfaces vary by operating system.

Mental model:

```text
Client connects to local socket
→ kernel associates process credentials
→ server queries peer identity
→ server applies local authorization
```

This avoids passing a reusable password between local processes.

---

## Where it fits

Useful for:

* Local credential agents
* Container runtime APIs
* System daemons
* Database peer authentication
* Administrative sockets
* Secret agents

Security depends on:

* Socket filesystem permissions
* Namespace boundaries
* UID mapping
* Process privilege
* Whether peer PID metadata can be trusted across namespaces
* Server authorization logic

---

# 11.52 Device and node attestation

A node identity system may use hardware-backed evidence.

Possible components:

* TPM
* Secure boot measurements
* Hardware key
* Firmware state
* Machine certificate
* Cloud provider identity
* Signed boot metadata

Conceptual flow:

```text
Node proves hardware-backed key possession
+
reports expected measurements
→ attestation service
→ node identity issued
```

Attestation answers a stronger question than a static certificate alone:

```text
Which device is this?
What software or boot state is it reporting?
```

The verifier must still define which measurements are acceptable.

---

## Attestation limitations

* Measurement values need policy interpretation.
* A valid node does not prove every hosted workload is safe.
* Hardware trust does not eliminate application vulnerabilities.
* Firmware and supply-chain trust remain dependencies.
* Attestation services become high-value infrastructure.
* Recovery and replacement require careful lifecycle handling.

---

# 11.53 Secret manager authorization

A secret manager should authenticate the workload and authorize each requested secret.

Example:

```text
Subject:
  payments-service

Action:
  read

Secret:
  payment-processor-api-key

Decision:
  allow
```

Another workload:

```text
Subject:
  orders-service

Secret:
  payment-processor-api-key

Decision:
  deny
```

---

## Secret lease

A secret manager may return a leased credential:

```text
Lease ID:
  lease-718

Credential:
  temporary database user

Expires:
  15 minutes

Renewable:
  yes
```

Lease tracking allows:

* Renewal
* Revocation
* Audit
* Automatic cleanup
* Credential lifecycle management

---

## Secret caching

Applications often cache secrets to reduce dependency and latency.

Tradeoff:

```text
Long cache lifetime
→ better availability
→ slower revocation

Short cache lifetime
→ faster revocation
→ more dependency on secret manager
```

A local agent can centralize caching and renewal.

---

# 11.54 Identity issuer as a high-value control plane

A workload identity issuer can create credentials accepted by many services.

Compromise may allow an attacker to issue:

```text
payments-service identity
database-admin identity
deployment-controller identity
```

Protections include:

* Offline or tightly controlled root keys
* Intermediate issuer separation
* Hardware-backed signing keys
* Narrow issuance policy
* Audit logs
* Short credential lifetime
* Issuer rotation
* Administrative separation
* Rate limits
* Anomaly detection
* Trust-domain isolation

---

## Separate trust domains

Possible separation:

```text
production.example
staging.example
development.example
```

Benefits:

* Development issuer cannot authenticate to production
* Compromise scope is reduced
* Policy is easier to reason about
* Certificate and identity names remain unambiguous

Cross-domain trust should be explicit and directional where possible.

---

# 11.55 Common workload-identity failures

## Failure: every workload uses one identity

```text
all microservices
→ platform-service-account
```

Result:

* No service separation
* Weak auditability
* One compromise affects every permission
* Authorization cannot distinguish callers

---

## Failure: identity based only on mutable labels

```text
label app=payments
→ receive payments identity
```

If any namespace user can set that label, they can obtain the identity.

Selectors must be backed by a trusted control-plane boundary.

---

## Failure: node identity mistaken for workload identity

```text
Any Pod on worker-17
→ inherits worker-17 cloud role
```

A compromised Pod gains node-wide authority.

---

## Failure: certificate chain validated but SAN ignored

```text
Certificate from trusted CA
→ accepted regardless of service name
```

An inventory certificate may impersonate payments.

---

## Failure: client certificate optional by mistake

Server configuration:

```text
request client certificate
but
continue when absent
```

Application assumes every connection is authenticated.

The result may be an anonymous connection entering an authenticated code path.

---

## Failure: authorization trusts certificate Common Name loosely

Example:

```text
CN=orders
```

Problems:

* Ambiguous naming
* Duplicate names
* Wrong parsing
* No trust-domain context
* Modern service identity belongs in explicitly verified SAN identifiers

---

## Failure: certificate and key mismatch

```text
new certificate
+
old private key
```

TLS authentication fails because the key cannot produce a valid proof for the certificate public key.

---

## Failure: stale trust bundle

Issuer rotates intermediate CA.

Some workloads trust:

```text
old intermediate only
```

New certificates fail on those workloads.

---

## Failure: broad Kubernetes default ServiceAccount

```text
default ServiceAccount
→ cluster-wide read access
```

Every Pod using the default identity inherits that access.

---

## Failure: token audience ignored

```text
Kubernetes API token
→ accepted by unrelated internal service
```

A stolen token gains authority beyond its intended recipient.

---

## Failure: token mounted where unnecessary

A static web container has no API requirement but receives a ServiceAccount token.

XSS or remote-code execution now exposes a control-plane credential.

---

## Failure: bearer token placed in logs

Common leakage points:

* Reverse proxy access logs
* Debug middleware
* Exception traces
* Shell history
* Kubernetes event text
* CI output
* Support bundles

---

## Failure: mTLS identity lost at proxy

```text
Client
→ mTLS proxy
→ plaintext backend
```

Backend trusts:

```http
X-Client-Identity: orders-service
```

but the backend is directly reachable.

An attacker bypasses the proxy and forges the header.

---

## Failure: long-lived connection survives revocation

Certificate is revoked or expires, but an existing gRPC connection remains active.

The service continues accepting requests over the previously authenticated channel.

---

# 11.56 Troubleshooting scenario: TLS handshake fails with unknown CA

Symptom:

```text
certificate verify failed
unable to get local issuer certificate
```

Possible causes:

* Root CA missing
* Intermediate missing
* Wrong trust domain
* Server sends incomplete chain
* Trust bundle not reloaded
* Wrong certificate selected
* CA rotation incomplete

Debug:

```text
Inspect live chain
→ inspect leaf issuer
→ inspect intermediate subject/issuer
→ verify chain locally
→ compare trust bundle
→ inspect rotation timestamps
```

Commands:

```bash
openssl s_client \
  -connect payments.internal:443 \
  -servername payments.internal \
  -showcerts
```

Then save relevant certificates and verify using:

```bash
openssl verify \
  -CAfile root-ca.crt \
  -untrusted intermediate-ca.crt \
  payments.crt
```

---

# 11.57 Troubleshooting scenario: certificate valid but hostname mismatch

Symptom:

```text
certificate verify failed
hostname mismatch
```

Inspect SAN:

```bash
openssl x509 \
  -in payments.crt \
  -noout \
  -ext subjectAltName
```

Compare:

```text
Expected:
  payments.internal

Certificate:
  DNS:payments.svc.internal
```

Possible causes:

* Wrong service name
* Wrong certificate deployed
* DNS alias not included
* Client connects using IP while certificate contains DNS only
* Service discovery uses different canonical name
* Proxy terminates TLS with another identity

Do not disable hostname verification.

Fix the identity contract.

---

# 11.58 Troubleshooting scenario: mTLS succeeds but API returns 403

Interpretation:

```text
Transport authentication succeeded
Application authorization failed
```

Inspect:

* Extracted peer identity
* Identity mapping
* Requested route and method
* Policy version
* Namespace or tenant
* End-user token
* Scope
* Resource ownership
* Proxy identity headers
* Authorization logs

This is analogous to:

```text
Valid human session
but
insufficient permission
```

---

# 11.59 Troubleshooting scenario: only some replicas fail

Possible causes:

* Replica has stale CA bundle
* Replica did not reload certificate
* Different system clock
* Old configuration
* Different proxy version
* Missing secret volume update
* Identity agent unavailable on one node
* DNS routes to an old deployment
* Key/certificate pair differs
* Authorization cache differs

Compare per replica:

```text
certificate serial
certificate expiry
issuer
trust-bundle hash
process start time
configuration version
node
clock
```

---

# 11.60 Troubleshooting scenario: Kubernetes Pod receives 401

`401` often points to authentication failure.

Check:

* Token mounted
* Token file readable
* Token not expired
* Correct issuer
* Correct audience
* API server address
* CA bundle
* Authorization header format
* Token bound to deleted Pod
* Kubelet rotation
* Time synchronization

A `403` usually means the identity was accepted but authorization denied, though intermediaries and applications do not always use status codes consistently.

---

# 11.61 Troubleshooting scenario: Kubernetes access works after token theft

An attacker copies a projected token.

Containment steps may include:

```text
Delete or replace compromised Pod
→ remove ServiceAccount permissions
→ investigate bound token state
→ rotate downstream credentials
→ inspect API audit logs
→ fix original container compromise
```

Deleting only the ServiceAccount token file inside the container is insufficient if the attacker already copied it.

---

# 11.62 Troubleshooting scenario: cloud role assumption denied

Trace:

```text
Local workload identity
→ signed assertion
→ cloud issuer trust
→ subject condition
→ audience condition
→ role policy
→ temporary credential issuance
```

Possible causes:

* Wrong issuer
* Wrong audience
* Wrong ServiceAccount namespace
* Subject pattern mismatch
* Token expired
* Cluster issuer changed
* Cloud trust configuration stale
* Workload mapped to wrong role
* Clock skew
* Token-exchange endpoint unavailable

---

# 11.63 Observability

Useful workload-identity events include:

```text
identity issuance requested
runtime attestation accepted or denied
certificate issued
certificate renewed
certificate reload succeeded
certificate reload failed
peer certificate accepted
peer certificate rejected
token exchange requested
token audience rejected
authorization decision
trust bundle updated
unknown issuer
unknown key ID
credential near expiry
```

---

## Example issuance event

```json
{
  "timestamp": "2026-07-31T11:10:00Z",
  "event_type": "workload_certificate_issued",
  "workload_id": "spiffe://production.example/ns/shop/sa/orders",
  "node_id": "worker-17",
  "certificate_serial": "71af...",
  "expires_at": "2026-07-31T11:40:00Z",
  "issuer": "production-workload-ca-3",
  "result": "success"
}
```

Do not log:

* Private keys
* Complete bearer tokens
* Cloud secret keys
* Database passwords
* Secret-manager lease credentials

---

## Example peer-authentication log

```json
{
  "timestamp": "2026-07-31T11:12:18Z",
  "event_type": "mtls_peer_authenticated",
  "local_workload": "payments-service",
  "peer_identity": "spiffe://production.example/ns/shop/sa/orders",
  "peer_certificate_serial": "71af...",
  "tls_version": "TLSv1.3",
  "authorization_decision": "allow",
  "request_id": "req-8192"
}
```

---

## Metrics

Useful metrics:

```text
workload_identity_issuance_total
workload_identity_issuance_failures_total
certificate_seconds_until_expiry
certificate_renewal_duration_seconds
certificate_reload_failures_total
mtls_handshake_failures_total
mtls_unknown_ca_total
mtls_identity_mismatch_total
token_exchange_failures_total
service_account_authorization_denials_total
trust_bundle_refresh_failures_total
```

Avoid high-cardinality labels such as:

* Certificate serial
* Pod UID
* Request ID
* Raw token subject for every instance

Use structured logs for per-instance detail.

---

# 11.64 Security considerations

## Prefer short-lived credentials

Short lifetimes reduce the usefulness of stolen credentials.

They require:

* Reliable renewal
* Accurate clocks
* Automated reload
* Issuer capacity
* Failure monitoring

---

## Bind credentials to an audience

A token for:

```text
inventory-api
```

should not be accepted by:

```text
payments-api
```

Audience restriction limits cross-service replay.

---

## Bind identity to immutable runtime properties

Prefer evidence that ordinary workload operators cannot freely forge.

Examples:

* ServiceAccount UID rather than only name
* Pod UID
* Signed scheduler state
* Node attestation
* Controlled namespace
* Image digest
* Hardware key

The correct combination depends on threat model.

---

## Separate production and development trust

A developer-controlled issuer should not create production identities.

```text
development root
≠
production root
```

---

## Minimize identity authority

A workload identity should grant only necessary permissions.

Examples:

```text
orders-service:
  create payment intents
  cannot issue refunds

refunds-service:
  issue approved refunds
  cannot create users
```

---

## Protect credential agents

An identity sidecar or node agent may hold or issue credentials for many workloads.

Its compromise can cross workload boundaries.

Protect it with:

* Restricted socket access
* Minimal privileges
* Mandatory confinement
* Separate namespaces
* Secure update path
* Audit logging
* Narrow registration policy

---

## Avoid identity through headers alone

Headers such as:

```http
X-Workload-Identity: orders-service
```

are trustworthy only if:

* A trusted component creates them
* Client copies are stripped
* Backend accepts traffic only from that component
* Connection between components is protected
* Direct bypass is impossible

---

# 11.65 Performance considerations

Workload authentication can affect:

* Connection establishment
* Certificate issuance
* Token exchange
* Policy calls
* Key operations
* Trust-bundle fetches
* Secret retrieval
* Database connection creation

---

## TLS handshake cost

A new mTLS connection may involve:

* TCP setup
* TLS key exchange
* Certificate transmission
* Two certificate validations
* Private-key signatures
* Authorization mapping

Connection reuse reduces repeated handshake work.

Overly long connection reuse can delay identity refresh and revocation.

---

## Certificate-validation caching

A proxy may cache:

```text
certificate chain
→ validated peer identity
```

Risks:

* Revocation delay
* Trust-bundle rotation delay
* Incorrect cache key
* Cross-tenant reuse
* Policy changes not reflected

Cache lifetime should not exceed the security model’s acceptable staleness.

---

## Issuer capacity

If 100,000 workloads renew 30-minute certificates at the same instant:

```text
renewal storm
```

Mitigations:

* Renewal jitter
* Local agent caching
* Intermediate issuers
* Streaming updates
* Regional issuance
* Backoff
* Capacity monitoring

Do not wait until the final seconds before expiry.

---

## Token-exchange latency

Service-to-service request path:

```text
request
→ discover token expired
→ contact token issuer
→ obtain new token
→ retry request
```

A token cache or proactive refresh can reduce latency.

The cache must be:

* Keyed by audience and scope
* Bound to subject
* Expiry-aware
* Thread-safe
* Protected from leakage

---

# 11.66 Simplified model versus exact system

## Simplified model

```text
Service gets certificate
→ services use mTLS
→ caller authenticated
→ policy decides access
```

## More exact implementation

```text
Scheduler starts workload
→ node runtime records metadata
→ identity agent inspects kernel/runtime state
→ registration policy maps state to identity
→ issuer signs short-lived credential
→ trust bundle distributed
→ workload or proxy loads key and certificate
→ DNS/service discovery chooses destination
→ TCP connection established
→ TLS negotiates algorithms
→ server chain validated
→ server identity matched
→ client certificate requested
→ client signs handshake transcript
→ client chain validated
→ client identity extracted
→ transport identity mapped to application principal
→ user-delegation token validated
→ authorization policy evaluated
→ downstream database identity authenticated
→ audit events correlated
→ certificates and trust bundles rotate
```

The simplified model is useful for architecture discussions.

The exact model is required when:

* Only some replicas fail
* Rotation breaks connections
* Trust domains federate
* Proxies terminate mTLS
* User identity is delegated
* Cloud credentials are exchanged
* Kubernetes ServiceAccounts are involved
* Revocation must be rapid

---

# 11.67 Questions: Infrastructure Identity

1. What is the difference between a workload identity and its current credential?
2. Why does an identity issuer need trusted runtime evidence before issuing a credential?
3. How can a scheduler label become a security vulnerability?
4. Why does a short-lived certificate not guarantee immediate revocation?
5. How can an existing HTTP/2 connection remain active after its certificate expires?
6. Which component should close long-lived connections after a workload is disabled?
7. Why does a valid certificate chain not prove the peer is the expected service?
8. What is the difference between trust-anchor validation and hostname or URI identity validation?
9. Why should production and development use separate trust domains?
10. How can an intermediate CA rotation cause only some replicas to fail?
11. What data should be included in a workload-certificate issuance audit event?
12. Why should a node identity not automatically grant every hosted Pod the same authority?
13. How does audience validation reduce bearer-token replay?
14. What is the confused-deputy risk in a credential-broker service?
15. Why may a service mesh authenticate a connection while leaving the backend vulnerable to local impersonation?
16. How can direct backend reachability defeat a proxy-injected identity header?
17. Why should end-user subject and calling workload actor both appear in audit logs?
18. What is lost when a frontend forwards one broad user token to every backend?
19. How does token exchange improve audience and scope containment?
20. Why must a message consumer validate both producer identity and message semantics?
21. How can a broker-authenticated producer still forge a `requested_by` field?
22. Why does encrypting queue transport not necessarily protect stored messages from broker administrators?
23. How does a dynamic database credential change the impact of a secret-store leak?
24. Why can database connection pooling complicate per-user authorization context?
25. What local kernel information can help authenticate a process connecting to a Unix-domain socket?
26. Why can user namespaces complicate peer-credential interpretation?
27. Which controls protect an identity agent that can issue credentials to several workloads?
28. What failure occurs when certificate and private key updates are not atomic?
29. Why should renewal include randomized jitter?
30. How would you distinguish a TLS trust failure from an application authorization failure?
31. Which Kubernetes identity does a Pod use when `serviceAccountName` is omitted?
32. Why is granting broad access to a namespace’s default ServiceAccount dangerous?
33. What is the difference between ServiceAccount authentication and RBAC authorization?
34. Why can `create pods` indirectly grant access to another ServiceAccount’s permissions?
35. How can permission to read Secrets become an authentication compromise?
36. Why should Pods that do not call the Kubernetes API disable token automounting?
37. What is the purpose of the audience claim in a projected ServiceAccount token?
38. Why should a ServiceAccount token issued for the API server be rejected by an unrelated internal API?
39. What is the difference between validating a Kubernetes token locally and using TokenReview?
40. How would API-server unavailability affect each validation model?
41. How could deleting a Pod affect a token bound to that Pod?
42. Why does copying the token before Pod deletion remain an incident even if it soon becomes invalid?
43. Which cloud trust-policy condition should bind a role to one Kubernetes workload?
44. Why is a wildcard subject condition dangerous in workload federation?
45. What does a SPIFFE trust domain represent?
46. Why does a SPIFFE ID path need an organization-defined semantic contract?
47. How does the Workload API avoid requiring another bootstrap password?
48. Which local isolation failure could allow one workload to obtain another workload’s SVID?
49. Why might X.509-SVIDs be preferable to bearer JWTs for direct service connections?
50. What should happen when no trusted bundle exists for a foreign SPIFFE trust domain?

# 12. Kerberos, LDAP, and Enterprise Directory Authentication

Enterprise identity environments commonly combine several systems:

```text
Directory:
  stores users, groups, machines, and attributes

Kerberos:
  proves identities to network services

LDAP:
  queries and modifies directory information

PAM:
  connects Linux login applications to authentication policy

NSS:
  maps account names to Unix UIDs and groups

GSS-API and SASL:
  allow applications to use mechanisms such as Kerberos
```

These components solve related but different problems.

A recurring debugging mistake is treating “the directory” as one indivisible authentication system.

---

# 12.1 Core distinction

## Directory service

A directory answers questions such as:

```text
Does Alice exist?
What is Alice's email?
Which groups contain Alice?
What Unix UID maps to Alice?
What is the service account's distinguished name?
```

LDAP is commonly used to access directory data.

---

## Authentication service

An authentication service answers:

```text
Can Alice prove control of her credential?
Can reports-service prove its service identity?
```

Kerberos is commonly used for this purpose.

---

## Authorization service

Authorization answers:

```text
May Alice access this server?
May Alice read this directory entry?
May reports-service impersonate Alice?
```

Authorization may use:

* LDAP groups
* Kerberos authorization data
* Local application roles
* Filesystem permissions
* Sudo policy
* Service ACLs
* Database permissions

---

## Combined architecture

```mermaid
flowchart LR
    U[Alice]
    K[Kerberos KDC]
    L[LDAP Directory]
    S[Network Service]
    P[Local Authorization Policy]

    U -->|Authenticate and obtain tickets| K
    U -->|Present service ticket| S
    S -->|Query user and group attributes| L
    S -->|Subject + attributes + action| P
    P -->|Allow or deny| S
```

A successful Kerberos ticket proves an authenticated principal.

It does not automatically answer every application authorization question.

---

# 12.2 Kerberos mental model

Kerberos is a trusted-third-party network authentication protocol.

Instead of sending a password to every service:

```text
Alice → password → file server
Alice → password → email server
Alice → password → database
```

Alice authenticates to a central Key Distribution Center and receives tickets:

```text
Alice → proves identity to KDC
KDC → issues Ticket-Granting Ticket
Alice → requests service-specific ticket
Alice → presents service ticket to target service
```

The target service does not need Alice’s password.

Kerberos V5 defines a Key Distribution Center that maintains principals and long-term keys, issues Ticket-Granting Tickets and service tickets, and allows applications to use Kerberos directly or through an abstraction such as GSS-API.

---

## 12.3 Main Kerberos components

| Component                      | Responsibility                                           |
| ------------------------------ | -------------------------------------------------------- |
| Principal                      | Named Kerberos identity                                  |
| Realm                          | Administrative Kerberos identity domain                  |
| KDC                            | Trusted ticket-issuing service                           |
| Authentication Server, or AS   | Issues initial Ticket-Granting Ticket                    |
| Ticket-Granting Server, or TGS | Issues service tickets                                   |
| Ticket-Granting Ticket, or TGT | Credential used to request service tickets               |
| Service ticket                 | Credential intended for one service principal            |
| Authenticator                  | Fresh client-created proof sent with a ticket            |
| Credential cache               | Client-side storage for tickets and session keys         |
| Keytab                         | Storage containing long-term keys for service principals |
| Replay cache                   | Tracks reused authenticators where implemented           |
| Service principal              | Kerberos identity of a network service                   |

The AS and TGS are logical roles and are commonly implemented by the same KDC service.

---

# 12.4 Realm

A Kerberos realm is an administrative namespace.

Example:

```text
EXAMPLE.INTERNAL
```

A user principal:

```text
alice@EXAMPLE.INTERNAL
```

A service principal:

```text
HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

A ticket-granting service principal:

```text
krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL
```

Realm names are commonly written in uppercase by convention, but implementations and identity mappings must follow configured semantics rather than assume that changing case is always harmless.

---

## Realm is not necessarily a DNS domain

A common mapping is:

```text
DNS domain:
  example.internal

Kerberos realm:
  EXAMPLE.INTERNAL
```

But this relationship comes from configuration.

A single DNS domain could use:

* More than one realm
* A differently named realm
* Cross-realm trust
* Explicit host-to-realm mappings

Do not derive realm security decisions from capitalization alone.

---

# 12.5 Principal names

A principal identifies a Kerberos subject.

Common forms:

```text
alice@EXAMPLE.INTERNAL
```

```text
host/server01.example.internal@EXAMPLE.INTERNAL
```

```text
HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

```text
postgres/db01.example.internal@EXAMPLE.INTERNAL
```

A service principal often contains:

```text
service name / instance @ realm
```

The instance is commonly a fully qualified hostname.

MIT Kerberos documentation distinguishes user principals from service principals whose instance commonly identifies the host providing the service.

---

## Principal is not a DNS connection

These may all differ:

```text
URL:
  https://reports.example.internal

DNS result:
  192.0.2.80

Load balancer:
  lb-prod-17.example.internal

Kerberos service principal:
  HTTP/reports.example.internal@EXAMPLE.INTERNAL

Backend host:
  reports-node-3.example.internal
```

The client and service must agree on which service principal protects the logical service.

---

# 12.6 Long-term keys and session keys

Kerberos uses several categories of cryptographic keys.

## User long-term key

Traditionally derived from or otherwise associated with the user’s password.

The KDC stores the user principal’s long-term key representation.

The client derives or obtains the matching key during initial authentication.

---

## Service long-term key

Associated with a service principal.

The KDC has a copy in its principal database.

The service has a copy in a keytab or equivalent protected key store.

```text
KDC:
  key for HTTP/reports.example.internal

Reports service:
  matching key in keytab
```

---

## Session key

A temporary key generated for a ticket exchange.

Examples:

* Client–TGS session key
* Client–service session key

Session keys reduce repeated use of long-term keys.

```text
Long-term key:
  establishes or protects initial credential

Temporary session key:
  protects current relationship
```

---

# 12.7 Kerberos ticket mental model

A service ticket conceptually contains:

```text
Client principal
Service principal
Client–service session key
Start time
Expiry time
Ticket flags
Authorization data
Other policy information
```

The ticket is encrypted for the target service.

```text
Ticket for reports-service
→ encrypted with reports-service long-term key
```

Alice can carry the ticket but cannot alter its protected contents.

Only a service with the correct long-term key can decrypt it.

---

## Client-side ticket response

The KDC also sends the client information such as:

```text
Client–service session key
Ticket lifetime
Service principal
Flags
```

This client portion is protected using a key already shared with the client.

The client therefore receives:

```text
Opaque service ticket for target
+
session key client can use with target
```

---

# 12.8 Authenticator

A ticket alone can be copied and replayed.

Kerberos therefore commonly combines a ticket with a fresh **authenticator**.

Conceptually:

```text
Authenticator:
  client principal
  current timestamp
  optional subkey
  optional sequence information
```

It is encrypted or integrity-protected using the session key associated with the ticket.

```text
Ticket:
  long-enough-lived credential

Authenticator:
  fresh proof for this connection or request
```

The service verifies:

1. Ticket is valid.
2. Ticket is intended for this service.
3. Authenticator can be verified using the ticket session key.
4. Authenticator identity matches the ticket.
5. Timestamp is within allowed clock skew.
6. Authenticator has not already been accepted.

---

# 12.9 Concrete example

We will follow:

> Alice authenticates to Kerberos and accesses the protected web service `reports.example.internal`.

Principals:

```text
User:
  alice@EXAMPLE.INTERNAL

Ticket-granting service:
  krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL

Web service:
  HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

Components:

```mermaid
flowchart LR
    A[Alice's workstation]
    AS[Authentication Server]
    TGS[Ticket-Granting Server]
    W[Reports Web Service]
    KC[(Credential Cache)]
    KT[(Service Keytab)]

    A --> AS
    A --> TGS
    A --> W
    A --> KC
    W --> KT
```

---

# 12.10 Stage 1: Alice requests an initial ticket

Alice runs:

```bash
kinit alice@EXAMPLE.INTERNAL
```

The Kerberos client sends an initial Authentication Service request:

```text
AS-REQ
```

Conceptual fields:

```text
Client:
  alice@EXAMPLE.INTERNAL

Requested service:
  krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL

Requested lifetime:
  site or client policy

Supported cryptographic algorithms:
  client-supported set

Preauthentication:
  proof of access to Alice's credential
```

`kinit` obtains and caches an initial Ticket-Granting Ticket for the requested principal.

---

# 12.11 Kerberos preauthentication

Without preauthentication, a KDC might return password-derived encrypted material to anyone claiming to be Alice.

That can allow offline password guessing against captured responses.

A common preauthentication mechanism requires Alice’s client to demonstrate access to the long-term key first.

Simplified password-based preauthentication:

```text
Current timestamp
→ encrypted using Alice's password-derived key
→ sent to KDC
```

The KDC:

1. Looks up Alice’s long-term key.
2. Decrypts the preauthentication data.
3. Checks freshness.
4. Confirms Alice’s client had the appropriate key.
5. Issues the initial response.

Kerberos V5 supports extensible preauthentication mechanisms; deployments may use passwords, PKINIT, hardware-backed methods, or other mechanisms rather than one universal password flow.

---

## Simplified password path

```mermaid
sequenceDiagram
    actor Alice
    participant Client as Kerberos Client
    participant AS as KDC Authentication Server

    Alice->>Client: Enter password
    Client->>Client: Derive long-term key
    Client->>AS: AS-REQ + preauthentication proof
    AS->>AS: Look up Alice's principal key
    AS->>AS: Validate preauthentication
    AS-->>Client: AS-REP containing TGT
    Client->>Client: Decrypt client portion
    Client->>Client: Store TGT in credential cache
```

The password itself does not need to be sent to the target application service.

---

# 12.12 Stage 2: KDC issues the TGT

The AS creates:

```text
Ticket-Granting Ticket
```

Conceptually:

```text
TGT contents:
  client = alice@EXAMPLE.INTERNAL
  service = krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL
  client–TGS session key
  start time
  end time
  flags
  authorization data
```

The TGT is encrypted using the ticket-granting service’s long-term key.

```text
Only KDC/TGS
can normally decrypt or create valid TGT contents
```

The client carries the TGT but does not need to decrypt its protected body.

---

## Client portion of AS reply

The client also receives:

```text
Client–TGS session key
TGT metadata
```

protected using Alice’s long-term key or another key established by preauthentication.

If Alice typed the wrong password:

```text
Client cannot correctly use protected AS reply
or
preauthentication fails
```

No usable TGT is stored.

---

# 12.13 Credential cache

After `kinit`, Alice’s workstation stores:

```text
TGT
Client–TGS session key
Ticket metadata
```

in a **credential cache**, often abbreviated `ccache`.

Possible cache implementations include:

* File
* Kernel keyring
* Process memory
* Collection managed by a credential service
* Platform-specific cache

The exact default varies by operating system and Kerberos implementation.

MIT Kerberos uses `KRB5CCNAME` to select a credential cache and supports several cache types.

---

## Credential cache is sensitive

A stolen credential cache may let an attacker use Alice’s tickets without knowing Alice’s password.

This is often described as:

```text
Pass the ticket
```

The attacker may be limited by:

* Ticket expiry
* Ticket flags
* Network access
* Service authorization
* Credential-cache protections
* Channel binding
* Host confinement

But the tickets are active credentials.

---

# 12.14 Inspect tickets with `klist`

```bash
klist
```

### What it does

Lists the default credential cache and its Kerberos tickets.

### Why it is useful

It answers:

* Which client principal is active?
* Is a TGT present?
* When does it expire?
* Which service tickets have been obtained?
* Which realm issued them?

### Illustrative output

```text
Ticket cache: FILE:/tmp/krb5cc_1001
Default principal: alice@EXAMPLE.INTERNAL

Valid starting       Expires              Service principal
31/07/2026 16:10:00  01/08/2026 02:10:00  krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL
```

`klist` lists tickets in a credentials cache and can also inspect keytab entries.

### Safety

Ticket metadata is less sensitive than the ticket bytes, but it reveals:

* Identity names
* Service usage
* Realms
* Ticket lifetimes
* Internal hostnames

Do not copy credential-cache files into support tickets.

---

## Show flags and encryption types

```bash
klist -ef
```

### Important flags

* `-e`: display encryption types
* `-f`: display ticket flags

Illustrative output:

```text
31/07/2026 16:10:00  01/08/2026 02:10:00
krbtgt/EXAMPLE.INTERNAL@EXAMPLE.INTERNAL
    renew until 07/08/2026 16:10:00,
    Flags: FRI
    Etype: aes256-cts-hmac-sha384-192, ...
```

Exact algorithms and formatting vary by implementation and policy.

Do not infer “secure” from an algorithm name alone; verify whether it is permitted by current organizational cryptographic policy.

---

# 12.15 Stage 3: Client requests a service ticket

Alice’s browser or GSS-enabled HTTP client wants:

```text
HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

The client sends a Ticket-Granting Service request:

```text
TGS-REQ
```

Conceptually:

```text
Requested service principal
TGT
Fresh authenticator protected with client–TGS session key
Requested flags and lifetime
```

The TGS:

1. Decrypts the TGT.
2. Extracts Alice’s identity and the client–TGS session key.
3. Verifies the authenticator.
4. Checks ticket lifetime and policy.
5. Looks up the reports service principal.
6. Generates a client–reports session key.
7. Issues a reports service ticket.

---

## TGS response

The TGS sends:

```text
Service ticket:
  encrypted for reports-service

Client portion:
  encrypted for Alice using client–TGS session key
```

Client now possesses:

```text
Service ticket for reports-service
+
client–reports session key
```

Alice’s password is not used again for this service-ticket request.

---

# 12.16 Request a service ticket with `kvno`

```bash
kvno HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

### What it does

Requests a service ticket for the named service principal and prints its key version number.

### Why it is useful

It tests:

* TGT availability
* KDC reachability
* Realm routing
* Service-principal existence
* Ticket issuance
* Some encryption compatibility

MIT’s `kvno` acquires a service ticket for the specified principal and reports the service key version number.

### Illustrative output

```text
HTTP/reports.example.internal@EXAMPLE.INTERNAL: kvno = 7
```

### Interpretation

The KDC issued a ticket encrypted for key version 7 of the reports service principal.

This does not prove that the reports server’s keytab currently contains version 7.

---

# 12.17 Key version number

Kerberos long-term keys can be rotated.

Each generation is associated with a key version number, or `kvno`.

Example:

```text
Service principal:
  HTTP/reports.example.internal

Old key:
  kvno 6

Current key:
  kvno 7
```

A ticket issued using kvno 7 requires the service to have the corresponding key.

---

## Key-version mismatch

```text
KDC issues ticket with kvno 7
Service keytab contains only kvno 6
→ service cannot decrypt ticket
```

Common symptom:

```text
Key version number for principal in key table is incorrect
```

Possible causes:

* Service password or key was rotated.
* New keytab was not deployed.
* Old server replica still runs.
* Wrong keytab path is configured.
* Duplicate service principal exists in another identity system.
* Load balancer reaches an outdated backend.

---

# 12.18 Keytab

A keytab stores principal names and long-term keys.

Conceptual content:

```text
Principal:
  HTTP/reports.example.internal@EXAMPLE.INTERNAL

KVNO:
  7

Encryption type:
  approved algorithm

Long-term key:
  secret bytes
```

A keytab is equivalent to a service credential.

Anyone who steals it may be able to impersonate the service or obtain credentials as that principal, depending on principal policy and usage.

---

## Inspect keytab metadata

```bash
sudo klist -kte /etc/krb5.keytab
```

### What it does

Lists keytab entries.

### Important flags

* `-k`: inspect keytab instead of ticket cache
* `-t`: display entry timestamps
* `-e`: display encryption types

### Illustrative output

```text
Keytab name: FILE:/etc/krb5.keytab
KVNO Timestamp           Principal
---- ------------------- ---------------------------------------------------
   7 31/07/2026 15:40:00 HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

### Safety

Do not use:

```bash
klist -K
```

on a sensitive keytab unless you explicitly intend to display raw key material.

MIT documentation notes that `-K` displays key values.

---

## Keytab permissions

Inspect:

```bash
stat /etc/krb5.keytab
```

Typical intent:

```text
Readable only by root
or
by the exact service account requiring it
```

World-readable keytabs are critical credential exposures.

---

# 12.19 Stage 4: Client presents the service ticket

The client sends an application request containing a Kerberos application exchange:

```text
AP-REQ
```

Conceptual contents:

```text
Service ticket
Fresh authenticator
Optional flags
Optional subkey
```

The reports service:

1. Selects its service key from the keytab.
2. Decrypts the ticket.
3. Obtains Alice’s principal and the client–service session key.
4. Verifies the authenticator.
5. Checks time and replay state.
6. Confirms ticket is for the expected service.
7. Creates an authenticated application context.

```mermaid
sequenceDiagram
    participant Client as Alice's Client
    participant Service as Reports Service
    participant Keytab

    Client->>Service: AP-REQ: service ticket + authenticator
    Service->>Keytab: Load matching principal, kvno, and key
    Keytab-->>Service: Service long-term key
    Service->>Service: Decrypt ticket
    Service->>Service: Extract client–service session key
    Service->>Service: Verify authenticator and freshness
    Service-->>Client: Authentication result or mutual AP-REP
```

---

# 12.20 Mutual authentication

Kerberos can support mutual authentication.

The service returns an application reply:

```text
AP-REP
```

protected with the shared session key.

This allows the client to confirm:

```text
The server could decrypt my service ticket
and
possesses the expected service key
```

Without mutual authentication, the service authenticates the client, but the application may rely on another mechanism for server authentication.

---

# 12.21 Kerberos does not automatically encrypt application data

Kerberos authenticates principals and establishes shared session keys.

Applications can use those keys for:

* Integrity protection
* Confidentiality
* Sequence protection

But an application may use Kerberos only during connection setup and then transmit data without Kerberos message protection.

RFC 4120 explicitly notes that if an application does not apply encryption or integrity protection to the subsequent stream, Kerberos identity verification may apply only to connection initiation rather than every later message.

Applications may combine:

```text
Kerberos authentication
+
TLS transport encryption
```

or use GSS-API confidentiality and integrity services.

---

# 12.22 GSS-API

The Generic Security Service Application Program Interface, or GSS-API, provides a mechanism-independent authentication interface.

Application code asks for operations such as:

```text
Initiate security context
Accept security context
Wrap message
Unwrap message
Get peer name
Delegate credential
```

A configured mechanism supplies the protocol details.

Possible mechanisms can include:

* Kerberos
* Other platform-supported GSS mechanisms

GSS-API itself is an abstraction; Kerberos V5 is one defined GSS-API mechanism.

---

## Simplified GSS context flow

```mermaid
sequenceDiagram
    participant C as Initiator Application
    participant CG as Client GSS-API
    participant SG as Server GSS-API
    participant S as Acceptor Application

    C->>CG: Initialize context for service name
    CG-->>C: Output token
    C->>S: Send token
    S->>SG: Accept context token
    SG-->>S: Response token + authenticated principal
    S->>C: Send response token
    C->>CG: Continue context
    CG-->>C: Context established
```

The application may not parse raw Kerberos protocol messages itself.

---

## Initiator and acceptor

### Initiator

The side starting the security context.

Example:

```text
Alice's client
```

### Acceptor

The target service accepting authentication.

Example:

```text
reports-service
```

The acceptor needs access to the appropriate service key, commonly through a keytab.

---

## GSS name versus local account

GSS may authenticate:

```text
alice@EXAMPLE.INTERNAL
```

The application may map it to:

```text
Local Unix account:
  alice

Application user:
  u_1048
```

This mapping is a separate security decision.

Do not remove the realm suffix blindly if multiple realms are trusted.

---

# 12.23 GSS integrity and confidentiality

After a context is established, an application can request:

```text
Integrity:
  detect modification

Confidentiality:
  encrypt protected messages

Replay detection:
  detect reused messages

Sequence protection:
  detect reordering
```

The negotiated mechanism and application must agree on which services are required.

A context that authenticates a peer but does not protect messages may still be exposed to modification elsewhere in the stack.

---

# 12.24 Channel binding

Channel binding connects application authentication to properties of an underlying secure channel.

Conceptual goal:

```text
Kerberos/GSS authentication context
is bound to
this TLS connection
```

This can reduce credential forwarding or man-in-the-middle risks in some protocol designs.

Both ends must:

* Compute compatible channel-binding data.
* Pass it to the authentication mechanism.
* Enforce failure when binding is required.

A configuration where one side treats binding as optional and the other assumes it is required can create compatibility or security failures.

---

# 12.25 SASL

The Simple Authentication and Security Layer, or SASL, allows application protocols to negotiate authentication mechanisms.

Protocols using SASL may support mechanisms such as:

```text
GSSAPI
EXTERNAL
SCRAM
PLAIN
```

SASL does not itself define one universal credential format.

It provides a framework in which a protocol selects and performs a mechanism.

The `GSSAPI` SASL mechanism uses the Kerberos V5 GSS-API mechanism.

---

## Protocol layering

```text
LDAP
  uses SASL
    uses GSS-API
      uses Kerberos
        obtains tickets from KDC
```

```mermaid
flowchart TD
    L[LDAP Protocol]
    S[SASL Framework]
    G[GSS-API]
    K[Kerberos V5]
    T[Kerberos Tickets]

    L --> S --> G --> K --> T
```

When an LDAP client reports:

```text
SASL/GSSAPI authentication failure
```

the failure might originate from:

* LDAP mechanism negotiation
* GSS name construction
* Kerberos ticket acquisition
* Service-principal mismatch
* Keytab failure
* Channel binding
* LDAP authorization mapping

---

# 12.26 Credential delegation

A client may delegate Kerberos credentials to a service.

Example:

```text
Alice
→ authenticates to SSH server
→ SSH server needs to access file server as Alice
```

Without delegation:

```text
SSH server knows Alice authenticated
but
does not have Alice's credentials for file server
```

With delegation:

```text
SSH server receives delegated authority
→ obtains or uses downstream tickets as Alice
```

This is powerful and risky.

---

## Forwarded TGT

A traditional forwarding model gives the remote host a forwarded form of Alice’s TGT.

The remote host can then request other service tickets as Alice.

MIT documentation warns that forwardable credentials should not be sent to untrusted machines because those machines then possess the user’s tickets.

---

## Delegation risk

```text
Alice forwards ticket to server
→ server compromised
→ attacker steals delegated ticket
→ attacker accesses downstream services as Alice
```

Use delegation only when:

* The downstream action is required.
* The intermediate service is trusted.
* Credential lifetime is limited.
* Delegation scope is constrained where possible.
* Audit records preserve the delegation chain.

---

## Constrained delegation

A constrained model limits which downstream services an intermediary may access.

Conceptual policy:

```text
frontend-service
may act on behalf of users
only to
reports-service
```

It should not obtain user tickets for:

```text
directory-admin
database-admin
all internal services
```

Constrained delegation reduces authority but makes identity and policy configuration more complex.

---

# 12.27 Kerberos ticket flags

Common ticket properties include:

| Flag              | General meaning                                                  |
| ----------------- | ---------------------------------------------------------------- |
| Initial           | Issued through initial authentication                            |
| Pre-authenticated | Preauthentication was performed                                  |
| Forwardable       | Can be used to obtain forwarded credentials                      |
| Forwarded         | Ticket represents forwarded credentials                          |
| Renewable         | Can be renewed within a renewable lifetime                       |
| Proxiable         | Can support proxy-style ticket use                               |
| Renewable         | Ticket lifetime may be extended within policy                    |
| Postdated         | Intended to become valid later                                   |
| Okay-as-delegate  | KDC indicates service may be trusted for delegation under policy |

Exact security meaning depends on the application, KDC policy, and implementation.

A flag is an input to policy, not a command forcing an application to delegate.

---

# 12.28 Ticket lifetime

A ticket can have:

```text
Start time
End time
Renew-until time
```

Example:

```text
Valid:
  10:00–20:00

Renewable until:
  Friday 20:00
```

A renewable ticket can be extended without re-entering the original credential, but only within its renewable lifetime and KDC policy.

MIT `kinit -R` requests renewal; an expired ticket cannot generally be renewed after its renewable conditions are no longer satisfied.

---

## Request a specific ticket lifetime

```bash
kinit -l 8h alice@EXAMPLE.INTERNAL
```

### What it does

Requests an eight-hour initial ticket lifetime.

### Important flag

* `-l 8h`: requested lifetime

### Important limitation

The KDC can issue a shorter lifetime according to policy.

A client request does not override KDC limits.

---

## Request a renewable ticket

```bash
kinit -l 8h -r 7d alice@EXAMPLE.INTERNAL
```

### What it does

Requests:

```text
Initial lifetime:
  8 hours

Renewable lifetime:
  7 days
```

### Safety

Long renewable periods increase the duration for which stolen credentials may remain useful.

Use the shortest lifetime that supports the operational requirement.

---

# 12.29 Renew a TGT

```bash
kinit -R
```

### What it does

Requests renewal of the current renewable TGT.

### Why it is useful

It supports long-running sessions without re-entering the user credential each time.

### Failure causes

* Ticket not renewable
* Renewable lifetime ended
* KDC unreachable
* Principal disabled
* Policy changed
* Cache points to another principal
* Clock problem

---

# 12.30 Destroy tickets

```bash
kdestroy
```

### What it does

Destroys the default credential cache.

MIT documents `kdestroy` as deleting the active Kerberos tickets in the selected credential cache.

### Why it is useful

It reduces the chance that another process or person can reuse cached tickets after the user is finished.

### Important limitation

It does not necessarily:

* Revoke copies stolen earlier
* Terminate already-established service sessions
* Delete tickets in every cache collection
* Invalidate server-side application sessions
* Change the Kerberos password

---

## Destroy all caches in a collection

```bash
kdestroy -A
```

### Important flag

* `-A`: destroy all caches in the cache collection where supported

### Safety

This may terminate access for multiple active identities or applications in the current login session.

---

# 12.31 Service authentication with a keytab

A non-human service may obtain initial credentials using a keytab:

```bash
kinit \
  -k \
  -t /etc/reports-api.keytab \
  HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

### What it does

Requests a TGT using a key from the specified keytab instead of an interactively entered password.

### Important flags

* `-k`: use keytab
* `-t FILE`: select keytab file

MIT `kinit` supports acquiring initial credentials from a keytab using `-k` and `-t`.

### Why it is useful

It supports:

* Services needing outbound Kerberos credentials
* Automated jobs
* Controlled service-to-service authentication

### Safety

A keytab is a long-term credential.

Do not place it in:

* Container images
* Source repositories
* CI artifacts
* World-readable locations
* Unencrypted backups

---

# 12.32 Kerberos tracing

MIT Kerberos supports trace output through `KRB5_TRACE`.

Example:

```bash
KRB5_TRACE=/dev/stderr \
  kinit alice@EXAMPLE.INTERNAL
```

### What it does

Emits Kerberos library trace information to standard error while running `kinit`.

MIT documentation identifies `KRB5_TRACE` as the destination for Kerberos trace output.

### Why it is useful

It can show:

* Configuration file selection
* Realm discovery
* KDC addresses
* Preauthentication negotiation
* Encryption-type selection
* Cache behavior
* Referral processing
* Protocol errors

### Safety

Trace output may expose:

* Principal names
* Realm names
* Hostnames
* Ticket metadata
* Internal topology
* Preauthentication details

Do not share unredacted traces publicly.

---

# 12.33 Kerberos DNS discovery

A realm may advertise KDC services through DNS SRV records.

Inspect:

```bash
dig +short SRV _kerberos._udp.EXAMPLE.INTERNAL
```

and:

```bash
dig +short SRV _kerberos._tcp.EXAMPLE.INTERNAL
```

### What it does

Queries DNS for Kerberos service-location records.

### Why it is useful

It helps diagnose:

* Missing KDC discovery
* Wrong priority or port
* TCP versus UDP reachability
* Realm DNS misconfiguration

### Important limitation

DNS discovery must be interpreted according to the Kerberos client’s trust and configuration model.

A DNS answer identifies a network destination; it does not alone establish that the destination is a trustworthy KDC.

---

# 12.34 Clock synchronization

Kerberos authenticators and tickets depend heavily on time.

Typical failure:

```text
Clock skew too great
```

Possible causes:

* Client clock wrong
* KDC clock wrong
* Service clock wrong
* VM resumed with stale time
* Container inherits incorrect host time
* NTP unavailable
* Time zone displayed incorrectly even though UTC clock is correct

The important comparison is the underlying time value, not merely the displayed local time zone.

---

## Observe system time

```bash
timedatectl status
```

### What it does

On systemd-based Linux systems, displays:

* Local time
* Universal time
* Time zone
* Clock synchronization status
* NTP state

### Why it is useful

It provides an initial indication of time-health problems.

### Limitation

It does not prove that all machines in the exchange agree within required skew.

Compare client, KDC, and service systems.

---

# 12.35 Replay cache

A Kerberos service may record recently accepted authenticators.

Conceptual entry:

```text
Client principal
Server principal
Timestamp
Microseconds
Checksum or authenticator identity
Expiry
```

A second identical use is rejected.

Possible failure:

```text
Request is a replay
```

Causes can include:

* Actual credential replay
* Application retry reusing the same authenticator incorrectly
* Shared replay-cache configuration failure
* Cloned virtual machines with duplicate state
* Load-balanced servers with inconsistent replay handling
* Incorrect time
* Container filesystem problems

Disabling replay protection to remove errors can create a real authentication vulnerability.

---

# 12.36 Cross-realm authentication

Alice may belong to:

```text
ENGINEERING.EXAMPLE
```

while the service belongs to:

```text
FINANCE.EXAMPLE
```

Cross-realm trust allows a chain of ticket-granting relationships.

Conceptual path:

```text
alice@ENGINEERING.EXAMPLE
→ ticket for FINANCE.EXAMPLE TGS
→ service ticket in FINANCE.EXAMPLE
```

```mermaid
flowchart LR
    A[Alice in Engineering Realm]
    KE[Engineering KDC]
    KF[Finance KDC]
    S[Finance Service]

    A --> KE
    KE -->|Cross-realm TGT| A
    A --> KF
    KF -->|Finance service ticket| A
    A --> S
```

Kerberos V5 supports cross-realm operation and referrals, but the trust path and name mapping must be explicitly understood.

---

## Cross-realm is not automatically transitive authorization

A finance service may authenticate:

```text
alice@ENGINEERING.EXAMPLE
```

but still deny access because:

* Foreign realm is not authorized.
* No local account mapping exists.
* Alice lacks a finance group.
* Transited-realm policy fails.
* Principal name conflicts with a local user.
* Application strips the realm incorrectly.

---

# 12.37 Kerberos authentication versus authorization data

A ticket may contain authorization-related information.

Examples can include:

* Group membership data
* Privilege attributes
* Authentication indicators
* Delegation restrictions
* Platform-specific authorization data

The service must:

1. Verify the ticket.
2. Understand the authorization-data format.
3. Trust the issuer of that data.
4. Apply local authorization policy.
5. Handle stale or oversized data.

A ticket authenticating Alice does not inherently grant Unix UID 0, database ownership, or application administration.

---

# 12.38 Common Kerberos failures

## `Client not found in Kerberos database`

Interpretation:

```text
Requested client principal
does not exist in selected realm
```

Possible causes:

* Wrong realm
* Typo
* Account not provisioned
* Realm mapping error
* Principal alias not configured
* Client contacted wrong KDC

MIT documentation uses this error as the example of an unregistered Kerberos principal.

---

## `Server not found in Kerberos database`

Interpretation:

```text
Requested service principal
does not exist in KDC database
```

Possible causes:

* Wrong hostname
* Wrong service class
* DNS alias mismatch
* Service principal not provisioned
* Wrong realm
* Client canonicalized name unexpectedly

---

## `Password incorrect while getting initial credentials`

Possible causes:

* Wrong password
* Wrong principal
* Wrong realm
* Password recently changed and replication delayed
* Unsupported preauthentication mechanism
* Account disabled
* Client using stale cached configuration

---

## `Clock skew too great`

Check:

```text
Client time
KDC time
Service time
```

Do not only check the workstation.

---

## `Cannot contact any KDC`

Possible causes:

* DNS failure
* Firewall
* Wrong realm configuration
* KDC outage
* UDP fragmentation or filtering
* TCP unavailable
* Proxy/network path issue
* IPv4/IPv6 mismatch

---

## `Key table entry not found`

The service could not find a keytab entry matching:

* Requested service principal
* Required encryption type
* Required key version
* Configured keytab location

---

## `Key version number is incorrect`

KDC and service disagree on the current service key generation.

---

## `Ticket expired`

The cached ticket is outside its validity interval.

Possible response:

```bash
kinit
```

or renewal if valid and policy permits:

```bash
kinit -R
```

---

## `Credential cache not found`

Possible causes:

* User never ran `kinit`
* Cache was destroyed
* `KRB5CCNAME` points elsewhere
* Login session created another cache
* Container cannot access host cache
* File permissions deny access
* Cache collection selected another principal

---

# 12.39 Kerberos troubleshooting order

```mermaid
flowchart TD
    A[Kerberos operation failed]
    B{Principal and realm correct?}
    C[Check configuration and explicit principal]
    D{KDC reachable?}
    E[Check DNS, ports, routing, trace]
    F{Initial TGT available?}
    G[Run kinit and inspect klist]
    H{Service ticket obtainable?}
    I[Run kvno for exact SPN]
    J{Service can decrypt ticket?}
    K[Inspect keytab principal, kvno, enctype]
    L{Application mapping succeeds?}
    M[Inspect GSS name and local account mapping]
    N{Authorization succeeds?}
    O[Inspect groups, roles, and service ACL]
    P[Inspect message protection and application flow]

    A --> B
    B -->|No| C
    B -->|Yes| D
    D -->|No| E
    D -->|Yes| F
    F -->|No| G
    F -->|Yes| H
    H -->|No| I
    H -->|Yes| J
    J -->|No| K
    J -->|Yes| L
    L -->|No| M
    L -->|Yes| N
    N -->|No| O
    N -->|Yes| P
```

---

# 12.40 LDAP mental model

LDAP is a protocol for accessing a directory.

A directory stores entries organized in a hierarchy.

Each entry has:

* Distinguished Name
* Attributes
* Object classes
* Values
* Schema constraints

LDAP defines operations such as Bind, Search, Compare, Add, Modify, Delete, and extended operations. Requests and responses can be processed independently over a connection.

---

# 12.41 Directory Information Tree

Example directory:

```text
dc=example,dc=internal
├── ou=People
│   ├── uid=alice
│   └── uid=bob
├── ou=Groups
│   ├── cn=finance
│   └── cn=developers
└── ou=Services
    └── cn=reports-api
```

```mermaid
flowchart TD
    ROOT["dc=example,dc=internal"]
    PEOPLE["ou=People"]
    GROUPS["ou=Groups"]
    SERVICES["ou=Services"]
    ALICE["uid=alice"]
    BOB["uid=bob"]
    FIN["cn=finance"]
    DEV["cn=developers"]
    REPORTS["cn=reports-api"]

    ROOT --> PEOPLE
    ROOT --> GROUPS
    ROOT --> SERVICES
    PEOPLE --> ALICE
    PEOPLE --> BOB
    GROUPS --> FIN
    GROUPS --> DEV
    SERVICES --> REPORTS
```

This hierarchy is called a Directory Information Tree, or DIT.

---

# 12.42 Distinguished Name

A Distinguished Name, or DN, uniquely identifies an entry within a directory naming context.

Example:

```text
uid=alice,ou=People,dc=example,dc=internal
```

Components:

```text
uid=alice
ou=People
dc=example
dc=internal
```

The leftmost component is the entry’s Relative Distinguished Name relative to its parent.

---

## DN is not a filesystem path

DN parsing follows LDAP and X.500 string rules.

Special characters may require escaping.

Do not parse a DN by simply splitting on commas.

Example values can legally contain escaped commas or other special characters.

Use an LDAP library.

---

# 12.43 Attributes

Example entry:

```text
dn: uid=alice,ou=People,dc=example,dc=internal
objectClass: inetOrgPerson
objectClass: posixAccount
uid: alice
cn: Alice Example
sn: Example
mail: alice@example.internal
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/alice
loginShell: /bin/bash
```

An attribute may be:

* Single-valued or multi-valued
* Case-sensitive or case-insensitive according to matching rules
* String, number, time, DN, binary data, or another syntax
* Required or optional according to object classes

Do not compare every LDAP value using ordinary programming-language string equality.

---

# 12.44 Schema and object classes

A schema defines:

* Attribute types
* Object classes
* Required attributes
* Permitted attributes
* Value syntaxes
* Matching rules
* Naming behavior

Example:

```text
posixAccount
→ may require uidNumber, gidNumber, homeDirectory
```

An application that assumes every user entry has `uidNumber` may fail for directory users not intended to become Unix accounts.

---

# 12.45 LDAP connection and Bind

An LDAP connection begins without an automatically authenticated directory identity.

The client may perform a Bind operation.

Bind can establish or change the authentication state associated with the LDAP connection.

LDAP authentication mechanisms include simple Bind and SASL Bind, with TLS commonly used to protect credentials and data.

---

## Important mental model

```text
TCP/TLS connection
→ LDAP Bind
→ authenticated LDAP authorization identity
→ LDAP operations evaluated against directory ACLs
```

Bind does not retrieve user data by itself.

Search and other operations occur separately.

---

# 12.46 Anonymous LDAP state

Before authenticated Bind, the connection may have anonymous authorization identity.

Anonymous access may allow:

* Root DSE discovery
* Schema discovery
* Selected public attributes
* No useful data

The exact permissions are server policy.

Do not assume anonymous access is either completely disabled or completely unrestricted.

---

# 12.47 Simple Bind

A simple Bind supplies:

```text
Name
Password
```

Example name:

```text
uid=alice,ou=People,dc=example,dc=internal
```

The name is commonly a DN, though server-specific identity mapping may support other forms.

---

## Simple Bind requires channel protection

The name and password need protection from network observers.

Use:

* LDAP with StartTLS
* LDAP over a directly established TLS connection
* Another channel that provides appropriate confidentiality and server authentication

RFC 4513 recommends disabling name/password authentication when adequate data-security services are absent.

---

# 12.48 Unauthenticated simple Bind trap

An LDAP simple Bind with:

```text
non-empty name
empty password
```

can have special unauthenticated semantics rather than meaning “verify that this user has an empty password.”

A buggy application might:

1. Accept an empty password from a login form.
2. Send a Bind with Alice’s DN and empty password.
3. Receive an LDAP result it misinterprets.
4. Treat Alice as authenticated.

OpenLDAP documentation warns about applications accidentally producing unauthenticated Bind requests when authenticated access was intended and recommends not enabling such behavior generally.

Application rule:

```text
Reject empty user passwords before LDAP Bind
unless an explicitly designed mechanism requires them
```

---

# 12.49 StartTLS

StartTLS begins with a normal LDAP connection and upgrades it to TLS before credential submission.

```mermaid
sequenceDiagram
    participant Client
    participant LDAP as LDAP Server

    Client->>LDAP: TCP connection
    Client->>LDAP: StartTLS extended operation
    LDAP-->>Client: Ready for TLS
    Client->>LDAP: TLS handshake
    Client->>Client: Validate LDAP server certificate
    Client->>LDAP: Protected LDAP Bind
```

LDAP StartTLS is defined as an LDAP operation establishing TLS protection on the existing connection.

---

## Fail closed on StartTLS

The client must not continue with a password Bind if TLS upgrade fails.

OpenLDAP command-line tools distinguish:

```text
-Z:
  request StartTLS

-ZZ:
  require StartTLS and stop if it fails
```

OpenLDAP documents `-ZZ` as requiring successful StartTLS, while `-Z` can permit continued processing if TLS cannot be started.

For password authentication, prefer fail-closed behavior.

---

# 12.50 Direct LDAP over TLS

A client may connect using:

```text
ldaps://directory.example.internal
```

This establishes TLS from the beginning of the connection.

StartTLS commonly uses:

```text
ldap://directory.example.internal
```

followed by the StartTLS operation.

Both can provide protected LDAP when correctly configured and validated.

The important security properties are:

* TLS actually established
* Server certificate validated
* Expected server identity checked
* Credentials sent only after protection
* TLS failure does not downgrade silently

---

# 12.51 Search LDAP anonymously

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -x \
  -b "" \
  -s base \
  "(objectClass=*)" \
  namingContexts
```

### What it does

Requests the server’s root DSE information after requiring StartTLS.

### Important flags

* `-H URI`: LDAP server URI
* `-ZZ`: require StartTLS
* `-x`: use simple authentication rather than SASL
* `-b ""`: empty base DN, selecting root DSE
* `-s base`: search only the base object
* filter: match any object class
* `namingContexts`: request only this attribute

### Why it is useful

It discovers directory naming contexts without enumerating the directory.

### Safety

Anonymous discovery may be disabled.

Do not weaken server ACLs merely to make this diagnostic work.

---

# 12.52 Authenticated LDAP search

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -x \
  -D 'uid=directory-reader,ou=Services,dc=example,dc=internal' \
  -W \
  -b 'ou=People,dc=example,dc=internal' \
  '(uid=alice)' \
  dn uid cn mail
```

### What it does

1. Connects to the directory.
2. Requires StartTLS.
3. Prompts for the directory-reader password.
4. Binds using the provided DN.
5. Searches under `ou=People`.
6. Matches entries whose `uid` equals `alice`.
7. Returns selected attributes.

### Important flags

* `-D`: Bind DN
* `-W`: prompt for password
* `-b`: search base
* final values: requested attributes

### Why `-W` matters

It avoids placing the password directly in the command line.

Avoid:

```bash
-w 'password'
```

because command-line arguments may appear in:

* Shell history
* Process listings
* CI logs
* Terminal recordings

---

## Illustrative output

```text
dn: uid=alice,ou=People,dc=example,dc=internal
uid: alice
cn: Alice Example
mail: alice@example.internal
```

This proves only that the bound directory-reader identity could retrieve the entry.

It does not prove that Alice’s password is correct.

---

# 12.53 Search scopes

LDAP search scope can be:

| Scope                        | Meaning                                   |
| ---------------------------- | ----------------------------------------- |
| `base`                       | Search only the base entry                |
| `one`                        | Search immediate children                 |
| `sub`                        | Search the entire subtree                 |
| subordinate/subtree variants | Implementation or protocol-specific forms |

Example:

```bash
-s base
```

versus:

```bash
-s sub
```

A broad subtree search can be expensive in a large directory.

Use the narrowest base and scope that match the data model.

---

# 12.54 LDAP search filter

Example:

```text
(&(objectClass=person)(uid=alice))
```

Meaning:

```text
objectClass is person
AND
uid is alice
```

Other operators include:

```text
|
  OR

!
  NOT

*
  presence or substring depending on position
```

Example presence:

```text
(mail=*)
```

Example substring:

```text
(cn=Alice*)
```

---

# 12.55 LDAP injection

Unsafe application:

```python
filter_text = f"(uid={submitted_username})"
```

Attacker input:

```text
alice)(|(uid=*)
```

may change the filter’s structure.

Use an LDAP library’s filter-escaping function.

Do not attempt to escape using only:

```text
replace "("
replace ")"
```

LDAP filters and DNs have different escaping rules.

A value safe in a DN is not automatically safe in a search filter.

---

# 12.56 Search-first login pattern

Many applications receive a username rather than a full DN.

They use a service identity to locate the user entry.

Flow:

```mermaid
sequenceDiagram
    actor Alice
    participant App
    participant LDAP

    Alice->>App: Username + password
    App->>LDAP: Bind as directory-search service
    LDAP-->>App: Search identity accepted
    App->>LDAP: Search uid=alice
    LDAP-->>App: Alice's unique DN
    App->>LDAP: Bind as Alice using submitted password
    LDAP-->>App: Bind success or failure
    App->>LDAP: Rebind as service identity
    App->>LDAP: Fetch groups and attributes
    LDAP-->>App: Authorization data
    App->>App: Create application session
```

---

# 12.57 Search-first login step by step

## Step 1: Validate input

Reject:

* Empty password
* Oversized username
* Invalid encoding
* Ambiguous tenant
* Unexpected control characters

Apply rate limits before expensive directory operations.

---

## Step 2: Bind as search service

The application uses a narrowly privileged service account.

It should have permission to read only fields required for identity resolution.

It should not have:

* Password-change rights
* Broad administrative write access
* Access to unrelated secrets
* Permission to read password hash material

---

## Step 3: Search for user

Conceptual filter:

```text
(&(objectClass=person)(uid=alice))
```

The application should require exactly one acceptable result.

Cases:

```text
0 results:
  unknown user

1 result:
  continue

2 or more results:
  ambiguous identity; reject and investigate
```

Selecting the first result can authenticate the wrong account.

---

## Step 4: Bind as user

The application opens a separate connection or changes Bind identity:

```text
Bind DN:
  uid=alice,ou=People,dc=example,dc=internal

Password:
  submitted secret
```

Success indicates the directory authentication mechanism accepted the credential for that DN.

---

## Step 5: Restore service identity

After authenticating Alice, the application may need directory-readable group and profile attributes.

It must not assume the connection remains bound as the original service account.

Bind changes the connection’s authentication state.

Safe designs often use separate connection pools:

```text
Service-search pool:
  bound as directory reader

User-authentication connections:
  short-lived, never returned as service connections
```

This prevents connection-pool identity confusion.

---

## Step 6: Apply local policy

Directory authentication result:

```text
Alice's password accepted
```

Application policy still checks:

* Account enabled
* Application assignment
* Tenant
* Groups
* Required MFA
* Local suspension
* Role mapping
* Resource authorization

---

# 12.58 Direct Bind login pattern

Some directories support constructing a user Bind name directly.

Example:

```text
uid=alice,ou=People,dc=example,dc=internal
```

or a directory-specific login form.

Flow:

```text
Username
→ deterministic DN or authentication name
→ user Bind
```

Advantages:

* No search service account
* Fewer LDAP operations

Limitations:

* Requires predictable naming
* User may live in multiple organizational units
* Renames can change DN
* Aliases complicate mapping
* Directory-specific login names may differ from entry DNs

---

# 12.59 LDAP password comparison is not always appropriate

An application could theoretically:

```text
Read stored password representation
→ verify locally
```

This is usually undesirable because it requires access to sensitive password-verification data and duplicates directory policy.

Prefer a supported authentication operation such as:

* LDAP Bind
* Kerberos
* SASL mechanism
* Identity-provider flow

Do not grant applications read access to password hashes merely to simplify login.

---

# 12.60 SASL GSSAPI LDAP Bind

A client with a valid Kerberos ticket can authenticate to LDAP using SASL GSSAPI.

Example:

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -Y GSSAPI \
  -b 'ou=People,dc=example,dc=internal' \
  '(uid=alice)' \
  dn cn mail
```

### What it does

1. Requires StartTLS.
2. Negotiates the SASL `GSSAPI` mechanism.
3. Uses Kerberos/GSS credentials from the current credential cache.
4. Authenticates to the LDAP service.
5. Performs the search under the resulting LDAP authorization identity.

The SASL GSSAPI mechanism is defined as the use of the Kerberos V5 GSS mechanism inside SASL.

---

## Required service principal

The LDAP server may use a principal such as:

```text
ldap/directory.example.internal@EXAMPLE.INTERNAL
```

The client requests a ticket for that exact service identity.

Failure can arise if the server keytab instead contains:

```text
ldap/directory-node-1.example.internal@EXAMPLE.INTERNAL
```

while clients connect using the logical alias:

```text
directory.example.internal
```

Service-principal naming must match the service-discovery and load-balancing architecture.

---

# 12.61 SASL authorization identity

SASL may distinguish:

```text
Authentication identity:
  who proved the credential

Authorization identity:
  identity under which operations are performed
```

Common case:

```text
authcid = alice
authzid = alice
```

Proxy authorization case:

```text
authcid = directory-proxy
authzid = alice
```

The server must explicitly authorize such identity switching.

A client merely requesting another authorization identity must not be enough.

---

# 12.62 SASL EXTERNAL

The `EXTERNAL` mechanism uses identity established outside SASL.

Example:

```text
TLS client certificate authenticates peer
→ SASL EXTERNAL maps certificate identity
→ LDAP authorization identity created
```

OpenLDAP supports TLS client authentication combined with SASL EXTERNAL.

The server still needs a mapping from certificate identity to directory authorization identity.

---

# 12.63 Verify LDAP StartTLS with OpenSSL

```bash
openssl s_client \
  -connect directory.example.internal:389 \
  -starttls ldap \
  -servername directory.example.internal \
  -verify_hostname directory.example.internal \
  -verify_return_error
```

### What it does

* Connects to the LDAP port.
* Requests LDAP StartTLS.
* Performs a TLS handshake.
* Checks the expected server hostname.
* Fails on certificate verification errors.

### Why it is useful

It isolates TLS problems from LDAP Bind and search problems.

### What it does not test

* LDAP password
* Kerberos Bind
* Directory ACL
* Search filter
* Group lookup
* Application account mapping

---

# 12.64 LDAP access controls

LDAP servers apply ACLs to directory operations.

Authorization can depend on:

* Bound identity
* Target entry
* Requested attribute
* Operation type
* Connection security
* Group membership
* Network source
* Authentication strength
* Proxy authorization state

Example policy:

```text
Anonymous:
  read root DSE only

Authenticated users:
  read own profile

Managers:
  read direct-report phone numbers

Directory reader:
  search user IDs and groups

Password administrator:
  modify password attributes
```

A successful Bind does not imply unrestricted directory access.

---

# 12.65 Attribute-level authorization

An entry might contain:

```text
cn
mail
telephoneNumber
manager
employeeNumber
password policy state
authentication factors
```

The same Bind identity may be permitted to read:

```text
cn
mail
```

but denied:

```text
employeeNumber
authentication-factor metadata
```

Application errors can occur when:

```text
User search succeeds
but
required attribute is hidden by ACL
```

This is an authorization failure, not an authentication failure.

---

# 12.66 LDAP groups

Group representations vary.

Examples:

## Group containing member DNs

```text
dn: cn=finance,ou=Groups,dc=example,dc=internal
objectClass: groupOfNames
member: uid=alice,ou=People,dc=example,dc=internal
member: uid=bob,ou=People,dc=example,dc=internal
```

## User containing group names

```text
dn: uid=alice,ou=People,dc=example,dc=internal
memberOf: cn=finance,ou=Groups,dc=example,dc=internal
```

## POSIX-style group

```text
dn: cn=finance,ou=Groups,dc=example,dc=internal
objectClass: posixGroup
gidNumber: 2001
memberUid: alice
```

An application must understand the directory’s actual group model.

---

# 12.67 Nested groups

Example:

```text
Alice
→ member of finance-analysts

finance-analysts
→ member of finance-all

finance-all
→ allowed to use reports application
```

```mermaid
flowchart LR
    A[Alice]
    FA[finance-analysts]
    F[finance-all]
    APP[Reports Access]

    A -->|member| FA
    FA -->|member| F
    F --> APP
```

Challenges:

* Recursive traversal
* Cycles
* Large graphs
* Cache invalidation
* Cross-domain groups
* Maximum nesting depth
* Inconsistent group representations

---

## Nested group cycle

```text
Group A contains Group B
Group B contains Group C
Group C contains Group A
```

A naïve recursive algorithm may:

* Loop forever
* Exhaust memory
* Repeat directory queries
* Produce inconsistent membership

Use:

* Visited-set tracking
* Depth limits
* Server-supported membership operations where appropriate
* Well-defined cycle handling

---

# 12.68 Group-cache staleness

Suppose Alice is removed from:

```text
finance-admins
```

Directory update:

```text
immediate
```

Application group cache:

```text
valid for 30 minutes
```

Existing access may continue for 30 minutes.

Additional caches may exist in:

* SSSD
* NSS
* Identity provider
* Kerberos authorization data
* Application session
* API token
* Reverse proxy
* Authorization engine

Revocation analysis must identify every copy.

---

# 12.69 LDAP directory as NSS source

A Linux host may obtain:

```text
username
UID
GID
home directory
shell
supplementary groups
```

from LDAP through software such as SSSD or another NSS provider.

Flow:

```text
getent passwd alice
→ NSS
→ directory client/cache
→ LDAP directory
→ POSIX account attributes
```

Authentication might separately use Kerberos:

```text
PAM
→ Kerberos
→ KDC
```

Thus:

```text
LDAP:
  identity and group lookup

Kerberos:
  password or credential authentication
```

---

# 12.70 Combined Linux login example

Alice attempts SSH login.

```mermaid
sequenceDiagram
    actor Alice
    participant SSHD
    participant NSS
    participant LDAP
    participant PAM
    participant KDC
    participant Kernel

    Alice->>SSHD: Login as alice
    SSHD->>NSS: Resolve alice
    NSS->>LDAP: Search POSIX account entry
    LDAP-->>NSS: UID, GID, home, shell
    NSS-->>SSHD: Local account mapping

    SSHD->>PAM: Authenticate alice
    PAM->>KDC: Kerberos initial authentication
    KDC-->>PAM: Authentication result
    PAM->>LDAP: Optional account/group policy lookup
    LDAP-->>PAM: Account state and groups
    PAM-->>SSHD: Allow

    SSHD->>Kernel: Start shell using mapped UID and groups
```

Possible partial failures:

```text
Kerberos succeeds
but LDAP UID lookup fails
→ user authenticated but no usable local account
```

```text
LDAP resolves Alice
but Kerberos fails
→ account exists but credential rejected
```

```text
Both succeed
but PAM account policy denies
→ no session
```

---

# 12.71 Active Directory mental model

Microsoft Active Directory commonly integrates:

* Directory data accessible through LDAP
* Kerberos authentication
* DNS service discovery
* Machine and user accounts
* Group-based authorization data
* Domain and forest trust relationships
* Windows access-token construction

Conceptually:

```text
LDAP:
  users, computers, groups, attributes

Kerberos:
  tickets and service authentication

DNS:
  locate domain controllers and services

Windows logon:
  create local access token using authenticated identity and groups
```

The exact Active Directory implementation includes platform-specific behavior beyond generic LDAP and Kerberos.

---

# 12.72 Service Principal Names

In Active Directory terminology, service identities are commonly represented through Service Principal Names, or SPNs, associated with accounts.

Example:

```text
HTTP/reports.example.internal
```

The KDC must map the requested service principal to a unique account and key.

A duplicate SPN can create:

* Ticket issuance to the wrong account
* Service decryption failure
* Intermittent behavior
* Security ambiguity
* Load-balanced authentication failures

The uniqueness of the service-principal mapping is a core part of service authentication.

---

# 12.73 Password change and replication

In replicated identity environments:

```text
Alice changes password at controller A
Client authenticates against controller B
```

If replication has not completed:

```text
New password may fail temporarily
Old password behavior may depend on platform safeguards and cache
```

Similarly:

* New service key may not reach every KDC.
* New group membership may be stale.
* Disabled account may remain accepted by an offline cache.
* Keytab deployment may lag KDC update.

Distributed identity is subject to consistency and replication delays.

---

# 12.74 Directory availability and offline login

A directory client may cache enough information to allow login during an outage.

Possible cached state:

* User identity
* UID and GID
* Group membership
* Password-derived verification data
* Kerberos tickets
* Account policy

Availability benefit:

```text
Directory unavailable
→ previously known user can still log in
```

Security cost:

```text
User disabled centrally
→ offline host may still accept cached identity
```

Offline-login policy should define:

* Who is eligible
* How long cache remains valid
* Which services permit offline access
* Whether privileged users are excluded
* How revocation is handled

---

# 12.75 LDAP replication consistency

Directory replicas may disagree temporarily.

Examples:

```text
Replica A:
  Alice is finance-admin

Replica B:
  Alice has been removed
```

Application behavior depends on which replica it contacts.

Potential mitigations:

* Stronger consistency for security-sensitive writes
* Sticky reads after write
* Version or change-sequence checks
* Short cache lifetimes
* Central authorization service
* Revocation events
* Explicit replication monitoring

---

# 12.76 Directory service-account security

A directory-search account should be treated as a machine credential.

Protect:

* Password or keytab
* Allowed source hosts
* Bind mechanism
* TLS trust
* Read permissions
* Rotation process
* Logs
* Connection pools

Prefer:

```text
Read only necessary identity attributes
```

Avoid:

```text
Directory administrator account
embedded in every application
```

Compromise of a broadly privileged search account can expose the entire identity directory.

---

# 12.77 LDAP connection pooling

Applications pool LDAP connections for performance.

Each connection has state:

```text
TLS state
Bind identity
SASL security context
Outstanding operations
Server association
```

Dangerous flow:

```text
Connection bound as Alice
→ returned to service-account pool
→ next request unknowingly operates as Alice
```

Safe design:

* Separate user-authentication connections
* Clear or rebind before reuse
* Validate pool identity
* Do not multiplex incompatible Bind state
* Fail closed after Bind error
* Bound connection ownership must be explicit

---

# 12.78 LDAP referrals

An LDAP server can refer a client to another server.

Possible implications:

```text
Client sends search
→ server returns referral URL
→ client follows referral
```

Security questions:

* Is referral target trusted?
* Will credentials be forwarded?
* Is TLS required?
* Is the target in the same directory trust boundary?
* Are search bases equivalent?
* Could referral create SSRF?
* Could it leak Bind credentials?

Do not enable automatic credential forwarding to arbitrary referral targets without explicit trust rules.

---

# 12.79 Pagination and size limits

Directories may enforce:

* Maximum result count
* Time limit
* Administrative size limit
* Paged-result controls
* Search-depth limits

An application that assumes one search returns every group member can silently miss authorization data.

Example:

```text
Group has 20,000 members
Server limit returns first 1,000
Application interprets result as complete
```

Authorization logic must detect partial results and paging controls.

---

# 12.80 LDAP indexes and authentication performance

Login may require searches on:

```text
uid
mail
employeeNumber
member
memberOf
```

Without appropriate directory indexes:

```text
Login request
→ large subtree scan
→ high latency and directory load
```

Performance fixes should preserve security semantics.

Changing:

```text
unique user lookup
```

to:

```text
broad cached search
```

can create account ambiguity or stale access.

---

# 12.81 LDAP troubleshooting by stage

```mermaid
flowchart TD
    A[LDAP-backed login failed]
    B{Network and TLS succeed?}
    C[Check DNS, port, certificate, StartTLS]
    D{Service Bind succeeds?}
    E[Check Bind DN, password, SASL mechanism, ACL]
    F{User search returns exactly one entry?}
    G[Check base, scope, filter, escaping, indexes]
    H{User Bind succeeds?}
    I[Check empty password, DN, password, account state]
    J{Attributes and groups readable?}
    K[Check ACL, paging, nesting, schema]
    L{Local mapping succeeds?}
    M[Check UID, tenant, stable ID, duplicate accounts]
    N{Authorization succeeds?}
    O[Check roles, groups, cache, resource policy]

    A --> B
    B -->|No| C
    B -->|Yes| D
    D -->|No| E
    D -->|Yes| F
    F -->|No| G
    F -->|Yes| H
    H -->|No| I
    H -->|Yes| J
    J -->|No| K
    J -->|Yes| L
    L -->|No| M
    L -->|Yes| N
    N -->|No| O
```

---

# 12.82 Practical scenario: LDAP search succeeds, login fails

Observed:

```text
Directory reader can find Alice
User login says invalid credentials
```

Possible causes:

* Wrong user DN used for Bind
* Password wrong
* Empty password rejected
* User account locked
* Password expired
* Authentication disabled for that entry
* Client sends Bind before TLS
* User belongs to another directory realm
* Connection pool retains wrong Bind state
* Password changed on another replica

The search service’s success proves only its own Bind and search authorization.

---

# 12.83 Practical scenario: login succeeds, groups are empty

Possible causes:

* Group attribute hidden by ACL
* Wrong group schema expected
* Nested groups not expanded
* Search base excludes groups
* Directory replica stale
* Paging incomplete
* Application queries `memberOf` but directory does not maintain it
* DN normalization mismatch
* Group cache stale
* User authenticated against one tenant and searched in another

Debug the raw group data before changing application roles manually.

---

# 12.84 Practical scenario: Kerberos works by hostname but not alias

Working:

```text
ldap/node-1.example.internal
```

Failing:

```text
ldap/directory.example.internal
```

Likely issue:

```text
Client requests service principal for alias
but
KDC or keytab contains only node hostname principal
```

Possible solutions depend on architecture:

* Provision logical service principal.
* Configure service to accept it.
* Correct client canonicalization.
* Ensure unique SPN mapping.
* Avoid untrusted DNS alias manipulation.

Do not solve by disabling service-name verification.

---

# 12.85 Practical scenario: first request works, later request fails

Possible causes:

* Ticket expires
* Credential cache replaced
* LDAP pooled connection closes
* TLS session resumed with changed policy
* Group cache refreshes to a denial
* KDC key rotates
* Service keytab reload fails
* Long-running process points to deleted credential cache
* Renewal job runs under another UID
* Clock jumps

Capture:

```text
Ticket lifetime
Credential-cache path
Process UID
Service principal
KVNO
Connection identity
Policy version
Exact failure timestamp
```

---

# 12.86 Security considerations

## Protect KDCs

A KDC holds or controls keys for many principals.

Compromise can allow:

* Ticket forgery
* Service impersonation
* User impersonation
* Trust manipulation
* Long-term credential exposure
* Cross-realm compromise

KDC systems should have:

* Narrow administrative access
* Strong host security
* Protected backups
* Audit logging
* Replicated availability
* Controlled time synchronization
* Secure key lifecycle
* Minimal unrelated services

---

## Protect keytabs like passwords

A keytab is not harmless configuration.

Treat it as:

```text
Long-term non-human credential
```

Use:

* Minimal file permissions
* Separate keytab per service where practical
* Secret-managed deployment
* Rotation
* Inventory
* Avoidance of image embedding
* Limited principal authority

---

## Minimize delegated credentials

A forwarded TGT can represent broad user authority.

Prefer:

* No delegation when unnecessary
* Service-specific constrained delegation
* Short lifetimes
* Trusted intermediaries
* Explicit downstream audience
* Audited actor and subject

---

## Require TLS for password Bind

A directory password sent without confidentiality protection can be captured.

Client and server must both enforce:

```text
No password Bind before protected channel
```

Do not rely only on user instructions.

---

## Escape LDAP input correctly

Use separate library functions for:

* Search-filter values
* Distinguished-name values
* LDAP URLs where applicable

These syntaxes have different escaping rules.

---

## Treat group data as authorization-critical

Protect against:

* Unauthorized group writes
* Stale cache
* Nested membership confusion
* Duplicate groups
* Cross-tenant groups
* Partial search results
* Case and DN normalization errors

---

## Keep local and federated identity distinct

Do not reduce:

```text
alice@ENGINEERING.EXAMPLE
alice@FINANCE.EXAMPLE
```

to:

```text
alice
```

unless a trusted mapping policy guarantees they represent the same person.

---

# 12.87 Performance considerations

## KDC load

A user does not need to contact the KDC for every application message.

Typical pattern:

```text
Obtain TGT
→ cache it

Obtain one service ticket
→ cache it

Reuse ticket or established context
```

Performance depends on:

* Ticket-cache behavior
* Service-ticket lifetime
* Number of distinct SPNs
* KDC location
* DNS
* Cryptographic operations
* Preauthentication method

---

## LDAP search load

A single login can perform:

```text
Service Bind
User search
User Bind
Group search
Nested-group expansion
Profile lookup
Policy lookup
```

At large scale, optimize using:

* Narrow search bases
* Indexed attributes
* Connection pools
* Bounded caches
* Batch group operations
* Avoidance of unnecessary attributes
* Server-side membership mechanisms where appropriate

Do not cache authentication results as if they were ordinary profile data.

---

## Negative caching

Directory clients may cache:

```text
Alice not found
```

If Alice is then provisioned:

```text
Lookup still returns not found until cache expires
```

Negative caching improves outage and typo performance but delays newly created identities.

---

## Ticket size

Kerberos tickets can become large when they contain substantial authorization data.

Consequences:

* UDP fragmentation
* Fallback to TCP
* Larger HTTP authentication headers
* Proxy header limits
* Application buffer issues
* Increased processing cost

Large group membership can therefore cause authentication failures that appear unrelated to authorization.

---

# 12.88 Exact data-flow comparison

## LDAP password login

```text
User password
→ application
→ protected LDAP Bind
→ directory verifies credential
→ application obtains identity and groups
→ local session
```

The application temporarily handles the user password.

---

## Kerberos login

```text
User credential
→ Kerberos client/KDC exchange
→ TGT
→ service ticket
→ service verifies ticket using keytab
→ local session or security context
```

The target application does not receive the user password.

---

## OIDC login

```text
User credential
→ identity provider
→ authorization code
→ ID Token
→ application validates token
→ local session
```

The relying application does not receive the user password.

---

## mTLS workload login

```text
Private-key proof
→ TLS handshake
→ certificate identity
→ application principal
→ authorization
```

No human password is involved.

---

# 12.89 Kerberos, LDAP, and OIDC comparison

| Property                         | Kerberos                             | LDAP Bind                                  | OpenID Connect                       |
| -------------------------------- | ------------------------------------ | ------------------------------------------ | ------------------------------------ |
| Primary purpose                  | Network authentication and ticketing | Directory authentication state and access  | Federated application authentication |
| User password reaches target app | No                                   | Often yes before LDAP Bind                 | No                                   |
| Credential for later services    | Service ticket                       | Usually none from Bind itself              | Access token or local session        |
| Directory query capability       | No                                   | Yes                                        | UserInfo or separate directory       |
| Strong dependence on time        | High                                 | TLS and policy dependent                   | Token and certificate time checks    |
| Service identity                 | Service principal/keytab             | LDAP server TLS identity and Bind identity | Issuer/client/audience model         |
| Common SSO behavior              | Ticket cache reused                  | Application-specific                       | Identity-provider session reused     |
| Common machine use               | Keytabs and service tickets          | Service Bind account                       | Client credentials/federation        |

---

# 12.90 Simplified model versus exact implementation

## Simplified model

```text
Kerberos authenticates users.
LDAP stores users and groups.
Application checks groups.
```

## More exact implementation

```text
Client resolves realm and KDC
→ negotiates preauthentication
→ derives or accesses long-term key
→ receives encrypted AS reply and TGT
→ stores credential in selected cache
→ constructs exact service principal
→ requests service ticket
→ KDC applies principal and delegation policy
→ service selects keytab entry by principal, kvno, and enctype
→ verifies ticket and authenticator
→ GSS/SASL maps authenticated name
→ application maps realm principal to local subject
→ directory connection establishes TLS
→ Bind identity is established
→ user and group searches run under LDAP ACL
→ nested membership is expanded
→ cached attributes are evaluated
→ application authorization decides access
→ local operating system enforces process and file permissions
```

The simplified model is suitable for architecture diagrams.

The exact model is required when:

* Authentication works on one server but not another.
* Host aliases are involved.
* Tickets fail after key rotation.
* Group removal does not revoke access.
* LDAP search succeeds but user Bind fails.
* Cross-realm users map incorrectly.
* Delegation or proxy authentication is used.
* Load balancers hide service instances.
* Directory replicas disagree.

---

# 12.91 Questions: Kerberos Fundamentals

1. Why does Alice receive both a TGT and a client–TGS session key?
2. Why is the TGT encrypted for the ticket-granting service rather than for Alice?
3. What does Kerberos preauthentication prove before the KDC issues a TGT?
4. How does preauthentication reduce offline-guessing exposure compared with an unrestricted AS reply?
5. Why can Alice carry a service ticket without being able to modify its identity or expiry?
6. What does the service authenticator add beyond the service ticket?
7. Why does Kerberos require synchronized clocks?
8. How does a replay cache distinguish a repeated authenticator from a new one?
9. Why is a credential cache equivalent to an active authentication credential?
10. What is the difference between a TGT and a service ticket?
11. Why does the service need a keytab but the user normally does not?
12. How does a `kvno` mismatch prevent service authentication?
13. Why can `kvno` succeed while the target application still fails?
14. What is the security effect of stealing a service keytab?
15. Why should a service principal identify the logical service rather than merely the current backend host?
16. How can a DNS alias produce a Kerberos service-principal mismatch?
17. Why does mutual Kerberos authentication require a response from the service?
18. What application-data risks remain if Kerberos is used only during connection establishment?
19. How does a renewable ticket differ from a long initial ticket?
20. Why is destroying the local credential cache not equivalent to globally revoking all tickets?

---

# 12.92 Questions: GSS-API and Delegation

1. What problem does GSS-API solve for application developers?
2. Why is GSS-API not itself a network identity database?
3. What is the difference between a GSS initiator and acceptor?
4. Which process needs access to the service keytab in a GSS Kerberos exchange?
5. Why must a GSS principal still be mapped to an application-local subject?
6. What risk appears if the realm portion is removed before identity mapping?
7. How do GSS integrity and confidentiality differ from authentication-only context establishment?
8. What security property can channel binding add?
9. Why can optional channel-binding enforcement create downgrade ambiguity?
10. What does SASL add above GSS-API?
11. How can an LDAP SASL GSSAPI failure originate in Kerberos rather than LDAP search policy?
12. Why is forwarding a TGT to an untrusted SSH server dangerous?
13. How does constrained delegation reduce the authority of an intermediary?
14. Which identities should an audit event retain during delegated service access?
15. How can long-lived delegated credentials undermine user revocation?

---

# 12.93 Questions: LDAP Internals

1. What is the difference between an LDAP Bind and an LDAP Search?
2. Why does a successful directory search not prove the searched user’s password?
3. Why must empty passwords be rejected before simple Bind?
4. What security failure occurs if a client attempts StartTLS but continues after upgrade failure?
5. Why is `-ZZ` generally safer than `-Z` for password-based diagnostics?
6. What does the LDAP search base control?
7. How does search scope affect performance and result correctness?
8. Why must DN escaping and search-filter escaping use different functions?
9. How can LDAP injection alter the structure of a search?
10. Why should a user search require exactly one matching result?
11. How can an LDAP connection pool accidentally reuse Alice’s Bind identity for another request?
12. Why should user-authentication connections and service-search connections often be separated?
13. What is the difference between authentication identity and authorization identity in SASL?
14. Under what conditions can SASL EXTERNAL map a TLS certificate to an LDAP identity?
15. Why can an LDAP ACL permit reading `mail` while denying `employeeNumber`?
16. How can a user authenticate successfully but receive no application role because of directory ACLs?
17. Why can nested groups create both performance and correctness problems?
18. How should a group resolver handle membership cycles?
19. Why can partial LDAP search results become an authorization vulnerability?
20. How can automatic referral following leak credentials or create SSRF?

---

# 12.94 Questions: Integrated Enterprise Identity

1. How can Kerberos authentication succeed while Linux account creation still fails?
2. What does LDAP provide to NSS that Kerberos does not?
3. What does Kerberos provide to PAM that a basic LDAP search does not?
4. Why might an existing Linux session retain old groups after LDAP membership changes?
5. How can a directory cache delay account disablement?
6. Why can two directory replicas produce different authorization decisions?
7. How does password-change replication create intermittent login failures?
8. Why does a duplicate Service Principal Name cause ambiguous service authentication?
9. What risks arise when a directory administrator credential is embedded in every application?
10. How can large group membership increase Kerberos ticket size?
11. Why can oversized Kerberos authentication headers fail at an HTTP reverse proxy?
12. What should be checked when authentication works using a server hostname but fails through a load-balancer alias?
13. How would you distinguish “user not found,” “password rejected,” “group unreadable,” and “application unauthorized”?
14. What caches must be invalidated when Alice is removed from a privileged group?
15. Why is a stable realm-qualified principal safer than a bare username for identity mapping?
16. Which layer should enforce whether an engineering-realm user may access a finance application?
17. How does offline login improve availability while weakening immediate central revocation?
18. What evidence would show that a Kerberos key rotation reached the KDC but not every service replica?
19. How could a connection bound as a directory service account unexpectedly become anonymous?
20. Why must authentication observability record the exact stage rather than only “login failed”?

# 13. Authentication and Authorization Debugging Toolkit

Authentication failures often appear as a single message:

```text
Login failed
```

or:

```text
Access denied
```

The real failure may occur at any of several layers:

```text
DNS
→ network connection
→ TLS
→ credential transmission
→ credential verification
→ identity mapping
→ account policy
→ session creation
→ token validation
→ authorization
→ resource enforcement
```

The goal of debugging is to identify:

> The earliest security invariant that failed.

---

# 13.1 Do not begin by changing permissions

A common debugging pattern is:

```text
Access denied
→ add administrator role
→ retry
→ disable certificate checking
→ retry
→ make file world-readable
```

This can:

* Hide the original defect
* Create unrelated vulnerabilities
* Destroy useful evidence
* Make rollback difficult
* Produce multiple simultaneous causes
* Grant excessive access permanently

Use an evidence-driven sequence:

```text
Observe
→ identify failed layer
→ form one hypothesis
→ perform one controlled test
→ compare result
→ restore temporary changes
```

---

# 13.2 First debugging questions

Before running commands, record:

```text
Who was the intended subject?
Which credential was used?
Which service received the request?
Which resource and action were requested?
What result was expected?
What result actually occurred?
When did it occur?
Which request or trace identifier applies?
```

Example incident record:

```yaml
timestamp: 2026-07-31T11:38:42Z
caller: finance-web
end_user: u_1048
target: reports-api
action: report.download
resource: report-71
expected: allow
observed: HTTP 403
request_id: req_8192
environment: production
region: ap-south
```

Avoid recording raw:

* Passwords
* Access tokens
* Refresh tokens
* Session cookies
* Authorization codes
* Private keys
* SAML assertions
* Kerberos ticket caches

---

# 13.3 Build a timeline

Distributed authentication produces events on different systems.

Example:

```text
11:38:40.115 Browser sends request
11:38:40.140 Gateway receives request
11:38:40.147 Gateway validates token
11:38:40.154 Application receives request
11:38:40.168 Policy engine returns deny
11:38:40.171 Application returns 403
11:38:40.179 Gateway forwards response
11:38:40.212 Browser receives response
```

A useful timeline contains:

* Absolute timestamp
* Time zone
* Component
* Request ID
* Actor
* Event
* Result
* Duration
* Error classification

---

## Why time synchronization matters

Without synchronized clocks:

```text
Gateway log:
  11:38:40

Application log:
  11:36:12

Identity provider log:
  11:41:02
```

The same transaction may appear to occur in the wrong order.

Time problems also affect:

* Token expiry
* Certificate validity
* Kerberos authenticators
* SAML assertions
* Session expiry
* Replay windows
* Audit reconstruction

---

# 13.4 Layered failure model

Use the following layers in order.

```mermaid
flowchart TD
    A[1. Name resolution]
    B[2. Network route]
    C[3. Listening socket]
    D[4. TLS or secure transport]
    E[5. HTTP or application protocol]
    F[6. Credential presence]
    G[7. Credential validation]
    H[8. Identity mapping]
    I[9. Account and session policy]
    J[10. Authorization decision]
    K[11. Resource enforcement]
    L[12. Response propagation]

    A --> B --> C --> D --> E --> F --> G --> H --> I --> J --> K --> L
```

Do not investigate application roles before confirming that the request reached the intended application under the intended identity.

---

# 13.5 Layer 1: Name resolution

The client begins with a name such as:

```text
reports.example.internal
```

That name may resolve through:

* `/etc/hosts`
* DNS
* mDNS
* NSS modules
* Container DNS
* Kubernetes DNS
* Service discovery
* Proxy configuration
* Application-specific resolution

A correct certificate and token cannot help if the client reaches the wrong host.

---

## Inspect host resolution through NSS

```bash
getent ahosts reports.example.internal
```

### What it does

Queries the system’s configured name-service sources.

### Why it is useful

It often reflects the resolution path used by ordinary applications more closely than querying DNS alone.

Illustrative output:

```text
192.0.2.80    STREAM reports.example.internal
192.0.2.80    DGRAM
192.0.2.80    RAW
```

### Questions to ask

* Is the address expected?
* Are both IPv4 and IPv6 returned?
* Does the application prefer another address family?
* Is `/etc/hosts` overriding DNS?
* Does a container see the same answer as the host?

---

## Query DNS directly

```bash
dig reports.example.internal A
```

For IPv6:

```bash
dig reports.example.internal AAAA
```

A compact output:

```bash
dig +short reports.example.internal
```

### Why compare `dig` with `getent`?

```text
dig:
  queries DNS

getent:
  follows configured system name-service behavior
```

Different results may indicate:

* `/etc/hosts` override
* NSS ordering
* Local resolver cache
* Split-horizon DNS
* VPN-specific DNS
* Container resolver differences
* Search-domain expansion

---

## Inspect canonical name chains

```bash
dig reports.example.internal CNAME
```

Then resolve each returned name.

A chain may lead to:

```text
reports.example.internal
→ reports-lb.example.internal
→ region-a-lb.provider.example
→ 192.0.2.80
```

This affects:

* Certificate naming
* Kerberos service principals
* Routing
* Load-balancer selection
* Regional behavior

---

## Test from the actual runtime

A host-level command may succeed while a container fails.

Run the test from:

* The affected Pod
* The service container
* The same network namespace
* The same VM
* The same user account where resolution policy differs

Example:

```bash
kubectl exec \
  --namespace finance \
  deploy/finance-web \
  -- getent ahosts reports-api.finance.svc.cluster.local
```

### Safety

`kubectl exec` gives command execution inside a workload.

Use a controlled diagnostic container when the application image lacks tools.

Do not install untracked packages into a production container merely for convenience.

---

# 13.6 Layer 2: Network path

After resolution, determine whether the client can reach the destination address and port.

A failure may occur because of:

* Routing
* Firewall
* Security group
* NetworkPolicy
* Proxy
* NAT
* Load balancer
* Service mesh
* Incorrect port
* Address-family mismatch
* Egress restriction
* Server not listening

---

## Inspect route selection

```bash
ip route get 192.0.2.80
```

Illustrative output:

```text
192.0.2.80 via 192.0.2.1 dev eth0 src 192.0.2.51
```

This shows the kernel’s selected:

* Route
* Interface
* Gateway
* Source address

The application may still use another network namespace or proxy.

---

## Test TCP establishment

Where `nc` is installed:

```bash
nc -vz reports.example.internal 443
```

### What it tests

Primarily:

```text
Can a TCP connection be established?
```

### What it does not test

* TLS certificate
* HTTP behavior
* Authentication
* Authorization
* Correct virtual host
* Application health

Possible output:

```text
Connection to reports.example.internal 443 port [tcp/https] succeeded!
```

A successful TCP connection proves only that something accepted the connection.

---

## Interpret common connection errors

### Connection refused

```text
TCP destination actively rejected connection
```

Possible causes:

* No process listening
* Firewall actively rejects
* Wrong port
* Application crashed
* Service bound to another interface

### Connection timed out

Possible causes:

* Packet dropped by firewall
* No route
* Security group
* NetworkPolicy
* Broken return path
* Destination unavailable

### No route to host

Possible causes:

* Missing route
* Interface down
* Network namespace mismatch
* Firewall-generated unreachable response

Error messages vary by operating system and network stack.

---

# 13.7 Layer 3: Listening socket

On the target host, verify which process is listening.

```bash
sudo ss -ltnp
```

### Important flags

* `-l`: listening sockets
* `-t`: TCP
* `-n`: numeric addresses and ports
* `-p`: process information where permitted

The `ss` utility can show listening or established sockets, numeric endpoints, process information, queues, and detailed TCP state.

---

## Filter one port

```bash
sudo ss -ltnp 'sport = :443'
```

Illustrative output:

```text
State  Recv-Q Send-Q Local Address:Port Peer Address:Port Process
LISTEN 0      4096   0.0.0.0:443      0.0.0.0:*     users:(("gateway",pid=21871,fd=9))
```

Interpret:

```text
0.0.0.0:443
→ listening on all IPv4 interfaces
```

```text
127.0.0.1:443
→ listening only on loopback
```

```text
[::]:443
→ listening on IPv6 wildcard, with IPv4 behavior depending on system configuration
```

---

## Common bind mismatch

Application configuration:

```text
listen_address = 127.0.0.1
```

Load balancer connects to:

```text
server_private_ip:443
```

Result:

```text
Application healthy locally
but unreachable remotely
```

---

## Inspect established connections

```bash
sudo ss -tnp state established
```

This can show whether:

* The client reached the host
* A proxy has open backend connections
* Connections are stuck
* Unexpected peers are connected

Do not infer authenticated identity solely from socket endpoints.

---

# 13.8 Layer 4: TLS

A TCP connection can succeed while TLS fails.

TLS failure categories include:

* Unsupported protocol version
* No common cipher or signature algorithm
* Wrong server name
* Unknown CA
* Missing intermediate
* Expired certificate
* Hostname mismatch
* Client certificate missing
* Client certificate rejected
* Private-key mismatch
* Proxy interception
* Revocation or policy failure

---

## Inspect a live TLS server

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -showcerts \
  -verify_return_error \
  -verify_hostname reports.example.internal
```

OpenSSL’s current `s_client` interface supports explicit connection targets, SNI, certificate display, verification failures, hostname checking, client certificates, custom CA stores, StartTLS, and protocol diagnostics.

### Important flags

* `-connect`: network destination
* `-servername`: Server Name Indication value
* `-showcerts`: show certificates sent by the server
* `-verify_return_error`: terminate on verification failure
* `-verify_hostname`: verify expected DNS identity

### Why SNI matters

A server may host several TLS identities on one address.

Without:

```text
-servername reports.example.internal
```

the server may return its default certificate.

---

## Use an internal CA

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -CAfile internal-root-ca.crt \
  -verify_return_error \
  -verify_hostname reports.example.internal
```

Do not respond to:

```text
unknown CA
```

by disabling certificate verification.

Fix:

* Trust bundle
* Certificate chain
* Target identity
* CA rotation
* Proxy configuration

---

## Brief output

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -brief \
  -verify_return_error \
  -verify_hostname reports.example.internal
```

This is useful for a compact protocol and verification summary where supported.

---

## Test a specific protocol boundary

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -tls1_3
```

or:

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -tls1_2
```

Use protocol forcing only as a diagnostic.

Do not permanently re-enable obsolete protocols merely to make an old client work.

---

# 13.9 Read TLS results correctly

A useful checklist:

```text
TCP connected?
SNI correct?
Peer chain received?
Chain trusted?
Hostname matched?
Validity interval accepted?
Client certificate requested?
Client certificate sent?
Handshake completed?
Application data exchanged?
```

---

## Successful connection is not sufficient

A completed `s_client` process does not necessarily mean verification succeeded.

Look for:

```text
Verification: OK
```

or equivalent successful validation output.

Using `-verify_return_error` avoids silently proceeding after some verification failures.

---

## Unknown CA

Possible causes:

* Private CA absent from trust store
* Wrong trust store used
* Missing intermediate
* Server sent wrong chain
* Client running inside container with another CA bundle
* CA rotated
* TLS-intercepting proxy issued another certificate

---

## Hostname mismatch

Expected:

```text
reports.example.internal
```

Presented SAN:

```text
DNS:reports-node-3.example.internal
```

The certificate may be valid for the node but not the logical service name.

Do not replace hostname verification with a generic “certificate is signed” check.

---

## Expired or not-yet-valid

Inspect current time and certificate dates.

```bash
date -u
```

```bash
openssl s_client \
  -connect reports.example.internal:443 \
  -servername reports.example.internal \
  -showcerts </dev/null 2>/dev/null |
openssl x509 -noout -dates
```

The pipeline assumes the first parsed certificate is the intended leaf; complex output should be saved and inspected deliberately.

---

# 13.10 Inspect certificate files

```bash
openssl x509 \
  -in server.crt \
  -noout \
  -subject \
  -issuer \
  -serial \
  -dates \
  -fingerprint \
  -sha256
```

Inspect SANs:

```bash
openssl x509 \
  -in server.crt \
  -noout \
  -ext subjectAltName
```

Inspect usage:

```bash
openssl x509 \
  -in server.crt \
  -noout \
  -ext keyUsage \
  -ext extendedKeyUsage
```

---

## Compare certificate and private key

For RSA keys, one traditional comparison computes public-key-derived values, but algorithm-specific commands are easy to misuse.

A more general method extracts public keys:

```bash
openssl x509 \
  -in server.crt \
  -pubkey \
  -noout > cert-public.pem
```

```bash
openssl pkey \
  -in server.key \
  -pubout > key-public.pem
```

Compare:

```bash
cmp cert-public.pem key-public.pem
echo $?
```

Result:

```text
0
→ public keys match
```

### Safety

`server.key` is a private key.

Commands that read it should run only in a controlled environment.

Delete temporary public-key files when finished if operational policy requires it.

---

# 13.11 Layer 5: HTTP protocol

After TLS succeeds, inspect HTTP behavior.

Questions:

* Which method was sent?
* Which host was requested?
* Which path and query were sent?
* Which status was returned?
* Was there a redirect?
* Which headers were added or removed?
* Was a cookie set?
* Was a body returned?
* Which proxy answered?
* Did the application receive the request?

---

## Capture headers and body separately

```bash
curl -sS \
  -D response-headers.txt \
  -o response-body.txt \
  https://reports.example.internal/api/reports
```

### Important flags

* `-sS`: suppress progress output but display errors
* `-D FILE`: write response headers
* `-o FILE`: write response body

curl supports verbose diagnostics, trace files, included headers, and separate header dumps through `-v`, `--trace`, `--trace-ascii`, `-i`, and `-D`.

---

## Include headers in terminal output

```bash
curl -i https://reports.example.internal/api/reports
```

Useful for quick inspection.

Avoid it when the response body is:

* Large
* Binary
* Sensitive
* A downloadable file

---

## Verbose request

```bash
curl -v \
  https://reports.example.internal/api/reports \
  -o /dev/null
```

Verbose output can reveal:

* Resolution
* Connection address
* Proxy usage
* TLS negotiation summary
* Request headers
* Response headers
* Redirect behavior

### Safety

Verbose output may contain:

* `Authorization`
* `Cookie`
* `Set-Cookie`
* Client certificate information
* Internal hostnames

Do not paste raw verbose output into public systems.

---

# 13.12 High-detail curl trace

```bash
curl \
  --trace-ascii curl-trace.txt \
  https://reports.example.internal/api/reports \
  -o response-body.txt
```

curl’s official manual recommends `--trace` or `--trace-ascii` for greater protocol detail than ordinary verbose mode.

### Why it is useful

It can show:

* Connection establishment
* Protocol bytes
* Headers
* Redirects
* Timing order
* Proxy exchanges

### Critical safety warning

A trace can contain complete credentials and application data.

Protect it:

```bash
umask 077
```

before creation, or:

```bash
chmod 600 curl-trace.txt
```

after creation.

Redact carefully before sharing.

---

# 13.13 HTTP timing breakdown

```bash
curl -sS \
  -o /dev/null \
  -w 'dns=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nfirst_byte=%{time_starttransfer}\ntotal=%{time_total}\nstatus=%{http_code}\n' \
  https://reports.example.internal/api/reports
```

Illustrative output:

```text
dns=0.004812
connect=0.021303
tls=0.064212
first_byte=0.181440
total=0.182007
status=401
```

Interpretation:

```text
DNS delay:
  time_namelookup

TCP establishment:
  time_connect

TLS establishment:
  time_appconnect

Application response start:
  time_starttransfer

Total:
  time_total
```

This helps distinguish:

* Resolver delay
* Network delay
* TLS delay
* Backend delay

It does not identify every internal application step.

---

# 13.14 Follow redirects deliberately

```bash
curl -i \
  -L \
  https://reports.example.internal/login
```

### Important flag

* `-L`: follow redirects

Before following automatically, inspect:

* Destination host
* Scheme
* Status code
* Cookie scope
* Whether credentials may be resent
* Whether method changes
* Redirect loops

For authentication flows, preserving each intermediate response is often more useful than immediately using `-L`.

---

## Inspect one redirect at a time

```bash
curl -sS \
  -D - \
  -o /dev/null \
  https://reports.example.internal/login
```

Illustrative response:

```http
HTTP/1.1 302 Found
Location: https://identity.example.internal/authorize?...
Set-Cookie: login_transaction=...; Secure; HttpOnly
```

Record the transition, but redact:

* State values
* Nonces
* Authorization codes
* Transaction cookies

---

# 13.15 Force a specific DNS result

```bash
curl \
  --resolve reports.example.internal:443:192.0.2.80 \
  https://reports.example.internal/health
```

### What it does

Uses:

```text
URL hostname:
  reports.example.internal

Network destination:
  192.0.2.80
```

while preserving the logical hostname for HTTP and TLS identity handling.

### Why it is useful

It tests one server or load-balancer address without changing system DNS.

### Safety

This bypasses ordinary DNS routing and may hit:

* A retired instance
* A backend not intended for direct access
* Another environment
* A host without normal gateway protections

Use only against authorized infrastructure.

---

# 13.16 Detect proxy influence

Inspect environment:

```bash
env | grep -iE '^(http|https|all|no)_proxy='
```

A request may travel through:

* Corporate HTTP proxy
* Sidecar
* Transparent proxy
* Service mesh
* Local debugging proxy
* VPN gateway

Compare normal behavior with an explicitly controlled proxy setting only when policy allows.

Example:

```bash
curl --noproxy '*' \
  https://reports.example.internal/health
```

This can determine whether a configured proxy is involved.

### Safety

Bypassing a required organizational proxy may violate policy or remove security controls.

Use it only in an approved test environment.

---

# 13.17 HTTP status classification

Status codes are clues, not complete diagnoses.

| Status         | Common interpretation                                     |
| -------------- | --------------------------------------------------------- |
| `200`          | Request succeeded at HTTP level                           |
| `201`          | Resource created                                          |
| `302` or `303` | Redirect, often login or callback flow                    |
| `400`          | Malformed request or protocol validation failure          |
| `401`          | Authentication absent, invalid, or insufficiently current |
| `403`          | Identity known but action forbidden                       |
| `404`          | Resource absent or existence intentionally concealed      |
| `409`          | Resource state conflicts with operation                   |
| `429`          | Rate, quota, or abuse limit                               |
| `500`          | Server-side unhandled failure                             |
| `502`          | Gateway could not obtain valid upstream response          |
| `503`          | Service unavailable or overloaded                         |
| `504`          | Gateway timed out waiting for upstream                    |

Implementations do not always use these consistently.

Correlate the response with internal logs.

---

# 13.18 Distinguish gateway and application responses

Look for headers such as:

```text
Server
Via
X-Request-ID
Traceparent
X-Proxy-Upstream
```

Do not trust these as authenticated facts unless the architecture defines them.

Compare response body style:

```text
Gateway-generated JSON:
  {"error":"invalid_token"}

Application-generated JSON:
  {"error":"report_access_denied"}
```

The first failure may occur before the application receives the request.

---

# 13.19 Layer 6: Credential presence

Before validating a credential, confirm it was transmitted.

Examples:

```http
Authorization: Bearer ...
```

```http
Cookie: __Host-session=...
```

```text
TLS client certificate
```

```text
Kerberos AP-REQ
```

```text
SSH public-key proof
```

A missing credential is different from a rejected credential.

---

## Why a credential may be missing

* Browser cookie domain mismatch
* Cookie path mismatch
* `Secure` cookie over HTTP
* SameSite behavior
* Redirect to another host
* Reverse proxy strips `Authorization`
* Client did not attach token
* Token cache empty
* Client certificate not selected
* ServiceAccount token not mounted
* Kerberos cache unavailable
* Environment variable absent
* Header size exceeded
* CORS or browser code path did not run

---

# 13.20 Test bearer-token request safely

Read token without echo:

```bash
read -r -s ACCESS_TOKEN
printf '\n'
```

Send:

```bash
curl -sS \
  -D response-headers.txt \
  -o response-body.json \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  https://reports.example.internal/api/reports
```

Remove shell variable:

```bash
unset ACCESS_TOKEN
```

### Limitations

The token may still briefly appear in process arguments depending on how the shell and operating system expose command lines.

A more controlled diagnostic program or protected curl configuration may be preferable on multi-user systems.

curl can read options from an alternate config file or standard input using `-K`/`--config`, which can reduce direct command-line exposure, though the config content itself must still be protected.

---

## Protected curl config example

Create:

```bash
umask 077
cat > auth-request.curl <<'EOF'
url = "https://reports.example.internal/api/reports"
header = "Authorization: Bearer REDACTED_TEST_TOKEN"
dump-header = "response-headers.txt"
output = "response-body.json"
silent
show-error
EOF
```

Run:

```bash
curl --config auth-request.curl
```

Delete:

```bash
rm -f auth-request.curl
```

Use only a dedicated test token with minimal scope and short expiry.

---

# 13.21 Cookie diagnostics

Store cookies:

```bash
curl -sS \
  -c cookies.txt \
  -D response-headers.txt \
  -o response-body.html \
  https://app.example.internal/login
```

Send them later:

```bash
curl -sS \
  -b cookies.txt \
  -D - \
  -o /dev/null \
  https://app.example.internal/reports
```

curl supports receiving cookies, storing them, and supplying stored cookies on later transfers.

---

## Inspect metadata without exposing values

```bash
awk '
  BEGIN { OFS="\t" }
  /^#/ { next }
  NF >= 7 {
    print "domain=" $1,
          "path=" $3,
          "secure=" $4,
          "expiry=" $5,
          "name=" $6,
          "value=[REDACTED]"
  }
' cookies.txt
```

Check:

* Domain
* Path
* Secure flag
* Expiry
* Name collisions
* Host-only versus domain cookie

---

## Browser-specific issues

curl is not a browser.

It does not reproduce every browser behavior involving:

* SameSite context
* CORS
* JavaScript
* Content Security Policy
* Browser privacy features
* Partitioned storage
* Extension interference
* Service workers

Use browser developer tools to inspect:

* Network requests
* Request cookies
* Response `Set-Cookie`
* Cookie rejection reasons
* Redirect chain
* Initiator
* CORS errors
* Storage state

Do not capture screenshots containing active tokens or cookies.

---

# 13.22 Layer 7: Credential validation

Credential presence does not imply validity.

Validation stages may include:

```text
Parse
→ identify credential type
→ select trusted issuer or key
→ verify cryptographic proof
→ validate time
→ validate audience
→ validate nonce or replay state
→ validate credential status
```

Record the exact failing stage.

Avoid one generic log:

```text
token invalid
```

Prefer:

```text
token rejected:
  reason = audience_mismatch
```

without logging the raw token.

---

# 13.23 JWT inspection

A JWT can be decoded without being verified.

Local payload inspection:

```bash
python3 - "$TOKEN" <<'PY'
import base64
import json
import sys

token = sys.argv[1]
parts = token.split(".")

if len(parts) != 3:
    raise SystemExit("Expected compact three-part JWT")

def decode_segment(segment: str) -> bytes:
    segment += "=" * (-len(segment) % 4)
    return base64.urlsafe_b64decode(segment)

header = json.loads(decode_segment(parts[0]))
payload = json.loads(decode_segment(parts[1]))

print("Header:")
print(json.dumps(header, indent=2, sort_keys=True))
print("\nPayload:")
print(json.dumps(payload, indent=2, sort_keys=True))
PY
```

### Critical interpretation

This reveals claimed data.

It does not prove:

* Signature validity
* Issuer trust
* Audience
* Expiry acceptance
* Correct token type
* Revocation status

---

## Inspect time claims

```bash
date -u -d '@1785490272'
```

GNU `date` can convert a Unix timestamp where supported.

Python alternative:

```bash
python3 - <<'PY'
from datetime import datetime, timezone

value = 1785490272
print(datetime.fromtimestamp(value, timezone.utc).isoformat())
PY
```

Compare:

* `iat`
* `nbf`
* `exp`
* Current UTC
* Configured clock skew

---

## Common JWT validation errors

### Signature failure

Possible causes:

* Wrong issuer
* Unknown key ID
* Stale JWKS cache
* Algorithm mismatch
* Modified token
* Wrong token copied
* Key rotation race

### Audience mismatch

Possible causes:

* Token intended for another API
* Client requested wrong resource
* API configured with wrong audience
* Token forwarding across services

### Expired token

Possible causes:

* Normal expiry
* Refresh failed
* Clock ahead
* Token cached too long
* Request queued too long

### Issuer mismatch

Possible causes:

* Wrong identity-provider tenant
* Environment mismatch
* Multi-issuer confusion
* Discovery misconfiguration
* Attacker-supplied issuer

---

# 13.24 Inspect OIDC metadata and keys

```bash
curl -sS \
  https://identity.example.internal/.well-known/openid-configuration \
  | jq
```

Extract selected fields:

```bash
curl -sS \
  https://identity.example.internal/.well-known/openid-configuration \
  | jq '{
      issuer,
      authorization_endpoint,
      token_endpoint,
      jwks_uri
    }'
```

Then query configured JWKS URI.

Check:

* Expected issuer exactly matches
* `kid` exists
* Key usage is appropriate
* Key set is current
* Environment is correct
* No untrusted dynamic issuer was selected

---

# 13.25 Layer 8: Identity mapping

A credential may validate cryptographically but fail to map to a local identity.

Examples:

```text
OIDC:
  (issuer, sub) → local user

SAML:
  (IdP, NameID) → local user

Kerberos:
  realm principal → local account

mTLS:
  SAN identity → workload principal

LDAP:
  user DN → application account

Kubernetes:
  ServiceAccount subject → service policy
```

---

## Mapping failure categories

* Unknown external subject
* Duplicate mapping
* Email changed
* Wrong tenant
* Realm stripped incorrectly
* Case normalization mismatch
* Deleted local account
* JIT provisioning disabled
* Required attribute missing
* Certificate SAN parsing error
* Proxy replaced identity
* Stale identity cache

---

## Preserve stable keys

Prefer:

```text
issuer = https://identity.example.internal
subject = idp-user-731
```

over:

```text
email = alice@example.internal
```

for permanent identity linkage.

Log a hash or internal mapping ID when sensitive external identifiers should not be exposed.

---

# 13.26 Layer 9: Account and session policy

Authentication may succeed while account policy denies access.

Possible reasons:

* Account disabled
* Account expired
* Password change required
* User not assigned to application
* MFA required
* Authentication too old
* Device not compliant
* Login outside allowed hours
* Local suspension
* Tenant disabled
* Risk policy triggered
* Concurrent-session limit
* Session store unavailable

A correct password or valid token does not override these controls.

---

## Session creation checks

Inspect:

```text
Was a session ID generated?
Was the record committed?
Was a cookie returned?
Was cookie accepted?
Can next request resolve it?
Is the session already expired?
Are signing keys consistent?
```

---

## Session record diagnostic

Use only non-secret fields:

```json
{
  "session_id_hash": "sha256:...",
  "subject_id": "u_1048",
  "status": "active",
  "created_at": "2026-07-31T11:38:40Z",
  "idle_expiry": "2026-07-31T12:08:40Z",
  "absolute_expiry": "2026-07-31T19:38:40Z",
  "auth_methods": ["password", "totp"],
  "generation": 18
}
```

Compare with:

* User session generation
* Current time
* Account status
* Cache state
* Region replica

---

# 13.27 Layer 10: Authorization decision

At this point, determine the complete decision tuple:

```text
Subject
Actor
Action
Resource
Tenant
Context
Policy version
```

Example:

```yaml
subject: u_1048
actor: finance-web
action: report.download
resource: report-71
tenant: tenant-42
context:
  authentication_age_seconds: 34
  device_managed: true
  network_zone: corporate
policy_version: reports-v17
```

---

## Do not debug only the role name

A role may be correct while another condition denies access.

Example:

```text
role = financial_analyst
```

but:

```text
resource.department = legal
subject.department = finance
```

Decision:

```text
deny
```

---

## Record policy explanation

Useful result:

```json
{
  "decision": "deny",
  "reason": "department_mismatch",
  "policy_id": "report-download",
  "policy_version": "17",
  "missing_attributes": []
}
```

Less useful:

```text
permission denied
```

---

## Verify policy inputs at their source

Do not only inspect the final assembled object.

Trace:

```text
department:
  identity directory

tenant:
  local account membership

resource owner:
  reports database

device state:
  device management service

scope:
  access token

authentication method:
  IdP authentication event
```

An input may be:

* Stale
* Missing
* Forged
* Mapped incorrectly
* Read from the wrong tenant
* Converted to the wrong type

---

# 13.28 Layer 11: Enforcement

A policy engine can return the correct decision while enforcement fails.

Examples:

```text
Policy says deny
but
application continues to query and return data
```

```text
Gateway checks access
but
backend is directly reachable
```

```text
UI hides delete button
but
DELETE endpoint remains callable
```

```text
API denies object read
but
CSV export bypasses the same function
```

---

## Verify the operation did not occur

For a denied write:

* Was database row changed?
* Was queue message published?
* Was file created?
* Was cloud action performed?
* Was external API called?
* Was a partial transaction committed?
* Did retry logic repeat it?

Authorization must occur before irreversible side effects or inside the same atomic transaction.

---

# 13.29 Layer 12: Response propagation

The application may return the correct result, but an intermediary changes it.

Examples:

* Gateway converts `401` to `302`
* Proxy caches another user’s response
* Error handler converts `403` to `500`
* Browser follows redirect and shows login page
* CDN serves stale denial
* Client SDK retries with another credential
* Service mesh returns its own `503`

Capture responses at:

```text
Application
→ sidecar
→ gateway
→ load balancer
→ browser
```

Use request IDs and timestamps to prove which component generated the final response.

---

# 13.30 System logs with `journalctl`

Inspect one service:

```bash
sudo journalctl \
  -u reports-api.service \
  --since "15 minutes ago"
```

### Important options

* `-u`: filter by unit
* `--since`: lower time boundary

`journalctl` supports time filtering, unit and field matching, following live output, and structured formats such as JSON.

---

## Follow live events

```bash
sudo journalctl \
  -u reports-api.service \
  -f
```

### Important flag

* `-f`: follow newly appended entries

Perform one controlled request while following the log.

Stop with:

```text
Ctrl+C
```

---

## Include recent context

```bash
sudo journalctl \
  -u reports-api.service \
  -n 200 \
  --no-pager
```

### Important flags

* `-n 200`: last 200 entries
* `--no-pager`: write directly rather than opening a pager

---

## Filter by priority

```bash
sudo journalctl \
  -u reports-api.service \
  -p warning \
  --since "1 hour ago"
```

This shows warning and more severe priorities according to journal filtering semantics.

Authentication failures may be recorded at informational priority, so do not rely only on severe messages.

---

# 13.31 Structured journal output

```bash
sudo journalctl \
  -u reports-api.service \
  --since "15 minutes ago" \
  -o json-pretty
```

Useful fields can include:

```text
MESSAGE
PRIORITY
_SYSTEMD_UNIT
_PID
_UID
_GID
_EXE
_COMM
_HOSTNAME
```

Application-defined structured fields may also appear.

---

## Query a request ID

```bash
sudo journalctl \
  -u reports-api.service \
  --since "1 hour ago" \
  -o json |
jq -r '
  select(.REQUEST_ID == "req_8192") |
  {
    timestamp: .__REALTIME_TIMESTAMP,
    message: .MESSAGE,
    request_id: .REQUEST_ID,
    subject: .SUBJECT_ID,
    decision: .AUTHZ_DECISION
  }
'
```

Exact field names depend on application logging.

---

# 13.32 Correlation identifiers

A request ID should be:

* Generated at a trusted ingress
* Validated for length and format
* Propagated downstream
* Added to logs
* Returned to clients where appropriate
* Distinct from credentials
* Bounded to prevent log abuse

Example:

```http
X-Request-ID: req_8192
```

An untrusted client may supply a duplicate or malicious value.

A gateway can:

* Replace it
* Namespace it
* Store original separately
* Generate a new trace ID

---

## Trace context

Distributed tracing may propagate fields such as:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Trace context should not contain:

* User secrets
* Tokens
* Passwords
* Full authorization policy data
* Sensitive object contents

Trace identifiers help correlate events but do not authenticate callers.

---

# 13.33 Application logging design

A good authentication event separates:

```text
External result:
  generic

Internal classification:
  specific
```

Example:

```json
{
  "event_type": "authentication_failure",
  "request_id": "req_8192",
  "stage": "token_validation",
  "reason": "audience_mismatch",
  "issuer": "https://identity.example.internal",
  "expected_audience": "reports-api",
  "subject_hash": "sha256:...",
  "source_ip": "192.0.2.51"
}
```

Do not include the raw token.

---

## Useful stage names

```text
request_received
credential_missing
credential_parsed
signature_validated
issuer_validated
audience_validated
identity_mapped
account_policy_checked
session_created
authorization_evaluated
resource_accessed
response_sent
```

These make it possible to identify the first missing stage.

---

# 13.34 Process identity inspection

Find service process:

```bash
systemctl status reports-api.service
```

Inspect selected properties:

```bash
systemctl show reports-api.service \
  -p MainPID \
  -p User \
  -p Group \
  -p Environment \
  -p FragmentPath
```

Be cautious: environment output can expose secrets.

---

## Inspect actual process

```bash
PID="$(systemctl show \
  --property MainPID \
  --value \
  reports-api.service)"

ps -o pid,ppid,user,group,comm,args -p "${PID}"
```

Inspect Linux process credentials:

```bash
grep -E '^(Uid|Gid|Groups|Cap):' \
  "/proc/${PID}/status"
```

Check executable:

```bash
readlink -f "/proc/${PID}/exe"
```

Check working directory:

```bash
readlink -f "/proc/${PID}/cwd"
```

Check environment only with strong justification:

```bash
sudo tr '\0' '\n' < "/proc/${PID}/environ"
```

### Safety

Process environments frequently contain credentials.

Do not run or share this output casually.

---

# 13.35 File-access debugging

Check every path component:

```bash
namei -l /srv/reports/finance/q2.pdf
```

Check file metadata:

```bash
stat /srv/reports/finance/q2.pdf
```

Check ACL:

```bash
getfacl /srv/reports/finance/q2.pdf
```

Check service identity:

```bash
sudo -u reports-api \
  -- test -r /srv/reports/finance/q2.pdf

echo $?
```

Check mandatory-control context where applicable:

```bash
ls -Z /srv/reports/finance/q2.pdf
```

---

## Observe the exact failing system call

```bash
sudo strace \
  -f \
  -e trace=openat,connect,bind,listen,accept \
  -p "${PID}"
```

### Important flags

* `-f`: follow child processes
* `-e trace=...`: restrict events
* `-p`: attach to PID

### Safety

Tracing may expose:

* Paths
* Connection addresses
* Process behavior
* Sensitive data in other selected system calls
* Performance-sensitive timing

Prefer a staging reproduction.

---

# 13.36 Packet capture

A packet capture can answer:

```text
Did packets leave?
Which address was contacted?
Did TCP complete?
Which side closed?
Was a TLS alert sent?
Were retransmissions occurring?
```

It usually cannot reveal encrypted HTTP contents without additional key material.

---

## Capture one TLS flow

```bash
sudo tcpdump \
  -i any \
  -nn \
  -s 0 \
  -w auth-debug.pcap \
  'host 192.0.2.80 and tcp port 443'
```

### Important flags

* `-i any`: capture on available interfaces through the pseudo-interface where supported
* `-nn`: avoid name and service resolution
* `-s 0`: capture complete packets according to implementation behavior
* `-w`: write packet records to a file
* final expression: packet filter

The `tcpdump` manual documents interface selection, numeric output, snapshot length, writing capture files, and filter expressions.

### Safety

Packet captures may contain:

* Internal addresses
* DNS names
* Unencrypted credentials
* Cookies
* Tokens
* Application data
* Other users’ traffic

Use the narrowest filter and duration.

Protect the file:

```bash
chmod 600 auth-debug.pcap
```

Delete it according to incident-data policy.

---

## Capture without writing payload

A shorter snapshot length can reduce data collection, but may remove information needed to diagnose handshakes.

Design the capture based on the exact question.

Avoid collecting entire interfaces “just in case.”

---

## Read a capture summary

```bash
tcpdump \
  -nn \
  -r auth-debug.pcap
```

Look for:

* SYN
* SYN-ACK
* RST
* FIN
* Retransmission patterns
* TLS alerts where decoded
* Unexpected destination

Use a protocol analyzer for deeper inspection, subject to organizational authorization.

---

# 13.37 Network capture interpretation

## SYN sent, no response

Possible causes:

* Firewall drop
* Wrong route
* Server unavailable
* Return-path failure
* NetworkPolicy
* Security group

## Immediate RST

Possible causes:

* No listener
* Active rejection
* Proxy closed connection
* Protocol sent to wrong port

## TCP succeeds, TLS alert follows

Possible causes:

* Certificate requirement
* Protocol mismatch
* Signature algorithm mismatch
* Unknown CA
* Client certificate rejection
* SNI routing error

## TLS succeeds, server returns HTTP 401

Transport succeeded.

Investigate application credential presence and validation.

---

# 13.38 Database authorization debugging

A web authorization decision may succeed while the database denies the query.

Possible causes:

* Wrong database role
* Expired database credential
* Row-level security
* Schema privilege
* Tenant context missing
* Connection pool retained old context
* Read replica lacks data
* Stored procedure applies additional checks

---

## Identify current database principal

SQL syntax varies by database.

Examples should be run through an approved read-only session.

Conceptual queries:

```sql
SELECT current_user;
```

```sql
SELECT session_user;
```

Inspect:

* Authenticated database identity
* Effective role
* Tenant/session variables
* Search path
* Transaction state

---

## Authorization-aware query

A request for report 71 may execute:

```sql
SELECT *
FROM reports
WHERE report_id = 71
  AND tenant_id = 'tenant-42';
```

A zero-row result can mean:

* Resource absent
* Wrong tenant
* Replication lag
* Authorization filter denied it

Do not automatically convert zero rows into:

```text
database failure
```

---

# 13.39 Kubernetes debugging

Authentication and authorization may fail at:

```text
kubectl client
→ API server TLS
→ bearer token or client certificate
→ authenticator
→ user mapping
→ RBAC or webhook
→ admission controller
→ resource operation
```

---

## Test authorization

```bash
kubectl auth can-i get reports.example.io \
  --namespace finance \
  --as=system:serviceaccount:finance:reports-api
```

Check subresources separately:

```bash
kubectl auth can-i create pods/exec \
  --namespace finance \
  --as=system:serviceaccount:finance:support-tool
```

---

## Inspect ServiceAccount

```bash
kubectl get pod reports-api-abc123 \
  --namespace finance \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

Inspect RoleBindings:

```bash
kubectl get rolebinding \
  --namespace finance \
  -o yaml
```

Inspect a specific binding rather than exporting the entire cluster authorization model where possible.

---

## Distinguish API response classes

```text
401:
  credential absent or rejected

403:
  identity accepted, authorization denied

admission denial:
  authorization may have allowed request, but policy rejected object

timeout:
  network, webhook, or control-plane availability issue
```

Read the response reason and API audit logs.

---

# 13.40 Kerberos debugging quick sequence

```bash
klist
```

Confirm TGT.

```bash
kvno HTTP/reports.example.internal@EXAMPLE.INTERNAL
```

Confirm service-ticket issuance.

```bash
sudo klist -kte /etc/reports.keytab
```

Confirm service principal, key version, and supported key entries.

Enable controlled trace:

```bash
KRB5_TRACE=/dev/stderr \
  curl \
  --negotiate \
  -u : \
  https://reports.example.internal/
```

### Safety

The trace can expose identity and infrastructure information.

A successful service-ticket request does not prove that the server can decrypt or authorize it.

---

# 13.41 LDAP debugging quick sequence

## TLS first

```bash
openssl s_client \
  -connect directory.example.internal:389 \
  -starttls ldap \
  -servername directory.example.internal \
  -verify_hostname directory.example.internal \
  -verify_return_error
```

## Service Bind and search

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -x \
  -D 'uid=directory-reader,ou=Services,dc=example,dc=internal' \
  -W \
  -b 'ou=People,dc=example,dc=internal' \
  '(&(objectClass=person)(uid=alice))' \
  dn uid accountStatus
```

## Validate result count

```text
0:
  identity lookup failed

1:
  continue

more than 1:
  ambiguous; reject
```

## User Bind

Use a dedicated diagnostic command that prompts for the password.

Never put a production user password directly in the shell command.

---

# 13.42 Compare successful and failing requests

A single failing trace can be difficult to interpret.

Collect one:

```text
Known-good request
```

and one:

```text
Known-bad request
```

under controlled conditions.

Compare:

| Dimension       | Good           | Bad             |
| --------------- | -------------- | --------------- |
| Client host     | workstation-17 | workstation-18  |
| DNS result      | 192.0.2.80     | 192.0.2.81      |
| TLS issuer      | Workload CA 3  | Workload CA 2   |
| Token issuer    | production IdP | staging IdP     |
| Audience        | reports-api    | finance-web     |
| Subject         | u_1048         | u_1048          |
| Tenant          | tenant-42      | tenant-43       |
| Policy version  | 17             | 16              |
| Backend replica | reports-3      | reports-7       |
| Time            | synchronized   | 6 minutes ahead |

Change one variable at a time where possible.

---

# 13.43 Binary-search the request path

Suppose the path is:

```text
Browser
→ CDN
→ gateway
→ service mesh
→ application
→ policy engine
→ database
```

Test controlled points:

1. Browser to public endpoint
2. Diagnostic client to public endpoint
3. Gateway to application
4. Sidecar to application
5. Application to policy engine
6. Application to database

The first boundary where behavior changes identifies the likely fault domain.

Do not bypass security controls in production without explicit authorization.

---

# 13.44 Reproduce with minimum authority

Use a dedicated test principal:

```text
test-report-reader
```

with:

* One tenant
* One test resource
* Short-lived credential
* No administrative access
* Clear audit label
* Explicit expiry

Avoid debugging with a global administrator account.

An administrator success proves little about ordinary authorization.

---

# 13.45 Redaction strategy

Redaction should preserve diagnostic structure.

Bad redaction:

```text
Authorization: [REDACTED]
```

Better:

```text
Authorization:
  scheme = Bearer
  token_hash_prefix = sha256:91ac...
  length = 1284
```

For certificate:

```text
subject = [retained]
issuer = [retained]
serial = [retained]
private key = never collected
```

For cookie:

```text
name = __Host-session
domain = app.example.internal
path = /
secure = true
http_only = true
value = [REDACTED]
```

---

## Hashing credentials for correlation

A controlled debugging system may compute:

```text
SHA-256(raw credential)
```

to correlate the same credential across logs without storing it directly.

Considerations:

* Hash remains sensitive metadata.
* Low-entropy credentials can be guessed.
* The hashing process must not log the raw input.
* Use full digest internally; prefixes risk collisions.
* Retention should be limited.

Do not hash user passwords for diagnostic correlation outside the password-verification design.

---

# 13.46 Common debugging mistakes

## Mistake: testing only with administrator access

Result:

```text
Admin succeeds
→ assumption that system works
```

Object-level restrictions may remain broken for normal users.

---

## Mistake: disabling TLS verification

This changes the security protocol being tested.

A successful insecure request does not prove the secure configuration works.

---

## Mistake: copying production tokens to online decoders

This exposes bearer credentials to an external party.

Decode locally.

---

## Mistake: using `curl -v` in shared CI logs

Verbose output may include credentials.

Use controlled redaction and short-lived test credentials.

---

## Mistake: changing several controls simultaneously

Example:

```text
Add admin role
disable proxy
disable certificate checks
restart all replicas
```

The actual cause becomes unknown.

---

## Mistake: restarting before preserving evidence

Restarting may delete:

* In-memory sessions
* Connection state
* Temporary caches
* Process environment
* Failed credential state
* Relevant logs
* Core dumps

Preserve necessary evidence first.

---

## Mistake: treating `401` as a password failure

The actual credential may be:

* Session cookie
* OAuth token
* Client certificate
* Kerberos ticket
* API key

---

## Mistake: treating `403` as authentication failure

A valid authenticated subject may lack:

* Scope
* Role
* Tenant membership
* Resource ownership
* Required context

---

## Mistake: trusting client-side identity displays

A browser may display:

```text
Logged in as Alice
```

while the backend receives:

```text
anonymous
```

or another stale session.

Inspect server-side identity resolution.

---

## Mistake: trusting role claims without issuer context

```text
role=admin
```

has no safe meaning until the system establishes:

* Who issued it
* For which audience
* Under which tenant
* Using which policy version
* Whether the claim is current

---

# 13.47 End-to-end debugging decision tree

```mermaid
flowchart TD
    A[Request failed]
    B{Name resolves correctly?}
    C[Fix DNS or service discovery]
    D{TCP connection succeeds?}
    E[Fix route, firewall, listener, or port]
    F{TLS succeeds and peer identity matches?}
    G[Fix trust, certificate, SNI, time, or client auth]
    H{Expected HTTP endpoint reached?}
    I[Fix proxy, host, path, method, or redirect]
    J{Credential present?}
    K[Fix cookie, header, token retrieval, or client certificate selection]
    L{Credential validates?}
    M[Fix signature, issuer, audience, expiry, replay, or key rotation]
    N{Identity maps correctly?}
    O[Fix subject mapping, tenant, realm, or provisioning]
    P{Account/session policy allows?}
    Q[Fix account state, MFA, device, session, or risk policy]
    R{Authorization decision allows?}
    S[Fix scope, role, relationship, attributes, or policy]
    T{Enforcement operation succeeds?}
    U[Fix filesystem, database, downstream service, or atomicity]
    V{Response arrives unchanged?}
    W[Fix gateway, caching, redirect, or client handling]
    X[Operation completed]

    A --> B
    B -->|No| C
    B -->|Yes| D
    D -->|No| E
    D -->|Yes| F
    F -->|No| G
    F -->|Yes| H
    H -->|No| I
    H -->|Yes| J
    J -->|No| K
    J -->|Yes| L
    L -->|No| M
    L -->|Yes| N
    N -->|No| O
    N -->|Yes| P
    P -->|No| Q
    P -->|Yes| R
    R -->|No| S
    R -->|Yes| T
    T -->|No| U
    T -->|Yes| V
    V -->|No| W
    V -->|Yes| X
```

---

# 13.48 Scenario: browser loops between application and identity provider

Symptom:

```text
Application
→ IdP
→ callback
→ application
→ IdP again
```

Check in order:

1. Callback reaches correct application.
2. `state` validates.
3. Code exchange succeeds.
4. ID Token validates.
5. External subject maps to local user.
6. Local session record is created.
7. `Set-Cookie` is returned.
8. Browser accepts cookie.
9. Next request sends cookie.
10. Application resolves session.
11. Session keys are consistent across replicas.
12. Proxy preserves scheme and host.

Likely fault domains:

* Callback transaction
* Local session
* Cookie
* Load-balanced state
* Reverse-proxy configuration

---

# 13.49 Scenario: token works through gateway but fails directly

Possible explanation:

```text
Gateway validates external token
→ exchanges it for internal token
→ adds trusted context
```

Direct backend call uses the original external token.

Backend rejects:

* Wrong audience
* Missing mTLS
* Missing internal token
* Untrusted identity header
* Network policy

The direct call may be intentionally unsupported.

Do not weaken backend validation to make a diagnostic shortcut work.

---

# 13.50 Scenario: one replica returns 401

Compare failing and working replicas:

```text
Signing-key cache
Issuer configuration
Clock
Trust bundle
Token audience
Application version
Environment variables
Process start time
Network path
JWKS refresh status
```

A load balancer may make the problem appear intermittent.

Force a controlled target only when authorized, using:

* Diagnostic routing header
* `curl --resolve`
* Direct service endpoint
* Port forward in staging
* Replica-specific test service

---

# 13.51 Scenario: mTLS succeeds but application sees anonymous

Likely architecture:

```text
Client
→ mTLS proxy
→ backend
```

Check:

* Proxy verified certificate
* Proxy extracted correct SAN
* Proxy stripped client-supplied identity headers
* Proxy inserted trusted identity context
* Backend accepted traffic only from proxy
* Header name matched application configuration
* Backend did not overwrite context
* Direct path was blocked

Transport authentication may terminate before the application.

Identity must be propagated across that trust boundary securely.

---

# 13.52 Scenario: correct role but wrong tenant

Observed:

```text
role = administrator
tenant = tenant-42
resource tenant = tenant-43
```

A tenant-scoped administrator should still be denied.

Check whether:

* Tenant came from token
* Tenant came from URL
* Tenant came from database
* Membership exists
* Policy cache includes tenant
* Cross-tenant administration is separate
* Resource was loaded before tenant filtering

Do not resolve by granting a global role unless that is the intended business authority.

---

# 13.53 Scenario: password accepted but session not created

Evidence:

```text
password_verification = success
HTTP response = 500
```

Check:

* Session store availability
* Secure random generation
* Database transaction
* Cache connection
* Cookie-signing key
* Session record schema
* Concurrent-session limit
* User ID mapping
* Serialization
* Audit sink blocking request

Authentication success does not guarantee login completion.

---

# 13.54 Scenario: authorization allows but database returns no rows

Possible causes:

* Resource does not exist
* Tenant filter mismatch
* Row-level security
* Wrong database role
* Transaction context missing
* Read replica lag
* Soft deletion
* Data classification filter
* Policy and database use different identifiers

Capture:

```text
Application authorization decision
SQL parameters
database principal
tenant/session context
affected row count
database policy result
```

Do not log sensitive query values indiscriminately.

---

# 13.55 Scenario: logout appears ineffective

Possible layers:

```text
Browser cookie
Local application session
IdP session
Access token
Refresh token
API cache
Long-lived connection
```

Check which credential is still active.

Example:

```text
Local session revoked
but
IdP session remains
→ user is immediately signed in again
```

or:

```text
Refresh token revoked
but
existing access token remains valid for 10 minutes
```

Define logout scope precisely.

---

# 13.56 Scenario: user removed from group but retains access

Trace every representation:

```text
Directory group
→ directory replica
→ identity provider claim
→ token
→ application session
→ authorization cache
→ downstream token
→ database session
```

Possible persistence:

* Replication delay
* Cached group result
* Existing long-lived token
* Session role snapshot
* Policy cache
* Queued job
* Active transport connection

Revocation speed is determined by the slowest relevant layer.

---

# 13.57 Minimal incident evidence package

A useful package may include:

```text
Incident summary
Exact timestamps and time zones
Affected principal IDs
Action and resource
Expected and actual result
Request/trace IDs
Sanitized HTTP headers
Status code
Relevant application logs
Relevant gateway logs
Policy decision and version
Certificate metadata
Token metadata without raw token
DNS result
Destination address
Replica and deployment version
Clock status
Recent configuration changes
```

Do not automatically include:

* Raw packet capture
* Full process environment
* Complete database dump
* Private keys
* Credential files
* Entire directory export

Collect the minimum necessary evidence.

---

# 13.58 Practical investigation worksheet

```yaml
incident:
  summary:
  first_observed:
  environment:
  affected_users_or_workloads:
  business_impact:

request:
  method:
  target:
  action:
  resource:
  tenant:
  request_id:
  trace_id:

network:
  resolved_addresses:
  selected_address:
  tcp_result:
  proxy_or_mesh:

tls:
  peer_identity:
  issuer:
  serial:
  valid_from:
  valid_until:
  verification_result:
  client_certificate_used:

credential:
  type:
  issuer:
  audience:
  expiry:
  credential_hash:
  validation_stage:
  validation_result:

identity:
  external_subject:
  local_subject:
  actor:
  mapping_result:

session:
  session_hash:
  status:
  created_at:
  expires_at:
  store_region:

authorization:
  policy_id:
  policy_version:
  decision:
  reason:
  relevant_roles:
  relevant_attributes:

enforcement:
  downstream_service:
  database_principal:
  filesystem_identity:
  operation_result:

changes:
  recent_deployments:
  key_rotations:
  policy_changes:
  directory_changes:
```

---

# 13.59 Questions: Debugging Fundamentals

1. Why should debugging search for the earliest failed security invariant?
2. What evidence should be captured before restarting an authentication service?
3. Why can changing several permissions simultaneously make the root cause impossible to identify?
4. How does clock skew interfere with both protocol validation and log correlation?
5. Why should a request ID not be treated as proof of caller identity?
6. What is the difference between a missing credential and an invalid credential?
7. Why can a valid token still fail local account mapping?
8. How can authentication succeed while session creation fails?
9. Why must a denied authorization decision be verified at the enforcement layer?
10. How can an intermediary change a correct application response into a misleading client symptom?

---

# 13.60 Questions: Network and TLS Tools

1. Why can `dig` and `getent` return different results for the same hostname?
2. What does a successful `nc -vz host 443` test prove?
3. Why can a service listening on `127.0.0.1:443` be healthy locally but unreachable through a load balancer?
4. What information does `ss -ltnp` provide that a remote connection test does not?
5. Why is SNI required when several TLS sites share one IP address?
6. What does `-verify_return_error` change in an OpenSSL diagnostic?
7. Why is a trusted certificate chain insufficient without hostname or URI matching?
8. How can a missing intermediate certificate create different behavior on different clients?
9. Why should TLS verification not be disabled merely to determine whether HTTP works?
10. What packet pattern distinguishes a connection timeout from an immediate rejection?
11. Which information can a packet capture reveal despite TLS encryption?
12. Why should packet-capture filters be as narrow as possible?
13. How could IPv6 preference explain a failure when IPv4 testing succeeds?
14. What does `curl --resolve` preserve that directly requesting an IP address does not?
15. Why can a proxy produce a certificate from an unexpected issuer?

---

# 13.61 Questions: HTTP and Session Tools

1. Why should response headers and body often be saved separately?
2. What credential exposure can occur in `curl -v` or `--trace-ascii` output?
3. Why does a `302` response require inspection before automatic redirect following?
4. How can a browser reject a cookie that curl accepts?
5. Which cookie attributes should be inspected when login succeeds but the next request appears anonymous?
6. Why can `HttpOnly` make a cookie invisible to JavaScript while the browser still sends it?
7. What does a `401` from a gateway tell you about whether the application received the request?
8. Why can a `403` be the correct result for a valid administrator role?
9. How can two cookies with the same name but different paths produce intermittent behavior?
10. Why should logout testing identify each independent session and token layer?
11. How can a load balancer expose an application session-store consistency defect?
12. Why is a successful password-verification event not equivalent to a successful login?

---

# 13.62 Questions: Token and Identity Debugging

1. Why is decoding a JWT not evidence that its claims are trustworthy?
2. Which claims commonly identify issuer, audience, subject, and expiry?
3. How can a stale JWKS cache cause only newly issued tokens to fail?
4. Why should an unknown `kid` not trigger trust of a key embedded in the token?
5. How can forwarding a token to another service produce an audience mismatch?
6. Why is email a poor permanent key for federated identity mapping?
7. What information should be retained when a token is redacted?
8. How can a token be valid while its local account is disabled?
9. Why can role removal remain ineffective until a long-lived token expires?
10. How does token exchange make downstream debugging more complex but improve containment?
11. Which identity should a backend log when a service acts on behalf of a human?
12. How can a proxy correctly authenticate mTLS while the application still loses the peer identity?

---

# 13.63 Questions: Authorization and Enforcement Debugging

1. Why is inspecting only a user’s role insufficient for ABAC authorization?
2. What complete tuple should be captured for an authorization decision?
3. How can a missing policy attribute differ from a false attribute?
4. Why should the policy version appear in decision logs?
5. How can a correct gateway policy be bypassed through direct backend access?
6. Why may an authorization-aware database query return zero rows instead of an explicit denial?
7. How can row-level security deny a query after application authorization allows it?
8. Why must batch, export, and background-job paths be tested separately?
9. How can an authorization cache key omit a critical tenant dimension?
10. What evidence proves that a denied write produced no side effect?
11. Why should a test use a minimum-authority principal rather than a global administrator?
12. How would you compare one successful request and one failing request without changing several variables simultaneously?
13. Which caches may preserve access after group removal?
14. How can connection pooling retain an incorrect tenant or database identity?
15. What is the difference between an authorization decision failure and an enforcement failure?

# 14. Security, Performance, and Reliability Engineering

Authentication and authorization systems are security controls, but they are also distributed production systems.

They must remain:

```text
Correct
Secure
Available
Fast
Observable
Recoverable
```

A design that is cryptographically sound but frequently unavailable may prevent legitimate work.

A design that is fast but accepts stale or untrusted identity data may grant unauthorized access.

The engineering problem is therefore:

```text
Maintain trustworthy decisions
under attack, failure, scale, and change
```

---

# 14.1 Security goals

A useful starting point is to define the required security properties.

## Authentication goals

* Correct subjects can authenticate.
* Incorrect subjects cannot authenticate as someone else.
* Credentials are not unnecessarily disclosed.
* Authentication results cannot be replayed outside their intended context.
* Account recovery does not provide a weaker hidden bypass.
* Authentication strength matches operation risk.
* Compromise can be detected and contained.

## Authorization goals

* Every protected operation is checked.
* Decisions use trusted and current inputs.
* Permissions are no broader than required.
* Tenant and object boundaries are preserved.
* Policy changes are traceable.
* Revocation occurs within an acceptable period.
* Denied operations produce no protected side effects.

## Availability goals

* Legitimate users and workloads can authenticate.
* Identity dependencies tolerate expected failures.
* One malformed or expensive request cannot exhaust the system.
* Credential rotation does not cause widespread outages.
* Recovery procedures do not require disabling security controls.

---

# 14.2 Threat modeling

Threat modeling identifies:

```text
What must be protected?
From whom?
Through which trust boundaries?
With what consequences?
```

A practical threat model includes:

* Assets
* Actors
* Entry points
* Trust boundaries
* Credential types
* Security assumptions
* Abuse cases
* Failure impact
* Detection controls
* Recovery controls

---

## 14.3 Example assets

Authentication and authorization assets include:

* Password-verification records
* MFA seeds
* Passkey public-key registrations
* Recovery codes
* Session identifiers
* Refresh tokens
* API keys
* Signing keys
* Certificate-authority keys
* Service keytabs
* Authorization policies
* Group membership
* Tenant mappings
* Audit records
* Identity-provider configuration
* Trust bundles
* Account-recovery channels

Not every asset has the same confidentiality requirement.

Example:

```text
Public key:
  integrity-critical
  usually not confidential

Private key:
  integrity-critical
  confidentiality-critical
```

---

## 14.4 Threat actors

Possible actors include:

* Unauthenticated internet attacker
* Authenticated ordinary user
* Malicious tenant member
* Compromised browser
* Compromised workload
* Malicious administrator
* Insider with directory access
* Compromised CI/CD system
* Supply-chain attacker
* Stolen-device user
* Network attacker
* Cloud-account attacker
* Third-party identity-provider compromise

Threat models should not assume every request comes from an anonymous outsider.

Many authorization failures are exploitable by already authenticated users.

---

## 14.5 Trust-boundary diagram

```mermaid
flowchart LR
    U[User Device]
    E[Edge or Gateway]
    A[Application]
    P[Policy Service]
    I[Identity Provider]
    D[(Application Database)]
    R[(Directory)]
    K[Key Management System]

    U -->|Untrusted network| E
    E -->|Authenticated application traffic| A
    A -->|Policy request| P
    A -->|OIDC or token operations| I
    A -->|Data query| D
    I -->|Identity lookup| R
    I -->|Signing operation| K
```

Each arrow should answer:

* How is the caller authenticated?
* How is the target authenticated?
* Is confidentiality required?
* What data crosses the boundary?
* Can data be replayed?
* Can the caller choose identity attributes?
* What happens if the dependency is unavailable?

---

# 14.6 Abuse-case modeling

A normal use case says:

```text
Alice signs in and downloads her report.
```

An abuse case asks:

```text
Can Alice download Bob's report by changing the ID?
Can an attacker force Alice's browser to submit the request?
Can a stolen session be replayed from another device?
Can a compromised service request a broader token?
Can a support agent impersonate Alice without audit?
```

Good threat modeling creates concrete tests.

---

## Abuse-case table

| Abuse case                         | Required defense                             |
| ---------------------------------- | -------------------------------------------- |
| Guess passwords rapidly            | Rate limits, MFA, password hashing           |
| Reuse breached passwords           | Breach screening, MFA, monitoring            |
| Steal browser session              | Secure cookies, XSS controls, short exposure |
| Force browser action               | CSRF defenses, origin checks                 |
| Change object identifier           | Object-level authorization                   |
| Replay access token at another API | Audience validation                          |
| Forge identity header              | Trusted proxy boundary                       |
| Obtain another workload’s identity | Strong workload attestation                  |
| Abuse recovery flow                | Recovery assurance and alerts                |
| Persist after role removal         | Revocation and freshness controls            |
| Exfiltrate signing key             | HSM/KMS controls and key rotation            |

---

# 14.7 Security invariants

An invariant is a property that must remain true.

Examples:

```text
No application accepts identity headers from an untrusted network.
```

```text
Every report query includes a tenant authorization constraint.
```

```text
No refresh token appears in browser-accessible storage.
```

```text
Every accepted JWT has an expected issuer and audience.
```

```text
Every privileged action records actor and effective subject.
```

```text
Every password is stored using an approved password-hashing function.
```

Invariants are stronger than general recommendations because they can be:

* Tested
* Monitored
* Reviewed
* Enforced in deployment
* Included in incident analysis

---

# 14.8 Defense in depth

Defense in depth uses independent controls so that one failure does not immediately produce total compromise.

Example:

```text
Application object authorization
+
tenant-constrained SQL query
+
database row-level security
```

Another example:

```text
Short-lived access token
+
audience restriction
+
mTLS workload identity
+
resource-level authorization
```

Independence matters.

Three controls derived from the same untrusted header are not meaningful defense in depth.

---

## Correlated failure

Suppose:

```text
Gateway extracts X-Tenant-ID from request
Application trusts gateway tenant
Database session uses same tenant header
```

If the gateway fails to validate the header, every layer receives the same false input.

The layers appear separate but share one failure source.

---

# 14.9 Least privilege

Least privilege means a subject receives only the authority required for its current function.

Dimensions include:

* Actions
* Resources
* Tenant
* Environment
* Time
* Network
* Delegation
* Credential lifetime
* Maximum transaction size
* Administrative capabilities

Example:

```text
Bad:
  reports-service has database owner role

Better:
  SELECT on report tables
  INSERT on audit table
  no schema modification
  no user administration
```

---

## Temporal least privilege

Authority can be granted only when needed.

Example:

```text
Administrator receives production access
for 30 minutes
after approval and MFA
```

After expiry:

```text
Credential becomes invalid
role assignment is removed
active session is terminated where required
```

---

## Just-in-time privilege

A just-in-time system may require:

1. User requests privilege.
2. Policy checks role and context.
3. Approval occurs where required.
4. Step-up authentication completes.
5. Short-lived credential is issued.
6. Actions are audited.
7. Access automatically expires.

This reduces persistent administrative access.

---

# 14.10 Separation of duties

Separation of duties prevents one identity from controlling an entire sensitive process.

Examples:

```text
Developer writes code
Reviewer approves code
Deployment system releases code
Security administrator manages signing keys
```

```text
Payment creator
≠
Payment approver
```

```text
Identity administrator
≠
Audit-log administrator
```

Technical controls should enforce the separation rather than rely only on policy documents.

---

# 14.11 Minimize credential exposure

Credential exposure occurs through:

* Network transmission
* Browser storage
* Process memory
* Environment variables
* Command-line arguments
* Logs
* Crash dumps
* Support bundles
* Debug tools
* Backups
* Build artifacts
* Clipboard history
* Screenshots

For each credential, ask:

```text
Which components can read it?
For how long?
Why do they need it?
Can a reference or signing API replace direct access?
```

---

## Credential-handling matrix

| Credential           | Components that should normally see it               |
| -------------------- | ---------------------------------------------------- |
| User password        | User agent and designated verifier path              |
| Session cookie       | Browser and application session endpoint             |
| OAuth access token   | Client and intended resource server                  |
| Refresh token        | Client and authorization server                      |
| Private TLS key      | Workload, proxy, agent, or secure signer             |
| SAML assertion       | Browser front channel and intended SP                |
| Kerberos service key | KDC and intended service                             |
| Password-reset token | User, reset endpoint, protected store representation |

---

# 14.12 Avoid credentials in URLs

URLs may be recorded in:

* Browser history
* Proxy logs
* Referrer headers
* Analytics systems
* Load-balancer logs
* Screenshots
* Bookmarks

Do not place long-lived bearer credentials in query parameters.

Prefer protected headers or protocol-defined secure exchanges.

Authorization codes and reset tokens sometimes appear in URLs because they are designed as short-lived, constrained credentials.

They still require:

* Short lifetime
* Single use
* Redacted logs
* Exact redirect handling
* Replay protection

---

# 14.13 Secret storage hierarchy

A practical hierarchy, from weaker to stronger operational handling, may look like:

```text
Hard-coded source secret
< plaintext configuration file
< protected configuration file
< deployment secret store
< managed secret service
< short-lived dynamically issued credential
< hardware-backed non-exportable key
```

This is not absolute.

A badly configured secret manager may be less secure than a narrowly protected local file.

The complete security boundary matters.

---

# 14.14 Environment variables

Environment variables are convenient for secrets but may leak through:

* Process inspection
* Crash reporting
* Debug pages
* Child processes
* Service-manager output
* Container metadata
* Support scripts

Example risk:

```bash
ps eww -p "${PID}"
```

may expose process environment to authorized observers.

Use environment variables only with an understood threat model.

---

# 14.15 File-based secrets

A secret file can benefit from:

* Filesystem permissions
* Separate mount
* Rotation through atomic replacement
* Avoiding command-line exposure
* Limited process access

Inspect mode:

```bash
stat -c '%A %a %U %G %n' /run/secrets/database-password
```

Illustrative result:

```text
-r-------- 400 reports-api reports-api /run/secrets/database-password
```

File permissions do not protect against a compromised process already running as `reports-api`.

---

# 14.16 Memory exposure

Credentials must often exist temporarily in memory.

Risks include:

* Core dumps
* Process debugging
* Memory-disclosure vulnerabilities
* Swap
* Serialization
* Heap snapshots
* Language-runtime diagnostics

Mitigations may include:

* Avoid unnecessary copies
* Clear mutable buffers where practical
* Disable or protect core dumps
* Restrict process tracing
* Use non-exportable key interfaces
* Use short-lived credentials
* Separate credential-handling processes

Managed languages may make guaranteed memory clearing difficult.

Do not make claims of perfect secret erasure when runtime behavior cannot guarantee it.

---

# 14.17 Core-dump considerations

Inspect current core-dump resource limit:

```bash
ulimit -c
```

Possible output:

```text
0
```

indicates that the shell’s processes are configured not to produce traditional core files through this limit.

System-level crash services may have additional behavior.

Changing core-dump settings affects troubleshooting and incident response.

Balance:

```text
Diagnostic value
against
secret exposure
```

---

# 14.18 Logging security

Authentication logs must be useful without becoming a credential database.

Never log:

* Passwords
* Password-reset token values
* Session-cookie values
* Refresh tokens
* Full API keys
* Private keys
* MFA seeds
* Recovery codes
* Full SAML assertions
* Kerberos credential-cache contents

Usually safe and useful:

* Credential type
* Issuer
* Audience
* Key ID
* Token expiry
* Session hash
* Certificate serial
* Authentication method
* Policy version
* Decision reason
* Internal subject ID

---

## Log injection

Untrusted values may include:

```text
username
user agent
request ID
certificate subject
SAML attribute
resource name
```

An attacker may include newline or control characters.

Structured logging and bounded values help prevent:

* Forged log entries
* Parser confusion
* Terminal control sequences
* Excessive log volume
* Search-index abuse

---

# 14.19 Audit-log integrity

Audit logs may be evidence of:

* Privileged actions
* Impersonation
* Policy changes
* Credential issuance
* Account recovery
* Access denials
* Data exports

Protect them from:

* Deletion
* Modification
* Unauthorized reading
* Clock manipulation
* Selective event suppression

Possible controls:

* Append-oriented storage
* Separate administrative domain
* Remote collection
* Cryptographic integrity mechanisms
* Retention policy
* Restricted query access
* Alerting on gaps

---

# 14.20 Key management

Cryptographic keys may be used for:

* Token signing
* Token encryption
* TLS
* Certificate authority
* Cookie signing
* Database encryption
* Password peppering
* SAML signing
* SSH host authentication
* Kerberos service identity

Each key should have:

* Owner
* Purpose
* Algorithm
* Creation date
* Activation date
* Expiry or rotation date
* Storage location
* Access policy
* Backup policy
* Destruction policy
* Incident-response plan

---

# 14.21 Separate keys by purpose

Avoid using one key for unrelated functions.

Bad:

```text
One RSA key:
  signs ID Tokens
  decrypts application data
  signs SAML assertions
  authenticates TLS
```

Problems:

* Broader access
* Difficult rotation
* Algorithm constraints conflict
* Compromise affects several protocols
* Audit meaning is unclear

Prefer:

```text
OIDC signing key
SAML signing key
TLS private key
data-encryption key
```

---

# 14.22 Key hierarchy

A key hierarchy can reduce exposure.

```mermaid
flowchart TD
    R[Offline Root Key]
    I1[Production Token Intermediate]
    I2[Production Workload CA]
    I3[Staging Workload CA]
    L1[OIDC Signing Key]
    L2[Workload Leaf Certificates]
    L3[Staging Leaf Certificates]

    R --> I1
    R --> I2
    R --> I3
    I1 --> L1
    I2 --> L2
    I3 --> L3
```

The root can remain offline or highly restricted.

Operational issuers handle routine signing.

---

# 14.23 Hardware security modules and KMS

A hardware security module or managed key service may keep private-key material non-exportable.

The application requests:

```text
Sign this digest
```

rather than:

```text
Give me the private key
```

Benefits:

* Reduced key copying
* Central access controls
* Audit logs
* Hardware-backed isolation
* Rotation and versioning

Costs:

* Network or device dependency
* Latency
* Rate limits
* Availability coupling
* Vendor-specific behavior
* More complex disaster recovery

---

# 14.24 Key rotation

Safe rotation usually requires overlap.

Example token-signing rotation:

```text
1. Publish new public key.
2. Allow verifiers to cache it.
3. Begin signing new tokens with new private key.
4. Continue publishing old public key.
5. Wait until all old tokens expire.
6. Remove old public key.
7. Destroy or archive old private key according to policy.
```

```mermaid
timeline
    title Signing-Key Rotation
    09:00 : Publish public key B
    10:00 : Begin signing with key B
    10:00 : Continue accepting key A
    11:00 : Last key A token expires
    11:30 : Remove public key A
```

---

## Rotation failure modes

* New key used before publication
* Verifier cache does not refresh
* Old key removed too early
* Some issuers still sign with old key
* Key ID duplicated
* Clock skew extends token validity
* Rollback restores old signing key
* Disaster-recovery region lacks new key
* Algorithm configuration differs across replicas

---

# 14.25 Emergency key compromise

If a signing key is compromised:

1. Stop using the key.
2. Prevent new credential issuance.
3. Publish replacement trust material.
4. Remove compromised trust where possible.
5. Revoke or reject affected credentials.
6. Terminate sessions where required.
7. Rotate downstream credentials.
8. Investigate issuance and usage logs.
9. Determine compromise start time.
10. Communicate affected assurance limitations.

Short token lifetimes reduce but do not eliminate the response problem.

---

# 14.26 Password security review

A password system should be reviewed for:

* Approved password-hashing algorithm
* Cost parameters
* Unique salts
* Pepper handling where used
* Rehash-on-login
* Password-length handling
* Breached-password screening
* Rate limits
* Account enumeration resistance
* Reset security
* Recovery alerts
* MFA support
* Credential-stuffing detection

---

## Benchmark password hashing

A production parameter must be measured on representative hardware.

Conceptual Python benchmark:

```python
import statistics
import time

def benchmark(verify_once, iterations: int = 20) -> None:
    samples = []

    for _ in range(iterations):
        started = time.perf_counter()
        verify_once()
        samples.append(time.perf_counter() - started)

    print(f"median_ms={statistics.median(samples) * 1000:.1f}")
    print(f"max_ms={max(samples) * 1000:.1f}")
```

Measure:

* Median
* High percentile
* Concurrent load
* Memory use
* CPU saturation
* Container limits
* Slowest supported hardware

A single local timing is insufficient.

---

# 14.27 Authentication rate limiting

Rate limiting must balance:

```text
Attack resistance
against
legitimate availability
```

Dimensions may include:

* Source IP
* Account
* Device
* Tenant
* Network block
* Credential fingerprint
* Endpoint
* Client application

A single global limit can be abused to deny service to all users.

A strict per-account limit can be abused to lock out a victim.

---

## Layered example

```text
Per IP:
  100 attempts per 10 minutes

Per account:
  progressive delay

Per IP-account pair:
  10 attempts per 10 minutes

Global:
  protect hashing capacity

Risk-based:
  require stronger authentication
```

Exact values require traffic and threat analysis.

---

# 14.28 Token-bucket review

A token bucket maintains:

```text
capacity = B
refill rate = R tokens per second
current tokens = T
```

For each request:

```text
if T >= cost:
  T = T - cost
  allow
else:
  limit
```

This supports bursts while bounding sustained traffic.

Expensive operations can use higher cost.

Example:

```text
Password verification:
  cost 10

Username availability check:
  cost 1
```

---

# 14.29 Distributed rate limiting

A distributed system must decide where state lives.

Options:

* Gateway-local counters
* Shared central store
* Regional counters
* Approximate probabilistic counters
* Account-state record
* Identity-provider enforcement

Tradeoffs:

| Design              | Benefit    | Risk                                             |
| ------------------- | ---------- | ------------------------------------------------ |
| Local counter       | Fast       | Easy to bypass across replicas                   |
| Central counter     | Consistent | Shared latency and outage dependency             |
| Regional counter    | Scalable   | Cross-region aggregate may exceed intended limit |
| Approximate counter | Efficient  | Less precise decisions                           |

---

# 14.30 Retry behavior

Clients retry failures.

An uncontrolled retry loop can amplify an authentication outage.

Example:

```text
Token endpoint returns 503
10,000 clients retry immediately
→ token endpoint receives 10,000 more requests
→ outage worsens
```

Use:

* Exponential backoff
* Randomized jitter
* Maximum attempts
* Respect for `Retry-After`
* Circuit breakers
* Proactive refresh
* Request deduplication

---

## Exponential backoff

Conceptual delay:

```text
base × 2^attempt + random jitter
```

Example:

```text
Attempt 1: approximately 1 second
Attempt 2: approximately 2 seconds
Attempt 3: approximately 4 seconds
Attempt 4: approximately 8 seconds
```

Cap the maximum delay.

---

# 14.31 Credential refresh storms

Suppose 100,000 access tokens expire at exactly 12:00.

At 12:00:

```text
100,000 clients request refresh
```

Mitigations:

* Randomized issuance times
* Proactive refresh window
* Jitter
* Shared token cache for one client identity
* Single-flight refresh
* Regional token services
* Graceful server capacity

---

## Single-flight refresh

Within one application instance:

```text
Many requests discover token near expiry
→ one thread refreshes
→ others wait for result
```

Without coordination:

```text
100 concurrent requests
→ 100 refresh requests
```

---

# 14.32 Cache design

Authentication systems cache:

* Discovery metadata
* JWKS
* Certificate chains
* User attributes
* Group memberships
* Session records
* Introspection responses
* Authorization decisions
* DNS
* KDC tickets
* LDAP lookups
* Secret leases

Every cache needs:

* Key
* Value
* Source of truth
* TTL
* Invalidation strategy
* Failure behavior
* Maximum staleness
* Tenant context
* Policy version where relevant

---

# 14.33 Security cache equation

A cache trades freshness for speed and availability.

Conceptually:

```text
Longer TTL
→ fewer dependency calls
→ lower latency
→ slower revocation
```

```text
Shorter TTL
→ fresher decisions
→ higher dependency load
→ greater outage sensitivity
```

There is no universally correct TTL.

The required revocation objective determines the upper bound.

---

# 14.34 Cache-key completeness

Suppose authorization depends on:

```text
subject
tenant
action
resource
policy version
```

Unsafe key:

```text
subject
```

Safer key:

```text
subject | tenant | action | resource | policy_version
```

Missing one dimension can reuse an allow decision in the wrong context.

---

# 14.35 Negative caching

Negative caching stores failures such as:

```text
unknown user
unknown key ID
resource not found
token inactive
```

Benefits:

* Reduces repeated expensive lookups
* Limits some enumeration attacks
* Protects unavailable dependencies

Risks:

* Newly provisioned identity remains unavailable
* New signing key remains unknown
* Recently created resource appears absent
* Temporary failure becomes persistent

Use shorter TTLs for rapidly changing negative results.

---

# 14.36 JWKS caching

A verifier may cache issuer public keys.

Normal path:

```text
Token kid found
→ verify locally
```

Unknown key path:

```text
Unknown kid
→ refresh trusted issuer JWKS
→ retry once
→ reject if still unknown
```

Do not fetch an arbitrary URL from the token header.

The JWKS location should come from trusted issuer configuration or validated discovery.

---

## Unknown-key denial of service

An attacker sends many tokens with random `kid` values.

Naïve verifier:

```text
Every unknown kid
→ network refresh
```

Result:

```text
attacker drives unlimited key-fetch traffic
```

Defenses:

* Refresh cooldown
* Rate limits
* Negative key-ID cache
* Single-flight refresh
* Bounded issuer set
* Reject malformed tokens before fetch

---

# 14.37 Session-store performance

Every authenticated request may require:

```text
cookie parse
→ session ID hash
→ session lookup
→ expiry check
→ account check
→ optional idle update
```

Capacity planning should include:

* Requests per second
* Read latency
* Write latency
* Session count
* Record size
* Replication
* Expiry cleanup
* Cache hit rate
* Hot users
* Global logout operations

---

## Session-store hot keys

One session may receive high traffic.

If each request updates:

```text
last_seen_at
```

the record becomes a hot write key.

Mitigations:

* Coarse idle-update interval
* Separate activity store
* Local aggregation
* Asynchronous update
* Sliding expiry with bounded writes

---

# 14.38 Expiry cleanup

Expired sessions and tokens must be removed or ignored.

Approaches:

* Database TTL
* Background deletion
* Partition expiration
* Lazy deletion on access
* Time-indexed cleanup
* Cache-native expiry

A cleanup failure can create:

* Storage growth
* Slow indexes
* Backup expansion
* Operational costs

Security must not depend solely on physical deletion.

The validation path must reject expired records even before cleanup occurs.

---

# 14.39 Authorization-service performance

Policy evaluation may require:

```text
subject attributes
resource attributes
group graph
tenant membership
risk context
policy execution
```

Latency strategies:

* Local policy evaluation
* Sidecar policy engine
* Cached attributes
* Batch checks
* Precomputed relationships
* Query-aware authorization
* Partial evaluation
* Decision cache

---

## Policy-service failure behavior

Possible choices:

```text
Fail closed:
  deny when policy service unavailable

Fail open:
  allow when policy service unavailable

Use cached decision:
  accept recent known result

Degraded mode:
  permit low-risk reads, deny writes
```

This is a business and threat-model decision.

A blanket fail-open policy is dangerous.

A blanket fail-closed policy can cause total operational outage.

---

# 14.40 Risk-based degraded mode

Example:

```text
Policy service unavailable

Public catalog read:
  allow using local fallback

Existing user reads own non-sensitive profile:
  allow from recent cache

Payment approval:
  deny

Administrator privilege change:
  deny
```

The degraded policy must be defined and tested before an outage.

Do not invent it during an incident.

---

# 14.41 Availability architecture

Identity systems are critical dependencies.

Potential failure domains:

* Identity provider
* KDC
* Directory
* Session store
* Token issuer
* Policy service
* Key-management service
* Certificate issuer
* DNS
* Time service
* Network
* Region
* Cloud provider

Architecture should identify:

```text
Which failures prevent new login?
Which failures affect existing sessions?
Which failures affect token refresh?
Which failures affect every request?
```

---

# 14.42 Existing-session resilience

During an identity-provider outage:

```text
New OIDC login:
  fails

Existing local session:
  may continue

Access-token refresh:
  may fail

Already valid API token:
  may continue until expiry
```

Longer local sessions improve outage resilience but extend stolen-session exposure.

The design should set explicit policy.

---

# 14.43 Multi-region identity

A multi-region system may replicate:

* Session records
* User state
* Revocation state
* Authorization policy
* Signing keys
* Trust bundles
* Rate-limit counters

Questions:

* Which data requires strong consistency?
* Which data tolerates eventual consistency?
* Can each region issue credentials?
* Can every region revoke them?
* How quickly do policy changes propagate?
* What happens during network partition?

---

## Split-brain risk

Region A:

```text
Account disabled
```

Region B:

```text
Account still active
```

If both regions issue sessions independently, the subject may continue logging in through B.

Security-sensitive state may require:

* Single authoritative writer
* Consensus
* Strongly consistent datastore
* Region isolation during partition
* Short issuance lifetime
* Central revocation check

---

# 14.44 Availability versus immediate revocation

These goals can conflict.

Immediate revocation often requires:

```text
online state check
```

Offline-verifiable credentials provide:

```text
better runtime independence
```

but remain valid until expiry unless another mechanism is added.

Design choices include:

| Design                      | Availability          | Revocation speed          |
| --------------------------- | --------------------- | ------------------------- |
| Long-lived offline JWT      | High                  | Slow                      |
| Short-lived offline JWT     | High between renewals | Bounded by short lifetime |
| Introspection every request | Lower                 | Fast                      |
| Cached introspection        | Medium                | Cache-bounded             |
| Server-side session         | Store-dependent       | Fast if store reachable   |

---

# 14.45 Circuit breakers

A circuit breaker prevents repeated calls to a failing dependency.

States:

```text
Closed:
  calls flow normally

Open:
  calls fail quickly

Half-open:
  limited probes test recovery
```

For an authorization dependency, the circuit-breaker fallback must preserve the defined security posture.

---

# 14.46 Load shedding

When overloaded, reject work before total collapse.

Possible prioritization:

```text
Preserve:
  session validation
  token verification
  logout and revocation
  administrator lockout

Shed:
  expensive profile enrichment
  nonessential audit search
  bulk export
  background analytics
```

Authentication endpoints may use stricter per-client quotas while preserving capacity for legitimate traffic.

---

# 14.47 Backpressure

If an audit pipeline is slow, should login block?

Possible models:

### Synchronous mandatory audit

```text
Audit write fails
→ security operation fails
```

Strong record guarantee, but audit outage becomes authentication outage.

### Buffered audit

```text
Operation completes
→ event placed in durable local queue
```

Requires bounded buffering and recovery.

### Best-effort audit

```text
Operation completes even if audit is lost
```

Usually inappropriate for high-value administrative operations.

Choose by event criticality.

---

# 14.48 Queue saturation

A bounded queue protects memory.

When full, choose:

* Block producer
* Reject operation
* Drop low-value event
* Spill to durable storage
* Trigger degraded mode

Never silently drop privileged-action audit events without an explicit and monitored policy.

---

# 14.49 Performance metrics

Useful authentication metrics:

```text
authentication_attempts_total
authentication_success_total
authentication_failure_total
password_verification_duration_seconds
mfa_challenge_duration_seconds
session_lookup_duration_seconds
token_validation_duration_seconds
token_refresh_duration_seconds
oidc_login_duration_seconds
saml_validation_duration_seconds
kerberos_authentication_duration_seconds
```

Useful authorization metrics:

```text
authorization_decisions_total
authorization_denials_total
authorization_evaluation_duration_seconds
policy_dependency_duration_seconds
relationship_lookup_duration_seconds
decision_cache_hit_ratio
```

---

# 14.50 Percentiles

Average latency can hide slow requests.

Example:

```text
Average token validation:
  3 ms

p99:
  250 ms
```

Possible cause:

```text
Most validations use cached key
1% trigger remote JWKS refresh
```

Track:

* Median
* p90
* p95
* p99
* Maximum where useful

---

# 14.51 Avoid misleading metric labels

Do not use unbounded labels such as:

```text
user_id
session_id
request_id
token_id
resource_id
email
```

These create high-cardinality metric systems.

Prefer bounded labels:

```text
result = success|failure
stage = signature|audience|expiry
credential_type = session|jwt|mtls
region = known finite set
policy_id = controlled finite set
```

Use logs or traces for per-request identity.

---

# 14.52 Service-level indicators

Possible authentication SLIs:

```text
Successful login rate
Login latency
Token-issuance availability
Session-validation availability
Credential-renewal success rate
```

Possible authorization SLIs:

```text
Decision availability
Decision latency
Policy propagation delay
Revocation propagation delay
False-denial incident rate
Unauthorized-access incident rate
```

Security correctness is harder to reduce to one percentage.

Combine telemetry with testing and review.

---

# 14.53 Example objectives

Illustrative, not universal:

```text
99.9% of valid existing sessions are resolved within 50 ms.
```

```text
99.95% of token-validation requests complete without network dependency.
```

```text
Emergency account disablement reaches all enforcement points within 60 seconds.
```

```text
99% of workload certificates renew before 50% of their lifetime remains.
```

An objective must define:

* Population
* Time window
* Measurement point
* Exclusions
* Failure meaning

---

# 14.54 Revocation objective

A security-specific objective can be written as:

```text
From accepted disablement event
to rejection at all critical resource servers
must be less than 60 seconds.
```

Measure each stage:

```text
Directory update
→ identity event
→ session invalidation
→ token deny state
→ policy-cache invalidation
→ resource-server enforcement
```

---

# 14.55 Capacity planning

Estimate workload:

```text
Login attempts per second
Password-hash cost
Concurrent MFA challenges
Session reads per second
Session writes per second
Token refreshes per second
Policy evaluations per second
Directory queries per second
Certificate renewals per minute
```

Include:

* Normal peak
* Deployment surge
* Monday morning login peak
* Regional failover
* Credential refresh storm
* Attack traffic
* Retry amplification

---

# 14.56 Password-hash capacity example

Suppose:

```text
One password verification:
  150 ms CPU-equivalent

Available verification workers:
  100
```

Approximate maximum steady rate before other overhead:

```text
100 / 0.150
≈ 666 verifications per second
```

This is simplified.

Real capacity also depends on:

* Memory hardness
* CPU contention
* Worker scheduling
* Container limits
* Garbage collection
* Database lookup
* Rate-limit storage
* Dummy verification for unknown users

---

# 14.57 Calculator check

```bash
python3 - <<'PY'
workers = 100
seconds_per_verification = 0.150
capacity = workers / seconds_per_verification
print(f"approx_verifications_per_second={capacity:.1f}")
PY
```

This is a rough queueing estimate, not a production benchmark.

---

# 14.58 Queueing effects

As utilization approaches full capacity, wait time can rise sharply.

Example:

```text
Hash workers 95% busy
→ small traffic increase
→ large queue
→ login timeout
→ client retries
→ larger queue
```

Maintain headroom.

Rate limiting protects both security and availability.

---

# 14.59 Load testing authentication safely

Use:

* Dedicated environment
* Synthetic accounts
* Test credentials
* Controlled IP ranges
* Explicit rate limits
* No production email or SMS delivery
* Bounded test duration
* Monitoring
* Abort conditions

Do not load-test password endpoints against production without authorization.

---

## What to measure

* Throughput
* Latency percentiles
* Error classification
* CPU
* Memory
* Password-hash memory pressure
* Session-store latency
* Directory latency
* Token-issuer latency
* Queue depth
* Retry rate
* Lockout behavior

---

# 14.60 Security test categories

## Unit tests

Test:

* Claim validation
* Policy rules
* Password normalization
* Session expiry
* Role mapping
* CSRF-token comparison
* Tenant constraint construction

## Integration tests

Test:

* Identity provider
* Session store
* Directory
* Policy engine
* Database enforcement
* Certificate rotation

## Negative tests

Test:

* Missing credential
* Wrong issuer
* Wrong audience
* Expired token
* Replayed callback
* Wrong tenant
* Unauthorized object
* Stale group
* Direct backend access
* Malformed XML or JWT

## Adversarial tests

Test:

* Injection
* Race conditions
* Token substitution
* Session fixation
* Credential stuffing
* Cache poisoning
* Request smuggling effects
* Proxy-header spoofing
* XML signature-wrapping defenses

---

# 14.61 Property-based authorization tests

Instead of testing only examples, define properties.

Example:

```text
For every user in tenant A
and every resource in tenant B
where A != B
read must be denied
unless an explicit cross-tenant role exists.
```

Generate many combinations automatically.

This can find edge cases missed by hand-written tests.

---

# 14.62 Mutation testing

Authorization mutation testing deliberately changes security logic.

Examples:

```text
Change tenant equality to inequality
Remove audience check
Skip object ownership condition
Treat missing attribute as allow
```

The test suite should fail.

If tests still pass, they do not adequately cover the control.

---

# 14.63 Deployment safety

Authentication and authorization changes should use:

* Versioned configuration
* Code review
* Automated tests
* Staging
* Canary rollout
* Shadow evaluation
* Rollback
* Change audit
* Compatibility planning

Examples of high-risk changes:

* Signing-key rotation
* Cookie-domain change
* Session serialization change
* Password-hash parameter increase
* Identity claim mapping
* Role hierarchy
* Tenant policy
* CA rotation
* Service-principal rename

---

# 14.64 Canary deployment

A canary sends a small portion of traffic to the new version.

Monitor:

* Authentication success rate
* Authorization denial rate
* Unknown key IDs
* Session deserialization failures
* Login-loop rate
* Policy-decision differences
* Latency
* Error distribution by tenant

Do not evaluate only overall success rate.

A 1% tenant-specific outage may disappear inside a global average.

---

# 14.65 Shadow authorization

Candidate policy runs without enforcement.

Example:

```json
{
  "current_decision": "allow",
  "candidate_decision": "deny",
  "subject": "u_1048",
  "action": "report.download",
  "resource": "report-71",
  "candidate_policy_version": "18"
}
```

Analyze differences before activation.

Protect shadow logs because they reveal permission structure.

---

# 14.66 Backward compatibility

During rolling deployment, old and new versions coexist.

Potential incompatibilities:

* New session record cannot be parsed by old version
* Old version does not recognize new signing key
* New role claim is ignored
* Cookie name differs
* Token audience changes
* Certificate trust differs
* Policy input schema changes

Use versioned formats and overlap periods.

---

# 14.67 Session format migration

Suppose session schema changes:

```json
{
  "version": 2,
  "subject_id": "u_1048",
  "auth_context": {
    "methods": ["password", "totp"]
  }
}
```

Strategies:

* Readers support old and new formats.
* Writers emit new format after reader rollout.
* Old sessions are lazily upgraded.
* Sessions are deliberately invalidated.

Avoid a deployment where half the fleet creates sessions the other half cannot read.

---

# 14.68 Token-claim evolution

Adding a claim is usually easier than changing its meaning.

Unsafe evolution:

```text
roles:
  previously list of role names
  now list of permission names
```

Old consumers may interpret new values incorrectly.

Prefer:

* New claim name
* Explicit token version
* Audience-specific tokens
* Contract tests
* Controlled deprecation

---

# 14.69 Incident-response preparation

Authentication incidents require prewritten procedures.

Scenarios:

* Password database leak
* Session-signing key compromise
* OIDC signing-key compromise
* Refresh-token theft
* CA compromise
* Service keytab leak
* Administrator account takeover
* Directory group corruption
* Authorization-policy regression
* Audit-log loss

Each runbook should define:

* Detection
* Immediate containment
* Credential rotation
* Session invalidation
* Scope assessment
* Evidence preservation
* Communication
* Recovery validation

---

# 14.70 Session-signing key incident

Possible response:

1. Stop accepting newly forged sessions.
2. Rotate signing key.
3. Invalidate all sessions signed by compromised key.
4. Require reauthentication.
5. Investigate session creation and use.
6. Review access during compromise window.
7. Fix key exposure path.
8. Verify old key is removed from every replica and backup path.

If session records are server-side and the cookie only contains a random identifier, the incident response differs from a self-contained signed-cookie design.

---

# 14.71 Password database incident

Assess whether attackers obtained:

* Password hashes
* Salts
* Pepper
* Usernames
* MFA seeds
* Recovery codes
* Account metadata

Possible response:

* Reset affected passwords
* Rotate pepper if exposed
* Revoke sessions
* Require MFA
* Notify users
* Detect credential stuffing
* Increase monitoring
* Preserve evidence
* Review hashing parameters

A stolen strong password-hash database still enables offline guessing.

---

# 14.72 Refresh-token theft

Response may include:

```text
Revoke token family
Terminate related session
Require reauthentication
Inspect reuse events
Rotate downstream access
Alert user
Investigate client compromise
```

Do not only revoke the current access token if the attacker retains a refresh token.

---

# 14.73 Certificate-authority compromise

A compromised CA can issue trusted workload identities.

Response may require:

* Stop issuer
* Replace intermediate or root trust
* Distribute new trust bundles
* Reissue all valid certificates
* Reject compromised chain
* Restart or drain long-lived connections
* Rotate downstream credentials
* Review issued-certificate logs
* Investigate unauthorized identities

Root compromise is especially disruptive.

This is one reason to keep root keys highly restricted.

---

# 14.74 Recovery testing

A recovery plan that has never been tested is an assumption.

Test:

* Signing-key rotation
* Full session invalidation
* Identity-provider failover
* Directory outage
* CA rotation
* Policy rollback
* Audit pipeline failover
* Emergency administrator access
* Lost-device recovery
* Backup restoration

Use controlled exercises.

---

# 14.75 Break-glass access

Break-glass access is emergency privilege used when normal identity systems fail.

It must not become an ordinary administrative shortcut.

Controls may include:

* Separate credentials
* Hardware storage
* Dual control
* Physical access requirements
* Short activation
* Immediate alerting
* Full command audit
* Post-use rotation
* Periodic testing

---

## Break-glass tradeoff

Without emergency access:

```text
Identity outage
→ administrators cannot recover system
```

With weak emergency access:

```text
Attacker bypasses normal controls
```

The design must make emergency authority available but difficult to abuse silently.

---

# 14.76 Privacy and data minimization

Identity systems process personal information.

Collect only what is needed.

Example:

```text
Authorization needs:
  employee department

Token contains:
  home address
  date of birth
  personal phone
```

This creates unnecessary exposure.

Use:

* Minimal claims
* Audience-specific claims
* Short retention
* Purpose limitation
* Restricted audit access
* Pseudonymous identifiers where appropriate

---

# 14.77 Pairwise identifiers

A provider may issue different subject identifiers to different clients.

Example:

```text
Finance app:
  sub = p-81f2

HR app:
  sub = p-a731
```

This reduces cross-application correlation.

An authorized central system may retain the internal mapping.

Pairwise identifiers improve privacy but complicate account linking across applications.

---

# 14.78 Error-message privacy

External authentication error:

```text
Invalid credentials
```

Internal reason:

```text
account_exists=true
password_match=false
account_locked=false
```

Detailed external errors can enable:

* Account enumeration
* Group discovery
* Resource-existence discovery
* Policy reconnaissance

Internal detail should remain available to authorized operators.

---

# 14.79 Side-channel considerations

Observable differences may reveal information through:

* Response time
* Status code
* Response length
* Redirect destination
* MFA prompt
* Password-hash timing
* Error text
* Rate-limit behavior

Example:

```text
Known user:
  password hash takes 200 ms

Unknown user:
  immediate rejection in 2 ms
```

An attacker can enumerate accounts by timing.

Dummy verification reduces this difference.

---

# 14.80 Constant-time operations

Constant-time comparison is relevant when comparing secret-derived values such as:

* MACs
* Signatures in some contexts
* Reset-token hashes
* CSRF-token values
* API-key hashes

Use standard library functions.

Do not write a custom byte-by-byte early-exit comparison.

Network timing noise does not guarantee safety.

Attackers can repeat measurements.

---

# 14.81 Resource-exhaustion attacks

Authentication endpoints often perform expensive work.

Targets include:

* Password hashing
* XML parsing
* JWT key fetching
* LDAP searches
* Nested-group expansion
* Certificate path building
* Policy graph traversal
* Session-store writes
* SMS or email delivery

Apply:

* Input-size limits
* Structural limits
* Timeouts
* Rate limits
* Bounded recursion
* Bounded concurrency
* Caching
* Circuit breakers
* Work queues

---

# 14.82 Input-size limits

Examples:

```text
Username:
  bounded length

Password:
  supported maximum

JWT:
  bounded encoded size

SAML Response:
  bounded body size

Certificate chain:
  bounded number and size

LDAP filter:
  generated by application, not arbitrary user syntax
```

Reject oversized input before expensive parsing where possible.

---

# 14.83 XML security limits

For SAML and other XML:

* Disable external entities
* Disable arbitrary DTDs
* Limit entity expansion
* Limit depth
* Limit element count
* Limit document size
* Reject duplicate IDs
* Enforce expected schema structure
* Validate the exact consumed signed node

Security and performance controls overlap.

---

# 14.84 JWT complexity limits

A verifier should bound:

* Token length
* Number of segments
* Header size
* Claim count
* Nested JSON depth
* Audience list length
* Key-ID length
* Accepted algorithms

Reject malformed input before remote key lookup.

---

# 14.85 Certificate-chain limits

A malicious peer may send:

* Very long chain
* Large certificates
* Complex names
* Unsupported algorithms
* Invalid critical extensions

TLS libraries should enforce reasonable path and message limits.

Application code should not attempt unlimited alternate path construction.

---

# 14.86 Authorization graph limits

Relationship-based authorization may encounter:

* Deep nesting
* Cycles
* High fan-out
* Huge groups

Use:

* Maximum traversal depth
* Visited-set tracking
* Query budget
* Time limit
* Precomputed edges
* Explicit “unknown” or failure result

Do not convert computation timeout into allow.

---

# 14.87 Timeout design

Every network dependency needs an explicit timeout.

Examples:

```text
Directory lookup:
  500 ms

Policy request:
  100 ms

Token introspection:
  300 ms

Secret renewal:
  several seconds with background retries
```

Values depend on architecture.

A timeout should be shorter than the caller’s remaining deadline.

---

# 14.88 Deadline propagation

Request deadline:

```text
Client allows 2 seconds
```

Gateway spends:

```text
500 ms
```

Application should not give policy service another full 2 seconds.

It has approximately:

```text
1.5 seconds minus response overhead
```

Propagate bounded deadlines.

This prevents abandoned work after the caller has already timed out.

---

# 14.89 Cancellation

When a request is cancelled:

* Stop directory queries
* Stop policy graph traversal
* Avoid unnecessary token refresh
* Abort downstream database work where safe
* Release worker capacity

Cancellation must not leave partial security state.

Example:

```text
Session record inserted
but
response cancelled
```

The client may not know a session was created.

Transaction design should handle this deliberately.

---

# 14.90 Idempotency

Retries can repeat security-sensitive operations.

Examples:

* Password reset
* MFA enrollment
* Token exchange
* Payment approval
* Session revocation
* Role assignment

Use idempotency keys or single-use state where appropriate.

Example:

```text
Authorization code:
  single use

Password-reset token:
  single use

Role-change request:
  idempotency key
```

---

# 14.91 Race conditions

Common races include:

```text
Two refresh requests use the same refresh token.
```

```text
Role removed while request is authorized.
```

```text
Session revoked while another node refreshes cache.
```

```text
Password reset and login occur concurrently.
```

```text
Certificate rotates while server reloads key files.
```

Use transactions, versioning, locks, atomic updates, or token-family logic as appropriate.

---

# 14.92 Refresh-token race example

Requests A and B simultaneously present token R1.

Unsafe behavior:

```text
A receives R2
B receives R3
Both remain active
```

Safer designs may:

* Serialize token-family use
* Permit a narrow retry grace window
* Detect reuse after committed rotation
* Revoke family on suspicious reuse

The correct policy must account for legitimate client concurrency.

---

# 14.93 Session-revocation race

Request:

```text
1. Session cache says active.
2. Administrator revokes session.
3. Request performs sensitive write.
```

Possible controls:

* Check revocation inside transaction
* Require fresh authorization for high-risk writes
* Use session generation
* Invalidate caches synchronously
* Accept a documented revocation delay

Absolute immediate revocation across distributed systems is expensive.

Define the required bound.

---

# 14.94 Policy-update race

Policy version 17 allows action.

Version 18 denies it.

During rollout:

```text
Region A uses 18
Region B uses 17
```

A user sees inconsistent decisions.

Controls:

* Version-pinned request
* Atomic policy activation
* Central decision service
* Deployment barrier
* Shadow evaluation
* Explicit propagation monitoring

---

# 14.95 Secure defaults

Default behavior should be safe when configuration is incomplete.

Examples:

```text
Unknown issuer:
  reject

Unknown token algorithm:
  reject

Missing tenant:
  reject

Missing required policy attribute:
  reject or explicit indeterminate handling

Unrecognized session version:
  reject

TLS verification failure:
  stop

Unknown client certificate:
  reject
```

A default allow should require a deliberate and documented reason.

---

# 14.96 Configuration validation

At startup, validate:

* Issuer URLs
* Allowed audiences
* Redirect URIs
* Cookie settings
* Key availability
* Trust bundles
* Session-store connectivity
* Policy schema
* Required secrets
* Timeout ranges
* Allowed algorithms
* Tenant configuration

Failing startup can be safer than running with partial security configuration.

Availability design should provide redundant valid configuration rather than unsafe fallback.

---

# 14.97 Configuration provenance

Operators should know:

```text
Where did this setting come from?
```

Possible sources:

* Built-in default
* Configuration file
* Environment variable
* Secret store
* Command-line flag
* Remote configuration service
* Deployment override

Log non-secret effective configuration provenance.

Do not log secret values.

---

# 14.98 Algorithm agility

Cryptographic algorithms and key sizes change over time.

A system should support controlled migration without accepting arbitrary peer choices.

Use:

* Explicit allowlists
* Versioned configuration
* Overlap during migration
* Inventory of consumers
* Metrics for old-algorithm use
* Removal deadline

Do not let the token or certificate alone choose any algorithm the library supports.

---

# 14.99 Dependency and library security

Authentication libraries are high-value dependencies.

Manage:

* Version inventory
* Security advisories
* Reproducible builds
* Signature or provenance verification
* Update testing
* Transitive dependencies
* Unsupported versions
* Configuration changes across upgrades

Avoid implementing complex standards manually unless necessary and reviewed.

---

# 14.100 Supply-chain risk

An attacker compromising:

* Identity SDK
* OAuth library
* SAML parser
* Container base image
* CI runner
* Package repository

may capture credentials or bypass validation.

Controls include:

* Dependency pinning
* Artifact signing
* Build isolation
* Software bills of materials
* Minimal build credentials
* Review of release provenance
* Runtime egress restrictions
* Secret-free build stages

---

# 14.101 Administrative-plane security

Identity administration can grant broad access.

Protect actions such as:

* Create administrator
* Add user to privileged group
* Change redirect URI
* Upload signing certificate
* Add trusted issuer
* Modify authorization policy
* Disable MFA
* Create recovery code
* Issue service credential
* Change audit retention

Require:

* Strong authentication
* Step-up
* Separation of duties
* Detailed audit
* Alerts
* Change review
* Short administrative sessions

---

# 14.102 Trusted-configuration changes

A redirect URI or trust-anchor change can be equivalent to credential compromise.

Example:

```text
Attacker changes OIDC redirect URI
→ authorization codes delivered to attacker
```

```text
Attacker adds new trusted CA
→ attacker-issued workload certificates accepted
```

Treat configuration administration as a security-critical operation.

---

# 14.103 Monitoring anomalies

Potential signals:

* Sudden increase in failed logins
* Password spraying across many users
* One account from many regions
* Refresh-token reuse
* New certificate issuer
* Unknown key IDs
* Unusual service-to-service edge
* Administrator action outside normal hours
* Large data export
* Role assignment followed by immediate access
* Session use after password reset
* ServiceAccount token used from unexpected network

Signals require context to avoid excessive false positives.

---

# 14.104 Credential-stuffing pattern

Illustrative pattern:

```text
Many username-password pairs
Few attempts per account
Many accounts
Distributed source addresses
```

Per-account lockout alone may miss it.

Detection can use:

* Known breached credentials
* Device and network reputation
* Cross-account patterns
* Failed-login velocity
* Successful login after failures
* MFA challenge outcomes

---

# 14.105 Password-spraying pattern

Illustrative pattern:

```text
One or a few passwords
tested across many accounts
```

Signals:

* Same password cannot normally be logged, so detect indirectly
* Similar timing and source
* Many account failures
* Low attempts per account
* Common target groups
* Authentication-protocol similarity

Avoid logging raw attempted passwords.

---

# 14.106 Session-theft signals

Possible indicators:

* Session appears from distant locations rapidly
* User-agent changes abruptly
* Concurrent activity inconsistent with normal use
* Session used after device revocation
* Sensitive operation without recent authentication
* CSRF or XSS indicators
* Session ID reused after logout

IP changes alone are not proof.

Use risk signals proportionally.

---

# 14.107 Automated response

Possible actions:

* Require step-up authentication
* Revoke one session
* Revoke all sessions
* Suspend token family
* Reduce transaction limits
* Block source network
* Alert user
* Require password reset
* Escalate to operator review

Automated controls can be abused for denial of service.

Use graduated responses.

---

# 14.108 Security review checklist

## Authentication

* Credentials are appropriate to risk.
* Passwords use approved hashing.
* MFA and recovery are designed together.
* Rate limits resist abuse without easy lockout.
* Authentication errors avoid enumeration.
* Sessions rotate after authentication.
* Credentials are not logged.

## Tokens

* Issuer and audience are validated.
* Token type is enforced.
* Algorithms are allowlisted.
* Keys rotate safely.
* Expiry and clock skew are bounded.
* Refresh-token reuse is handled.
* Browser storage is intentional.

## Authorization

* Default deny.
* Object and tenant checks exist.
* Policy inputs are trusted.
* Cache keys are complete.
* Revocation objectives are defined.
* Batch and background paths are tested.
* Actor and subject are preserved.

## Infrastructure

* Workloads have distinct identities.
* Static secrets are minimized.
* mTLS identities are matched explicitly.
* Trust domains are separated.
* ServiceAccount permissions are narrow.
* Long-lived connections have revocation policy.

## Operations

* Logs are useful and credential-safe.
* Key compromise runbooks exist.
* Failover is tested.
* Changes are versioned.
* Metrics cover correctness and latency.
* Break-glass access is controlled.

---

# 14.109 Performance review checklist

* Latency percentiles are measured.
* Expensive operations have concurrency limits.
* Caches have explicit freshness bounds.
* Retry behavior uses backoff and jitter.
* Credential refresh is proactive.
* Session writes are bounded.
* Directory searches are indexed.
* Relationship traversal is bounded.
* Timeouts are shorter than caller deadlines.
* Failure modes avoid retry storms.
* Regional failover capacity is tested.
* Security-critical revocation propagation is measured.

---

# 14.110 Reliability failure matrix

| Dependency failure     | New login                         | Existing session                     | API token validation                          | Authorization               |
| ---------------------- | --------------------------------- | ------------------------------------ | --------------------------------------------- | --------------------------- |
| Identity provider down | Often fails                       | May continue                         | Existing offline token may continue           | Usually unaffected directly |
| Session store down     | Login may fail                    | Often fails                          | Token-only APIs may continue                  | Depends on architecture     |
| JWKS endpoint down     | Cached keys may work              | Unaffected                           | New key may fail                              | Unaffected after identity   |
| Policy service down    | Login may succeed                 | Session may resolve                  | Token may validate                            | May deny or degrade         |
| Directory down         | Password/directory login may fail | Cached identity may continue         | Existing token may continue                   | Group freshness affected    |
| KMS down               | New signing may fail              | Existing signed credentials may work | Verification may work with cached public keys | Depends on key use          |
| Time service failure   | Increasingly unsafe               | Expiry errors                        | Token/certificate failures                    | Time-based policy errors    |

---

# 14.111 Example production design

Consider a finance application:

```text
Browser
→ BFF
→ Reports API
→ Policy Engine
→ Database
```

Security design:

```text
Browser:
  HttpOnly, Secure, SameSite session cookie

BFF:
  server-side session
  access tokens stored backend-side
  CSRF protection
  step-up for exports

Reports API:
  validates audience-specific token
  authenticates BFF workload using mTLS
  performs object-level authorization

Policy:
  subject, tenant, resource, classification
  versioned rules
  default deny

Database:
  tenant-constrained query
  row-level security
  least-privilege database role
```

Reliability design:

```text
Session store:
  multi-zone

Token verification:
  cached public keys

Policy:
  local cached policy bundle
  bounded attribute cache

Revocation:
  session generation
  short access-token lifetime
  emergency deny list

Audit:
  durable buffered pipeline
```

---

# 14.112 End-to-end security reasoning

Alice requests report 71.

The system should be able to explain:

```text
1. Browser possessed an active session cookie.
2. BFF resolved it to local user u_1048.
3. Session authentication used password plus TOTP.
4. Account remained active.
5. BFF obtained an access token for reports-api.
6. Reports API validated issuer, audience, expiry, and signature.
7. mTLS authenticated BFF workload.
8. Policy matched subject tenant to report tenant.
9. Report classification was within user clearance.
10. Database query included tenant constraint.
11. Audit event recorded actor, subject, policy version, and report.
```

Security is the complete chain.

Not one isolated token check.

---

# 14.113 Questions: Threat Modeling and Security Architecture

1. What assets in an authentication system require confidentiality, and which mainly require integrity?
2. Why must threat models include already authenticated malicious users?
3. What makes a control independent enough to count as defense in depth?
4. How can several layers inherit one correlated failure from a forged tenant header?
5. What is the difference between least privilege and temporal least privilege?
6. How does just-in-time administration reduce persistent attack surface?
7. Which high-value operations should require separation of duties?
8. Why is a trusted redirect-URI configuration as security-sensitive as a credential?
9. What security invariant would prevent direct spoofing of reverse-proxy identity headers?
10. How would an abuse case differ from a normal use case?

---

# 14.114 Questions: Credential and Key Security

1. Why is a secret stored in an environment variable still exposed to several system components?
2. What security benefit does a non-exportable signing key provide?
3. Why should signing and encryption keys normally be separated by purpose?
4. What overlap is required during signing-key rotation?
5. Why must an old public key remain available after new tokens begin using a new key?
6. What happens if a compromised signing key remains trusted until every old token expires?
7. Why does short credential lifetime not eliminate the need for incident response?
8. Which credentials should never appear in debug logs?
9. How can hashing a credential assist correlation while still creating sensitive metadata?
10. Why can a hardware security module improve key protection while reducing availability?
11. What should be rotated after a service keytab is stolen?
12. How does a CA compromise differ in impact from one leaf-certificate compromise?
13. Why must backup and disaster-recovery systems be included in key deletion plans?
14. What configuration mistakes can make key rotation fail on only some replicas?
15. Why should algorithm selection come from local policy rather than an untrusted token header?

---

# 14.115 Questions: Availability and Caching

1. Why do longer authorization-cache TTLs slow revocation?
2. Which dimensions belong in an object-level authorization cache key?
3. How can negative caching delay a newly provisioned identity?
4. Why can random `kid` values create a JWKS-refresh denial of service?
5. How should a verifier respond to an unknown key ID without fetching arbitrary URLs?
6. Why can a session `last_seen_at` update create a hot storage key?
7. What security control must remain active even if expired-session cleanup is delayed?
8. How does an introspection-based token design trade availability for revocation speed?
9. Which operations could continue during an identity-provider outage?
10. Why can a network partition create contradictory account state across regions?
11. What is the difference between fail closed, fail open, cached fallback, and degraded mode?
12. Which low-risk operation might reasonably continue when a central policy service is unavailable?
13. Why should degraded authorization policy be designed before an outage?
14. How does a circuit breaker prevent retry amplification?
15. Why must retry delays include jitter?

---

# 14.116 Questions: Performance and Capacity

1. Why can average authentication latency hide a serious user-facing problem?
2. What does p99 latency reveal that median latency does not?
3. How can password hashing become an application-level denial-of-service target?
4. Why is concurrency capacity as important as single-request password-hash time?
5. How can token expiry synchronization create a refresh storm?
6. What problem does single-flight token refresh solve?
7. Why should certificate-renewal times be randomized?
8. How can long-lived connection reuse improve performance while delaying revocation?
9. Why must every authentication dependency have a timeout?
10. How should a downstream deadline relate to the caller’s remaining deadline?
11. What happens when cancelled authentication work continues in the background?
12. Why should relationship-graph traversal have a query budget?
13. How can a large LDAP group become both a performance and authorization correctness problem?
14. Why does load testing require synthetic minimum-authority identities?
15. Which capacity assumptions must change during regional failover?

---

# 14.117 Questions: Testing and Deployment

1. What authorization property would prove tenant isolation across arbitrary subjects and resources?
2. How does mutation testing reveal missing security test coverage?
3. Why should a candidate policy run in shadow mode before enforcement?
4. What information should be compared between current and candidate policy decisions?
5. How can a rolling deployment break session compatibility?
6. Why should session formats include a version?
7. What can happen if a token claim changes meaning without changing its name?
8. Which metrics should be monitored during an identity-related canary rollout?
9. Why can a global success rate hide a complete outage for one tenant?
10. What authentication changes require compatibility overlap?
11. How would you test full session invalidation safely?
12. Why must recovery and key-rotation procedures be exercised rather than merely documented?

---

# 14.118 Questions: Incident Response and Monitoring

1. What immediate actions follow compromise of a session-signing key?
2. Why might all active sessions need invalidation after such a compromise?
3. What additional risk exists if password hashes and a password pepper are stolen together?
4. Why must refresh-token theft response revoke more than the current access token?
5. What trust material must change after an intermediate workload CA compromise?
6. Why may active mTLS connections survive CA trust removal?
7. What controls make break-glass access available without making it invisible?
8. Which anomaly can indicate refresh-token replay?
9. Why is a sudden increase in authorization denials not always an attack?
10. How can automatic account lockout become a denial-of-service mechanism?
11. What actor and subject information should appear in an impersonation alert?
12. How would you measure emergency revocation propagation across a distributed system?
13. What evidence must be preserved before rotating compromised keys?
14. Why should privileged audit logs be administered separately from identity policy?
15. How can monitoring detect that one replica is using stale trust material?

# 15. Advanced Authentication and Authorization Internals

At an advanced level, authentication and authorization are not isolated checks.

They are transformations of security state across several systems:

```text id="53zg0d"
Physical or cryptographic evidence
→ authenticated principal
→ local identity
→ delegated authority
→ session or credential
→ authorization context
→ enforced operation
→ audit evidence
```

Each transformation can weaken, strengthen, or accidentally change the meaning of the original identity.

The most important advanced question is:

> What exactly is cryptographically or operationally bound to what?

---

# 15.1 Security context as a state machine

Authentication is better modeled as a state machine than as one Boolean field.

A simplistic model is:

```text id="y8mxhg"
authenticated = true
```

A more realistic context includes:

```json id="kp7mjq"
{
  "principal": "u_1048",
  "actor": "finance-web",
  "issuer": "https://identity.example.internal",
  "authentication_time": "2026-07-31T11:31:02Z",
  "authentication_methods": ["password", "totp"],
  "assurance_level": "corporate-mfa",
  "device_trust": "managed",
  "tenant": "tenant-42",
  "session_generation": 18,
  "delegation_depth": 1,
  "impersonation": false
}
```

A request can move through states such as:

```mermaid id="kik89x"
stateDiagram-v2
    [*] --> Anonymous
    Anonymous --> PrimaryAuthenticated: Password or passkey accepted
    PrimaryAuthenticated --> MFAAuthenticated: Second factor accepted
    MFAAuthenticated --> SessionEstablished: Session created
    SessionEstablished --> StepUpRequired: Sensitive action requested
    StepUpRequired --> HighAssurance: Recent strong authentication
    SessionEstablished --> Revoked: Logout or administrator revocation
    HighAssurance --> Revoked
    SessionEstablished --> Expired: Idle or absolute expiry
    HighAssurance --> Expired
    Revoked --> [*]
    Expired --> [*]
```

Authorization policy should use the appropriate state.

Example:

```text id="oqb6ln"
View dashboard:
  session established

Export all customer records:
  recent high-assurance authentication
  managed device
  approved administrator role
```

---

## Authentication state must not only increase

A common mistake is assuming a session can only become stronger.

It may become weaker because:

* MFA enrollment was removed.
* Device became unmanaged.
* Account was suspended.
* Password was reset.
* Risk increased.
* Session became too old.
* Trusted network condition changed.
* Delegated authority expired.
* Administrator role was removed.

The state machine needs downward transitions:

```text id="cdj77v"
high assurance
→ ordinary session
→ restricted session
→ revoked
```

---

# 15.2 Authentication event versus continuing session

An authentication event occurred at a particular time.

A session represents continued reliance on that event.

```text id="sj79jd"
Authentication event:
  Alice proved password and TOTP at 11:31

Session:
  Application continues accepting requests until policy says otherwise
```

The session is not repeated authentication.

It is a cached security conclusion.

The session’s validity depends on assumptions such as:

* Session credential remains secret.
* Account remains active.
* Authentication age remains acceptable.
* Device state remains acceptable.
* Session record has not been revoked.
* Security policy has not changed incompatibly.

---

## Security-context freshness

Different properties can have different freshness requirements.

| Property                | Possible freshness requirement |
| ----------------------- | ------------------------------ |
| Password authentication | Within session lifetime        |
| MFA                     | Within 12 hours                |
| High-risk step-up       | Within 5 minutes               |
| Group membership        | Within 10 minutes              |
| Device compliance       | Within 2 minutes               |
| Account enabled status  | Near-real-time                 |
| Resource ownership      | At transaction time            |

A single session creation timestamp cannot represent all of these.

---

# 15.3 Protocol composition

Most production flows compose several protocols.

Example:

```text id="dnrq5p"
Browser session
→ OIDC login
→ OAuth access token
→ mTLS workload identity
→ database role
```

Each protocol authenticates or authorizes a different relationship.

```mermaid id="qu3fc9"
flowchart LR
    U[Human User]
    B[Browser Session]
    F[Frontend Workload]
    T[OAuth Token]
    A[API Workload]
    D[Database Principal]

    U -->|OIDC authentication| B
    B -->|Local session cookie| F
    F -->|OAuth delegation| T
    F -->|mTLS identity| A
    A -->|Database authentication| D
```

A protocol-composition defect occurs when one layer assumes another layer proved more than it actually did.

---

## Composition example: trusted gateway

Architecture:

```text id="0ucgl1"
Client
→ gateway validates access token
→ gateway sends request to backend
```

Gateway adds:

```http id="zm1xp9"
X-Authenticated-Subject: u_1048
X-Authenticated-Tenant: tenant-42
```

The backend can trust these only if:

1. The gateway removes client-supplied copies.
2. The backend is unreachable except through the gateway.
3. Gateway-to-backend transport is authenticated.
4. Header format and encoding are unambiguous.
5. Gateway policy is compatible with backend assumptions.
6. Backend treats missing headers as unauthenticated.
7. Multiple gateways use equivalent validation policy.

Otherwise, identity becomes forgeable at the composition boundary.

---

# 15.4 Context binding

A credential is safer when it is bound to a specific context.

Possible bindings include:

* Issuer
* Audience
* Client
* Redirect URI
* Browser transaction
* TLS channel
* Public key
* Device
* Workload
* Resource
* Tenant
* Request method
* Request URL
* Time window

A generic bearer credential has fewer bindings.

---

## Binding examples

### Authorization code

May be bound to:

```text id="31x2ud"
client ID
redirect URI
PKCE challenge
short expiry
single-use state
```

### ID Token

May be bound to:

```text id="0rbyke"
issuer
client audience
authentication request nonce
time interval
```

### Access token

May be bound to:

```text id="5h8xlr"
issuer
resource audience
scope
subject
client
confirmation key
```

### SAML assertion

May be bound to:

```text id="3qn3k8"
issuer
audience
recipient
destination
request ID
time interval
```

### Kerberos ticket

May be bound to:

```text id="42c4ei"
client principal
service principal
session key
time
ticket flags
```

---

# 15.5 Bearer credentials

A bearer credential works because the presenter possesses it.

```text id="5lvwrr"
possess credential
→ may exercise represented authority
```

Examples:

* Session ID
* OAuth bearer token
* API key
* Password-reset token
* SAML bearer assertion
* Some signed cookies

Security depends heavily on preventing copying.

---

## Bearer-token theft

Suppose an access token is stolen from application logs.

The attacker may be able to send:

```http id="y3m3xp"
Authorization: Bearer STOLEN_TOKEN
```

The resource server cannot distinguish:

```text id="8e34nm"
legitimate client
from
attacker holding same bytes
```

unless another binding mechanism exists.

---

# 15.6 Proof-of-possession credentials

A proof-of-possession credential requires both:

```text id="6zi5eq"
Credential
+
proof of control of associated key
```

The token may contain or reference a confirmation key.

Conceptual token:

```json id="xy1ff8"
{
  "iss": "https://identity.example.internal",
  "sub": "u_1048",
  "aud": "reports-api",
  "scope": "reports.read",
  "cnf": {
    "jkt": "thumbprint-of-client-public-key"
  }
}
```

For each request, the client proves control of the corresponding private key.

A copied token alone is insufficient.

---

## Proof-of-possession flow

```mermaid id="59g6xk"
sequenceDiagram
    participant Client
    participant AS as Authorization Server
    participant API as Resource Server

    Client->>Client: Generate or access key pair
    Client->>AS: Request token bound to public key
    AS-->>Client: Access token containing key confirmation
    Client->>Client: Sign request-specific proof
    Client->>API: Token + signed proof
    API->>API: Validate token
    API->>API: Validate proof and key binding
    API-->>Client: Response
```

---

# 15.7 Request-bound proofs

A request proof can include:

```text id="tfc20g"
HTTP method
target URL
issued-at time
unique proof ID
access-token hash
public-key identity
```

This can prevent a proof created for:

```text id="m6xfhq"
GET https://api.example/reports/71
```

from being reused for:

```text id="4bvjnm"
DELETE https://api.example/reports/71
```

or:

```text id="67s9pw"
POST https://payments.example/refund
```

---

## Proof validation

The resource server checks:

1. Signature is valid.
2. Proof public key matches token confirmation data.
3. Method matches actual request.
4. Target URI matches actual request.
5. Proof is recent.
6. Proof identifier has not been replayed.
7. Token hash matches where required.
8. Algorithm is allowed.
9. Proxy canonicalization is handled safely.

---

# 15.8 mTLS-bound tokens

An authorization server can bind an access token to a client certificate.

Conceptual token:

```json id="cs10f4"
{
  "sub": "orders-service",
  "aud": "payments-api",
  "cnf": {
    "x5t#S256": "certificate-thumbprint"
  }
}
```

At the resource server:

```text id="9jix8g"
Presented token certificate thumbprint
must equal
authenticated TLS client certificate thumbprint
```

A stolen token cannot be used without the corresponding private key and certificate.

---

## Proxy complications

If mTLS terminates at a gateway:

```text id="z4auln"
Client
→ mTLS gateway
→ backend resource server
```

The backend does not directly see the client certificate.

The gateway must securely convey:

* Validated certificate identity
* Certificate thumbprint or confirmation value
* Authentication result

The backend must trust only the authenticated gateway path.

This moves token-binding enforcement into the gateway or a securely composed gateway-backend protocol.

---

# 15.9 Key confirmation versus identity

Proof of possession proves:

```text id="nlyy86"
Presenter controls private key K
```

The token or certificate determines what K represents.

A valid request signature alone does not prove:

```text id="l2ysbw"
K belongs to Alice
```

unless a trusted issuer or registration binds K to Alice.

This distinction appears in:

* WebAuthn
* TLS certificates
* SSH keys
* DPoP-style request proofs
* Signed API requests
* Device identities

---

# 15.10 Explicit key confirmation

Some protocols provide explicit evidence that both peers derived or possess the expected key.

Example:

```text id="rihz1x"
Client and server derive session key
→ each sends authenticated Finished message
→ each confirms the other reached the same handshake state
```

Without key confirmation, a party may know a key was intended for a peer without proving the peer actually possesses it.

---

# 15.11 Replay protection taxonomy

Replay protection can use several mechanisms.

| Mechanism                 | Main idea                                 |
| ------------------------- | ----------------------------------------- |
| Nonce                     | Verifier supplies unpredictable challenge |
| Timestamp                 | Accept only within short time window      |
| Sequence number           | Require monotonic request order           |
| Unique identifier cache   | Reject previously used identifier         |
| Single-use database state | Mark credential consumed atomically       |
| Channel binding           | Restrict credential to one secure channel |
| Key-bound request proof   | Require fresh request signature           |
| Idempotency record        | Prevent duplicate business operation      |

Often several are needed.

---

## Nonce-based replay protection

```mermaid id="uqr7cw"
sequenceDiagram
    participant V as Verifier
    participant P as Prover

    V->>P: Random nonce N
    P->>P: Sign N and context
    P-->>V: Signature
    V->>V: Verify signature and nonce
```

The nonce should be:

* Unpredictable
* Unique enough
* Bound to protocol context
* Used once or tracked appropriately
* Generated by a secure random source

---

## Timestamp-based replay protection

Request:

```json id="bp2c68"
{
  "issued_at": "2026-07-31T12:00:00Z",
  "request_id": "proof-718",
  "signature": "..."
}
```

Server accepts only if:

```text id="f5cne4"
current time - issued_at
is within allowed window
```

Timestamp alone does not prevent repeated use during that window.

Add a unique identifier and replay cache.

---

## Sequence numbers

A session may expect:

```text id="o0orva"
1, 2, 3, 4, ...
```

If request 3 appears again:

```text id="1nhekn"
reject replay
```

Challenges:

* Concurrent requests
* Packet reordering
* Session resumption
* Failover
* Distributed server state
* Counter persistence

---

# 15.12 Replay-cache engineering

A replay cache key may include:

```text id="904cq8"
issuer
subject
credential ID
proof ID
target service
```

Cache retention must cover the acceptance window.

Example:

```text id="ckqupx"
Proof valid for 5 minutes
→ replay record retained at least 5 minutes plus skew
```

---

## Distributed replay cache

If requests are load-balanced:

```text id="35k1ei"
First replay attempt reaches replica A
Second reaches replica B
```

A replica-local cache may fail to detect reuse.

Options:

* Shared strongly consistent store
* Deterministic routing
* Regional replay domain
* Cryptographically unique single-use database state
* Short-lived tokens plus risk acceptance
* Protocol-specific replay design

---

## Atomic consume operation

Single-use token record:

```sql id="1js7qj"
UPDATE reset_tokens
SET used_at = CURRENT_TIMESTAMP
WHERE token_hash = :token_hash
  AND used_at IS NULL
  AND expires_at > CURRENT_TIMESTAMP;
```

Then require:

```text id="u5sdc1"
affected rows = 1
```

Two concurrent requests cannot both successfully consume the same row under appropriate transaction semantics.

---

# 15.13 Canonicalization

Cryptographic signatures operate on bytes.

Applications operate on structured data.

A canonicalization process defines which bytes represent the structured message.

Without a shared canonical representation:

```text id="kihejn"
Signer sees one meaning
Verifier reconstructs another
```

---

## Ambiguous HTTP representation

These may be semantically similar or different depending on server handling:

```text id="8hm1un"
/reports/71
/reports/%37%31
/reports/./71
/reports//71
```

Query parameters may differ by:

* Order
* Duplicate names
* Encoding
* Case
* Empty values
* Plus-versus-space handling

A request-signing scheme must define exactly how to canonicalize them.

---

## Duplicate header ambiguity

Request:

```http id="2dpo4e"
X-Tenant-ID: tenant-42
X-Tenant-ID: tenant-43
```

Different components may:

* Use first
* Use last
* Join with comma
* Reject
* Pass both

If the gateway signs one interpretation and the backend uses another, authorization can be bypassed.

Security-sensitive headers should have unambiguous duplicate-handling rules.

---

# 15.14 Parser differentials

A parser differential occurs when two components interpret the same bytes differently.

Examples:

* Gateway and application disagree on path normalization.
* JWT library and application disagree on duplicate JSON keys.
* XML signature validator and business parser select different nodes.
* Proxy and backend disagree on request length.
* URL parser and policy engine normalize hostnames differently.
* LDAP filter escaping differs across layers.

Attackers exploit the gap between interpretations.

---

## Duplicate JSON keys

Payload:

```json id="qvi670"
{
  "role": "user",
  "role": "admin"
}
```

Possible parser behavior:

* First value wins
* Last value wins
* Reject duplicate
* Preserve both

Signed JSON protocols should specify or enforce duplicate-key behavior.

A secure parser often rejects ambiguous duplicate keys.

---

# 15.15 Signed object versus consumed object

The verifier should return the exact validated object.

Unsafe flow:

```text id="xdbk99"
1. Signature library validates one XML node.
2. Application reparses entire document.
3. Application selects a different node.
```

Safer flow:

```text id="3svv4m"
Signature library validates node
→ returns validated node
→ application consumes only returned node
```

This principle applies beyond XML:

* Signed HTTP requests
* JWT envelopes
* Webhook signatures
* Configuration signatures
* Software-update metadata

---

# 15.16 Confused deputy

A confused deputy is a privileged service tricked into exercising its authority for an unauthorized caller.

Example:

```text id="hnk42z"
Image service can read any cloud object.
User supplies arbitrary object URL.
Image service fetches object using service credential.
```

The image service becomes the deputy.

---

## Authentication is not enough

The image service knows:

```text id="rteh57"
caller = Alice
```

It still must check:

```text id="w48x2q"
May Alice cause the service to read this object?
```

The service’s own cloud authority may be broader than Alice’s authority.

---

## Preventing confused-deputy problems

Use:

* Explicit resource authorization
* Audience-bound delegation
* Tenant-bound resource identifiers
* Narrow service credentials
* Token exchange
* Capability attenuation
* Request-origin validation
* Separate credentials for separate callers
* External IDs or anti-confusion values in cross-account systems

---

# 15.17 Delegation

Delegation means one principal permits another principal to exercise some authority on its behalf.

Example:

```text id="mg0xal"
Alice
delegates
reports.read
to finance-web
```

Delegation should identify:

* Subject
* Actor
* Audience
* Scope
* Resource
* Expiry
* Delegation depth
* Whether further delegation is allowed

---

## Delegation chain

```text id="exd0f3"
Alice
→ finance-web
→ report-export-service
→ object-storage-service
```

Each hop should narrow or preserve authority.

It should not silently expand it.

```mermaid id="9t6fw1"
flowchart LR
    U[Alice]
    F[finance-web]
    E[report-export-service]
    O[object-storage-service]

    U -->|reports.export| F
    F -->|export report 71| E
    E -->|write object export-918| O
```

---

## Delegation monotonicity

A useful invariant is:

```text id="u0nmxl"
Downstream delegated authority
must not exceed
upstream authority
```

Example violation:

```text id="18qljo"
Alice grants report.read
→ intermediary obtains report.admin
```

---

## Delegation depth

A token may restrict:

```text id="myj107"
maximum delegation depth = 1
```

This prevents an intermediary from creating an unlimited chain.

Long chains complicate:

* Audit
* Revocation
* Audience reasoning
* Scope reasoning
* Incident response
* Policy evaluation

---

# 15.18 Impersonation

Impersonation is an explicit administrative action where one actor operates as another subject.

Example:

```text id="si4r3n"
Support agent:
  actor

Customer Alice:
  effective subject
```

Audit context:

```json id="89o9j5"
{
  "actor": "support-agent-17",
  "subject": "u_1048",
  "impersonation": true,
  "reason": "ticket-9182",
  "approved_by": "support-manager-4",
  "started_at": "2026-07-31T12:15:00Z",
  "expires_at": "2026-07-31T12:30:00Z"
}
```

---

## Impersonation requirements

* Explicit entry action
* Strong authentication
* Narrow permission
* User-visible indicator
* Short expiry
* Reason or ticket
* Detailed audit
* Restricted sensitive operations
* Explicit exit
* No silent credential export

---

## Impersonation versus login as user

Bad design:

```text id="ecll9i"
Administrator resets Alice's password
→ logs in as Alice
```

This destroys accountability.

Better:

```text id="o7yo18"
Administrator remains authenticated as self
→ assumes controlled impersonation context
→ actor and subject both preserved
```

---

# 15.19 Capability-based authorization

A capability is an unforgeable reference that directly carries authority.

Conceptual capability:

```text id="lxzcmd"
Capability:
  read report 71
  until 12:30
```

Possession allows the operation.

The capability may be represented as:

* Random opaque token
* Signed structured token
* Object reference
* File descriptor
* Delegated URL
* Cryptographic authorization object

---

## Identity-based versus capability-based access

Identity-based:

```text id="ft559a"
Who are you?
What roles do you have?
Do those roles permit report 71?
```

Capability-based:

```text id="4g3thj"
Do you possess a valid capability authorizing report 71?
```

Capability systems can reduce repeated identity-policy lookup.

They make credential handling and delegation semantics especially important.

---

# 15.20 Capability attenuation

Attenuation creates a less-powerful capability from a more-powerful one.

Original:

```text id="10cqy2"
Read all reports
until 18:00
```

Attenuated:

```text id="k0pj5x"
Read report 71 only
until 12:30
from export-service
```

The holder can reduce authority without obtaining the issuer’s private signing key, depending on capability design.

---

## Attenuation invariant

```text id="yng1xx"
Derived capability
cannot grant more authority
than parent capability
```

Restrictions may include:

* Resource
* Action
* Time
* Network
* Tenant
* Delegation depth
* Maximum amount
* Required third-party approval

---

# 15.21 Caveated credentials

A caveat is a condition attached to a credential.

Example:

```text id="qq3l46"
Allow report download
only if:
  current time < 12:30
  report_id = 71
  caller_service = export-service
```

The verifier evaluates all caveats.

Adding a caveat makes the credential narrower.

---

## First-party caveat

The target service can evaluate it directly.

```text id="5nv1fq"
time < 12:30
```

```text id="bxj0if"
tenant = tenant-42
```

---

## Third-party caveat

Another authority must discharge a condition.

Example:

```text id="esqs4e"
Payment may proceed
only if
fraud-service approves
```

The requester obtains a discharge credential from the fraud service.

The payment service validates both.

---

# 15.22 Capability revocation

Capabilities are often designed for offline verification.

Revocation is therefore difficult.

Approaches:

* Short expiry
* Online revocation check
* Indirection through a revocable reference
* Key rotation
* Capability generation numbers
* Revocation list
* Object version binding
* Root capability invalidation

Tradeoff:

```text id="4aq0hm"
More offline independence
→ harder immediate revocation
```

---

# 15.23 Pre-signed URLs

A pre-signed URL is a common capability.

Example:

```text id="whzbz6"
https://storage.example/object/report-71
  ?expires=...
  &signature=...
```

It may authorize one operation on one resource for a limited time.

---

## Security requirements

* Exact method binding
* Exact resource binding
* Short expiry
* Canonicalized query and headers
* Signature verification
* Limited permissions
* TLS
* Redacted logs
* No open redirects
* Optional content-length or checksum constraints

---

## Leakage risk

A pre-signed URL may leak through:

* Browser history
* Referrer
* Proxy logs
* Chat messages
* Analytics
* Screenshots

Treat it as a bearer credential until expiry.

---

# 15.24 Distributed authorization graphs

Relationship-based systems may store tuples such as:

```text id="x39dbg"
document:71#viewer@user:alice
```

```text id="32me99"
folder:finance#viewer@group:analysts
```

```text id="myw5pv"
group:analysts#member@user:alice
```

Authorization asks:

```text id="9h8j6j"
Does user:alice have viewer relation to document:71?
```

---

## Graph traversal

```mermaid id="zevr7m"
flowchart LR
    A[user:alice]
    G[group:analysts]
    F[folder:finance]
    D[document:71]

    A -->|member| G
    G -->|viewer| F
    D -->|parent| F
```

A policy may infer:

```text id="hgf119"
member of group
+
group viewer of parent folder
+
document inherits viewer from parent
→ Alice can view document
```

---

# 15.25 Tuple semantics

A tuple often has:

```text id="ul2von"
object
relation
subject
```

Example:

```text id="at6zsx"
object:
  document:71

relation:
  viewer

subject:
  user:alice
```

Subjects can also be subject sets:

```text id="mg7pr1"
group:analysts#member
```

Meaning:

```text id="lmr28c"
Every member of group analysts
is related through this edge
```

---

# 15.26 Authorization schema

A schema defines how relations compose.

Conceptual schema:

```text id="j4i816"
document.viewer =
  direct viewer
  OR
  viewer of parent folder
  OR
  member of group assigned as viewer
```

A schema change can alter access for millions of objects without changing individual tuples.

Schema deployment is therefore security-critical.

---

# 15.27 Graph cycles and recursion

Example:

```text id="8ez6k4"
group A includes group B
group B includes group A
```

The evaluator must avoid infinite recursion.

Use:

* Visited nodes
* Bounded depth
* Memoization
* Query budgets
* Cycle-safe fixed-point evaluation

A timeout should not become an allow result.

---

# 15.28 Consistency problem

Suppose:

1. Alice is removed from group `finance-admins`.
2. A permission check immediately follows.
3. The check reaches a replica that has not received the removal.

Result:

```text id="fvg14a"
stale allow
```

Authorization systems need an explicit consistency model.

---

## Consistency choices

### Eventual consistency

Fast and scalable, but recent changes may not appear immediately.

### Read-after-write consistency

The writer can ensure subsequent checks observe its change.

### Snapshot consistency

A group of checks uses one logical database revision.

### Strong consistency

Every check observes the latest committed change according to the system’s ordering guarantees.

Stronger consistency usually costs latency and availability.

---

# 15.29 Revision tokens

A write can return a revision:

```text id="pkqjv9"
revision = 918271
```

A later authorization check can require:

```text id="0j8tgj"
evaluate at revision >= 918271
```

This prevents a stale replica from authorizing based on state older than the security change.

---

## Example

Role removal:

```text id="ggcdiq"
remove alice from finance-admins
→ revision 918271
```

Sensitive check:

```text id="293qmd"
check payment.approve
with minimum revision 918271
```

The evaluator:

* Waits for a sufficiently current replica
* Routes to an authoritative replica
* Rejects if freshness cannot be satisfied

---

# 15.30 Snapshot tokens

A list operation may return:

```text id="t4pmo3"
authorization snapshot = revision 918271
```

Subsequent item checks use the same snapshot.

This avoids inconsistent pagination such as:

```text id="ds5bnd"
Page 1 evaluated before group removal
Page 2 evaluated after group removal
```

Snapshot consistency improves explainability.

---

# 15.31 Authorization filtering

Two broad designs exist.

## Fetch then filter

```text id="j7mq76"
Query many resources
→ call authorization for each
→ discard denied resources
```

Problems:

* High policy-call count
* Timing leakage
* Pagination errors
* Memory use
* Accidental unfiltered response
* Side effects before filtering

---

## Authorization-aware query

```text id="u7crea"
Translate policy constraints
→ query only authorized resources
```

Example:

```sql id="c2mmh7"
SELECT *
FROM reports
WHERE tenant_id = :tenant_id
  AND (
    owner_id = :subject_id
    OR department_id IN (:allowed_departments)
  );
```

Challenges:

* Policy translation correctness
* Dynamic relationships
* Complex conditions
* Database consistency
* Policy versioning

---

# 15.32 Check and list consistency

An authorization system may expose:

```text id="hygd3q"
Check:
  Can Alice view document 71?

List objects:
  Which documents can Alice view?

List subjects:
  Who can view document 71?
```

These operations must share equivalent semantics.

A defect where `Check` denies but `List objects` includes the resource can expose data through search or autocomplete.

---

# 15.33 Authorization result beyond Boolean

A decision engine may return:

```text id="zq0mx0"
allow
deny
indeterminate
not_applicable
```

### Allow

Policy explicitly permits.

### Deny

Policy explicitly rejects.

### Indeterminate

Decision could not be computed.

Examples:

* Missing attribute
* Dependency timeout
* Invalid data type
* Policy error

### Not applicable

A policy does not apply to the request.

The combining layer decides the final outcome.

---

# 15.34 Policy combining

Suppose several policies return:

```text id="ls483i"
Policy A:
  allow

Policy B:
  deny

Policy C:
  not applicable
```

Combining algorithms can include:

* Deny overrides
* Permit overrides
* First applicable
* Only one applicable
* Ordered evaluation

For sensitive systems, explicit deny often overrides allow.

But semantics must be defined, not assumed.

---

## Missing attribute behavior

Policy:

```text id="0sgzwi"
allow if device.managed == true
```

Request lacks `device.managed`.

Possible outcomes:

```text id="z66szf"
false
indeterminate
deny
use default
```

These are not equivalent.

A security schema should define required attributes and failure behavior.

---

# 15.35 Type safety in policy

Unsafe comparison:

```text id="1qva49"
transaction.amount = "10000"
```

Policy expects number:

```text id="evoj9k"
transaction.amount > 5000
```

Implicit conversions may differ across engines.

Authorization input should have:

* Explicit schema
* Validated types
* Bounded values
* Defined missing behavior
* Versioning

---

# 15.36 Partial evaluation

A policy may have static and dynamic components.

Policy:

```text id="s3w758"
allow if:
  subject.role == "analyst"
  AND resource.department == subject.department
  AND request.time within business hours
```

For a known subject, partial evaluation can precompute:

```text id="s3cdu1"
subject.role == analyst
subject.department == finance
```

leaving:

```text id="h8s4rf"
resource.department == finance
AND request.time within business hours
```

This can be translated into a database query or edge filter.

---

## Partial-evaluation risk

If subject state changes:

```text id="5q7k8m"
Alice removed from analyst role
```

the precomputed result becomes stale.

Partial results need:

* Subject version
* Policy version
* Expiry
* Invalidation
* Tenant binding

---

# 15.37 Policy decision provenance

A high-value decision should be explainable through provenance.

Example:

```json id="rwk0sh"
{
  "decision": "allow",
  "policy_version": "reports-18",
  "rules": [
    "tenant-membership",
    "department-read"
  ],
  "inputs": {
    "subject_id": "u_1048",
    "subject_department": "finance",
    "resource_id": "report-71",
    "resource_department": "finance"
  },
  "input_versions": {
    "directory_revision": "81721",
    "resource_revision": "99281"
  }
}
```

Avoid exposing sensitive policy detail to untrusted clients.

Internal provenance improves:

* Debugging
* Audit
* Dispute resolution
* Policy migration
* Incident analysis

---

# 15.38 Trusted computing base

The Trusted Computing Base, or TCB, includes every component whose compromise can violate the security property.

For token authentication, the TCB may include:

* Authorization server
* Signing-key storage
* Token library
* Resource-server validation code
* Issuer configuration
* Time source
* Deployment system
* Reverse proxy where relevant

It is larger than the cryptographic algorithm.

---

## TCB minimization

Strategies:

* Centralize complex protocol validation.
* Use small hardened libraries.
* Isolate signing keys.
* Reduce privileged components.
* Remove direct backend access.
* Use narrow service identities.
* Keep policy language deterministic.
* Separate control plane and data plane.

A smaller TCB is easier to review.

---

# 15.39 Control plane versus data plane

## Control plane

Configures security state.

Examples:

* Create user
* Add role
* Register OAuth client
* Rotate key
* Publish policy
* Create ServiceAccount
* Add trust anchor

## Data plane

Uses configured state to make runtime decisions.

Examples:

* Validate token
* Resolve session
* Check permission
* Accept mTLS connection
* Read resource

```mermaid id="fceg03"
flowchart TD
    C[Security Control Plane]
    P[Published Policy and Trust State]
    D1[Gateway Data Plane]
    D2[API Data Plane]
    D3[Database Enforcement]

    C --> P
    P --> D1
    P --> D2
    P --> D3
```

Control-plane compromise can change future decisions across the entire data plane.

---

# 15.40 Policy distribution

A central policy system may distribute signed policy bundles.

Flow:

```text id="iamqi7"
Policy author
→ review
→ policy compiler
→ signed bundle
→ distribution
→ local evaluator
```

Local evaluators verify:

* Bundle signature
* Version
* Environment
* Validity interval
* Schema compatibility
* Rollback protection

---

## Rollback attack

An attacker replaces policy version 18 with older version 12 where access was broader.

Defenses:

* Monotonic version
* Signed metadata
* Minimum accepted version
* Deployment ledger
* Expiry
* Control-plane audit

Cryptographic signature alone does not prove freshness.

A correctly signed old policy may still be unsafe.

---

# 15.41 Trust agility and trust-store changes

Adding a trust anchor expands who can authenticate.

Removing one contracts trust.

Trust-store changes should be treated like authorization changes.

Example:

```text id="u8fq8h"
Add partner CA
→ every certificate issued under partner CA may become potentially acceptable
```

Application policy must still constrain:

* Expected identities
* Trust domain
* Certificate purpose
* Audience
* Allowed partner resources

---

# 15.42 Multi-issuer systems

A service may trust multiple issuers.

Example:

```text id="0hx8pm"
Corporate IdP
Partner IdP
Customer identity platform
Automation issuer
```

Identity must be namespaced by issuer.

Unsafe:

```text id="1p4ua5"
sub = 731
```

Safe conceptual key:

```text id="6rb31a"
issuer = partner-a
subject = 731
```

---

## Issuer-specific policy

A claim should not have universal meaning.

Example:

```text id="6l8atq"
role=admin
```

from customer issuer must not automatically map to internal administrator.

Policy:

```text id="hhvisn"
Corporate IdP admin claim:
  may map to internal administrator

Customer IdP admin claim:
  tenant-local customer administrator only
```

---

# 15.43 Identity collision

Two issuers may assert:

```text id="zs8y3j"
email = alice@example.com
```

They may represent:

* Same human
* Different accounts
* Reassigned address
* Malicious issuer assertion
* Test versus production identity

Account linking requires explicit assurance.

Do not merge automatically based only on a mutable shared attribute.

---

# 15.44 Account linking

A user may link:

```text id="0fu47e"
Corporate identity
Personal passkey
Partner identity
Recovery email
```

The account-link action itself is security-sensitive.

Safe linking may require:

1. Existing account is strongly authenticated.
2. New identity is independently authenticated.
3. User explicitly confirms linking.
4. Conflicting existing mappings are rejected.
5. Audit and notification occur.
6. High-risk identities require administrator approval.

---

## Account-linking attack

Attacker controls weak identity `E`.

They convince system to link `E` to Alice’s account.

Later:

```text id="7h4ob6"
Attacker authenticates as E
→ gains Alice's account
```

The security of the account becomes the security of its weakest linked identity unless policy differentiates them.

---

# 15.45 Assurance-aware account linking

Different identities may grant different capabilities.

Example:

```text id="i98c1n"
Corporate passkey:
  full account access

Recovery email:
  recovery initiation only

Partner identity:
  partner-tenant resources only
```

Linking need not mean every credential is equivalent.

---

# 15.46 Identity lifecycle

An identity moves through lifecycle states:

```mermaid id="0px4ly"
stateDiagram-v2
    [*] --> Requested
    Requested --> Provisioned
    Provisioned --> Active
    Active --> Suspended
    Suspended --> Active
    Active --> Disabled
    Suspended --> Disabled
    Disabled --> Deprovisioned
    Deprovisioned --> [*]
```

Possible events:

* Hire
* Contractor onboarding
* Role change
* Department transfer
* Leave
* Termination
* Rehire
* Legal hold
* Account merger
* Tenant migration

---

# 15.47 Provisioning versus authentication

Provisioning creates or updates account state.

Authentication proves control of a credential.

```text id="x42knh"
Provisioned account
does not mean
currently authenticated
```

```text id="uanh5b"
Valid external authentication
does not mean
local account is provisioned or active
```

---

# 15.48 Just-in-time provisioning

On first federated login:

```text id="r7efls"
External identity validates
→ local account absent
→ create local account
```

JIT provisioning should define:

* Trusted issuer
* Stable identity key
* Tenant mapping
* Default role
* Required attributes
* Duplicate handling
* Deprovisioning process
* Attribute-update policy

---

## JIT privilege risk

Dangerous mapping:

```text id="9r6tdt"
external group contains "admin"
→ create local global administrator
```

Safer:

```text id="lzdcwm"
issuer-specific external group
→ explicitly mapped local tenant role
→ administrator approval for global privilege
```

---

# 15.49 Automated provisioning protocols

Directory provisioning systems may create, update, suspend, and delete accounts through standardized or proprietary APIs.

A typical flow:

```text id="u811zn"
HR system
→ identity governance
→ application provisioning API
→ local account
```

Provisioning credentials are highly privileged.

They may be able to:

* Create users
* Set attributes
* Assign groups
* Disable accounts
* Delete accounts

---

## Provisioning authorization

The provisioning client should be constrained by:

* Tenant
* Object type
* Allowed attributes
* Role-assignment limits
* Source network
* Client identity
* Change approval
* Rate limits

Do not give a provisioning integration unrestricted application administration.

---

# 15.50 Deprovisioning

Deprovisioning must address more than account deletion.

Possible residual authority:

* Active sessions
* Refresh tokens
* API keys
* SSH keys
* Service credentials
* Group membership
* Shared resources
* Delegated access
* Scheduled jobs
* Personal access tokens
* Recovery methods
* Cached authorization
* Offline credentials

A complete termination flow should enumerate every credential class.

---

# 15.51 Disable versus delete

## Disable

* Prevent new authentication
* Preserve audit and ownership history
* Allow investigation
* Retain record references

## Delete

* Remove account data according to retention policy
* May break historical ownership references
* May allow identifier reuse
* May complicate audit

A common sequence is:

```text id="pd5x6d"
disable immediately
→ revoke credentials
→ transfer ownership
→ retain for policy period
→ delete or anonymize later
```

---

# 15.52 Identifier reuse

Suppose username `alice` is deleted and later reused.

Old records may refer to:

```text id="duzsb5"
owner = alice
```

The new Alice may inherit old authority.

Use stable non-reused internal identifiers:

```text id="njtp0u"
user_id = u_1048
```

Display names and login names can change independently.

---

# 15.53 Group lifecycle

Groups can also be:

* Created
* Nested
* Renamed
* Repurposed
* Deleted
* Recreated

If authorization stores group names rather than stable IDs, recreating a group with the same name may restore old permissions unexpectedly.

---

# 15.54 Role mining and privilege drift

Over time, users accumulate roles.

Example:

```text id="eicg9u"
Alice joins finance
→ gets finance-reader

Alice becomes manager
→ gets finance-approver

Alice transfers to legal
→ gets legal-reader

Old finance roles remain
```

This is privilege drift.

Controls:

* Birthright roles
* Time-bound assignments
* Periodic access review
* Manager attestation
* Automatic removal on role change
* Separation-of-duties analysis
* Usage-based review

---

# 15.55 Access review

An access review asks:

```text id="k96s8q"
Who has this permission?
Why do they have it?
When was it granted?
When was it last used?
Should it remain?
```

Review evidence should include:

* Direct assignments
* Inherited roles
* Group paths
* Delegation
* Exceptions
* Expiry
* Owner
* Approval history

A flat list of effective permissions may be insufficient without provenance.

---

# 15.56 Permission provenance

Example:

```text id="4tetpr"
Alice can approve payments because:
  Alice is member of finance-managers
  finance-managers has role payment-approver
  payment-approver grants payment.approve
```

If access is inappropriate, provenance shows which edge to remove.

---

# 15.57 Dormant credentials

A credential that has not been used for a long time may still be valid.

Examples:

* API key
* SSH key
* Personal access token
* Service password
* Recovery code
* Old mobile session

Dormant-credential policy may:

* Alert
* Require reauthentication
* Disable
* Rotate
* Delete after review

Usage data must be interpreted carefully because some legitimate emergency credentials are rarely used.

---

# 15.58 Credential inventory

A security inventory should answer:

```text id="q9a21n"
Which credentials exist?
Who or what owns them?
Where are they stored?
What authority do they carry?
When do they expire?
When were they last used?
How are they revoked?
```

Without inventory, rotation and incident response are incomplete.

---

# 15.59 Recovery as an alternate authentication protocol

Password recovery is not merely customer support.

It is an authentication mechanism.

Flow:

```text id="wduhid"
Claim account
→ prove control of recovery channel
→ establish new credential
→ regain account
```

The recovery path must provide assurance appropriate to the account.

---

## Recovery weaknesses

* Recycled phone number
* Compromised email
* Social engineering
* Weak support questions
* Public personal data
* Insider override
* Recovery link leakage
* Long-lived reset token
* No notification
* Existing sessions left active

The strongest login method is ineffective if recovery is much weaker.

---

# 15.60 Recovery-token design

A reset token should be:

* High entropy
* Short-lived
* Single-use
* Bound to one account and purpose
* Stored hashed where practical
* Invalidated after use
* Invalidated when superseded
* Protected from logs
* Followed by session review or revocation

Conceptual record:

```json id="33mja6"
{
  "token_hash": "sha256:...",
  "user_id": "u_1048",
  "purpose": "password_reset",
  "created_at": "2026-07-31T12:00:00Z",
  "expires_at": "2026-07-31T12:15:00Z",
  "used_at": null
}
```

---

# 15.61 Recovery race

Two requests present the same reset token.

The consume operation must be atomic.

Unsafe:

```text id="ky5s0j"
Check used_at is null
→ request A and B both see null
→ both reset password
```

Safe:

```text id="5ahx1k"
Atomic conditional update
→ exactly one request succeeds
```

---

# 15.62 MFA enrollment security

Adding a new factor increases future authentication authority.

Enrollment should require:

* Strong existing authentication
* Recent authentication
* User confirmation
* Notification
* Recovery-code regeneration policy
* Duplicate-factor handling
* Device naming
* Administrative audit where appropriate

An attacker with a stolen session should not be able to silently enroll their own factor.

---

# 15.63 MFA removal

Removing a factor may be riskier than adding one.

Safe controls may include:

* Step-up with another factor
* Recovery process
* Delay
* User notification
* Session revocation
* Support approval
* Risk review

Do not allow a session authenticated only by the factor being removed to remove that factor without additional assurance.

---

# 15.64 Passkey and WebAuthn internals

A passkey or WebAuthn credential generally involves:

```text id="uwf6n2"
Relying party identity
User account
Credential ID
Public key
Authenticator
Private key
Signature counter or other authenticator data
```

The server stores a public key.

The authenticator protects the private key.

---

## Registration flow

```mermaid id="hqj42u"
sequenceDiagram
    actor User
    participant Browser
    participant RP as Relying Party
    participant Auth as Authenticator

    User->>Browser: Begin passkey registration
    Browser->>RP: Request registration options
    RP-->>Browser: Challenge + RP information + user information
    Browser->>Auth: Create credential
    Auth->>Auth: Generate key pair
    Auth-->>Browser: Public key + attestation/authenticator data
    Browser->>RP: Registration response
    RP->>RP: Validate challenge, origin, RP binding, and data
    RP->>RP: Store credential ID and public key
```

---

## Authentication flow

```mermaid id="gsu05u"
sequenceDiagram
    actor User
    participant Browser
    participant RP as Relying Party
    participant Auth as Authenticator

    User->>Browser: Begin sign-in
    Browser->>RP: Request authentication options
    RP-->>Browser: Random challenge + RP ID
    Browser->>Auth: Request assertion
    Auth->>Auth: Verify user presence or user verification
    Auth-->>Browser: Signed assertion
    Browser->>RP: Credential ID + signature + authenticator data
    RP->>RP: Validate challenge, origin, RP ID, signature, and policy
```

---

# 15.65 WebAuthn binding properties

The assertion is bound to:

* Relying-party identifier
* Browser origin
* Server-generated challenge
* Credential private key
* Authenticator state
* User presence or verification flags
* Authentication ceremony context

This resists ordinary credential phishing because a fake origin does not satisfy the legitimate relying-party binding.

---

## User presence versus user verification

### User presence

The user interacted with the authenticator.

Example:

```text id="a6lj8d"
touch security key
```

### User verification

The authenticator verified the user through:

* PIN
* Biometric
* Device authentication
* Another authenticator-local mechanism

Policy must decide which is required.

---

# 15.66 Discoverable and non-discoverable credentials

A discoverable credential can be found by the authenticator for the relying party without the server first naming a credential ID.

This enables username-less experiences.

A non-discoverable credential generally requires the server or client to provide acceptable credential IDs.

The relying party must still map the authenticated credential to the correct local account.

---

# 15.67 Synced passkeys

A passkey may be synchronized across a user’s trusted devices through a credential provider.

This changes the threat model:

```text id="ux1lw0"
Credential is phishing-resistant
but
account recovery and synchronization ecosystem become part of credential security
```

Risk analysis should include:

* Device account security
* Cloud account recovery
* Device unlock
* Provider compromise
* Cross-device enrollment
* Enterprise policy

---

# 15.68 Attestation

Authenticator attestation may provide information about how a credential was created.

Potential uses:

* Require approved hardware
* Enforce enterprise authenticator policy
* Identify security-key models
* Assess key protection

Costs:

* Privacy impact
* Metadata management
* Vendor dependence
* Certificate-chain validation
* Model lifecycle
* Attestation-format complexity

Many consumer applications do not need strict attestation policy.

---

# 15.69 Signature counter limitations

Some authenticators provide counters intended to help identify cloned credentials.

A decreasing or duplicated counter may indicate:

* Credential clone
* Authenticator reset
* Synced credential behavior
* Device migration
* Implementation-specific semantics

Do not treat every counter anomaly as definitive compromise without understanding authenticator behavior.

---

# 15.70 WebAuthn challenge state

The server must bind the challenge to:

* Browser transaction
* Intended account or discoverable flow
* Relying-party action
* Expiry
* Expected user-verification policy

A challenge should be:

* Random
* Short-lived
* Single-use
* Stored or protected against tampering

---

# 15.71 Origin and relying-party validation

A valid signature from a registered public key is not sufficient if the ceremony occurred for the wrong origin or relying party.

The server must validate the browser-provided client data and authenticator data according to the expected registration or authentication ceremony.

This is another example of:

```text id="4j2mpd"
signature validity
≠
complete protocol validity
```

---

# 15.72 Device-bound sessions

A session may be tied to a device key.

Flow:

```text id="s23n8y"
Browser or device generates key
→ session record binds public key
→ each sensitive request includes signed proof
```

Benefits:

* Copied cookie alone may be insufficient.
* Device continuity can be established.
* High-risk actions can require key proof.

Challenges:

* Key backup
* Browser support
* Device replacement
* Privacy
* Multi-device sessions
* XSS controlling legitimate signing APIs
* Recovery

---

# 15.73 Trusted execution and hardware-backed identity

A private key may be protected by:

* TPM
* Secure enclave
* Hardware security module
* Smart card
* Security key

The key may be non-exportable.

The device provides signing operations rather than raw key access.

---

## What hardware protection does not prove

Hardware-backed key possession does not automatically prove:

* Application is uncompromised.
* User intended this exact transaction.
* Display is trustworthy.
* Server authorization is correct.
* Recovery account is secure.
* Malware cannot invoke the signing API.

Transaction confirmation may need secure display or explicit user interaction.

---

# 15.74 Transaction signing

A stronger authentication ceremony can bind user approval to transaction data.

Example:

```text id="z21dyk"
Pay:
  ₹42,000

To:
  Merchant 781

Currency:
  INR

Nonce:
  91ab...
```

The authenticator signs the transaction representation.

The server verifies that the signed details match the operation executed.

---

## Transaction-signing pitfalls

* Ambiguous data encoding
* Truncated display
* Currency confusion
* Invisible fields
* Server changes data after signing
* Replay of signed transaction
* User signs opaque hash without understanding
* Incorrect locale or decimal handling

The human-visible representation and signed bytes must correspond.

---

# 15.75 Continuous authentication

Continuous authentication reevaluates risk during a session.

Signals may include:

* Device health
* Network change
* Behavioral anomaly
* Credential compromise event
* Session location
* Browser integrity
* User interaction
* Sensitive action pattern

Possible outcomes:

```text id="mxz7jg"
continue
restrict
require step-up
terminate session
```

---

## Continuous authorization

Authorization may also be reevaluated during long-lived operations.

Examples:

* Streaming data session
* Database connection
* gRPC channel
* Remote administrative session
* Long-running export
* Message subscription

A decision at connection start may become stale.

---

# 15.76 Long-lived stream authorization

Suppose Alice subscribes to finance events.

At 12:00:

```text id="qe0xrk"
Alice is finance analyst
→ stream allowed
```

At 12:10:

```text id="1xe9fz"
Alice removed from finance
```

If the stream remains open indefinitely, access persists.

Controls:

* Maximum stream lifetime
* Periodic reauthorization
* Revocation event
* Connection termination
* Per-message authorization
* Short-lived subscription credential

---

# 15.77 Time-of-check to time-of-use

TOCTOU occurs when authorization is checked and the protected operation happens later against changed state.

Example:

```text id="y0wtjb"
1. Check Alice owns report 71.
2. Ownership changes.
3. Delete report 71 based on old check.
```

---

## Atomic authorization

Where possible, combine check and operation.

```sql id="s08a40"
DELETE FROM reports
WHERE report_id = :report_id
  AND owner_id = :subject_id;
```

Then require:

```text id="95l59h"
affected rows = 1
```

This makes resource state and authorization condition part of one database operation.

---

## Version-bound operation

Read resource revision:

```text id="c6h0rs"
revision = 18
```

Authorize based on revision 18.

Write:

```sql id="b7cjg7"
UPDATE reports
SET status = 'archived',
    revision = revision + 1
WHERE report_id = :report_id
  AND revision = 18
  AND owner_id = :subject_id;
```

If state changed, zero rows are updated and the operation is retried or rejected.

---

# 15.78 Distributed TOCTOU

Authorization service checks data at revision A.

Resource service changes data at revision B.

The final operation must ensure the authorized assumptions remain true.

Options:

* Include resource version in authorization request.
* Return authorization decision bound to resource version.
* Recheck inside resource transaction.
* Use shared database transaction where possible.
* Use short decision lifetime.
* Reject when resource changed.

---

# 15.79 Decision tokens

An authorization service may return a signed decision token.

Conceptual token:

```json id="3rfkzd"
{
  "sub": "u_1048",
  "act": "finance-web",
  "action": "report.download",
  "resource": "report-71",
  "resource_version": 18,
  "decision": "allow",
  "policy_version": "reports-18",
  "exp": 1785499800
}
```

The resource service verifies:

* Decision issuer
* Signature
* Audience
* Action
* Resource
* Resource version
* Subject and actor
* Expiry
* One-time or replay policy

---

## Decision-token tradeoff

Benefits:

* Reduced policy-service calls
* Auditable decision artifact
* Explicit policy version
* Offline enforcement

Risks:

* Stale authorization
* Replay
* Resource-version mismatch
* Token leakage
* Revocation delay
* Complex audience handling

---

# 15.80 Formal reasoning about authorization

A policy can be modeled as a function:

```text id="w2ahvb"
Decision =
  P(subject, actor, action, resource, context, policy_state)
```

For deterministic evaluation:

```text id="gny8jc"
same complete inputs
+
same policy version
→ same decision
```

If decisions differ, some hidden input exists:

* Time
* Network lookup
* Randomness
* Mutable global state
* Cache state
* External policy dependency

Hidden inputs reduce reproducibility.

---

# 15.81 Monotonic and non-monotonic policy

## Monotonic authorization

Adding facts can only add permissions.

Example:

```text id="z5gx1s"
Alice is a viewer
```

Adding another unrelated group does not remove viewer permission.

## Non-monotonic authorization

Adding facts can remove permissions.

Example:

```text id="ivw3du"
Allow if user is employee
unless user is suspended
```

Adding:

```text id="qtf3sx"
suspended = true
```

changes allow to deny.

Explicit denies and risk rules create non-monotonic behavior.

These systems require careful cache invalidation and evaluation ordering.

---

# 15.82 Safety properties and liveness properties

## Safety property

Something bad never happens.

Example:

```text id="1o49qg"
No tenant can read another tenant's report.
```

## Liveness property

Something good eventually happens.

Example:

```text id="vc11te"
A newly provisioned employee can eventually sign in.
```

Authentication engineering needs both.

An always-deny system may satisfy some safety properties but fail liveness completely.

---

# 15.83 Formal protocol verification

Security protocols can be modeled to analyze properties such as:

* Secrecy
* Authentication
* Freshness
* Replay resistance
* Key agreement
* Forward secrecy
* Unknown-key-share resistance
* Identity binding
* Downgrade resistance

Tools and methods may include:

* Symbolic protocol models
* State-machine analysis
* Model checking
* Theorem proving
* Protocol fuzzing
* Differential testing

Formal verification proves properties of a model.

It does not prove:

* Implementation has no bug.
* Deployment matches model.
* Random number generator is sound.
* Keys are protected.
* Authorization policy is correct.
* User interface prevents deception.

---

# 15.84 Unknown key-share attack

In an unknown key-share scenario, two parties establish the same key but disagree about who the peer is.

Conceptually:

```text id="ohx90z"
Alice believes key is shared with Bob.
Bob believes key is shared with Mallory.
```

Protocol messages must bind identities to key establishment.

This is why peer identities, audience, server names, and transcript data matter.

---

# 15.85 Downgrade attacks

A downgrade attack causes peers to use a weaker mechanism.

Examples:

* Strong authentication falls back to password.
* TLS version falls back.
* PKCE `S256` falls back to weaker behavior.
* MFA requirement is omitted.
* Signed SAML request becomes unsigned.
* Channel binding becomes optional.
* Secure cookie is replaced by insecure session mode.

Defenses:

* Explicit minimum policy
* Downgrade-detection signals
* No silent fallback
* Version binding
* Secure defaults
* Audit

---

# 15.86 Algorithm confusion

Algorithm confusion occurs when verifier behavior is influenced unsafely by untrusted algorithm declarations.

Unsafe flow:

```text id="pc0s5n"
Token says alg=X
→ verifier automatically uses X
```

Safe flow:

```text id="jll4sv"
Application policy allows algorithms {A, B}
→ token algorithm must be one of them
→ key type and algorithm must match
```

The token does not define the verifier’s trust policy.

---

# 15.87 Cross-protocol key reuse

Using one key across protocols can enable confusion.

Example:

```text id="mr3mtn"
Same key used for:
  JWT signing
  SAML signing
  arbitrary webhook signing
```

An object valid in one protocol may be transformed or misinterpreted in another.

Use domain separation:

* Different keys
* Different algorithms where appropriate
* Different signed-message prefixes
* Different audiences and types
* Explicit token type

---

# 15.88 Domain separation

Domain separation ensures the same cryptographic operation has distinct meaning in different contexts.

Instead of signing:

```text id="y4hrkz"
message bytes
```

sign:

```text id="zqqpjh"
"PAYMENT_APPROVAL_V1" || message bytes
```

Another protocol uses:

```text id="hvfl0o"
"SESSION_TOKEN_V1" || message bytes
```

A signature from one domain cannot be interpreted safely as another without matching prefix and format.

---

# 15.89 Forward secrecy

Forward secrecy means compromise of a long-term key does not reveal past session keys for previously recorded traffic, assuming ephemeral key agreement and no endpoint compromise at the time.

It protects past transport confidentiality.

It does not retroactively invalidate:

* Signed tokens
* Recorded application data
* Stolen session cookies
* Database records
* Long-term credentials

---

# 15.90 Post-compromise security

Post-compromise security asks whether a system can recover future security after a temporary compromise.

Examples:

* Rotate session keys
* Reauthenticate
* Replace device keys
* Advance ratcheting keys
* Revoke compromised credentials
* Rebuild workload identity
* Reissue trust material

Some protocols provide stronger automatic recovery properties than ordinary static credential systems.

---

# 15.91 Identity privacy

Authentication can expose:

* Stable identifiers
* Device identifiers
* Group membership
* Employer
* Authentication methods
* Login timing
* Service usage

Privacy-preserving designs may use:

* Pairwise identifiers
* Minimal claims
* Selective disclosure
* Pseudonyms
* Anonymous credentials
* Short-lived identifiers
* Local authorization proofs

---

# 15.92 Selective disclosure

A relying service may need to know:

```text id="92s28l"
User is over required age
```

without learning:

```text id="hy7pkj"
Exact date of birth
```

Selective-disclosure systems can prove attributes or predicates while minimizing unrelated data.

Operational challenges include:

* Revocation
* Issuer trust
* Wallet security
* Correlation
* Device recovery
* Verifier compatibility
* Policy semantics

---

# 15.93 Anonymous credentials

An anonymous credential may allow a holder to prove:

```text id="osq702"
Member of approved organization
```

without revealing one stable global identity.

Some designs support:

* Unlinkable presentations
* Selective disclosure
* Predicate proofs
* Limited-use credentials

They do not remove authorization policy.

They change which identity information is available to policy.

---

# 15.94 Zero-knowledge proofs

A zero-knowledge proof allows one party to prove a statement without revealing the underlying secret beyond what the statement implies.

Conceptual example:

```text id="awivkc"
Prove:
  know a credential signed by trusted issuer
  and age attribute satisfies threshold

Reveal:
  threshold result

Do not reveal:
  full credential or date of birth
```

Practical deployment requires careful protocol and implementation selection.

Cryptographic proof correctness does not solve:

* Device theft
* Issuer compromise
* Recovery
* User consent
* Policy misuse
* Metadata leakage

---

# 15.95 Revocation in privacy-preserving systems

Revocation can undermine privacy if every verification contacts the issuer with a stable credential identifier.

Alternatives may include:

* Accumulators
* Short-lived credentials
* Privacy-preserving status proofs
* Epoch-based validity
* Downloadable revocation data

Each design trades:

* Privacy
* verifier complexity
* credential size
* freshness
* availability

---

# 15.96 Authorization as dataflow control

Traditional authorization controls whether an operation starts.

Information-flow control asks:

```text id="m22akx"
Where may data flow after access?
```

Example:

```text id="vgndiu"
Alice may read confidential report
but
report must not be sent to public analytics service
```

This introduces labels such as:

```text id="6hrmbb"
public
internal
confidential
restricted
```

---

## Information-flow rule

Conceptual rule:

```text id="q8e36f"
Data may flow
from source label L1
to destination label L2
only if policy permits L1 → L2
```

Access control and information-flow control complement each other.

---

# 15.97 Taint propagation

A system may mark data originating from:

```text id="0pfc9m"
tenant-42 confidential source
```

Derived output retains or combines labels.

Challenges:

* Data aggregation
* Label explosion
* Declassification
* Performance
* External integrations
* Human-created copies
* Database and application coordination

---

# 15.98 Declassification

Declassification intentionally lowers data sensitivity.

Example:

```text id="nhuc19"
Raw customer transactions:
  restricted

Approved aggregate statistics:
  public
```

Declassification should require:

* Authorized transformation
* Defined privacy threshold
* Review
* Audit
* Protection against reconstruction

---

# 15.99 Non-interference concept

A strong confidentiality property says:

```text id="4u48rk"
Changes to secret data
should not affect
what an unauthorized observer can see
```

Real systems often leak through:

* Timing
* Error messages
* Resource existence
* Response size
* Cache behavior
* Search counts
* Rate limits

Perfect non-interference is difficult.

The concept helps identify side channels.

---

# 15.100 Authorization and business invariants

Some authorization rules cannot be expressed as simple role checks.

Examples:

```text id="g02x5d"
A payment approver cannot approve a payment they created.
```

```text id="hh8mgi"
A refund may not exceed the settled payment.
```

```text id="5ln8ta"
A user may export no more than 10,000 records without secondary approval.
```

```text id="a2i6n1"
Two distinct administrators must approve root-key rotation.
```

These are business invariants enforced across state transitions.

---

# 15.101 Workflow authorization

A workflow can be modeled as:

```mermaid id="f3qcfo"
stateDiagram-v2
    [*] --> Draft
    Draft --> Submitted: creator submits
    Submitted --> Approved: independent approver approves
    Submitted --> Rejected: approver rejects
    Approved --> Executed: payment service executes
    Rejected --> [*]
    Executed --> [*]
```

Authorization depends on:

* Current state
* Actor
* Previous actors
* Amount
* Tenant
* Time
* Approval count

---

# 15.102 Separation-of-duty state check

Unsafe:

```text id="4k7lk5"
User has approver role
→ allow approval
```

Safer:

```text id="ya54pa"
User has approver role
AND
user did not create payment
AND
payment is submitted
AND
no conflicting approval exists
```

Evaluate and update atomically.

---

# 15.103 Authorization inside transactions

Pseudocode:

```sql id="gcd47z"
UPDATE payments
SET status = 'approved',
    approved_by = :actor_id,
    approved_at = CURRENT_TIMESTAMP
WHERE payment_id = :payment_id
  AND status = 'submitted'
  AND created_by <> :actor_id;
```

Then require one affected row.

Role authorization still occurs before or alongside this query.

The database condition enforces state-dependent invariants.

---

# 15.104 Event-driven authorization

In asynchronous systems, authorization may occur:

* When command is accepted
* When event is published
* When event is consumed
* When side effect is executed

A delayed worker must decide whether to use:

```text id="qv978r"
Authority at submission time
```

or:

```text id="xy2y3v"
Authority at execution time
```

---

## Submission-time authority

Example:

```text id="fjrlvw"
Alice was allowed to export report at 12:00.
Job executes at 12:10 after Alice loses role.
```

Should export continue?

Possible policy:

```text id="6v1y83"
Accepted authorization remains valid for this approved job
```

or:

```text id="h09m7c"
Worker must reauthorize at execution time
```

The decision depends on business semantics.

---

## Authorization envelope for jobs

A queued job may include:

```json id="24s1ce"
{
  "job_id": "job-9182",
  "actor": "finance-web",
  "subject": "u_1048",
  "action": "report.export",
  "resource": "report-71",
  "authorized_at": "2026-07-31T12:00:00Z",
  "policy_version": "reports-18",
  "authorization_reference": "decision-771",
  "expires_at": "2026-07-31T12:15:00Z"
}
```

The worker validates the envelope and applies execution-time policy.

---

# 15.105 Exactly-once authorization is difficult

Distributed systems rarely provide literal exactly-once execution across all components.

A job can be:

* Delivered twice
* Retried after timeout
* Executed but acknowledgment lost
* Partially completed
* Replayed by an attacker

Use:

* Idempotency keys
* Deduplication
* Transactional outbox
* Unique constraints
* State machines
* Replay protection

Authorization should be checked consistently across retries.

---

# 15.106 Transactional outbox

Business transaction:

```text id="67i2zc"
Approve payment
+
publish payment-approved event
```

Unsafe:

```text id="14mbgz"
Database commits approval
event publish fails
```

or:

```text id="a8pt9o"
Event publishes
database rollback occurs
```

Transactional outbox:

1. Update business state.
2. Insert event record in same transaction.
3. Commit.
4. Publisher later sends outbox event.
5. Mark event delivered.

This preserves authorization-related state transition and event consistency.

---

# 15.107 Auditing asynchronous delegation

Audit chain:

```text id="h827bu"
Alice requested export
finance-web authorized request
export-worker executed job
storage-service wrote object
notification-service sent link
```

Record:

* Original subject
* Each actor
* Authorization time
* Execution time
* Policy version
* Resource
* Output
* Credential or decision references

---

# 15.108 Policy as code

Policy as code places authorization rules in version-controlled, testable artifacts.

Benefits:

* Review
* Version history
* Automated testing
* Reproducible deployment
* Shadow evaluation
* Rollback

Risks:

* Policy language complexity
* Hidden data dependencies
* Unsafe defaults
* Schema drift
* Overly broad reusable rules
* Review fatigue

---

# 15.109 Policy schema evolution

Version 1 input:

```json id="92jizy"
{
  "user_role": "analyst"
}
```

Version 2 input:

```json id="9cwfsr"
{
  "subject": {
    "roles": ["analyst"]
  }
}
```

During migration, some callers may send old input.

The evaluator must:

* Reject unknown schema
* Support explicit versions
* Adapt through a trusted compatibility layer
* Avoid silently treating missing fields as allow

---

# 15.110 Policy linting

Static analysis can detect:

* Unreachable rules
* Always-allow conditions
* Contradictory rules
* Missing default
* Unbounded recursion
* Unknown attributes
* Type mismatches
* Privilege escalation paths
* Shadowed deny rules

Linting complements tests.

It does not prove business correctness.

---

# 15.111 Policy differential testing

Run two policy versions over the same recorded input set.

Output:

```text id="1i5bwi"
Version 17:
  allow

Version 18:
  deny
```

Classify differences:

* Intended restriction
* Intended grant
* Missing attribute
* Regression
* Data-quality issue

Use sanitized or synthetic input where production identity data is sensitive.

---

# 15.112 Formal authorization properties

Examples:

## Tenant isolation

```text id="ns9oex"
For all users U and resources R:
if tenant(U) != tenant(R)
and U lacks explicit cross-tenant permission
then decision(U, read, R) = deny
```

## Separation of duty

```text id="rme8f3"
For every payment P:
creator(P) cannot approve P
```

## Delegation attenuation

```text id="8bb6d8"
Authority(child_token)
is a subset of
Authority(parent_token)
```

## Revocation freshness

```text id="a093y4"
After disablement revision V:
no critical decision at revision >= V may allow subject
```

These properties can drive automated testing.

---

# 15.113 Security proofs versus operational evidence

A formal proof may establish:

```text id="wh0jpo"
Protocol resists replay under stated assumptions.
```

Operations must still prove:

* Nonces are generated securely.
* Replay cache is shared correctly.
* Clocks are synchronized.
* Keys are not leaked.
* Proxy does not strip context.
* Libraries enforce the modeled checks.
* Policy configuration is correct.

Security requires both design reasoning and runtime evidence.

---

# 15.114 Advanced troubleshooting: valid signature, invalid context

A signed request may fail because:

* Wrong audience
* Wrong method
* Wrong URL
* Wrong body hash
* Proof expired
* Proof replayed
* Proxy changed path
* Host canonicalization differs
* Key does not match token confirmation
* Signature uses disallowed algorithm

Debug every binding independently.

---

# 15.115 Advanced troubleshooting: stale authorization after revocation

Trace:

```text id="0jbbqu"
Source directory revision
→ provisioning event
→ identity provider claim cache
→ application session
→ token
→ policy attribute cache
→ decision cache
→ long-lived connection
```

The slowest layer defines practical revocation time.

Measure each propagation step.

---

# 15.116 Advanced troubleshooting: permission differs by API path

Possible explanation:

```text id="cj786s"
/reports/71:
  object-level policy

/export?report=71:
  role-only policy
```

or:

```text id="erjg06"
REST endpoint:
  gateway enforcement

GraphQL endpoint:
  direct backend resolver
```

Create an enforcement inventory covering:

* REST
* GraphQL
* WebSocket
* gRPC
* Batch
* Export
* Admin
* Background jobs
* Internal service routes

---

# 15.117 Advanced troubleshooting: policy says deny but action occurs

Possible causes:

* Decision not checked
* Stale cached allow
* Result inverted
* Error converted to allow
* Side effect occurs before policy call
* Another route bypasses enforcement
* Retry executes without rechecking
* Database procedure performs broader action
* Message consumer trusts forged authorization envelope

Trace the actual side effect and request path.

---

# 15.118 Advanced troubleshooting: proof-of-possession fails behind proxy

Potential mismatch:

```text id="ji8gcs"
Client signs:
  https://api.example/reports

Backend sees:
  http://reports-api:8080/reports
```

The proxy changed:

* Scheme
* Host
* Port
* Path
* Encoding

The architecture needs one canonical external request representation or gateway-side proof validation.

Do not let untrusted forwarding headers define the signed target.

---

# 15.119 Advanced troubleshooting: intermittent replay errors

Possible causes:

* Client reuses proof identifier
* Retry reuses signed request
* Two workers share one credential and nonce generator
* Clock jumps backward
* Replay cache replicates inconsistently
* Load balancer duplicates request
* Message broker redelivers
* Request ID generation is not unique
* Virtual-machine snapshot cloned state

Record:

* Proof ID
* Credential identity
* Timestamp
* Replica
* Request method and target hash
* Retry attempt
* Cache region

---

# 15.120 Advanced troubleshooting: identity collision after migration

Symptoms:

```text id="csu83a"
Two external accounts map to one local user
```

Possible causes:

* Email-based linking
* Realm stripped
* Issuer omitted
* Case normalization changed
* Old directory identifier reused
* JIT provisioning race
* Migration table duplicate

Use a uniqueness constraint on stable external identity:

```text id="g8zgjr"
UNIQUE(external_issuer, external_subject)
```

Account merges should be explicit and audited.

---

# 15.121 Advanced design checklist

## Protocol binding

* Issuer is explicit.
* Audience is explicit.
* Token or assertion type is enforced.
* Redirect or recipient is exact.
* Nonce and state are transaction-bound.
* Request proofs bind method and target.
* Canonicalization is defined.
* Duplicate fields are rejected or normalized consistently.

## Replay resistance

* Credentials have bounded lifetime.
* Single-use values are consumed atomically.
* Replay caches cover the validity window.
* Distributed replay semantics are defined.
* Retries and idempotency are separated from replay attacks.

## Delegation

* Actor and subject are distinct.
* Audience and scope are narrowed.
* Delegation depth is bounded.
* Downstream authority cannot exceed upstream authority.
* Impersonation is explicit and audited.

## Distributed authorization

* Consistency model is documented.
* Revision tokens exist where security changes require freshness.
* Cache keys include tenant, resource, action, and policy version.
* Graph traversal is bounded.
* Check and list APIs share semantics.

## Identity lifecycle

* Stable IDs are never reused.
* Provisioning and authentication are separate.
* Deprovisioning revokes every credential type.
* Account linking requires strong proof.
* Dormant credentials are inventoried.
* Role drift is reviewed.

## Advanced cryptography

* Keys are separated by protocol and purpose.
* Algorithms are locally allowlisted.
* Domain separation prevents cross-protocol reuse.
* Hardware-backed claims are not overinterpreted.
* Formal proofs are matched by deployment controls.

---

# 15.122 Questions: Protocol Binding and Replay

1. What is the difference between a bearer token and a proof-of-possession token?
2. Why can a copied proof-of-possession token remain unusable to the thief?
3. Which request fields should a request-bound proof cover?
4. Why must a proof identifier be cached during its acceptance window?
5. How can load balancing defeat a replica-local replay cache?
6. Why does a timestamp not prevent replay by itself?
7. What does a nonce prove that a timestamp does not?
8. Why must a single-use credential be consumed atomically?
9. How can URL canonicalization differences break or bypass signed-request validation?
10. What risk arises when a gateway and backend disagree about duplicate headers?
11. Why should ambiguous duplicate JSON keys be rejected in signed security objects?
12. What is the difference between validating a signature and validating the signed protocol?
13. How does channel binding restrict credential forwarding?
14. Why can mTLS token binding fail when TLS terminates at a proxy?
15. What does explicit key confirmation add to key establishment?

---

# 15.123 Questions: Delegation, Impersonation, and Capabilities

1. What is the difference between the subject and actor of a delegated request?
2. Why must downstream delegated authority be no broader than upstream authority?
3. How can unlimited delegation depth complicate incident response?
4. What is a confused deputy?
5. Why can a service’s own broad cloud role create a confused-deputy vulnerability?
6. How does token exchange reduce confused-deputy risk?
7. What security properties distinguish controlled impersonation from resetting a user’s password?
8. Which fields should appear in an impersonation audit event?
9. What is a capability?
10. How does capability-based access differ from role lookup?
11. What does capability attenuation guarantee?
12. What is a first-party caveat?
13. What role does a third-party discharge credential play?
14. Why are offline-verifiable capabilities difficult to revoke immediately?
15. Why should a pre-signed URL be treated as a bearer credential?

---

# 15.124 Questions: Distributed Authorization

1. What does a relationship tuple represent?
2. How can group cycles affect graph authorization?
3. Why must authorization graph traversal have depth and cost limits?
4. What inconsistency occurs when a group removal reaches one replica but not another?
5. How does a minimum revision token improve revocation correctness?
6. What is snapshot consistency useful for during paginated listing?
7. Why can post-filtering resources after pagination produce incorrect results?
8. How should `Check`, `List objects`, and `List subjects` semantics remain aligned?
9. What is the difference between deny, indeterminate, and not applicable?
10. Why must a policy-combining algorithm be explicit?
11. How should a missing required device attribute be handled?
12. Why should authorization input use an explicit schema?
13. What does partial evaluation precompute?
14. How can partial evaluation become stale?
15. What provenance should accompany a high-value authorization decision?

---

# 15.125 Questions: Identity Lifecycle

1. Why are provisioning and authentication separate processes?
2. How can JIT provisioning create excessive privilege?
3. Why should local identity use a stable internal identifier rather than username?
4. How can identifier reuse transfer old ownership to a new person?
5. What is privilege drift?
6. Which events should trigger automatic role removal?
7. Why does account disablement require more than blocking new login?
8. Which credential types must be considered during deprovisioning?
9. Why is deleting an account immediately sometimes worse than disabling it?
10. What information should an access review include beyond effective permission?
11. How does permission provenance help remove inappropriate access?
12. Why are dormant API keys still dangerous?
13. What makes password recovery an authentication protocol?
14. Why must reset-token consumption be atomic?
15. Why can account security become equal to its weakest linked identity?
16. How can assurance-aware account linking prevent this?
17. Why should MFA removal require stronger controls than an ordinary profile edit?
18. What risk arises when an attacker with a stolen session enrolls a new authenticator?
19. How can group deletion and recreation restore old authorization unexpectedly?
20. What should happen to active sessions after an employee is terminated?

---

# 15.126 Questions: Passkeys and Hardware-Backed Identity

1. Which data does a WebAuthn relying party store?
2. What does the authenticator keep secret?
3. Why is the server challenge required?
4. How does relying-party binding resist phishing?
5. What is the difference between user presence and user verification?
6. How does a discoverable credential enable username-less authentication?
7. What new trust dependencies arise with synchronized passkeys?
8. What can authenticator attestation prove?
9. Why may strict attestation reduce privacy and interoperability?
10. Why is a signature-counter anomaly not always definitive evidence of cloning?
11. What must a server validate besides the assertion signature?
12. How can a hardware-backed key remain vulnerable to malware invoking its signing API?
13. What is transaction signing?
14. Why must the user-visible transaction match the signed bytes?
15. How can ambiguous currency or decimal formatting undermine transaction signing?

---

# 15.127 Questions: Formal and Advanced Security Properties

1. What is included in the Trusted Computing Base of a JWT-based API?
2. How does reducing the TCB improve reviewability?
3. What is the difference between the security control plane and data plane?
4. Why is a correctly signed old policy still potentially dangerous?
5. What is a rollback attack?
6. How does domain separation reduce cross-protocol signature reuse?
7. What is an unknown key-share attack?
8. How do downgrade attacks differ from ordinary implementation bugs?
9. What does forward secrecy protect?
10. What does it not protect?
11. What is the difference between a safety property and a liveness property?
12. Why can an always-deny system satisfy safety while being unusable?
13. What can formal protocol verification prove?
14. Which deployment assumptions remain outside the proof?
15. What does delegation monotonicity mean?
16. What is a non-monotonic authorization policy?
17. Why do explicit-deny rules complicate caching?
18. What does non-interference attempt to guarantee?
19. Which side channels can violate non-interference?
20. How can information-flow controls complement ordinary access checks?

---

# 15.128 Questions: Workflow and Asynchronous Authorization

1. Why is a role check insufficient for payment approval?
2. How can the database enforce creator-versus-approver separation atomically?
3. What is a TOCTOU authorization defect?
4. How does resource-version binding reduce distributed TOCTOU risk?
5. What information could an authorization decision token contain?
6. What are the revocation costs of decision tokens?
7. Should a queued job use submission-time or execution-time authority?
8. Which business factors determine that choice?
9. What should an authorization envelope for a background job record?
10. Why is exactly-once processing difficult in distributed systems?
11. How does an idempotency key differ from a replay nonce?
12. What problem does a transactional outbox solve?
13. Which identities belong in an asynchronous audit chain?
14. How can a duplicate message cause an authorization-sensitive side effect twice?
15. Why should retries reapply consistent authorization semantics?

# 16. Practical Reference Architectures and Implementation Blueprints

This section converts the earlier principles into concrete system designs.

The goal is not to prescribe one universal stack.

The goal is to show how to preserve the same security properties across different architectures:

```text
Authenticate the correct principal
→ establish bounded security context
→ authorize every protected operation
→ enforce the result
→ preserve audit evidence
→ support revocation and recovery
```

---

# 16.1 Architecture-selection questions

Before choosing a mechanism, answer:

1. Is the client a browser, native application, server, device, or administrator?
2. Is the subject a human, workload, or both?
3. Does the client safely store secrets?
4. Does the application need delegated access to other APIs?
5. Must credentials work offline?
6. How quickly must revocation take effect?
7. Is the system single-tenant or multi-tenant?
8. Are requests synchronous, asynchronous, or both?
9. Which components are exposed to untrusted networks?
10. Which identity provider, directory, or cloud control plane already exists?
11. Which operations require step-up authentication?
12. Which identity or policy outages must existing sessions survive?

---

## Architecture decision matrix

| Requirement                         | Common design                                  |
| ----------------------------------- | ---------------------------------------------- |
| Traditional server-rendered web app | Server-side session cookie                     |
| Browser app calling several APIs    | Backend-for-Frontend                           |
| Native mobile or desktop client     | Authorization Code with PKCE                   |
| Machine-to-machine API              | Workload identity, mTLS, or client credentials |
| Internal Kubernetes workloads       | ServiceAccount plus workload identity          |
| Enterprise browser SSO              | OIDC or SAML                                   |
| Enterprise host login               | Kerberos, PAM, directory integration           |
| Short-lived delegated API access    | OAuth access token                             |
| Rapid central revocation            | Server-side session or introspection           |
| Offline API validation              | Short-lived signed access token                |
| Highly sensitive administration     | Dedicated admin plane plus step-up             |
| Cross-service end-user propagation  | Token exchange or constrained delegation       |

---

# 16.2 Reference Architecture A: Server-Rendered Web Application

Use case:

```text
Browser
→ server-rendered application
→ application database
```

Examples:

* Internal reporting portal
* Administrative dashboard
* Customer account website
* Traditional monolithic business application

Recommended identity pattern:

```text
User authenticates
→ application creates server-side session
→ browser receives opaque cookie
→ every protected request resolves session
→ application authorizes resource
```

---

## Component diagram

```mermaid
flowchart LR
    B[Browser]
    W[Web Application]
    S[(Session Store)]
    U[(User Database)]
    P[Authorization Module]
    D[(Business Database)]
    A[(Audit Log)]

    B -->|HTTPS + session cookie| W
    W --> S
    W --> U
    W --> P
    W --> D
    W --> A
```

---

# 16.3 Login flow

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant App
    participant Users as User Store
    participant Sessions as Session Store
    participant Audit

    User->>Browser: Submit username and password
    Browser->>App: POST /login
    App->>App: Validate CSRF and input limits
    App->>Users: Load password-verification record
    Users-->>App: User record or indistinguishable absence
    App->>App: Verify password
    App->>App: Check account and MFA policy
    App->>Sessions: Create random server-side session
    Sessions-->>App: Session committed
    App->>Audit: Record successful authentication
    App-->>Browser: Set-Cookie + redirect
    Browser->>App: GET /dashboard + cookie
    App->>Sessions: Resolve session
    Sessions-->>App: Active security context
    App-->>Browser: Dashboard
```

---

# 16.4 Suggested session cookie

Illustrative response:

```http
Set-Cookie: __Host-session=RANDOM_OPAQUE_VALUE; Path=/; Secure; HttpOnly; SameSite=Lax
```

Properties:

* `__Host-` prefix requires host-only secure semantics in supporting browsers.
* `Path=/` covers the application.
* `Secure` restricts transmission to HTTPS.
* `HttpOnly` prevents ordinary JavaScript access.
* `SameSite=Lax` provides useful cross-site request restrictions while supporting common top-level navigation flows.

Exact SameSite policy depends on application and federation architecture.

---

## Do not include authority in the cookie

Avoid:

```json
{
  "user": "alice",
  "role": "admin",
  "tenant": "tenant-42"
}
```

unless the entire structure is securely signed or encrypted and revocation semantics are deliberately designed.

A safer server-side model uses:

```text
Cookie:
  random session identifier

Server-side record:
  user, authentication context, expiry, generation
```

---

# 16.5 Session schema

Illustrative SQL:

```sql
CREATE TABLE application_sessions (
    session_hash        BYTEA PRIMARY KEY,
    user_id             UUID NOT NULL,
    tenant_id           UUID,
    created_at          TIMESTAMPTZ NOT NULL,
    last_seen_at        TIMESTAMPTZ NOT NULL,
    idle_expires_at     TIMESTAMPTZ NOT NULL,
    absolute_expires_at TIMESTAMPTZ NOT NULL,
    auth_time           TIMESTAMPTZ NOT NULL,
    auth_methods        TEXT[] NOT NULL,
    assurance_level     TEXT NOT NULL,
    session_generation  BIGINT NOT NULL,
    revoked_at          TIMESTAMPTZ,
    revocation_reason   TEXT,
    device_id           UUID,
    source_risk_class   TEXT
);
```

Recommended properties:

* Store a hash of the session ID rather than the raw cookie value.
* Use random high-entropy session IDs.
* Record idle and absolute expiry separately.
* Preserve authentication time.
* Preserve authentication methods and assurance.
* Support explicit revocation.
* Include a generation or version for global invalidation.
* Do not store secrets that the session does not require.

---

# 16.6 Session creation pseudocode

```python
from __future__ import annotations

import hashlib
import secrets
from dataclasses import dataclass
from datetime import datetime, timedelta, timezone


@dataclass(frozen=True)
class CreatedSession:
    cookie_value: str
    stored_hash: bytes
    idle_expires_at: datetime
    absolute_expires_at: datetime


def create_session(
    *,
    idle_lifetime: timedelta = timedelta(minutes=30),
    absolute_lifetime: timedelta = timedelta(hours=8),
) -> CreatedSession:
    now = datetime.now(timezone.utc)

    # 32 random bytes provide 256 bits before encoding.
    cookie_value = secrets.token_urlsafe(32)
    stored_hash = hashlib.sha256(cookie_value.encode("ascii")).digest()

    return CreatedSession(
        cookie_value=cookie_value,
        stored_hash=stored_hash,
        idle_expires_at=now + idle_lifetime,
        absolute_expires_at=now + absolute_lifetime,
    )
```

The database insertion must complete before the cookie is returned.

If insertion fails:

```text
Do not issue the cookie
Do not report successful login
```

---

# 16.7 Session validation pseudocode

```python
from __future__ import annotations

import hashlib
import hmac
from dataclasses import dataclass
from datetime import datetime, timezone
from typing import Protocol


@dataclass(frozen=True)
class SecurityContext:
    user_id: str
    tenant_id: str | None
    auth_time: datetime
    auth_methods: tuple[str, ...]
    assurance_level: str


class SessionRepository(Protocol):
    def get_by_hash(self, session_hash: bytes): ...


def resolve_session(
    raw_cookie: str | None,
    repository: SessionRepository,
    *,
    expected_generation: int,
) -> SecurityContext | None:
    if raw_cookie is None or len(raw_cookie) > 512:
        return None

    candidate_hash = hashlib.sha256(raw_cookie.encode("ascii")).digest()
    session = repository.get_by_hash(candidate_hash)

    if session is None:
        return None

    now = datetime.now(timezone.utc)

    if session.revoked_at is not None:
        return None

    if now >= session.idle_expires_at:
        return None

    if now >= session.absolute_expires_at:
        return None

    if session.session_generation != expected_generation:
        return None

    return SecurityContext(
        user_id=session.user_id,
        tenant_id=session.tenant_id,
        auth_time=session.auth_time,
        auth_methods=tuple(session.auth_methods),
        assurance_level=session.assurance_level,
    )
```

The lookup should use an indexed fixed-length hash.

Do not use a linear scan with constant-time comparison across all session records.

---

# 16.8 Session fixation prevention

After successful login:

```text
Anonymous session identifier
must not become
authenticated session identifier unchanged
```

Instead:

1. Invalidate or detach anonymous session.
2. Generate a new random session ID.
3. Transfer only explicitly safe state.
4. Store authenticated session.
5. Issue new cookie.

Repeat rotation after:

* MFA completion
* Privilege elevation
* Impersonation entry
* Account recovery
* Significant identity change

---

# 16.9 CSRF protection

A cookie-authenticated application should protect state-changing requests.

Common controls:

* SameSite cookies
* Synchronizer token
* Double-submit pattern where correctly implemented
* Origin validation
* Referer validation as a supporting control
* Reauthentication for high-risk actions

Illustrative form:

```html
<form method="post" action="/reports/71/delete">
  <input type="hidden" name="csrf_token" value="RANDOM_BOUND_VALUE">
  <button type="submit">Delete report</button>
</form>
```

The server checks that the token:

* Belongs to the current session
* Is unpredictable
* Is correct for the request model
* Uses constant-time comparison where appropriate
* Has not been replaced by an attacker-controlled value

---

# 16.10 Object authorization in the monolith

Unsafe:

```python
report = reports.get(report_id)
return render(report)
```

Safer:

```python
report = reports.get_authorized(
    report_id=report_id,
    subject_id=context.user_id,
    tenant_id=context.tenant_id,
)

if report is None:
    raise NotFoundOrForbidden()
```

Database-level approach:

```sql
SELECT report_id, title, body
FROM reports
WHERE report_id = :report_id
  AND tenant_id = :tenant_id
  AND (
        owner_user_id = :subject_id
        OR department_id = ANY(:authorized_departments)
      );
```

Authorization belongs in the data-access path, not only in the user interface.

---

# 16.11 Logout flow

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant App
    participant Sessions

    User->>Browser: Select logout
    Browser->>App: POST /logout + session cookie + CSRF token
    App->>Sessions: Mark session revoked
    Sessions-->>App: Commit success
    App-->>Browser: Expire session cookie
    Browser->>App: Later request without valid session
    App-->>Browser: Redirect to login or return 401
```

Cookie removal alone is insufficient if the server-side session remains active.

---

## Questions: Server-Rendered Application

1. Why should the session record be committed before returning `Set-Cookie`?
2. What does hashing the stored session identifier protect against?
3. Why should the session ID rotate after MFA completion?
4. What is the difference between idle and absolute expiry?
5. Why should object authorization be expressed in the data query where possible?
6. How can a global session generation invalidate every session for one user?
7. Why does deleting the browser cookie not revoke a copied cookie?
8. Which state can be safely transferred from an anonymous session into an authenticated session?
9. Why is `HttpOnly` not a complete defense against XSS-driven account actions?
10. Which operations should require fresh step-up authentication?

---

# 16.12 Reference Architecture B: Browser Application with BFF

Use case:

```text
Browser JavaScript application
→ Backend-for-Frontend
→ internal APIs
→ external identity provider
```

The BFF keeps OAuth tokens out of browser JavaScript.

```mermaid
flowchart LR
    B[Browser SPA]
    F[BFF]
    I[OIDC Provider]
    A1[Reports API]
    A2[Payments API]
    S[(Session and Token Store)]

    B -->|Session cookie| F
    F -->|OIDC back channel| I
    F -->|Audience token| A1
    F -->|Audience token| A2
    F --> S
```

---

# 16.13 BFF security boundaries

Browser receives:

```text
Opaque session cookie
```

BFF stores:

```text
Access tokens
Refresh token
OIDC transaction state
User identity
Token expiries
```

APIs receive:

```text
Audience-specific access token
Authenticated BFF workload identity
```

---

# 16.14 BFF login sequence

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant BFF
    participant IdP
    participant Store

    User->>Browser: Open application
    Browser->>BFF: GET /
    BFF->>BFF: Generate state, nonce, PKCE verifier
    BFF->>Store: Save login transaction
    BFF-->>Browser: Redirect to IdP

    Browser->>IdP: Authorization request
    IdP->>User: Authenticate and obtain consent
    IdP-->>Browser: Redirect with code and state
    Browser->>BFF: GET /callback?code=...&state=...

    BFF->>Store: Load and consume transaction
    BFF->>BFF: Validate state
    BFF->>IdP: Exchange code + PKCE verifier
    IdP-->>BFF: ID Token, access token, refresh token
    BFF->>BFF: Validate ID Token
    BFF->>Store: Create local session and protect tokens
    BFF-->>Browser: Set session cookie and redirect
```

---

# 16.15 BFF API call

```mermaid
sequenceDiagram
    participant Browser
    participant BFF
    participant Store
    participant API

    Browser->>BFF: GET /bff/reports/71 + session cookie
    BFF->>Store: Resolve session and API token
    Store-->>BFF: User context + token metadata
    BFF->>BFF: Check CSRF/origin where applicable
    BFF->>API: GET /reports/71 + access token
    API->>API: Validate token and authorize resource
    API-->>BFF: Report
    BFF-->>Browser: Report
```

The BFF should not become an authorization bypass.

The API should still enforce its own resource policy.

---

# 16.16 Token storage schema

Illustrative fields:

```sql
CREATE TABLE oauth_token_grants (
    grant_id                   UUID PRIMARY KEY,
    session_hash               BYTEA NOT NULL,
    issuer                     TEXT NOT NULL,
    subject                    TEXT NOT NULL,
    client_id                  TEXT NOT NULL,
    refresh_token_ciphertext   BYTEA,
    refresh_token_key_version  TEXT,
    refresh_token_expires_at   TIMESTAMPTZ,
    token_family_id            UUID,
    revoked_at                 TIMESTAMPTZ,
    created_at                 TIMESTAMPTZ NOT NULL,
    updated_at                 TIMESTAMPTZ NOT NULL
);
```

Access tokens may be:

* Cached in encrypted server-side storage
* Requested through token exchange
* Kept in memory with bounded lifetime
* Reobtained using refresh token

Do not store more tokens than required.

---

# 16.17 Token refresh coordination

Pseudocode:

```python
class TokenCache:
    def get_valid_token(self, grant_id: str, audience: str) -> str:
        cache_key = (grant_id, audience)

        token = self.read(cache_key)
        if token is not None and token.expires_in_seconds > 60:
            return token.value

        # The lock implements single-flight refresh for this grant and audience.
        with self.lock(cache_key):
            token = self.read(cache_key)
            if token is not None and token.expires_in_seconds > 60:
                return token.value

            refreshed = self.refresh_from_issuer(
                grant_id=grant_id,
                audience=audience,
            )
            self.write(cache_key, refreshed)
            return refreshed.value
```

The cache key must include:

* Grant
* Audience
* Scope or permission set
* Tenant where relevant
* Token type

---

# 16.18 BFF CSRF model

The BFF uses cookies, so browser-originated state-changing calls still need CSRF controls.

Possible policy:

```text
GET and HEAD:
  no state change

POST, PUT, PATCH, DELETE:
  require:
    session cookie
    CSRF token
    accepted Origin
    JSON content type where applicable
```

Do not treat an access token stored inside the BFF as automatic CSRF protection.

The browser sends the session cookie automatically.

---

# 16.19 BFF route allowlist

Avoid creating a fully generic proxy:

```text
/bff/proxy?url=https://arbitrary-target
```

This can produce:

* SSRF
* Token leakage
* Audience confusion
* Cross-service privilege escalation
* Uncontrolled methods and headers

Prefer explicit routes:

```text
GET  /bff/reports/{id}
POST /bff/payments
GET  /bff/profile
```

Each route selects:

* Fixed downstream service
* Fixed or bounded method
* Intended token audience
* Allowed request headers
* Response transformation

---

# 16.20 Token exchange at the BFF

For a downstream API, the BFF may request:

```text
subject = Alice
actor = finance-web BFF
audience = reports-api
scope = reports.read
lifetime = 5 minutes
```

The API learns both:

```text
Alice is the subject
finance-web is the actor
```

This is safer than forwarding one general-purpose user token everywhere.

---

## Questions: BFF Architecture

1. Why does a BFF reduce token exposure in browser JavaScript?
2. Why does a BFF still require CSRF protection?
3. Why should the downstream API authorize the report again?
4. Which fields belong in the token-cache key?
5. What problem does single-flight token refresh solve?
6. Why is a generic BFF URL proxy dangerous?
7. How does token exchange preserve both subject and actor?
8. What should happen if the BFF refresh token is stolen?
9. Why should tokens for reports-api and payments-api have different audiences?
10. How can a BFF outage affect existing browser sessions even when the IdP remains healthy?

---

# 16.21 Reference Architecture C: Native Mobile or Desktop Application

Use case:

```text
Native application
→ external browser
→ authorization server
→ APIs
```

Recommended flow:

```text
Authorization Code
+
PKCE
+
system browser
+
application redirect
```

---

## Architecture

```mermaid
flowchart LR
    A[Native Application]
    B[System Browser]
    I[Authorization Server]
    API[Resource API]
    K[Platform Secure Storage]

    A --> B
    B --> I
    I --> B
    B --> A
    A --> K
    A --> API
```

The native application is normally a public client.

It cannot safely depend on an embedded static client secret.

---

# 16.22 Native application flow

```mermaid
sequenceDiagram
    actor User
    participant App as Native App
    participant Browser as System Browser
    participant AS as Authorization Server
    participant API

    App->>App: Generate state and PKCE verifier
    App->>Browser: Open authorization URL
    Browser->>AS: Authorization request + PKCE challenge
    AS->>User: Authenticate
    AS-->>Browser: Redirect with code and state
    Browser->>App: Invoke registered app redirect
    App->>App: Validate state
    App->>AS: Code + PKCE verifier
    AS-->>App: Access and refresh tokens
    App->>App: Store refresh token in secure platform storage
    App->>API: Access token
    API-->>App: Protected result
```

---

# 16.23 Redirect choices

Possible patterns:

* Claimed HTTPS redirect
* Application-specific URI scheme
* Loopback redirect for desktop application

Security requirements:

* Only the intended application should receive the callback where possible.
* `state` must bind the response to the original transaction.
* PKCE must prevent a stolen authorization code from being redeemed by another application.
* Redirect URI must match registration policy.
* Authorization codes must be short-lived and single-use.

---

# 16.24 Native token storage

Use platform-provided secure storage where available.

Examples of desired properties:

* Access controlled by application identity
* Device lock integration
* Hardware-backed protection where available
* Backup behavior understood
* Data removed on account logout where required

Do not assume “secure storage” prevents malware running with equivalent device privileges.

---

# 16.25 Native logout

Logout may involve:

1. Delete local access and refresh tokens.
2. Revoke refresh token or token family.
3. Clear local application session state.
4. Optionally initiate provider logout.
5. Preserve or remove account-selection hints according to privacy policy.

Deleting only the access token is insufficient if the refresh token remains.

---

# 16.26 Offline operation

A native app may retain locally cached data while offline.

Authentication and authorization questions become:

```text
May the user open cached data without contacting server?
How long after account disablement?
Must device unlock be required?
Is data encrypted at rest?
Can screenshots or backups expose it?
```

Offline access is a separate authorization policy.

---

## Questions: Native Applications

1. Why is a native application generally considered a public client?
2. What attack does PKCE mitigate?
3. Why should the system browser normally be used instead of an embedded password form?
4. Which callback patterns can a native application use?
5. Why must the redirect response still be checked with `state`?
6. What must logout delete besides the current access token?
7. How does offline cached data change revocation semantics?
8. What security assumptions are introduced by device secure storage?
9. Why does a client secret compiled into an application provide weak authentication?
10. How can a malicious application attempt to intercept a custom-scheme redirect?

---

# 16.27 Reference Architecture D: Machine-to-Machine API

Use case:

```text
batch-exporter
→ reports-api
```

No human is directly present.

Preferred mechanisms:

* Workload-issued mTLS certificate
* Short-lived OAuth client credential
* Cloud workload federation
* Signed request
* Kerberos service credential in enterprise environments

Avoid permanent shared API keys where a stronger workload identity is available.

---

## Architecture

```mermaid
flowchart LR
    B[Batch Exporter]
    I[Workload Identity Issuer]
    T[Authorization Server]
    R[Reports API]
    P[Policy Engine]

    B --> I
    B --> T
    B -->|mTLS + audience token| R
    R --> P
```

---

# 16.28 Combined mTLS and token model

mTLS answers:

```text
Which workload established this transport connection?
```

OAuth token answers:

```text
Which delegated or application authority is presented for this API?
```

Policy can require both:

```text
transport identity = batch-exporter
AND
token client = batch-exporter
AND
audience = reports-api
AND
scope includes reports.export
```

This prevents a valid token stolen by another workload from being sufficient.

---

# 16.29 Client-credentials request

Illustrative command:

```bash
curl -sS \
  -u "${CLIENT_ID}:${CLIENT_SECRET}" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=client_credentials' \
  --data-urlencode 'scope=reports.export' \
  https://identity.example.internal/oauth/token
```

### Important safety points

* Shell variables may appear in process environments or debugging output.
* The constructed Basic credential may be visible in process arguments depending on invocation.
* Use a short-lived test credential in diagnostics.
* Prefer workload federation or private-key client authentication for production where available.
* Never place the client secret in source code.

---

# 16.30 Private-key client authentication

Instead of a shared client secret, a client can authenticate using a private key.

Conceptual flow:

```text
Client creates signed assertion
→ token endpoint verifies registered public key
→ token issued
```

The assertion should be bound to:

* Client ID
* Token endpoint audience
* Issuer
* Short time interval
* Unique identifier

Replay protection and key rotation remain necessary.

---

# 16.31 Service authorization policy

Example policy:

```text
ALLOW:
  actor workload = batch-exporter
  action = reports.export
  tenant = tenant-42
  export format = CSV
  maximum rows = 10,000

DENY:
  batch-exporter may not:
    delete reports
    change access policy
    export another tenant
```

A machine identity is not automatically entitled to every action exposed by the API.

---

# 16.32 API request example

```http
POST /v1/report-exports HTTP/1.1
Host: reports-api.internal
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
Idempotency-Key: export-job-9182

{
  "tenant_id": "tenant-42",
  "report_id": "report-71",
  "format": "csv"
}
```

Server validates:

1. TLS client identity
2. Access-token signature and claims
3. Token binding where used
4. Idempotency key
5. Tenant authority
6. Report access
7. Export limits
8. Current policy
9. Existing duplicate job

---

# 16.33 Machine credential rotation

Rotation sequence:

```text
Issue new credential
→ distribute or make discoverable
→ begin using new credential
→ confirm success
→ stop using old credential
→ revoke old credential
→ verify no old use remains
```

Telemetry should identify credential generation without logging the credential.

Example:

```json
{
  "client_id": "batch-exporter",
  "credential_generation": "key-2026-07",
  "result": "success"
}
```

---

## Questions: Machine-to-Machine Architecture

1. What different facts do mTLS and an OAuth access token prove?
2. Why can combining them reduce the impact of token theft?
3. Why is a shared API key across many replicas difficult to audit?
4. What should a service-client credential be bound to?
5. Why should machine permissions include tenant and transaction constraints?
6. What replay problem can an idempotency key solve?
7. Why is private-key client authentication often preferable to a shared client secret?
8. What must be monitored during credential rotation?
9. Why does a valid workload identity not grant every API permission?
10. Which events should revoke or replace a workload credential immediately?

---

# 16.34 Reference Architecture E: Kubernetes Workload to Cloud Resource

Use case:

```text
Kubernetes Pod
→ cloud token service
→ object storage
```

Preferred flow:

```text
Kubernetes ServiceAccount assertion
→ cloud federation exchange
→ temporary role credential
→ cloud API
```

No permanent cloud access key is stored in the cluster.

---

## Architecture

```mermaid
flowchart LR
    P[Pod]
    K[Kubernetes Token Issuer]
    C[Cloud Token Service]
    O[Object Storage]
    A[Cloud Audit]

    K -->|Projected ServiceAccount token| P
    P -->|Federation exchange| C
    C -->|Temporary role credential| P
    P -->|Signed object request| O
    O --> A
```

---

# 16.35 Kubernetes objects

ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: report-exporter
  namespace: finance
automountServiceAccountToken: false
```

Pod with an explicitly projected token:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: report-exporter
  namespace: finance
spec:
  serviceAccountName: report-exporter
  automountServiceAccountToken: false
  containers:
    - name: exporter
      image: registry.example.internal/report-exporter:2.4.1
      volumeMounts:
        - name: workload-token
          mountPath: /var/run/workload-identity
          readOnly: true
  volumes:
    - name: workload-token
      projected:
        sources:
          - serviceAccountToken:
              path: token
              audience: cloud-token-service
              expirationSeconds: 3600
```

This requests a token intended for:

```text
cloud-token-service
```

rather than using a generic token intended for every possible verifier.

---

# 16.36 Cloud trust mapping

Conceptual trust policy:

```text
Trusted issuer:
  production cluster issuer

Required audience:
  cloud-token-service

Required subject:
  system:serviceaccount:finance:report-exporter

Mapped role:
  finance-report-object-writer
```

Cloud role permissions:

```text
Allow:
  write objects under:
    finance-exports/tenant-42/

Deny:
  delete buckets
  read other tenants
  modify access policy
```

---

# 16.37 Pod flow

```mermaid
sequenceDiagram
    participant Pod
    participant Kubelet
    participant K8s as Kubernetes API
    participant CloudSTS as Cloud Token Service
    participant Storage

    Kubelet->>K8s: Request bounded ServiceAccount token
    K8s-->>Kubelet: Signed token
    Kubelet->>Pod: Project token file

    Pod->>CloudSTS: Exchange ServiceAccount token
    CloudSTS->>CloudSTS: Validate issuer, audience, subject
    CloudSTS-->>Pod: Temporary cloud role credential

    Pod->>Storage: Signed object write
    Storage->>Storage: Authenticate cloud role
    Storage->>Storage: Authorize object path
    Storage-->>Pod: Success
```

---

# 16.38 Rotation and file watching

Projected token files may rotate.

Application code should:

* Reopen or reread token when needed.
* Avoid reading only once at image startup.
* Handle atomic file replacement.
* Refresh cloud credentials before expiry.
* Back off with jitter on token-service failures.
* Retain current valid cloud credential until safe refresh threshold.
* Never log token file contents.

---

# 16.39 Debugging from inside the Pod

Inspect ServiceAccount:

```bash
kubectl get pod report-exporter \
  --namespace finance \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

Inspect projected files without displaying token:

```bash
kubectl exec \
  --namespace finance \
  report-exporter \
  -- sh -c '
      ls -l /var/run/workload-identity &&
      wc -c /var/run/workload-identity/token
    '
```

### Safety

Do not run:

```bash
cat /var/run/workload-identity/token
```

during routine troubleshooting.

The token is a live bearer credential.

---

# 16.40 Verify cloud exchange assumptions

Capture only metadata:

```text
Kubernetes issuer
ServiceAccount subject
Token audience
Token expiry
Cloud role name
Cloud credential expiry
Cloud request identity
Object path
```

Do not record:

* Raw projected token
* Temporary access secret
* Cloud session token
* Signed authorization header

---

## Questions: Kubernetes-to-Cloud Architecture

1. Why should the projected token use a cloud-token-service audience?
2. What exact Kubernetes subject should the cloud trust policy require?
3. Why is namespace part of the ServiceAccount identity?
4. Why should automatic token mounting be disabled when explicit projection is used?
5. How does temporary cloud federation reduce secret-zero risk?
6. What could happen if the trust policy accepts every ServiceAccount in the cluster?
7. Why must the application reread the projected token after rotation?
8. Which cloud permissions should be restricted by object path or resource?
9. How can a compromised Pod use an overly broad cloud role?
10. Which logs connect the Kubernetes workload identity to the cloud API action?

---

# 16.41 Reference Architecture F: Multi-Tenant SaaS

Use case:

```text
Many organizations use one application
Each organization must remain isolated
Users may belong to more than one tenant
```

The critical rule is:

```text
Identity is global or issuer-scoped
Authorization is tenant-scoped
```

---

## Core entities

```text
User:
  stable human or external identity

Tenant:
  organization boundary

Membership:
  relation between user and tenant

Role assignment:
  authority within one tenant

Resource:
  owned by one tenant
```

---

# 16.42 Suggested data model

```sql
CREATE TABLE users (
    user_id          UUID PRIMARY KEY,
    status           TEXT NOT NULL,
    created_at       TIMESTAMPTZ NOT NULL
);

CREATE TABLE external_identities (
    issuer           TEXT NOT NULL,
    subject          TEXT NOT NULL,
    user_id          UUID NOT NULL REFERENCES users(user_id),
    created_at       TIMESTAMPTZ NOT NULL,
    PRIMARY KEY (issuer, subject),
    UNIQUE (issuer, subject, user_id)
);

CREATE TABLE tenants (
    tenant_id        UUID PRIMARY KEY,
    name             TEXT NOT NULL,
    status           TEXT NOT NULL
);

CREATE TABLE tenant_memberships (
    tenant_id        UUID NOT NULL REFERENCES tenants(tenant_id),
    user_id          UUID NOT NULL REFERENCES users(user_id),
    status           TEXT NOT NULL,
    membership_version BIGINT NOT NULL,
    PRIMARY KEY (tenant_id, user_id)
);

CREATE TABLE tenant_role_assignments (
    tenant_id        UUID NOT NULL,
    user_id          UUID NOT NULL,
    role_name        TEXT NOT NULL,
    granted_at       TIMESTAMPTZ NOT NULL,
    expires_at       TIMESTAMPTZ,
    granted_by       UUID NOT NULL,
    PRIMARY KEY (tenant_id, user_id, role_name),
    FOREIGN KEY (tenant_id, user_id)
        REFERENCES tenant_memberships(tenant_id, user_id)
);
```

---

# 16.43 Resource ownership schema

```sql
CREATE TABLE reports (
    tenant_id        UUID NOT NULL,
    report_id        UUID NOT NULL,
    owner_user_id    UUID NOT NULL,
    department_id    UUID,
    classification   TEXT NOT NULL,
    title            TEXT NOT NULL,
    body              TEXT NOT NULL,
    revision          BIGINT NOT NULL,
    PRIMARY KEY (tenant_id, report_id)
);
```

Putting `tenant_id` into the primary key or unique key helps prevent ambiguous cross-tenant references.

---

# 16.44 Tenant selection

A user may belong to:

```text
tenant-42
tenant-77
```

The current tenant can come from:

* Hostname
* URL path
* Selected session context
* Explicit token claim
* Organization-specific login route

Do not trust a client-supplied tenant ID without checking membership.

---

## Tenant-switch flow

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant App
    participant Memberships
    participant Sessions

    User->>Browser: Select tenant-77
    Browser->>App: POST /switch-tenant tenant-77
    App->>Memberships: Is user active member of tenant-77?
    Memberships-->>App: Yes
    App->>Sessions: Update selected tenant and rotate session
    App-->>Browser: New session cookie or updated session
```

Session rotation helps prevent fixation and makes the context transition explicit.

---

# 16.45 Tenant authorization function

```python
def require_tenant_membership(
    *,
    subject_id: str,
    requested_tenant_id: str,
    membership_repository,
):
    membership = membership_repository.get(
        user_id=subject_id,
        tenant_id=requested_tenant_id,
    )

    if membership is None or membership.status != "active":
        raise AccessDenied("tenant_membership_required")

    return membership
```

Every tenant resource lookup should use the verified tenant context.

---

# 16.46 Unsafe object lookup

```sql
SELECT *
FROM reports
WHERE report_id = :report_id;
```

If report IDs are globally unique, the query still permits cross-tenant retrieval unless another check occurs.

Safer:

```sql
SELECT *
FROM reports
WHERE tenant_id = :authorized_tenant_id
  AND report_id = :report_id;
```

---

# 16.47 Global administrators

A global administrator should be represented separately from tenant-local roles.

Avoid:

```text
role = admin
```

Prefer:

```text
tenant role:
  tenant-42:billing-admin

global role:
  platform-security-admin
```

Global roles should require:

* Dedicated administrator identity
* Strong MFA
* Short session
* Step-up
* Explicit reason
* Restricted administrative interface
* Detailed audit

---

# 16.48 Tenant-aware token

A token may include:

```json
{
  "sub": "u_1048",
  "aud": "reports-api",
  "tenant": "tenant-42",
  "scope": "reports.read",
  "membership_version": 817
}
```

The API must still verify:

* Tenant exists and is active.
* Token tenant matches request tenant.
* Resource belongs to token tenant.
* Membership version is sufficiently current if used.
* Token issuer is authorized to assert tenant context.

---

# 16.49 Tenant isolation test property

```text
For every user U and report R:

if U is not an active member of tenant(R)
and U lacks explicit global cross-tenant authority

then:
  read(U, R) = deny
  modify(U, R) = deny
  export(U, R) = deny
  search_result_contains(U, R) = false
```

Test this across:

* Direct object APIs
* Search
* Export
* Batch
* GraphQL
* Background jobs
* Audit queries
* Attachments
* Pre-signed URLs
* Cache keys

---

# 16.50 Tenant cache isolation

Unsafe cache key:

```text
report_id
```

Safer cache key:

```text
tenant_id | report_id
```

Unsafe authorization cache:

```text
subject | report.read
```

Safer:

```text
subject | tenant | report | action | policy version
```

Cross-tenant cache contamination can expose data even when database authorization is correct.

---

## Questions: Multi-Tenant SaaS

1. Why should external identity and tenant membership be modeled separately?
2. What makes `(issuer, subject)` a safer external identity key than email?
3. Why should tenant ID be included in resource keys and cache keys?
4. How can a tenant-switch operation create a new security context?
5. Why should tenant-local administrator and global administrator be distinct roles?
6. What does a membership version enable?
7. Why must search and export paths be included in tenant-isolation testing?
8. How can a globally unique report ID still permit cross-tenant access?
9. Which identity-provider claims may be unsafe to treat as global roles?
10. How should a disabled tenant affect active user sessions?

---

# 16.51 Reference Architecture G: Administrative Control Plane

Administrative operations can alter the entire trust system.

Examples:

* Create administrator
* Add trusted issuer
* Register redirect URI
* Rotate signing key
* Change authorization policy
* Grant workload identity
* Disable audit
* Create break-glass credential

Administration should use a separate trust boundary.

---

## Architecture

```mermaid
flowchart LR
    A[Administrator Workstation]
    G[Admin Gateway]
    I[Privileged Identity Provider]
    C[Control Plane]
    P[Approval Service]
    K[Key Management]
    L[(Immutable Audit)]

    A -->|Strong authentication| G
    G --> I
    G --> C
    C --> P
    C --> K
    C --> L
```

---

# 16.52 Administrative identity

Prefer:

```text
Ordinary identity:
  alice@example.internal

Administrative identity:
  alice-admin@example.internal
```

Benefits:

* Separate session and device policy
* Reduced accidental privilege use
* Clear audit
* Different phishing and recovery policy
* Easier privilege expiration

The administrative identity should not be used for routine email or browsing.

---

# 16.53 Administrative session controls

Possible controls:

* Hardware-backed phishing-resistant authentication
* Managed administrator workstation
* Restricted network
* Short absolute lifetime
* No persistent refresh token
* Step-up for high-risk changes
* Command or change approval
* Session recording where lawful and appropriate
* Immediate alerting
* No silent impersonation

---

# 16.54 Change request state machine

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Submitted: Administrator submits
    Submitted --> Approved: Independent approver
    Submitted --> Rejected
    Approved --> Applied: Control plane executes
    Applied --> Verified: Post-change checks pass
    Applied --> RolledBack: Verification fails
    Rejected --> [*]
    Verified --> [*]
    RolledBack --> [*]
```

---

# 16.55 Example trusted-issuer change

Request:

```yaml
change_type: add_oidc_issuer
issuer: https://partner-id.example
allowed_audiences:
  - partner-portal
allowed_tenants:
  - tenant-77
claim_mapping_version: partner-v3
requested_by: admin-17
ticket: SEC-9182
```

Validation:

* HTTPS issuer
* Discovery retrieved through approved process
* Signing keys trusted only for this issuer
* Audience allowlist
* Tenant restrictions
* Claim mappings reviewed
* No global administrator mapping
* Test login in isolated environment
* Rollback plan

---

# 16.56 Key-rotation change

Key rotation should be a workflow, not one manual command.

Stages:

```text
Generate
→ register metadata
→ publish public material
→ verify distribution
→ activate
→ monitor
→ retire old key
→ destroy according to policy
```

Every stage should have:

* Actor
* Approval
* Timestamp
* Key identifier
* Environment
* Verification result

Never log raw private-key material.

---

# 16.57 Emergency break-glass path

Break-glass design:

```text
Normal IdP unavailable
→ two authorized custodians approve
→ retrieve emergency credential
→ authenticate through isolated endpoint
→ short emergency session
→ full audit and alert
→ rotate credential after use
```

Break-glass should not depend entirely on the failed identity system.

It must still depend on protected, testable controls.

---

## Questions: Administrative Plane

1. Why should administrative identities be separate from ordinary identities?
2. Which configuration changes are equivalent in impact to credential compromise?
3. Why should an administrator not directly edit production authorization data without a workflow?
4. What role does independent approval provide?
5. Why should a break-glass path not rely entirely on the normal IdP?
6. Which controls prevent break-glass access from becoming a routine shortcut?
7. Why should signing-key activation occur only after public-key distribution is verified?
8. What audit information belongs to a trusted-issuer change?
9. Why should partner claims be tenant-restricted?
10. What must happen immediately after a break-glass credential is used?

---

# 16.58 Shared Authentication Middleware Blueprint

A resource server commonly needs middleware that performs:

```text
Transport trust
→ credential extraction
→ cryptographic validation
→ identity mapping
→ security context creation
→ route authorization
```

---

## Middleware ordering

```mermaid
flowchart TD
    R[Incoming Request]
    L[Request Size and Syntax Limits]
    T[Trusted Proxy and Transport Processing]
    C[Credential Extraction]
    V[Credential Validation]
    M[Identity Mapping]
    A[Account Policy]
    Z[Authorization]
    H[Handler]
    U[Audit and Response]

    R --> L --> T --> C --> V --> M --> A --> Z --> H --> U
```

Do not run sensitive business handlers before authorization middleware.

---

# 16.59 Security-context object

```python
from dataclasses import dataclass
from datetime import datetime


@dataclass(frozen=True)
class SecurityContext:
    subject_id: str
    actor_id: str
    tenant_id: str | None
    issuer: str
    credential_type: str
    audience: str
    scopes: frozenset[str]
    roles: frozenset[str]
    auth_time: datetime | None
    authentication_methods: tuple[str, ...]
    assurance_level: str | None
    token_id_hash: str | None
    session_id_hash: str | None
```

Make security context immutable after validation.

Do not let route handlers rewrite:

```text
subject
tenant
issuer
assurance
```

---

# 16.60 Credential extraction

Pseudocode:

```python
def extract_credential(request):
    authorization = request.headers.get("Authorization")
    session_cookie = request.cookies.get("__Host-session")

    if authorization and session_cookie:
        raise AuthenticationError("multiple_primary_credentials")

    if authorization:
        scheme, _, value = authorization.partition(" ")

        if scheme.lower() != "bearer" or not value:
            raise AuthenticationError("unsupported_authorization_scheme")

        if len(value) > 16_384:
            raise AuthenticationError("credential_too_large")

        return BearerCredential(value=value)

    if session_cookie:
        return SessionCredential(value=session_cookie)

    return AnonymousCredential()
```

Policy must define whether multiple credentials are:

* Rejected
* Ordered
* Combined
* Restricted by endpoint

Silent precedence can create credential-confusion attacks.

---

# 16.61 JWT validation outline

```python
def validate_access_token(token: str, config, key_cache):
    header = parse_header_with_limits(token)

    if header.algorithm not in config.allowed_algorithms:
        raise AuthenticationError("algorithm_not_allowed")

    key = key_cache.get(
        issuer=config.expected_issuer,
        key_id=header.key_id,
        algorithm=header.algorithm,
    )

    claims = verify_signature_and_parse_claims(
        token=token,
        key=key,
        allowed_algorithm=header.algorithm,
    )

    require_exact_issuer(claims, config.expected_issuer)
    require_token_type(header, claims, config.expected_token_type)
    require_audience(claims, config.expected_audience)
    require_time_validity(claims, config.allowed_clock_skew)
    require_required_claims(claims, config.required_claims)

    return claims
```

Do not:

* Select issuer from untrusted token data without a trusted mapping.
* Fetch arbitrary key URLs from token headers.
* Accept every algorithm supported by the library.
* Skip audience because signature is valid.
* Treat an ID Token as an API access token.

---

# 16.62 Authorization middleware

```python
def authorize_request(
    *,
    context: SecurityContext,
    action: str,
    resource,
    policy_engine,
):
    decision = policy_engine.evaluate(
        subject=context.subject_id,
        actor=context.actor_id,
        tenant=context.tenant_id,
        action=action,
        resource=resource.authorization_attributes(),
        context={
            "assurance_level": context.assurance_level,
            "auth_time": context.auth_time,
            "scopes": sorted(context.scopes),
            "roles": sorted(context.roles),
        },
    )

    if decision.result != "allow":
        raise AccessDenied(
            reason=decision.reason,
            policy_id=decision.policy_id,
            policy_version=decision.policy_version,
        )

    return decision
```

The handler should receive the authorized resource or a decision tied to the same resource revision.

---

# 16.63 Centralized versus local authorization

## Central policy decision

```text
API
→ remote policy service
→ allow or deny
```

Benefits:

* Central policy
* Shared decision logs
* Rapid updates

Costs:

* Network dependency
* Latency
* Availability coupling
* Data disclosure to policy service

---

## Local policy evaluation

```text
Signed policy bundle
→ local evaluator
```

Benefits:

* Low latency
* Better runtime availability
* No network call per request

Costs:

* Policy distribution
* Revocation propagation
* Version consistency
* Larger application TCB

Hybrid designs are common.

---

# 16.64 Error response mapping

External responses:

```text
No acceptable credential:
  401

Valid identity but forbidden:
  403

Resource existence should be concealed:
  404

Step-up required:
  protocol-specific 401, redirect, or challenge response

Rate limited:
  429
```

Internal logs should preserve exact reason.

External errors should not reveal unnecessary account or resource information.

---

# 16.65 Audit event blueprint

```json
{
  "event_type": "authorization_decision",
  "timestamp": "2026-07-31T12:38:40Z",
  "request_id": "req_8192",
  "trace_id": "4bf92f...",
  "subject_id": "u_1048",
  "actor_id": "finance-web",
  "tenant_id": "tenant-42",
  "credential_type": "access_token",
  "action": "report.download",
  "resource_type": "report",
  "resource_id": "report-71",
  "decision": "allow",
  "reason": "department_reader",
  "policy_id": "reports-read",
  "policy_version": "18",
  "authentication_methods": ["password", "totp"],
  "assurance_level": "corporate-mfa"
}
```

Do not include:

* Raw token
* Session cookie
* Password
* Private key
* Complete sensitive report metadata

---

# 16.66 Database enforcement blueprint

Application role:

```sql
CREATE ROLE reports_api_runtime NOLOGIN;
```

Runtime login or dynamic credential receives that role.

Grant only necessary operations:

```sql
GRANT SELECT, INSERT, UPDATE
ON reports, report_exports
TO reports_api_runtime;
```

Avoid granting:

```text
database owner
user administration
schema ownership
unrestricted function execution
```

Exact commands vary by database system.

---

# 16.67 Row-level security concept

Conceptual policy:

```sql
CREATE POLICY tenant_isolation
ON reports
USING (
    tenant_id = current_setting('app.tenant_id')::uuid
);
```

Before queries, the application sets a transaction-local tenant context through a trusted database mechanism.

Security requirements:

* Application role cannot bypass policy.
* Tenant context is set transaction-locally.
* Connection pool clears context.
* Client cannot set arbitrary tenant directly.
* Administrative paths are separate.

---

# 16.68 Connection-pool context safety

Unsafe:

```text
Request A:
  sets tenant-42

Connection returns to pool

Request B:
  forgets to set tenant
  inherits tenant-42
```

Safer:

* Start transaction.
* Set tenant context locally.
* Execute authorized queries.
* Commit or rollback.
* Verify reset.
* Do not use session-wide mutable authorization state when avoidable.

---

# 16.69 Background-job authorization blueprint

Submission:

```text
API authenticates user
→ authorizes job creation
→ creates job record and authorization envelope
```

Execution:

```text
Worker authenticates as worker
→ validates job envelope
→ checks expiry and policy requirements
→ performs idempotent operation
```

Audit preserves:

```text
subject = Alice
actor at submission = finance-web
actor at execution = export-worker
```

---

# 16.70 Job schema

```sql
CREATE TABLE export_jobs (
    job_id                   UUID PRIMARY KEY,
    tenant_id                UUID NOT NULL,
    report_id                UUID NOT NULL,
    subject_user_id          UUID NOT NULL,
    submitted_by_actor       TEXT NOT NULL,
    execution_service        TEXT,
    authorization_reference  TEXT NOT NULL,
    policy_version           TEXT NOT NULL,
    authorized_at            TIMESTAMPTZ NOT NULL,
    authorization_expires_at TIMESTAMPTZ NOT NULL,
    status                   TEXT NOT NULL,
    idempotency_key          TEXT NOT NULL,
    created_at               TIMESTAMPTZ NOT NULL,
    UNIQUE (tenant_id, idempotency_key)
);
```

---

# 16.71 Test Plan Blueprint

A complete security test plan needs positive and negative cases.

---

## Authentication tests

* Correct password
* Incorrect password
* Unknown user
* Empty password
* Oversized input
* Account disabled
* Account expired
* MFA required
* Incorrect MFA
* Replayed MFA response
* Password reset invalidates expected sessions
* Session fixation attempt
* Expired session
* Revoked session
* Stale session generation

---

## OIDC tests

* Correct authorization response
* State mismatch
* Missing state
* Replayed authorization code
* Wrong redirect URI
* PKCE verifier mismatch
* Nonce mismatch
* Wrong issuer
* Wrong audience
* Expired ID Token
* Unknown signing key
* Stale JWKS
* Token from another tenant
* ID Token used as API token

---

## Token tests

* Missing token
* Malformed token
* Disallowed algorithm
* Invalid signature
* Wrong issuer
* Wrong audience
* Expired
* Not yet valid
* Wrong token type
* Missing required claim
* Oversized claim set
* Duplicate JSON key
* Proof-of-possession mismatch
* Replayed request proof

---

## Authorization tests

* Correct role and resource
* Correct role but wrong tenant
* Correct tenant but wrong object owner
* Missing attribute
* Stale group
* Explicit deny
* Policy dependency timeout
* Direct backend bypass
* Search-path leak
* Export-path leak
* Batch-operation partial authorization
* Background-job reauthorization
* Cache-key tenant isolation
* Connection-pool tenant reset

---

## Infrastructure tests

* Expired client certificate
* Wrong SAN
* Unknown CA
* Missing client certificate
* Certificate/key mismatch
* Stale trust bundle
* Workload identity for wrong namespace
* Kubernetes token wrong audience
* Cloud trust policy subject mismatch
* Rotated credential not reloaded
* Long-lived connection after revocation

---

# 16.72 Property-based tenant test

Pseudocode:

```python
@given(
    subject=subjects(),
    resource=resources(),
)
def test_cross_tenant_read_is_denied(subject, resource):
    assume(subject.tenant_id != resource.tenant_id)
    assume(not subject.has_global_cross_tenant_role)

    decision = authorize(
        subject=subject,
        action="report.read",
        resource=resource,
    )

    assert decision == "deny"
```

Generate cases covering:

* Different roles
* Different resource states
* Missing tenant
* Suspended membership
* Global exception
* Nested group membership

---

# 16.73 Integration test environment

Use:

* Isolated identity tenant or realm
* Synthetic users
* Dedicated test issuer
* Separate signing keys
* Test email/SMS sink
* Short token lifetimes
* Resettable policy data
* No production trust relationship

Do not reuse production signing keys in development.

---

# 16.74 Security regression fixtures

Keep test fixtures for previously discovered defects.

Examples:

```text
JWT with duplicate audience claim
SAML response with wrapped assertion
Cross-tenant report ID
Old session schema
Replayed refresh token
Kerberos alias SPN mismatch
LDAP filter-injection username
Proxy-spoofed identity header
```

A fixed vulnerability should become a permanent regression test.

---

# 16.75 Rollout Blueprint

Identity changes require phased rollout.

Example signing-key rollout:

1. Generate key B.
2. Publish public key B.
3. Verify every region can retrieve B.
4. Canary issuer signs a small portion with B.
5. Monitor unknown-key and signature failures.
6. Increase signing percentage.
7. Stop signing with key A.
8. Wait past maximum A-token lifetime and skew.
9. Remove public key A.
10. Destroy or archive private key A.

---

# 16.76 Session-schema rollout

Safe sequence:

```text
1. Deploy readers that understand versions 1 and 2.
2. Verify fleet adoption.
3. Switch writers to version 2.
4. Monitor parse failures.
5. Migrate or expire version 1 sessions.
6. Remove version 1 reader after maximum lifetime.
```

Unsafe:

```text
New writer deployed first
→ old readers reject new sessions
→ intermittent login failures
```

---

# 16.77 Authorization-policy rollout

Recommended stages:

```text
Develop
→ static analysis
→ unit tests
→ property tests
→ replay historical decisions
→ shadow production
→ review differences
→ canary enforcement
→ full enforcement
→ propagation verification
```

Rollback must preserve knowledge of:

* Previous policy version
* Schema compatibility
* Data migrations
* Cache invalidation

---

# 16.78 Claim-mapping rollout

A claim-mapping change can create or remove privilege.

Example:

```text
Old:
  external group finance-admins
  → tenant billing-admin

New:
  external group finance-admins
  → global platform-admin
```

This should require:

* Explicit review
* Test identities
* Difference report
* Tenant scoping
* Approval
* Audit
* Rollback

---

# 16.79 Observability during rollout

Monitor:

```text
login success rate
authentication failure stage
session creation failures
unknown key IDs
issuer mismatches
audience mismatches
authorization deny rate
policy difference rate
cross-tenant denial rate
credential renewal failures
certificate expiry margins
```

Segment by:

* Region
* Replica version
* Tenant
* Client type
* Identity provider
* Credential type

Avoid exposing individual user IDs in high-cardinality metrics.

---

# 16.80 Operational Runbook: Authentication Failure Spike

1. Confirm scope and start time.
2. Separate unknown-user, password, MFA, token, and session failures.
3. Check recent deployments and configuration changes.
4. Check IdP, directory, KDC, session store, and key service health.
5. Compare regions and replicas.
6. Check clock synchronization.
7. Inspect signing-key and trust-bundle changes.
8. Check attack patterns such as spraying or credential stuffing.
9. Preserve logs and request identifiers.
10. Apply rate limits or rollback without disabling core validation.
11. Verify recovery with synthetic accounts.
12. Continue monitoring after apparent recovery.

---

# 16.81 Operational Runbook: Authorization Denial Spike

1. Identify policy ID and version.
2. Determine affected action, resource type, tenant, and region.
3. Compare old and new policy decisions.
4. Check missing or stale attributes.
5. Inspect directory and membership updates.
6. Check policy-service health and fallback mode.
7. Verify cache invalidation.
8. Confirm database and downstream enforcement.
9. Roll back policy only if safe and schema-compatible.
10. Verify no unauthorized actions occurred during the change.

---

# 16.82 Operational Runbook: Credential Compromise

1. Identify credential type.
2. Identify subject, actor, audience, and authority.
3. Determine earliest possible compromise time.
4. Stop new issuance where necessary.
5. Revoke credential and related family.
6. Terminate active sessions or connections as required.
7. Rotate underlying keys or secrets.
8. Search usage logs.
9. Rotate downstream credentials obtained through it.
10. Fix the exposure path.
11. Notify affected users or operators according to policy.
12. Validate that old credentials are rejected everywhere.

---

# 16.83 Common Implementation Anti-Patterns

## Role-only authorization

```text
if role == admin:
  allow
```

Missing:

* Tenant
* Resource
* State
* Actor
* Authentication assurance
* Explicit global scope

---

## Client-side enforcement

```text
Hide button
→ assume operation protected
```

The API remains callable.

---

## Email as permanent identity

Email can be renamed, reassigned, or asserted by another issuer.

---

## Static service key in source code

Rotation and incident containment become difficult.

---

## One access token for every API

Audience and least privilege are lost.

---

## Trusting any token from a valid key

Issuer, audience, type, time, and context still require validation.

---

## Generic identity proxy headers

Backend accepts identity headers from direct clients.

---

## Admin role in browser storage

Client-controlled state becomes authority.

---

## `403` after side effect

Authorization occurs too late.

---

## Fail-open on policy exception

Parser or dependency errors become grants.

---

## Shared tenant cache keys

Data or decisions cross tenant boundaries.

---

## Long-lived refresh token without family tracking

Theft and replay are difficult to detect.

---

## Certificate validation without identity matching

Any certificate from the CA may impersonate any service.

---

## Password reset without session revocation policy

Attacker may retain existing stolen session.

---

## Group removal without token/session invalidation

Access persists until all cached representations expire.

---

# 16.84 Implementation Review Checklist

## Identity

* Stable subject keys are issuer-scoped.
* Usernames and email are mutable attributes.
* Human and workload identities are distinct.
* Actor and subject are preserved.
* Tenant membership is explicit.
* Account-linking flow is controlled.

## Credentials

* Credential type matches client capability.
* Long-lived secrets are minimized.
* Browser tokens are not exposed unnecessarily.
* Credentials have audience and expiry.
* Rotation and revocation exist.
* Credential values are not logged.

## Sessions

* Session IDs are random and opaque.
* Stored identifiers are protected.
* Session rotates after authentication changes.
* Idle and absolute expiry exist.
* Logout revokes server state.
* Global and per-session revocation exist.

## Authorization

* Default deny.
* Every protected route has enforcement.
* Object and tenant checks exist.
* Policy inputs are trusted and typed.
* Batch and asynchronous paths are included.
* Cache keys include all decision dimensions.
* High-risk decisions preserve provenance.

## Infrastructure

* TLS verifies expected peer identity.
* Workloads have distinct identities.
* Kubernetes tokens use restricted audience.
* Cloud federation subjects are exact.
* Service credentials rotate.
* Long-lived connections have reauthorization policy.

## Operations

* Logs contain stage and reason.
* Metrics show latency and denial rate.
* Key and policy rollout is phased.
* Incident runbooks are tested.
* Break-glass access is controlled.
* Production and non-production trust are separate.

---

# 16.85 Questions: Implementation and Architecture Review

1. Which client types can safely hold a confidential client secret?
2. Why does architecture selection begin with the client and trust boundary?
3. When is a server-side session preferable to a self-contained token?
4. When is an offline-verifiable access token preferable to introspection?
5. Why should a downstream API enforce authorization even behind a BFF?
6. How can database constraints provide independent tenant isolation?
7. Why must connection pools reset authorization context?
8. Which claims should be immutable inside a request security context?
9. Why should middleware reject ambiguous multiple primary credentials?
10. What is the danger of a fully generic authenticated reverse proxy?
11. How can a policy service become a confused deputy?
12. Why should a background worker authenticate as itself while preserving the original user subject?
13. What should an authorization envelope bind to?
14. Why should every fixed security defect become a regression test?
15. Which rollout order prevents old application versions from rejecting new session formats?
16. Why should public verification keys be distributed before new private keys are activated?
17. What metrics reveal a partial key-rotation failure?
18. Why can a policy rollback be unsafe after an input-schema migration?
19. How should a production test identity differ from an administrator account?
20. Which artifacts belong in a credential-compromise investigation?
21. Why should test and production identity systems use separate trust roots?
22. What tenant-isolation property should hold for search and export, not only direct object reads?
23. How can a ServiceAccount token with the wrong audience be abused?
24. What makes an administrative control plane a separate security system?
25. Why should a break-glass exercise include post-use credential rotation?
26. Which security properties are lost when one API token is forwarded to every backend?
27. How can a session-store outage affect a BFF differently from a token-only API?
28. Why should external errors be generic while internal logs remain specific?
29. How can an authorization allow be invalidated by a resource revision change?
30. What end-to-end evidence should prove that report 71 was downloaded by the intended user through the intended workload?

# 17. Hands-On Laboratories and Capstone Exercises

These labs are designed to turn the earlier concepts into observable system behavior.

Use only:

* A disposable virtual machine
* A local development computer
* A dedicated test cluster
* Synthetic identities
* Short-lived test credentials
* Non-production trust roots

Never run experimental authentication commands against production unless the change has been explicitly reviewed and authorized.

---

# 17.1 General Laboratory Safety

Before beginning:

```bash
umask 077
```

### What it does

Sets restrictive default permissions for newly created files.

A typical resulting mode is:

```text
Regular files:
  600

Directories:
  700
```

The exact result also depends on the permissions requested by the creating program.

---

## Create an isolated workspace

```bash
LAB_DIR="$(mktemp -d)"
printf 'Lab directory: %s\n' "${LAB_DIR}"
cd "${LAB_DIR}"
```

Verify:

```bash
stat -c '%A %a %U %G %n' "${LAB_DIR}"
```

Expected form:

```text
drwx------ 700 alice alice /tmp/tmp.example
```

---

## Record tool versions

```bash
python3 --version
openssl version
curl --version
```

For Kubernetes labs:

```bash
kubectl version --client
```

For Linux authorization labs:

```bash
uname -a
id
```

Version information matters because:

* Command flags change.
* Cryptographic defaults change.
* Certificate behavior changes.
* Kubernetes token behavior depends on cluster version.
* Operating-system access-control facilities differ.

---

## Do not use real secrets

Use values such as:

```text
lab-password-DO-NOT-REUSE
```

Never use:

* Corporate password
* Personal password
* Production API key
* Real session cookie
* Production signing key
* Existing SSH private key
* Production certificate authority

---

## Cleanup checklist

At the end of each lab:

* Stop background servers.
* Delete temporary credentials.
* Remove test users.
* Delete test namespaces.
* Clear exported variables.
* Remove packet captures.
* Destroy test ticket caches.
* Remove temporary trust roots.
* Confirm no test process remains.

---

# 17.2 Lab 1: Password Hashing and Verification

## Objective

Observe how a password is transformed into a stored verification record.

The lab demonstrates:

* Random salts
* Memory-hard password hashing
* Parameter storage
* Verification
* Constant-time comparison
* Rehash decisions
* Unknown-user timing equalization

This lab uses Python’s standard-library `scrypt` implementation for portability.

That does not mean every production system should select scrypt automatically. Production password-hashing policy may instead require Argon2id or another approved mechanism.

---

## Create the lab program

```bash
cat > password_lab.py <<'PY'
from __future__ import annotations

import base64
import hashlib
import hmac
import json
import secrets
import time
from dataclasses import asdict, dataclass


@dataclass(frozen=True)
class PasswordRecord:
    algorithm: str
    n: int
    r: int
    p: int
    dklen: int
    salt_b64: str
    hash_b64: str


def b64encode(value: bytes) -> str:
    return base64.urlsafe_b64encode(value).decode("ascii")


def b64decode(value: str) -> bytes:
    return base64.urlsafe_b64decode(value.encode("ascii"))


def hash_password(
    password: str,
    *,
    n: int = 2**15,
    r: int = 8,
    p: int = 1,
    dklen: int = 32,
) -> PasswordRecord:
    if not password:
        raise ValueError("Password must not be empty")

    salt = secrets.token_bytes(16)

    derived = hashlib.scrypt(
        password.encode("utf-8"),
        salt=salt,
        n=n,
        r=r,
        p=p,
        dklen=dklen,
    )

    return PasswordRecord(
        algorithm="scrypt",
        n=n,
        r=r,
        p=p,
        dklen=dklen,
        salt_b64=b64encode(salt),
        hash_b64=b64encode(derived),
    )


def verify_password(password: str, record: PasswordRecord) -> bool:
    if record.algorithm != "scrypt":
        raise ValueError("Unsupported password algorithm")

    candidate = hashlib.scrypt(
        password.encode("utf-8"),
        salt=b64decode(record.salt_b64),
        n=record.n,
        r=record.r,
        p=record.p,
        dklen=record.dklen,
    )

    expected = b64decode(record.hash_b64)
    return hmac.compare_digest(candidate, expected)


def needs_rehash(
    record: PasswordRecord,
    *,
    required_n: int = 2**15,
    required_r: int = 8,
    required_p: int = 1,
    required_dklen: int = 32,
) -> bool:
    return (
        record.algorithm != "scrypt"
        or record.n < required_n
        or record.r < required_r
        or record.p < required_p
        or record.dklen < required_dklen
    )


def benchmark(password: str, record: PasswordRecord, rounds: int = 5) -> None:
    samples_ms: list[float] = []

    for _ in range(rounds):
        started = time.perf_counter()
        verify_password(password, record)
        elapsed_ms = (time.perf_counter() - started) * 1000
        samples_ms.append(elapsed_ms)

    samples_ms.sort()
    median = samples_ms[len(samples_ms) // 2]

    print(f"verification_samples_ms={samples_ms}")
    print(f"verification_median_ms={median:.1f}")


def main() -> None:
    password = "lab-password-DO-NOT-REUSE"

    record = hash_password(password)

    print("Stored record:")
    print(json.dumps(asdict(record), indent=2))

    print()
    print("Correct password:", verify_password(password, record))
    print("Wrong password:", verify_password("wrong-password", record))
    print("Needs rehash:", needs_rehash(record))

    print()
    benchmark(password, record)


if __name__ == "__main__":
    main()
PY
```

Run:

```bash
python3 password_lab.py
```

---

## Illustrative output

```text
Stored record:
{
  "algorithm": "scrypt",
  "n": 32768,
  "r": 8,
  "p": 1,
  "dklen": 32,
  "salt_b64": "RANDOM_SALT",
  "hash_b64": "DERIVED_VALUE"
}

Correct password: True
Wrong password: False
Needs rehash: False

verification_samples_ms=[...]
verification_median_ms=...
```

Your values and timing will differ.

---

## Run it twice

```bash
python3 password_lab.py
python3 password_lab.py
```

Observe:

```text
Same password
+
different random salt
→ different stored hash
```

This prevents identical passwords from producing identical database values.

---

## Change one parameter

Edit:

```python
n=2**15
```

to a higher supported value such as:

```python
n=2**16
```

Then rerun.

Observe:

* Verification time
* Memory requirements
* Whether the runtime rejects the parameter because of local memory limits
* Whether concurrent operation would remain practical

Do not select production parameters from a single laptop timing.

---

## Add unknown-user equalization

A login system should avoid:

```text
Known user:
  expensive password verification

Unknown user:
  immediate rejection
```

Create a dummy record at application startup:

```python
DUMMY_RECORD = hash_password(
    "random-startup-value-" + secrets.token_urlsafe(32)
)
```

Login logic:

```python
record = user_repository.find_password_record(username)

if record is None:
    verify_password(submitted_password, DUMMY_RECORD)
    return generic_failure()

if not verify_password(submitted_password, record):
    return generic_failure()
```

This reduces timing differences.

It does not guarantee perfectly identical response time across the entire stack.

---

## Questions

1. Why is the salt stored with the password record?
2. Why does each account need a different random salt?
3. Why is a salt not required to be secret?
4. Which parameters must be stored so the password can be verified later?
5. Why is `hmac.compare_digest` preferable to an ordinary equality check for secret-derived values?
6. How does increasing the work factor affect legitimate login capacity?
7. Why should unknown users perform a dummy password calculation?
8. When should a valid password record be rehashed?
9. Why must password length and encoding rules remain stable?
10. What production measurements are needed before selecting parameters?

---

# 17.3 Lab 2: Server-Side Session Lifecycle

## Objective

Create, resolve, rotate, expire, and revoke an opaque server-side session.

The browser-side value will contain no username, role, or tenant.

---

## Create the session program

```bash
cat > session_lab.py <<'PY'
from __future__ import annotations

import hashlib
import secrets
import sqlite3
import time
from dataclasses import dataclass


DATABASE = "sessions.db"


@dataclass(frozen=True)
class SessionContext:
    user_id: str
    generation: int


def connect() -> sqlite3.Connection:
    connection = sqlite3.connect(DATABASE)
    connection.row_factory = sqlite3.Row
    return connection


def initialize() -> None:
    with connect() as database:
        database.execute(
            """
            CREATE TABLE IF NOT EXISTS sessions (
                session_hash TEXT PRIMARY KEY,
                user_id TEXT NOT NULL,
                generation INTEGER NOT NULL,
                created_at INTEGER NOT NULL,
                expires_at INTEGER NOT NULL,
                revoked_at INTEGER
            )
            """
        )


def digest_session(raw_session_id: str) -> str:
    return hashlib.sha256(raw_session_id.encode("ascii")).hexdigest()


def create_session(
    *,
    user_id: str,
    generation: int,
    lifetime_seconds: int,
) -> str:
    raw_session_id = secrets.token_urlsafe(32)
    session_hash = digest_session(raw_session_id)
    now = int(time.time())

    with connect() as database:
        database.execute(
            """
            INSERT INTO sessions (
                session_hash,
                user_id,
                generation,
                created_at,
                expires_at,
                revoked_at
            )
            VALUES (?, ?, ?, ?, ?, NULL)
            """,
            (
                session_hash,
                user_id,
                generation,
                now,
                now + lifetime_seconds,
            ),
        )

    return raw_session_id


def resolve_session(
    raw_session_id: str,
    *,
    expected_generation: int,
) -> SessionContext | None:
    if len(raw_session_id) > 512:
        return None

    session_hash = digest_session(raw_session_id)
    now = int(time.time())

    with connect() as database:
        row = database.execute(
            """
            SELECT
                user_id,
                generation,
                expires_at,
                revoked_at
            FROM sessions
            WHERE session_hash = ?
            """,
            (session_hash,),
        ).fetchone()

    if row is None:
        return None

    if row["revoked_at"] is not None:
        return None

    if now >= row["expires_at"]:
        return None

    if row["generation"] != expected_generation:
        return None

    return SessionContext(
        user_id=row["user_id"],
        generation=row["generation"],
    )


def revoke_session(raw_session_id: str) -> None:
    session_hash = digest_session(raw_session_id)
    now = int(time.time())

    with connect() as database:
        database.execute(
            """
            UPDATE sessions
            SET revoked_at = ?
            WHERE session_hash = ?
              AND revoked_at IS NULL
            """,
            (now, session_hash),
        )


def main() -> None:
    initialize()

    current_generation = 7

    session_id = create_session(
        user_id="u_1048",
        generation=current_generation,
        lifetime_seconds=3,
    )

    print("Created session")
    print("Cookie value is intentionally not printed")

    valid = resolve_session(
        session_id,
        expected_generation=current_generation,
    )
    print("Initial resolution:", valid)

    wrong_generation = resolve_session(
        session_id,
        expected_generation=current_generation + 1,
    )
    print("Generation mismatch:", wrong_generation)

    revoke_session(session_id)

    revoked = resolve_session(
        session_id,
        expected_generation=current_generation,
    )
    print("After revocation:", revoked)

    short_session = create_session(
        user_id="u_1048",
        generation=current_generation,
        lifetime_seconds=1,
    )

    time.sleep(2)

    expired = resolve_session(
        short_session,
        expected_generation=current_generation,
    )
    print("After expiry:", expired)


if __name__ == "__main__":
    main()
PY
```

Run:

```bash
python3 session_lab.py
```

---

## Inspect stored values

```bash
python3 - <<'PY'
import sqlite3

database = sqlite3.connect("sessions.db")

for row in database.execute(
    """
    SELECT
        session_hash,
        user_id,
        generation,
        created_at,
        expires_at,
        revoked_at
    FROM sessions
    """
):
    print(row)
PY
```

Observe:

```text
Database contains:
  SHA-256 hash of session identifier

Database does not contain:
  raw cookie value
```

---

## Test a copied identifier

Temporarily modify the program to print the session value only in this isolated lab.

Use it twice.

Observe:

```text
Original browser and attacker with copied session ID
are indistinguishable to a pure bearer-session system
```

Then remove the print statement and delete the database.

---

## Session rotation exercise

Implement:

```text
Old session
→ create new session
→ revoke old session
→ return new cookie
```

The transition should be atomic enough that:

* The new session is not returned if storage fails.
* The old session is revoked after successful privilege transition.
* Parallel requests do not preserve the old authority indefinitely.

---

## Generation invalidation exercise

Suppose user record contains:

```text
current_session_generation = 8
```

Existing session contains:

```text
generation = 7
```

Resolution fails.

This enables:

```text
Password reset
→ increment generation
→ all generation-7 sessions rejected
```

---

## Cleanup

```bash
rm -f session_lab.py sessions.db
```

---

## Questions

1. Why is the raw session ID absent from the database?
2. What happens if both database and cookie values are stolen?
3. How does a session generation support global logout?
4. Why does session expiry need to be checked even if a cleanup job deletes old rows?
5. What race can occur when rotating a session?
6. Why should privilege elevation rotate the session identifier?
7. Which fields would be needed for idle expiry?
8. How should multiple devices be represented?
9. What should happen to sessions after password recovery?
10. Why does a server-side session depend on session-store availability?

---

# 17.4 Lab 3: JWT Decoding Versus Validation

## Objective

Demonstrate that:

```text
Decode
≠
Verify
≠
Fully validate
```

This lab creates a simple HMAC-signed token only for local education.

Do not implement a production JWT system by copying this script.

Use a mature, reviewed protocol library.

---

## Create the script

```bash
cat > jwt_lab.py <<'PY'
from __future__ import annotations

import base64
import hashlib
import hmac
import json
import secrets
import time
from typing import Any


def b64url_encode(value: bytes) -> str:
    return base64.urlsafe_b64encode(value).rstrip(b"=").decode("ascii")


def b64url_decode(value: str) -> bytes:
    padded = value + "=" * (-len(value) % 4)
    return base64.urlsafe_b64decode(padded.encode("ascii"))


def encode_json(value: dict[str, Any]) -> str:
    compact = json.dumps(
        value,
        separators=(",", ":"),
        sort_keys=True,
    ).encode("utf-8")

    return b64url_encode(compact)


def create_token(
    claims: dict[str, Any],
    *,
    secret: bytes,
) -> str:
    header = {
        "alg": "HS256",
        "typ": "at+jwt",
        "kid": "lab-key-1",
    }

    encoded_header = encode_json(header)
    encoded_claims = encode_json(claims)
    signing_input = f"{encoded_header}.{encoded_claims}".encode("ascii")

    signature = hmac.new(
        secret,
        signing_input,
        hashlib.sha256,
    ).digest()

    return (
        f"{encoded_header}."
        f"{encoded_claims}."
        f"{b64url_encode(signature)}"
    )


def decode_without_verification(token: str) -> tuple[dict, dict]:
    parts = token.split(".")

    if len(parts) != 3:
        raise ValueError("Expected three token segments")

    header = json.loads(b64url_decode(parts[0]))
    claims = json.loads(b64url_decode(parts[1]))

    return header, claims


def validate_token(
    token: str,
    *,
    secret: bytes,
    expected_issuer: str,
    expected_audience: str,
    now: int | None = None,
) -> dict[str, Any]:
    parts = token.split(".")

    if len(parts) != 3:
        raise ValueError("Malformed token")

    header = json.loads(b64url_decode(parts[0]))

    if header.get("alg") != "HS256":
        raise ValueError("Algorithm not allowed")

    if header.get("typ") != "at+jwt":
        raise ValueError("Wrong token type")

    signing_input = f"{parts[0]}.{parts[1]}".encode("ascii")

    expected_signature = hmac.new(
        secret,
        signing_input,
        hashlib.sha256,
    ).digest()

    presented_signature = b64url_decode(parts[2])

    if not hmac.compare_digest(
        expected_signature,
        presented_signature,
    ):
        raise ValueError("Invalid signature")

    claims = json.loads(b64url_decode(parts[1]))

    current_time = int(time.time()) if now is None else now

    if claims.get("iss") != expected_issuer:
        raise ValueError("Issuer mismatch")

    audience = claims.get("aud")
    if audience != expected_audience:
        raise ValueError("Audience mismatch")

    if not isinstance(claims.get("exp"), int):
        raise ValueError("Missing or invalid expiry")

    if current_time >= claims["exp"]:
        raise ValueError("Token expired")

    if isinstance(claims.get("nbf"), int):
        if current_time < claims["nbf"]:
            raise ValueError("Token not yet valid")

    return claims


def tamper_subject(token: str) -> str:
    header_segment, claim_segment, signature_segment = token.split(".")

    claims = json.loads(b64url_decode(claim_segment))
    claims["sub"] = "attacker"

    tampered_claim_segment = encode_json(claims)

    # The original signature is deliberately reused.
    return (
        f"{header_segment}."
        f"{tampered_claim_segment}."
        f"{signature_segment}"
    )


def main() -> None:
    secret = secrets.token_bytes(32)
    now = int(time.time())

    token = create_token(
        {
            "iss": "https://issuer.auth-lab.invalid",
            "sub": "u_1048",
            "aud": "reports-api",
            "scope": "reports.read",
            "iat": now,
            "exp": now + 300,
        },
        secret=secret,
    )

    header, claims = decode_without_verification(token)

    print("Decoded header:")
    print(json.dumps(header, indent=2))

    print("Decoded claims:")
    print(json.dumps(claims, indent=2))

    verified = validate_token(
        token,
        secret=secret,
        expected_issuer="https://issuer.auth-lab.invalid",
        expected_audience="reports-api",
    )

    print("Validated subject:", verified["sub"])

    tampered = tamper_subject(token)

    _, tampered_claims = decode_without_verification(tampered)
    print("Decoded tampered subject:", tampered_claims["sub"])

    try:
        validate_token(
            tampered,
            secret=secret,
            expected_issuer="https://issuer.auth-lab.invalid",
            expected_audience="reports-api",
        )
    except ValueError as error:
        print("Tampered validation result:", error)

    try:
        validate_token(
            token,
            secret=secret,
            expected_issuer="https://issuer.auth-lab.invalid",
            expected_audience="payments-api",
        )
    except ValueError as error:
        print("Wrong-audience result:", error)


if __name__ == "__main__":
    main()
PY
```

Run:

```bash
python3 jwt_lab.py
```

---

## Expected observations

The tampered token can still be decoded:

```text
sub = attacker
```

But validation fails:

```text
Invalid signature
```

The original validly signed token also fails when checked by the wrong API:

```text
Audience mismatch
```

---

## Additional exercises

### Expired token

Change:

```python
"exp": now + 300
```

to:

```python
"exp": now - 1
```

Observe:

```text
Token expired
```

### Wrong issuer

Validate using:

```python
expected_issuer="https://other-issuer.invalid"
```

Observe:

```text
Issuer mismatch
```

### Wrong token type

Change header:

```python
"typ": "JWT"
```

while validator expects:

```text
at+jwt
```

Observe explicit rejection.

---

## Production differences

A production asymmetric token system typically uses:

* Authorization-server private signing key
* Resource-server public verification key
* Trusted issuer configuration
* JWKS rotation
* Algorithm allowlist
* Token-type checks
* Audience checks
* Claim schema validation
* Clock-skew policy
* Key-fetch rate limits

Do not reuse one HMAC secret across unrelated services.

---

## Cleanup

```bash
rm -f jwt_lab.py
```

---

## Questions

1. Why can a modified token still be decoded?
2. Which step detects the modification?
3. Why must the resource server validate audience?
4. Why is signature validation insufficient without issuer validation?
5. Why should token type be checked?
6. What happens if the verifier accepts every algorithm named by the token?
7. Why should resource servers not fetch arbitrary key URLs from headers?
8. How can an unknown key ID cause a denial-of-service problem?
9. Why should access and identity tokens have distinct validation profiles?
10. Which claims should be required for the application’s security model?

---

# 17.5 Lab 4: Local Mutual TLS

## Objective

Build a temporary certificate authority, issue a server certificate and client certificate, run a server requiring client authentication, and test successful and failed connections.

The generated private keys are unencrypted for laboratory automation.

Never use these keys or this root CA outside the temporary lab.

---

## Create a protected directory

```bash
umask 077
MTLS_LAB="$(mktemp -d)"
cd "${MTLS_LAB}"
printf 'mTLS lab: %s\n' "${MTLS_LAB}"
```

---

## Create a temporary root CA

```bash
openssl req \
  -x509 \
  -newkey rsa:3072 \
  -sha256 \
  -days 2 \
  -nodes \
  -keyout ca.key \
  -out ca.crt \
  -subj '/CN=Authentication Lab Root CA'
```

### Important flags

* `-x509`: create a self-signed certificate
* `-newkey rsa:3072`: generate a new RSA key
* `-sha256`: use SHA-256 in certificate signing
* `-days 2`: short validity for lab
* `-nodes`: do not encrypt private key
* `-keyout`: private-key output
* `-out`: certificate output
* `-subj`: noninteractive subject

`-nodes` is convenient but means anyone who reads `ca.key` can use it.

---

## Inspect the CA

```bash
openssl x509 \
  -in ca.crt \
  -noout \
  -subject \
  -issuer \
  -dates \
  -ext basicConstraints
```

Expected relationship:

```text
subject = issuer
```

because this is a self-signed root.

Expected constraint:

```text
CA:TRUE
```

---

## Create the server key and request

```bash
openssl req \
  -new \
  -newkey rsa:3072 \
  -sha256 \
  -nodes \
  -keyout server.key \
  -out server.csr \
  -subj '/CN=localhost'
```

Create extensions:

```bash
cat > server.ext <<'EOF'
basicConstraints=critical,CA:FALSE
keyUsage=critical,digitalSignature,keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName=DNS:localhost,IP:127.0.0.1
EOF
```

Sign:

```bash
openssl x509 \
  -req \
  -in server.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out server.crt \
  -days 2 \
  -sha256 \
  -extfile server.ext
```

---

## Create the client key and request

```bash
openssl req \
  -new \
  -newkey rsa:3072 \
  -sha256 \
  -nodes \
  -keyout client.key \
  -out client.csr \
  -subj '/CN=orders-service'
```

Create extensions:

```bash
cat > client.ext <<'EOF'
basicConstraints=critical,CA:FALSE
keyUsage=critical,digitalSignature
extendedKeyUsage=clientAuth
subjectAltName=URI:spiffe://auth-lab.invalid/service/orders
EOF
```

Sign:

```bash
openssl x509 \
  -req \
  -in client.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAserial ca.srl \
  -out client.crt \
  -days 2 \
  -sha256 \
  -extfile client.ext
```

---

## Inspect identities

```bash
openssl x509 \
  -in server.crt \
  -noout \
  -subject \
  -issuer \
  -ext subjectAltName \
  -ext extendedKeyUsage
```

```bash
openssl x509 \
  -in client.crt \
  -noout \
  -subject \
  -issuer \
  -ext subjectAltName \
  -ext extendedKeyUsage
```

---

## Verify certificates

Server:

```bash
openssl verify \
  -CAfile ca.crt \
  -purpose sslserver \
  -verify_hostname localhost \
  server.crt
```

Client:

```bash
openssl verify \
  -CAfile ca.crt \
  -purpose sslclient \
  client.crt
```

Expected:

```text
server.crt: OK
client.crt: OK
```

---

## Start an mTLS server

In one terminal:

```bash
cd "${MTLS_LAB}"

openssl s_server \
  -accept 8443 \
  -cert server.crt \
  -key server.key \
  -CAfile ca.crt \
  -Verify 1 \
  -www
```

### Important options

* `-accept 8443`: listen on local TCP port 8443
* `-cert`: server certificate
* `-key`: server private key
* `-CAfile`: trusted client CA
* `-Verify 1`: require and verify a client certificate with configured depth
* `-www`: return a simple diagnostic HTTP response

Keep the terminal open.

---

## Test without a client certificate

In another terminal:

```bash
curl -i \
  --cacert "${MTLS_LAB}/ca.crt" \
  https://localhost:8443/
```

Expected result:

```text
TLS handshake fails
```

The exact TLS alert depends on OpenSSL and curl versions.

---

## Test with a client certificate

```bash
curl -i \
  --cacert "${MTLS_LAB}/ca.crt" \
  --cert "${MTLS_LAB}/client.crt" \
  --key "${MTLS_LAB}/client.key" \
  https://localhost:8443/
```

Expected:

```http
HTTP/1.0 200 ok
```

The server diagnostic page should show TLS and certificate information.

---

## Test hostname mismatch

```bash
curl -i \
  --resolve wrong-name.invalid:8443:127.0.0.1 \
  --cacert "${MTLS_LAB}/ca.crt" \
  --cert "${MTLS_LAB}/client.crt" \
  --key "${MTLS_LAB}/client.key" \
  https://wrong-name.invalid:8443/
```

Expected:

```text
certificate name mismatch
```

The certificate is trusted but identifies `localhost`, not `wrong-name.invalid`.

---

## Test an untrusted client certificate

Create a second unrelated CA:

```bash
openssl req \
  -x509 \
  -newkey rsa:3072 \
  -sha256 \
  -days 2 \
  -nodes \
  -keyout attacker-ca.key \
  -out attacker-ca.crt \
  -subj '/CN=Untrusted Lab CA'
```

Create and sign another client certificate under that CA.

The server should reject it because it trusts only `ca.crt`.

This demonstrates:

```text
Valid certificate signature
does not imply
trusted issuer
```

---

## Stop and clean up

Stop `openssl s_server` with:

```text
Ctrl+C
```

Then:

```bash
cd /
rm -rf "${MTLS_LAB}"
unset MTLS_LAB
```

Never install the lab root into the system trust store.

---

## Questions

1. Which private key signs the server certificate?
2. Which private key proves the client identity during TLS?
3. Why does the client need `ca.crt`?
4. Why does the server need `ca.crt`?
5. Why does the server certificate contain `serverAuth`?
6. Why does the client certificate contain `clientAuth`?
7. Why does a trusted chain not fix a hostname mismatch?
8. What would happen if `client.key` were stolen?
9. Why should a service authorize the URI SAN after TLS succeeds?
10. What changes when TLS terminates at a reverse proxy?

---

# 17.6 Lab 5: Linux Process and File Authorization

## Objective

Observe how:

* User identity
* Group identity
* Directory traversal
* File modes
* ACLs
* Service execution identity

combine to determine access.

Use a disposable Linux virtual machine.

---

## Create a service account

```bash
sudo useradd \
  --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  authlab-service
```

Some systems use another path for `nologin`.

Check:

```bash
getent passwd authlab-service
```

---

## Create protected data

```bash
sudo install \
  -d \
  -o root \
  -g authlab-service \
  -m 0750 \
  /srv/authlab
```

Create file:

```bash
printf 'classified lab report\n' |
sudo tee /srv/authlab/report.txt >/dev/null
```

Set ownership and mode:

```bash
sudo chown root:authlab-service /srv/authlab/report.txt
sudo chmod 0640 /srv/authlab/report.txt
```

Inspect:

```bash
namei -l /srv/authlab/report.txt
```

```bash
stat /srv/authlab/report.txt
```

---

## Test as the service account

```bash
sudo -u authlab-service \
  -- cat /srv/authlab/report.txt
```

Expected:

```text
classified lab report
```

The service account can:

* Traverse `/srv/authlab` through group execute permission
* Read `report.txt` through group read permission

---

## Remove directory traversal

```bash
sudo chmod 0740 /srv/authlab
```

Retest:

```bash
sudo -u authlab-service \
  -- cat /srv/authlab/report.txt
```

Expected:

```text
Permission denied
```

Even though the file itself remains group-readable, the service cannot traverse the parent directory.

Restore:

```bash
sudo chmod 0750 /srv/authlab
```

---

## Remove file read permission

```bash
sudo chmod 0600 /srv/authlab/report.txt
```

Retest.

Expected denial.

Restore:

```bash
sudo chmod 0640 /srv/authlab/report.txt
```

---

## Add an ACL for another user

Choose a disposable ordinary test user or create one:

```bash
sudo useradd \
  --create-home \
  --shell /bin/bash \
  authlab-reader
```

Grant access:

```bash
sudo setfacl \
  -m u:authlab-reader:rx \
  /srv/authlab
```

```bash
sudo setfacl \
  -m u:authlab-reader:r \
  /srv/authlab/report.txt
```

Inspect:

```bash
getfacl /srv/authlab
getfacl /srv/authlab/report.txt
```

Test:

```bash
sudo -u authlab-reader \
  -- cat /srv/authlab/report.txt
```

---

## Remove the ACL

```bash
sudo setfacl \
  -x u:authlab-reader \
  /srv/authlab/report.txt
```

```bash
sudo setfacl \
  -x u:authlab-reader \
  /srv/authlab
```

Retest.

---

## Trace the denied access

```bash
sudo -u authlab-reader \
  -- strace \
  -e trace=openat \
  cat /srv/authlab/report.txt
```

Look for:

```text
EACCES
```

This shows the kernel-level denial.

---

## Inspect service capabilities

```bash
sudo -u authlab-service \
  -- sh -c 'grep -E "^(Uid|Gid|Groups|Cap)" /proc/self/status'
```

Observe:

* Real UID
* Effective UID
* Groups
* Capability sets

Ordinary filesystem access normally follows effective credentials plus relevant security controls.

---

## Cleanup

```bash
sudo rm -rf /srv/authlab
sudo userdel authlab-reader
sudo userdel authlab-service
```

Check that no files remain:

```bash
test ! -e /srv/authlab && echo "cleanup complete"
```

---

## Questions

1. Why did group read permission fail when directory execute permission was removed?
2. What does execute permission mean on a directory?
3. How can an ACL grant access beyond the basic owner/group/other mode bits?
4. Why should the service run under a dedicated account?
5. What would change if the process ran as root?
6. Which syscall returned `EACCES`?
7. How can SELinux or AppArmor deny an operation that mode bits allow?
8. Why should the application not solve a permission problem by running as root?
9. How can supplementary groups change access?
10. What should be logged when a service cannot read a required file?

---

# 17.7 Lab 6: Multi-Tenant Object Authorization

## Objective

Create a small authorization model and test the invariant:

```text
A user cannot read another tenant's report
unless explicitly granted global authority
```

---

## Create the test program

```bash
cat > tenant_auth_lab.py <<'PY'
from __future__ import annotations

from dataclasses import dataclass
from itertools import product


@dataclass(frozen=True)
class Subject:
    user_id: str
    tenant_ids: frozenset[str]
    global_roles: frozenset[str]


@dataclass(frozen=True)
class Report:
    report_id: str
    tenant_id: str
    owner_user_id: str
    classification: str


def can_read_report(subject: Subject, report: Report) -> bool:
    if "platform-security-admin" in subject.global_roles:
        return True

    if report.tenant_id not in subject.tenant_ids:
        return False

    if report.owner_user_id == subject.user_id:
        return True

    if report.classification == "tenant-public":
        return True

    return False


def main() -> None:
    subjects = [
        Subject(
            user_id="alice",
            tenant_ids=frozenset({"tenant-a"}),
            global_roles=frozenset(),
        ),
        Subject(
            user_id="bob",
            tenant_ids=frozenset({"tenant-b"}),
            global_roles=frozenset(),
        ),
        Subject(
            user_id="security-admin",
            tenant_ids=frozenset(),
            global_roles=frozenset({"platform-security-admin"}),
        ),
    ]

    reports = [
        Report(
            report_id="report-a-private",
            tenant_id="tenant-a",
            owner_user_id="alice",
            classification="private",
        ),
        Report(
            report_id="report-a-public",
            tenant_id="tenant-a",
            owner_user_id="other-a-user",
            classification="tenant-public",
        ),
        Report(
            report_id="report-b-private",
            tenant_id="tenant-b",
            owner_user_id="bob",
            classification="private",
        ),
    ]

    for subject, report in product(subjects, reports):
        decision = can_read_report(subject, report)

        print(
            f"subject={subject.user_id:14} "
            f"report={report.report_id:18} "
            f"tenant={report.tenant_id:8} "
            f"decision={'allow' if decision else 'deny'}"
        )

    # Security property:
    for subject, report in product(subjects, reports):
        if "platform-security-admin" in subject.global_roles:
            continue

        if report.tenant_id not in subject.tenant_ids:
            assert not can_read_report(subject, report), (
                f"Cross-tenant violation: "
                f"{subject.user_id} could read {report.report_id}"
            )

    print("Cross-tenant isolation property passed")


if __name__ == "__main__":
    main()
PY
```

Run:

```bash
python3 tenant_auth_lab.py
```

---

## Introduce a defect

Change:

```python
if report.tenant_id not in subject.tenant_ids:
    return False
```

to:

```python
if report.tenant_id in subject.tenant_ids:
    return False
```

Rerun.

The property assertion should fail.

This is a small form of mutation testing.

---

## Add an unsafe cache

Conceptual cache:

```python
decision_cache[(subject.user_id, "report.read")] = decision
```

This omits:

* Tenant
* Resource
* Classification
* Policy version

A previous allow can be reused for the wrong report.

A safer key includes all relevant dimensions:

```python
decision_cache[
    (
        subject.user_id,
        report.tenant_id,
        report.report_id,
        "report.read",
        POLICY_VERSION,
    )
] = decision
```

---

## Add resource versioning

Extend `Report`:

```python
revision: int
```

Bind decisions to:

```text
report ID
+
revision
```

If ownership or tenant changes, the old decision must not authorize the new state.

---

## Add actor and subject

Suppose:

```text
subject:
  Alice

actor:
  export-worker
```

The worker must be authorized:

* To execute export jobs
* For the correct tenant
* On behalf of Alice
* Within the authorization-envelope lifetime

Do not replace Alice with the worker identity.

---

## Cleanup

```bash
rm -f tenant_auth_lab.py
```

---

## Questions

1. Why is tenant membership checked before resource classification?
2. Why is a globally privileged role modeled separately?
3. What did mutation testing reveal?
4. Which fields belong in a decision-cache key?
5. Why should a decision be bound to a resource revision?
6. How should a disabled tenant affect the result?
7. How would nested groups change the test model?
8. Why should search results use the same authorization semantics?
9. How should actor and subject be represented in background jobs?
10. Which property tests would cover writes and exports?

---

# 17.8 Lab 7: Kubernetes ServiceAccount and RBAC

## Objective

Create a namespace, ServiceAccount, Role, and RoleBinding, then verify the resulting permissions.

Use a dedicated non-production cluster.

---

## Create namespace

```bash
kubectl create namespace auth-lab
```

---

## Create resources

```bash
cat > auth-lab-rbac.yaml <<'YAML'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: config-reader
  namespace: auth-lab
automountServiceAccountToken: false
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: configmap-reader
  namespace: auth-lab
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: config-reader-binding
  namespace: auth-lab
subjects:
  - kind: ServiceAccount
    name: config-reader
    namespace: auth-lab
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: configmap-reader
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: lab-settings
  namespace: auth-lab
data:
  mode: test
YAML
```

Apply:

```bash
kubectl apply -f auth-lab-rbac.yaml
```

---

## Check allowed action

```bash
kubectl auth can-i \
  get configmaps \
  --namespace auth-lab \
  --as=system:serviceaccount:auth-lab:config-reader
```

Expected:

```text
yes
```

---

## Check denied action

```bash
kubectl auth can-i \
  delete configmaps \
  --namespace auth-lab \
  --as=system:serviceaccount:auth-lab:config-reader
```

Expected:

```text
no
```

---

## Check another namespace

```bash
kubectl auth can-i \
  get configmaps \
  --namespace default \
  --as=system:serviceaccount:auth-lab:config-reader
```

Expected:

```text
no
```

The RoleBinding grants namespace-local authority.

---

## List effective permissions

```bash
kubectl auth can-i \
  --list \
  --namespace auth-lab \
  --as=system:serviceaccount:auth-lab:config-reader
```

Review whether the result contains unexpected permissions.

---

## Request a bounded token

```bash
TOKEN="$(
  kubectl create token config-reader \
    --namespace auth-lab \
    --audience auth-lab-api \
    --duration 10m
)"
```

Do not print the token.

Inspect its length:

```bash
printf 'token_length=%s\n' "${#TOKEN}"
```

Decode claims locally without printing the signature:

```bash
python3 - "${TOKEN}" <<'PY'
import base64
import json
import sys

token = sys.argv[1]
parts = token.split(".")

if len(parts) != 3:
    raise SystemExit("Not a compact three-part token")

payload = parts[1] + "=" * (-len(parts[1]) % 4)
claims = json.loads(base64.urlsafe_b64decode(payload))

selected = {
    key: claims.get(key)
    for key in ("iss", "sub", "aud", "iat", "nbf", "exp")
}

print(json.dumps(selected, indent=2))
PY
```

Remember:

```text
Decoded claims
are not independently verified claims
```

The API server or intended verifier must validate the token.

Clear:

```bash
unset TOKEN
```

---

## Remove the RoleBinding

```bash
kubectl delete rolebinding \
  config-reader-binding \
  --namespace auth-lab
```

Retest:

```bash
kubectl auth can-i \
  get configmaps \
  --namespace auth-lab \
  --as=system:serviceaccount:auth-lab:config-reader
```

Expected:

```text
no
```

This demonstrates that:

```text
ServiceAccount authentication identity
is separate from
RBAC permission
```

---

## Cleanup

```bash
kubectl delete namespace auth-lab
rm -f auth-lab-rbac.yaml
```

Wait for deletion:

```bash
kubectl get namespace auth-lab
```

Expected eventually:

```text
NotFound
```

---

## Questions

1. Why does creating a ServiceAccount not grant access by itself?
2. Why did the Role not apply to the `default` namespace?
3. What is the ServiceAccount’s authenticated username?
4. Why should a token audience be restricted?
5. Why is decoding the token not sufficient validation?
6. What changed when the RoleBinding was deleted?
7. Why is `automountServiceAccountToken: false` a useful default?
8. What privilege would `create pods` indirectly provide?
9. Why is `ClusterRoleBinding` more dangerous than a namespace RoleBinding?
10. How would this ServiceAccount federate to a cloud role safely?

---

# 17.9 Lab 8: HMAC Request Signing and Replay Protection

## Objective

Create a simplified signed-request scheme that binds:

* Key identifier
* Timestamp
* Nonce
* HTTP method
* Request path
* Body digest

This is an educational design.

Do not invent a proprietary production signing protocol unless standard mechanisms cannot meet the requirement and the design receives expert review.

---

## Create the script

```bash
cat > request_signing_lab.py <<'PY'
from __future__ import annotations

import hashlib
import hmac
import secrets
import time
from dataclasses import dataclass


ALLOWED_CLOCK_SKEW_SECONDS = 30


@dataclass(frozen=True)
class SignedRequest:
    key_id: str
    timestamp: int
    nonce: str
    method: str
    path: str
    body: bytes
    signature_hex: str


class ReplayCache:
    def __init__(self) -> None:
        self._entries: dict[tuple[str, str], int] = {}

    def consume(
        self,
        *,
        key_id: str,
        nonce: str,
        expires_at: int,
        now: int,
    ) -> bool:
        expired = [
            key
            for key, expiry in self._entries.items()
            if expiry <= now
        ]

        for key in expired:
            self._entries.pop(key, None)

        replay_key = (key_id, nonce)

        if replay_key in self._entries:
            return False

        self._entries[replay_key] = expires_at
        return True


def body_digest(body: bytes) -> str:
    return hashlib.sha256(body).hexdigest()


def canonical_request(
    *,
    key_id: str,
    timestamp: int,
    nonce: str,
    method: str,
    path: str,
    body: bytes,
) -> bytes:
    if "\n" in path or "\r" in path:
        raise ValueError("Invalid path")

    lines = [
        "AUTH-LAB-HMAC-V1",
        key_id,
        str(timestamp),
        nonce,
        method.upper(),
        path,
        body_digest(body),
    ]

    return "\n".join(lines).encode("utf-8")


def sign_request(
    *,
    key_id: str,
    secret: bytes,
    method: str,
    path: str,
    body: bytes,
    timestamp: int | None = None,
    nonce: str | None = None,
) -> SignedRequest:
    effective_timestamp = (
        int(time.time()) if timestamp is None else timestamp
    )

    effective_nonce = (
        secrets.token_urlsafe(18) if nonce is None else nonce
    )

    canonical = canonical_request(
        key_id=key_id,
        timestamp=effective_timestamp,
        nonce=effective_nonce,
        method=method,
        path=path,
        body=body,
    )

    signature = hmac.new(
        secret,
        canonical,
        hashlib.sha256,
    ).hexdigest()

    return SignedRequest(
        key_id=key_id,
        timestamp=effective_timestamp,
        nonce=effective_nonce,
        method=method,
        path=path,
        body=body,
        signature_hex=signature,
    )


def verify_request(
    request: SignedRequest,
    *,
    secret_lookup: dict[str, bytes],
    replay_cache: ReplayCache,
    now: int | None = None,
) -> bool:
    current_time = int(time.time()) if now is None else now

    secret = secret_lookup.get(request.key_id)
    if secret is None:
        return False

    if abs(current_time - request.timestamp) > ALLOWED_CLOCK_SKEW_SECONDS:
        return False

    canonical = canonical_request(
        key_id=request.key_id,
        timestamp=request.timestamp,
        nonce=request.nonce,
        method=request.method,
        path=request.path,
        body=request.body,
    )

    expected_signature = hmac.new(
        secret,
        canonical,
        hashlib.sha256,
    ).hexdigest()

    if not hmac.compare_digest(
        expected_signature,
        request.signature_hex,
    ):
        return False

    if not replay_cache.consume(
        key_id=request.key_id,
        nonce=request.nonce,
        expires_at=request.timestamp + ALLOWED_CLOCK_SKEW_SECONDS + 1,
        now=current_time,
    ):
        return False

    return True


def main() -> None:
    key_id = "orders-service-key-1"
    secret = secrets.token_bytes(32)
    secrets_by_id = {key_id: secret}
    replay_cache = ReplayCache()

    request = sign_request(
        key_id=key_id,
        secret=secret,
        method="POST",
        path="/v1/payments",
        body=b'{"order_id":"order-781","amount":4200}',
    )

    print(
        "First verification:",
        verify_request(
            request,
            secret_lookup=secrets_by_id,
            replay_cache=replay_cache,
        ),
    )

    print(
        "Replay verification:",
        verify_request(
            request,
            secret_lookup=secrets_by_id,
            replay_cache=replay_cache,
        ),
    )

    tampered = SignedRequest(
        key_id=request.key_id,
        timestamp=request.timestamp,
        nonce=secrets.token_urlsafe(18),
        method=request.method,
        path=request.path,
        body=b'{"order_id":"order-781","amount":999999}',
        signature_hex=request.signature_hex,
    )

    print(
        "Tampered verification:",
        verify_request(
            tampered,
            secret_lookup=secrets_by_id,
            replay_cache=replay_cache,
        ),
    )


if __name__ == "__main__":
    main()
PY
```

Run:

```bash
python3 request_signing_lab.py
```

Expected:

```text
First verification: True
Replay verification: False
Tampered verification: False
```

---

## Change the method

Change the verified request from:

```text
POST
```

to:

```text
DELETE
```

without recomputing the signature.

Validation should fail.

---

## Change the path

Change:

```text
/v1/payments
```

to:

```text
/v1/refunds
```

Validation should fail.

---

## Distributed replay exercise

Imagine:

```text
Request 1 reaches replica A
Replay reaches replica B
```

If A and B use independent `ReplayCache` objects, both may accept.

A production design needs:

* Shared replay state
* Deterministic routing
* One-time database state
* Protocol-specific distributed replay semantics
* Another acceptable containment strategy

---

## Canonicalization exercise

Consider:

```text
/v1/reports/71
/v1/reports/%37%31
/v1/reports/./71
```

Decide which are equivalent.

The signer, gateway, framework, authorization layer, and handler must agree.

Do not apply one normalization during signing and another during routing.

---

## Cleanup

```bash
rm -f request_signing_lab.py
```

---

## Questions

1. Why is the body digest signed?
2. Why are method and path included?
3. Why is the key ID not itself a secret?
4. Why is the timestamp insufficient without a nonce?
5. Why must the nonce be checked after signature validation?
6. Why should invalid signatures not fill the replay cache?
7. How does distributed load balancing affect replay protection?
8. What canonicalization ambiguities remain in the simplified scheme?
9. How would a reverse proxy changing the external path affect validation?
10. When would mTLS-bound tokens be preferable to an HMAC shared secret?

---

# 17.10 Guided Enterprise Lab: Kerberos and LDAP Diagnostics

This lab requires an existing authorized test realm and directory.

Do not attempt to create test principals in a corporate environment without approval.

---

## Step 1: Identify the current Kerberos cache

```bash
klist
```

Record:

* Default principal
* TGT presence
* Start time
* Expiry
* Realm

Do not copy the credential-cache file.

---

## Step 2: Obtain a test TGT

```bash
kinit test-user@TEST.EXAMPLE
```

Use a synthetic account.

Confirm:

```bash
klist -ef
```

---

## Step 3: Obtain a service ticket

```bash
kvno HTTP/test-app.test.example@TEST.EXAMPLE
```

Record:

* Requested service principal
* Key version
* Failure code if any

---

## Step 4: Trace one request

```bash
KRB5_TRACE=/dev/stderr \
  curl \
  --negotiate \
  -u : \
  https://test-app.test.example/
```

Redact:

* Principal details where sensitive
* Internal hostnames
* Ticket metadata
* HTTP cookies
* Authorization headers

---

## Step 5: Test LDAP TLS

```bash
openssl s_client \
  -connect directory.test.example:389 \
  -starttls ldap \
  -servername directory.test.example \
  -verify_hostname directory.test.example \
  -verify_return_error
```

Confirm:

* Trusted chain
* Expected server identity
* Valid certificate time
* Successful StartTLS

---

## Step 6: Search with a service identity

```bash
ldapsearch \
  -H ldap://directory.test.example \
  -ZZ \
  -x \
  -D 'uid=test-reader,ou=Services,dc=test,dc=example' \
  -W \
  -b 'ou=People,dc=test,dc=example' \
  '(&(objectClass=person)(uid=test-user))' \
  dn uid cn
```

Require exactly one entry.

---

## Step 7: Test SASL GSSAPI

```bash
ldapsearch \
  -H ldap://directory.test.example \
  -ZZ \
  -Y GSSAPI \
  -b 'ou=People,dc=test,dc=example' \
  '(uid=test-user)' \
  dn uid cn
```

This combines:

```text
LDAP
→ SASL
→ GSS-API
→ Kerberos
```

A failure may originate at any layer.

---

## Step 8: Destroy credentials

```bash
kdestroy
```

Confirm:

```bash
klist
```

Expected:

```text
No credentials cache found
```

or equivalent.

---

## Questions

1. What does `kinit` obtain?
2. What additional fact does `kvno` test?
3. Why can `kvno` succeed while the web application still fails?
4. Which component uses the service keytab?
5. Why must the LDAP StartTLS check happen before password Bind testing?
6. What does a successful directory search prove?
7. What does it not prove?
8. Why can SASL GSSAPI fail because of a service-principal alias?
9. How can clock skew affect the lab?
10. Why must the ticket cache be destroyed after testing?

---

# 17.11 Capstone Architecture

## Scenario

Design a multi-tenant financial reporting platform.

Components:

```text
Browser
→ Backend-for-Frontend
→ Reports API
→ Export Worker
→ Object Storage
→ Audit Service

Identity Provider
Session Store
Authorization Service
Kubernetes
Cloud Token Service
```

Requirements:

* Human users authenticate through enterprise OIDC.
* Browser JavaScript must not store refresh tokens.
* Every user belongs to one or more tenants.
* Reports belong to exactly one tenant.
* Exports run asynchronously.
* Export Worker writes only to the correct tenant storage prefix.
* Support agents may impersonate users through a controlled workflow.
* Administrator actions require phishing-resistant step-up.
* Account disablement must block critical new actions within 60 seconds.
* Existing ordinary read sessions may remain usable for up to 10 minutes during an IdP outage.
* No permanent cloud key may be stored in Kubernetes.

---

## Reference architecture

```mermaid
flowchart LR
    U[User Browser]
    B[BFF]
    I[OIDC Provider]
    S[(Session and Token Store)]
    R[Reports API]
    P[Authorization Service]
    D[(Reports Database)]
    Q[Job Queue]
    W[Export Worker]
    STS[Cloud Token Service]
    O[Object Storage]
    A[Audit Service]
    K[Kubernetes Identity]

    U -->|Opaque cookie| B
    B -->|OIDC back channel| I
    B --> S
    B -->|Audience token + mTLS| R
    R --> P
    R --> D
    R --> Q
    Q --> W
    K -->|Projected identity| W
    W -->|Federation exchange| STS
    STS -->|Temporary cloud credential| W
    W --> O
    B --> A
    R --> A
    W --> A
```

---

# 17.12 Capstone Identity Inventory

## Human subject

```text
issuer:
  enterprise IdP

external subject:
  issuer-scoped immutable subject

local user:
  stable application user ID
```

## BFF workload

```text
workload identity:
  spiffe://production.example/ns/finance/sa/finance-bff
```

## Reports API workload

```text
workload identity:
  spiffe://production.example/ns/finance/sa/reports-api
```

## Export Worker workload

```text
Kubernetes identity:
  system:serviceaccount:finance:export-worker

cloud role:
  tenant-export-object-writer
```

## Support impersonation

```text
actor:
  support agent

subject:
  customer user

reason:
  support ticket

expiry:
  short-lived
```

---

# 17.13 Capstone Credential Inventory

| Relationship                  | Credential                                             |
| ----------------------------- | ------------------------------------------------------ |
| Browser to BFF                | Opaque session cookie                                  |
| BFF to IdP                    | Authorization code, refresh token                      |
| BFF to Reports API            | Audience access token plus mTLS                        |
| API to authorization service  | Workload-authenticated policy request                  |
| API to queue                  | Workload credential                                    |
| Worker to cloud token service | Projected ServiceAccount token                         |
| Worker to object storage      | Temporary cloud role credential                        |
| Administrative session        | Dedicated admin session with step-up                   |
| Impersonation                 | Server-side impersonation context or constrained token |

---

# 17.14 Capstone Authorization Rules

## Report read

```text
ALLOW if:
  user has active membership in report tenant
  AND
  user is owner
  OR report classification permits tenant-wide read
  OR tenant role grants read
```

Operator precedence must be explicit:

```text
ALLOW if:
  tenant membership is active
  AND (
    user is owner
    OR classification permits tenant-wide read
    OR tenant role grants read
  )
```

Without parentheses, an implementation may accidentally allow a role from another tenant.

---

## Report export

```text
ALLOW if:
  report.read is allowed
  AND export role exists
  AND report is not legally held against export
  AND requested row count is within limit
  AND recent MFA exists for restricted classification
```

---

## Object-storage write

```text
ALLOW if:
  workload = export-worker
  AND temporary role is valid
  AND object prefix matches tenant
  AND object name matches authorized job
  AND operation is write-only
```

---

## Impersonation

```text
ALLOW if:
  actor has support-impersonate permission
  AND approved ticket exists
  AND subject belongs to permitted tenant
  AND session uses required MFA
  AND operation is not prohibited during impersonation
  AND impersonation context has not expired
```

Prohibited actions may include:

* Change password
* Add MFA factor
* Change payment destination
* Grant administrator role
* Generate API key
* Delete audit history

---

# 17.15 Capstone Session Model

Session record:

```json
{
  "session_hash": "sha256:...",
  "user_id": "u_1048",
  "selected_tenant": "tenant-42",
  "auth_time": "2026-07-31T12:00:00Z",
  "auth_methods": ["password", "webauthn"],
  "assurance_level": "phishing-resistant",
  "idle_expires_at": "2026-07-31T12:30:00Z",
  "absolute_expires_at": "2026-07-31T20:00:00Z",
  "generation": 18,
  "impersonation": null
}
```

Impersonation transition:

```text
ordinary session
→ explicit support action
→ new short-lived impersonation session
```

Do not overwrite the actor identity.

---

# 17.16 Capstone Export Job

Submission record:

```json
{
  "job_id": "job-9182",
  "tenant_id": "tenant-42",
  "report_id": "report-71",
  "report_revision": 18,
  "subject": "u_1048",
  "submission_actor": "finance-bff",
  "execution_actor": "export-worker",
  "action": "report.export",
  "authorized_at": "2026-07-31T12:10:00Z",
  "authorization_expires_at": "2026-07-31T12:20:00Z",
  "policy_version": "reports-18",
  "idempotency_key": "tenant-42-export-9182"
}
```

Worker checks:

1. Job signature or trusted queue origin.
2. Job has not already completed.
3. Authorization envelope has not expired.
4. Report revision still matches or reauthorization succeeds.
5. Tenant remains active.
6. Subject has not been critically disabled, according to policy.
7. Output path matches tenant.
8. Cloud credential is short-lived and workload-bound.

---

# 17.17 Capstone Revocation Design

Requirement:

```text
Critical new actions blocked within 60 seconds
```

Possible controls:

* Account disablement event
* User session generation increment
* BFF session cache invalidation
* Short authorization-decision cache
* API deny-state feed
* Five-minute access token plus emergency deny list
* Worker checks current critical-disable state before export
* Long-lived streams terminated

Measure:

```text
disable accepted
→ session rejected
→ API token rejected for critical action
→ queued export blocked
→ workload audit confirms containment
```

---

# 17.18 Capstone IdP Outage Design

Requirement:

```text
Existing ordinary read sessions may continue for 10 minutes
```

Possible policy:

```text
New login:
  unavailable

Token refresh:
  unavailable

Existing BFF session:
  usable for low-risk reads

Existing API token:
  usable until short expiry

Restricted export:
  deny if fresh token or step-up unavailable

Administration:
  deny except controlled break-glass
```

This degraded mode must be implemented and tested before the outage.

---

# 17.19 Capstone Threat Model

## Threat: stolen browser cookie

Defenses:

* Secure and HttpOnly cookie
* Short idle expiry
* Server-side revocation
* Session rotation
* Step-up for export
* Risk detection
* Device-bound proof where justified

Residual risk:

```text
Attacker with valid bearer cookie
may act as user until detected, expired, or revoked
```

---

## Threat: cross-tenant report ID manipulation

Defenses:

* Tenant-qualified database query
* Policy tenant check
* Tenant-aware cache key
* Property tests
* Row-level security
* No existence leakage where required

---

## Threat: compromised BFF

Potential authority:

* User sessions
* Refresh tokens
* Downstream user delegation
* Browser responses

Defenses:

* Token encryption
* Short downstream tokens
* Workload identity
* Egress restrictions
* Narrow audiences
* Separate administrative plane
* Strong monitoring
* Minimal operating-system privilege

---

## Threat: compromised Export Worker

Potential authority:

* Read job messages
* Obtain cloud temporary role
* Write export objects

Defenses:

* Exact ServiceAccount subject
* Restricted cloud object prefix
* No bucket administration
* Job-envelope validation
* No refresh tokens
* Short credentials
* Network restrictions
* Image integrity and runtime isolation

---

## Threat: forged support impersonation

Defenses:

* Dedicated permission
* Approval or ticket
* Step-up
* Short context
* Actor and subject logs
* Restricted actions
* User notification where appropriate
* No client-controlled impersonation headers

---

# 17.20 Capstone Test Matrix

| Test                                                 | Expected result                                |
| ---------------------------------------------------- | ---------------------------------------------- |
| Tenant A user reads Tenant A public report           | Allow                                          |
| Tenant A user reads Tenant B report                  | Deny                                           |
| Tenant-local admin reads another tenant              | Deny                                           |
| Global security admin uses normal session            | Deny or require admin plane                    |
| Export without recent MFA for restricted report      | Step-up required                               |
| Replayed export idempotency key                      | Existing job returned or duplicate denied      |
| Worker writes outside tenant prefix                  | Cloud authorization denial                     |
| Wrong ServiceAccount exchanges for cloud role        | Deny                                           |
| Disabled user submits new export after 60 seconds    | Deny                                           |
| Existing low-risk read during short IdP outage       | Allow according to degraded policy             |
| Existing privileged admin action during IdP outage   | Deny                                           |
| Support agent modifies user MFA during impersonation | Deny                                           |
| Old report revision used by queued job               | Reauthorize or deny                            |
| Direct call bypasses BFF                             | API still validates identity and authorization |
| Stale tenant cache entry reused for another tenant   | Test must detect and fail                      |

---

# 17.21 Capstone Failure Injection

Perform only in a test environment.

## Failure 1: Stale JWKS

* Prevent one API replica from refreshing keys.
* Rotate issuer signing key.
* Send tokens signed by the new key.
* Confirm only stale replica fails.
* Verify unknown-key metrics identify it.
* Restore refresh and confirm recovery.

## Failure 2: Session-store outage

* Block BFF session-store access.
* Test existing sessions.
* Confirm fail behavior matches design.
* Verify no insecure fallback creates unsigned local sessions.

## Failure 3: Policy-service timeout

* Delay policy responses.
* Confirm high-risk writes fail closed.
* Confirm approved low-risk degraded behavior where defined.
* Verify timeout does not become allow.

## Failure 4: Group revocation

* Remove export role.
* Measure propagation through directory, session, token, policy cache, and worker.
* Compare result with revocation objective.

## Failure 5: Certificate rotation

* Issue new workload certificate.
* Rotate trust bundle.
* Verify overlapping trust.
* Confirm every replica loads the new identity.
* Remove old trust only after verification.

## Failure 6: Queue redelivery

* Deliver the same export job twice.
* Confirm idempotency prevents duplicate object creation or billing.

---

# 17.22 Capstone Audit Trail

A successful export should produce linked events.

```text
1. User authentication
2. Session creation
3. Export authorization
4. Job creation
5. Queue publication
6. Worker job consumption
7. Cloud credential issuance
8. Object write
9. Export completion
10. User notification
```

Shared fields:

* Trace ID
* Job ID
* Tenant ID
* Report ID
* Subject
* Actor
* Policy version
* Resource revision
* Timestamp

Do not use the raw access token as the correlation mechanism.

---

# 17.23 Capstone Deliverables

A complete implementation review should produce:

1. Architecture diagram
2. Trust-boundary diagram
3. Identity inventory
4. Credential inventory
5. Data-flow diagram
6. Authorization matrix
7. Session lifecycle
8. Token-validation profile
9. Workload identity policy
10. Tenant-isolation property tests
11. Threat model
12. Revocation objective
13. Failure-mode matrix
14. Key-rotation runbook
15. Credential-compromise runbook
16. Audit schema
17. Monitoring dashboard
18. Deployment and rollback plan

---

# 17.24 Final Laboratory Review Questions

1. Which lab most clearly demonstrated that authentication and authorization are separate?
2. Which credential types in the labs were bearer credentials?
3. Which credentials required proof of private-key possession?
4. Where did replay protection require server-side state?
5. Which labs depended on accurate time?
6. Which validation steps could be performed offline?
7. Which steps required a live authority or datastore?
8. How did tenant context affect both queries and cache keys?
9. Why did the Kubernetes ServiceAccount remain authenticated after its RoleBinding was removed?
10. Why did the mTLS server reject a certificate from another valid CA?
11. How did the JWT lab demonstrate the difference between decoding and verification?
12. Why did the session lab store a hash instead of the raw identifier?
13. Which Linux permission applied to directory traversal?
14. Why must a background worker retain the original user subject?
15. Which capstone operation required fresh authentication?
16. How did the capstone avoid storing permanent cloud keys?
17. What mechanism bounded the authority of the Export Worker?
18. Which failure injections tested cache or replication freshness?
19. How would you prove the 60-second revocation objective?
20. Which artifacts would be required before approving the capstone for production?

# 18. Final Synthesis, Operational Reference, and Comprehensive Review

Authentication and authorization are best understood as one continuous security pipeline.

```text
Evidence
→ authenticated principal
→ mapped local identity
→ bounded session or credential
→ authorization context
→ policy decision
→ enforcement
→ audit and revocation
```

Every arrow is a trust transition.

A system is secure only when each transition preserves the intended meaning of the identity and authority.

---

# 18.1 The Complete Mental Model

A protected operation can be represented as:

```text
May subject S,
acting through actor A,
perform action X,
on resource R,
within tenant T,
under context C,
according to policy version P?
```

Where:

* **Subject** is the human or workload whose authority is being exercised.
* **Actor** is the immediate service, administrator, or delegated process performing the operation.
* **Action** is the specific operation.
* **Resource** is the exact object or collection being accessed.
* **Tenant** is the isolation boundary.
* **Context** includes authentication method, device state, time, network, risk, and delegation.
* **Policy version** identifies the rules used for the decision.

A role name alone does not fully answer the question.

---

## Example

```yaml
subject: u_1048
actor: finance-bff
action: report.export
resource:
  type: report
  id: report-71
  tenant: tenant-42
  revision: 18
context:
  authentication_methods:
    - password
    - webauthn
  authentication_age_seconds: 96
  device_managed: true
  impersonation: false
policy_version: reports-18
```

Possible decision:

```yaml
decision: allow
reason: tenant_exporter_with_recent_mfa
expires_at: 2026-07-31T14:35:00Z
```

The resulting operation should remain bound to:

* Report 71
* Revision 18
* Tenant 42
* Export action
* The specified subject and actor
* The decision lifetime

---

# 18.2 Authentication Is Evidence Evaluation

Authentication begins with evidence.

Examples:

| Evidence               | What it can prove                                    |
| ---------------------- | ---------------------------------------------------- |
| Password               | Knowledge of a shared secret                         |
| TOTP                   | Knowledge of a seed-derived current code             |
| Passkey assertion      | Control of a registered private key                  |
| TLS client proof       | Control of a certificate-associated private key      |
| Kerberos authenticator | Possession of a ticket session key                   |
| OAuth token            | Authority asserted by a trusted issuer               |
| Session cookie         | Possession of a server-recognized session identifier |
| Device attestation     | Claims about device or workload state                |
| Unix peer credentials  | Kernel-observed process UID or GID                   |

Evidence must be evaluated within context.

A valid signature alone does not establish:

* Expected issuer
* Correct audience
* Intended protocol
* Current validity
* Local account state
* Tenant membership
* Permission for the requested operation

---

# 18.3 Authorization Is State Evaluation

Authorization normally evaluates facts from several sources.

```text
Identity facts:
  subject, actor, groups, tenant membership

Credential facts:
  issuer, audience, scopes, authentication methods

Resource facts:
  owner, tenant, classification, state, revision

Environmental facts:
  time, network, device, risk, location

Policy facts:
  rules, version, exceptions, explicit denies
```

Authorization is not just a lookup of:

```text
user → role
```

It is often a distributed state computation.

---

# 18.4 The Four Independent Questions

For any request, answer these separately.

## Question 1: Who established the transport?

Examples:

* TLS server certificate
* TLS client certificate
* SSH host key
* Service-mesh workload identity
* Unix socket peer credentials

## Question 2: Which principal is authenticated?

Examples:

* Alice
* `orders-service`
* Kubernetes ServiceAccount
* Kerberos service principal
* OAuth client

## Question 3: Whose authority is being exercised?

Examples:

* The workload’s own authority
* Alice’s delegated authority
* Support agent acting as Alice
* Background worker executing an approved job

## Question 4: Is this exact operation permitted?

Examples:

* Read report 71
* Export tenant-42 records
* Delete one Pod
* Approve a payment not created by the approver

Combining these questions into one vague “authenticated” flag creates security ambiguity.

---

# 18.5 Trust-Boundary Review Method

For every boundary, document:

```text
Source component
Destination component
Network or IPC path
Source authentication
Destination authentication
Credential or assertion
Confidentiality protection
Replay protection
Authorization responsibility
Failure behavior
Audit evidence
```

Example:

```yaml
source: finance-bff
destination: reports-api
path: service-mesh mTLS
source_authentication: SPIFFE workload certificate
destination_authentication: reports-api SPIFFE identity
delegated_credential: OAuth access token
token_audience: reports-api
token_subject: u_1048
token_actor: finance-bff
replay_control: short expiry and token binding
authorization_owner: reports-api
failure_behavior: deny
audit: subject, actor, tenant, action, policy version
```

---

# 18.6 Authentication Architecture Decision Guide

## Use a server-side session when

* The client is a browser.
* Rapid central revocation matters.
* The server can maintain session state.
* Browser JavaScript should not access bearer tokens.
* Existing sessions may outlive an upstream identity-provider interruption.

## Use a short-lived signed access token when

* Several independent APIs must validate authority.
* Offline validation improves availability.
* Audience and scope can be narrowly defined.
* Revocation can be bounded by short lifetime.
* Key distribution and rotation are operationally mature.

## Use token introspection when

* Near-real-time token status matters.
* The authorization server can support runtime traffic.
* Resource servers can tolerate an online dependency.
* Opaque-token privacy is desired.

## Use mTLS when

* Both endpoints are controlled workloads.
* Strong transport identity is required.
* Certificate issuance and rotation can be automated.
* Peer identity can be matched exactly.

## Use workload federation when

* A workload already has a platform identity.
* Permanent cloud keys should be avoided.
* A trusted token service can issue temporary credentials.
* Exact subject and audience matching is available.

## Use Kerberos when

* An enterprise realm and KDC already exist.
* Network SSO is required.
* Services can manage principal names and keytabs.
* Time synchronization and DNS are well controlled.

## Use LDAP Bind when

* The directory is the intended credential verifier.
* Password transmission occurs only through protected transport.
* The application understands Bind state and directory ACLs.
* User DNs can be safely resolved.

---

# 18.7 Authorization Model Decision Guide

## RBAC

Use when:

* Permissions align with stable job functions.
* Role count remains manageable.
* Resource-specific relationships are limited.

Watch for:

* Role explosion
* Tenant ambiguity
* Permanent privilege accumulation
* Global roles accidentally used as tenant roles

## ABAC

Use when decisions depend on:

* Department
* Classification
* Device state
* Time
* Transaction amount
* Authentication strength

Watch for:

* Missing attributes
* Type conversion
* Stale attributes
* Difficult policy explanation

## ReBAC

Use when access depends on relationships:

* Owner
* Viewer
* Group member
* Parent folder
* Project participant
* Organization hierarchy

Watch for:

* Graph cycles
* Stale replicas
* Expensive traversal
* Check/list semantic differences

## Capability-based authorization

Use when:

* Authority can be represented as a bounded object.
* Delegation and attenuation are important.
* Offline verification is useful.
* Possession semantics are acceptable.

Watch for:

* Leakage
* Replay
* Difficult revocation
* Excessively broad capabilities

Most large systems combine several models.

---

# 18.8 Complete Request-Processing Blueprint

```mermaid
flowchart TD
    A[Receive request]
    B[Apply size and syntax limits]
    C[Establish trusted proxy and transport context]
    D[Extract credential]
    E[Validate credential structure]
    F[Verify cryptographic proof]
    G[Validate issuer, audience, type, and time]
    H[Map to stable local subject]
    I[Check account and session state]
    J[Resolve tenant and actor]
    K[Load resource authorization attributes]
    L[Evaluate policy]
    M{Allow?}
    N[Execute operation with constrained authority]
    O[Record audit event]
    P[Return success]
    Q[Return safe denial]

    A --> B --> C --> D --> E --> F --> G --> H --> I --> J --> K --> L --> M
    M -->|Yes| N --> O --> P
    M -->|No| Q
```

The operation must not occur before the authorization decision.

---

# 18.9 Secure Middleware Ordering

Recommended order:

```text
1. Request size limits
2. Protocol parsing
3. Trusted proxy processing
4. Transport identity
5. Credential extraction
6. Credential validation
7. Identity mapping
8. Account-state checks
9. Tenant resolution
10. Route-level permission
11. Resource-level permission
12. Business invariant
13. Side effect
14. Audit result
```

Unsafe ordering:

```text
Load and modify resource
→ call authorization
→ return 403
```

The user sees a denial, but the protected action has already occurred.

---

# 18.10 Credential Validation Profiles

Each credential type should have an explicit profile.

## Access-token profile

```yaml
credential_type: OAuth access token
expected_issuer: https://identity.example.internal
expected_audience: reports-api
allowed_algorithms:
  - approved-asymmetric-algorithm
required_claims:
  - iss
  - sub
  - aud
  - exp
  - iat
expected_token_type: access-token
maximum_lifetime: 5m
clock_skew: bounded
key_source: trusted issuer JWKS
unknown_key_behavior: rate-limited refresh, then reject
```

## Session-cookie profile

```yaml
credential_type: opaque session
cookie_name: __Host-session
secure: true
http_only: true
same_site: Lax
server_store: required
idle_lifetime: 30m
absolute_lifetime: 8h
rotation:
  - login
  - MFA completion
  - privilege elevation
  - impersonation transition
revocation: server-side
```

## mTLS profile

```yaml
credential_type: X.509 client certificate
trusted_ca: production workload CA
required_eku: clientAuth
identity_source: URI SAN
accepted_trust_domain: production.example
maximum_leaf_lifetime: short
revocation_strategy:
  - short lifetime
  - connection lifetime limit
  - emergency trust removal
```

---

# 18.11 Session Security Checklist

A secure session system should answer yes to the following:

* Is the session ID generated using a cryptographically secure random source?
* Is it long enough to prevent guessing?
* Is the raw identifier absent from ordinary logs?
* Is server-side storage indexed by a protected representation?
* Does the cookie use `Secure`?
* Does it use `HttpOnly`?
* Is SameSite policy intentional?
* Is the session rotated after authentication?
* Is it rotated after privilege changes?
* Are idle and absolute expiry separate?
* Can one session be revoked?
* Can all sessions for one user be revoked?
* Does password recovery apply a defined revocation policy?
* Are session-store replicas consistent enough for the revocation objective?
* Are long-lived WebSocket or streaming connections handled?
* Is CSRF protection present for cookie-authenticated writes?

---

# 18.12 Token Security Checklist

* Is the issuer configured locally?
* Is the exact audience validated?
* Is the token type enforced?
* Are allowed algorithms explicitly configured?
* Is key selection limited to trusted issuer material?
* Is expiry required?
* Is `nbf` handled where used?
* Is clock skew bounded?
* Is the token lifetime appropriate?
* Are scopes interpreted by the intended API?
* Is tenant context validated?
* Are actor and subject distinct where delegation occurs?
* Are tokens prevented from appearing in URLs or logs?
* Is key rotation tested?
* Are unknown key IDs rate limited?
* Are refresh-token families tracked?
* Is token replay acceptable or mitigated?
* Is an ID Token rejected as an API access token?

---

# 18.13 Password Security Checklist

* Approved password-hashing function
* Unique random salt
* Measured work parameters
* Stable encoding and normalization policy
* Maximum input length
* Breached-password screening where appropriate
* Rehash-on-login
* Generic external errors
* Dummy verification for unknown users
* Rate limits at several dimensions
* MFA integration
* Strong password-reset flow
* Session review after reset
* No password logging
* No plaintext or reversible password storage
* No use of a general fast hash such as plain SHA-256 for password storage

---

# 18.14 Authorization Checklist

For every protected route:

* What action is being performed?
* Which exact resource is affected?
* Which tenant owns it?
* Who is the subject?
* Who is the actor?
* Which policy version applies?
* Which attributes are required?
* What happens when an attribute is missing?
* Is an explicit deny possible?
* Does the route share semantics with search, export, and batch operations?
* Is the database query tenant constrained?
* Is the authorization cache key complete?
* Is the decision bound to resource revision?
* Can direct backend access bypass the check?
* Are side effects atomic with required state conditions?
* Is the decision auditable?

---

# 18.15 Multi-Tenant Isolation Checklist

Tenant isolation must apply to:

* Direct object reads
* Direct object writes
* Search
* Pagination
* Counts
* Exports
* Attachments
* Background jobs
* Event streams
* Notifications
* Caches
* Logs
* Audit search
* Pre-signed URLs
* Database backups exposed through product features
* Administrator tools
* Support impersonation

A tenant ID in the URL is not proof of tenant membership.

---

# 18.16 Workload Identity Checklist

* Does every workload have a distinct identity?
* Is identity derived from platform attestation rather than copied static secrets?
* Is the identity bound to namespace, account, or deployment?
* Does the credential have a narrow audience?
* Is the credential short lived?
* Does the workload reread rotated credentials?
* Are downstream permissions resource constrained?
* Can one compromised workload obtain another workload’s identity?
* Are node-wide credentials exposed to Pods?
* Are metadata-service paths restricted?
* Does audit identify the exact workload?
* Are human and workload authority recorded separately?

---

# 18.17 Administrative Security Checklist

* Dedicated administrative identity
* Phishing-resistant MFA
* Managed administrative device
* Short session
* Restricted network or access path
* Step-up for critical changes
* Independent approval where required
* Actor identity preserved
* No use of ordinary customer impersonation headers
* Versioned configuration
* Change diff
* Rollback plan
* Immutable or separately protected audit
* Break-glass procedure
* Post-break-glass credential rotation
* Periodic access review

---

# 18.18 Asynchronous Authorization Checklist

For every queued job:

* Who requested it?
* Which workload submitted it?
* Which workload executes it?
* Which action was authorized?
* Which resource and revision were authorized?
* Which tenant applies?
* Which policy version approved it?
* When does authorization expire?
* Must authority be rechecked at execution?
* Is the job idempotent?
* Can the message be replayed?
* Is the queue origin authenticated?
* Are duplicate deliveries safe?
* Does the worker have only necessary infrastructure permissions?
* Does audit preserve the complete actor chain?

---

# 18.19 Revocation Checklist

Identify every representation of authority:

```text
Directory group
Local role
Application session
Access token
Refresh token
API key
Certificate
Kerberos ticket
Delegated token
Queued authorization
Long-lived connection
Policy cache
Database session
Cloud temporary credential
```

For each, document:

* Source of truth
* Lifetime
* Cache
* Revocation mechanism
* Maximum stale period
* Emergency response
* Monitoring

The effective revocation time is determined by the slowest relevant representation.

---

# 18.20 Reliability Checklist

* Are identity dependencies redundant?
* Can cached public keys support temporary issuer failure?
* Can existing sessions survive a planned IdP interruption?
* Is degraded behavior explicitly defined?
* Do retries use exponential backoff and jitter?
* Are refresh storms prevented?
* Are timeouts bounded?
* Are downstream deadlines propagated?
* Are expensive password operations concurrency limited?
* Are replay and decision caches capacity bounded?
* Is fail-open behavior prohibited for critical actions?
* Can key and certificate rotation occur without downtime?
* Is failover tested under realistic load?
* Are regional consistency limitations documented?

---

# 18.21 Observability Checklist

Authentication logs should include:

* Timestamp
* Request ID
* Stage
* Credential type
* Issuer
* Audience
* Subject hash or internal ID
* Actor
* Result
* Failure reason
* Authentication methods
* Replica and region

Authorization logs should include:

* Subject
* Actor
* Tenant
* Action
* Resource
* Decision
* Reason
* Policy ID
* Policy version
* Relevant input versions
* Resource revision

Do not include raw credentials.

---

# 18.22 Incident Triage Quick Reference

## Symptom: no network connection

Check:

```bash
getent ahosts service.example.internal
ip route get 192.0.2.80
nc -vz service.example.internal 443
ss -ltnp
```

Likely layers:

* DNS
* Routing
* Firewall
* Network policy
* Listener
* Wrong port

---

## Symptom: TLS handshake failure

Check:

```bash
openssl s_client \
  -connect service.example.internal:443 \
  -servername service.example.internal \
  -verify_hostname service.example.internal \
  -verify_return_error
```

Likely causes:

* Trust store
* Certificate chain
* Hostname
* SNI
* Time
* Client certificate
* Protocol mismatch
* Key mismatch

---

## Symptom: HTTP 401

Check:

* Credential present?
* Correct credential type?
* Issuer?
* Audience?
* Signature?
* Expiry?
* Session state?
* Proxy stripped header?
* Cookie sent?

Do not assume the password was wrong.

---

## Symptom: HTTP 403

Check:

* Authenticated subject
* Actor
* Tenant
* Action
* Resource
* Roles
* Scopes
* Attributes
* Policy version
* Explicit deny
* Resource state
* Authentication freshness

---

## Symptom: intermittent authentication failure

Compare:

* Replica
* Region
* Clock
* Trust bundle
* Signing-key cache
* Session-store replica
* Policy version
* DNS result
* Certificate serial
* Deployment version

---

## Symptom: group removal does not revoke access

Trace:

```text
Directory
→ replication
→ group cache
→ identity-provider claim
→ token
→ session
→ policy cache
→ connection
```

---

## Symptom: mTLS succeeds but user delegation fails

Transport may authenticate the workload correctly while:

* User token has wrong audience.
* Subject is missing.
* Actor mapping is wrong.
* Token binding does not match the certificate.
* API policy denies the user action.

---

# 18.23 Command Quick Reference

Tool behavior and flags can vary by platform and release. Consult the installed tool’s manual before using commands in sensitive environments.

---

## Identity and process context

```bash
id
```

```bash
getent passwd alice
```

```bash
getent group finance
```

```bash
ps -o pid,ppid,user,group,euid,egid,comm,args -p "${PID}"
```

```bash
grep -E '^(Uid|Gid|Groups|Cap)' "/proc/${PID}/status"
```

---

## Filesystem authorization

```bash
namei -l /path/to/resource
```

```bash
stat /path/to/resource
```

```bash
getfacl /path/to/resource
```

```bash
getcap /path/to/executable
```

```bash
sudo -u service-account -- test -r /path/to/resource
```

---

## Network and socket inspection

```bash
getent ahosts service.example.internal
```

```bash
dig +short service.example.internal
```

```bash
ip route get 192.0.2.80
```

```bash
nc -vz service.example.internal 443
```

```bash
sudo ss -ltnp
```

```bash
sudo ss -tnp state established
```

---

## HTTP diagnostics

```bash
curl -sS \
  -D response-headers.txt \
  -o response-body.txt \
  https://service.example.internal/path
```

```bash
curl -v \
  -o /dev/null \
  https://service.example.internal/path
```

```bash
curl -sS \
  -o /dev/null \
  -w 'dns=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nfirst_byte=%{time_starttransfer}\ntotal=%{time_total}\nstatus=%{http_code}\n' \
  https://service.example.internal/path
```

```bash
curl \
  --resolve service.example.internal:443:192.0.2.80 \
  https://service.example.internal/path
```

Verbose and trace output may reveal credentials.

---

## TLS diagnostics

```bash
openssl s_client \
  -connect service.example.internal:443 \
  -servername service.example.internal \
  -showcerts \
  -verify_hostname service.example.internal \
  -verify_return_error
```

```bash
openssl x509 \
  -in certificate.pem \
  -noout \
  -subject \
  -issuer \
  -serial \
  -dates \
  -ext subjectAltName \
  -ext extendedKeyUsage
```

```bash
openssl verify \
  -CAfile root-ca.pem \
  certificate.pem
```

---

## Journal inspection

```bash
sudo journalctl \
  -u service-name.service \
  --since "15 minutes ago"
```

```bash
sudo journalctl \
  -u service-name.service \
  -f
```

```bash
sudo journalctl \
  -u service-name.service \
  -o json-pretty
```

---

## Packet capture

```bash
sudo tcpdump \
  -i any \
  -nn \
  -s 0 \
  -w focused-capture.pcap \
  'host 192.0.2.80 and tcp port 443'
```

```bash
tcpdump -nn -r focused-capture.pcap
```

Capture only authorized, narrowly filtered traffic.

---

## Kerberos

```bash
kinit alice@EXAMPLE.INTERNAL
```

```bash
klist -ef
```

```bash
kvno HTTP/service.example.internal@EXAMPLE.INTERNAL
```

```bash
sudo klist -kte /etc/service.keytab
```

```bash
KRB5_TRACE=/dev/stderr \
  kinit alice@EXAMPLE.INTERNAL
```

```bash
kdestroy
```

---

## LDAP

```bash
openssl s_client \
  -connect directory.example.internal:389 \
  -starttls ldap \
  -servername directory.example.internal \
  -verify_hostname directory.example.internal \
  -verify_return_error
```

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -x \
  -D 'uid=reader,ou=Services,dc=example,dc=internal' \
  -W \
  -b 'ou=People,dc=example,dc=internal' \
  '(uid=alice)' \
  dn uid cn
```

```bash
ldapsearch \
  -H ldap://directory.example.internal \
  -ZZ \
  -Y GSSAPI \
  -b 'ou=People,dc=example,dc=internal' \
  '(uid=alice)' \
  dn uid cn
```

---

## Kubernetes

```bash
kubectl auth can-i \
  get configmaps \
  --namespace finance \
  --as=system:serviceaccount:finance:reports-api
```

```bash
kubectl auth can-i \
  --list \
  --namespace finance \
  --as=system:serviceaccount:finance:reports-api
```

```bash
kubectl get pod POD_NAME \
  --namespace finance \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

```bash
kubectl create token reports-api \
  --namespace finance \
  --audience reports-api \
  --duration 10m
```

Do not print or share the resulting token.

---

# 18.24 Common Error Interpretation Matrix

| Observation                   | What it proves                            | What it does not prove                         |
| ----------------------------- | ----------------------------------------- | ---------------------------------------------- |
| DNS resolves                  | A name mapped to an address               | Correct service or trustworthy destination     |
| TCP connects                  | A listener accepted TCP                   | TLS, application health, or authorization      |
| TLS validates                 | Peer certificate and trust checks passed  | Application identity mapping or permission     |
| Password verifies             | Submitted password matches record         | Account active or operation authorized         |
| JWT signature verifies        | Token was signed by selected key          | Expected issuer, audience, type, or permission |
| Kerberos `kvno` succeeds      | KDC issued a service ticket               | Service keytab can decrypt it                  |
| LDAP search succeeds          | Bound identity may read entry             | User password is correct                       |
| Session lookup succeeds       | Session record exists                     | Account remains permitted                      |
| `kubectl auth can-i` says yes | RBAC evaluation permits modeled action    | Admission or application policy will permit it |
| Policy returns allow          | Decision engine permitted modeled request | Enforcement actually occurred correctly        |
| UI hides operation            | User interface omits control              | Backend endpoint is protected                  |
| HTTP 403 returned             | Some layer denied request                 | No protected side effect occurred              |

---

# 18.25 Common Security Failure Patterns

## Authentication without identity binding

```text
Signature valid
but
issuer, audience, recipient, or server name not validated
```

## Authorization without resource binding

```text
User has role
but
resource tenant and ownership are ignored
```

## Delegation without actor identity

```text
Downstream log shows Alice
but
does not show which service acted for Alice
```

## Revocation without cache analysis

```text
Role removed
but
token and session remain active
```

## Transport trust without application trust

```text
Gateway authenticates client
but
backend accepts spoofable identity headers
```

## Secure protocol with unsafe recovery

```text
Passkey login
but
account can be recovered through weak support questions
```

## Correct policy with bypass path

```text
REST API protected
but
export or GraphQL route omits the same check
```

## Secure credential with broad authority

```text
Short-lived token
but
audience and scope permit every internal API
```

---

# 18.26 Architecture Review Exercise 1

## Scenario

A browser sends an access token stored in local storage directly to five internal APIs.

The same token contains:

```json
{
  "sub": "u_1048",
  "aud": "internal-services",
  "scope": "all"
}
```

### Problems

* XSS can read the token.
* One token is valid at several services.
* Audience is broad.
* Scope is broad.
* Token theft affects many APIs.
* Each API must independently implement browser-facing token handling.
* Refresh-token storage may also be exposed.
* Central logout may be slow.

### Improved architecture

```text
Browser
→ HttpOnly session cookie
→ BFF
→ audience-specific short-lived API tokens
```

Each API receives only the authority it needs.

---

# 18.27 Architecture Review Exercise 2

## Scenario

A gateway validates user tokens and sends:

```http
X-User: alice
X-Role: admin
```

The backend is also reachable through an internal load balancer.

### Vulnerability

A caller reaching the backend directly can supply the headers.

### Required controls

* Backend reachable only through trusted gateway path
* Gateway-to-backend authentication
* Gateway strips incoming identity headers
* Structured and integrity-protected identity context
* Backend rejects missing context
* Global and tenant roles distinguished
* Backend performs resource authorization

---

# 18.28 Architecture Review Exercise 3

## Scenario

A report-export worker receives:

```json
{
  "user_id": "u_1048",
  "report_id": "report-71"
}
```

The worker has unrestricted database and object-storage access.

### Problems

* Tenant absent
* Actor absent
* Authorization time absent
* Policy version absent
* Resource revision absent
* Job can be replayed
* Worker infrastructure authority is too broad
* No proof that the API authorized the export

### Improved envelope

```json
{
  "job_id": "job-9182",
  "tenant_id": "tenant-42",
  "report_id": "report-71",
  "report_revision": 18,
  "subject": "u_1048",
  "submission_actor": "finance-bff",
  "execution_actor": "export-worker",
  "action": "report.export",
  "authorized_at": "2026-07-31T14:00:00Z",
  "authorization_expires_at": "2026-07-31T14:10:00Z",
  "policy_version": "reports-18",
  "idempotency_key": "tenant-42-export-9182"
}
```

The worker also receives a short-lived cloud credential restricted to the tenant’s export prefix.

---

# 18.29 Architecture Review Exercise 4

## Scenario

An application validates a JWT signature and reads:

```json
{
  "role": "admin"
}
```

It grants platform administration.

### Missing validation

* Issuer
* Audience
* Token type
* Expiry
* Tenant
* Claim-mapping policy
* Administrator assurance
* Actor context

A customer-controlled identity provider may legitimately issue `role=admin` for its own tenant.

It must not become a global platform administrator.

---

# 18.30 Architecture Review Exercise 5

## Scenario

A user is removed from `finance-admins`, but access continues for 45 minutes.

### Investigation path

```text
Directory replication
→ group lookup cache
→ IdP token claim
→ access-token lifetime
→ application session snapshot
→ policy decision cache
→ active streaming connection
```

### Corrective design

* Define revocation objective.
* Shorten or invalidate critical caches.
* Use account or membership generation.
* Use short access tokens.
* Add emergency deny state for critical operations.
* Reauthorize long-lived streams.
* Measure propagation.

---

# 18.31 Comprehensive Final Examination

## Part A: Foundations

1. Define identification, authentication, authorization, and accounting.
2. Why is authentication not equivalent to authorization?
3. What is a principal?
4. What is a credential?
5. What is an authenticator?
6. What is a session?
7. What is a claim?
8. What is an issuer?
9. What is an audience?
10. What is a trust boundary?

## Part B: Passwords and Factors

11. Why must password hashes use salts?
12. Why should password hashes use deliberately expensive algorithms?
13. What is the difference between a password salt and a pepper?
14. Why can password-policy complexity rules reduce usability without stopping credential stuffing?
15. What attack does MFA primarily reduce?
16. Why is SMS generally weaker than phishing-resistant authentication?
17. What is the distinction between user presence and user verification?
18. Why must MFA enrollment require strong existing authentication?
19. Why is account recovery part of authentication security?
20. What should happen to sessions after a high-risk password reset?

## Part C: Sessions and Tokens

21. Why should session IDs be random and opaque?
22. Why should sessions rotate after login?
23. What is session fixation?
24. What is the difference between idle and absolute expiry?
25. Why is a copied session cookie usually sufficient for a bearer-session attacker?
26. What does JWT decoding prove?
27. What does JWT signature validation prove?
28. Which additional JWT checks are required?
29. Why must an API reject a token intended for another API?
30. What is the difference between an access token and an ID Token?
31. Why are refresh tokens more sensitive than short access tokens?
32. What is refresh-token rotation?
33. What is token-family reuse detection?
34. Why can a signed token remain dangerous after a user is disabled?
35. How does introspection change token availability and revocation properties?

## Part D: Federation

36. What does OAuth primarily provide?
37. What does OpenID Connect add?
38. What does SAML commonly provide?
39. What does the OAuth `state` value protect?
40. What does the OIDC nonce bind?
41. What attack does PKCE mitigate?
42. Why must redirect URIs be matched precisely?
43. Why should an authorization code be short-lived and single-use?
44. Why must the relying party validate the OIDC issuer?
45. Why is a signed SAML assertion not sufficient without audience and recipient checks?
46. What is XML signature wrapping?
47. Why should the application consume the exact signed assertion node?
48. What is front-channel logout versus back-channel revocation?
49. Why can identity-provider logout fail to terminate a local application session?
50. Why should federation claim mappings be issuer specific?

## Part E: Operating Systems and Enterprise Identity

51. What is the difference between real and effective UID?
52. What does execute permission on a directory allow?
53. How do ACLs interact with basic Unix mode bits?
54. What problem does PAM solve?
55. What problem does NSS solve?
56. Why can a Linux user be found through NSS but still fail authentication?
57. What does a Kerberos TGT permit?
58. What does a Kerberos service ticket identify?
59. Why does the service need a keytab?
60. What is a Kerberos authenticator?
61. Why is time synchronization critical to Kerberos?
62. What does `kvno` test?
63. What is the difference between LDAP Bind and Search?
64. Why must simple Bind use protected transport?
65. Why should an empty password be rejected before LDAP Bind?
66. How can LDAP connection pooling produce identity confusion?
67. Why are LDAP filter escaping and DN escaping different?
68. What is a Service Principal Name?
69. Why can a load-balancer alias cause Kerberos failure?
70. What is constrained delegation?

## Part F: Workload Identity

71. Why are static cloud keys a secret-zero problem?
72. What does mTLS authenticate?
73. Why must certificate identity be checked after chain validation?
74. What is an X.509 SAN?
75. What is an EKU?
76. Why should workload certificates be short lived?
77. What is a Kubernetes ServiceAccount identity?
78. Why does a ServiceAccount not automatically have permissions?
79. What is token audience restriction?
80. How does cloud workload federation work?
81. Why should a cloud trust policy match the exact Kubernetes subject?
82. What is SPIFFE?
83. What is an SVID?
84. How can a service mesh simplify workload identity?
85. Why must application authorization still exist inside a service mesh?
86. What is token exchange?
87. Why should actor and subject both be retained?
88. What is a confused deputy?
89. How does proof of possession reduce bearer-token theft risk?
90. What problem remains if the associated private key is also stolen?

## Part G: Authorization

91. What is RBAC?
92. What is ABAC?
93. What is ReBAC?
94. What is a capability?
95. What is capability attenuation?
96. What does default deny mean?
97. Why is role-only authorization often insufficient?
98. What is object-level authorization?
99. Why must tenant context appear in data queries?
100. What is an explicit deny?
101. What is an indeterminate policy result?
102. Why must missing attributes have defined behavior?
103. What is policy provenance?
104. What is a policy version used for?
105. What is a complete authorization cache key?
106. Why can stale cache state preserve revoked access?
107. What is authorization consistency?
108. What is a revision token?
109. Why must list and check APIs use equivalent semantics?
110. What is a TOCTOU authorization defect?

## Part H: Security and Reliability

111. What is least privilege?
112. What is temporal least privilege?
113. What is separation of duties?
114. What is defense in depth?
115. What is a correlated failure?
116. Why should keys be separated by purpose?
117. Why does key rotation require overlap?
118. What happens if a new signing key is used before its public key is distributed?
119. What is a replay cache?
120. Why must distributed replay handling be designed explicitly?
121. What is canonicalization?
122. What is a parser differential?
123. What is algorithm confusion?
124. What is domain separation?
125. What is a downgrade attack?
126. What does forward secrecy protect?
127. What is fail closed?
128. When can fail closed create an availability problem?
129. What is a defined degraded mode?
130. Why do retries need jitter?
131. What is a credential refresh storm?
132. What is single-flight refresh?
133. What is a negative cache?
134. Why can an unknown JWT key ID cause denial of service?
135. What is a revocation objective?
136. Why should authorization latency be measured with percentiles?
137. Why should metrics avoid user IDs as labels?
138. What is a break-glass credential?
139. Why must break-glass use be followed by credential rotation?
140. Why must recovery plans be tested?

## Part I: Practical Diagnosis

141. What does a successful DNS lookup prove?
142. What does a successful TCP connection prove?
143. What does successful TLS verification prove?
144. What is the first question to ask for a 401?
145. What is the first authorization tuple to capture for a 403?
146. Why should `curl -v` output be protected?
147. Why is an online JWT decoder unsafe for production credentials?
148. What does `namei -l` help diagnose?
149. What does `journalctl -u` filter?
150. What does a packet capture reveal when application data is encrypted?
151. Why can one replica fail after signing-key rotation?
152. Why can a user authenticate successfully but fail session creation?
153. Why can policy return allow while a database denies the operation?
154. Why must denied writes be checked for side effects?
155. How would you trace continued access after group removal?
156. How would you diagnose mTLS success with an anonymous application identity?
157. Why can `kubectl auth can-i` say yes while admission rejects an object?
158. Why can LDAP search succeed while user login fails?
159. Why can `kvno` succeed while the service returns an authentication error?
160. What is the value of comparing one known-good and one known-bad request?

---

# 18.32 Final Examination Answer Key

## Answers 1–10

1. Identification states an identity; authentication verifies evidence; authorization decides permitted actions; accounting records relevant activity.
2. Authentication proves or accepts identity evidence, while authorization evaluates whether that identity may perform an operation.
3. A principal is a named security identity.
4. A credential is evidence or an object used to establish or exercise identity or authority.
5. An authenticator is a factor or mechanism used to prove control of a credential.
6. A session is continuing reliance on a prior authentication event.
7. A claim is an asserted property about a subject, actor, credential, or context.
8. An issuer is the authority that created or vouched for a credential or assertion.
9. An audience identifies the intended recipient or resource server.
10. A trust boundary is a transition where data or authority moves between components with different trust assumptions.

## Answers 11–20

11. Salts prevent identical passwords from producing identical records and defeat broad precomputation.
12. Expensive hashing slows offline guessing and online verification abuse.
13. A salt is unique public input stored with the record; a pepper is a separately protected secret input.
14. Complexity rules do not stop attackers from reusing already breached passwords and can encourage predictable patterns.
15. MFA reduces account takeover when one factor, especially a password, is compromised.
16. SMS depends on telecom and phone-account security and is more vulnerable to interception and account takeover than origin-bound cryptographic methods.
17. User presence proves interaction; user verification proves the authenticator locally verified the user.
18. Factor enrollment creates future login authority and must not be available to an attacker holding only a weak or stolen session.
19. Recovery establishes a new credential and can bypass normal login controls.
20. High-risk reset should trigger a defined review or revocation of active sessions and refresh tokens.

## Answers 21–35

21. Random opaque values prevent guessing and avoid exposing identity or authorization state.
22. Rotation prevents an attacker from preselecting or retaining an identifier through authentication.
23. Session fixation is causing a victim to authenticate using an identifier already known to the attacker.
24. Idle expiry limits inactivity; absolute expiry limits total lifetime regardless of activity.
25. The server normally sees only possession of the same random value and cannot distinguish the thief from the original browser.
26. Decoding reveals unverified claimed data.
27. Signature validation proves the token was signed by the holder of the selected key and was not modified afterward.
28. Issuer, audience, type, time, required claims, algorithm policy, and application-specific semantics.
29. A token issued for one API may represent inappropriate authority at another API.
30. An access token authorizes API access; an ID Token communicates an authentication result to a client.
31. A refresh token can obtain new access tokens and often persists much longer.
32. Rotation replaces a used refresh token with a new one.
33. Reuse detection identifies presentation of an old token after rotation and may revoke the whole family.
34. Offline-verifiable tokens can remain valid until expiry unless revocation state is checked.
35. Introspection adds an online status check, improving central control but creating latency and availability dependency.

## Answers 36–50

36. OAuth provides delegated authorization.
37. OIDC adds an authentication layer and identity claims.
38. SAML commonly provides federated authentication and attribute assertion.
39. `state` binds the response to the initiating browser transaction and helps prevent request-forgery and response-mix-up attacks.
40. The nonce binds the ID Token to the corresponding authentication request.
41. PKCE prevents an intercepted authorization code from being redeemed without the verifier.
42. Loose matching can redirect credentials or codes to an attacker-controlled destination.
43. Short, single-use codes reduce replay and leakage impact.
44. Otherwise a token from another issuer could be accepted.
45. A valid signature does not prove the assertion was intended for this service or endpoint.
46. XML signature wrapping causes the validator and application to process different elements.
47. Re-parsing or selecting another node can discard the signature’s protection.
48. Front-channel logout uses the browser; back-channel revocation communicates directly between systems.
49. The local session is independent state and may not be deleted by provider logout alone.
50. The same claim can have different trust and authorization meaning depending on issuer.

## Answers 51–70

51. Real UID represents the original identity; effective UID is commonly used for permission checks.
52. Directory execute permission allows traversal and lookup of contained names.
53. ACLs add named-user and named-group permissions, with effective rights influenced by the ACL mask.
54. PAM provides a configurable authentication and account/session policy framework for applications.
55. NSS resolves names such as users, groups, and hosts.
56. Account lookup and credential authentication are separate operations.
57. A TGT permits the client to request service tickets from the TGS.
58. It binds a client principal to a particular service principal for a limited time.
59. The service needs its long-term key to decrypt tickets issued for its principal.
60. A fresh client-created proof protected by the ticket session key.
61. Tickets and authenticators rely on time windows for freshness and replay resistance.
62. It tests whether the KDC can issue a ticket for the exact service principal.
63. Bind establishes connection authentication state; Search queries directory entries.
64. A simple Bind transmits a name and password and requires confidentiality and server authentication.
65. Some LDAP semantics can treat a nonempty name with empty password as unauthenticated rather than as failed password verification.
66. A connection bound as one user can be returned to a pool and reused under the wrong identity.
67. DNs and search filters use different grammars and escaping rules.
68. An SPN identifies a Kerberos service associated with an account and key.
69. Clients request a principal for the alias while the KDC or keytab may contain only the node hostname.
70. Constrained delegation limits which downstream services an intermediary may access for a user.

## Answers 71–90

71. A permanent key must already be delivered securely before it can authenticate the workload.
72. mTLS authenticates peer certificate identities and establishes a protected transport.
73. A valid chain may include many identities; the application must require the intended SAN, hostname, URI, or principal.
74. The Subject Alternative Name carries identities such as DNS names, IP addresses, or URIs.
75. Extended Key Usage constrains intended certificate purposes such as client or server authentication.
76. Short lifetimes reduce compromise and revocation exposure and encourage automated renewal.
77. It is the namespace-qualified identity represented by a Kubernetes ServiceAccount token.
78. Authentication identity and RBAC authorization are separate.
79. Audience limits which verifier should accept a token.
80. A platform identity assertion is exchanged for a temporary credential in another trust domain.
81. Broad subject matching could let unintended Pods assume the role.
82. SPIFFE is a workload-identity specification using trust-domain-scoped identities.
83. An SVID is a verifiable identity document representing a SPIFFE identity.
84. It can automate certificate issuance, rotation, peer authentication, and policy integration.
85. Mesh authentication does not decide whether an authenticated workload may perform each business action.
86. Token exchange converts one credential into a narrower downstream credential.
87. The subject identifies whose authority is exercised; the actor identifies which service performed it.
88. A privileged service is tricked into using its authority for an unauthorized caller.
89. A copied token is insufficient without proof of the associated private key.
90. Theft of both token and key restores the attacker’s ability to present the credential.

## Answers 91–110

91. RBAC grants permissions through roles.
92. ABAC evaluates attributes of subject, resource, action, and environment.
93. ReBAC evaluates relationships between entities.
94. A capability is an unforgeable bearer or proof object representing authority.
95. Attenuation derives a credential with equal or narrower authority.
96. Requests are denied unless policy explicitly permits them.
97. Roles often omit tenant, resource ownership, state, context, and business invariants.
98. It determines permission for the exact resource instance.
99. Otherwise a request can retrieve another tenant’s object before or without authorization.
100. A policy result that intentionally rejects access and commonly overrides permits.
101. The engine could not produce a trustworthy allow or deny because of missing data, errors, or dependency failure.
102. Otherwise different components may convert missing data into inconsistent or unsafe decisions.
103. Provenance explains which rules, inputs, and versions produced a decision.
104. It supports reproducibility, debugging, rollout, rollback, and audit.
105. It includes every decision dimension: subject, actor where relevant, tenant, action, resource, context class, and policy version.
106. The cache can continue returning an earlier allow after source state changes.
107. It defines which policy and relationship updates a decision is guaranteed to observe.
108. It identifies a minimum or exact authorization-data revision.
109. Otherwise listing can reveal resources that direct checks deny, or omit permitted resources.
110. State changes between authorization check and protected operation.

## Answers 111–140

111. Grant only authority required for the intended task.
112. Grant authority only for the period in which it is needed.
113. Require different principals for conflicting or high-impact stages.
114. Use independent controls so one failure does not directly produce compromise.
115. Several apparent controls depend on the same flawed input or component.
116. Separation limits compromise impact and avoids cross-protocol confusion.
117. Verifiers need time to learn the new key while old credentials remain valid.
118. Verifiers may reject every credential signed with the unknown key.
119. State recording recently used authenticators, nonces, or proofs so duplicates can be rejected.
120. A replay can reach another replica that has not recorded the first presentation.
121. Canonicalization defines one byte representation of structured data for signing or comparison.
122. Different components interpret the same bytes differently.
123. The verifier follows an untrusted algorithm declaration outside its policy.
124. Distinct prefixes, formats, or keys prevent one cryptographic object from being interpreted in another protocol.
125. A peer is forced to use a weaker mechanism or omit a security feature.
126. Compromise of a long-term key does not reveal past ephemeral session traffic.
127. Deny or stop when required security evaluation cannot be completed.
128. Failure of one dependency may deny all legitimate operations.
129. A predesigned limited operating mode specifying which actions remain allowed during a dependency outage.
130. Jitter prevents synchronized clients from retrying simultaneously.
131. Many credentials expire together, causing mass renewal traffic.
132. Concurrent callers share one refresh operation.
133. A cache of failed or absent results.
134. A naïve verifier may perform a network refresh for every random key ID.
135. A measurable maximum time from revocation event to effective denial.
136. Averages hide slow-tail behavior.
137. Per-user labels create excessive cardinality and privacy risk.
138. An emergency credential used when normal administration or identity systems are unavailable.
139. Its secrecy has been exposed through retrieval and use.
140. Untested procedures may depend on missing access, stale credentials, or incompatible systems.

## Answers 141–160

141. A configured resolver mapped the name to an address.
142. Something accepted a TCP connection at the destination.
143. The TLS peer’s certificate and configured identity checks passed and a protected channel was established.
144. Determine whether the expected credential was present.
145. Subject, actor, action, resource, tenant, context, and policy version.
146. It may contain Authorization headers, cookies, tokens, and internal topology.
147. The decoder receives a live bearer credential.
148. It shows permissions and ownership for every path component.
149. It limits journal output to a specified systemd unit.
150. Endpoints, timing, packet direction, retransmissions, closures, DNS, and some handshake metadata.
151. Its key cache or trust material may be stale.
152. The session store, serialization, random generation, cookie creation, or transaction may fail after credential verification.
153. The database may apply a different role, tenant constraint, row policy, or resource state.
154. A denial response does not prove the operation was not already committed.
155. Trace every replicated, cached, tokenized, session, policy, and connection representation of group state.
156. Verify how the terminating proxy converts the validated certificate identity into trusted backend context.
157. RBAC authorization and admission policy are separate stages.
158. The service account can search while the user password, DN, account state, or Bind transport fails.
159. The KDC issued a ticket, but the service may lack the correct key, principal mapping, or authorization.
160. It exposes the smallest differing dimensions and helps isolate the first failing boundary.

---

# 18.33 Mastery Criteria

A practitioner has a strong foundational understanding when they can:

* Explain authentication and authorization without conflating them.
* Trace one request across every trust boundary.
* Identify subject and actor separately.
* Explain why a valid signature is not complete validation.
* Diagnose a 401 without assuming password failure.
* Diagnose a 403 using the complete authorization tuple.
* Design a secure browser session.
* Explain OAuth, OIDC, and SAML roles.
* Explain a Kerberos TGT and service ticket.
* Explain LDAP Bind versus Search.
* Trace Linux file access through UID, groups, modes, ACLs, and mandatory controls.
* Design workload identity without permanent cloud keys.
* Define tenant-isolation properties.
* Design bounded delegation.
* Define a measurable revocation objective.
* Plan safe key rotation.
* Build useful logs without storing credentials.
* Design degraded operation without accidental fail-open behavior.
* Test search, export, batch, and asynchronous authorization paths.
* Produce a complete incident timeline from distributed evidence.

---

# 18.34 Final Principles

1. **Identity must be namespaced.**

   A bare username, role, or subject value is not globally meaningful.

2. **Signatures prove integrity, not complete protocol validity.**

   Issuer, audience, type, time, context, and recipient still matter.

3. **Authentication does not grant authority.**

   Every protected action needs an authorization decision.

4. **Authorization must bind to the exact resource and tenant.**

   Route-level roles are not sufficient.

5. **Actor and subject must remain distinct.**

   Delegation and impersonation require both.

6. **Credentials should be narrow and short lived.**

   Limit audience, scope, resource, tenant, and lifetime.

7. **Workloads require first-class identities.**

   Do not treat network location as identity.

8. **Recovery is part of authentication.**

   The weakest recovery path can define total account security.

9. **Revocation is a distributed-systems problem.**

   Every cache, token, session, ticket, and connection matters.

10. **Authorization must precede side effects.**

    A denial response after execution is not security.

11. **Security controls need observable stages.**

    “Login failed” is insufficient operational evidence.

12. **Secure systems must remain operable under failure.**

    Degraded behavior must be explicit, bounded, and tested.

13. **Every trust change is a security change.**

    Issuers, CAs, redirect URIs, claim mappings, and policies deserve controlled rollout.

14. **Every fixed defect should become a test.**

    Security knowledge must be retained in the implementation.

15. **The complete chain determines security.**

```text
Credential handling
+
protocol validation
+
identity mapping
+
session management
+
authorization policy
+
enforcement
+
audit
+
revocation
+
recovery
=
effective security
```
