# JuaJobs API Payment Integration Design

## 1. Payment Methods Support

### Mobile Money Integration
```json
{
  "payment_methods": {
    "mpesa": {
      "countries": ["KE"],
      "currencies": ["KES"],
      "type": "mobile_money",
      "provider": "Safaricom",
      "flow": "stk_push",
      "validation_regex": "^254[17]\\d{8}$"
    },
    "airtel_money": {
      "countries": ["KE", "UG", "TZ"],
      "currencies": ["KES", "UGX", "TZS"],
      "type": "mobile_money",
      "provider": "Airtel",
      "flow": "ussd_push",
      "validation_regex": "^(?:254|255|256)\\d{9}$"
    },
    "orange_money": {
      "countries": ["CI", "SN", "CM"],
      "currencies": ["XOF", "XAF"],
      "type": "mobile_money",
      "provider": "Orange",
      "flow": "direct_charge",
      "validation_regex": "^(?:225|221|237)\\d{8}$"
    }
  }
}
```

### Bank Transfer Support
```json
{
  "bank_transfers": {
    "pesalink": {
      "countries": ["KE"],
      "currencies": ["KES"],
      "type": "instant_transfer",
      "provider": "Kenya Bankers Association",
      "required_fields": [
        "account_number",
        "bank_code"
      ]
    },
    "nibss": {
      "countries": ["NG"],
      "currencies": ["NGN"],
      "type": "instant_transfer",
      "provider": "NIBSS",
      "required_fields": [
        "account_number",
        "bank_code"
      ]
    }
  }
}
```

### Card Payments
```json
{
  "card_payments": {
    "supported_networks": [
      "Visa",
      "Mastercard",
      "Verve"
    ],
    "processors": {
      "flutterwave": {
        "countries": ["NG", "KE", "ZA"],
        "currencies": ["NGN", "KES", "ZAR"]
      },
      "paystack": {
        "countries": ["NG", "GH"],
        "currencies": ["NGN", "GHS"]
      }
    },
    "security": {
      "3d_secure": true,
      "tokenization": true,
      "cvv_required": true
    }
  }
}
```

## 2. Payment Flow Design

### Payment Creation
```http
POST /v1/payments
Content-Type: application/json

{
  "amount": 1000.00,
  "currency": "KES",
  "payment_method": {
    "type": "mobile_money",
    "provider": "mpesa",
    "phone_number": "+254712345678"
  },
  "job_id": "job_123",
  "description": "Payment for Web Development Project"
}
```

### Payment Response
```json
{
  "id": "pay_123",
  "status": "PENDING",
  "amount": 1000.00,
  "currency": "KES",
  "payment_method": {
    "type": "mobile_money",
    "provider": "mpesa",
    "phone_number": "+254712345678",
    "checkout_id": "ws_CO_123"
  },
  "created_at": "2024-01-20T12:00:00Z",
  "expires_at": "2024-01-20T12:15:00Z"
}
```

### Payment Status Updates
```http
GET /v1/payments/pay_123

{
  "id": "pay_123",
  "status": "COMPLETED",
  "transaction_id": "MPESA123456",
  "paid_at": "2024-01-20T12:05:00Z",
  "receipt_number": "RCP123456"
}
```

## 3. Transaction State Management

### Transaction States
```json
{
  "transaction_states": {
    "INITIATED": {
      "description": "Payment request created",
      "allowed_transitions": ["PENDING", "FAILED"]
    },
    "PENDING": {
      "description": "Awaiting user action",
      "allowed_transitions": ["PROCESSING", "FAILED", "EXPIRED"]
    },
    "PROCESSING": {
      "description": "Payment being processed",
      "allowed_transitions": ["COMPLETED", "FAILED"]
    },
    "COMPLETED": {
      "description": "Payment successful",
      "allowed_transitions": ["REFUNDED"]
    },
    "FAILED": {
      "description": "Payment failed",
      "allowed_transitions": ["INITIATED"]
    },
    "EXPIRED": {
      "description": "Payment request expired",
      "allowed_transitions": ["INITIATED"]
    },
    "REFUNDED": {
      "description": "Payment refunded",
      "allowed_transitions": []
    }
  }
}
```

### State Transitions
```python
class TransactionStateManager:
    def transition_state(self, payment_id, new_state):
        payment = self.get_payment(payment_id)
        current_state = payment.status
        
        if new_state not in self.allowed_transitions[current_state]:
            raise InvalidTransitionError(
                f"Cannot transition from {current_state} to {new_state}"
            )
            
        payment.status = new_state
        payment.status_updated_at = datetime.utcnow()
        
        self.notify_status_change(payment)
        return payment
```

## 4. Security Implementation

### Payment Encryption
```python
class PaymentEncryption:
    def encrypt_payment_data(self, payment_data):
        # Use AES-256-GCM for encryption
        encryption_key = self.get_encryption_key()
        nonce = os.urandom(12)
        
        cipher = AES.new(encryption_key, AES.MODE_GCM, nonce=nonce)
        ciphertext, tag = cipher.encrypt_and_digest(
            json.dumps(payment_data).encode()
        )
        
        return {
            "encrypted_data": base64.b64encode(ciphertext),
            "nonce": base64.b64encode(nonce),
            "tag": base64.b64encode(tag)
        }
```

### Webhook Verification
```python
class WebhookVerifier:
    def verify_signature(self, payload, signature, timestamp):
        # Compute HMAC
        message = f"{timestamp}.{payload}"
        expected_signature = hmac.new(
            self.webhook_secret,
            message.encode(),
            hashlib.sha256
        ).hexdigest()
        
        return hmac.compare_digest(
            signature,
            expected_signature
        )
```

## 5. Error Handling

### Payment Errors
```json
{
  "error_codes": {
    "INSUFFICIENT_FUNDS": {
      "code": "P001",
      "message": "Insufficient funds in the account",
      "recovery_suggestion": "Please ensure sufficient funds and try again"
    },
    "INVALID_ACCOUNT": {
      "code": "P002",
      "message": "Invalid account or phone number",
      "recovery_suggestion": "Verify the account details and try again"
    },
    "PROVIDER_DOWN": {
      "code": "P003",
      "message": "Payment provider temporarily unavailable",
      "recovery_suggestion": "Please try again in a few minutes"
    }
  }
}
```

### Error Response
```json
{
  "error": {
    "code": "P001",
    "message": "Insufficient funds in the account",
    "transaction_id": "pay_123",
    "provider_error_code": "M0001",
    "recovery_suggestion": "Please ensure sufficient funds and try again",
    "support_reference": "ERR_123456"
  }
}
```

## 6. Integration Guidelines

### Mobile Money Integration
```python
class MobileMoneyProcessor:
    def process_payment(self, payment):
        # Initialize provider client
        client = self.get_provider_client(payment.provider)
        
        # Format phone number
        phone = self.format_phone_number(
            payment.phone_number,
            payment.country
        )
        
        # Create payment request
        response = client.create_payment({
            "amount": payment.amount,
            "currency": payment.currency,
            "phone_number": phone,
            "reference": payment.id,
            "callback_url": self.get_callback_url(payment)
        })
        
        return self.handle_response(response)
```

### Bank Transfer Integration
```python
class BankTransferProcessor:
    def initiate_transfer(self, payment):
        # Validate bank details
        self.validate_bank_details(
            payment.bank_code,
            payment.account_number
        )
        
        # Create transfer request
        response = self.bank_client.create_transfer({
            "amount": payment.amount,
            "currency": payment.currency,
            "bank_code": payment.bank_code,
            "account_number": payment.account_number,
            "reference": payment.id
        })
        
        return self.handle_response(response)
```

## 7. Testing & Simulation

### Test Credentials
```json
{
  "test_accounts": {
    "mpesa": {
      "phone_numbers": [
        "+254708374149",
        "+254708374150"
      ],
      "test_amounts": {
        "success": 100,
        "insufficient_funds": 999999,
        "timeout": 110
      }
    }
  }
}
```

### Payment Simulation
```http
POST /v1/payments/simulate
Content-Type: application/json

{
  "payment_id": "pay_123",
  "simulation_type": "success",
  "delay": 5,
  "response": {
    "transaction_id": "MPESA123456",
    "status": "COMPLETED"
  }
}
``` 