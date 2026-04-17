# A01:Broken Access Control
## 1. Overview

- **Definition:** Occurs when applications fail to enforce restrictions on authenticated users’ actions.
- **Impact:** Unauthorized data exposure, privilege escalation, account takeover, full system compromise.
- **OWASP Ranking:** Listed as one of the most critical risks in the OWASP Top 10.

## 2. Key Concepts

### Authentication vs Authorization

- **Authentication:** Verifies identity (login).
- **Authorization:** Determines allowed actions.
- **Broken Access Control:** Failure of authorization, not authentication.

### Common Issues

- Lack of proper permission checks.
- Direct access to restricted resources (URL/API manipulation).
- Privilege escalation:
    
    - **Horizontal:** Accessing another user’s data at same level.
    - **Vertical:** Gaining admin or higher-level access.
- Insecure Direct Object References (IDOR).
- Missing server-side validation.
- Failure to follow “deny by default” principle.

## 3. Why It’s Critical

- Exposes sensitive personal/financial data.
- Enables modification/deletion of records.
- Allows attackers to gain administrative control.
- Often leads to full application compromise.

## 4. Insecure Direct Object Reference (IDOR)

### Definition

- Occurs when internal object identifiers (IDs, filenames, account numbers) are exposed without verifying ownership.

### How It Works

- Example:
    
    - `GET /api/user/1001` → attacker changes to `GET /api/user/1002`.
    - If server doesn’t check ownership, attacker gains unauthorized access.

### Common Locations

- URL parameters, REST API endpoints, hidden form fields, JSON bodies, file paths.

### Types

- **Horizontal IDOR:** Accessing peer data.
- **Vertical IDOR:** Accessing admin-level data.
- **Blind IDOR:** Inferring existence via responses.

### Root Causes

- Missing server-side checks.
- Client-side validation only.
- Predictable IDs.
- Lack of centralized access control.

### Impact

- Unauthorized disclosure, modification, deletion.
- Account takeover.
- Financial fraud.
- Business logic compromise.

### Prevention

- Verify ownership server-side.
- Use indirect references (UUIDs).
- Implement role/attribute-based access control.
- Apply least privilege.
- Log unauthorized attempts.

## 5. Parameter Tampering

### Definition

- Manipulation of parameters exchanged between client and server to bypass restrictions or gain unauthorized access.

### Examples

- Price manipulation (`price=1000 → price=1`).
- Role manipulation (`role=user → role=admin`).
- Account ID manipulation.
- Quantity manipulation.
- Business logic manipulation.

### Root Causes

- Trusting client-side input.
- Missing server-side validation.
- Exposing sensitive parameters in hidden fields.
- Poor access control.
- Insecure API design.

### Impact

- Financial fraud.
- Privilege escalation.
- Unauthorized data access.
- Workflow bypass.

### Prevention

- Validate parameters server-side.
- Do not trust hidden fields.
- Fetch sensitive values from database.
- Enforce strict role-based access control.
- Log suspicious changes.

## 6. Horizontal Privilege Escalation

### Definition

- User accesses/manipulates resources belonging to another user at the same privilege level.

### Characteristics

- Equal privilege level.
- Exploits missing ownership validation.
- Often tied to IDOR.

### Attack Scenarios

- Viewing/modifying another user’s profile, invoices, exam results, account settings, or funds.

### Root Causes

- Missing ownership checks.
- Predictable IDs.
- Poor session-to-resource binding.

### Impact

- Unauthorized data disclosure.
- Fraud.
- Account takeover.
- Compliance violations.

### Prevention

- Enforce server-side ownership checks.
- Validate resource ownership.
- Centralized authorization logic.
- Avoid predictable identifiers.
- Monitor unauthorized attempts.

## 7. Vertical Privilege Escalation

### Definition

- User with lower privileges gains access to higher-privileged functions/resources (e.g., admin).

### Characteristics

- Movement upward in privilege hierarchy.
- Exploits missing role validation.
- Common in APIs/backends.

### Attack Techniques

- Direct access to admin URLs.
- Role parameter manipulation.
- JWT token tampering.
- Exploiting hidden endpoints.
- Using outdated API versions.

### Root Causes

- Missing server-side role checks.
- Client-side controls only.
- Improper RBAC implementation.
- Trusting user input.

### Impact

- Full administrative access.
- Account creation/modification/deletion.
- Access to system configurations.
- Data manipulation/destruction.
- Full application compromise.

### Prevention

- Strict server-side RBAC.
- Principle of least privilege.
- Centralized authorization middleware.
- Secure role/permission changes.
- Monitor restricted endpoint access.

