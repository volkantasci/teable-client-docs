# Best Practices

This guide covers best practices and recommended patterns for using the Teable-Client library effectively. Learn about optimization, security, error handling, and performance considerations.

## Code Organization

### Client Configuration

```python
from teable import TeableClient, TeableConfig
import os

def create_client(environment='production'):
    """Create a properly configured client instance."""
    configs = {
        'production': {
            'api_url': os.getenv('TEABLE_PROD_API_URL'),
            'api_key': os.getenv('TEABLE_PROD_API_KEY')
        },
        'staging': {
            'api_url': os.getenv('TEABLE_STAGING_API_URL'),
            'api_key': os.getenv('TEABLE_STAGING_API_KEY')
        }
    }
    
    env_config = configs.get(environment)
    if not env_config:
        raise ValueError(f"Invalid environment: {environment}")
        
    config = TeableConfig(
        api_url=env_config['api_url'],
        api_key=env_config['api_key']
    )
    
    return TeableClient(config)
```

### Resource Management

```python
class TeableManager:
    """Centralized manager for Teable resources."""
    
    def __init__(self, client):
        self.client = client
        self._space_cache = {}
        self._base_cache = {}
        self._table_cache = {}
    
    def get_space(self, space_id):
        """Get space with caching."""
        if space_id not in self._space_cache:
            self._space_cache[space_id] = self.client.get_space(space_id)
        return self._space_cache[space_id]
    
    def get_base(self, base_id):
        """Get base with caching."""
        if base_id not in self._base_cache:
            self._base_cache[base_id] = self.client.get_base(base_id)
        return self._base_cache[base_id]
    
    def get_table(self, table_id):
        """Get table with caching."""
        if table_id not in self._table_cache:
            self._table_cache[table_id] = self.client.get_table(table_id)
        return self._table_cache[table_id]
    
    def clear_cache(self):
        """Clear all caches."""
        self._space_cache.clear()
        self._base_cache.clear()
        self._table_cache.clear()
```

## Performance Optimization

### Batch Operations

```python
def batch_process_records(table, records, batch_size=1000):
    """Process records in batches for optimal performance."""
    for i in range(0, len(records), batch_size):
        batch = records[i:i + batch_size]
        result = table.batch_create_records(batch)
        yield result

# Usage
records = [{"Name": f"Record {i}"} for i in range(10000)]
for result in batch_process_records(table, records):
    print(f"Processed batch: {result.success_count} successful")
```

### Query Optimization

```python
def optimize_query(table, filters=None, sort=None, fields=None):
    """Create optimized queries."""
    query = table.query()
    
    # Add specific field projection
    if fields:
        query = query.select(fields)
    
    # Add filters
    if filters:
        for field, operator, value in filters:
            query = query.filter(field, operator, value)
    
    # Add sorting
    if sort:
        for field, direction in sort:
            query = query.sort(field, direction)
    
    return query

# Usage
query = optimize_query(
    table,
    filters=[("Status", "=", "Active")],
    sort=[("CreatedDate", "desc")],
    fields=["Name", "Status", "CreatedDate"]
)
records = table.get_records(query=query)
```

## Data Validation

### Field Validation

```python
def validate_record_data(table, data):
    """Validate record data before creation/update."""
    required_fields = [f for f in table.fields if f.is_required]
    
    # Check required fields
    for field in required_fields:
        if field.name not in data:
            raise ValidationError(f"Missing required field: {field.name}")
    
    # Validate field values
    for field_name, value in data.items():
        field = next((f for f in table.fields if f.name == field_name), None)
        if field:
            field.validate_value(value)
        else:
            raise ValidationError(f"Unknown field: {field_name}")
    
    return True
```

### Data Cleaning

```python
def clean_record_data(data):
    """Clean and normalize record data."""
    cleaned = {}
    
    for key, value in data.items():
        # Remove whitespace
        if isinstance(value, str):
            value = value.strip()
        
        # Convert empty strings to None
        if value == "":
            value = None
            
        # Skip None values
        if value is not None:
            cleaned[key] = value
    
    return cleaned
```

## Error Handling

### Comprehensive Error Handling

```python
from contextlib import contextmanager

@contextmanager
def teable_operation():
    """Context manager for Teable operations."""
    try:
        yield
    except ValidationError as e:
        logger.error(f"Validation error: {e}")
        raise
    except AuthenticationError as e:
        logger.error(f"Authentication error: {e}")
        raise
    except RateLimitError as e:
        logger.warning(f"Rate limit exceeded: {e}")
        raise
    except TeableError as e:
        logger.error(f"Teable error: {e}")
        raise
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        raise

# Usage
with teable_operation():
    record = table.create_record(data)
```

## Security

### Secure Configuration

```python
def load_secure_config():
    """Load secure configuration from environment."""
    required_vars = [
        'TEABLE_API_URL',
        'TEABLE_API_KEY'
    ]
    
    # Verify all required variables
    missing = [var for var in required_vars if not os.getenv(var)]
    if missing:
        raise ValueError(f"Missing environment variables: {missing}")
    
    return TeableConfig(
        api_url=os.getenv('TEABLE_API_URL'),
        api_key=os.getenv('TEABLE_API_KEY')
    )
```

## Logging and Monitoring

### Enhanced Logging

```python
import logging

def setup_logging():
    """Set up enhanced logging."""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    # Create file handler
    fh = logging.FileHandler('teable.log')
    fh.setLevel(logging.INFO)
    
    # Create console handler
    ch = logging.StreamHandler()
    ch.setLevel(logging.INFO)
    
    # Create formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    fh.setFormatter(formatter)
    ch.setFormatter(formatter)
    
    # Get logger
    logger = logging.getLogger('teable')
    logger.addHandler(fh)
    logger.addHandler(ch)
    
    return logger
```

## Testing

### Test Utilities

```python
class TeableTestCase:
    """Base class for Teable tests."""
    
    def setUp(self):
        """Set up test environment."""
        self.client = TeableClient(TeableConfig(
            api_url=os.getenv('TEABLE_TEST_API_URL'),
            api_key=os.getenv('TEABLE_TEST_API_KEY')
        ))
        
    def tearDown(self):
        """Clean up test environment."""
        # Clean up test data
        pass
    
    def create_test_space(self):
        """Create a test space."""
        return self.client.create_space(f"Test Space {uuid.uuid4()}")
    
    def create_test_base(self, space):
        """Create a test base."""
        return space.create_base(f"Test Base {uuid.uuid4()}")
```

## General Best Practices

1. **Resource Management**
   - Use context managers
   - Implement proper cleanup
   - Cache frequently accessed resources
   - Monitor resource usage

2. **Performance**
   - Use batch operations
   - Optimize queries
   - Implement caching
   - Monitor performance metrics

3. **Error Handling**
   - Implement comprehensive error handling
   - Use appropriate error types
   - Log errors with context
   - Implement retry mechanisms

4. **Security**
   - Use environment variables
   - Implement proper authentication
   - Validate input data
   - Follow security best practices

## Next Steps

- [Authentication](authentication.md)
- [Error Handling](error-handling.md)
- [API Reference](../api/client.md)
