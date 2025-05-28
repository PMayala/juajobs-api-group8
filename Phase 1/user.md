# User Resource

## Overview
The User resource is the foundation of the JuaJobs platform, representing all actors within the system including workers, clients, and administrators.

## Resource Attributes

### Base User Attributes
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| id | UUID | Yes (System) | Unique identifier | System-generated |
| email | String | Yes | User's email address | Valid email format |
| phone_number | String | Yes | User's phone number | Valid African phone number format |
| password_hash | String | Yes (System) | Hashed password | System-managed |
| first_name | String | Yes | User's first name | 2-50 characters |
| last_name | String | Yes | User's last name | 2-50 characters |
| role | Enum | Yes | User type (WORKER, CLIENT, ADMIN) | Valid role enum |
| status | Enum | Yes | Account status (ACTIVE, INACTIVE, SUSPENDED) | Valid status enum |
| created_at | DateTime | Yes (System) | Account creation timestamp | System-generated |
| updated_at | DateTime | Yes (System) | Last update timestamp | System-generated |
| profile_image | String | No | URL to profile image | Valid URL format |
| language_preferences | Array[String] | No | Preferred languages | ISO language codes |
| notification_settings | Object | Yes | Notification preferences | Valid settings object |

### Location Information
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| country | String | Yes | User's country | Valid African country code |
| city | String | Yes | User's city | Non-empty string |
| address | String | No | Detailed address | Max 200 characters |
| coordinates | Object | No | GPS coordinates | Valid lat/long object |

### Verification Attributes
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| email_verified | Boolean | Yes | Email verification status | Default: false |
| phone_verified | Boolean | Yes | Phone verification status | Default: false |
| id_verification_status | Enum | Yes | ID verification status | Valid verification status |
| verification_documents | Array[Object] | No | Verification document details | Valid document objects |

## Computed Attributes
| Attribute | Description | Computation Logic |
|-----------|-------------|-------------------|
| full_name | User's full name | Concatenation of first_name and last_name |
| rating | Average user rating | Calculated from associated reviews |
| completion_rate | Job completion rate | For workers: Completed jobs / Total jobs |
| response_rate | Response rate to inquiries | Responses within 24h / Total inquiries |

## Resource States
- PENDING_VERIFICATION
- ACTIVE
- SUSPENDED
- INACTIVE
- BANNED

## Access Control
- Users can view and edit their own profiles
- Admins can view and manage all user profiles
- Public profiles show limited information
- Sensitive data (password_hash, verification_documents) are never exposed via API

## Validation Rules
1. Email must be unique across the system
2. Phone number must be unique across the system
3. Password must meet minimum security requirements
4. Country must be a valid African country code
5. Profile image must be under 5MB
6. Verification documents must be valid image or PDF files

## Related Resources
- Worker Profile (for users with WORKER role)
- Job Postings (for users with CLIENT role)
- Reviews (as reviewer or reviewee)
- Applications (as applicant or job poster)
- Payment Information
- Messages 