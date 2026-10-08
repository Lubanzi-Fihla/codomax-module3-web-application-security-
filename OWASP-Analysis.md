# OWASP Top 10 Analysis

**Student:** Lubanzi Fihla  
**Internship:** Codomax Cybersecurity Internship  
**Module:** Module 3 - Web Application Security  
**Assessment**Environment:** Local OWASP Juice Shop Laboratory  

## 1. Introduction

The OWASP Top 10 provides awareness of significant security risks affecting modern web applications.

During Module 3, I studied the OWASP Top 10:2025 and applied relevant categories to findings and observations identified during a controlled assessment of OWASP Juice Shop.

The assessment was performed against an intentionally vulnerable application hosted locally at:

```text
http://127.0.0.1:3000
```

The practical assessment focused on understanding security weaknesses, gathering evidence, evaluating their potential impact, and developing appropriate remediation recommendations.

## 2. OWASP Top 10:2025

The OWASP Top 10:2025 categories are:

1. A01:2025 - Broken Access Control
2. A02:2025 - Security Misconfiguration
3. A03:2025 - Software Supply Chain Failures
4. A04:2025 - Cryptographic Failures
5. A05:2025 - Injection
6. A06:2025 - Insecure Design
7. A07:2025 - Authentication Failures
8. A08:2025 - Software or Data Integrity Failures
9. A09:2025 - Security Logging and Alerting Failures
10. A10:2025 - Mishandling of Exceptional Conditions

The practical lab focused especially on categories relevant to the observed application behavior.

---

## 3. A01:2025 Broken Access Control

### Finding

WEB-002 - Horizontal Broken Access Control

### Observation

My authenticated test account was associated with:

```text
User ID: 25
Basket ID: 6
```

A legitimate request to the user's own basket:

```text
GET /rest/basket/6
```

returned an HTTP `200 OK` response.

Using the same authenticated session, I changed only the requested basket identifier:

```text
GET /rest/basket/5
```

The application again returned:

```text
HTTP/1.1 200 OK
```

The returned resource was identified as:

```text
Basket ID: 5
User ID: 16
```

The authenticated account therefore accessed a basket belonging to a different user.

### Security Analysis

This demonstrated insufficient object-level authorization.

Authentication successfully established the user's identity, but authorization was not adequately enforced when the application retrieved another user's basket.

This is an example of horizontal Broken Access Control because one normal user was able to access a resource belonging to another normal user.

### Recommended Security Controls

- Enforce authorization on every protected resource request.
- Verify resource ownership server-side.
- Deny access by default.
- Apply least privilege.
- Do not rely on user-controlled object identifiers as proof of authorization.

### Result

**Confirmed Finding**

---

## 4. A02:2025 Security Misconfiguration

### Finding

WEB-001 - Improper Error Handling

### Observation

During the controlled lab, an unexpected route was requested:

```text
GET /rest/qwertz
```

The application returned:

```text
HTTP/1.1 500 Internal Server Error
```

and exposed detailed error content to the client.

### Security Analysis

Applications can encounter unexpected conditions, but detailed internal diagnostic information should not normally be exposed to end users.

Verbose error information may provide unnecessary insight into application behavior, internal implementation details, or technology components.

### Recommended Security Controls

- Implement centralized exception handling.
- Return generic client-facing error messages.
- Store detailed diagnostic information in protected server-side logs.
- Restrict sensitive debugging information to authorized administrators and developers.
- Monitor repeated or unusual application errors.

### Result

**Confirmed Finding**

---

## 5. A05:2025 Injection

### Observation

WEB-003 - Unexpected Input Handling

The Juice Shop product search functionality was first tested using normal input:

```text
apple
```

The application returned Apple-related product results as expected.

The same search functionality was then tested using a single apostrophe as unexpected input:

```text
'
```

The application returned an HTTP `200 OK` response with behavior different from the normal search baseline.

### Security Analysis

Unexpected behavior after receiving a metacharacter indicates that the application's handling of user-controlled input deserves additional security review.

However, the limited evidence gathered during this assessment did not conclusively demonstrate that the input modified a database query.

The behavior was therefore documented as an observation rather than incorrectly reported as confirmed SQL Injection.

### Recommended Security Controls

- Validate user input according to expected format, type, length, and business requirements.
- Use parameterized queries or prepared statements for database operations.
- Treat user-controlled input strictly as data.
- Avoid directly concatenating untrusted input into database queries.
- Apply least privilege to application database accounts.
- Handle unexpected input safely.

### Result

**Observation - Further Validation Required**

---

## 6. A07:2025 Authentication Failures

### Test

WEB-004 - Authentication and Session Logout Analysis

### Authenticated Baseline

Before logout, the application's `/rest/user/whoami` endpoint returned:

```text
HTTP/1.1 200 OK
```

and identified:

```text
User ID: 25
Email: lubanzi-lab1@example.com
```

This demonstrated that the application associated the browser session with the authenticated test account.

### Logout Test

The application's normal Logout function was used.

After logout:

- The authenticated email address was no longer displayed.
- The Account menu displayed `Login`.
- The shopping basket displayed `anonymous`.

### Security Analysis

The browser's authenticated state was removed correctly during the tested logout workflow.

No authentication or session logout weakness was identified during this limited test.

### Recommended Security Controls

Applications should continue to:

- Protect authentication and session tokens.
- Use securely generated session identifiers.
- Expire sessions appropriately.
- Remove authenticated client state after logout.
- Invalidate server-side sessions or tokens when required.
- Require authentication before protected resources can be accessed.

### Result

**PASS**

---

## 7. OWASP Mapping Summary

```text
WEB-001
Improper Error Handling
A02:2025 Security Misconfiguration
Result: Confirmed Finding

WEB-002
Horizontal Broken Access Control
A01:2025 Broken Access Control
Result: Confirmed Finding

WEB-003
Unexpected Input Handling
A05:2025 Injection Relevance
Result: Observation - Further Validation Required

WEB-004
Authentication and Session Logout Analysis
A07:2025 Authentication Failures
Result: PASS
```

## 8. Key Lesson

The assessment demonstrated the importance of distinguishing between:

```text
Confirmed Vulnerability
        ↓
Supported by evidence

Observation
        ↓
Interesting behavior requiring further validation

Passed Security Control
        ↓
Tested behavior operated as expected
```

A security finding should be based on evidence rather than assumptions.

This approach reduces false positives and produces clearer, more defensible vulnerability reports.

## 9. Conclusion

The controlled OWASP Juice Shop assessment demonstrated practical examples of Broken Access Control and Security Misconfiguration.

The assessment also identified unexpected input-handling behavior relevant to Injection testing, while correctly avoiding an unsupported conclusion that SQL Injection had been proven.

Finally, the authentication and logout test demonstrated expected session behavior during the limited assessment.

The exercise strengthened my ability to map technical observations to OWASP Top 10 categories and distinguish confirmed vulnerabilities from observations and passing security controls.