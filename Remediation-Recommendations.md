# Web Application Security Remediation Recommendations

**Student:** Lubanzi Fihla  
**Internship:** Codomax Cybersecurity Internship  
**Module:** Module 3 - Web Application Security  
**Assessment Target:** OWASP Juice Shop  
**Environment:** Controlled Local Laboratory  

---

## 1. Introduction

This document provides remediation recommendations based on the security findings and observations identified during the controlled OWASP Juice Shop assessment.

The purpose of remediation is not only to correct an individual vulnerability but also to improve the security controls that prevent similar weaknesses from appearing elsewhere in an application.

The recommendations focus on:

- Secure error handling
- Server-side authorization
- Input validation
- Parameterized database queries
- Least privilege
- Authentication and session security

---

# 2. WEB-001: Improper Error Handling

**OWASP Category:** A02:2025 Security Misconfiguration  
**Assessment Status:** Confirmed Finding

## Problem

The application returned an HTTP `500 Internal Server Error` containing detailed error information after receiving an unexpected request.

Example tested route:

```text
/rest/qwertz
```

Detailed application errors can unnecessarily expose information about internal application behavior.

## Recommended Remediation

### 2.1 Implement Centralized Exception Handling

Applications should use a consistent exception-handling mechanism.

Unexpected errors should be intercepted before detailed diagnostic information is returned to the client.

The desired flow is:

```text
Application Error
       ↓
Central Exception Handler
       ↓
Detailed Information
       ↓
Protected Server Log

Client Response
       ↓
Generic Error Message
```

### 2.2 Return Generic User-Facing Errors

Users should receive a simple response such as:

```text
An unexpected error occurred.
Please try again later.
```

Internal details should not be included in the response.

### 2.3 Protect Diagnostic Information

Detailed debugging information should be available only through protected server-side logging and monitoring systems.

Avoid exposing:

- Stack traces
- Internal file paths
- Database query information
- Framework debugging information
- Internal application configuration
- Sensitive exception details

### 2.4 Disable Debug Features in Production

Production environments should not expose development or debugging functionality that returns excessive technical information.

---

# 3. WEB-002: Horizontal Broken Access Control

**OWASP Category:** A01:2025 Broken Access Control  
**Assessment Status:** Confirmed Finding

## Problem

The authenticated test user was associated with:

```text
User ID: 25
Basket ID: 6
```

However, the same authenticated session successfully retrieved:

```text
Basket ID: 5
User ID: 16
```

The application therefore did not adequately verify that the requesting user was authorized to access the requested basket.

## Recommended Remediation

### 3.1 Enforce Authorization Server-Side

Authorization must be performed by the server for every protected resource request.

The application should not assume that a user is permitted to access a resource simply because the user knows or submits its identifier.

### 3.2 Verify Resource Ownership

Before returning a user-specific resource, compare the authenticated identity with the resource owner.

Example authorization logic:

```text
Authenticated User
       ↓
Requested Basket
       ↓
Read Resource Owner
       ↓
Does Resource Owner
Match Authenticated User?
       │
       ├── YES → Allow
       │
       └── NO  → Deny
```

### 3.3 Deny by Default

Requests should be rejected unless authorization has explicitly been established.

The security model should follow:

```text
Not explicitly authorized
        ↓
DENY
```

rather than:

```text
Not explicitly denied
        ↓
ALLOW
```

### 3.4 Apply Least Privilege

Users should only receive access to the resources and operations required for their legitimate application role.

### 3.5 Do Not Trust Object Identifiers

Values such as:

```text
/rest/basket/5
```

should be treated only as resource identifiers.

Possession of an identifier must not be interpreted as proof that the requester is authorized to access the resource.

### 3.6 Return Appropriate Access-Denied Responses

If a user attempts to access a resource without authorization, the application should deny the operation and return an appropriate response without exposing unnecessary information.

---

# 4. WEB-003: Unexpected Input Handling

**OWASP Relevance:** A05:2025 Injection  
**Assessment Status:** Observation - Further Validation Required

## Problem

The product search behaved differently when receiving unexpected input consisting of a single apostrophe.

The test did not conclusively demonstrate SQL Injection.

However, unusual behavior following user-controlled metacharacter input indicates that input handling should be reviewed.

## Recommended Remediation

### 4.1 Validate Input Server-Side

Input should be checked against the business requirements of each application field.

Validation may include:

- Expected data type
- Minimum and maximum length
- Allowed format
- Allowed values
- Numerical ranges
- Required fields

Validation should be performed on the server, even when client-side validation is also used.

### 4.2 Use Parameterized Queries

Database operations should use parameterized queries or prepared statements.

The application should separate:

```text
SQL Instructions
```

from:

```text
User-Controlled Data
```

Conceptually:

```text
BAD

SQL Command + User Input
          ↓
       Database
```

A safer design is:

```text
SQL Statement
     +
Parameter
     ↓
Database safely treats
input as data
```

### 4.3 Avoid Query String Concatenation

Applications should not construct database queries by directly joining untrusted user input with SQL instructions.

### 4.4 Apply Least Privilege to Database Accounts

The application's database account should only receive the permissions required by the application.

For example, an application component that only needs to read product data should not automatically receive unrestricted database administrative privileges.

### 4.5 Handle Invalid Input Safely

Unexpected input should not expose diagnostic information or create unsafe application states.

---

# 5. WEB-004: Authentication and Session Logout

**Related OWASP Category:** A07:2025 Authentication Failures  
**Assessment Status:** PASS

## Observed Control

The tested logout workflow successfully removed the authenticated browser state.

After logout:

- The authenticated email address disappeared.
- The Account menu returned to `Login`.
- The shopping basket displayed `anonymous`.

No vulnerability was identified during this limited test.

## Recommended Security Practices

Although the test passed, applications should continue to implement strong authentication and session controls.

### 5.1 Protect Session Tokens

Authentication and session tokens should be protected from unauthorized disclosure.

Sensitive token values should not be exposed in:

- URLs
- Application logs
- Error messages
- Public screenshots
- Client-side debugging output

### 5.2 Use Secure Session Identifiers

Session identifiers should be generated using secure, unpredictable values.

### 5.3 Configure Appropriate Session Expiration

Sessions should expire after an appropriate period of inactivity or when required by the application's security requirements.

### 5.4 Secure Logout

Logout should:

```text
Remove authenticated client state
           ↓
Invalidate server-side session/token
where applicable
           ↓
Require authentication
for protected resources
```

### 5.5 Protect Cookies

Where cookies are used for authentication or session management, appropriate protections should be considered, including:

```text
Secure
HttpOnly
SameSite
```

according to the application's architecture and requirements.

---

# 6. Remediation Priority

## High Priority

### WEB-002: Broken Access Control

This should receive the highest remediation priority because insufficient authorization can expose resources belonging to other users.

Primary fix:

```text
Server-side object-level authorization
```

---

## Medium Priority

### WEB-001: Improper Error Handling

Detailed technical errors should be removed from client-facing responses.

Primary fix:

```text
Centralized secure exception handling
```

---

## Requires Further Validation

### WEB-003: Unexpected Input Handling

The observed input behavior should receive additional controlled security testing and code review before being classified as a confirmed Injection vulnerability.

Primary preventive controls:

```text
Input Validation
Parameterized Queries
Safe Error Handling
```

---

## Control Passed

### WEB-004: Authentication and Session Logout

No weakness was identified during the limited logout test.

Continue maintaining strong authentication and session-management controls.

---

# 7. Secure Development Principles

The findings demonstrate several broader secure-development principles:

```text
Never trust user input
        ↓
Validate

Database interaction
        ↓
Parameterize

Protected resources
        ↓
Authorize

Application failure
        ↓
Fail securely

User permissions
        ↓
Least privilege

Authentication sessions
        ↓
Protect and expire securely
```

Security controls should be implemented throughout application design, development, testing, deployment, and maintenance.

---

# 8. Conclusion

The assessment demonstrated that remediation should address the root security control rather than only the specific request used to demonstrate a weakness.

For WEB-002, changing or hiding basket identifiers would not solve the underlying problem. The correct remediation is server-side authorization and ownership verification.

For WEB-001, blocking the specific `/rest/qwertz` request would not address the broader issue. The correct remediation is centralized secure error handling.

For WEB-003, blocking a single apostrophe would not provide reliable Injection protection. Secure database interaction should rely on parameterized queries together with appropriate input validation.

WEB-004 demonstrated expected logout behavior during the limited assessment, but secure session-management practices should continue to be maintained.

Effective remediation combines secure design, defensive coding, access control, least privilege, secure configuration, and continuous security testing.