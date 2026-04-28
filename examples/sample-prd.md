# PRD: User Authentication Service

## Summary
Add email/password authentication to the API. Users can register, log in, and receive a JWT. Logout invalidates the token server-side.

## Goals
- Users can register with email + password
- Users can log in and receive a short-lived JWT
- Authenticated endpoints reject requests without a valid token

## Non-Goals
- OAuth / social login
- Password reset flow (deferred to v2)

## Users & Use Cases
| User | Need | Success Looks Like |
|------|------|--------------------|
| End user | Access protected resources | Login returns token; token gates API |
| Developer | Secure endpoints quickly | Middleware rejects invalid tokens with 401 |

## Functional Requirements
| ID | Requirement | Priority | Validation Layer | Acceptance Criteria |
|----|-------------|----------|------------------|---------------------|
| FR1 | POST /auth/register accepts email + password, returns user ID | Must | Integration | Given valid email+password, returns 201 with userId; duplicate email returns 409 |
| FR2 | POST /auth/login accepts credentials, returns JWT | Must | Integration | Valid creds return 200 + token; invalid creds return 401 |
| FR3 | Auth middleware validates JWT on protected routes | Must | Unit + Integration | Valid token passes; expired/missing token returns 401 |
| FR4 | POST /auth/logout invalidates token server-side | Should | Integration | Token rejected after logout |

## Milestones
| ID | Title | Outcome |
|----|-------|---------|
| M0 | Schema freeze | User schema and JWT contract finalized |
| M1 | Auth endpoints live | Register, login, logout, middleware working |
