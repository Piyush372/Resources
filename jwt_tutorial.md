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

# 🛡️ Security Considerations
- Never store sensitive data in JWT
- Use strong, unique secret keys
- Implement short token lifetimes
- Always use HTTPS
- Validate all claims
  
## **Where to Store Tokens on the Client Side?**

### **1. Local Storage**
- **Description**: Tokens are stored using the `localStorage` API.
- **Pros**:
  - Easy to implement.
  - Persistent (remains even after the browser is closed).
- **Cons**:
  - Vulnerable to **Cross-Site Scripting (XSS)** attacks, as malicious scripts can access localStorage.

### **2. Session Storage**
- **Description**: Tokens are stored using the `sessionStorage` API.
- **Pros**:
  - Only persists for the browser session (cleared when the browser tab is closed).
- **Cons**:
  - Like `localStorage`, it is vulnerable to **XSS**.

### **3. HTTP-Only Cookies** *(Recommended for sensitive tokens)*
- **Description**: Tokens are stored in cookies with the `HttpOnly` flag enabled, preventing client-side scripts from accessing them.
- **Pros**:
  - Protects against XSS attacks because scripts cannot access cookies with `HttpOnly`.
  - Can be made secure by adding the `Secure` flag to ensure transmission over HTTPS only.
- **Cons**:
  - Requires additional backend setup (e.g., to handle `Set-Cookie` and token rotation).
  - Vulnerable to **Cross-Site Request Forgery (CSRF)** unless proper CSRF protection is implemented.

## **Best Practices for Storing Tokens**

### **Short-Term Access Tokens (e.g., JWTs)**
- Store **access tokens** in **memory** (not in localStorage or sessionStorage) for short-lived usage.
- Use cookies with the `HttpOnly` and `Secure` flags if you need to store them persistently.

### **Refresh Tokens**
- Store **refresh tokens** in **HttpOnly cookies**, as these tokens are long-lived and require greater protection.

### **Hybrid Approach**
- Use `HttpOnly` cookies for refresh tokens and keep access tokens in memory.
- Issue short-lived access tokens and use the refresh token to obtain new access tokens.

## **Preventing Token Theft**

### **1. Protect Against Cross-Site Scripting (XSS)**
- **How**:
  - Validate and sanitize user input to prevent injection of malicious scripts.
  - Use Content Security Policy (CSP) headers to restrict the execution of scripts.
  - Set `HttpOnly` and `Secure` flags for cookies to prevent access by JavaScript.

### **2. Secure Token Transmission**
- Always use **HTTPS** to encrypt tokens in transit.
- Never transmit tokens over unencrypted HTTP connections.

### **3. Protect Against Cross-Site Request Forgery (CSRF)**
- **How**:
  - Use CSRF tokens in addition to authentication tokens to validate requests.
  - Implement **SameSite cookies** to restrict the scope of cookies:
    - `SameSite=Strict` for maximum protection (prevents cookies from being sent with cross-site requests).
    - `SameSite=Lax` for moderate protection.
      
### **4. Implement Token Expiry**
- Set a short lifespan for access tokens (e.g., 5–15 minutes) to minimize the risk of misuse.
- Use refresh tokens with a longer lifespan to allow seamless re-authentication.

### **5. Limit Token Scope**
- Use **scopes** in your tokens to limit what actions the token can perform.
- This minimizes the damage a stolen token can cause.

### **6. Revoke Tokens**
- Maintain a **revocation list** on the server to invalidate tokens that are suspected to be stolen or compromised.
## 🚀 Advanced Concepts
### Token Types
- Access Token
- Refresh Token
- Stateless Authentication

## **Summary**
- Use **HttpOnly cookies** for refresh tokens to secure them against XSS.
- Avoid storing tokens in `localStorage` or `sessionStorage` if possible.
- Mitigate XSS and CSRF risks with proper security headers and token management.
- Implement token expiration, rotation, and revocation to reduce the impact of a stolen token.
  
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
## **Token Revocation**

### **What is Token Revocation?**
- Token revocation is the process of invalidating an issued token to prevent its further use.
- It is essential for scenarios such as user logout, compromised tokens, or account deactivation.

### **How is Token Revocation Implemented?**
1. **Blacklist/Revocation List**:
   - Maintain a server-side list of revoked tokens.
   - Check the list during token validation.
   - Drawback: This approach may not scale well for high-volume systems.

2. **Token Expiry**:
   - Issue short-lived tokens that expire quickly, reducing the window for misuse.
   - Combine with refresh tokens to allow legitimate users to obtain new tokens after expiration.

3. **Token Rotation**:
   - Rotate refresh tokens upon each use.
   - Invalidate the previous refresh token when a new one is issued.

4. **Logout Mechanism**:
   - Invalidate tokens issued to the user during logout by adding them to the revocation list or deleting session entries.

5. **Revocation with Stateful Tokens**:
   - Use server-side session storage for tokens and delete the session to revoke access.

### **Challenges in Token Revocation**
- Revoking stateless tokens like JWTs is challenging because they are not stored on the server.
- Blacklisting tokens requires additional server overhead and storage.

---
## **Refresh Tokens**

### **What is a Refresh Token?**
- A refresh token is a long-lived token used to obtain new access tokens without requiring the user to reauthenticate.
- It is typically issued alongside a short-lived access token.

### **How Does a Refresh Token Work?**
1. The client stores the refresh token securely (e.g., in an HttpOnly cookie).
2. When the access token expires, the client sends the refresh token to the server.
3. The server validates the refresh token and issues a new access token (and optionally a new refresh token).

### **Best Practices for Refresh Tokens**
1. **Secure Storage**:
   - Store refresh tokens in HttpOnly cookies to protect against XSS attacks.

2. **Token Rotation**:
   - Issue a new refresh token upon each use.
   - Invalidate the old refresh token to prevent reuse (replay attacks).

3. **Limited Scope**:
   - Restrict refresh token functionality to specific endpoints.

4. **Shortened Expiry**:
   - Use reasonable expiration times for refresh tokens to reduce their exposure.

5. **Revocation Mechanisms**:
   - Maintain a blacklist or revocation list to invalidate compromised refresh tokens.

6. **IP and Device Binding**:
   - Bind refresh tokens to a specific device or IP address to prevent misuse on unauthorized systems.

7. **Detection of Anomalous Behavior**:
   - Monitor token usage for unusual patterns (e.g., location changes) and revoke tokens if necessary.

### **Advantages of Refresh Tokens**
- Reduces the need for frequent reauthentication by the user.
- Enhances security by allowing short-lived access tokens.

### **Disadvantages of Refresh Tokens**
- If a refresh token is stolen, it could be misused to generate new access tokens.
- Requires additional server-side logic to manage token rotation and revocation.
---

# Practical Questions on Token Revocation and Token Concepts

## **Token Revocation**

### **1. Why can’t we update the claims of an issued token, like the expiration (exp) claim, to force it to expire earlier?**
**Answer:**
Modifying the `exp` claim or any other claim in an issued token would invalidate the token's signature, as the signature is a cryptographic hash of the token's header, payload, and the secret key. Changing any part of the token (like `exp`) would result in a mismatch between the token and its signature, causing it to fail validation. when you update any claims and encrypt again the encoded data, you essentially creating new token. It doesn't solve the token revocation problem.

OR

Updating an issued token’s claims would invalidate its signature because the signature is generated using the original header and payload. Changing any part of the token requires regenerating the signature using the private key, which the client does not possess. This ensures that tokens remain immutable and tamper-proof.

---

### **2. How would you implement token revocation in a distributed system?**
**Answer:**
In a distributed system:
1. Use a shared database or cache (e.g., Redis) to maintain a blacklist of revoked tokens.
2. Use message queues or event-driven architectures to notify all services about a token revocation.
3. Ensure that services periodically sync with the central revocation list.
4. For performance, consider using token versioning and only checking the revocation list for specific tokens.

---

### **3. What are the trade-offs between using a blacklist for token revocation versus using short-lived tokens with refresh tokens?**
**Answer:**
- **Blacklist:** Allows real-time revocation but requires centralized storage and lookup, which can become a bottleneck.
- **Short-lived tokens with refresh tokens:** Reduces the need for a blacklist but increases the risk of unauthorized use if tokens are stolen.
- **Trade-off:** Short-lived tokens are better for stateless systems, while blacklists provide immediate control over token validity.

---

### **4. How does rotating refresh tokens help prevent token replay attacks?**
**Answer:**
Rotating refresh tokens ensures that each refresh token can only be used once. If an attacker steals a refresh token, it becomes invalid as soon as it is used. The server issues a new token upon successful validation, invalidating the old one. This limits the attack window.

---

### **5. Design a system to notify all microservices about a revoked token.**
**Answer:**
1. Use an event-driven system with a message broker (e.g., Kafka, RabbitMQ) to publish token revocation events.
2. Each microservice subscribes to these events and updates its local cache or revocation list.
3. Include a mechanism for periodic sync to ensure consistency in case of missed events.

---

### **6. How can you bind a token to a specific device or IP address?**
**Answer:**
- Include device or IP metadata as claims in the token.
- Validate these claims on each request.
- Limitations: IP addresses can change, and device-specific identifiers may not always be available.

---

### **7. How would you handle token revocation for a stateless authentication system?**
**Answer:**
Use a distributed cache (e.g., Redis) to maintain a blacklist of revoked tokens and check the cache during token validation. Another approach is to use short-lived tokens combined with refresh tokens, reducing the need for real-time revocation.

---

## **Refresh Tokens**

### **8. What is the difference between access tokens and refresh tokens?**
**Answer:**
- **Access Tokens:** Short-lived tokens used to access resources. They contain claims about the user and permissions.
- **Refresh Tokens:** Long-lived tokens used to obtain new access tokens. They are stored securely and not shared with APIs.

---

### **9. How does the use of refresh tokens improve security in token-based authentication?**
**Answer:**
Refresh tokens allow access tokens to have a shorter lifespan, reducing the impact of token theft. Even if an access token is stolen, it will expire quickly, and the refresh token remains securely stored and protected.

---

### **10. What are the risks of using long-lived refresh tokens? How can you mitigate them?**
**Answer:**
**Risks:**
- If stolen, they provide prolonged access.
- Increased attack surface if not properly secured.
**Mitigations:**
- Use token rotation to invalidate old tokens.
- Store tokens securely (e.g., HttpOnly cookies).
- Implement device binding or IP restrictions.

---

### **11. How would you implement a refresh token revocation strategy?**
**Answer:**
1. Store refresh tokens in a database with their status.
2. Revoke tokens explicitly by marking them as invalid.
3. Use a `jti` (JWT ID) claim to identify and track tokens.

---

### **12. How can you ensure that refresh tokens are not reused during rotation?**
**Answer:**
Generate a new refresh token each time the old one is used. Mark the old token as invalid. If a token is reused, detect it and revoke all related tokens.

---

### **13. What is the best way to store refresh tokens on the client side?**
**Answer:**
- Use secure, HttpOnly cookies to prevent JavaScript access.
- Avoid localStorage/sessionStorage for sensitive tokens.
- Consider using secure device storage APIs on mobile devices.

---

## **General Token Questions**

### **14. How does OAuth 2.0 handle token revocation?**
**Answer:**
OAuth 2.0 provides a token revocation endpoint where clients can explicitly request the revocation of a token. The server marks the token as invalid and ensures it cannot be used again.

---

### **15. What are some ways to prevent tokens from being stolen during transmission?**
**Answer:**
- Use HTTPS for all communications.
- Implement Content Security Policies (CSPs).
- Use SameSite cookies to prevent cross-site attacks.
- Enable certificate pinning.

---

### **16. Design an API that supports token revocation and refresh token rotation.**
**Answer:**
1. **Endpoints:**
   - `/revoke`: Accepts token to revoke.
   - `/refresh`: Issues new tokens after validating the old refresh token.
2. **Logic:**
   - Store token states in a database.
   - Check the database for token validity during each request.
   - Rotate refresh tokens and blacklist the old ones.

---

### **17. What is the impact of token revocation on Single Sign-On (SSO) systems?**
**Answer:**
In SSO, revoking a token for one service may require notifying other services. Use a central revocation list or event-driven architecture to propagate revocation events.

---

### **18. How would you handle a scenario where a token is compromised but has not yet expired?**
**Answer:**
1. Add the token to a revocation list.
2. Force a logout for the user.
3. Notify the user and recommend resetting credentials.

---

### **19. How can you revoke tokens in real-time for a high-traffic application?**
**Answer:**
- Use a distributed in-memory cache (e.g., Redis) for token blacklists.
- Implement token versioning to avoid frequent database checks.
- Use event-driven notifications to update revocation lists across servers.

---

### **20. How does the `jti` (JWT ID) claim help in token revocation?**
**Answer:**
The `jti` claim uniquely identifies a token. It can be stored in a database or cache to track token statuses. When a token is revoked, its `jti` is marked invalid, and any requests with that token are denied.

---

## **21. How can we handle token expiration during an ongoing user session?**
**Answer:**
- Use silent token renewal mechanisms, such as iframe-based refresh (for SPAs).
- Notify the user proactively before the token expires and renew it in the background.

---

## **22. What is token binding, and how does it enhance security?**
**Answer:**
Token binding ties the token to a specific client or device. For example, tokens can include cryptographic keys or device identifiers in their payload, ensuring that even if the token is stolen, it cannot be used from a different device.

---

## **23. What are the various approaches to token revocation?**
**Answer:**
1. **Server-Side Blacklists**: Maintain a blacklist of tokens to deny access to revoked tokens.
2. **Token Versioning**: Associate tokens with a `version` claim and invalidate tokens when a new version is issued.
3. **Short-Lived Tokens with Refresh Tokens**: Minimize the impact of stolen tokens by using short expiration times.
4. **Revoke Associated Refresh Tokens**: Invalidate all tokens issued with a specific refresh token.
5. **OAuth 2.0 Revocation Endpoint**: Use a dedicated endpoint to revoke tokens.

---

## **24. What are the different types of tokens?**
**Answer:**
1. **Bearer Tokens**: Most common, used in HTTP headers for authentication.
2. **Refresh Tokens**: Long-lived tokens used to generate new access tokens.
3. **ID Tokens**: Contain user identity information (used in OpenID Connect).
4. **Macaroon Tokens**: Include caveats (restrictions) for fine-grained access control.
5. **Opaque Tokens**: Cannot be decoded; require a server lookup to validate.

---

## **25. What are the key considerations when implementing refresh tokens?**
**Answer:**
- **Storage**: Securely store refresh tokens (e.g., in secure cookies or OS-provided secure storage).
- **Revocation**: Implement mechanisms to revoke refresh tokens if compromised.
- **Rotation**: Use refresh token rotation to ensure each token can only be used once.
- **Expiration**: Set a reasonable expiration time for refresh tokens to limit misuse.

---

## **26. What are the challenges of using short-lived tokens?**
**Answer:**
- Frequent token expiration can result in higher latency due to repeated refresh requests.
- Requires robust handling of token renewal on the client side.
- Increased complexity in handling offline scenarios where refresh tokens are required.

---

## **27. How does token versioning help in token revocation?**
**Answer:**
Token versioning involves assigning a version number to tokens and storing this version in the user’s record on the server. When a token is revoked (e.g., due to logout), the server updates the version number. Any tokens with an outdated version are considered invalid.

---

## **28. Why should access tokens have a short expiration time?**
**Answer:**
Short expiration times limit the impact of stolen tokens by reducing the window of vulnerability. Even if a token is compromised, it will only remain valid for a short period, reducing the risk of unauthorized access.

---

## **29. Why is rotating refresh tokens more secure than static refresh tokens?**
**Answer:**
Rotating refresh tokens ensure that each token is valid for a single use only. If a token is intercepted, it cannot be reused as it becomes invalid after being exchanged for a new token. This reduces the risk of misuse.

---

## **30. How can we handle token replay attacks?**
**Answer:**
- Use token binding to tie tokens to specific clients or sessions.
- Implement one-time-use tokens for critical operations.
- Monitor unusual usage patterns (e.g., multiple requests with the same token).

---

## **31. What are Macaroon tokens, and when should they be used?**
**Answer:**
Macaroon tokens are cryptographically secured tokens with caveats (restrictions) that define their scope, validity period, or other constraints. They are useful for fine-grained access control, especially in distributed systems where traditional token revocation mechanisms are difficult to implement.

---
