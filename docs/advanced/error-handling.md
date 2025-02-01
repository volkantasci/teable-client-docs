# Error Handling

The Teable Client Library provides comprehensive error handling through specific exception types and validation mechanisms. This guide covers how to handle errors effectively in your applications.

## Exception Types

### ValidationError

Raised when input data fails validation:

```python
from teable.exceptions import ValidationError

try:
    # Invalid email format
    client.auth.signin(
        email="invalid-email",
        password="ValidPass123"
    )
except ValidationError as e:
    print(f"Validation error: {str(e)}")
```

Common validation errors:
- Invalid email format
- Password requirements not met
- Required fields missing
- Invalid field types
- Value range violations

### APIError

Raised when the API returns an error response:

```python
from teable.exceptions import APIError

try:
    # Attempt to access non-existent resource
    client.spaces.get_space("invalid_space_id")
except APIError as e:
    print(f"API error: {str(e)}")
    print(f"Status code: {e.status_code}")
    print(f"Error details: {e.details}")
```

Common API errors:
- Resource not found
- Permission denied
- Rate limit exceeded
- Invalid API key
- Server errors

## Error Handling Patterns

### Basic Error Handling

```python
from teable.exceptions import ValidationError, APIError

try:
    # Attempt operation
    record = table.create_record({
        "Name": "",  # Required field is empty
        "Email": "invalid-email"
    })
except ValidationError as e:
    print(f"Validation error: {str(e)}")
except APIError as e:
    print(f"API error: {str(e)}")
except Exception as e:
    print(f"Unexpected error: {str(e)}")
```

### Batch Operation Error Handling

```python
# Batch create records with error handling
records_data = [
    {"Name": "Valid Name", "Email": "valid@example.com"},
    {"Name": "", "Email": "invalid-email"}  # Invalid record
]

try:
    batch_result = table.batch_create_records(records_data)
    
    # Check for partial success
    if batch_result.failure_count > 0:
        print(f"Failed to create {batch_result.failure_count} records")
        for error in batch_result.errors:
            print(f"Error in record {error.index}: {error.message}")
            
    print(f"Successfully created {batch_result.success_count} records")
    
except ValidationError as e:
    print(f"Batch validation error: {str(e)}")
except APIError as e:
    print(f"API error during batch operation: {str(e)}")
```

### Resource Management Errors

```python
try:
    # Create a space
    space = client.spaces.create_space(name="Test Space")
    
    try:
        # Perform operations
        base = space.create_base(name="Test Base")
        # ... more operations ...
        
    finally:
        # Clean up resources even if operations fail
        client.spaces.permanently_delete_space(space.space_id)
        
except APIError as e:
    if e.status_code == 404:
        print("Resource not found")
    elif e.status_code == 403:
        print("Permission denied")
    else:
        print(f"API error: {str(e)}")
```

## Validation Rules

### Field Validation

```python
# Create a table with validation rules
try:
    table = client.tables.create_table(
        base_id=base.base_id,
        name="Employees",
        db_table_name="employees",
        fields=[
            {
                "name": "Name",
                "type": "singleLineText",
                "required": True
            },
            {
                "name": "Age",
                "type": "number",
                "precision": 0,
                "min": 18,
                "max": 100
            }
        ]
    )
except ValidationError as e:
    print(f"Field validation error: {str(e)}")
```

### Record Validation

```python
try:
    # Create record with validation
    record = table.create_record({
        "Name": "John Doe",
        "Age": 150  # Exceeds maximum value
    })
except ValidationError as e:
    print(f"Record validation error: {str(e)}")
```

## Best Practices

### 1. Use Specific Exception Types

```python
from teable.exceptions import ValidationError, APIError

try:
    # Operation
    pass
except ValidationError:
    # Handle validation errors
    pass
except APIError:
    # Handle API errors
    pass
except Exception:
    # Handle unexpected errors
    pass
```

### 2. Implement Retry Logic

```python
import time
from teable.exceptions import APIError

def retry_operation(operation, max_retries=3, delay=1):
    for attempt in range(max_retries):
        try:
            return operation()
        except APIError as e:
            if e.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    time.sleep(delay * (attempt + 1))
                    continue
            raise
    return None

# Usage
result = retry_operation(
    lambda: client.spaces.get_spaces()
)
```

### 3. Resource Cleanup

```python
def safe_resource_operation():
    space = None
    try:
        # Create resources
        space = client.spaces.create_space(name="Temporary Space")
        # Perform operations
        return True
    except Exception as e:
        print(f"Operation failed: {str(e)}")
        return False
    finally:
        # Clean up resources
        if space:
            try:
                client.spaces.permanently_delete_space(space.space_id)
            except Exception as e:
                print(f"Cleanup failed: {str(e)}")
```

### 4. Logging Errors

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

try:
    # Operation
    pass
except ValidationError as e:
    logger.error(f"Validation error: {str(e)}")
    # Handle error
except APIError as e:
    logger.error(f"API error: {str(e)}", extra={
        "status_code": e.status_code,
        "details": e.details
    })
    # Handle error
```

## Common Error Scenarios

### 1. Authentication Errors

```python
try:
    client.auth.signin(email="user@example.com", password="wrong")
except APIError as e:
    if e.status_code == 401:
        print("Invalid credentials")
    elif e.status_code == 403:
        print("Account locked")
```

### 2. Resource Not Found

```python
try:
    table = client.get_table("non_existent_id")
except APIError as e:
    if e.status_code == 404:
        print("Table not found")
```

### 3. Rate Limiting

```python
try:
    # Batch operation
    pass
except APIError as e:
    if e.status_code == 429:
        print("Rate limit exceeded")
        print(f"Retry after: {e.headers.get('Retry-After')} seconds")
```

## Error Prevention

1. **Validate Input Data**:
   - Check data types before API calls
   - Validate required fields
   - Verify value ranges

2. **Handle Rate Limits**:
   - Implement backoff strategies
   - Monitor API usage
   - Cache responses when possible

3. **Resource Management**:
   - Always clean up temporary resources
   - Use context managers when appropriate
   - Implement proper error recovery

4. **Authentication**:
   - Verify credentials before operations
   - Handle token expiration
   - Implement proper security practices
