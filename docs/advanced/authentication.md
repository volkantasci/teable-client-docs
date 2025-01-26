# Authentication

This guide covers authentication methods and security best practices for the Teable-Client library.

## Email/Password Authentication

### Sign In

```python
from teable import TeableClient

# Initialize client
client = TeableClient()

# Sign in with email and password
user = client.auth.signin(email="your-email@example.com", password="YourPassword123")

# Get current user info
current_user = client.auth.get_user()

# Sign out
client.auth.signout()
```

### Password Requirements
- Must contain at least one uppercase letter
- Must contain at least one number
- Must be of sufficient length (minimum 8 characters recommended)

### Environment Variables

For better security, use environment variables:

```python
import os
from teable import TeableClient

client = TeableClient()
user = client.auth.signin(
    email=os.getenv("TEABLE_EMAIL"),
    password=os.getenv("TEABLE_PASSWORD")
)
```

## API Key Authentication

### Basic Authentication

```python
from teable import TeableClient, TeableConfig

# Initialize with API key
client = TeableClient(TeableConfig(
    api_url="https://your-teable-instance.com/api",
    api_key="your-api-key"
))
```

### Environment Variables

For better security, use environment variables:

```python
import os
from teable import TeableClient, TeableConfig

# Load API key from environment
client = TeableClient(TeableConfig(
    api_url=os.getenv("TEABLE_API_URL"),
    api_key=os.getenv("TEABLE_API_KEY")
))
```

## Security Best Practices

1. **API Key Security**
   - Never hardcode API keys in your code
   - Use environment variables or secure configuration management
   - Keep your API key secret and secure
   - Use different API keys for development and production environments

2. **General Security**
   - Always use HTTPS for API communication
   - Implement proper error handling
   - Follow the principle of least privilege
   - Regularly review your security practices

## Error Handling

```python
from teable.exceptions import AuthenticationError

try:
    client = TeableClient(TeableConfig(
        api_url="https://your-teable-instance.com/api",
        api_key="invalid-key"
    ))
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
```

## Next Steps

- [Error Handling](error-handling.md)
- [Best Practices](best-practices.md)
- [API Reference](../api/client.md)
