# Authentication

The Teable Client Library supports two levels of authentication:

1. API Key Authentication
2. User Authentication (Email/Password)

Understanding these authentication methods and when to use them is crucial for working with the Teable API effectively.

## Authentication Levels

### 1. API Key Authentication

API key authentication provides basic access to the Teable API. This level of authentication is suitable for:

- Reading public data
- Basic table operations
- Record operations within accessible tables

```python
from teable import TeableClient, TeableConfig

# Initialize with API key
client = TeableClient(TeableConfig(
    api_key="your_api_key_here",
    api_url="https://api.teable.io"
))
```

### 2. User Authentication

User authentication provides full access to all API features. This level is required for:

- Space management
- Base creation/deletion
- User management
- Permission changes
- Advanced operations

```python
# First initialize with API key
client = TeableClient(TeableConfig(
    api_key="your_api_key_here"
))

# Then sign in with user credentials
user = client.auth.signin(
    email="your-email@example.com",
    password="your-password"
)

# Get current user info
current_user = client.auth.get_user()
print(f"Signed in as: {current_user.email}")

# Sign out when done
client.auth.signout()
```

## Configuration Methods

### Environment Variables

You can use environment variables for configuration:

```bash
# .env file
TEABLE_API_KEY=your_api_key_here
TEABLE_API_URL=https://api.teable.io
TEABLE_EMAIL=your-email@example.com
TEABLE_PASSWORD=your-password
```

```python
from teable import TeableClient
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Initialize client from environment
client = TeableClient.from_env()
```

### Direct Configuration

You can also configure the client directly in code:

```python
from teable import TeableClient, TeableConfig

config = TeableConfig(
    api_key="your_api_key_here",
    api_url="https://api.teable.io"  # Optional
)
client = TeableClient(config)
```

## Password Requirements

When signing in, passwords must meet these requirements:

- Minimum length: 8 characters
- Must contain at least one uppercase letter
- Must contain at least one number
- Must not contain invalid characters

```python
# Example of password validation
try:
    client.auth.signin(
        email="test@example.com",
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
        email="invalid-email",  # Invalid email format
        password="ValidPass123"
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
```

## Best Practices

1. **API Key Security**:
   - Never commit API keys to version control
   - Use environment variables or secure configuration management
   - Rotate API keys periodically

2. **User Authentication**:
   - Sign in only when needed
   - Sign out when operations are complete
   - Handle authentication errors gracefully
   - Implement proper password security

3. **Error Recovery**:
   - Implement retry logic for temporary failures
   - Handle token expiration appropriately
   - Log authentication failures for monitoring

4. **Configuration Management**:
   - Use environment variables in production
   - Keep credentials secure
   - Implement proper secret management

## Operation Requirements

Here's a quick reference for which operations require which level of authentication:

### API Key Only
- Read public records
- Basic table queries
- Field validation
- View access

### User Authentication Required
- Space creation/deletion
- Base management
- Permission changes
- User invitations
- Advanced operations

## Example Workflow

Here's a complete example of proper authentication usage:

```python
from teable import TeableClient, TeableConfig
from teable.exceptions import ValidationError, APIError

def setup_teable_client():
    try:
        # Initialize with API key
        client = TeableClient(TeableConfig(
            api_key="your_api_key_here"
        ))
        
        # Sign in for full access
        client.auth.signin(
            email="your-email@example.com",
            password="your-password"
        )
        
        return client
    
    except ValidationError as e:
        print(f"Configuration error: {str(e)}")
        raise
    except APIError as e:
        print(f"API error: {str(e)}")
        raise
    except Exception as e:
        print(f"Unexpected error: {str(e)}")
        raise

def main():
    try:
        # Set up client
        client = setup_teable_client()
        
        # Perform operations requiring authentication
        space = client.spaces.create_space(name="New Project")
        
        # Clean up
        client.auth.signout()
        
    except Exception as e:
        print(f"Error: {str(e)}")
        # Implement proper error recovery

if __name__ == "__main__":
    main()
```

## Troubleshooting

### Common Issues

1. **Invalid API Key**:
   - Verify the API key is correct
   - Check if the API key has the necessary permissions
   - Ensure the API key is active

2. **Authentication Failures**:
   - Verify email and password are correct
   - Check if the account is active
   - Ensure the API URL is correct

3. **Permission Errors**:
   - Verify the user has necessary permissions
   - Check if authentication was successful
   - Ensure operations match authentication level

### Getting Help

If you encounter authentication issues:

1. Check the error message for specific details
2. Verify your credentials and configuration
3. Review the [error handling documentation](error-handling.md)
4. Contact support with specific error details
