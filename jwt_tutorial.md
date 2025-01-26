# JSON Web Tokens (JWT) Learning Journey

## 📚 Table of Contents
- [Introduction](#introduction)
- [Fundamentals](#fundamentals)
- [Terminology](#-terminology-explained)
- [JWT Structure](#jwt-structure)
- [Authentication Flow](#authentication-flow)
- [Implementation](#implementation)
- [Security Considerations](#security-considerations)
- [Advanced Concepts](#advanced-concepts)
- [Code Examples](#code-examples)
- [Resources](#resources)

## 🌟 Introduction
JSON Web Tokens (JWTs) are a compact, URL-safe method for representing claims between two parties.

## 🧩 Fundamentals
### What is a JWT?
- Self-contained token for securely transmitting information
- Used for authentication and information exchange
- Consists of three parts: Header, Payload, Signature

### Key Characteristics
- Stateless
- Compact
- Can be verified and trusted
- Suitable for distributed systems

## 🔍 Terminology Explained

### URL Safe
- Encoding that ensures token can be used directly in URLs
- JWTs are often transmitted via URLs (e.g., in query strings or headers).
  Base64Url encoding ensures that the token can be safely transported in these formats without introducing special characters like +, /, or = that may have special meanings in URLs.
- How Base64Url Differs from Base64:
  - Base64 encoding uses +, /, and = characters.
  - Replaces problematic characters:
    - `+` becomes `-`
    - `/` becomes `_`
    - Removes trailing `=` padding

### Signature
- Cryptographic mechanism to verify token integrity
- Created by:
  1. Encoding header and payload
  2. Applying a secret key
  3. Using a hashing algorithm (e.g., HMAC-SHA256)
     
  e.g. Signature = HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
- Ensures token:
  - Hasn't been tampered with
  - Was created by authentic source

### Token Encoding
Reasons for encoding header and payload:
- Compression of token size
- Ensure URL safety
- Obfuscate raw JSON content
- Standardize parsing
- **Important**: Encoding does not encrypt the data. Anyone who obtains the JWT can decode and read the header and payload.

### Claims in JWT
Types of claims in a token:

1. **Registered Claims** (Standard)
   - `iss`: Issuer
   - `sub`: Subject
   - `aud`: Audience
   - `exp`: Expiration Time
   - `nbf`: Not Before
   - `iat`: Issued At
   - `jti`: JWT ID

2. **Public Claims**
   - Custom, shared claims
   - Examples: `name`, `role`, `email`

3. **Private Claims**
   - Application-specific information
   - Not shared publicly
   - Examples: `user_id`, `department`

### Stateless Authentication
1. Server doesn't store session information
    - Server doesn't track active sessions
    - Each request is self-contained
    - Authentication happens via the token itself
2. Token contains all necessary authentication details
    - User details embedded in JWT
    - Server can validate token without database lookup
    - Reduces server-side storage and lookup overhead
3. Benefits:
    - No server-side session storage
    - Easy horizontal scaling
    - Independent token verification

### Self-Contained Token
- Token includes all essential authentication information
- No external lookup required
- Contains:
  - User ID
  - Roles/Permissions
  - Expiration time
  - Issuer information
- When this token is received, the server can:
  - Verify its authenticity
  - Check user permissions
  - Determine token validity
  - Make authorization decisions

## 🔍 JWT Structure
### 1. Header
```json
{
  "alg": "HS256",  // Hashing Algorithm
  "typ": "JWT"     // Token Type
}
```

### 2. Payload
```json
{
  "sub": "1234567890",  // Subject (User ID)
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022     // Issued At Time
}
```

### 3. Signature
- Created by encoding header and payload
- Signed with a secret key
- Ensures token integrity

## 🔐 Authentication Flow
1. User Login
2. Server Verifies Credentials
3. Generate JWT
4. Send Token to Client
5. Client Stores Token
6. Subsequent Requests Include Token

## 💻 Implementation
### Python JWT Example
```python
import jwt

def generate_token(user_id):
    payload = {
        'user_id': user_id,
        'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=24)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        return None
```

## 🛡️ Security Considerations
- Never store sensitive data in JWT
- Use strong, unique secret keys
- Implement short token lifetimes
- Always use HTTPS
- Validate all claims

## 🚀 Advanced Concepts
### Token Types
- Access Token
- Refresh Token
- Stateless Authentication

### Signature Algorithms
- HS256 (HMAC with SHA-256)
- RS256 (RSA Signature)
- ES256 (ECDSA)

## 📖 Code Examples
### [Full JWT Authentication System](/path/to/jwt_auth_system.py)
- Token Generation
- Token Verification
- Token Refresh Mechanism

## 🔗 Resources
### Recommended Reading
- [JWT.io Official Website](https://jwt.io)
- [IETF RFC 7519 - JWT Standard](https://tools.ietf.org/html/rfc7519)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

## 📝 Personal Notes
- Add your personal insights
- Track your learning progress
- Note down challenges and solutions

---
