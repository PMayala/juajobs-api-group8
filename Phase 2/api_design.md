# JuaJobs API Design Specification

## 1. API Versioning Strategy

### Version Format
- Format: `v{major}.{minor}`
- Example: `v1.0`, `v1.1`, `v2.0`

### Implementation Method
```
Base URL: https://api.juajobs.com/v1/
Version Header: X-API-Version: 1.0
Accept Header: application/vnd.juajobs.v1+json
```

### Versioning Rules
1. Major version changes (`v1` → `v2`):
   - Breaking changes to response structure
   - Removal of endpoints
   - Fundamental auth changes

2. Minor version changes (`v1.0` → `v1.1`):
   - Adding new endpoints
   - Adding optional parameters
   - Extending response objects

### Deprecation Policy
- 6 months notice for major version deprecation
- Sunset header included: `Sunset: Thu, 01 Jan 2025 00:00:00 GMT`
- Deprecated header for endpoints: `X-API-Deprecated: true`
- Migration guides provided for version transitions

## 2. Core API Endpoints

### User Management
```
GET    /users                  # List users (admin only)
POST   /users                  # Create user
GET    /users/{id}            # Get user details
PATCH  /users/{id}            # Update user
DELETE /users/{id}            # Deactivate user
GET    /users/{id}/profile    # Get worker profile
POST   /users/{id}/verify     # Verify user
```

### Job Postings
```
GET    /jobs                  # List job postings
POST   /jobs                  # Create job posting
GET    /jobs/{id}            # Get job details
PATCH  /jobs/{id}            # Update job posting
DELETE /jobs/{id}            # Remove job posting
GET    /jobs/{id}/applications # List applications
POST   /jobs/{id}/applications # Submit application
```

### Applications
```
GET    /applications         # List applications (filtered by user)
GET    /applications/{id}    # Get application details
PATCH  /applications/{id}    # Update application
DELETE /applications/{id}    # Withdraw application
POST   /applications/{id}/interview # Schedule interview
```

### Reviews
```
GET    /reviews             # List reviews
POST   /reviews            # Create review
GET    /reviews/{id}       # Get review details
PATCH  /reviews/{id}       # Update review
GET    /users/{id}/reviews # Get user reviews
```

### Skills & Categories
```
GET    /skills             # List skills
GET    /skills/{id}        # Get skill details
GET    /categories        # List categories
GET    /categories/{id}   # Get category details
```

### Payments
```
GET    /payments          # List payments
POST   /payments         # Create payment
GET    /payments/{id}    # Get payment details
POST   /payments/webhook # Payment webhook
```

## 3. Query Parameters & Filtering

### Standard Query Parameters
```
?page=1              # Page number
?per_page=20        # Items per page
?sort=created_at    # Sort field
?order=desc         # Sort order
?fields=id,name     # Field selection
?include=reviews    # Include related resources
```

### Filtering Parameters
```
?filter[status]=active           # Simple filter
?filter[created_at][gt]=2024-01 # Range filter
?filter[skills]=1,2,3           # Multiple values
?search=keyword                 # Full-text search
?near=lat,lng&radius=10km      # Geo search
```

### Field Selection
```
?fields=id,title,description    # Basic fields
?fields[jobs]=id,title         # Resource-specific fields
?fields[user]=id,name          # Related resource fields
```

### Pagination Response Format
```json
{
  "data": [...],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 100
  },
  "links": {
    "self": "...",
    "first": "...",
    "last": "...",
    "prev": null,
    "next": "..."
  }
}
```

## 4. HTTP Status Codes & Error Handling

### Success Codes
- 200 OK: Successful request
- 201 Created: Resource created
- 204 No Content: Successful deletion

### Client Error Codes
- 400 Bad Request: Invalid parameters
- 401 Unauthorized: Authentication required
- 403 Forbidden: Insufficient permissions
- 404 Not Found: Resource not found
- 409 Conflict: Resource conflict
- 422 Unprocessable Entity: Validation failed

### Server Error Codes
- 500 Internal Server Error: Server failure
- 503 Service Unavailable: System maintenance

### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid parameters",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Invalid email format"
      }
    ],
    "request_id": "req_123",
    "documentation_url": "https://docs.juajobs.com/errors/VALIDATION_ERROR"
  }
}
```

### Error Categories
1. Validation Errors (VALIDATION_*)
2. Authentication Errors (AUTH_*)
3. Permission Errors (PERMISSION_*)
4. Resource Errors (RESOURCE_*)
5. Business Logic Errors (BUSINESS_*)
6. System Errors (SYSTEM_*)

## 5. Request/Response Examples

### Create Job Posting
```http
POST /v1/jobs
Content-Type: application/json
Authorization: Bearer <token>

{
  "title": "Senior Software Developer",
  "description": "...",
  "category_id": "cat_123",
  "required_skills": ["skill_1", "skill_2"],
  "location": {
    "type": "ONSITE",
    "country": "KE",
    "city": "Nairobi"
  },
  "payment": {
    "type": "FIXED",
    "amount": 1000,
    "currency": "KES"
  }
}
```

### Success Response
```http
HTTP/1.1 201 Created
Location: /v1/jobs/job_123

{
  "data": {
    "id": "job_123",
    "type": "job_posting",
    "attributes": {
      "title": "Senior Software Developer",
      "status": "DRAFT",
      "created_at": "2024-01-20T12:00:00Z"
    },
    "relationships": {
      "client": {
        "data": { "type": "user", "id": "user_123" }
      }
    }
  }
}
```

### Error Response
```http
HTTP/1.1 422 Unprocessable Entity

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid job posting parameters",
    "details": [
      {
        "field": "payment.amount",
        "code": "MIN_VALUE",
        "message": "Amount must be greater than 0"
      }
    ]
  }
}
```

## 6. API Security

### Authentication
- Bearer token authentication
- API key for server-to-server
- OAuth2 for third-party integrations

### Rate Limiting
```http
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4999
X-RateLimit-Reset: 1640995200
```

### Security Headers
```http
Content-Security-Policy: default-src 'none'
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```

## 7. Caching Strategy

### Cache Headers
```http
Cache-Control: private, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

### Conditional Requests
```http
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
``` 