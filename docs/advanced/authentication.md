# Authentication

The Teable Client Library provides comprehensive authentication and user management functionality through the `AuthManager` class.

## Authentication Methods

### 1. API Key Authentication

Required for all API access:

```python
from teable import TeableClient, TeableConfig

# Initialize with API key
client = TeableClient(TeableConfig(
    api_key="your_api_key_here",
    api_url="https://api.teable.io"
))
```

### 2. User Authentication

Required for advanced operations like space management:

```python
# Sign in with email/password
user = client.auth.signin(
    email="your-email@example.com",
    password="your-password"
)

# Get current user info
current_user = client.auth.get_user()
print(f"Signed in as: {current_user.email}")

# Get detailed user info
detailed_info = client.auth.get_user_info()

# Sign out when done
client.auth.signout()
```

## User Management

### Profile Management

```python
# Update user name
client.auth.update_user_name("New Name")

# Update avatar
with open("avatar.jpg", "rb") as f:
    avatar_data = f.read()
client.auth.update_user_avatar(
    avatar_data,
    mime_type="image/jpeg"
)

# Update notification settings
client.auth.update_user_notify_meta(email=True)
```

### Email Management

```python
# Send email change verification code
result = client.auth.send_change_email_code(
    email="new-email@example.com",
    password="current-password"
)
token = result["token"]

# Change email with verification
client.auth.change_email(
    email="new-email@example.com",
    token=token,
    code="123456"  # Code received in email
)
```

### Password Management

```python
# Add password (for users without password)
client.auth.add_password("NewPassword123")

# Change password
client.auth.change_password(
    password="CurrentPassword123",
    new_password="NewPassword123"
)

# Reset password flow
client.auth.send_reset_password_email("user@example.com")
client.auth.reset_password(
    password="NewPassword123",
    code="123456"  # Code received in email
)
```

### Account Creation

```python
# Send signup verification code
result = client.auth.send_signup_verification_code("new-user@example.com")
verification = {
    "token": result["token"],
    "code": "123456"  # Code received in email
}

# Sign up new user
user = client.auth.signup(
    email="new-user@example.com",
    password="Password123",
    default_space_name="My Space",  # Optional
    verification=verification,
    ref_meta={  # Optional
        "query": "source=website",
        "referer": "https://example.com"
    }
)
```

## Validation Rules

### Email Validation

Emails must:
- Be a valid string
- Match standard email format (user@domain.tld)
- Pass server-side validation

```python
# Example of email validation
try:
    client.auth.signin(
        email="invalid-email",  # Will raise ValidationError
        password="ValidPass123"
    )
except ValidationError as e:
    print(f"Invalid email: {str(e)}")
```

### Password Validation

Passwords must:
- Be a string
- Be at least 8 characters long
- Contain at least one uppercase letter
- Contain at least one number
- Not contain invalid characters

```python
# Example of password validation
try:
    client.auth.signin(
        email="user@example.com",
        password="short"  # Will raise ValidationError
    )
except ValidationError as e:
    print(f"Invalid password: {str(e)}")
```

## Error Handling

Handle authentication errors appropriately:

```python
from teable.exceptions import ValidationError, APIError

try:
    client.auth.signin(
        email="user@example.com",
        password="wrong-password"
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    if e.status_code == 401:
        print("Invalid credentials")
    elif e.status_code == 403:
        print("Account locked")
    else:
        print(f"API error: {str(e)}")
```

## Best Practices

### 1. Secure Credential Management

```python
import os
from dotenv import load_dotenv

# Load credentials from environment
load_dotenv()

client = TeableClient(TeableConfig(
    api_key=os.getenv('TEABLE_API_KEY')
))

client.auth.signin(
    email=os.getenv('TEABLE_EMAIL'),
    password=os.getenv('TEABLE_PASSWORD')
)
```

### 2. Session Management

```python
def get_authenticated_client():
    client = TeableClient(TeableConfig(
        api_key=os.getenv('TEABLE_API_KEY')
    ))
    
    try:
        client.auth.signin(
            email=os.getenv('TEABLE_EMAIL'),
            password=os.getenv('TEABLE_PASSWORD')
        )
        return client
    except Exception as e:
        print(f"Authentication failed: {str(e)}")
        raise

def cleanup_client(client):
    try:
        client.auth.signout()
    except Exception as e:
        print(f"Signout failed: {str(e)}")
```

### 3. Error Recovery

```python
import time
from teable.exceptions import APIError

def retry_auth(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                    continue
            raise
    raise Exception("Max retries exceeded")

# Usage
client = retry_auth(get_authenticated_client)
```

## Operation Requirements

Here's a reference for which operations require which level of authentication:

### API Key Only
- Basic table operations
- Record reading
- Field validation
- View access

### User Authentication Required
- Space management
- Base operations
- User profile updates
- Permission changes
- Invitation management
- Advanced operations

## Security Considerations

1. **API Key Security**:
   - Never commit API keys to version control
   - Use environment variables or secure vaults
   - Rotate keys periodically
   - Use different keys for development/production

2. **Password Security**:
   - Implement proper password policies
   - Handle password resets securely
   - Use secure password storage
   - Implement rate limiting for auth attempts

3. **Session Management**:
   - Sign out when done
   - Implement session timeouts
   - Handle token expiration
   - Secure token storage

4. **Error Handling**:
   - Don't expose sensitive info in errors
   - Log authentication failures
   - Implement proper error recovery
   - Use secure error messages
