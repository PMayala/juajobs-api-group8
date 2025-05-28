# Query Parameters & Filtering Specification

## Overview
This document details the standardized approach to filtering, sorting, pagination, and field selection across all JuaJobs API endpoints.

## 1. Common Query Parameters

### Pagination
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | Integer | 1 | Page number |
| per_page | Integer | 20 | Items per page (max: 100) |

### Sorting
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sort | String | created_at | Field to sort by |
| order | String | desc | Sort order (asc/desc) |

### Field Selection
| Parameter | Type | Example | Description |
|-----------|------|---------|-------------|
| fields | String | id,name,email | Comma-separated list of fields |
| fields[resource] | String | fields[user]=id,name | Resource-specific fields |

### Relationships
| Parameter | Type | Example | Description |
|-----------|------|---------|-------------|
| include | String | reviews,profile | Include related resources |
| include[resource] | String | include[reviews]=user | Nested includes |

## 2. Filtering Capabilities

### Simple Filters
```http
GET /jobs?filter[status]=active
GET /users?filter[country]=KE
GET /skills?filter[category]=technology
```

### Multiple Values
```http
GET /jobs?filter[status]=active,draft
GET /users?filter[role]=worker,client
GET /skills?filter[level]=beginner,intermediate
```

### Range Filters
```http
GET /jobs?filter[created_at][gt]=2024-01-01
GET /payments?filter[amount][gte]=1000
GET /reviews?filter[rating][lte]=4
```

### Complex Filters
```http
GET /jobs?filter[and][0][status]=active&filter[and][1][country]=KE
GET /users?filter[or][0][role]=worker&filter[or][1][role]=client
```

### Full-Text Search
```http
GET /jobs?search=software developer
GET /skills?search=javascript
```

### Geo-Location Filters
```http
GET /jobs?near=1.2921,36.8219&radius=10km
GET /workers?location[country]=KE&location[city]=Nairobi
```

## 3. Filter Operators

### Comparison Operators
| Operator | Description | Example |
|----------|-------------|---------|
| eq | Equals | filter[status][eq]=active |
| neq | Not equals | filter[status][neq]=deleted |
| gt | Greater than | filter[amount][gt]=1000 |
| gte | Greater than or equal | filter[rating][gte]=4 |
| lt | Less than | filter[experience][lt]=5 |
| lte | Less than or equal | filter[age][lte]=30 |

### Array Operators
| Operator | Description | Example |
|----------|-------------|---------|
| in | In array | filter[status][in]=active,draft |
| nin | Not in array | filter[role][nin]=admin,moderator |
| all | Contains all | filter[skills][all]=js,python |
| any | Contains any | filter[categories][any]=tech,design |

### String Operators
| Operator | Description | Example |
|----------|-------------|---------|
| like | Pattern matching | filter[name][like]=john% |
| ilike | Case-insensitive pattern | filter[email][ilike]=%@gmail.com |
| regex | Regular expression | filter[phone][regex]=^254 |

## 4. Response Format

### Success Response
```json
{
  "data": [...],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 100,
    "filters_applied": {
      "status": "active",
      "country": "KE"
    }
  },
  "links": {
    "self": "/jobs?page=1&per_page=20",
    "first": "/jobs?page=1&per_page=20",
    "last": "/jobs?page=5&per_page=20",
    "prev": null,
    "next": "/jobs?page=2&per_page=20"
  }
}
```

### Error Response
```json
{
  "error": {
    "code": "INVALID_FILTER",
    "message": "Invalid filter parameters provided",
    "details": [
      {
        "field": "filter[amount]",
        "code": "INVALID_OPERATOR",
        "message": "Operator 'invalid' is not supported"
      }
    ]
  }
}
```

## 5. Field Selection Examples

### Basic Field Selection
```http
GET /jobs?fields=id,title,description
GET /users?fields=id,name,email
```

### Resource-Specific Fields
```http
GET /jobs?fields[job]=id,title&fields[client]=id,name
GET /reviews?fields[review]=rating,comment&fields[user]=name
```

### Relationship Field Selection
```http
GET /jobs?include=client&fields[job]=id,title&fields[client]=name
GET /users?include=reviews&fields[review]=rating
```

## 6. Implementation Guidelines

### Performance Considerations
1. Index commonly filtered fields
2. Limit maximum page size
3. Cache frequently accessed filters
4. Use database-specific optimizations

### Security Guidelines
1. Validate all filter parameters
2. Sanitize search inputs
3. Limit fields based on user permissions
4. Rate limit complex queries

### Best Practices
1. Document all available filters
2. Provide filter examples
3. Use consistent parameter naming
4. Support backward compatibility
5. Handle errors gracefully

## 7. Filter Validation Rules

### General Rules
1. Field names must be valid
2. Operators must be supported
3. Values must match field types
4. Ranges must be logical
5. Arrays must not exceed limits

### Type-Specific Rules
1. Dates must be ISO 8601 format
2. Numbers must be valid decimals
3. Strings must not exceed limits
4. Arrays must contain valid values
5. Coordinates must be valid 