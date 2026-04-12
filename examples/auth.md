---
schema_version: "0.1.0"
module_id: auth
module_path: src/auth
language: typescript
extracted_at: "2026-04-11T00:00:00Z"
confidence_summary:
  confirmed: 3
  inferred: 1
  uncertain: 0
derive_version: "0.1.0"
source_file_count: 4
---

# auth Module

## Entry Points

### login
- **Signature:** `(email: string, password: string) => Promise<boolean>`
- **Confidence:** confirmed
- **Description:** Authenticate a user by email and password

### logout
- **Signature:** `() => void`
- **Confidence:** confirmed

### AuthService
- **Signature:** `class AuthService { login, logout, validateToken }`
- **Confidence:** confirmed
- **Description:** User authentication service

## Constraints

### Passwords must be hashed before storage
- **Category:** security
- **Confidence:** inferred
- **Rationale:** PCI compliance requires bcrypt or equivalent hashing for all stored credentials
- **Rejected Alternatives:**
  - Store plaintext passwords — *Violates PCI DSS requirement 8.2.1*
  - Use MD5 hashing — *Cryptographically broken since 2004*

## Dependencies

### External
- `bcrypt` (confirmed)
- `jsonwebtoken` (confirmed)

### Internal
- `./models/user` (confirmed)
