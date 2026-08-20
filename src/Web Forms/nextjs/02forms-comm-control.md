# Scalable Email Verification, Session & Form Submission Guide

## Next.js + TypeScript + React Hook Form + Zod + SMTP + Database

> Scope: Contact and Start a Project / RFQ forms with global email verification, 30-minute sessions, secure cookies, rate limiting, database persistence, SMTP abstraction, reusable services/hooks/components, and automated testing.

---

# Phase 1 — Target Communication Flow

```text
Contact / RFQ Form
        ↓
useEmailVerification()
        ↓
Check existing verified-email session
        ↓
     ┌──┴──┐
     │     │
    YES    NO
     │     │
     │   Send OTP
     │     ↓
     │   Verify OTP
     │     ↓
     │   Create 30-min session
     │     │
     └──┬──┘
        ↓
Email marked "Verified"
        ↓
Auto-populate email
        ↓
Complete form
        ↓
Client validation
        ↓
Server validation
        ↓
Verify email session
        ↓
Rate limiting / bot checks
        ↓
Business validation
        ↓
Persist submission
        ↓
Business notification email
        ↓
User confirmation email
        ↓
Controlled response
```

**Core principle:** email verification is a global service, not a Contact/RFQ-specific feature.

---

# Phase 2 — Provider Independence

Never couple forms directly to SMTP, DB, storage, or rate-limit providers.

Avoid:

```text
ContactForm → Nodemailer
RFQForm → Firebase
OTPModal → Firestore
```

Prefer:

```text
Application
    ↓
Service Layer
    ↓
Provider Interface
    ↓
Provider Adapter
    ↓
External Provider
```

This lets you replace:

```text
SMTP A → SMTP B
Firebase → PostgreSQL
Redis → another rate-limit provider
```

without rewriting forms.

---

# Phase 3 — Project Structure

```text
src/
├── app/
│   └── api/
│       ├── auth/
│       │   └── email/
│       │       ├── request-otp/route.ts
│       │       └── verify-otp/route.ts
│       ├── contact/route.ts
│       └── rfq/route.ts
│
├── components/
│   ├── forms/
│   │   ├── contact-form.tsx
│   │   ├── rfq-form.tsx
│   │   └── email-field.tsx
│   └── auth/
│       └── email-verification/
│           ├── email-verification.tsx
│           ├── email-input.tsx
│           ├── otp-modal.tsx
│           ├── verified-badge.tsx
│           └── edit-email-button.tsx
│
├── hooks/
│   ├── use-email-verification.ts
│   ├── use-verified-email.ts
│   └── use-form-submission.ts
│
├── lib/
│   ├── auth/
│   │   ├── email-verification.service.ts
│   │   ├── otp.service.ts
│   │   ├── session.service.ts
│   │   └── cookies.ts
│   │
│   ├── email/
│   │   ├── smtp.ts
│   │   ├── email.service.ts
│   │   ├── types.ts
│   │   └── templates/
│   │       ├── otp.ts
│   │       ├── submission-received.ts
│   │       └── submission-confirmation.ts
│   │
│   ├── db/
│   │   ├── index.ts
│   │   ├── types.ts
│   │   ├── repositories/
│   │   │   ├── verification.repository.ts
│   │   │   ├── session.repository.ts
│   │   │   ├── submission.repository.ts
│   │   │   └── rate-limit.repository.ts
│   │   └── providers/
│   │       └── ...
│   │
│   ├── security/
│   │   ├── hash.ts
│   │   ├── rate-limit.ts
│   │   ├── sanitize.ts
│   │   └── constants.ts
│   │
│   ├── validations/
│   │   ├── common.schema.ts
│   │   ├── contact.schema.ts
│   │   ├── rfq.schema.ts
│   │   └── otp.schema.ts
│   │
│   └── forms/
│       ├── constants.ts
│       └── types.ts
│
├── config/
│   ├── env.ts
│   └── email.ts
│
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```

---

# Phase 4 — Environment Configuration

Centralize provider configuration:

```text
# Application
APP_URL=
NODE_ENV=

# Database
DB_PROVIDER=
DB_URL=

# Email
EMAIL_PROVIDER=smtp
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASSWORD=
SMTP_FROM=

# Verification
OTP_TTL_SECONDS=300
OTP_MAX_ATTEMPTS=5
OTP_RESEND_COOLDOWN_SECONDS=60
OTP_MAX_RESENDS=3
EMAIL_SESSION_TTL_SECONDS=1800

# Security
SESSION_SECRET=
RATE_LIMIT_PROVIDER=
```

Use `.env.example` for placeholders and never commit `.env.local`.

The application should not contain provider credentials outside the environment/config layer.

---

# Phase 5 — SMTP Abstraction

### `src/lib/email/smtp.ts`

Responsible only for provider communication:

```text
connect
send
provider-level error handling
```

### `src/lib/email/email.service.ts`

Responsible for application-level communication:

```text
sendVerificationOTP()
sendSubmissionNotification()
sendSubmissionConfirmation()
```

The form must not directly call an SMTP library.

Use:

```text
Form
 ↓
EmailService
 ↓
SMTP Adapter
 ↓
SMTP Provider
```

not:

```text
Form
 ↓
nodemailer.sendMail()
```

---

# Phase 6 — Database Abstraction

Use:

```text
Service
   ↓
Repository
   ↓
DB Adapter
   ↓
Database
```

Repositories expose application operations such as:

```text
createVerification()
findVerification()
markVerified()

createSession()
getSession()
invalidateSession()

createSubmission()

checkRateLimit()
recordRateLimit()
```

Changing Firebase to PostgreSQL should primarily affect the DB adapter/repositories and configuration, not forms, hooks, or UI.

---

# Phase 7 — Database Model

Recommended logical entities:

## `email_verifications`

```text
id
email
otp_hash
attempts
resend_count
expires_at
created_at
verified_at
```

## `email_sessions`

```text
id
email
session_token_hash
expires_at
created_at
last_activity_at
```

## `form_submissions`

```text
id
form_type
email
payload/reference
status
created_at
```

## `rate_limits`

```text
key
count
window_start
expires_at
```

## `audit_logs`

Use only where useful:

```text
event
form_type
reference
timestamp
status
metadata
```

Avoid unnecessary sensitive data.

---

# Phase 8 — Email Normalization

Use one normalization function everywhere:

```text
User input
 ↓
Trim
 ↓
Normalize
 ↓
Validate
 ↓
Use normalized email
```

Use the normalized value for:

```text
OTP
DB lookup
session
rate limiting
form submission
```

This prevents inconsistent identity handling.

---

# Phase 9 — OTP Lifecycle

```text
User enters email
        ↓
Normalize email
        ↓
Rate-limit request
        ↓
Generate cryptographically secure OTP
        ↓
Protect OTP representation
        ↓
Store verification record
        ↓
Send OTP
```

Recommended starting values:

```text
OTP TTL          = 5 minutes
Max attempts     = 5
Resend cooldown  = 60 seconds
Max resends      = 3
```

Make them configurable.

---

# Phase 10 — OTP Security

Never store:

```text
123456
```

as plaintext.

For short-lived OTPs, use a keyed HMAC/secure hash approach.

Conceptually:

```text
OTP
 ↓
HMAC/secure hash + server secret
 ↓
Database
```

Verification:

```text
Submitted OTP
 ↓
Same protection method
 ↓
Compare
```

Invalidate the OTP after:

```text
successful verification
maximum attempts
expiration
```

---

# Phase 11 — Hashing Strategy

Do not use one hashing mechanism for everything.

```text
Passwords
    → Argon2id / bcrypt

Short-lived OTP
    → HMAC / secure hash

Session token
    → cryptographically random token
       + hash stored server-side
```

`bcrypt` is primarily for passwords; do not introduce it merely because the application has OTPs.

---

# Phase 12 — OTP Verification

```text
OTP submitted
    ↓
Validate format
    ↓
Find active verification
    ↓
Check expiration
    ↓
Check attempt limit
    ↓
Compare protected OTP
    ↓
Mark email verified
    ↓
Create verification session
    ↓
Set secure cookie
```

Invalid attempts increment the attempt count.

Maximum attempts invalidate the OTP.

---

# Phase 13 — Global 30-Minute Verification Session

After successful OTP verification:

```text
Verified Email
      ↓
Create session
      ↓
30-minute expiry
      ↓
Secure HttpOnly cookie
```

The browser should hold only an opaque session token.

Never trust:

```text
localStorage.isVerified = true
```

The server must resolve the session.

---

# Phase 14 — Secure Cookie

Recommended characteristics:

```text
HttpOnly
Secure
SameSite=Lax/Strict
Path=/
Max-Age=1800
```

Conceptually:

```text
Cookie:
email_verification_session=<opaque-random-token>
```

Database:

```text
session_token_hash
email
expires_at
```

Prefer:

```text
Browser → raw token
Database → token hash
```

---

# Phase 15 — Session Validation

At protected form submission:

```text
Cookie
 ↓
Find session
 ↓
Validate token
 ↓
Check expiration
 ↓
Retrieve verified email
```

Then enforce:

```text
submittedEmail === verifiedSession.email
```

This prevents:

```text
Verify email A
      ↓
Submit form using email B
```

---

# Phase 16 — Global Verification Across Forms

Example:

```text
Contact Form
    ↓
user@example.com
    ↓
OTP
    ↓
30-min session
```

Then:

```text
Start a Project / RFQ
    ↓
Existing session found
    ↓
user@example.com ✓ Verified
```

No second OTP is required while the session is valid.

---

# Phase 17 — Reusable Email Verification UI

Create:

```text
EmailVerificationField
```

### Unverified

```text
Email
[ user@example.com ]

[ Verify Email ]
```

### Verified

```text
Email
[ user@example.com ] ✓ Verified
                         ✎
```

### Editing

```text
Email
[ newemail@example.com ]

[ Verify New Email ]
```

Use the same component in:

```text
ContactForm
RFQForm
Future forms
```

---

# Phase 18 — Changing the Verified Email

```text
Existing verified email
        ↓
Edit
        ↓
New email
        ↓
Compare with current verified email
        ↓
Different?
        ↓
Request OTP
        ↓
Verify new email
        ↓
Invalidate old session
        ↓
Create new session
```

A previously verified email must not automatically verify a newly entered email.

---

# Phase 19 — Global Verification Hook

Create:

```text
src/hooks/use-email-verification.ts
```

Expose workflow/state such as:

```text
email
isVerified
isLoading
sendOTP()
verifyOTP()
changeEmail()
refreshSession()
```

Both forms consume:

```text
ContactForm
    ↓
useEmailVerification()

RFQForm
    ↓
useEmailVerification()
```

Avoid duplicated OTP hooks unless requirements genuinely differ.

---

# Phase 20 — Verification Service

Create:

```text
src/lib/auth/email-verification.service.ts
```

Responsibilities:

```text
normalize email
request OTP
verify OTP
create session
validate session
invalidate session
change verified email
```

Keep business logic here rather than inside components or route handlers.

---

# Phase 21 — Form Submission

```text
User completes form
        ↓
React Hook Form
        ↓
Zod client validation
        ↓
POST /api/contact
or
POST /api/rfq
        ↓
Server-side Zod validation
        ↓
Validate email session
        ↓
Check submitted email
        ↓
Rate-limit
        ↓
Bot/security checks
        ↓
Business validation
        ↓
Persist submission
        ↓
Send business notification
        ↓
Send user confirmation
        ↓
Return success
```

---

# Phase 22 — Submission Persistence

The database is the source of truth.

Do not use:

```text
Form → SMTP → Done
```

Prefer:

```text
Form
 ↓
Validate
 ↓
Database
 ↓
Email notification
```

If:

```text
DB save ✓
Email send ✗
```

the submission still exists and can be retried.

For higher scale:

```text
Submission
 ↓
Database
 ↓
Queue/Event
 ↓
Email worker
 ↓
SMTP
```

A portfolio can initially use synchronous email sending if desired, while keeping `EmailService` replaceable with a queue later.

---

# Phase 23 — Email Templates

Create:

```text
src/lib/email/templates/
├── otp.ts
├── submission-received.ts
└── submission-confirmation.ts
```

### OTP

```text
Your verification code is {{otp}}.

It expires in {{minutes}} minutes.
```

### Business notification

```text
New Contact/RFQ received.

Name:
Email:
Company:
Project:
...
```

### User confirmation

```text
Thank you for contacting us.

Your request has been received.
We will review it and get back to you.
```

---

# Phase 24 — Email Service API

Application code should call:

```text
emailService.sendOTP()
emailService.sendSubmissionNotification()
emailService.sendSubmissionConfirmation()
```

Never put:

```text
smtp.sendMail(...)
```

inside every API route.

---

# Phase 25 — Rate Limiting

Use multiple dimensions where appropriate.

### OTP

```text
email + IP
```

### Form submissions

```text
IP
```

Starting values:

```text
OTP requests:
3 / 10 minutes / email

OTP attempts:
5 / OTP

Contact:
5–10 / hour / IP

RFQ:
3–5 / hour / IP
```

Tune based on real traffic.

For production/serverless systems, use a distributed rate-limit store instead of an in-memory `Map`.

---

# Phase 26 — Bot Protection

Start with:

```text
Honeypot
Rate limiting
Server-side validation
```

If spam becomes significant, add a dedicated CAPTCHA/anti-bot solution.

Never trust a frontend-only bot check.

---

# Phase 27 — Security Controls

Protect against:

```text
OTP abuse
Brute-force OTP attempts
Form spam
XSS
HTML injection
Email header injection
Oversized requests
Malicious filenames
Executable uploads
Invalid MIME types
Unexpected enum values
Malformed URLs
Session theft
Session tampering
```

Rules:

```text
Never expose secrets to the client.
Never expose SMTP credentials.
Never expose DB credentials.
Never trust client validation.
Never store plaintext OTPs.
Never store raw session tokens unnecessarily.
```

---

# Phase 28 — File Handling

For Contact/RFQ uploads:

```text
Browser
   ↓
Server validation
   ↓
Size validation
   ↓
MIME validation
   ↓
Extension validation
   ↓
Content/signature validation where practical
   ↓
Private storage
```

Do not store private BRDs in:

```text
/public
```

Recommended maximum:

```text
10 MB per file
```

Store metadata such as:

```text
file ID
safe filename
original filename
MIME type
size
storage key
submission ID
created_at
```

---

# Phase 29 — API Responsibilities

## `/api/auth/email/request-otp`

```text
validate email
normalize email
rate-limit
generate OTP
store protected OTP
send OTP
return controlled response
```

## `/api/auth/email/verify-otp`

```text
validate email + OTP
find verification
check expiry
check attempts
compare OTP
mark verified
create session
set secure cookie
return controlled response
```

## `/api/contact`

```text
validate request
validate files
validate session
validate submitted email
rate-limit
business validation
persist submission
send notification
send confirmation
```

## `/api/rfq`

Same structure plus RFQ-specific business rules.

---

# Phase 30 — Error Handling

Use controlled responses:

```text
400 → Invalid input
401/403 → Verification/session problem
413 → Request/file too large
429 → Too many requests
500 → Internal failure
```

Do not expose:

```text
database errors
SMTP credentials
stack traces
internal implementation details
```

to users.

---

# Phase 31 — SMTP Provider Independence

Use:

```text
Application
    ↓
EmailService Interface
    ↓
SMTP Adapter
    ↓
Provider
```

Configuration comes from `.env.local`.

Changing SMTP credentials/provider configuration should not require changes to:

```text
ContactForm
RFQForm
OTPModal
useEmailVerification
SubmissionService
```

Only provider/configuration code should change.

---

# Phase 32 — Database Provider Independence

Use:

```text
Service
   ↓
Repository Interface
   ↓
Provider Adapter
   ↓
Database
```

Changing:

```text
Firebase
```

to:

```text
PostgreSQL
```

should primarily involve the DB provider/repository implementation and configuration.

Forms, hooks, validation, and UI should remain unchanged.

---

# Phase 33 — Service Boundaries

Recommended services:

```text
EmailVerificationService
OTPService
SessionService
SubmissionService
EmailService
RateLimitService
FileStorageService
```

| Service | Responsibility |
|---|---|
| OTPService | Generate/protect/verify OTP |
| EmailVerificationService | Verification workflow |
| SessionService | Create/validate/invalidate sessions |
| SubmissionService | Contact/RFQ business workflow |
| EmailService | Application emails |
| RateLimitService | Abuse prevention |
| FileStorageService | Secure file storage |

---

# Phase 34 — Testing Architecture

```text
tests/
├── unit/
├── integration/
└── e2e/
```

## Unit

Test:

```text
OTP generation
OTP protection/comparison
email normalization
validation schemas
session expiry logic
rate-limit calculations
```

## Integration

Test:

```text
request OTP API
verify OTP API
session creation
cookie creation
Contact API
RFQ API
repositories
EmailService
RateLimitService
```

## E2E

Test:

```text
Contact → verify → submit → confirmation
RFQ → verify → submit → confirmation
```

---

# Phase 35 — Critical Test Cases

## OTP

```text
valid OTP
invalid OTP
expired OTP
maximum attempts
resend cooldown
maximum resends
new OTP handling
```

## Session

```text
valid session
expired session
invalid token
tampered token
email mismatch
email change invalidates old session
```

## Forms

```text
unverified email
verified email
auto-populated email
edit verified email
new email requires OTP
valid Contact submission
valid RFQ submission
invalid data
rate-limited request
invalid file
oversized file
```

## Email

```text
OTP sent
business notification sent
user confirmation sent
SMTP failure
DB success + email failure
retry behaviour where implemented
```

---

# Phase 36 — Agile Development Roadmap

## Sprint 1 — Foundation

- [ ] Define requirements
- [ ] Define email verification lifecycle
- [ ] Define session policy
- [ ] Define OTP limits
- [ ] Define rate limits
- [ ] Define DB entities
- [ ] Define provider interfaces
- [ ] Create folders
- [ ] Create environment configuration

## Sprint 2 — Database Layer

- [ ] Create DB abstraction
- [ ] Create repositories
- [ ] Verification repository
- [ ] Session repository
- [ ] Submission repository
- [ ] Rate-limit repository
- [ ] Add required indexes/constraints

## Sprint 3 — Email Layer

- [ ] Create email interface
- [ ] Create SMTP adapter
- [ ] Create EmailService
- [ ] OTP template
- [ ] Business notification template
- [ ] Confirmation template
- [ ] Test SMTP

## Sprint 4 — OTP System

- [ ] Normalize email
- [ ] Generate secure OTP
- [ ] Protect OTP
- [ ] Store verification
- [ ] Send OTP
- [ ] Verify OTP
- [ ] Attempt limits
- [ ] Resend cooldown
- [ ] Expiration

## Sprint 5 — Global Session

- [ ] Generate secure session token
- [ ] Store token hash
- [ ] Create 30-minute session
- [ ] Set HttpOnly cookie
- [ ] Validate session
- [ ] Expire session
- [ ] Invalidate session
- [ ] Implement global hook

## Sprint 6 — Reusable Verification UI

- [ ] EmailVerificationField
- [ ] OTPModal
- [ ] VerifiedBadge
- [ ] EditEmailButton
- [ ] Auto-populate verified email
- [ ] New-email verification workflow

## Sprint 7 — Contact Submission

- [ ] Client validation
- [ ] Server validation
- [ ] Session validation
- [ ] File validation
- [ ] Rate limiting
- [ ] Persist submission
- [ ] Business notification
- [ ] User confirmation

## Sprint 8 — RFQ Submission

- [ ] Client validation
- [ ] Server validation
- [ ] Date relationship validation
- [ ] Project type validation
- [ ] Budget range validation
- [ ] Tech-stack validation
- [ ] BRD validation
- [ ] Rate limiting
- [ ] Persist submission
- [ ] Business notification
- [ ] User confirmation

## Sprint 9 — Security Hardening

- [ ] Rate limiting
- [ ] Honeypot
- [ ] CAPTCHA if required
- [ ] Secure cookies
- [ ] Session token protection
- [ ] OTP protection
- [ ] File validation
- [ ] Private storage
- [ ] Request size limits
- [ ] Email header protection
- [ ] Security review

## Sprint 10 — Testing

- [ ] Unit tests
- [ ] Integration tests
- [ ] API tests
- [ ] Email service tests
- [ ] Repository tests
- [ ] Session tests
- [ ] Rate-limit tests
- [ ] Contact E2E
- [ ] RFQ E2E
- [ ] Failure scenarios

## Sprint 11 — Production Hardening

- [ ] Verify production environment variables
- [ ] Verify SMTP configuration
- [ ] Verify DB configuration
- [ ] Verify private storage
- [ ] Verify distributed rate limiting
- [ ] Verify HTTPS cookies
- [ ] Verify error handling
- [ ] Verify logs
- [ ] Verify retry/failure handling
- [ ] Verify monitoring
- [ ] Run final security review

---

# Phase 37 — Definition of Done

```text
✓ Email verification is global
✓ OTP is short-lived
✓ OTP attempts are limited
✓ OTP resend is limited
✓ Email is normalized
✓ Verified email creates a 30-minute session
✓ Session uses secure HttpOnly cookie
✓ Session token is protected server-side
✓ Contact and RFQ share verification
✓ Verified email auto-populates
✓ User can edit/change email
✓ New email requires verification
✓ Previous session is invalidated appropriately
✓ Submitted email matches verified session
✓ Contact/RFQ data is persisted
✓ Business receives notification
✓ User receives confirmation
✓ SMTP is provider-independent
✓ Database is provider-independent
✓ Rate limiting is reusable
✓ File uploads are secured
✓ BRDs are private
✓ Unit tests exist
✓ Integration tests exist
✓ E2E tests exist
✓ Secrets are environment-based
✓ Internal errors are not exposed
```

---

# Phase 38 — Final Architecture

```text
                             USER
                              │
                  ┌───────────┴───────────┐
                  ↓                       ↓
             Contact Form             RFQ Form
                  │                       │
                  └───────────┬───────────┘
                              ↓
                 useEmailVerification()
                              ↓
                     Verification API
                              ↓
                 EmailVerificationService
                         ↙          ↘
                        ↓            ↓
                   OTPService    SessionService
                        │            │
                        ↓            ↓
                   DB Repository  DB Repository
                        │            │
                        └─────┬──────┘
                              ↓
                         DB Provider
                              ↓
                    Firebase/PostgreSQL/...


OTPService
    ↓
EmailService
    ↓
SMTP Adapter
    ↓
SMTP Provider


After verification:

Contact/RFQ
    ↓
React Hook Form
    ↓
Zod
    ↓
API Route
    ↓
Server-side Zod
    ↓
Session Validation
    ↓
Email Match Validation
    ↓
Rate Limit
    ↓
Bot/Security Checks
    ↓
Business Rules
    ↓
SubmissionService
    ↓
Submission Repository
    ↓
Database
    ↓
EmailService
    ├──→ Business Notification
    └──→ User Confirmation
```

---

# Core Principles

1. **One global email-verification system.**
2. **One reusable 30-minute verification session.**
3. **One reusable `useEmailVerification()` hook.**
4. **One reusable email-verification UI component.**
5. **One EmailService for application-level email communication.**
6. **One SMTP adapter boundary.**
7. **One database abstraction boundary.**
8. **One repository layer between services and DB.**
9. **One reusable rate-limit service.**
10. **Zod remains the validation source of truth.**
11. **Client validation is UX; server validation is security.**
12. **Never trust client-side verification state.**
13. **Always compare submitted email with the verified session email.**
14. **Never store plaintext OTPs.**
15. **Use the appropriate protection mechanism for each secret; bcrypt/Argon2id for passwords, not automatically for OTPs.**
16. **Use cryptographically random session tokens and protect them server-side.**
17. **Use secure HttpOnly cookies for verification sessions.**
18. **Do not expose private BRDs through `/public`.**
19. **Database persistence should not depend on successful email delivery.**
20. **Design provider boundaries so SMTP and DB can be replaced independently.**
21. **Use services for business logic, repositories for persistence, adapters for providers, and components/hooks for UI workflow.**
22. **Test at unit, integration, and E2E levels.**
23. **Build incrementally through the Agile phases.**

---

# Target Result

```text
Global Email Identity
        +
OTP Verification
        +
30-Minute Secure Session
        +
Reusable Components/Hooks
        +
Provider-Independent SMTP
        +
Provider-Independent Database
        +
Rate Limiting
        +
Secure File Handling
        +
Submission Persistence
        +
Business Notification
        +
User Confirmation
        +
Automated Testing
        =
Clean, Secure, Scalable Form Communication Architecture
```
