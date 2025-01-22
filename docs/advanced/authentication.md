# Authentication

This guide covers authentication methods and security best practices for the Teable-Client library.

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

## OAuth Authentication

### OAuth Flow

1. Register your application
2. Get client credentials
3. Implement OAuth flow
4. Handle token refresh

```python
from teable import TeableClient, TeableConfig, OAuth2Config

# OAuth configuration
oauth_config = OAuth2Config(
    client_id="your-client-id",
    client_secret="your-client-secret",
    redirect_uri="your-redirect-uri"
)

# Initialize with OAuth
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    oauth=oauth_config
)
client = TeableClient(config)
```

## Token Management

### Token Refresh

```python
# Automatic token refresh
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    oauth=oauth_config,
    auto_refresh_token=True
)

# Manual token refresh
new_token = client.refresh_access_token()
```

### Token Storage

```python
class TokenStorage:
    def save_token(self, token):
        # Implement secure token storage
        pass
        
    def load_token(self):
        # Implement secure token loading
        pass

# Use custom token storage
config = TeableConfig(
    api_url="https://your-teable-instance.com/api",
    oauth=oauth_config,
    token_storage=TokenStorage()
)
```

## Security Best Practices

1. **API Key Security**
   - Never hardcode API keys
   - Use environment variables
   - Rotate keys regularly
   - Use separate keys for different environments

2. **OAuth Security**
   - Use HTTPS
   - Validate redirect URIs
   - Implement PKCE
   - Handle token expiration

3. **General Security**
   - Implement rate limiting
   - Monitor API usage
   - Log authentication events
   - Regular security audits

## Error Handling

```python
from teable.exceptions import AuthenticationError, TokenExpiredError

try:
    client = TeableClient(config)
    # Use client...
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
except TokenExpiredError as e:
    print(f"Token expired: {e}")
    # Implement token refresh
```

## Next Steps

- [Error Handling](error-handling.md)
- [Best Practices](best-practices.md)
- [API Reference](../api/client.md)
