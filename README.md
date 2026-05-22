# Web Application Security Assessment Project

**Cybersecurity Internship Assignment — 3 Week Project**
**Student:** Noor Fatima
**Date:** April 27, 2026

---

## Project Overview

This project is a Node.js and Express web application that was built with intentional security vulnerabilities for the purpose of a 3-week cybersecurity internship assignment. The goal was to:

1. Identify real security vulnerabilities through manual testing
2. Implement industry-standard security fixes
3. Add security logging
4. Document all findings and fixes

The application has three pages: **Signup**, **Login**, and **Profile**.

---

## Folder Structure

```
myapp/
├── app.js              ← Secure version (Week 2 fixes applied)
├── app_old.js          ← Original vulnerable version (Week 1 testing)
├── security.log        ← Winston security log file (generated at runtime)
├── package.json        ← Project dependencies
└── node_modules/       ← Installed packages
```

---

## Setup Instructions

### Requirements
- Node.js v16 or higher — download from https://nodejs.org
- npm (comes with Node.js)
- Git — download from https://git-scm.com

### Steps to Run Locally

**1. Clone the repository:**
```bash
git clone https://github.com/[YOUR-USERNAME]/[YOUR-REPO-NAME].git
```

**2. Go into the project folder:**
```bash
cd myapp
```

**3. Install dependencies:**
```bash
npm install
```

**4. Start the application:**
```bash
node app.js
```

**5. Open your browser and go to:**
```
http://localhost:3000/signup
http://localhost:3000/login
http://localhost:3000/profile
```

---

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| express | latest | Web framework |
| bcryptjs | latest | Password hashing |
| jsonwebtoken | latest | JWT token authentication |
| helmet | latest | Secure HTTP headers |
| validator | latest | Input validation and sanitization |
| winston | latest | Security event logging |

Install all dependencies with:
```bash
npm install express bcryptjs jsonwebtoken helmet validator winston
```

---

## Vulnerabilities Found (Week 1)

These vulnerabilities were found in the original `app_old.js` file:

| ID | Vulnerability | Risk Level | Status |
|----|--------------|------------|--------|
| V-001 | Stored Cross-Site Scripting (XSS) | High | Fixed |
| V-002 | SQL Injection (tested) | None | Not vulnerable |
| V-003 | Plain text password storage | High | Fixed |
| V-004 | Missing HTTP security headers | Medium | Fixed |

---

## Security Improvements Made (Week 2)

### 1. Input Validation
All user inputs are now validated using the `validator` package:
- Usernames must be alphanumeric only — blocks XSS and script injection
- Passwords must be at least 6 characters long

```javascript
if (!validator.isAlphanumeric(username)) {
  return res.send('Invalid username. Only letters and numbers allowed.');
}
if (!validator.isLength(password, { min: 6 })) {
  return res.send('Password must be at least 6 characters.');
}
```

### 2. Password Hashing with bcrypt
Passwords are now hashed before storage using bcryptjs with a salt factor of 10:
```javascript
const hashedPassword = await bcrypt.hash(password, 10);
users.push({ username, password: hashedPassword });
```

During login, passwords are compared securely:
```javascript
const match = await bcrypt.compare(password, user.password);
```

### 3. JWT Token Authentication
After a successful login, a JWT token is issued. The `/profile` route is protected and requires a valid token in the Authorization header:
```javascript
const token = jwt.sign({ username }, JWT_SECRET, { expiresIn: '1h' });
```

### 4. Secure HTTP Headers with Helmet
Helmet middleware was added to set all required security headers automatically and remove the `X-Powered-By: Express` header:
```javascript
app.use(helmet());
```

Headers added by helmet:
- `Content-Security-Policy`
- `Strict-Transport-Security`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: SAMEORIGIN`
- `Referrer-Policy: no-referrer`

---

## Security Logging (Week 3)

Winston logging was added to record all security events to both the console and `security.log` file:

| Event | Log Level |
|-------|-----------|
| App startup | INFO |
| New user signup | INFO |
| Successful login | INFO |
| Failed login — wrong password | WARN |
| Invalid username input (XSS attempt) | WARN |
| Password too short | WARN |
| Profile access — no token | WARN |

Example log output:
```
2026-04-27T11:03:13.238Z [INFO]: Secure app started at http://localhost:3000
2026-04-27T11:05:21.798Z [WARN]: Invalid signup attempt - bad username: <script>alert('XSS')</script>
2026-04-27T11:07:44.885Z [INFO]: New user signed up: testuser
2026-04-27T11:07:57.651Z [WARN]: Failed login attempt - wrong password for user: testuser
2026-04-27T11:09:17.406Z [INFO]: Successful login: noor
```

---

## How to Test the App

### Test 1 — XSS is blocked
1. Go to `http://localhost:3000/signup`
2. Enter `<script>alert('XSS')</script>` as the username
3. Expected result: "Invalid username. Only letters and numbers allowed."

### Test 2 — Password validation works
1. Go to `http://localhost:3000/signup`
2. Enter a valid username with a password less than 6 characters
3. Expected result: "Password must be at least 6 characters."

### Test 3 — Normal signup and login
1. Go to `http://localhost:3000/signup`
2. Enter username: `testuser` and password: `hello123`
3. Go to `http://localhost:3000/login` and use the same credentials
4. Expected result: "Login successful!" with a JWT token displayed

### Test 4 — Security headers are set
1. Go to `http://localhost:3000/login`
2. Press F12 > Network tab > click the login request > Response Headers
3. Expected result: Content-Security-Policy, X-Frame-Options, and other headers visible. X-Powered-By should NOT be present.

---

## Screenshots

> Replace each placeholder below with your actual screenshots

| Screenshot | Description |
|-----------|-------------|
| ![XSS Popup](screenshots/xss_popup.png) | XSS alert popup on vulnerable app |
| ![XSS Blocked](screenshots/xss_blocked.png) | XSS blocked after fix |
| ![Password Short](screenshots/password_short.png) | Password too short validation |
| ![JWT Token](screenshots/jwt_token.png) | JWT token after successful login |
| ![Headers Before](screenshots/headers_before.png) | Response headers before fix — X-Powered-By visible |
| ![Headers After](screenshots/headers_after.png) | Response headers after fix — security headers present |
| ![Winston Logs](screenshots/winston_logs.png) | Winston security logs in terminal |

---

## Before Fix vs After Fix

| Feature | Before (app_old.js) | After (app.js) |
|---------|-------------------|----------------|
| XSS | Script tags executed | Blocked by input validation |
| Passwords | Stored in plain text | Hashed with bcrypt |
| Authentication | None | JWT token required |
| Security headers | Missing | Added via helmet |
| Logging | None | Winston logging to file and console |

---

## Known Limitations

- JWT secret is currently hardcoded in the source code. In a production environment, this should be moved to an environment variable using a `.env` file and the `dotenv` package.
- User data is stored in memory (JavaScript array) and is lost when the app restarts. A real application would use a database like MongoDB or PostgreSQL.
- No rate limiting is implemented on the login route. A real application should limit failed login attempts to prevent brute force attacks.

---
# Cyber Security Internship Tasks (Weeks 4–6)

## Overview
This repository contains cybersecurity internship tasks completed during Weeks 4–6. 
The project focuses on web security hardening, ethical hacking, penetration testing, 
security auditing, and secure deployment practices.

---

# Week 4 – Advanced Threat Detection & Web Security

## Implemented Features
- API Rate Limiting
- Helmet Security Headers
- Content Security Policy (CSP)
- HSTS Configuration
- Secure CORS Policies
- Logging & Monitoring

## Files
- app_4.js

---

# Week 5 – Ethical Hacking & Vulnerability Testing

## Activities Performed
- SQL Injection Testing using SQLMap
- CSRF Testing using Burp Suite
- Ethical Hacking using Kali Linux
- Secure Coding Improvements

## Files
- app_5.js
- app_5new.js

---

# Week 6 – Security Audits & Secure Deployment

## Activities Performed
- OWASP ZAP Security Audit
- Penetration Testing
- Security Hardening
- Deployment Security Review

## Reports
- ZAP HTML Reports

---

# Bonus Tasks

## Implemented
- Web Application Firewall (WAF)
- Advanced Security Enhancements
- Zero Trust Security Concepts

## Files
- app_bonus.js
- app_waf.js

---

# Tools & Technologies
- Node.js
- Express.js
- OWASP ZAP
- Burp Suite
- SQLMap
- Helmet
- JWT
- Kali Linux

## References

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- helmet docs: https://helmetjs.github.io/
- bcryptjs docs: https://www.npmjs.com/package/bcryptjs
- jsonwebtoken docs: https://www.npmjs.com/package/jsonwebtoken
- validator docs: https://www.npmjs.com/package/validator
- winston docs: https://www.npmjs.com/package/winston
