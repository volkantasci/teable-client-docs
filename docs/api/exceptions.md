# API Reference: Exceptions

This document provides detailed API reference for all exception classes in the Teable-Client library.

## Exception Hierarchy

```
TeableError
├── ValidationError
├── AuthenticationError
├── ResourceNotFoundError
├── RateLimitError
├── NetworkError
└── ConfigurationError
```

## Base Exception

### TeableError

Base exception class for all Teable-specific errors.

```python
from teable.exceptions import TeableError

try:
    # Perform operation
    pass
except TeableError as e:
    print(f"Operation failed: {e}")
    print(f"Error code: {e.code}")
    print(f"Error details: {e.details}")
```

Properties:
- message (str): Human-readable error message
- code (str): Error code identifier
- details (Optional[Dict]): Additional error details

## Validation Errors

### ValidationError

Raised when data validation fails.

```python
from teable.exceptions import ValidationError

try:
    table.create_record({
        "Name": "John Doe",
        "Age": "invalid"  # Should be a number
    })
except ValidationError as e:
    print(f"Validation failed: {e}")
    print(f"Field: {e.field}")
    print(f"Value: {e.value}")
    print(f"Constraint: {e.constraint}")
```

Properties:
- field (str): Name of the field that failed validation
- value (Any): Invalid value that caused the error
- constraint (str): Description of the validation constraint that failed

## Authentication Errors

### AuthenticationError

Raised when authentication fails.

```python
from teable.exceptions import AuthenticationError

try:
    client = TeableClient(TeableConfig(
        api_url="https://example.com",
        api_key="invalid-key"
    ))
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
    print(f"Reason: {e.reason}")
```

Properties:
- reason (str): Specific reason for authentication failure
- is_expired (bool): Whether the error is due to expired credentials

### TokenExpiredError

Raised when an OAuth token has expired.

```python
from teable.exceptions import TokenExpiredError

try:
    # Perform authenticated operation
    pass
except TokenExpiredError as e:
    print(f"Token expired: {e}")
    print(f"Expiry time: {e.expiry_time}")
    # Implement token refresh
```

Properties:
- expiry_time (datetime): When the token expired
- can_refresh (bool): Whether the token can be refreshed

## Resource Errors

### ResourceNotFoundError

Raised when a requested resource doesn't exist.

```python
from teable.exceptions import ResourceNotFoundError

try:
    table = client.get_table("non-existent-id")
except ResourceNotFoundError as e:
    print(f"Resource not found: {e}")
    print(f"Resource type: {e.resource_type}")
    print(f"Resource ID: {e.resource_id}")
```

Properties:
- resource_type (str): Type of resource that wasn't found
- resource_id (str): ID of the resource that wasn't found

## Rate Limiting

### RateLimitError

Raised when API rate limits are exceeded.

```python
from teable.exceptions import RateLimitError

try:
    # Perform many operations
    pass
except RateLimitError as e:
    print(f"Rate limit exceeded: {e}")
    print(f"Reset time: {e.reset_time}")
    print(f"Retry after: {e.retry_after} seconds")
```

Properties:
- reset_time (datetime): When the rate limit will reset
- retry_after (int): Seconds until next retry is allowed
- limit (int): Current rate limit
- remaining (int): Remaining requests allowed

## Network Errors

### NetworkError

Raised for network-related issues.

```python
from teable.exceptions import NetworkError

try:
    # Perform network operation
    pass
except NetworkError as e:
    print(f"Network error: {e}")
    print(f"Status code: {e.status_code}")
    print(f"Is timeout: {e.is_timeout}")
```

Properties:
- status_code (Optional[int]): HTTP status code if available
- is_timeout (bool): Whether the error is due to timeout
- is_connection_error (bool): Whether it's a connection error

## Configuration Errors

### ConfigurationError

Raised when there are configuration issues.

```python
from teable.exceptions import ConfigurationError

try:
    client = TeableClient(TeableConfig(
        api_url="invalid-url"
    ))
except ConfigurationError as e:
    print(f"Configuration error: {e}")
    print(f"Parameter: {e.parameter}")
```

Properties:
- parameter (str): Name of the invalid configuration parameter
- reason (str): Reason for the configuration error

## Error Handling Best Practices

### Comprehensive Error Handling

```python
from teable.exceptions import (
    TeableError,
    ValidationError,
    AuthenticationError,
    ResourceNotFoundError,
    RateLimitError,
    NetworkError
)

try:
    # Perform operations
    client = TeableClient(config)
    table = client.get_table("table_id")
    record = table.create_record(data)
    
except ValidationError as e:
    # Handle data validation errors
    print(f"Invalid data: {e}")
    
except AuthenticationError as e:
    # Handle authentication issues
    print(f"Authentication failed: {e}")
    
except ResourceNotFoundError as e:
    # Handle missing resources
    print(f"Resource not found: {e}")
    
except RateLimitError as e:
    # Handle rate limiting
    print(f"Rate limit exceeded. Retry after {e.retry_after} seconds")
    
except NetworkError as e:
    # Handle network issues
    print(f"Network error: {e}")
    
except TeableError as e:
    # Handle any other Teable-specific errors
    print(f"Operation failed: {e}")
    
except Exception as e:
    # Handle unexpected errors
    print(f"Unexpected error: {e}")
```

## Next Steps

- [Client Reference](client.md)
- [Models Reference](models.md)
- [Error Handling Guide](../advanced/error-handling.md)
