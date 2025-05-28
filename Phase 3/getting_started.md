# Getting Started with JuaJobs API

Welcome to the JuaJobs API! This guide will help you get started with integrating our API into your applications.

## Table of Contents
1. [Authentication](#authentication)
2. [Making Your First Request](#making-your-first-request)
3. [Common Use Cases](#common-use-cases)
4. [Error Handling](#error-handling)
5. [Best Practices](#best-practices)

## Authentication

### Getting API Credentials
1. Create a JuaJobs account at [https://juajobs.com/signup](https://juajobs.com/signup)
2. Navigate to the Developer Dashboard
3. Create a new API key or OAuth2 application

### Authentication Methods

#### Bearer Token (OAuth2)
```http
GET /v1/users
Authorization: Bearer YOUR_ACCESS_TOKEN
```

#### API Key (Server-to-Server)
```http
GET /v1/users
X-API-Key: YOUR_API_KEY
```

### OAuth2 Flow
1. Redirect users to authorization URL:
```
https://auth.juajobs.com/oauth/authorize?
  client_id=YOUR_CLIENT_ID&
  response_type=code&
  scope=read write&
  redirect_uri=YOUR_REDIRECT_URI
```

2. Receive authorization code

3. Exchange code for access token:
```http
POST https://auth.juajobs.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTHORIZATION_CODE&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET&
redirect_uri=YOUR_REDIRECT_URI
```

## Making Your First Request

### List Job Postings
```http
GET https://api.juajobs.com/v1/jobs
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Response:
```json
{
  "data": [
    {
      "id": "job_123",
      "type": "job_posting",
      "attributes": {
        "title": "Senior Software Developer",
        "description": "...",
        "status": "PUBLISHED",
        "created_at": "2024-01-20T12:00:00Z"
      }
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 100
  }
}
```

### Create a Job Posting
```http
POST https://api.juajobs.com/v1/jobs
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "title": "Senior Software Developer",
  "description": "We are looking for an experienced developer...",
  "category_id": "cat_123",
  "required_skills": ["skill_1", "skill_2"],
  "location": {
    "type": "ONSITE",
    "country": "KE",
    "city": "Nairobi"
  },
  "payment": {
    "type": "FIXED",
    "amount": 1000.00,
    "currency": "KES"
  }
}
```

## Common Use Cases

### 1. Job Search Integration

Search for jobs with filters:
```http
GET /v1/jobs?
  search=software+developer&
  filter[status]=PUBLISHED&
  filter[location][country]=KE&
  near=-1.2921,36.8219,10km
```

### 2. Worker Profile Management

Create a worker profile:
```http
POST /v1/users/{user_id}/profile
Content-Type: application/json

{
  "title": "Full Stack Developer",
  "skills": ["skill_1", "skill_2"],
  "experience_years": 5,
  "hourly_rate": {
    "amount": 50.00,
    "currency": "USD"
  }
}
```

### 3. Job Application Flow

Submit an application:
```http
POST /v1/jobs/{job_id}/applications
Content-Type: application/json

{
  "cover_letter": "I am interested in this position...",
  "expected_salary": {
    "amount": 5000.00,
    "currency": "KES"
  },
  "availability": "2024-02-01"
}
```

## Error Handling

### Common Error Codes
- 400: Bad Request (invalid parameters)
- 401: Unauthorized (invalid/missing token)
- 403: Forbidden (insufficient permissions)
- 404: Not Found (invalid resource ID)
- 422: Validation Error (invalid input)
- 429: Rate Limit Exceeded

### Error Response Example
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Invalid email format"
      }
    ],
    "request_id": "req_123abc"
  }
}
```

### Rate Limiting
Monitor rate limits through response headers:
```http
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4999
X-RateLimit-Reset: 1640995200
```

## Best Practices

### 1. Pagination
Always use pagination for list endpoints:
```http
GET /v1/jobs?page=2&per_page=20
```

### 2. Field Selection
Request only needed fields:
```http
GET /v1/jobs?fields=id,title,status
```

### 3. Include Related Resources
Reduce API calls by including related data:
```http
GET /v1/jobs?include=client,category
```

### 4. Error Handling
- Implement exponential backoff for rate limits
- Handle all possible error codes
- Log request IDs for debugging
- Show user-friendly error messages

### 5. Security
- Store credentials securely
- Use HTTPS for all requests
- Implement token refresh logic
- Validate user input
- Follow security best practices

## SDK Libraries

### Official Libraries
- [Python SDK](https://github.com/juajobs/python-sdk)
- [JavaScript SDK](https://github.com/juajobs/javascript-sdk)
- [PHP SDK](https://github.com/juajobs/php-sdk)

### Example: Python SDK
```python
from juajobs import Client

client = Client('YOUR_ACCESS_TOKEN')

# List jobs
jobs = client.jobs.list(
    status='PUBLISHED',
    location_country='KE'
)

# Create job
job = client.jobs.create(
    title='Software Developer',
    description='...',
    category_id='cat_123'
)
```

## Need Help?

- [API Documentation](https://docs.juajobs.com)
- [Developer Forum](https://forum.juajobs.com)
- [Support Email](mailto:api@juajobs.com)
- [Status Page](https://status.juajobs.com) 