# JuaJobs Resource Relationships

## Overview
This document outlines the relationships and dependencies between resources in the JuaJobs platform. Understanding these relationships is crucial for maintaining data integrity and implementing business logic.

## Core Resource Relationships

### User-Centric Relationships
```
User
├── Worker Profile (1:1)
│   ├── Skills (M:N)
│   ├── Categories (M:N)
│   └── Reviews (1:N)
├── Job Postings (1:N) [as Client]
├── Applications (1:N) [as Worker]
├── Reviews Given (1:N)
├── Reviews Received (1:N)
└── Payment Transactions (1:N)
```

### Job-Centric Relationships
```
Job Posting
├── Client (N:1)
├── Applications (1:N)
├── Required Skills (M:N)
├── Categories (M:N)
├── Reviews (1:N)
└── Payment Transactions (1:N)
```

### Application-Centric Relationships
```
Application
├── Job Posting (N:1)
├── Worker (N:1)
├── Interview Records (1:N)
├── Messages (1:N)
└── Payment Transaction (1:1)
```

### Review-Centric Relationships
```
Review
├── Job Posting (N:1)
├── Reviewer (N:1)
├── Reviewee (N:1)
└── Skills Demonstrated (M:N)
```

### Payment-Centric Relationships
```
Payment Transaction
├── Sender (N:1)
├── Receiver (N:1)
├── Job Posting (N:1)
├── Application (1:1)
└── Escrow Account (N:1)
```

## Resource Cardinality Matrix

| Resource A | Relationship | Resource B | Notes |
|------------|--------------|------------|--------|
| User | 1:1 | Worker Profile | When user is worker |
| User | 1:N | Job Postings | When user is client |
| User | 1:N | Applications | When user is worker |
| User | 1:N | Reviews Given | As reviewer |
| User | 1:N | Reviews Received | As reviewee |
| Job Posting | N:1 | User | Posted by client |
| Job Posting | 1:N | Applications | From workers |
| Job Posting | M:N | Skills | Required skills |
| Job Posting | M:N | Categories | Job categories |
| Application | N:1 | Job Posting | Applied to job |
| Application | N:1 | User | From worker |
| Review | N:1 | Job Posting | For completed job |
| Review | N:1 | User | Both parties |
| Payment | N:1 | User | Both parties |
| Payment | N:1 | Job Posting | Job payment |
| Skill | M:N | Category | Skill classification |
| Skill | M:N | Worker Profile | Worker skills |

## Dependency Rules

1. **Cascading Deletes**
   - Deleting a User cascades to Worker Profile
   - Deleting a Job Posting cascades to Applications
   - Deleting a User preserves Reviews for platform integrity

2. **Required References**
   - Applications require valid Job Posting and Worker
   - Reviews require valid Job Posting and both Users
   - Payments require valid Sender and Receiver

3. **Soft Deletes**
   - Users are soft deleted to preserve history
   - Job Postings are soft deleted to maintain records
   - Reviews are never deleted, only hidden

4. **Integrity Constraints**
   - Worker can't apply to own Job Posting
   - User can't review themselves
   - Payment must reference valid transaction parties

## State Dependencies

1. **Job Posting States**
   - DRAFT → PUBLISHED → CLOSED → COMPLETED
   - Can't accept applications when CLOSED
   - Must be COMPLETED for reviews

2. **Application States**
   - DRAFT → SUBMITTED → ACCEPTED/REJECTED
   - Must be ACCEPTED for payment processing
   - Affects Worker Profile metrics

3. **Payment States**
   - INITIATED → PROCESSING → COMPLETED
   - Must complete for job completion
   - Affects platform analytics

## Implementation Considerations

1. **Performance**
   - Index frequently queried relationships
   - Cache common relationship lookups
   - Optimize M:N join tables

2. **Scalability**
   - Shard data by geographical region
   - Partition large tables
   - Consider eventual consistency

3. **Security**
   - Implement row-level security
   - Validate all relationships
   - Audit relationship changes