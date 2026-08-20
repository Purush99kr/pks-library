# HLD — Secure & Scalable Form + Email Communication System

## Next.js + TypeScript + React Hook Form + Zod + SMTP + Database

---

# 1. System Objective

Build a reusable communication platform for:

- Contact Form
- Start a Project / RFQ Form
- Global email verification
- 30-minute verified-email session
- Secure cookies
- OTP lifecycle
- Rate limiting
- File/BRD uploads
- Database persistence
- Business notification emails
- User confirmation emails
- Provider-independent SMTP
- Provider-independent database
- Reusable components/hooks/services
- Unit, integration and E2E testing

Core principle:

```text
New Form
   ↓
Reuse validation
   ↓
Reuse email verification
   ↓
Reuse security
   ↓
Reuse submission service
   ↓
Reuse email service
   ↓
Reuse database abstraction
```

---

# 2. High-Level Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                         CLIENT                              │
│                                                             │
│  Contact Form                  RFQ Form                    │
│      │                            │                         │
│      └──────────────┬─────────────┘                         │
│                     ↓                                       │
│          React Hook Form + Zod                              │
│                     │                                       │
│          useEmailVerification()                            │
│                     │                                       │
│          Email Verification UI                             │
└─────────────────────┼───────────────────────────────────────┘
                      │ HTTPS
                      ↓
┌─────────────────────────────────────────────────────────────┐
│                     NEXT.JS SERVER                          │
│                                                             │
│  API Routes                                                 │
│      │                                                      │
│      ├── /api/auth/email/request-otp                        │
│      ├── /api/auth/email/verify-otp                         │
│      ├── /api/contact                                       │
│      └── /api/rfq                                           │
│                     │                                       │
│                     ↓                                       │
│              Validation Layer                               │
│                     │                                       │
│                     ↓                                       │
│               Security Layer                                │
│          ┌──────────┼──────────┐                            │
│          ↓          ↓          ↓                            │
│      Session     Rate       Upload                          │
│      Security    Limit      Security                        │
│          │          │          │                            │
│          └──────────┼──────────┘                            │
│                     ↓                                       │
│               Service Layer                                 │
│          ┌──────────┼─────────────┐                         │
│          ↓          ↓             ↓                         │
│      OTPService  Submission   EmailService                  │
│                     Service                                 │
│          │          │             │                         │
│          └──────────┼─────────────┘                         │
│                     ↓                                       │
│             Repository Layer                                │
└─────────────────────┼───────────────────────────────────────┘
                      │
          ┌───────────┼───────────────────┐
          ↓           ↓                   ↓
   ┌────────────┐ ┌────────────┐   ┌──────────────┐
   │ Database   │ │ SMTP       │   │ File Storage │
   │ Provider   │ │ Provider   │   │ Provider     │
   └────────────┘ └────────────┘   └──────────────┘
```

---

# 3. Architectural Layers

```text
UI
 ↓
API
 ↓
Validation
 ↓
Security
 ↓
Service
 ↓
Repository
 ↓
Provider
```

## UI

Responsible for:

```text
Rendering
Form state
Client validation
Loading/error/success states
OTP modal
Email verification UI
```

Never expose:

```text
SMTP credentials
DB credentials
session secrets
server business rules
provider secrets
```

## API

HTTP boundary only:

```text
receive → validate → call service → handle error → respond
```

Keep route handlers thin.

## Validation

Use Zod:

```text
common.schema.ts
contact.schema.ts
rfq.schema.ts
otp.schema.ts
```

Principle:

```text
Client validation = UX
Server validation = Security
```

## Security

Centralize:

```text
rate limiting
session protection
hash/protection helpers
sanitization
request limits
file security
```

## Service

Contains business workflows:

```text
OTPService
SessionService
EmailVerificationService
SubmissionService
ContactService
RFQService
EmailService
RateLimitService
FileStorageService
```

## Repository

Owns database persistence.

## Provider

Owns provider-specific SDK/API details.

---

# 4. Complete Project Structure

```text
src/
│
├── app/
│   └── api/
│       ├── auth/
│       │   └── email/
│       │       ├── request-otp/
│       │       │   └── route.ts
│       │       └── verify-otp/
│       │           └── route.ts
│       ├── contact/
│       │   └── route.ts
│       └── rfq/
│           └── route.ts
│
├── components/
│   ├── forms/
│   │   ├── contact-form.tsx
│   │   ├── rfq-form.tsx
│   │   ├── email-field.tsx
│   │   └── file-upload.tsx
│   │
│   ├── auth/
│   │   └── email-verification/
│   │       ├── email-verification.tsx
│   │       ├── email-input.tsx
│   │       ├── otp-modal.tsx
│   │       ├── verified-badge.tsx
│   │       └── edit-email-button.tsx
│   │
│   └── common/
│       ├── form-error.tsx
│       ├── submit-button.tsx
│       └── submission-dialog.tsx
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
│   │   ├── email.service.ts
│   │   ├── smtp.ts
│   │   ├── types.ts
│   │   ├── errors.ts
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
│   │       ├── firebase/
│   │       └── postgres/
│   │
│   ├── storage/
│   │   ├── file-storage.service.ts
│   │   └── providers/
│   │       └── ...
│   │
│   ├── security/
│   │   ├── hash.ts
│   │   ├── rate-limit.ts
│   │   ├── sanitize.ts
│   │   ├── file-security.ts
│   │   └── constants.ts
│   │
│   ├── validations/
│   │   ├── common.schema.ts
│   │   ├── contact.schema.ts
│   │   ├── rfq.schema.ts
│   │   └── otp.schema.ts
│   │
│   ├── submissions/
│   │   ├── submission.service.ts
│   │   ├── contact.service.ts
│   │   └── rfq.service.ts
│   │
│   └── forms/
│       ├── constants.ts
│       └── types.ts
│
├── config/
│   ├── env.ts
│   ├── email.ts
│   ├── database.ts
│   └── security.ts
│
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```

---

# 5. Global Email Verification Architecture

Email verification is independent of forms.

```text
                 Email Verification
                         │
            ┌────────────┴────────────┐
            ↓                         ↓
       Contact Form              RFQ Form
            │                         │
            └────────────┬────────────┘
                         ↓
              useEmailVerification()
                         ↓
                  Verification API
                         ↓
             EmailVerificationService
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
        OTPService             SessionService
             │                       │
             └───────────┬───────────┘
                         ↓
                   DB Repository
```

Future forms reuse the same mechanism.

---

# 6. OTP Lifecycle

```text
User enters email
       ↓
Normalize
       ↓
Validate
       ↓
Rate-limit OTP request
       ↓
Generate secure OTP
       ↓
Protect OTP representation
       ↓
Store verification record
       ↓
Send OTP email
       ↓
User enters OTP
       ↓
Validate OTP
       ↓
Check expiry/attempts
       ↓
Compare protected OTP
       ↓
Mark email verified
       ↓
Create session
       ↓
Set secure cookie
```

Recommended starting configuration:

```text
OTP length             = 6 digits
OTP lifetime           = 5 minutes
Maximum attempts       = 5
Resend cooldown        = 60 seconds
Maximum resends        = 3
Session lifetime       = 30 minutes
```

Make values configurable.

---

# 7. OTP Security

Never store plaintext OTPs.

```text
OTP
 ↓
HMAC / secure hash
 ↓
Database
```

Verification:

```text
Submitted OTP
 ↓
Same protection mechanism
 ↓
Compare
```

Invalidate after:

```text
success
expiration
maximum attempts
```

---

# 8. Hashing Strategy

Do not use bcrypt for every secret.

```text
Passwords
    → Argon2id / bcrypt

Short-lived OTP
    → HMAC / secure hash

Session token
    → cryptographically random token
       + server-side hash
```

Use the protection mechanism appropriate to the secret.

---

# 9. Session Architecture

After successful OTP verification:

```text
Verified Email
      ↓
Generate cryptographically random token
      ↓
Hash token
      ↓
Store hash in DB
      ↓
Set raw token in HttpOnly cookie
```

Database:

```text
email
session_token_hash
expires_at
created_at
```

Cookie:

```text
email_verification_session=<opaque-token>
```

Cookie settings:

```text
HttpOnly
Secure
SameSite=Lax/Strict
Path=/
Max-Age=1800
```

Never use client state/localStorage as the security authority.

---

# 10. Global Verification Across Forms

```text
Contact
  ↓
Verify user@example.com
  ↓
30-min session
  ↓
Open RFQ
  ↓
Read same session
  ↓
user@example.com ✓ Verified
```

No second OTP while the session is valid.

---

# 11. Email Change Flow

```text
Existing verified email
        ↓
Edit
        ↓
New email
        ↓
Compare
        ↓
If different
        ↓
Request OTP
        ↓
Verify new email
        ↓
Invalidate old session
        ↓
Create new session
```

A newly entered email must never inherit verification from the old email.

---

# 12. Contact Form Flow

Fields:

```text
name
email
subject
phone
projectDetails
file
```

```text
Contact Form
    ↓
React Hook Form
    ↓
Zod
    ↓
Email Verification
    ↓
POST /api/contact
    ↓
Server Zod
    ↓
Session validation
    ↓
Email/session match
    ↓
Rate limit
    ↓
File validation
    ↓
ContactService
    ↓
SubmissionRepository
    ↓
Database
    ↓
EmailService
    ├──→ Business notification
    └──→ User confirmation
```

---

# 13. RFQ Form Flow

Fields:

```text
name
email
company
projectType
techStack
requiredFeatures
budgetRange
expectedStartDate
expectedEndDate
referenceWebsite
additionalDetails
brdDocuments
```

```text
RFQ Form
    ↓
React Hook Form
    ↓
Zod
    ↓
Global Email Verification
    ↓
POST /api/rfq
    ↓
Server Validation
    ↓
Session Validation
    ↓
Rate Limiting
    ↓
Business Rules
    ↓
File Security
    ↓
RFQService
    ↓
SubmissionRepository
    ↓
Database
    ↓
EmailService
```

Business validation includes:

```text
startDate < endDate
allowed project type
allowed budget range
allowed tech-stack values
valid reference URL
```

---

# 14. File/BRD Security

```text
Browser
  ↓
Basic client validation
  ↓
Server
  ↓
Size validation
  ↓
Extension validation
  ↓
MIME validation
  ↓
Content/signature validation where practical
  ↓
Private storage
  ↓
Metadata persisted
```

Suggested starting maximum:

```text
10 MB per file
```

Example allowed BRD formats:

```text
.pdf
.doc
.docx
```

Optionally:

```text
.ppt
.pptx
.xls
.xlsx
```

Only allow formats actually required.

Never trust filename extension or MIME type alone.

Never expose private BRDs from `/public`.

---

# 15. Submission Architecture

Use one shared submission layer:

```text
ContactForm
    ↓
SubmissionService
    ↓
Contact-specific rules
```

```text
RFQForm
    ↓
SubmissionService
    ↓
RFQ-specific rules
```

Shared workflow:

```text
session
 ↓
rate limit
 ↓
security
 ↓
persistence
 ↓
email communication
```

---

# 16. Database Architecture

```text
Application
     ↓
Repository Interface
     ↓
DB Adapter
     ↓
Database Provider
```

Recommended repositories:

```text
VerificationRepository
SessionRepository
SubmissionRepository
RateLimitRepository
```

Logical entities:

```text
email_verifications
email_sessions
form_submissions
rate_limits
audit_logs
```

The form should never directly call a database SDK.

---

# 17. SMTP Architecture

```text
Application
     ↓
EmailService
     ↓
Email Interface
     ↓
SMTP Adapter
     ↓
SMTP Provider
```

Application API:

```text
sendVerificationOTP()
sendSubmissionNotification()
sendSubmissionConfirmation()
```

The application should not contain:

```text
smtp.sendMail(...)
```

inside individual forms/routes.

---

# 18. Provider Independence

```text
                    Application
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
   DB Interface    Email Interface   Storage Interface
        ↓               ↓                ↓
    DB Adapter      SMTP Adapter    Storage Adapter
        ↓               ↓                ↓
     Provider        Provider         Provider
```

Configuration:

```text
.env.local
```

controls:

```text
DB_PROVIDER
DB_URL

EMAIL_PROVIDER
SMTP_HOST
SMTP_PORT
SMTP_USER
SMTP_PASSWORD

RATE_LIMIT_PROVIDER

STORAGE_PROVIDER
STORAGE_BUCKET
```

Changing providers should not require UI rewrites.

---

# 19. Request Processing Pipeline

Every protected submission should follow:

```text
HTTP Request
    ↓
Request size check
    ↓
Parse
    ↓
Schema validation
    ↓
Normalize
    ↓
Session validation
    ↓
Email/session match
    ↓
Rate limiting
    ↓
Bot checks
    ↓
File security
    ↓
Business validation
    ↓
Service
    ↓
Repository
    ↓
Database
    ↓
Email communication
    ↓
Response
```

---

# 20. Database-First Submission Strategy

Recommended:

```text
Validate
   ↓
Persist submission
   ↓
Send emails
```

Database is the source of truth.

If:

```text
DB save ✓
Email send ✗
```

the submission remains stored.

At higher scale:

```text
Submission
 ↓
Database
 ↓
Outbox/Queue
 ↓
Email Worker
 ↓
SMTP
```

---

# 21. Rate Limiting

Use a reusable:

```text
RateLimitService
```

Suggested dimensions:

```text
OTP → email + IP
Contact → IP
RFQ → IP + email
```

Starting limits:

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

For production/serverless, prefer a distributed store over an in-memory `Map`.

---

# 22. Bot Protection

Layer:

```text
Client validation
      ↓
Honeypot
      ↓
Rate limiting
      ↓
Server validation
      ↓
Optional CAPTCHA
```

Use CAPTCHA if actual spam volume requires it.

---

# 23. Error Architecture

Define controlled error categories:

```text
ValidationError
VerificationError
RateLimitError
FileValidationError
BusinessRuleError
ProviderError
InternalError
```

Map to:

```text
400 → invalid input
401/403 → verification/session issue
413 → request/file too large
429 → rate limit
500 → internal/provider failure
```

Never expose:

```text
stack traces
DB credentials
SMTP credentials
provider internals
```

---

# 24. Email Templates

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

### Confirmation

```text
Thank you for contacting us.

Your request has been received.
```

---

# 25. Observability

Log operational events:

```text
OTP requested
OTP verification success/failure
Session created/expired
Form submitted
Submission persisted
Email sent/failed
Rate limit triggered
File rejected
```

Never log:

```text
OTP plaintext
SMTP password
DB password
session token
unnecessary sensitive form contents
```

Use:

```text
requestId
```

to correlate API, DB, email, and error events.

---

# 26. Retry Architecture

Retry only transient failures.

Potential retry candidates:

```text
temporary SMTP failure
temporary network/provider failure
```

Do not blindly retry:

```text
invalid input
invalid OTP
rate limit
authorization failure
```

At higher scale:

```text
Database
   ↓
Outbox
   ↓
Queue
   ↓
Worker
   ↓
Retry
   ↓
Dead Letter Queue
```

---

# 27. API Responsibilities

## `/api/auth/email/request-otp`

```text
validate email
normalize
rate-limit
generate OTP
store protected OTP
send OTP
return controlled response
```

## `/api/auth/email/verify-otp`

```text
validate OTP
find verification
check expiry
check attempts
compare OTP
mark verified
create session
set cookie
return controlled response
```

## `/api/contact`

```text
validate
validate session
check email/session match
rate-limit
validate file
apply business rules
persist
send notification
send confirmation
```

## `/api/rfq`

Same pattern with RFQ-specific business rules.

---

# 28. Service Responsibilities

| Service | Responsibility |
|---|---|
| OTPService | Generate/protect/verify OTP |
| SessionService | Create/validate/invalidate sessions |
| EmailVerificationService | Verification workflow |
| SubmissionService | Generic submission workflow |
| ContactService | Contact-specific rules |
| RFQService | RFQ-specific rules |
| EmailService | Application emails |
| RateLimitService | Abuse prevention |
| FileStorageService | Secure file operations |

---

# 29. Component Responsibilities

```text
EmailVerificationField
    → email UI/state

OTPModal
    → OTP input UI

VerifiedBadge
    → verified state

EditEmailButton
    → email edit workflow

FileUpload
    → upload UI

ContactForm
    → Contact UI

RFQForm
    → RFQ UI
```

Components must not implement:

```text
OTP hashing
DB operations
SMTP communication
session creation
```

---

# 30. Testing Architecture

```text
tests/
├── unit/
├── integration/
└── e2e/
```

## Unit

```text
OTP generation
OTP protection
email normalization
Zod schemas
session expiry
rate-limit calculations
```

## Integration

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

```text
Contact → verify → submit → confirmation
RFQ → verify → submit → confirmation
```

---

# 31. Critical Test Matrix

| Area | Tests |
|---|---|
| Email | invalid email, normalization |
| OTP | valid, invalid, expired |
| OTP | max attempts, cooldown, resends |
| Session | valid, expired, invalid token |
| Session | tampering, email mismatch |
| Forms | invalid/unverified/verified |
| Forms | edit email/new verification |
| Contact | valid submission |
| RFQ | valid submission/date rules |
| File | extension/MIME/size |
| Security | rate limit/bot |
| Email | success/provider failure |
| DB | persistence/provider failure |
| Integration | DB success + email failure |
| E2E | complete Contact flow |
| E2E | complete RFQ flow |

---

# 32. Agile Implementation Order

```text
1. HLD / architecture
      ↓
2. Environment/config
      ↓
3. Validation foundation
      ↓
4. DB abstraction
      ↓
5. SMTP abstraction
      ↓
6. OTP service
      ↓
7. Session service
      ↓
8. Global verification UI
      ↓
9. Rate limiting
      ↓
10. File security/storage
      ↓
11. Contact submission
      ↓
12. RFQ submission
      ↓
13. Email templates
      ↓
14. Testing
      ↓
15. Security hardening
      ↓
16. Production deployment
      ↓
17. Monitoring/optimization
```

---

# 33. Production Definition of Done

```text
✓ Global email verification
✓ Short-lived protected OTP
✓ OTP attempt/resend limits
✓ Normalized email
✓ 30-minute session
✓ Secure HttpOnly cookie
✓ Server-side session validation
✓ Verified email auto-population
✓ Email editing + re-verification
✓ Submitted email bound to session
✓ Client + server validation
✓ Contact persistence
✓ RFQ persistence
✓ Private BRD storage
✓ Distributed production rate limiting
✓ SMTP abstraction
✓ DB abstraction
✓ Storage abstraction
✓ Reusable email templates
✓ Business notification
✓ User confirmation
✓ Submission survives email failure
✓ Unit tests
✓ Integration tests
✓ Critical E2E tests
✓ Secrets excluded from logs/client
✓ Environment-driven configuration
✓ Provider replacement without UI rewrites
```

---

# 34. Complete Contact Flow

```text
USER
 │
 ↓
Contact Form
 │
 ↓
EmailVerificationField
 │
 ↓
useEmailVerification()
 │
 ├── Existing session?
 │       │
 │      YES ───────────────────┐
 │       │                     │
 │      NO                     │
 │       ↓                     │
 │   Request OTP               │
 │       ↓                     │
 │   EmailService              │
 │       ↓                     │
 │      SMTP                   │
 │       ↓                     │
 │   User enters OTP           │
 │       ↓                     │
 │   Verify OTP                │
 │       ↓                     │
 │   Create session            │
 │       └─────────────────────┘
 │
 ↓
Complete form
 │
 ↓
POST /api/contact
 │
 ↓
Server validation
 │
 ↓
Session validation
 │
 ↓
Rate limit
 │
 ↓
File security
 │
 ↓
ContactService
 │
 ↓
SubmissionRepository
 │
 ↓
Database
 │
 ↓
EmailService
 ├────────→ Business
 └────────→ Customer
```

---

# 35. Complete RFQ Flow

```text
USER
 │
 ↓
RFQ Form
 │
 ↓
EmailVerificationField
 │
 ↓
Global verified session
 │
 ↓
Complete RFQ
 │
 ↓
Upload BRD
 │
 ↓
POST /api/rfq
 │
 ↓
Server validation
 │
 ↓
Session validation
 │
 ↓
Rate limit
 │
 ↓
File security
 │
 ↓
RFQ business rules
 │
 ↓
RFQService
 │
 ├──→ FileStorageService
 │
 └──→ SubmissionRepository
             ↓
          Database
             ↓
        EmailService
          ├──────→ Business
          └──────→ Customer
```

---

# 36. Scalability Path

## Stage 1 — Portfolio

```text
Next.js
+
Database
+
SMTP
+
Private Storage
+
Distributed Rate Limit
```

Synchronous email may be acceptable.

## Stage 2 — Higher Traffic

Add:

```text
Outbox
Queue
Background email worker
Monitoring
Centralized logging
```

## Stage 3 — Larger System

```text
Web Application
      ↓
API
      ↓
Services
      ↓
Database
      ↓
Outbox
      ↓
Message Queue
      ↓
Workers
      ├── Email
      ├── File processing
      └── Notifications
```

---

# 37. Final HLD Mental Model

Think in five independent domains:

```text
                    FORM PLATFORM
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   VALIDATION         IDENTITY          SUBMISSION
        │                 │                 │
      Zod           Email + OTP        Contact/RFQ
                        │                 │
                     Session              │
                        │                 │
                        └────────┬────────┘
                                 ↓
                              SECURITY
                                 │
                   ┌─────────────┼─────────────┐
                   ↓             ↓             ↓
               Rate Limit     Cookies       Files
                   │
                   └─────────────┬─────────────┘
                                 ↓
                           COMMUNICATION
                                 │
                         ┌───────┴────────┐
                         ↓                ↓
                       SMTP             DB
                         │                │
                         ↓                ↓
                    Notification      Persistence
```

The design rule:

```text
Forms know WHAT they collect.

Validation knows WHAT is structurally valid.

Services know WHAT the business does.

Repositories know HOW data is persisted.

Adapters know WHICH external provider is used.

Components/hooks know HOW the user interacts.

Security is enforced on the server.

SMTP/DB/storage providers are replaceable.
```

---

# 38. Final System

```text
                     ┌──────────────────────┐
                     │        USER          │
                     └──────────┬───────────┘
                                │
                ┌───────────────┴───────────────┐
                ↓                               ↓
        ┌───────────────┐               ┌───────────────┐
        │ Contact Form  │               │   RFQ Form    │
        └───────┬───────┘               └───────┬───────┘
                │                               │
                └───────────────┬───────────────┘
                                ↓
                    ┌────────────────────┐
                    │ Email Verification │
                    │  Hook + Component  │
                    └─────────┬──────────┘
                              ↓
                    ┌────────────────────┐
                    │ Verification APIs  │
                    └─────────┬──────────┘
                              ↓
                 ┌──────────────────────────┐
                 │ EmailVerificationService│
                 └────────────┬─────────────┘
                              ↓
                  ┌───────────┴───────────┐
                  ↓                       ↓
             OTPService             SessionService
                  │                       │
                  └───────────┬───────────┘
                              ↓
                     Repository Layer
                              ↓
                         DB Provider
                              │
                         Verified ✓
                              │
                              ↓
                    Contact / RFQ Submit
                              │
                              ↓
                       API Validation
                              │
                              ↓
                         Security
                  ┌───────────┼───────────┐
                  ↓           ↓           ↓
              Session      Rate       File Security
              Check        Limit
                  └───────────┼───────────┘
                              ↓
                      Business Service
                              ↓
                    Submission Repository
                              ↓
                         Database
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
               EmailService       FileStorage
                    │
               SMTP Adapter
                    │
             ┌──────┴───────┐
             ↓              ↓
        Business         Customer
       Notification     Confirmation
```
