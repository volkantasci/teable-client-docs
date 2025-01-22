# API Reference: Exceptions

This document provides detailed API reference for all exception classes in the Teable-Client library.

## Exception Hierarchy

```
TeableError
├── APIError
│   ├── RateLimitError
│   ├── ResourceNotFoundError
│   └── AuthenticationError
├── ValidationError
├── ConfigurationError
├── NetworkError
└── BatchOperationError
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
```

### APIError

Base exception for all API-related errors. Provides additional context about the API response.

```python
from teable.exceptions import APIError

try:
    # Perform API operation
    pass
except APIError as e:
    print(f"API error: {e}")
    print(f"Status code: {e.status_code}")
    print(f"Response body: {e.response_body}")
```

Properties:
- message (str): Human-readable error description
- status_code (Optional[int]): HTTP status code if available
- response_body (Optional[str]): Raw response body if available

## Validation Errors

### ValidationError

Raised when data validation fails. This could be due to:
- Invalid field types
- Missing required fields
- Value constraints violations

```python
from teable.exceptions import ValidationError

try:
    table.create_record({
        "Name": "John Doe",
        "Age": "invalid"  # Should be a number
    })
except ValidationError as e:
    print(f"Validation failed: {e}")
```

## Authentication Errors

### AuthenticationError

Raised when authentication fails. This could be due to:
- Invalid API key
- Expired credentials
- Insufficient permissions

```python
from teable.exceptions import AuthenticationError

try:
    client = TeableClient(TeableConfig(
        api_url="https://example.com",
        api_key="invalid-key"
    ))
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
    print(f"Status code: {e.status_code}")
```

## Resource Errors

### ResourceNotFoundError

Raised when a requested resource is not found. This could be due to:
- Invalid record ID
- Invalid table ID
- Invalid view ID
- Invalid field ID

```python
from teable.exceptions import ResourceNotFoundError

try:
    table = client.tables.get("non-existent-id")
except ResourceNotFoundError as e:
    print(f"Resource not found: {e}")
    print(f"Resource type: {e.resource_type}")
    print(f"Resource ID: {e.resource_id}")
```

Properties:
- resource_type (str): Type of resource that wasn't found
- resource_id (str): ID of the resource that wasn't found
- status_code (Optional[int]): HTTP status code (typically 404)

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
    print(f"Rate limit: {e.limit}")
```

Properties:
- reset_time (Optional[float]): Unix timestamp when the rate limit resets
- limit (Optional[int]): The rate limit ceiling for the given endpoint
- status_code (Optional[int]): HTTP status code (typically 429)

## Network Errors

### NetworkError

Raised when network-related issues occur. This could be due to:
- Connection timeouts
- DNS resolution failures
- SSL/TLS errors

```python
from teable.exceptions import NetworkError

try:
    # Perform network operation
    pass
except NetworkError as e:
    print(f"Network error: {e}")
```

## Configuration Errors

### ConfigurationError

Raised when there are issues with client configuration. This could be due to:
- Missing API credentials
- Invalid API URL
- Missing required configuration parameters

```python
from teable.exceptions import ConfigurationError

try:
    client = TeableClient(TeableConfig(
        api_url="invalid-url"
    ))
except ConfigurationError as e:
    print(f"Configuration error: {e}")
```

## Batch Operation Errors

### BatchOperationError

Raised when a batch operation partially fails.

```python
from teable.exceptions import BatchOperationError

try:
    # Perform batch operation
    pass
except BatchOperationError as e:
    print(f"Batch operation error: {e}")
    print(f"Successful operations: {e.successful_operations}")
    print(f"Failed operations: {e.failed_operations}")
```

Properties:
- successful_operations (list): List of successfully processed items
- failed_operations (list): List of failed operations with their errors

## Error Handling Best Practices

### Comprehensive Error Handling

```python
from teable.exceptions import (
    TeableError,
    APIError,
    ValidationError,
    AuthenticationError,
    ResourceNotFoundError,
    RateLimitError,
    NetworkError,
    ConfigurationError,
    BatchOperationError
)

try:
    # Perform operations
    client = TeableClient(config)
    table = client.tables.get("table_id")
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
    print(f"Rate limit exceeded. Reset at {e.reset_time}")
    
except NetworkError as e:
    # Handle network issues
    print(f"Network error: {e}")
    
except ConfigurationError as e:
    # Handle configuration issues
    print(f"Configuration error: {e}")
    
except BatchOperationError as e:
    # Handle batch operation failures
    print(f"Batch operation partially failed: {e}")
    
except APIError as e:
    # Handle any other API-related errors
    print(f"API error: {e}")
    
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
