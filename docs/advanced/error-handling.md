# Error Handling

This guide covers error handling strategies for the Teable-Client library.

## Exception Types

### Core Exceptions

```python
from teable.exceptions import (
    TeableError,           # Base exception for all Teable errors
    APIError,             # Base exception for API-related errors
    ValidationError,       # Data validation errors
    AuthenticationError,   # Authentication failures
    ResourceNotFoundError, # Resource not found errors
    RateLimitError,       # API rate limit exceeded
    NetworkError,         # Network-related errors
    ConfigurationError,   # Configuration-related errors
    BatchOperationError   # Batch operation failures
)
```

## Basic Error Handling

### Simple Try-Catch

```python
from teable import TeableClient, TeableConfig
from teable.exceptions import (
    TeableError,
    ValidationError,
    AuthenticationError,
    ResourceNotFoundError
)

try:
    # Initialize client
    client = TeableClient(TeableConfig(
        api_url="https://your-teable-instance.com/api",
        api_key="your-api-key"
    ))
    
    # Get a table
    table = client.tables.get("table_id")
    
    # Create a record
    record = client.records.create(
        table_id="table_id",
        fields={"Name": "Test"}
    )
    
except ValidationError as e:
    print(f"Data validation error: {e}")
except AuthenticationError as e:
    print(f"Authentication failed: {e}")
except ResourceNotFoundError as e:
    print(f"Resource not found: {e}")
except TeableError as e:
    print(f"Teable error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Handling API Errors

```python
from teable.exceptions import APIError

try:
    response = client.tables.get("table_id")
except APIError as e:
    print(f"API error: {e}")
    print(f"Status code: {e.status_code}")
    print(f"Response body: {e.response_body}")
```

## Handling Rate Limits

```python
from teable.exceptions import RateLimitError

try:
    # Perform many operations
    for i in range(1000):
        client.records.create(
            table_id="table_id",
            fields={"Name": f"Test {i}"}
        )
except RateLimitError as e:
    print(f"Rate limit exceeded: {e}")
    print(f"Reset time: {e.reset_time}")
    print(f"Rate limit: {e.limit}")
```

## Handling Batch Operations

```python
from teable.exceptions import BatchOperationError

try:
    # Batch create records
    records = [
        {"Name": f"Test {i}"}
        for i in range(100)
    ]
    
    result = client.records.batch_create_records(
        table_id="table_id",
        records=records
    )
    
    # Check results
    print(f"Successfully created: {result.success_count}")
    print(f"Failed operations: {result.failure_count}")
    print(f"Success rate: {result.success_rate}%")
    
    # Process successful records
    for record in result.successful:
        print(f"Created record: {record.record_id}")
    
    # Process failed operations
    for failure in result.failed:
        print(f"Failed operation: {failure}")
        
except BatchOperationError as e:
    print(f"Batch operation error: {e}")
    print(f"Successful operations: {e.successful_operations}")
    print(f"Failed operations: {e.failed_operations}")
```

## Best Practices

1. **Handle Specific Exceptions**
   - Catch specific exceptions before general ones
   - Handle each type of error appropriately
   - Provide meaningful error messages

2. **Batch Operations**
   - Use batch operations for multiple records
   - Check success and failure counts
   - Process successful and failed operations separately

3. **Rate Limiting**
   - Handle rate limit errors gracefully
   - Implement appropriate delays when needed
   - Monitor rate limit usage

4. **Error Logging**
   - Log errors with context
   - Include relevant error details
   - Use appropriate log levels

## Next Steps

- [Best Practices](best-practices.md)
- [Authentication](authentication.md)
- [API Reference](../api/client.md)
