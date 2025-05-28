# Error Handling & Status Codes Specification

## Overview
This document defines the standardized approach to error handling across the JuaJobs API, including HTTP status codes, error response formats, and error categorization.

## 1. HTTP Status Codes

### Success Codes (2xx)
| Code | Name | Description | Usage |
|------|------|-------------|--------|
| 200 | OK | Request succeeded | General success response |
| 201 | Created | Resource created | After POST requests |
| 202 | Accepted | Request accepted | Long-running processes |
| 204 | No Content | Request succeeded, no content | After DELETE |

### Client Error Codes (4xx)
| Code | Name | Description | Usage |
|------|------|-------------|--------|
| 400 | Bad Request | Invalid request format | Malformed requests |
| 401 | Unauthorized | Authentication required | Missing/invalid auth |
| 403 | Forbidden | Permission denied | Insufficient rights |
| 404 | Not Found | Resource not found | Invalid resource ID |
| 409 | Conflict | Resource conflict | Duplicate entries |
| 422 | Unprocessable Entity | Validation failed | Invalid parameters |
| 429 | Too Many Requests | Rate limit exceeded | API throttling |

### Server Error Codes (5xx)
| Code | Name | Description | Usage |
|------|------|-------------|--------|
| 500 | Internal Server Error | Server failure | Unexpected errors |
| 502 | Bad Gateway | Invalid response | Gateway errors |
| 503 | Service Unavailable | System maintenance | Planned downtime |
| 504 | Gateway Timeout | Request timeout | Long operations |

## 2. Error Response Format

### Standard Error Structure
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": [
      {
        "field": "affected_field",
        "code": "SPECIFIC_ERROR",
        "message": "Field-specific message"
      }
    ],
    "request_id": "req_123abc",
    "documentation_url": "https://docs.juajobs.com/errors/ERROR_CODE"
  }
}
```

### Error Properties
| Property | Type | Required | Description |
|----------|------|----------|-------------|
| code | String | Yes | Unique error identifier |
| message | String | Yes | User-friendly message |
| details | Array | No | Detailed error info |
| request_id | String | Yes | Request tracking ID |
| documentation_url | String | No | Help documentation |

## 3. Error Categories

### Validation Errors (VALIDATION_*)
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
      },
      {
        "field": "phone",
        "code": "REQUIRED",
        "message": "Phone number is required"
      }
    ]
  }
}
```

### Authentication Errors (AUTH_*)
```json
{
  "error": {
    "code": "AUTH_INVALID_TOKEN",
    "message": "Invalid or expired authentication token",
    "details": [
      {
        "code": "TOKEN_EXPIRED",
        "message": "Token expired at 2024-01-20T12:00:00Z"
      }
    ]
  }
}
```

### Permission Errors (PERMISSION_*)
```json
{
  "error": {
    "code": "PERMISSION_DENIED",
    "message": "Insufficient permissions for this action",
    "details": [
      {
        "code": "ROLE_REQUIRED",
        "message": "Admin role required for this operation"
      }
    ]
  }
}
```

### Resource Errors (RESOURCE_*)
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found",
    "details": [
      {
        "code": "INVALID_ID",
        "message": "Job posting with ID 'job_123' does not exist"
      }
    ]
  }
}
```

### Business Logic Errors (BUSINESS_*)
```json
{
  "error": {
    "code": "BUSINESS_RULE_VIOLATION",
    "message": "Operation violates business rules",
    "details": [
      {
        "code": "INSUFFICIENT_FUNDS",
        "message": "Account balance too low for withdrawal"
      }
    ]
  }
}
```

### System Errors (SYSTEM_*)
```json
{
  "error": {
    "code": "SYSTEM_ERROR",
    "message": "An unexpected error occurred",
    "details": [
      {
        "code": "DATABASE_ERROR",
        "message": "Database connection failed"
      }
    ]
  }
}
```

## 4. Error Handling Guidelines

### General Principles
1. Be consistent with error formats
2. Use appropriate HTTP status codes
3. Provide actionable error messages
4. Include request tracking IDs
5. Document all error codes

### Security Considerations
1. Don't expose sensitive information
2. Sanitize error messages
3. Log detailed errors internally
4. Rate limit error responses
5. Validate error outputs

### User Experience
1. Use clear, friendly messages
2. Provide solution suggestions
3. Link to relevant documentation
4. Support multiple languages
5. Include support contact info

## 5. Common Error Scenarios

### Input Validation
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "amount",
        "code": "MIN_VALUE",
        "message": "Amount must be greater than 0"
      }
    ]
  }
}
```

### Rate Limiting
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "details": [
      {
        "code": "HOURLY_LIMIT",
        "message": "Hourly limit of 1000 requests exceeded",
        "reset_at": "2024-01-20T13:00:00Z"
      }
    ]
  }
}
```

### Business Rules
```json
{
  "error": {
    "code": "BUSINESS_RULE_VIOLATION",
    "message": "Cannot perform operation",
    "details": [
      {
        "code": "JOB_CLOSED",
        "message": "Cannot apply to closed job posting"
      }
    ]
  }
}
```

## 6. Implementation Best Practices

### Error Logging
1. Log all errors with context
2. Include stack traces internally
3. Track error frequencies
4. Monitor error patterns
5. Alert on critical errors

### Error Recovery
1. Implement retry mechanisms
2. Provide fallback options
3. Handle partial failures
4. Support idempotent operations
5. Maintain system state

### Client Guidelines
1. Handle all error codes
2. Implement exponential backoff
3. Display user-friendly messages
4. Support offline operations
5. Cache error responses

## 7. Error Documentation

### API Reference
1. List all error codes
2. Provide error examples
3. Document recovery steps
4. Include troubleshooting guides
5. Update regularly

### Developer Support
1. Error debugging tools
2. Testing environments
3. Error simulation endpoints
4. Support channels
5. Documentation feedback 