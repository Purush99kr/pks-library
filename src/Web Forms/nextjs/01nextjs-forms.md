# Scalable Form Validation, Security & Testing Guide
## Next.js + TypeScript + React Hook Form + Zod

> Scope: Portfolio website with two production forms:
> 1. Contact Form
> 2. Start a Project / RFQ Form
>
> Goal: Build the forms using a reusable, secure, testable architecture that can scale to additional forms later.

---

# Phase 1 — Define the Form Contracts

## 1.1 Contact Form

Fields:

- `name` — required
- `email` — required
- `subject` — required
- `phone` — optional
- `projectDetails` — required
- `file` — optional

Recommended limits:

| Field | Rule |
|---|---|
| Name | 2–100 characters |
| Email | Valid email, max 254 characters |
| Subject | 3–150 characters |
| Phone | 7–20 characters; optional |
| Project Details | 20–5,000 characters |
| File | Optional, max 10 MB |

Recommended document/image extensions:

- `.pdf`
- `.doc`
- `.docx`
- `.txt`
- `.png`
- `.jpg`
- `.jpeg`
- `.webp`

If the contact form only needs documents, prefer:

- `.pdf`
- `.doc`
- `.docx`
- `.txt`

---

## 1.2 Start a Project / RFQ Form

Fields:

- `name` — required
- `email` — required
- `companyBusiness` — required
- `projectType` — required
- `techStack` — required/controlled values
- `requiredFeatures` — required
- `budgetRange` — required
- `expectedStartDate` — required
- `expectedEndDate` — required
- `referenceWebsite` — optional
- `additionalDetails` — optional
- `brdDocuments` — optional

Business rules:

- Start date must be before end date.
- Project type must be from an allowed set.
- Budget range must be from an allowed set.
- Controlled tech-stack values should be validated.
- Reference website must be a valid URL when supplied.
- BRD documents must pass file validation.

---

# Phase 2 — Establish the Architecture

Use this validation pipeline:

```text
User Input
    ↓
React Hook Form
    ↓
Zod Schema
    ↓
Client Validation
    ↓
Next.js API / Server Action
    ↓
Zod Validation Again
    ↓
File Validation
    ↓
Rate Limit / Bot Protection
    ↓
Sanitization / Business Rules
    ↓
Email / Database / Storage
    ↓
Controlled Response
```

## Core principle

Client-side validation is for **UX**.

Server-side validation is mandatory for **security and correctness**.

Never trust:

```text
Browser → API → Trust data
```

Always use:

```text
Browser → API → Validate → Secure → Process
```

---

# Phase 3 — Create the Project Structure

Recommended structure:

```text
src/
├── app/
│   ├── api/
│   │   ├── contact/
│   │   │   └── route.ts
│   │   └── rfq/
│   │       └── route.ts
│   │
│   └── ...
│
├── components/
│   └── forms/
│       ├── contact-form.tsx
│       ├── rfq-form.tsx
│       │
│       └── fields/
│           ├── form-input.tsx
│           ├── form-textarea.tsx
│           ├── form-select.tsx
│           ├── form-multi-select.tsx
│           ├── form-date-picker.tsx
│           ├── form-file-upload.tsx
│           └── form-error.tsx
│
├── hooks/
│   ├── use-contact-form.ts
│   └── use-rfq-form.ts
│
├── lib/
│   ├── validations/
│   │   ├── common.schema.ts
│   │   ├── contact.schema.ts
│   │   ├── rfq.schema.ts
│   │   └── file.schema.ts
│   │
│   ├── security/
│   │   ├── rate-limit.ts
│   │   ├── honeypot.ts
│   │   └── sanitize.ts
│   │
│   ├── forms/
│   │   ├── constants.ts
│   │   └── types.ts
│   │
│   └── ...
│
└── tests/
    ├── unit/
    │   └── validations/
    │       ├── common.schema.test.ts
    │       ├── contact.schema.test.ts
    │       ├── rfq.schema.test.ts
    │       └── file.schema.test.ts
    │
    ├── integration/
    │   ├── contact-api.test.ts
    │   └── rfq-api.test.ts
    │
    └── e2e/
        ├── contact-form.spec.ts
        └── rfq-form.spec.ts
```

---

# Phase 4 — Build Reusable Validation Primitives

Create common validation rules in:

```text
src/lib/validations/common.schema.ts
```

Reusable primitives should cover:

```text
name
email
phone
company/business
subject
text
URL
date
budget
```

Conceptually:

```text
nameSchema
emailSchema
phoneSchema
urlSchema
textSchema
```

These become the foundation for every future form.

## Principle

Do not create a completely separate implementation of common rules for every form.

Instead:

```text
Common Validation Rules
          ↓
Form-Specific Schemas
```

---

# Phase 5 — Build Form-Specific Schemas

## 5.1 Contact Schema

File:

```text
src/lib/validations/contact.schema.ts
```

Structure:

```text
name
email
subject
phone
projectDetails
file
```

Compose reusable rules from `common.schema.ts`.

The contact schema should add only contact-specific requirements.

---

## 5.2 RFQ Schema

File:

```text
src/lib/validations/rfq.schema.ts
```

Structure:

```text
name
email
companyBusiness
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

Use schema-level validation for relationships between fields.

Example:

```text
expectedStartDate < expectedEndDate
```

This is important because both dates may individually be valid while the overall form is logically invalid.

---

# Phase 6 — Centralize Form Constants

File:

```text
src/lib/forms/constants.ts
```

Keep controlled values here:

```text
PROJECT_TYPES
TECH_STACKS
BUDGET_RANGES
ALLOWED_FILE_TYPES
MAX_FILE_SIZE
MAX_TEXT_LENGTHS
```

Example conceptual structure:

```text
PROJECT_TYPES
    Website
    Web Application
    E-commerce
    Portfolio
    CMS
    Other

BUDGET_RANGES
    < ₹25K
    ₹25K–₹50K
    ₹50K–₹1L
    ₹1L+
```

Do not rely only on frontend dropdowns.

The server must validate these values again.

---

# Phase 7 — Infer TypeScript Types from Zod

Avoid maintaining separate validation and type definitions when possible.

Use the schema as the source of truth:

```text
Zod Schema
    ↓
z.infer
    ↓
TypeScript Form Type
```

Conceptually:

```text
ContactFormValues = inferred from contactSchema
RFQFormValues     = inferred from rfqSchema
```

This prevents the validation rules and TypeScript types from drifting apart.

---

# Phase 8 — Integrate React Hook Form

Each form should use:

```text
React Hook Form
+
zodResolver
+
Form-specific Zod schema
```

Architecture:

```text
ContactForm
    ↓
contactSchema
    ↓
useForm<ContactFormValues>()

RFQForm
    ↓
rfqSchema
    ↓
useForm<RFQFormValues>()
```

Reusable form components should handle common UI behaviour:

```text
FormInput
FormTextarea
FormSelect
FormMultiSelect
FormDatePicker
FormFileUpload
FormError
```

This avoids duplicating field-level UI logic.

---

# Phase 9 — Implement File Validation

Create:

```text
src/lib/validations/file.schema.ts
```

Validate:

```text
1. File exists when optional field is supplied
2. File size
3. MIME type
4. Extension
5. Filename
6. File content/signature where practical
```

Recommended maximum:

```text
10 MB per file
```

Never trust only:

```text
filename.pdf
```

or:

```text
file.type
```

because browser-provided metadata can be manipulated.

Reject executable/dangerous types.

Do not allow arbitrary uploads.

---

# Phase 10 — Secure File Storage

Do not store uploaded files directly in:

```text
/public/uploads/
```

because that can expose private documents.

Preferred production flow:

```text
Browser
   ↓
Secure upload mechanism
   ↓
Private object storage
   ↓
Store metadata/reference
```

For BRDs especially, use private storage and controlled access.

Store information such as:

```text
file ID
original filename
safe filename
MIME type
size
storage key
submission ID
created timestamp
```

---

# Phase 11 — Build the API Layer

Create:

```text
src/app/api/contact/route.ts
src/app/api/rfq/route.ts
```

Every request should follow:

```text
1. Parse request
2. Check request size where applicable
3. Validate request data with Zod
4. Validate uploaded files
5. Check rate limit
6. Check honeypot/bot protection
7. Sanitize where required
8. Apply business rules
9. Process submission
10. Return controlled response
```

Never rely on frontend validation.

The same schema should be used again on the server.

---

# Phase 12 — Add Security Controls

Create reusable security utilities:

```text
src/lib/security/
├── rate-limit.ts
├── honeypot.ts
└── sanitize.ts
```

## Rate Limiting

Example starting policy:

```text
Contact:
5–10 submissions/IP/hour

RFQ:
3–5 submissions/IP/hour
```

Tune these limits based on real traffic.

For production/serverless deployments, use a shared external rate-limit store rather than an in-memory Map.

---

## Bot Protection

Use a hidden honeypot field and, if spam becomes significant, add a proper CAPTCHA/anti-bot solution.

The server must evaluate the bot-protection result.

---

## Input Security

Protect against:

```text
XSS
HTML injection
Email header injection
Oversized requests
Malicious filenames
Executable uploads
Unexpected enum values
Malformed URLs
Abusive submissions
```

Do not blindly HTML-escape everything at the validation layer.

Validate data according to its intended output/context and sanitize when rendering or passing it to a system that requires it.

---

# Phase 13 — Email Security

If submissions trigger email:

```text
Form
 ↓
Server
 ↓
Validated submission
 ↓
Email service
```

Never construct email headers directly from arbitrary user input.

For example, user-controlled values must never become arbitrary:

```text
To:
CC:
BCC:
Reply-To:
```

without strict validation.

Prefer:

```text
To → your configured business email
Reply-To → validated user's email
```

Also consider email provider rate limits and failure handling.

---

# Phase 14 — Business Validation

Separate basic validation from business rules.

### Basic validation

```text
email is valid
name length is valid
file is below 10 MB
URL is valid
```

### Business validation

```text
start date < end date
selected project type is supported
budget range is accepted
required features contain meaningful information
BRD is allowed for this submission
```

This distinction keeps the architecture maintainable.

---

# Phase 15 — Standard Error Handling

Do not expose internal server errors to users.

Use controlled responses.

Conceptually:

```text
400 → Invalid form data
413 → Request/file too large
429 → Too many requests
500 → Something went wrong
```

Frontend should display useful user-facing messages.

Backend logs should contain diagnostic information without exposing secrets or unnecessary personal data.

---

# Phase 16 — Unit Testing

Use unit tests for reusable validation logic.

Files:

```text
tests/unit/validations/
├── common.schema.test.ts
├── contact.schema.test.ts
├── rfq.schema.test.ts
└── file.schema.test.ts
```

## Common validation tests

Test:

```text
valid name
empty name
too-short name
too-long name

valid email
invalid email
empty email

valid phone
invalid phone
optional phone

valid URL
invalid URL
optional URL
```

## File tests

Test:

```text
valid file
unsupported extension
unsupported MIME type
file > 10 MB
empty file
multiple files when only one is allowed
malicious filename
```

## Contact tests

Test:

```text
valid contact
missing name
missing email
missing subject
missing project details
invalid phone
invalid file
```

## RFQ tests

Test:

```text
valid RFQ
missing required fields
invalid project type
invalid budget range
invalid URL
invalid date
start date after end date
start date equals end date
invalid BRD file
```

---

# Phase 17 — Integration Testing

Test the actual API routes:

```text
tests/integration/
├── contact-api.test.ts
└── rfq-api.test.ts
```

Examples:

```text
POST /api/contact
POST /api/rfq
```

Verify:

```text
Valid request → success
Invalid body → 400
Invalid file → rejected
Rate limit exceeded → 429
Unexpected enum → rejected
Malformed input → rejected
Server failure → controlled error
```

Integration tests verify that the validation, security, and API layers work together.

---

# Phase 18 — E2E Testing

Test real user flows.

Files:

```text
tests/e2e/
├── contact-form.spec.ts
└── rfq-form.spec.ts
```

## Contact E2E

```text
Open Contact page
    ↓
Submit empty form
    ↓
Validation errors
    ↓
Enter invalid data
    ↓
Errors displayed
    ↓
Enter valid data
    ↓
Upload valid file
    ↓
Submit
    ↓
Success state
```

Also test:

```text
Oversized file
Unsupported file
Invalid email
Invalid phone
```

## RFQ E2E

```text
Open Start Project
    ↓
Submit empty form
    ↓
Validation errors
    ↓
Fill required fields
    ↓
Enter invalid date range
    ↓
Date validation appears
    ↓
Correct date range
    ↓
Upload BRD
    ↓
Submit
    ↓
Success state
```

---

# Phase 19 — Global Validation Strategy

Yes, validation should be reusable across all forms.

Use:

```text
common.schema.ts
        ↓
Reusable primitives
        ↓
Contact Schema
RFQ Schema
Future Form Schemas
```

Example:

```text
emailSchema
    ↓
contactSchema
rfqSchema
newsletterSchema
loginSchema
```

Do not create:

```text
contactEmailValidation()
rfqEmailValidation()
newsletterEmailValidation()
```

when the underlying rule is identical.

---

# Phase 20 — Global Testing Strategy

Tests should also be layered.

```text
                 Common Tests
                      │
             ┌────────┴────────┐
             ↓                 ↓
       Contact Tests       RFQ Tests
             │                 │
             └────────┬────────┘
                      ↓
                API Tests
                      ↓
                E2E Tests
```

Common rules should be tested once.

Form-specific rules should have their own tests.

API/security behaviour should be tested at the integration level.

Real user workflows should be tested at the E2E level.

---

# Phase 21 — Agile Development Order

Do not implement everything simultaneously.

Use small phases/sprints.

## Sprint 1 — Foundation

- [ ] Define all fields
- [ ] Define required/optional fields
- [ ] Define character limits
- [ ] Define controlled values
- [ ] Define file policies
- [ ] Create validation folder structure
- [ ] Create common schemas

## Sprint 2 — Contact Form

- [ ] Create contact schema
- [ ] Infer TypeScript type
- [ ] Integrate React Hook Form
- [ ] Integrate Zod resolver
- [ ] Add field-level errors
- [ ] Add file validation
- [ ] Add contact API route

## Sprint 3 — RFQ Form

- [ ] Create RFQ schema
- [ ] Add controlled enums
- [ ] Add date-range validation
- [ ] Add BRD validation
- [ ] Integrate React Hook Form
- [ ] Add RFQ API route

## Sprint 4 — Security

- [ ] Add rate limiting
- [ ] Add honeypot
- [ ] Add bot protection if required
- [ ] Add request/file size limits
- [ ] Add secure file storage
- [ ] Add filename/MIME validation
- [ ] Review email security
- [ ] Review server-side validation

## Sprint 5 — Testing

- [ ] Common schema unit tests
- [ ] Contact schema tests
- [ ] RFQ schema tests
- [ ] File validation tests
- [ ] Contact API integration tests
- [ ] RFQ API integration tests
- [ ] Contact E2E tests
- [ ] RFQ E2E tests

## Sprint 6 — Production Hardening

- [ ] Test production build
- [ ] Verify environment variables
- [ ] Verify rate limits
- [ ] Verify email failures
- [ ] Verify upload failures
- [ ] Verify API error handling
- [ ] Verify logs
- [ ] Verify no sensitive information is exposed
- [ ] Test mobile UX
- [ ] Test accessibility
- [ ] Test keyboard navigation
- [ ] Test screen-reader labels
- [ ] Run security review

---

# Phase 22 — Definition of Done

A form should not be considered production-ready merely because:

```text
Form submits successfully
```

The form is ready when:

```text
✓ Client validation works
✓ Server validation works
✓ Common rules are reusable
✓ Business rules are enforced
✓ File validation works
✓ File storage is private
✓ Rate limiting works
✓ Bot protection is considered
✓ Errors are controlled
✓ No user input is blindly trusted
✓ Unit tests pass
✓ Integration tests pass
✓ E2E tests pass
✓ Mobile UX works
✓ Accessibility works
✓ Production build works
✓ Failure scenarios are handled
```

---

# Phase 23 — Final Scalable Architecture

```text
                         ┌─────────────────────┐
                         │   Common Schemas    │
                         │ name/email/phone/URL│
                         └──────────┬──────────┘
                                    │
                   ┌────────────────┴────────────────┐
                   ↓                                 ↓
          ┌─────────────────┐               ┌─────────────────┐
          │ Contact Schema  │               │   RFQ Schema    │
          └────────┬────────┘               └────────┬────────┘
                   │                                 │
                   └────────────────┬────────────────┘
                                    ↓
                           React Hook Form
                                    ↓
                              Zod Resolver
                                    ↓
                           Client Validation
                                    ↓
                           Next.js API Route
                                    ↓
                         Server-side Zod Check
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                               ↓
              File Validation                 Security Layer
                    │                     ┌────────┼────────┐
                    │                     ↓        ↓        ↓
                    │                 Rate Limit Honeypot Bot
                    │
                    └───────────────┬───────────────┘
                                    ↓
                           Business Validation
                                    ↓
                       Email / DB / Private Storage
                                    ↓
                              Controlled Response
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                               ↓
               Unit Tests                     Integration Tests
                                                    ↓
                                               E2E Tests
```

---

# Core Principles

1. **Zod is the validation source of truth.**
2. **Use reusable common schemas.**
3. **Compose form-specific schemas from common schemas.**
4. **Infer TypeScript types from Zod whenever practical.**
5. **React Hook Form handles form state; Zod handles validation.**
6. **Validate on the client for UX.**
7. **Validate again on the server for security.**
8. **Never trust client-side validation.**
9. **Keep security utilities reusable.**
10. **Keep business rules separate from basic field validation.**
11. **Never trust file extensions or browser MIME types alone.**
12. **Keep uploaded BRDs/private documents outside public storage.**
13. **Use rate limiting for public forms.**
14. **Use automated tests at unit, integration, and E2E levels.**
15. **Test common rules once and form-specific rules separately.**
16. **Build incrementally using the Agile phases above.**
17. **Keep the architecture ready for future forms without creating one giant global schema.**

## Target Result

```text
Reusable
      +
Type-safe
      +
Secure
      +
Testable
      +
Maintainable
      +
Scalable
      =
Production-grade Next.js Forms
```
