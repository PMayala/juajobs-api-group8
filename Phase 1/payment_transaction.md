# Payment Transaction Resource

## Overview
The Payment Transaction resource manages all financial transactions within the platform, including job payments, service fees, and withdrawals. It ensures secure and traceable payment processing while handling multiple currencies and payment methods.

## Resource Attributes

### Base Transaction Attributes
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| id | UUID | Yes (System) | Unique identifier | System-generated |
| transaction_type | Enum | Yes | Type of transaction | Valid transaction type |
| status | Enum | Yes | Transaction status | Valid status enum |
| created_at | DateTime | Yes (System) | Transaction creation time | System-generated |
| updated_at | DateTime | Yes (System) | Last update time | System-generated |
| reference_id | String | Yes | External reference ID | Valid reference format |

### Payment Details
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| amount | Decimal | Yes | Transaction amount | > 0 |
| currency | String | Yes | Transaction currency | Valid currency code |
| exchange_rate | Decimal | No | Applied exchange rate | > 0 if provided |
| fees | Object | Yes | Platform fees breakdown | Valid fees object |
| payment_method | Object | Yes | Payment method details | Valid payment method |
| description | String | Yes | Transaction description | 5-200 characters |

### Transaction Parties
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| sender_id | UUID | Yes | Paying user/entity | Valid user/entity ID |
| receiver_id | UUID | Yes | Receiving user/entity | Valid user/entity ID |
| job_id | UUID | No | Associated job | Valid job ID if provided |
| escrow_id | UUID | No | Associated escrow | Valid escrow ID |
| platform_account_id | UUID | Yes | Platform account | Valid account ID |

### Payment Processing
| Attribute | Type | Required | Description | Validation Rules |
|-----------|------|----------|-------------|------------------|
| processing_details | Object | Yes | Payment processor data | Valid processing object |
| verification_status | Enum | Yes | Payment verification | Valid verification status |
| settlement_status | Enum | Yes | Settlement status | Valid settlement status |
| settlement_date | DateTime | No | Expected settlement | Future date if provided |
| failure_reason | String | No | Error description | Required if failed |

## Computed Attributes
| Attribute | Description | Computation Logic |
|-----------|-------------|-------------------|
| total_amount | Total with fees | Base amount + all fees |
| net_amount | Amount after fees | Base amount - platform fees |
| processing_time | Transaction duration | Time to completion/failure |
| risk_score | Transaction risk level | Based on various risk factors |

## Resource States
- INITIATED
- PROCESSING
- PENDING_VERIFICATION
- VERIFIED
- COMPLETED
- FAILED
- REVERSED
- REFUNDED
- DISPUTED

## Access Control
- Users can view their own transactions
- Admins can view and manage all transactions
- Payment processors have limited access
- Sensitive data is encrypted
- Audit logs track all changes

## Validation Rules
1. Transaction amount must be within allowed limits
2. Currency must be supported by platform
3. Sender must have sufficient funds
4. Payment method must be verified
5. Compliance checks must pass
6. Rate limits must be respected

## Transaction Types
- JOB_PAYMENT
- ESCROW_DEPOSIT
- ESCROW_RELEASE
- PLATFORM_FEE
- WITHDRAWAL
- REFUND
- DISPUTE_RESOLUTION
- SERVICE_FEE
- TAX_PAYMENT

## Related Resources
- User (Sender)
- User (Receiver)
- Job Posting
- Escrow Account
- Payment Method
- Bank Account
- Dispute
- Platform Account
- Tax Record
- Compliance Record 