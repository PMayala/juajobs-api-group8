# JuaJobs API Use Cases & Examples

This guide provides detailed examples for common integration scenarios with the JuaJobs API.

## 1. Job Platform Integration

### Scenario
Build a job board that displays JuaJobs listings and allows job posting.

### Implementation

#### 1.1 List Available Jobs
```python
# Python Example
from juajobs import Client

client = Client('YOUR_ACCESS_TOKEN')

# Get published jobs with filters
jobs = client.jobs.list(
    status='PUBLISHED',
    filter={
        'location': {'country': 'KE'},
        'category': 'technology'
    },
    sort='created_at',
    order='desc',
    page=1,
    per_page=20
)

# Display jobs
for job in jobs.data:
    print(f"Title: {job.title}")
    print(f"Location: {job.location.city}, {job.location.country}")
    print(f"Salary: {job.payment.formatted}")
    print("---")
```

```javascript
// JavaScript Example
const JuaJobs = require('juajobs');
const client = new JuaJobs('YOUR_ACCESS_TOKEN');

async function listJobs() {
  const jobs = await client.jobs.list({
    status: 'PUBLISHED',
    filter: {
      location: { country: 'KE' },
      category: 'technology'
    },
    sort: 'created_at',
    order: 'desc',
    page: 1,
    per_page: 20
  });

  jobs.data.forEach(job => {
    console.log(`Title: ${job.title}`);
    console.log(`Location: ${job.location.city}, ${job.location.country}`);
    console.log(`Salary: ${job.payment.formatted}`);
    console.log('---');
  });
}
```

#### 1.2 Post New Jobs
```python
# Python Example
job = client.jobs.create({
    'title': 'Senior Software Developer',
    'description': 'We are looking for an experienced developer...',
    'category_id': 'cat_123',
    'required_skills': ['skill_1', 'skill_2'],
    'location': {
        'type': 'ONSITE',
        'country': 'KE',
        'city': 'Nairobi'
    },
    'payment': {
        'type': 'FIXED',
        'amount': 1000.00,
        'currency': 'KES'
    }
})

print(f"Job created with ID: {job.id}")
```

## 2. Worker Profile Management

### Scenario
Build a talent marketplace with worker profiles and skill matching.

### Implementation

#### 2.1 Create Worker Profile
```python
# Python Example
profile = client.profiles.create(
    user_id='user_123',
    data={
        'title': 'Full Stack Developer',
        'bio': 'Experienced developer with 5+ years...',
        'skills': ['skill_1', 'skill_2'],
        'experience_years': 5,
        'education': [{
            'degree': 'Bachelor of Science',
            'field': 'Computer Science',
            'institution': 'University of Nairobi',
            'year': 2020
        }],
        'hourly_rate': {
            'amount': 50.00,
            'currency': 'USD'
        }
    }
)
```

#### 2.2 Search Workers by Skills
```python
# Python Example
workers = client.profiles.search(
    skills=['python', 'javascript'],
    location={
        'country': 'KE',
        'city': 'Nairobi'
    },
    availability='immediate',
    sort='rating',
    order='desc'
)
```

## 3. Job Application System

### Scenario
Implement a complete job application flow.

### Implementation

#### 3.1 Submit Application
```python
# Python Example
application = client.applications.create(
    job_id='job_123',
    data={
        'cover_letter': 'I am interested in this position...',
        'expected_salary': {
            'amount': 5000.00,
            'currency': 'KES'
        },
        'availability': '2024-02-01',
        'attachments': [{
            'type': 'RESUME',
            'url': 'https://example.com/resume.pdf'
        }]
    }
)
```

#### 3.2 Track Application Status
```python
# Python Example
def track_application(application_id):
    application = client.applications.get(application_id)
    
    print(f"Status: {application.status}")
    print(f"Applied: {application.created_at}")
    
    if application.interview:
        print(f"Interview scheduled: {application.interview.scheduled_at}")
    
    if application.status == 'REJECTED':
        print(f"Reason: {application.rejection_reason}")
```

## 4. Payment Integration

### Scenario
Handle secure payments for completed jobs.

### Implementation

#### 4.1 Create Payment
```python
# Python Example
payment = client.payments.create(
    job_id='job_123',
    data={
        'amount': 1000.00,
        'currency': 'KES',
        'payment_method': 'MPESA',
        'phone_number': '+254712345678'
    }
)

# Handle payment status
if payment.status == 'PENDING':
    print(f"Please complete payment using code: {payment.reference}")
```

#### 4.2 Verify Payment
```python
# Python Example
def verify_payment(payment_id):
    payment = client.payments.get(payment_id)
    
    if payment.status == 'COMPLETED':
        print("Payment successful!")
        print(f"Transaction ID: {payment.transaction_id}")
        print(f"Paid amount: {payment.amount} {payment.currency}")
    else:
        print(f"Payment status: {payment.status}")
```

## 5. Review System

### Scenario
Implement a two-way review system for jobs.

### Implementation

#### 5.1 Submit Review
```python
# Python Example
# Client reviews worker
review = client.reviews.create(
    job_id='job_123',
    reviewer_type='CLIENT',
    data={
        'rating': 5,
        'comment': 'Excellent work and communication',
        'aspects': {
            'quality': 5,
            'communication': 5,
            'timeliness': 4
        }
    }
)

# Worker reviews client
worker_review = client.reviews.create(
    job_id='job_123',
    reviewer_type='WORKER',
    data={
        'rating': 5,
        'comment': 'Great client to work with',
        'aspects': {
            'clarity': 5,
            'communication': 5,
            'payment': 5
        }
    }
)
```

#### 5.2 Get Reviews
```python
# Python Example
def get_user_reviews(user_id):
    reviews = client.reviews.list(
        user_id=user_id,
        include=['job'],
        sort='created_at',
        order='desc'
    )
    
    for review in reviews.data:
        print(f"Job: {review.job.title}")
        print(f"Rating: {review.rating}/5")
        print(f"Comment: {review.comment}")
        print("---")
```

## 6. Real-time Updates

### Scenario
Implement real-time notifications for job updates.

### Implementation

#### 6.1 WebSocket Connection
```javascript
// JavaScript Example
const ws = new WebSocket('wss://api.juajobs.com/v1/ws');

ws.onopen = () => {
  // Subscribe to updates
  ws.send(JSON.stringify({
    type: 'subscribe',
    channel: 'jobs',
    filter: {
      categories: ['technology'],
      locations: ['KE']
    }
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  switch(data.type) {
    case 'job.created':
      console.log('New job posted:', data.job);
      break;
    case 'job.updated':
      console.log('Job updated:', data.job);
      break;
    case 'application.status_changed':
      console.log('Application status changed:', data.application);
      break;
  }
};
```

## 7. Analytics Integration

### Scenario
Track job market trends and platform analytics.

### Implementation

#### 7.1 Get Market Analytics
```python
# Python Example
def get_market_analytics():
    analytics = client.analytics.get({
        'timeframe': 'last_30_days',
        'metrics': [
            'job_postings',
            'applications',
            'hired_rate',
            'average_salary'
        ],
        'dimensions': [
            'category',
            'location'
        ]
    })
    
    print("Job Market Trends:")
    print(f"Total Jobs: {analytics.metrics.job_postings}")
    print(f"Total Applications: {analytics.metrics.applications}")
    print(f"Hiring Rate: {analytics.metrics.hired_rate}%")
    print(f"Average Salary: {analytics.metrics.average_salary}")
```

## Need Help?

For more examples and detailed documentation:
- [API Documentation](https://docs.juajobs.com)
- [Code Samples Repository](https://github.com/juajobs/examples)
- [Developer Forum](https://forum.juajobs.com)
- [API Support](mailto:api@juajobs.com) 