# JuaJobs API Authentication Design

## 1. Authentication Methods

### OAuth2.0 + JWT (Primary Method)
Designed for web and mobile applications with user context.

#### Flow Types
1. **Authorization Code Flow with PKCE**
   - For mobile and single-page applications
   - Secure against CSRF and authorization code interception
   - Recommended for Android and iOS apps

```http
# Step 1: Generate Code Verifier and Challenge
code_verifier = generate_random_string(128)
code_challenge = base64url_encode(sha256(code_verifier))

# Step 2: Authorization Request
GET https://auth.juajobs.com/oauth/authorize
  ?client_id=YOUR_CLIENT_ID
  ?response_type=code
  ?code_challenge=CODE_CHALLENGE
  ?code_challenge_method=S256
  ?redirect_uri=YOUR_REDIRECT_URI
  ?scope=read write
  ?state=CSRF_TOKEN

# Step 3: Exchange Code for Tokens
POST https://auth.juajobs.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
code_verifier=CODE_VERIFIER&
client_id=YOUR_CLIENT_ID&
redirect_uri=YOUR_REDIRECT_URI
```

2. **Password Grant (Limited Use)**
   - Only for first-party applications
   - Requires strong justification for use
   - Not recommended for third-party integrations

```http
POST https://auth.juajobs.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&
username=user@example.com&
password=user_password&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET
```

### API Keys (Server-to-Server)
For backend service integrations without user context.

```http
GET /v1/jobs
X-API-Key: YOUR_API_KEY
```

## 2. Token Strategy

### Access Tokens (JWT)
- Short-lived (1 hour)
- Contains minimal user claims
- Signed using RS256 (asymmetric)
- Includes standard JWT claims

```json
{
  "iss": "https://auth.juajobs.com",
  "sub": "user_123",
  "aud": "https://api.juajobs.com",
  "exp": 1516239022,
  "iat": 1516235422,
  "scope": "read write",
  "role": "WORKER"
}
```

### Refresh Tokens
- Long-lived (30 days)
- Rotated on use (automatic refresh)
- Secure storage required
- Supports offline access

```http
POST https://auth.juajobs.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token=REFRESH_TOKEN&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET
```

### Mobile Considerations
1. **Offline Access**
   - Extended refresh token validity
   - Background token refresh
   - Queue operations during offline periods

2. **Token Storage**
   - Android: EncryptedSharedPreferences
   - iOS: Keychain Services
   - Never store in plain text

3. **Network Challenges**
   - Implement retry with exponential backoff
   - Cache necessary data locally
   - Queue authentication requests

## 3. Security Requirements

### Client Requirements
1. **Token Storage**
   - Never store tokens in localStorage (web)
   - Use secure storage mechanisms
   - Clear tokens on logout

2. **HTTPS**
   - Always use HTTPS
   - Certificate pinning recommended
   - HSTS required

3. **Input Validation**
   - Validate all user inputs
   - Sanitize data before sending
   - Handle errors gracefully

### Server Requirements
1. **Token Validation**
   - Verify token signature
   - Check expiration time
   - Validate required claims
   - Verify audience and issuer

2. **Rate Limiting**
   - Per client rate limits
   - Graduated rate limiting
   - Clear error messages

3. **Monitoring**
   - Log authentication failures
   - Monitor unusual patterns
   - Alert on security events

## 4. Implementation Guidelines

### Web Applications
```javascript
// Token Management
class TokenManager {
  async getAccessToken() {
    if (this.isTokenExpired()) {
      return this.refreshAccessToken();
    }
    return this.accessToken;
  }

  async refreshAccessToken() {
    const response = await fetch('https://auth.juajobs.com/oauth/token', {
      method: 'POST',
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: this.refreshToken,
        client_id: CLIENT_ID
      })
    });
    
    const data = await response.json();
    this.updateTokens(data);
    return data.access_token;
  }
}
```

### Mobile Applications
```swift
// iOS Example
class TokenManager {
    func storeTokens(_ tokens: Tokens) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: "refresh_token",
            kSecValueData as String: tokens.refreshToken.data(using: .utf8)!,
            kSecAttrAccessible as String: kSecAttrAccessibleAfterFirstUnlock
        ]
        SecItemAdd(query as CFDictionary, nil)
    }
}
```

```kotlin
// Android Example
class TokenManager {
    private val encryptedPrefs by lazy {
        EncryptedSharedPreferences.create(
            "auth_prefs",
            masterKeyAlias,
            context,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }

    fun storeTokens(tokens: Tokens) {
        encryptedPrefs.edit {
            putString("refresh_token", tokens.refreshToken)
            putString("access_token", tokens.accessToken)
        }
    }
}
```

## 5. Security Best Practices

### Token Security
1. **Token Transmission**
   - Always over HTTPS
   - Use secure headers
   - Implement CSRF protection

2. **Token Storage**
   - Use secure storage mechanisms
   - Encrypt sensitive data
   - Regular security audits

3. **Token Revocation**
   - Support immediate revocation
   - Maintain revocation lists
   - Regular token cleanup

### API Security
1. **Request Security**
   - Validate content types
   - Check request origins
   - Implement request signing

2. **Response Security**
   - Use security headers
   - Implement CORS properly
   - Minimize sensitive data

3. **Error Handling**
   - Don't leak sensitive info
   - Use standard error formats
   - Log security events

## 6. Emergency Procedures

### Token Compromise
1. Revoke affected tokens
2. Notify affected users
3. Issue new tokens
4. Monitor for suspicious activity

### Security Breach
1. Revoke all active tokens
2. Force password resets
3. Notify affected users
4. Implement additional security

## 7. Compliance Requirements

### Data Protection
1. Encrypt sensitive data
2. Implement data retention policies
3. Support data export
4. Enable data deletion

### Audit Trail
1. Log authentication events
2. Track token usage
3. Monitor security metrics
4. Regular security reviews 