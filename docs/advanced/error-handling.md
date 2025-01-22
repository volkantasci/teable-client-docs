# Error Handling

This guide covers comprehensive error handling strategies for the Teable-Client library. Learn how to handle various types of errors and implement robust error recovery mechanisms.

## Exception Types

### Core Exceptions

```python
from teable.exceptions import (
    TeableError,           # Base exception for all Teable errors
    ValidationError,       # Data validation errors
    AuthenticationError,   # Authentication failures
    ResourceNotFoundError, # Resource not found errors
    RateLimitError,       # API rate limit exceeded
    NetworkError          # Network-related errors
)
```

## Basic Error Handling

### Simple Try-Catch

```python
from teable import TeableClient, TeableConfig

try:
    client = TeableClient(TeableConfig(
        api_url="https://your-teable-instance.com/api",
        api_key="your-api-key"
    ))
    
    # Perform operations
    table = client.get_table("table_id")
    record = table.create_record({"Name": "Test"})
    
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

## Advanced Error Handling

### Custom Error Handler

```python
class ErrorHandler:
    def __init__(self, logger=None):
        self.logger = logger
        self.retry_count = 0
        self.max_retries = 3
    
    def handle_error(self, error, context=None):
        """Handle different types of errors."""
        if isinstance(error, ValidationError):
            self._handle_validation_error(error, context)
        elif isinstance(error, AuthenticationError):
            self._handle_auth_error(error)
        elif isinstance(error, RateLimitError):
            self._handle_rate_limit_error(error)
        elif isinstance(error, NetworkError):
            self._handle_network_error(error)
        else:
            self._handle_unknown_error(error)
    
    def _handle_validation_error(self, error, context):
        if self.logger:
            self.logger.error(f"Validation error: {error}")
            if context:
                self.logger.error(f"Context: {context}")
        # Implement validation error recovery
    
    def _handle_auth_error(self, error):
        if self.logger:
            self.logger.error(f"Authentication error: {error}")
        # Implement auth error recovery
    
    def _handle_rate_limit_error(self, error):
        if self.logger:
            self.logger.warning(f"Rate limit exceeded: {error}")
        # Implement rate limit handling
    
    def _handle_network_error(self, error):
        if self.logger:
            self.logger.error(f"Network error: {error}")
        # Implement network error recovery
    
    def _handle_unknown_error(self, error):
        if self.logger:
            self.logger.error(f"Unknown error: {error}")
        # Implement unknown error handling
```

### Using the Error Handler

```python
import logging

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Create error handler
error_handler = ErrorHandler(logger)

# Use in operations
try:
    client = TeableClient(config)
    table = client.get_table("table_id")
    record = table.create_record({"Name": "Test"})
except TeableError as e:
    error_handler.handle_error(e, context={"operation": "create_record"})
```

## Retry Mechanisms

### Basic Retry

```python
def retry_operation(operation, max_retries=3, delay=1):
    """Retry an operation with exponential backoff."""
    for attempt in range(max_retries):
        try:
            return operation()
        except (NetworkError, RateLimitError) as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(delay * (2 ** attempt))
```

### Advanced Retry with Backoff

```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1, max_delay=60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            retries = 0
            while retries < max_retries:
                try:
                    return func(*args, **kwargs)
                except (NetworkError, RateLimitError) as e:
                    retries += 1
                    if retries == max_retries:
                        raise
                    
                    delay = min(base_delay * (2 ** (retries - 1)), max_delay)
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

# Usage
@retry_with_backoff(max_retries=3)
def create_record(table, data):
    return table.create_record(data)
```

## Batch Operation Error Handling

### Handling Batch Failures

```python
def handle_batch_operation(table, records, operation='create'):
    """Handle batch operations with error recovery."""
    results = {
        'successful': [],
        'failed': [],
        'errors': []
    }
    
    try:
        if operation == 'create':
            batch_result = table.batch_create_records(records)
        elif operation == 'update':
            batch_result = table.batch_update_records(records)
        else:
            raise ValueError(f"Unsupported operation: {operation}")
            
        # Process results
        results['successful'] = batch_result.successful
        
        # Handle partial failures
        for error in batch_result.errors:
            results['failed'].append({
                'record': records[error.index],
                'error': error.message
            })
            
    except TeableError as e:
        results['errors'].append(str(e))
        
    return results
```

## Best Practices

1. **Error Classification**
   - Categorize errors appropriately
   - Handle specific exceptions before general ones
   - Log errors with context
   - Implement appropriate recovery strategies

2. **Retry Strategies**
   - Use exponential backoff
   - Set maximum retry limits
   - Handle rate limiting
   - Consider operation idempotency

3. **Error Recovery**
   - Implement graceful degradation
   - Maintain data consistency
   - Provide meaningful error messages
   - Consider partial success scenarios

4. **Logging and Monitoring**
   - Log errors with context
   - Monitor error rates
   - Set up alerts
   - Track error patterns

## Next Steps

- [Best Practices](best-practices.md)
- [Authentication](authentication.md)
- [API Reference](../api/client.md)
