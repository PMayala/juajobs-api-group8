# JuaJobs API Authorization Framework

## 1. Role-Based Access Control (RBAC)

### Core Roles
1. **Guest**
   - Public access
   - Read-only operations
   - Limited resource access

2. **Worker**
   - Create and manage profile
   - Apply to jobs
   - Submit reviews
   - Manage work history

3. **Client**
   - Post and manage jobs
   - Review applications
   - Manage payments
   - Submit reviews

4. **Admin**
   - Full system access
   - User management
   - System configuration
   - Analytics access

### Role Hierarchy
```
Admin
  ├─ System Management
  ├─ User Management
  └─ Analytics

Client
  ├─ Job Management
  ├─ Application Management
  └─ Payment Management

Worker
  ├─ Profile Management
  ├─ Application Management
  └─ Review Management

Guest
  └─ Public Access
```

## 2. Permission Matrix

### Resource Permissions

#### Jobs
| Operation | Guest | Worker | Client | Admin |
|-----------|-------|---------|---------|--------|
| List Jobs | ✓ | ✓ | ✓ | ✓ |
| View Job | ✓ | ✓ | ✓ | ✓ |
| Create Job | ✗ | ✗ | ✓ | ✓ |
| Update Job | ✗ | ✗ | Owner | ✓ |
| Delete Job | ✗ | ✗ | Owner | ✓ |
| Close Job | ✗ | ✗ | Owner | ✓ |

#### Applications
| Operation | Guest | Worker | Client | Admin |
|-----------|-------|---------|---------|--------|
| List Applications | ✗ | Own | Own Jobs | ✓ |
| View Application | ✗ | Own | Own Jobs | ✓ |
| Create Application | ✗ | ✓ | ✗ | ✓ |
| Update Application | ✗ | Own | ✗ | ✓ |
| Withdraw Application | ✗ | Own | ✗ | ✓ |

#### Reviews
| Operation | Guest | Worker | Client | Admin |
|-----------|-------|---------|---------|--------|
| List Reviews | ✓ | ✓ | ✓ | ✓ |
| View Review | ✓ | ✓ | ✓ | ✓ |
| Create Review | ✗ | Job Participant | Job Owner | ✓ |
| Update Review | ✗ | Own | Own | ✓ |
| Delete Review | ✗ | ✗ | ✗ | ✓ |

#### Payments
| Operation | Guest | Worker | Client | Admin |
|-----------|-------|---------|---------|--------|
| List Payments | ✗ | Own | Own | ✓ |
| View Payment | ✗ | Own | Own | ✓ |
| Create Payment | ✗ | ✗ | ✓ | ✓ |
| Process Refund | ✗ | ✗ | ✗ | ✓ |

## 3. Resource Ownership Model

### Direct Ownership
Resources directly owned by a user:
- User Profile
- Job Postings (Client)
- Applications (Worker)
- Reviews
- Payments

```json
{
  "resource_type": "job_posting",
  "resource_id": "job_123",
  "owner_id": "user_456",
  "owner_type": "CLIENT",
  "created_at": "2024-01-20T12:00:00Z"
}
```

### Delegated Access
Resources accessible through relationships:
- Job Applications (Client access through job ownership)
- Worker Profiles (Client access through application)
- Payment Records (Worker access through job participation)

```json
{
  "resource_type": "application",
  "resource_id": "app_123",
  "owner_id": "user_789",
  "owner_type": "WORKER",
  "delegated_access": [{
    "user_id": "user_456",
    "access_type": "VIEW",
    "granted_through": "job_123"
  }]
}
```

## 4. Permission Implementation

### Permission Checks
```python
class PermissionChecker:
    def check_permission(self, user, resource, action):
        # Check user role
        if not self.has_role_permission(user.role, action):
            return False
            
        # Check resource ownership
        if self.requires_ownership(action):
            if not self.is_resource_owner(user, resource):
                return False
                
        # Check delegated access
        if self.has_delegated_access(user, resource, action):
            return True
            
        return False
        
    def has_role_permission(self, role, action):
        return PERMISSION_MATRIX[role][action]
        
    def is_resource_owner(self, user, resource):
        return resource.owner_id == user.id
        
    def has_delegated_access(self, user, resource, action):
        delegated = resource.delegated_access
        return any(
            access.user_id == user.id and
            access.access_type == action
            for access in delegated
        )
```

### API Implementation
```python
@require_permission('job:create')
def create_job(request):
    # Implementation

@require_permission('application:view')
def view_application(request, application_id):
    # Implementation

@require_permission('payment:process')
def process_payment(request, payment_id):
    # Implementation
```

## 5. Delegated Permissions

### Delegation Types
1. **Time-Limited Access**
   ```json
   {
     "delegated_to": "user_123",
     "resource_type": "job_posting",
     "resource_id": "job_456",
     "permissions": ["view", "update"],
     "expires_at": "2024-02-20T12:00:00Z"
   }
   ```

2. **Role-Based Delegation**
   ```json
   {
     "delegated_to_role": "MANAGER",
     "resource_type": "department_jobs",
     "department_id": "dept_123",
     "permissions": ["view", "update", "close"]
   }
   ```

3. **Conditional Access**
   ```json
   {
     "delegated_to": "user_123",
     "resource_type": "applications",
     "conditions": {
       "job_id": "job_456",
       "status": ["PENDING", "INTERVIEWING"]
     },
     "permissions": ["view", "update_status"]
   }
   ```

### Delegation Management
```python
class DelegationManager:
    def delegate_access(self, grantor, grantee, resource, permissions, expiry=None):
        if not self.can_delegate(grantor, resource):
            raise PermissionError("Cannot delegate access")
            
        delegation = Delegation(
            grantor_id=grantor.id,
            grantee_id=grantee.id,
            resource_id=resource.id,
            permissions=permissions,
            expires_at=expiry
        )
        
        return self.store_delegation(delegation)
        
    def revoke_access(self, grantor, delegation_id):
        delegation = self.get_delegation(delegation_id)
        if delegation.grantor_id != grantor.id:
            raise PermissionError("Cannot revoke delegation")
            
        self.remove_delegation(delegation_id)
```

## 6. Audit & Compliance

### Permission Auditing
```python
class PermissionAuditor:
    def log_access(self, user, resource, action, success):
        audit_log = {
            "timestamp": datetime.utcnow(),
            "user_id": user.id,
            "resource_type": resource.type,
            "resource_id": resource.id,
            "action": action,
            "success": success,
            "client_ip": request.remote_addr,
            "user_agent": request.user_agent
        }
        self.store_audit_log(audit_log)
```

### Access Reviews
1. Regular review of delegated permissions
2. Automatic expiration of unused delegations
3. Periodic validation of role assignments
4. Audit trail of permission changes

## 7. Security Considerations

### Principle of Least Privilege
1. Grant minimal required permissions
2. Regular permission reviews
3. Automatic permission expiry
4. Clear delegation chains

### Separation of Duties
1. Prevent privilege escalation
2. Enforce role boundaries
3. Maintain audit trails
4. Regular compliance checks

### Security Monitoring
1. Track unusual access patterns
2. Monitor permission changes
3. Alert on suspicious activity
4. Regular security reviews 