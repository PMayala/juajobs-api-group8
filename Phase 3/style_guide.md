# JuaJobs API Style Guide

## 1. Naming Conventions

### Resource Names
- Use plural nouns for collection endpoints (e.g., `/users`, `/jobs`, `/applications`)
- Use lowercase letters and hyphens for multi-word resources (e.g., `/job-categories`)
- Keep names concise but descriptive
- Use consistent terminology across all endpoints

### Parameter Names
- Use snake_case for all parameter names
- Use descriptive names that indicate the parameter's purpose
- Prefix boolean parameters with verbs (e.g., `is_`, `has_`, `should_`)
- Use consistent parameter names across similar endpoints

Examples:
```
✓ first_name
✓ is_verified
✓ has_attachments
✗ firstName
✗ verified
✗ attach
```

### Query Parameters
- Use square brackets for nested filters: `filter[status]`, `filter[created_at][gt]`
- Use consistent parameter names for common operations:
  - `page` for pagination
  - `per_page` for page size
  - `sort` for sorting field
  - `order` for sort direction
  - `fields` for field selection
  - `include` for related resources

### Response Field Names
- Use snake_case for all field names
- Use consistent field names across all resources
- Include resource type and ID in relationships
- Use ISO formats for standardized fields

## 2. Data Formatting Standards

### Dates and Times
- Use ISO 8601 format for all dates and times
- Always use UTC timezone
- Include timezone offset when relevant
- Format: `YYYY-MM-DDTHH:mm:ss.sssZ`

Examples:
```
✓ "2024-01-20T12:00:00Z"
✓ "2024-01-20"  (date only)
✗ "01/20/2024"
✗ "2024-01-20 12:00:00"
```

### Currency and Money
- Use ISO 4217 currency codes
- Specify amount as decimal number
- Include both amount and currency in payment objects
- Support major African currencies by default

Example:
```json
{
  "amount": 1000.00,
  "currency": "KES",
  "formatted": "KES 1,000.00"
}
```

### Phone Numbers
- Use E.164 format for phone numbers
- Include country code
- Remove spaces and special characters
- Validate format on input

Examples:
```
✓ "+254712345678"
✓ "+27821234567"
✗ "0712345678"
✗ "+254 712 345 678"
```

### Localized Data
- Use ISO 639-1 language codes
- Use ISO 3166-1 alpha-2 country codes
- Support multiple languages through Accept-Language header
- Include language code in localized content

Example:
```json
{
  "title": {
    "en": "Software Developer",
    "sw": "Mtengenezaji wa Programu"
  },
  "country": "KE",
  "language": "en"
}
```

### Coordinates and Location
- Use decimal degrees for coordinates
- Order: latitude, longitude
- Use standardized unit abbreviations (km, mi)
- Include location type when relevant

Example:
```json
{
  "location": {
    "type": "ONSITE",
    "coordinates": {
      "latitude": -1.2921,
      "longitude": 36.8219
    },
    "city": "Nairobi",
    "country": "KE"
  }
}
```

## 3. Documentation Templates

### Endpoint Documentation
```markdown
### Endpoint Name
**GET /resource**

Description of what the endpoint does and when to use it.

#### Authentication
- Required scopes
- Rate limits

#### Parameters
| Name | Type | Required | Description |
|------|------|----------|-------------|
| param | type | Yes/No | Description |

#### Request Example
\`\`\`http
GET /v1/resource?param=value
Authorization: Bearer <token>
\`\`\`

#### Response Example
\`\`\`json
{
  "data": {
    // Response structure
  }
}
\`\`\`

#### Error Codes
| Code | Description |
|------|-------------|
| 400  | Reason |
```

### Schema Documentation
```markdown
### Resource Name
Description of the resource and its purpose.

#### Attributes
| Name | Type | Required | Description |
|------|------|----------|-------------|
| attr | type | Yes/No | Description |

#### Relationships
| Name | Type | Description |
|------|------|-------------|
| rel  | type | Description |
```

## 4. API Extension Guidelines

### Adding New Endpoints
1. Follow existing naming conventions
2. Document all parameters and responses
3. Include example requests and responses
4. Add appropriate validation rules
5. Consider backwards compatibility

### Versioning Changes
1. Document all changes in changelog
2. Maintain compatibility in minor versions
3. Provide migration guides for major versions
4. Support previous version during transition
5. Communicate deprecation timelines

### Error Handling
1. Use standard error response format
2. Include specific error codes
3. Provide actionable error messages
4. Document all possible error scenarios
5. Include request ID for tracking

### Security Considerations
1. Require HTTPS for all endpoints
2. Implement rate limiting
3. Validate all inputs
4. Use appropriate authentication
5. Follow OWASP guidelines

## 5. Best Practices

### Performance
1. Implement caching where appropriate
2. Use pagination for large collections
3. Allow field selection to reduce payload
4. Optimize database queries
5. Monitor response times

### Maintainability
1. Keep documentation up to date
2. Write clear commit messages
3. Follow code style guide
4. Review API changes
5. Maintain test coverage

### Scalability
1. Design for horizontal scaling
2. Use appropriate caching strategies
3. Implement rate limiting
4. Monitor API usage
5. Plan for growth

### Security
1. Validate all inputs
2. Sanitize all outputs
3. Use appropriate encryption
4. Implement access controls
5. Regular security audits 